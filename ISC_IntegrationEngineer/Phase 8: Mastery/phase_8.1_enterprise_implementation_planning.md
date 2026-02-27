# Planning a Large-Scale ISC Rollout — The 18-Month Implementation

Enterprise ISC implementations rarely fail because of technical problems. They fail because of sequencing problems, scope creep, stakeholder misalignment, and underestimating the organizational change required to shift from manual access management to automated identity governance.

This document maps a realistic 18-month implementation plan for a large enterprise — the phases, the decisions at each gate, the typical stumbling blocks, and the factors that determine whether you arrive at month 18 with a functioning governance platform or another partially-complete IT project.

The baseline scenario: Meridian Capital, 3,200 employees, 28 target systems, regulatory compliance requirements. Starting from zero ISC deployment.

---

## Why Phasing Matters

Every enterprise ISC project scope includes more than can be delivered at once. The temptation is to connect everything simultaneously — one giant implementation that produces full governance at the end. This approach produces:

- A 24-month project with no governance value for the first 22 months
- Scope changes that require rework of already-completed configuration
- Stakeholder patience exhausted before go-live
- A complex system that nobody fully understands because it was built all at once

Phased delivery produces governance value incrementally, builds organizational knowledge progressively, and creates momentum that sustains the project through the inevitable difficult periods.

---

## Phase 1: Foundation — Weeks 1–8

**Goal**: ISC is configured, identity data is flowing, and the platform team can operate it.

**Deliverables**:
- ISC tenant configured (VA clusters deployed, basic admin setup)
- Workday connector live (authoritative HR source aggregating)
- Identity profiles configured for all standard lifecycle states
- First full aggregation completed with >99% correlation
- ISC admin team trained on day-to-day operations
- Monitoring and alerting configured (VA health, aggregation failures)

**Key decisions in Phase 1**:
- VA placement and cluster architecture (settled in week 1 — affects everything else)
- Identity attribute model (which attributes ISC stores; this is hard to change later)
- Correlation attribute selection (must be consistent across all future systems)
- Identity Profile design (Pre-hire, Active, LOA, Terminated lifecycle states)

**Common Phase 1 problems**:
- Workday HR data quality is worse than expected (40 missing employee IDs, 20 contractor records with wrong format). Budget 2 weeks of data remediation here — it exists in virtually every implementation.
- VA connectivity to Workday takes longer than expected due to corporate proxy configuration. Start VA deployment in week 1, not week 3.
- ISC admin team training is underbudgeted. Operators who understand what they're operating respond to problems faster and make fewer configuration mistakes.

**Phase 1 exit criteria**:
- [ ] Workday aggregating with < 0.5% uncorrelated human accounts
- [ ] All standard lifecycle states tested with test identities
- [ ] ISC admin team can run daily operations without implementation team support
- [ ] Monitoring alerts configured and tested

---

## Phase 2: Directory Integration — Weeks 9–16

**Goal**: Active Directory is connected as both source (for AD account data) and managed target (for AD account provisioning).

**Deliverables**:
- AD connector configured for all in-scope domains (3 at Meridian)
- AD account aggregation running with >99% correlation
- AD provisioning tested (create, enable, disable, update)
- Basic role model: `Standard Employee Access` IT Role provisioning AD account
- Lifecycle actions: Pre-hire creates AD account; Terminated disables AD account

**Key decisions in Phase 2**:
- AD OU structure for new accounts provisioned by ISC (which OU do new hires land in?)
- AD group strategy (which AD groups become ISC-governed access profiles?)
- Service account governance approach (exclude from correlation? separate track?)
- Provisioning policy for AD account naming (samAccountName generation logic)

**Common Phase 2 problems**:
- AD OU structure wasn't documented before implementation started. Three weeks in, you discover the OU hierarchy doesn't match what HR calls "departments." AD team needs to restructure, or you build a complex mapping.
- Legacy AD accounts (from acquisitions, retired systems) surface during aggregation. Each batch of legacy accounts needs investigation — don't ignore them.
- AD provisioning requires the ISC service account to have write permissions on specific OUs. Getting the permissions approved and implemented takes longer than expected (change management process, network security review).

**Phase 2 exit criteria**:
- [ ] All in-scope AD domains aggregating
- [ ] New hire end-to-end provisioning tested (Workday create → ISC Pre-hire → AD account provisioned)
- [ ] Termination end-to-end tested (Workday terminate → ISC lifecycle change → AD account disabled)
- [ ] AD service account exclusions configured

---

## Phase 3: Priority Applications — Weeks 17–28

**Goal**: The highest-priority applications (financial systems at Meridian; typically 5–8 applications) are connected and governed. First access certification campaign completed.

**Deliverables**:
- 6–8 priority applications connected (Advent, Charles River, Bloomberg, Salesforce, ServiceNow)
- Access profiles created for all in-scope access tiers
- Business roles defined for primary job functions
- First certification campaign designed and executed
- SoD policies defined and enabled in detective mode

