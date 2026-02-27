# How to Test Integrations Properly

"It worked in my sandbox" is not a testing strategy. Neither is deploying to production and waiting for users to report problems. Integration testing is a structured process that verifies every component of an integration works correctly under realistic conditions — before it affects real access for real people.

Most integration engineers undertest. The pressure to ship is real, sandbox environments are often stale, and manually testing every scenario feels tedious. But the cost of finding a bug in production — a lifecycle state that doesn't trigger, a correlation rule that miscorrelates 50 accounts, a provisioning policy that creates accounts with wrong attributes — is an order of magnitude higher than finding it in testing.

This document gives you the testing framework. Use it every time.

---

## The Three Testing Layers

Integration testing happens at three levels, each serving a different purpose:

### Layer 1: Unit Testing — Individual Components

Unit testing verifies that individual pieces work in isolation: transforms produce the expected output, correlation rules return the expected matches, provisioning policy attribute mappings produce the correct values.

**Transform unit tests**: For each transform or transform chain in your integration, test:
- Happy path: standard input → expected output
- Null input: what happens if the source attribute is null?
- Boundary cases: empty string, maximum length value, special characters

**How to run transform tests in ISC**: Admin → Transforms → [Transform Name] → Test. Enter a test value, run the transform, verify the output. For chained transforms, test each step individually before testing the chain.

Example transform test matrix for a username generation transform (`Concat(Lower(firstname), ".", Lower(lastname))`):

| Input firstname | Input lastname | Expected output | Actual output | Pass? |
|---|---|---|---|---|
| `Sarah` | `Chen` | `sarah.chen` | `sarah.chen` | ✅ |
| `JAMES` | `RIVERA` | `james.rivera` | `james.rivera` | ✅ |
| ` Sarah ` | `Chen ` | `sarah.chen` | `sarah.chen` | ✅ (trim applied) |
| `null` | `Chen` | rule-defined fallback | Check | ⚠️ |
| `Sarah` | `O'Brien` | `sarah.o'brien` | ? | ⚠️ Test special chars |

**Correlation rule unit tests**: Use ISC's correlation preview mode against a set of test accounts where you know the expected outcome. Build a test account list with known expected correlations:

| Test Account | Expected Identity | Expected Result |
|---|---|---|
| AD: `schen` (employeeId: 48291) | Sarah Chen (employeeId: 48291) | Correlated |
| AD: `svc-backup-sql` (service account) | None | Excluded / Uncorrelated |
| AD: `jsmith` (no employeeId) | James Smith (email: `j.smith@…`) | Fallback rule |

Verify the preview matches expectations before live correlation.

---

### Layer 2: Integration Testing — The Complete Flow

Integration testing verifies that components work together: source aggregation → correlation → lifecycle evaluation → provisioning → target account creation.

**The standard integration test sequence**:

**Test 1 — New identity, full provisioning**:
Create a test identity in your authoritative source (Workday, AD, or wherever identities originate). Trigger aggregation. Verify:
- ISC created the identity with correct attributes
- Correct lifecycle state was assigned
- Birthright roles were assigned
- Provisioning executed for all connected targets
- Verify each target system directly: accounts exist with correct attributes and entitlements

**Test 2 — Attribute change propagation**:
Update an attribute on the test identity in the source (change department, job title, manager). Run aggregation. Verify:
- ISC identity attribute updated
- Any role or access profile assignments driven by that attribute updated
- Target systems received the attribute update (if configured for sync)

**Test 3 — Role change (requested access)**:
Submit an access request for the test identity. Complete the approval workflow. Verify:
- Approval workflow followed the correct approval chain
- Provisioning executed on approval
- Target system received the entitlement
- The access appears on the identity's access profile in ISC

**Test 4 — Leave of absence**:
Update the test identity's leave status in the source to simulate LOA. Run aggregation. Verify:
- Lifecycle state changed to Leave of Absence
- All target accounts were disabled (not deleted)
- Role assignments are preserved (not revoked)

**Test 5 — Return from LOA**:
Update leave status back to active. Run aggregation. Verify:
- Lifecycle state changed back to Active
- All target accounts re-enabled
- No reprovisioning was needed — access was preserved

**Test 6 — Termination**:
Update the test identity's termination date in the source to today. Run aggregation. Verify:
- Lifecycle state changed to Terminated
- All target accounts disabled within expected timeframe
- Governance alerts for the termination appear correctly
- The identity's access profiles show as revoked in ISC

> ⚠️ **COMMON MISTAKE:** Testing only provisioning, not deprovisioning. The termination lifecycle path is the most governance-critical flow and the one most often skipped in testing. If termination doesn't work, you have a security gap in production. Test it explicitly.

---

### Layer 3: User Acceptance Testing — Business Requirement Verification

UAT verifies that the integration meets the business requirements stakeholders agreed to in the design document — not just that it technically works, but that it works in the way the business needs.

**Who runs UAT**: Business stakeholders (HR Director, application owners, CISO) — not the integration engineer. The engineer sets up the test environment; the stakeholders verify their requirements are met.

**UAT scenarios for a Workday → ISC → target integration**:

