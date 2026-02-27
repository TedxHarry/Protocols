# A Day in the Life — Real Integration Scenario From Start to Finish

Theory is useful. Watching it applied to a complete, realistic scenario is how it becomes usable.

GlobalTech (12,000 employees, manufacturing and software divisions) is replacing their legacy on-premises HR system with Workday. The legacy system has been the authoritative source for employee identity in ISC for four years. The migration needs to happen without governance continuity gaps — employees can't lose access during the cutover, and terminations can't be delayed while two HR systems are in parallel operation.

This is a real category of integration project. The scenario, the problems encountered, and the decisions made are representative of actual ISC integration work.

---

## Phase 1: Requirements (Week 1)

**Day 1 — Stakeholder Meeting**

In the room: HR Director (owns Workday deployment), CISO (owns governance requirements), IT Director (owns ISC), ISC Admin (has run ISC for 4 years on the legacy source), and you, the integration engineer.

The stated requirement: "Connect Workday to ISC and deprecate the legacy HR connector."

What you hear: a project with unknown scope, unclear cutover risk, and at least three stakeholders who have different definitions of success.

You run discovery. The questions from the previous document, applied to this situation:

**Q: What data does Workday expose?**
HR Director confirms: Workday exposes employee data via SOAP/REST API (Workday REST API v34+). Key attributes available: Worker_ID, Legal name, Preferred name, Primary Work Email, Organization (department hierarchy), Job_Title, Worker_Type, Active_Status, Manager (via Worker_ID reference), Hire_Date, Termination_Date, Business_Site (location), Cost_Center, Job_Profile (job code).

**Q: Is Workday source-only, or will ISC provision back to it?**
Source-only. HR owns Workday. ISC reads it, doesn't write to it.

**Q: What attributes does the legacy system provide that Workday might not?**
ISC Admin pulls up the legacy connector configuration. Finds: the legacy system provided a custom `employeeType` field that maps to contractor/temp worker/FTE in ISC's provisioning policies. Workday has `Worker_Type` but the values are different: "Regular" vs. "FTE", "Contract" vs. "Contractor". Transform needed.

**Q: How many employees, and what's the current correlation approach?**
12,000 active records in legacy system. Correlation was done via employee ID (the legacy system used numeric IDs like `48291`; ISC stored them). Workday uses `GT-48291` format. All existing ISC identities have the numeric format. Workday will provide the prefixed format. Correlation rule update required.

**Q: What's the latency requirement for terminations?**
CISO: Terminations need to process within 15 minutes for privileged users (IT admins, finance executives), within 4 hours for standard users. The legacy system was nightly batch — this has been a known gap for two years and the CISO wants it fixed as part of this project.

**Q: Any edge cases to plan for?**
ISC Admin: The legacy system had 340 contractor records that weren't in the main HR system — they came from a spreadsheet the Operations team maintained. Need to determine if Workday will have all of them, or if there's a separate contractor source.

HR Director: Workday has all FTE contractors. Temporary agency workers (about 60 of the 340) are NOT in Workday — they're in a separate vendor management system. That integration is out of scope for this project. (Make a note: 60 contractor accounts will lose their identity source at cutover. Needs a decision.)

**Deliverable: Integration Design Document**

After the meeting plus two follow-up calls, you produce a design document covering:
- Attribute mapping (Workday attribute → ISC identity attribute, with transformation notes)
- Correlation strategy (updated to handle `GT-` prefix)
- Authoritative attribute definitions (what Workday owns, what AD continues to own)
- Termination handling approach (Workday webhook + ISC workflow for real-time processing)
- Contractor handling decision (60 temp workers → need separate governance track, flagged for separate project)
- Cutover plan outline (parallel operation period, validation approach, switchover criteria)

Stakeholders review and sign off on the design document before implementation begins.

> 🎯 **KEY POINT:** The design document sign-off is the contract. Every decision in the design document was reviewed and approved. When someone says "that's not what we meant" after implementation, the design document is the reference point. This doesn't make disagreements disappear — but it moves them to the right time (before build), not the wrong time (after go-live).

---

## Phase 2: Design (Week 2)

**Applying the Three-Question Framework**

**What data needs to move?**
Workday → ISC (source). Not ISC → Workday (source-only).

Attributes to aggregate: Worker_ID, Legal name (first/last separately), Primary Work Email, Organization hierarchy, Job_Title, Worker_Type, Active_Status, Manager reference, Hire_Date, Termination_Date, Cost_Center, Business_Site.