**Key decisions in Phase 3**:
- Access profile granularity (the decision that takes the longest to reach consensus)
- Approval workflow design (who approves what access? what's the chain?)
- First certification campaign scope (start narrow — financial systems only; prove the process works before scaling)
- SoD policy inventory (collaborate with Internal Audit; expect 4–6 weeks of workshops)

**Common Phase 3 problems**:
- Access profile design takes 3x longer than expected because application owners can't agree on what their access tiers mean. Set a deadline: 2 stakeholder workshops, then the integration team makes the call based on available information. Perfection is the enemy of progress.
- First certification campaign overwhelms reviewers (400 items, 2-week deadline, no descriptions on access profiles). Run a small pilot certification with one department before the full campaign. Learn from it.
- Existing Salesforce user accounts don't correlate because email formats differ from Workday. 3 weeks of data remediation. This is the Phase 2 data quality problem recurring in a new system.

**Phase 3 exit criteria**:
- [ ] Priority systems aggregating and provisioning
- [ ] First certification campaign complete with >80% completion rate
- [ ] SoD policies enabled in detective mode with defined remediation process
- [ ] Role-based provisioning working for primary job functions

> 🎯 **KEY POINT:** Phase 3's first certification campaign is the moment the business sees ISC producing governance value. A successful campaign — clean scope, understandable descriptions, real revocations acted on — builds stakeholder confidence. A failed campaign (low completion, rubber-stamps, unresolved remediations) kills momentum. Design the first campaign to succeed.

---

## Phase 4: Expansion — Weeks 29–44

**Goal**: Remaining systems (15–20 additional) connected. Role model expanded to cover all major job functions.

**Deliverables**:
- All remaining in-scope systems connected
- Role model covering 80% of job functions
- SoD policies in preventive mode (block conflicting access requests)
- Certification program running on full schedule (quarterly for high-risk, annually for standard)
- Compliance evidence package produced for first regulatory audit

**Common Phase 4 problems**:
- Systems that "everyone said would be easy" turn out to have complex authentication, unusual data models, or undocumented APIs. Add 20% buffer to integration estimates.
- Role model starts to proliferate (see Phase 5.2). Establish a role governance board to review and approve new role requests.
- With 25+ systems connected, VA performance degrades. Review VA cluster sizing; implement delta aggregations for all high-volume sources.

---

## Phase 5: Advanced Features — Weeks 45–56

**Goal**: AI recommendations, advanced workflows, real-time event integration, and full compliance automation.

**Deliverables**:
- Event-driven termination via Workday webhook (< 15-minute termination processing)
- AI access recommendations active in certification campaigns
- Advanced certification campaigns (risk-based scoping, escalation rules)
- SoD policy coverage extended based on Internal Audit feedback from Phase 3–4

**The threshold moment**: By Phase 5, ISC has enough historical data (aggregations, certification decisions, access patterns) for its AI capabilities to produce meaningful recommendations. Organizations that reach this stage with clean data and well-designed role models see certification completion rates improve and reviewer time-per-item decrease.

---

## Phase 6: Optimization — Weeks 57–72 (Ongoing)

**Goal**: Continuous improvement. ISC is operational; focus shifts to measuring outcomes and improving them.

**Ongoing activities**:
- Quarterly review of aggregation performance (optimize slow aggregations)
- Annual role model review (retire unused roles, split roles that have grown too large)
- Certification program metrics review (improve completion rates, revocation rates)
- New system additions via standard intake process (not custom projects)
- ISC upgrade management (quarterly ISC releases tested in sandbox before production)

---

## Risk Factors and Mitigation

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| HR data quality worse than expected | High | High | Budget 3–4 weeks data remediation in Phase 1 |
| Stakeholder availability for design workshops | High | Medium | Schedule workshops 6 weeks in advance; executive sponsor enforcement |
| AD permission approval delays | Medium | Medium | Submit change requests in week 1, not week 8 |
| Scope creep (adding systems mid-phase) | High | High | Formal scope change process; additions go in next phase |
| Role design paralysis | Medium | High | Time-box design decisions; integration team owns final call with stakeholder input |
| First certification campaign failure | Low | High | Pilot with one department first; address issues before full rollout |
| ISC upgrade breaking custom rules | Low | Medium | Test every upgrade against custom rules in sandbox first |

---

## Key Takeaways

- Phased delivery produces governance value incrementally — connect highest-risk systems first, not easiest systems first
- Phase 1 is foundation: if Workday correlation is poor or lifecycle states are wrong, everything built on top will be wrong
- HR data quality remediation and AD permission delays are the two most common project blockers — start both on day 1
- First certification campaign is the business-facing proof of value — design it to succeed rather than trying to be comprehensive
- Role model governance (a board that reviews new role requests) prevents proliferation from the start
- Budget 20% more time than estimates for Phase 4 integrations — the "easy" systems always have surprises

> 🔗 **CONNECTS TO:** An 18-month implementation plan covers the mechanics of delivery. The maturity model covers the outcomes — how to evaluate where an ISC implementation stands and what separates a basic deployment from a strategically valuable one.
