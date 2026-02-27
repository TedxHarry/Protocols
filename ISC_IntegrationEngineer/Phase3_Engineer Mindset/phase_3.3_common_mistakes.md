# Common Mistakes Beginners Make (And How to Avoid Them)

Every mistake in this document has happened in a real ISC implementation. Some of them have happened multiple times at the same organization. None of them are obvious to someone encountering ISC for the first time — they're the class of mistakes that look totally reasonable in the moment and catastrophically wrong in hindsight.

Learning from these isn't about memorizing failure modes. It's about developing the instinct to ask the right questions before you're in the situation where not asking costs you a week of remediation work.

---

## Mistake 1: Underestimating Correlation Complexity

**What happens**: The integration engineer sets up email correlation for Active Directory. Tests it against 20 accounts in the sandbox. Works perfectly. Runs it in production against 8,400 accounts. Gets 94% correlation rate. Treats the remaining 6% as "acceptable background noise" and moves forward.

**What actually happened**: Six percent of 8,400 accounts is 504 uncorrelated accounts. Among those 504:
- 40 are service accounts (expected, should be excluded)
- 80 are contractor accounts that used `@contractor.globaltech.com` email format instead of `@globaltech.com`
- 22 are executives who have personal email addresses set as primary in AD for some reason
- 12 are former employees whose identities were archived but whose accounts weren't deprovisioned
- 350 are accounts from an acquisition three years ago that used a different naming convention

Of the 12 former employee accounts: two have admin-level group memberships on production systems. They've been active, uncorrelated, and ungoverned for over a year.

The audit finding this generates isn't a minor documentation gap — it's evidence of a governance control failure. Active admin credentials belonging to people who no longer work at the company, completely outside governance scope.

**Why it happened**: The engineer tested correlation with a clean, representative subset. Real production data has edge cases that test data never does — legacy accounts from acquisitions, naming exceptions for executives, contractor email format variations. Testing against a subset is necessary but not sufficient. You must test against the full dataset before going live.

**Prevention strategy**:
1. After designing correlation rules, run them in "report only" mode first — no changes, just output what correlates and what doesn't
2. Export every uncorrelated account and manually categorize them: service accounts (build exclusions), data quality issues (fix at source), legitimate exceptions (document individually), former employees (deprovision immediately)
3. Don't go live until the uncorrelated rate for human accounts is under 1%
4. For the first 30 days post-launch, review the uncorrelated account list weekly

> ⚠️ **COMMON MISTAKE:** Setting a correlation rule "good enough" threshold and moving on. There is no acceptable threshold for uncorrelated human accounts. Each one is a governance blind spot. Service accounts can be systematically excluded; human accounts must be correlated or explicitly investigated.

---

## Mistake 2: Over-Engineering Transforms

**What happens**: A new integration engineer is connecting an HR system that provides department as a code: `FIN-GL-001`. ISC needs a human-readable department name: `Finance - General Ledger`. The engineer writes a 150-line BeanShell rule that parses the code, handles exceptions, applies business logic, formats the output, and logs errors.

**What actually happened**: The built-in ISC Lookup transform handles this in three configuration lines. The engineer built a custom rule because they were unfamiliar with the transform library and defaulted to what they know: code.

Six months later, ISC releases a minor update. The update changes the BeanShell API slightly. The rule breaks silently — it returns null instead of a department name. Every new hire provisioned after the update gets an empty department attribute. Access assignment rules that depend on department stop firing. Thirty-seven new employees get provisioned with no role-based access and nobody notices for two weeks because the provisioning doesn't fail — it just runs without role assignments.

**Why it happened**: The engineer solved the right problem with a more complicated tool than necessary. Custom rules work — but they have maintenance costs, upgrade risk, and debugging complexity that built-in transforms don't.

