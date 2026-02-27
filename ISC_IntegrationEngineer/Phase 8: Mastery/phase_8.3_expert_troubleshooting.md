# Advanced Problem-Solving — The Hard 20%

Phase 6.5 covered systematic troubleshooting methodology for common integration problems. Most ISC issues — authentication failures, correlation mismatches, provisioning errors — follow the patterns in that document and resolve within hours.

Then there's the other kind. The problem that doesn't respond to the standard diagnostic process. The aggregation that produces different correlation results on different days for no apparent reason. The provisioning that fails for exactly one specific identity out of 12,000. The performance degradation that started after an ISC upgrade but doesn't match any known breaking change.

These cases — the hard 20% that consume 80% of investigation time — require different approaches: deeper logging, data analysis, architectural investigation, and sometimes vendor involvement. This document covers three of these cases in detail and gives you the tools to approach novel hard problems.

---

## Case Study 1: Intermittent Correlation Failures Affecting 1,000+ Identities

**The reported problem**: Meridian Capital's ISC admin team notices that the weekly Friday full aggregation consistently produces ~200 more uncorrelated accounts than the Thursday delta aggregation. The correlation rate fluctuates — 99.6% on Wednesday, 99.4% on Thursday, 99.2% on Friday — and then recovers to 99.5% by Monday. No new accounts have been added. The pattern has repeated for 6 weeks.