Attributes NOT to aggregate: Compensation, banking information, personal phone numbers, medical leave details.

**When does data need to move?**
- Standard employee attribute changes: nightly full aggregation + 4-hour delta
- Terminations: event-driven via Workday Integration Studio webhook (sends event within minutes of status change in Workday)
- New hires: event-driven for start-date trigger (pre-create accounts day before start date)

**How complex are the transformations?**
- Worker_ID strip prefix: `Substring(Account.Worker_ID, 3)` (remove "GT-") → built-in transform
- Worker_Type normalization: Lookup table (`Regular` → `Full-Time`, `Contract` → `Contractor`, `Temporary` → `Temp`) → built-in Lookup transform
- Department from hierarchy: Workday provides full org path `Engineering > Software > Backend Platform`. Need top-level only. Custom rule required (string split, take first element, normalize).
- Manager reference: Workday provides manager's Worker_ID. ISC needs manager's email. Custom rule: look up manager's identity by Worker_ID, return email attribute.
- Email: direct map from `Primary_Work_Email` → `email`

Two custom rules needed. Rest handled by built-in transforms.

**Attribute authority decisions** (since AD is also a source for some attributes):

| Attribute | Authority | Logic |
|---|---|---|
| email | Workday | HR owns email format |
| displayName | ISC calculated | Concat(firstname + " " + lastname) |
| department | Workday | HR owns org structure |
| jobTitle | Workday | HR owns titles |
| manager | Workday | HR owns reporting structure |
| samAccountName | Active Directory | AD names assigned by IT |

---

## Phase 3: Implementation (Weeks 3–4)

**Connector Configuration**

Workday ISC connector setup:
- Authentication: OAuth 2.0 with ISC registered as OAuth client in Workday tenant
- API version: Workday REST API v34
- Aggregation endpoint: `/ccx/api/v1/{tenant}/workers`
- Entitlements: Workday doesn't have entitlements ISC manages (source-only). Entitlement aggregation not applicable.
- VA: Workday is a cloud system → VA not required for direct cloud-to-cloud connectivity

**Aggregation testing — Sandbox**

Run aggregation against Workday sandbox tenant (GlobalTech's Workday sandbox has 847 test records).

Results: 847 accounts processed. Correlation result: 799 correlated (94.3%), 48 uncorrelated.

Investigate the 48:
- 22 are test accounts in Workday sandbox without corresponding ISC test identities (expected — sandbox data mismatch, not a real problem)
- 15 have Worker_ID that doesn't match any existing ISC identity (these are new sandbox employees added after ISC sandbox was last refreshed)
- 8 have department codes that aren't in the lookup table (`ENG-NEW` — a new department created for the sandbox that wasn't in the lookup table configuration)
- 3 have null Primary_Work_Email (test accounts created without email)

Three real issues to fix:
1. Add `ENG-NEW` → `Engineering` to the department lookup table (and flag the process gap: who updates the lookup table when new departments are created?)
2. Add null email handling to the correlation fallback (use Worker_ID as secondary correlation when email is null)
3. Document the process for keeping sandbox data in sync with production

**Real discovery during implementation**: When building the manager reference rule, find that 23 active Workday records have manager Worker_IDs that don't match any active Workday record — either managers who've left (Workday shows departed manager as terminated but the employee record still references them) or data entry errors. Escalate to HR. HR corrects 18 records in Workday. 5 are intentional (employees in a "no manager" holding state during reorganization — configure ISC to skip manager attribute if null rather than error).

> 💡 **REMEMBER:** Implementation always surfaces data quality problems you didn't discover in requirements. This is normal. Budget time for it. The integration engineer's value isn't just building the connection — it's surfacing these problems and working with business owners to fix them at the source.

---

## Phase 4: Testing (Week 5)

**Full aggregation against production Workday (read-only — no ISC production changes yet)**:
12,000 records processed. Run against a cloned version of ISC production to test correlation at full scale.

Results:
- 11,847 correlate to existing ISC identities (correlation against legacy employee ID after prefix strip)
- 93 uncorrelated — investigate:
  - 47 are the agency contractors not in Workday (as expected from requirements)
  - 31 are employees who joined in the last 6 months and have a different ID format from a recent Workday configuration change
  - 15 are data entry errors in Workday (duplicate records, typos in Worker_ID)