| Business Requirement | UAT Scenario | Acceptance Criteria |
|---|---|---|
| New hires have access on day 1 | Create test employee starting tomorrow; verify by end of day today | All target accounts provisioned before 8 AM on start date |
| Terminations processed within 15 min for privileged users | Trigger termination for a test IT admin account; measure time to account disable | All accounts disabled within 15 minutes |
| LOA preserves access for return | Test LOA → return flow | Employee retains all access, can log in on return day |
| Department change updates access | Move test employee between departments | Old department-specific access revoked; new access provisioned |
| Certifications route to correct managers | Run a test certification campaign | All items route to the correct manager based on org hierarchy |

Each acceptance criterion is measurable. "Access on day 1" isn't acceptance criteria — "all target accounts provisioned before 8 AM on start date" is.

---

## Testing With Real Data Safely

Testing with a small, sanitized subset of real data catches problems that synthetic test data misses — encoding issues, unusual special characters, edge cases that only appear in actual records.

**Safe real data testing practices**:

**Use a dedicated ISC sandbox tenant**: SailPoint provides sandbox tenants for development and testing. A sandbox tenant is a separate ISC instance — actions there don't affect production identities or accounts. Configure your integration in sandbox first using a copy of the production configuration.

**Filter aggregation to a test population**: Configure the source in sandbox to aggregate only accounts in a designated test OU or matching a test attribute value (`employeeType = "TEST"`). This gives you real attribute formats and encoding without processing all 9,600 production accounts.

**Use a staging environment in target systems**: For Salesforce, ServiceNow, and most enterprise SaaS platforms, request a sandbox/staging instance. Configure the ISC sandbox to provision into the staging Salesforce environment. This tests end-to-end provisioning without touching production Salesforce users.

**Masking sensitive data**: If you must test in an environment that doesn't have a separate sandbox, mask personally identifiable information (names, emails) in the test dataset before using it. ISC admin tooling supports data export with field masking.

> 💡 **REMEMBER:** Never test provisioning operations against production target systems unless you have explicit authorization and a rollback plan. A test provisioning that creates a Salesforce account in production is an account that needs to be cleaned up manually — and a potential audit finding.

---

## Edge Case Testing Checklist

Standard test scenarios verify the happy path. Edge cases catch the failures that appear in production after go-live. Test these explicitly:

**Identity edge cases**:
- [ ] Employee with no manager (new VP, organization head)
- [ ] Employee with dual reporting (matrix organization)
- [ ] Contractor with no employee ID
- [ ] Employee with special characters in name (O'Brien, García, Müller)
- [ ] Employee with a hyphenated last name (Smith-Jones)
- [ ] Rehire — person who previously had accounts
- [ ] Name change — person whose name differs between systems

**Account edge cases**:
- [ ] Service account that matches human account naming pattern
- [ ] Shared mailbox / functional account
- [ ] Account in an unexpected OU
- [ ] Account with expired `accountExpires` date
- [ ] Account with all optional attributes null

**Provisioning edge cases**:
- [ ] Provisioning when the target system is in maintenance mode
- [ ] Provisioning for an identity whose source attributes are incomplete
- [ ] Provisioning when the identity already has a correlated account in the target
- [ ] Provisioning failure halfway through a multi-system role (some targets succeed, some fail)

---

## Pre-Go-Live Checklist

Before switching a new integration to production:

**Source integration**:
- [ ] Correlation rate ≥ 99% for human accounts
- [ ] All uncorrelated accounts categorized and documented
- [ ] Service account exclusions configured and verified
- [ ] Aggregation schedule set and tested
- [ ] Aggregation error monitoring configured
- [ ] Full aggregation duration within acceptable window

**Target integration**:
- [ ] All six test scenarios completed successfully (new, attribute change, requested access, LOA, return from LOA, termination)
- [ ] Provisioning error handling configured (review queue, retry policy)
- [ ] Deprovisioning tested — accounts disabled, not deleted
- [ ] Conflict resolution tested (existing account linked, not duplicated)
- [ ] Required field mapping complete and verified

**Governance**:
- [ ] Access profiles owned and described in plain language
- [ ] Approval workflows configured and tested
- [ ] Lifecycle state rules tested for all expected transitions
- [ ] First certification campaign scoped and scheduled

**Operations**:
- [ ] VA health monitoring alerts configured
- [ ] Aggregation failure alerts configured
- [ ] Provisioning review queue assigned to a team
- [ ] Runbook documented (what to do when specific things fail)
- [ ] Credentials documented (where service accounts are, who owns them)

---

## Key Takeaways

- Three testing layers: unit (individual components), integration (complete flows), UAT (business requirement verification)
- Test deprovisioning as rigorously as provisioning — it's more governance-critical and more often skipped
- Use dedicated sandbox tenants and staging environments for target systems — never test provisioning against production
- Build an explicit edge case test matrix: every unusual identity type, account type, and provisioning scenario
- UAT uses measurable acceptance criteria ("accounts provisioned before 8 AM") not subjective ones ("access on day 1")
- Complete the pre-go-live checklist for every integration — it's the difference between a smooth launch and a Monday morning incident

> 🔗 **CONNECTS TO:** Phase 6 has given you the practical skills to build, troubleshoot, and test integrations. Phase 7 moves into the advanced scenarios that appear in complex enterprise environments — multiple systems, complex organizational structures, and the governance requirements that go beyond individual integrations.