**Prevention strategy**:
1. Before writing any rule, spend 30 minutes reading the ISC transform documentation (it's comprehensive and well-organized)
2. Try to solve the transformation with built-in transforms first. Only write a rule when transforms genuinely can't do it.
3. If you must write a rule, keep it under 30 lines. If it's longer, ask: am I doing too much in one rule? Can I pre-process the data in a simpler way?
4. Test rules against edge cases: null values, empty strings, unexpected formats. BeanShell NullPointerExceptions are the most common rule failure mode.

> 💡 **REMEMBER:** The next engineer who works on this integration has to understand your code. A 150-line BeanShell rule is a maintenance debt. A 3-step transform chain is self-documenting. Write for the person who maintains it, not the person who builds it.

---

## Mistake 3: Forgetting Service Accounts

**What happens**: Integration engineer designs a complete Workday-to-AD integration. Correlation rules written for human accounts: `Identity.email = Account.mail`. Runs the first full aggregation in production.

ISC reports: 7,820 correlated accounts. 580 uncorrelated accounts. The engineer checks the number: that's 6.9%, higher than expected. Investigates a few. Finds they look weird — usernames like `svc-backup-sql`, `svc-jenkins-aws`, `svc-monitor-prod`. Service accounts.

Now there are 580 service accounts in ISC's uncorrelated list. They generate governance alerts. The security team sees 580 "unowned accounts" and escalates. An incident is opened. It takes two days to explain that these are intentional service accounts, not a security breach.

**What actually happened**: Service accounts were never considered in the integration design. The correlation rules had no exclusion logic for them. The security alert workflow was not briefed on the expected false positive.

**Prevention strategy**:
1. Ask during discovery: "How many service accounts are in this system? What naming convention do they follow?"
2. Build service account exclusion rules before running the first aggregation. Common patterns: prefix `svc-`, prefix `app-`, suffix `-sa`, OU exclusion (service accounts in dedicated OU)
3. Create a "Service Account" identity in ISC with a designated IT owner. Correlate service accounts to this identity or to individual application owners
4. Brief the security team before running first aggregation: "You will see X uncorrelated accounts. Here's what they are. Here's the plan."

> 🎯 **KEY POINT:** Service accounts are a governance requirement, not a governance exception. Every service account should have a documented owner, a documented purpose, and a review schedule. ISC can govern service accounts through a separate governance track — don't just exclude them and ignore them. But DO exclude them from the human-account correlation rules.

---

## Mistake 4: No Deprovisioning Plan

**What happens**: Project kickoff. Timeline is 12 weeks. Phase 1: connect Workday, connect AD, set up provisioning. Phase 2: connect Salesforce, GitHub, Jira. Phase 3 (someday): "handle terminations." The phrase "phase 3" is doing a lot of work here — it's a euphemism for "we haven't designed this yet."

Go-live for Phase 1 and 2 happens in week 14 (slightly over schedule). Phase 3 is officially "out of scope for initial launch but on the roadmap."

18 months later: Phase 3 has never happened. The engineering team turned over. The original ISC architect left the company. Nobody owns "phase 3" anymore. Three former employees who left over the past 18 months still have active Salesforce and GitHub accounts. An external audit finds them.

**Why it happened**: Deprovisioning is harder to design than provisioning, it doesn't have visible day-one impact (nobody sees a user NOT get access when they leave), and it gets deprioritized when timelines compress. By the time the audit finding arrives, the team that built the system has moved on.

**Prevention strategy**:
1. Design deprovisioning before designing provisioning. If you can provision it, you need a plan to deprovision it. No exceptions.
2. In scope planning: deprovisioning is not "phase 3." It's part of the definition of done for every system you connect.
3. Test termination workflows before go-live. Create a test identity, run the termination lifecycle, verify all accounts are disabled.
4. Include deprovisioning coverage in the go-live criteria: "ISC governs [X] systems including termination on all of them."

> ⚠️ **COMMON MISTAKE:** Treating provisioning and deprovisioning as separate projects. They're two sides of the same lifecycle. Every system you connect without deprovisioning capability is accumulating future audit risk from day one.

---

## Mistake 5: Assuming All Data Is Clean

**What happens**: Engineer designs correlation rule: `Identity.employeeId = Account.employeeId`. Clean, simple, reliable. Runs aggregation. 847 uncorrelated accounts. Investigates. Finds:

- 22 accounts have `employeeId = 0` (a legacy data migration set null values to 0 instead of null)
- 35 accounts have `employeeId = NULL` (contractors added without employee IDs)
- 8 accounts have duplicated employee IDs (a data entry error created two Workday records with the same ID, one of which is real and one is a test record someone forgot to delete)
- 15 accounts have employee IDs with extra whitespace: `GT-48291 ` (trailing space breaks exact match)
- 5 accounts have mixed-case IDs: `gt-48291` instead of `GT-48291`

None of these are the correlation rule's fault. All of them are data quality problems that exist in the source systems. All of them require individual investigation and remediation before they'll correlate correctly.

**Prevention strategy**:
1. Before designing correlation, run data quality checks on your source systems. Export the correlation attribute from both sides and look at the distribution: duplicates, nulls, format inconsistencies.
2. Build your correlation rules to handle known edge cases: trim whitespace, normalize case, handle null and zero values.
3. Work with the data owners (HR team, AD team) to fix data quality issues at the source — correlation rule workarounds treat symptoms, not causes.
4. Plan for 2-3 weeks of data remediation after the first aggregation. Real data is never as clean as test data.

---

## Mistake 6: Skipping Sandbox Testing

**What happens**: Engineer has built a correlation rule update to fix the edge cases from the last aggregation. Tests it mentally. Confident it's right. Deploys directly to production ISC tenant because the sandbox hasn't been refreshed in 3 months and doesn't reflect the current production state.

The rule has a subtle bug: it handles the NULL case incorrectly. Instead of excluding NULL employee ID accounts from correlation, it matches ALL NULL accounts to the first identity found. 847 accounts with NULL employee IDs all correlate to Sarah Chen (the first identity alphabetically). Sarah now has 848 linked accounts including 12 service accounts and 4 accounts belonging to other people.

Unraveling this takes 3 days of manual corrections and a full re-aggregation.

**Prevention strategy**:
1. Test correlation rule changes in a sandbox tenant with a representative data sample before production.
2. If the sandbox is stale, refresh it before testing. ISC has tools for tenant refreshes — use them.
3. For correlation rule changes specifically: always preview the impact (ISC has a correlation preview mode) before applying changes.
4. Authorization: correlation rule changes in production should require a change management ticket with review. They're not minor configuration tweaks.

> 🎯 **KEY POINT:** ISC sandbox tenants exist specifically to prevent production mistakes. Using them is not bureaucratic overhead — it's how you avoid 3-day remediation projects. Every rule change, workflow change, and connector configuration change should be tested in sandbox first. The sandbox is faster to fix than production.

---

## Key Takeaways

- Test correlation rules against the full production dataset, not a clean sample — edge cases only appear at scale
- Use built-in transforms before writing custom BeanShell rules — transform chains are more maintainable and less fragile than custom code
- Design service account handling before the first aggregation — they will appear and need a plan
- Deprovisioning is not phase 3 — it's part of the definition of done for every connected system
- Assume data is dirty until proven otherwise — plan for data quality remediation as a normal part of every integration project
- Test in sandbox first, every time — production ISC tenants handle real access for real people; mistakes there have real consequences

> 🔗 **CONNECTS TO:** You know the theory, the framework, and the failure modes. The next document puts it all together: a complete real integration scenario, from requirements to production, showing how everything applies in practice.