For the 31 with format difference: Workday changed from `GT-XXXXX` to `GTX-XXXXX` format for employees hired after June 2024. The correlation rule needs to handle both formats. Update the Substring transform to strip variable-length prefix.

For the 15 data errors: work with HR team to fix in Workday. This takes 3 days.

**Termination webhook testing**:
Working with Workday admin, configure a test integration in Workday to send termination events. Trigger a test termination for a test identity. ISC receives the event. Lifecycle state changes to Terminated. Deprovisioning workflow fires. All accounts disabled. Total elapsed time: 8 minutes from Workday action to all accounts disabled.

CISO requested 15 minutes for privileged users. 8 minutes is better than spec. Good.

**Edge case testing**:
- Contractor account (not in Workday): account remains correlated to legacy source until legacy connector is deprecated. Then becomes uncorrelated. Decision: maintain legacy connector for contractor accounts as a secondary source past the Workday cutover. Not in original scope — added.
- Executive account with non-standard email: manual correlation override established for 3 executive accounts with custom email formats.
- Hire date in future (pre-boarded employee): ISC correctly holds account creation until start date based on `lifecycleState` calculation from `Hire_Date`.

---

## Phase 5: Deployment (Week 6)

**Change management**: CISO, HR Director, IT Director briefed on cutover plan. Email to IT team about the switchover window.

**Cutover plan**:
1. Sunday 10 PM: disable Workday aggregation source (prevents parallel conflicts)
2. Run final aggregation from legacy system
3. Enable Workday connector
4. Run first full Workday aggregation in production
5. Verify correlation results against expected numbers
6. If results match expected (>99% correlation rate): proceed
7. Enable Workday termination webhook
8. Disable legacy HR connector (set to read-only monitoring mode for 30 days, not fully removed yet)
9. Monitor first 48 hours with daily status check-ins

**Cutover night**: Full aggregation completes in 41 minutes (12,000 records). 11,891 correlated (99.1%). 109 uncorrelated. Review the 109: 47 agency contractors (expected, documented), 62 new from the Workday ID format change discovered in testing (the fix was deployed but 62 records were still missed — investigation finds another ID format variant: `G-XXXXX` for executives. Quick rule update. Re-run delta. 62 more correlate. Now at 99.5%).

**Week 1 post-launch monitoring**:
- Day 2: One termination event fails to process (Workday webhook sent malformed JSON for a specific termination type — medical/disability termination uses a different Workday workflow). Fix the webhook handler to parse the alternate format. Re-process the event manually. Account disabled 3 hours late — documented exception, within acceptable window.
- Day 5: One manager reference error (manager left company but wasn't terminated in Workday until 2 days after IT disabled the manager's account — the ISC lookup rule tries to find the manager in ISC, finds the account but it's disabled, returns null instead of the manager's email). Edge case: handle disabled manager accounts in the lookup rule.
- Day 7: Everything stable. Hand off to ongoing operations team with documented known issues and their resolutions.

---

## What You Learn From This

**The technical work was 40% of the effort.** The rest: stakeholder alignment, requirements discovery, data quality remediation, edge case investigation, communication about delays and issues, change management.

**Real integrations always surface data problems.** The Workday ID format change for executives, the malformed webhook for specific termination types, the contractors not in Workday — none of these were in the original requirements. They emerged from actually running the system against real data.

**Plan for things you haven't thought of yet.** The agency contractor gap, the executive email format, the disabled-manager-in-lookup edge case — all were solvable but all required design decisions that weren't made in advance. Build buffer time for the unexpected.

**Deprovisioning being event-driven changed the CISO's posture.** Moving from 24-hour nightly batch to 8-minute event-driven termination was the highest-value outcome of the project. The CISO said "this is the first time I've felt like we actually have control over terminations."

## Key Takeaways

- Real integration projects surface data quality problems, scope gaps, and edge cases that no requirements document anticipates — budget time for discovery and remediation
- The design document is your contract with stakeholders — get sign-off before implementation
- Test against the full production dataset, not a sample — edge cases only appear at scale
- Deprovisioning is a higher-value outcome than provisioning for most stakeholders — it's what reduces security risk
- Post-launch monitoring for at least 7 days is essential — the first real data through a new integration always reveals something unexpected

> 🔗 **CONNECTS TO:** You've seen a complete integration scenario. The final document in Phase 3 zooms out to the architectural level — how to choose between integration approaches (hub-and-spoke, event-driven, flat file) when designing the overall integration architecture.