**What the standard investigation found (and didn't answer)**:
- Correlation rules haven't changed
- No accounts were added or removed in AD during the fluctuation window
- ISC upgrade notes for the relevant release mention no correlation changes
- The uncorrelated accounts in Friday's export are different each week — not the same accounts consistently failing

**The deeper investigation**:

**Observation 1**: The Friday full aggregation runs from 2–6 AM. The uncorrelated accounts are all in one specific AD OU: `OU=Employees,DC=corp,DC=globaltech,DC=com`. Other OUs don't fluctuate.

**Observation 2**: Looking at the accounts that are uncorrelated on Friday — their `employeeID` attribute is populated in AD but shows as null in ISC's aggregation log for that specific run. The next aggregation (delta on Monday) shows those same accounts with `employeeID` populated.

**Observation 3**: The `employeeID` attribute in AD is populated by an HR sync process that runs Thursday nights at 11 PM. It takes approximately 3 hours to complete for 9,600 accounts.

**Root cause**: The ISC full aggregation starts at 2 AM on Friday. The HR sync process doesn't complete until 2–3 AM on some weeks (when it runs longer than expected). ISC aggregates some AD accounts before the HR sync has populated their `employeeID` — those accounts have null `employeeID` during aggregation, fail the primary correlation rule, and fall through to the email-based fallback rule. For some accounts, the email fallback also fails (name-change accounts, contractor email formats).

**Fix**: Move the Friday full aggregation start time to 5 AM (after the HR sync is guaranteed to complete). Alternatively, implement a dependency check in the aggregation schedule — don't start ISC aggregation until the HR sync reports completion. The HR sync writes a timestamp to a shared location when done; ISC's custom pre-aggregation rule checks that timestamp before proceeding.

**Lesson**: Intermittent correlation failures often have external timing dependencies. When failures are non-deterministic and involve the same attributes, investigate what else touches those attributes around the same time window.

> 🎯 **KEY POINT:** Timestamp correlation is one of the most powerful diagnostic techniques for intermittent problems. Take the timestamps from multiple failed instances and compare them for patterns — time of day, day of week, proximity to other scheduled processes. Real intermittent bugs almost always have a temporal pattern.

---

## Case Study 2: Single Identity Provisioning Failure in Production

**The reported problem**: James Rivera, senior network engineer, was granted AWS Production Admin access via access request and manager approval. Every other access request in the same campaign provisioned successfully. James's AWS provisioning has been in "Pending" state for 3 days. IT help desk has escalated.

**Why this is hard**: 11,999 other identities provision to AWS correctly. One fails. The standard diagnostic (check provisioning activity, check logs) shows the same generic error as a permissions failure — but the ISC service account has AWS admin access that works for everyone else.

**The investigation**:

**Step 1 — Reproduce with test identity**: Create a test identity with identical attributes to James Rivera. Request the same AWS access. It provisions correctly. The problem is specific to James's identity, not the general provisioning configuration.

**Step 2 — Compare test identity to James's identity**: What's different?
- Both have the same department, lifecycle state, role assignments
- James has 47 previous access requests in his history; test identity has 0
- James has 3 manually correlated accounts (from a previous data remediation); test identity has none
- James's ISC identity was created in 2019 and has attributes from multiple previous source configurations

**Step 3 — Check James's identity for data anomalies**: ISC identity details → Accounts tab → review all correlated accounts. James has 4 AD accounts correlated — one from the main domain, one from a legacy domain (from an acquisition), one from a contractor period when he was a contractor, and the current active one. The legacy contractor account has a null `department` attribute.

**Step 4 — Trace the provisioning plan for James**: The AWS provisioning policy uses `department` to derive the AWS account assignment (James goes to the `GlobalTech-Infrastructure` AWS account based on his department). The department derivation rule iterates through all correlated accounts and returns the first non-null department value. For James, it finds the legacy contractor account first (alphabetical ordering of account names) — and that account has null department. The rule returns null. The provisioning policy's required `awsAccountId` field resolves to null. Plan compilation fails.

**Fix**: Update the department derivation rule to skip accounts where `accountDisabled = true` or accounts from specific legacy sources. The rule should only use the current, active account as the department source.

**Lesson**: Long-tenured identities accumulate complexity — multiple correlated accounts, historical data, legacy configurations. When a single identity fails in a way thousands of others don't, the distinguishing factor is almost always in the identity's account history or accumulated state.

> ⚠️ **COMMON MISTAKE:** Focusing on the latest error rather than the identity's history. ISC identity objects accumulate data over time. A rule that worked correctly when the identity was simple may fail years later when that identity has 4 correlated accounts, 3 former departments, and attributes from a deprecated source. Inspect the full account history of problematic identities, not just their current state.

---

## Case Study 3: Performance Degradation After ISC Upgrade

**The reported problem**: After ISC's quarterly upgrade (released and auto-applied on a Saturday), GlobalTech's Monday morning aggregation windows are running 40% longer than before. Friday evening (pre-upgrade): Workday full aggregation completed in 37 minutes. Monday morning (post-upgrade): same aggregation took 52 minutes. The ISC upgrade release notes don't mention aggregation performance changes.

**Investigation**:

**Step 1 — Isolate the phase**: Run the aggregation and monitor the ISC admin console for phase-by-phase progress timing. Retrieval phase: 18 minutes (same as before). Processing phase: 34 minutes (previously 19 minutes). The slowdown is in processing, not retrieval.

**Step 2 — Check for changed transforms or rules**: Were any transforms or BeanShell rules updated in the upgrade? Review the ISC upgrade release notes (full release notes, not the summary). Find: "Updated identity processing framework to support new AI features. Identity attribute evaluation now includes additional validation passes."

**Step 3 — Profile which rule is slow**: Enable connector debug logging (set to TRACE level during a test aggregation). Review the `ccg.log` for timing information around individual rule executions. One rule shows dramatically increased execution time: the manager reference resolution rule.

**Step 4 — Analyze the manager reference rule**: The rule looks up the manager's ISC identity by employee ID and returns their email address. Before the upgrade, this lookup took an average of 12ms per call. After the upgrade, the same lookup takes 45ms. The new "additional validation passes" introduced by the AI framework runs on every identity attribute lookup — including the lookups inside rules.

**Fix options**:
1. Cache the manager lookup results in rule execution context rather than calling the lookup for each identity
2. Pre-compute a manager ID → email mapping as a flat table during a separate step, then use a Lookup transform instead of the BeanShell rule
3. Contact SailPoint support and report the performance regression — they may have a configuration workaround or a hotfix

**Implementation**: Replace the rule with a pre-computed lookup table. Before each full aggregation, a scheduled task exports a manager ID → email CSV from Workday. A Lookup transform uses this CSV. Processing phase drops from 34 minutes to 22 minutes (better than pre-upgrade, because the old rule approach was inefficient regardless of the upgrade).

**Lesson**: ISC upgrades sometimes introduce performance regressions in edge cases that passed regression testing. Always run a performance baseline test in sandbox before applying major upgrades to production. If degradation appears, profile phase by phase to isolate the affected component.

---

## Advanced Diagnostic Tools

**VA Debug Logging (TRACE level)**: Change the log level in `/home/sailpoint/config/log4j.properties` to `TRACE` for specific components. This generates very verbose output — use only during active investigation, not continuously. TRACE logs reveal timing information for individual operations that standard logs aggregate.

**ISC Admin API for Identity State**: `GET /v3/identities/{id}` returns the full identity object including all attribute values, all correlated accounts, lifecycle state, and role assignments. Useful for programmatic comparison of many identities to find the one that's different.

**Aggregation Statistics Export**: ISC can export aggregation results in CSV format including per-account timing data. For performance debugging, this reveals which accounts take the longest to process.

**SailPoint Community and Support**: When the problem genuinely doesn't fit any known pattern, SailPoint's support and community are valuable. The SailPoint Compass community (community.sailpoint.com) has ISC practitioners who've seen most failure modes. SailPoint's enterprise support offers a case submission path for bugs and regressions. When submitting a support case, include: ISC version, VA version, specific error messages, log excerpts, and a minimal reproduction case.

---

## The Escalation Decision

Know when to escalate to SailPoint support:
- You've followed the diagnostic process fully and cannot identify the root cause
- The behavior appears to be a platform regression (worked before upgrade, not working now)
- Data integrity appears corrupted in ISC (duplicate identities, missing accounts that should exist)
- The issue is intermittent and not reproducible in sandbox despite matching configuration

Don't escalate prematurely — SailPoint support will ask for the same log evidence and diagnostic steps you should have already done. Document your investigation before opening a case.

---

## Key Takeaways

- Hard problems almost always have timing dependencies, accumulated state complexity, or platform interactions — look for what's unique, not what's common
- Intermittent failures have temporal patterns: compare timestamps across multiple occurrences
- Single-identity failures point to accumulated state on that identity: check all correlated accounts, historical attributes, and rule behavior against the specific identity's data
- ISC upgrade performance regressions: profile phase by phase, enable TRACE logging for the slow phase, check upgrade release notes for framework changes
- Advanced tools: TRACE logging, Admin API identity inspection, aggregation statistics export, SailPoint Community
- Escalate to SailPoint support when platform regression is suspected and you have reproducible evidence

> 🔗 **CONNECTS TO:** Advanced troubleshooting resolves hard problems in individual systems. Real case studies show how the full range of skills — from requirements gathering through performance optimization — applies in complete implementation stories.
