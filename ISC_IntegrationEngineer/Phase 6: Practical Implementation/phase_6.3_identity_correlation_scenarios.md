# Real Correlation Challenges and How to Solve Them

Correlation rules work perfectly in theory and messily in practice. You design a clean rule — `Account.email = Identity.email` — run it against 9,000 accounts, and get 94% correlation. The 6% that didn't match contain two former executives, forty contractors with non-standard emails, three people who changed their names, fifteen test accounts, and an unknown number of actual errors.

This document is a case-by-case guide to real correlation problems: what they look like, why they happen, and exactly how to fix them. These scenarios are drawn from patterns that appear repeatedly in enterprise ISC implementations. You will encounter most of them.

---

## Scenario 1: Clean Match — Email Works, Straightforward Case

**Situation**: You're adding Salesforce as a managed source to an ISC tenant that already has Workday-sourced identities. 215 of 220 Salesforce accounts correlate cleanly on email. Five do not.

**Investigation**: Export the five uncorrelated accounts. Check their Salesforce email addresses against ISC identity emails:

| Salesforce Email | ISC Email | Status |
|---|---|---|
| `j.smith@globaltech.com` | Not found | → |
| `jennifer.s@globaltech.com` | `jennifer.smith@globaltech.com` | Name abbreviated in Salesforce |
| `mike.jones@globaltech.com` | Not found | Typo in Salesforce? |
| `t.rodriguez@consultant.globaltech.com` | `t.rodriguez@globaltech.com` | Contractor subdomain |
| `admin@globaltech.com` | Not found | Functional mailbox, not a person |

**Fixes**:
- `j.smith` — investigate who this is; may be test account or former employee
- `jennifer.s` — data quality issue in Salesforce; fix the Salesforce email to match ISC (preferred) or add a secondary correlation rule for name-abbreviated formats
- `mike.jones` — check Salesforce vs AD; if typo, fix in Salesforce
- `t.rodriguez@consultant.globaltech.com` — contractor domain variation; add a BeanShell rule that strips the `consultant.` prefix before matching
- `admin@` — functional mailbox; this should not correlate to a person; add an exclusion filter for functional mailbox patterns

> 🎯 **KEY POINT:** A 98% correlation rate on initial aggregation is good. A 94% rate needs investigation before go-live. Every uncorrelated account is either a data quality problem to fix at source, an exclusion to configure, or a genuine governance exception to document. None of the above categories should be silently accepted.

---

## Scenario 2: Email Mismatch — Old Address Still in the System

**Situation**: A company migrated from `@acmecorp.com` email to `@globaltech.com` email two years ago after an acquisition. Workday was updated. Active Directory was updated. But ServiceNow — which is now being added as a managed source — still has 180 accounts with `@acmecorp.com` email addresses because nobody updated them when the acquisition completed.

**Why this happens**: SaaS applications don't automatically update user email addresses when you change them in your directory. Someone has to push the change to each system. This often gets deprioritized during acquisitions and never happens.

**Correlation result**: 180 ServiceNow accounts uncorrelated because ISC's identity email attributes all say `@globaltech.com`.

**Solutions** (in order of preference):

**Option 1 — Fix the data in ServiceNow (best)**:
Write a ServiceNow script or use the ServiceNow REST API to bulk-update email addresses from `@acmecorp.com` to `@globaltech.com`. After the update, standard email correlation works. This is the right fix because it corrects the root cause.

**Option 2 — Add a secondary correlation attribute**:
If `employeeId` is stored in ServiceNow (many service desk systems store an employee ID field), add a secondary correlation rule: `ServiceNow.employee_id = Identity.employeeId`. This doesn't fix the data, but it gets governance working while the data fix is planned.

**Option 3 — BeanShell rule to normalize email domain**:
```java
String email = account.getAttribute("email");
if (email == null) return null;
// Normalize old domain to new domain before matching
email = email.replace("@acmecorp.com", "@globaltech.com");
return Filter.eq("email", email);
```
This is a workaround, not a fix. Use it only as a temporary bridge while the data migration is scheduled. Document it clearly and set a date to remove the workaround after data is fixed.

> ⚠️ **COMMON MISTAKE:** Leaving domain-normalization BeanShell rules in production permanently because "it works." Workarounds accumulate. Two years later a new engineer inherits correlation rules full of undocumented transformations with no explanation of why they exist. Fix the data; remove the workaround.

---

## Scenario 3: Name Change — Person Got Married (or Divorced)

**Situation**: Sarah Thompson got married and changed her name to Sarah Chen. Workday was updated immediately. But GitHub, Confluence, and Jira still know her as `s.thompson`. After ISC aggregation, her Workday identity says `sarah.chen` but her GitHub account says `s.thompson` — the email correlation works (she kept her email address for now), but GitHub's `username` attribute doesn't match anything.

**This surfaces a design issue**: If your primary correlation attribute is something that changes (display name, generated username), name changes break correlation.

**Resolution in this case**: Email-based correlation still works because she kept her email. The GitHub account correlates correctly. No immediate action needed.

**The actual problem this reveals**: In organizations where the username IS the correlation attribute (for systems that don't store email), name changes cause permanent correlation failures. The fix is to ensure every target system stores a persistent, change-immune identifier — employee ID, a stable user GUID, or a generated persistent ID.

**Practical guidance for name change scenarios**:
1. Check whether your current correlation attribute changes on name change. Email often doesn't. Username often does.
2. If it does change: add a fallback correlation rule on a stable attribute
3. For the specific affected accounts: use manual correlation override to relink them after the name change
4. Long term: work with system owners to populate an `employeeId` field in target systems. This makes correlation durable across name changes.

> 💡 **REMEMBER:** `employeeId` is the most durable correlation attribute across all scenarios — name changes, email changes, username format changes. If a system can store an employee ID (even in a custom field), use it. Every target system your company uses should ideally store the corporate employee ID.

---

## Scenario 4: Duplicate Accounts — The Same Person in Two Sources

**Situation**: GlobalTech acquired a company. The acquired company's employees were added to the main Workday tenant. But some of them already had ISC identities from when they were using a shared services portal before the acquisition. Now ISC has two identity records for the same person: one from the old acquisition source, one from Workday.

**Result**: Each identity has some of the correct accounts correlated to it. One identity has the AD account; the other has the GitHub account. Neither identity has complete access visibility.

**Diagnosing duplicate identities**: ISC admin console → Identity Management → search by name. If two identities appear for "James Rivera," you have a duplicate.

**Resolution process**:

1. **Identify which identity is authoritative** — typically the Workday-sourced identity (it has the current, maintained data)

2. **Identify all accounts correlated to the duplicate** — export the account list from the duplicate identity

3. **Re-correlate accounts to the authoritative identity** — in ISC, manual correlation allows you to move an account from one identity to another: Accounts → select account → Edit Correlation → link to different identity

4. **Merge attribute data if needed** — if the old identity had attributes the new one doesn't, manually update the authoritative identity

5. **Delete or archive the duplicate identity** — after all accounts are re-correlated, the duplicate should have no accounts. Archive it (set lifecycle state to Terminated or delete if your policy allows)

6. **Investigate the root cause** — why were there two identities? The source that created the duplicate needs its creation logic or correlation rules reviewed to prevent recurrence.

> ⚠️ **COMMON MISTAKE:** Deleting the duplicate identity before re-correlating its accounts. Deleting an identity in ISC does not automatically re-correlate its accounts — those accounts become uncorrelated orphans. Always move the accounts first, then remove the identity.

---

## Scenario 5: No Matching Attribute — The System Has Nothing Useful

**Situation**: GlobalTech is onboarding a legacy inventory management system. It has 400 user accounts. The only attributes it exposes are: `username` (formatted as `JSMITH`, `MRIVERA`), `full_name` ("James Smith"), and `access_level` (an integer 1–5). No email. No employee ID. No persistent unique identifier of any kind.

**This is the hardest correlation scenario.** There is no clean attribute to match against ISC identity attributes.

**Available strategies**:

**Strategy 1 — Username pattern matching (if the pattern is consistent)**:
If `JSMITH` always maps to `john.smith@globaltech.com` via a deterministic rule (first initial + last name → email first component), write a BeanShell rule that derives the probable email from the username format and queries for it.

```java
String username = account.getAttribute("username");
if (username == null || username.length() < 2) return null;
// Pattern: JSMITH → first initial J, last name SMITH
String firstInitial = username.substring(0, 1).toLowerCase();
String lastName = username.substring(1).toLowerCase();
// Derive probable email format
String probableEmail = firstInitial + "." + lastName + "@globaltech.com";
return Filter.ignoreCase(Filter.eq("email", probableEmail));
```

**Risk**: This produces false positives when two people share the same initial + last name. Add validation: after correlation, spot-check 50 accounts to verify the matched identities are correct.

**Strategy 2 — Full name matching (use cautiously)**:
Match `full_name` against ISC `displayName`. Inherently unreliable for organizations with common names. Add multiple conditions: name match AND department match (if available) AND manager match.

**Strategy 3 — Bulk manual correlation with a spreadsheet**:
Export all 400 accounts. Share with the system owner who knows whose account is whose. They fill in a mapping: `JSMITH` = `j.smith@globaltech.com`. Import the mapping and use it to drive manual correlation. Tedious for 400 accounts, but correct.

**Strategy 4 — Populate a persistent ID field in the source system**:
Work with the system owner to add an `employeeId` field and populate it for all 400 accounts. Then use standard employeeId correlation. This is a project, not a quick fix — but it produces durable, maintainable correlation.

> 🎯 **KEY POINT:** The right answer for systems with no good correlation attribute is: fix the system first if possible (add an employee ID field), then correlate. Pattern-matching correlation rules are a last resort with accuracy risks. Document any pattern-matching rules with examples and accuracy confidence levels — the next engineer needs to know these rules are approximate.

---

## Choosing the Right Correlation Strategy

| Scenario | Strategy | Confidence | Maintenance |
|---|---|---|---|
| Unique ID match (employeeId, account ID) | Attribute rule | High | Low |
| Email match (clean data) | Attribute rule | High | Low |
| Email match (with domain/format variations) | BeanShell with normalization | Medium | Medium |
| Name change situations | Fallback rule chain + manual override | Medium | Medium |
| Duplicate identities | Manual re-correlation + deduplication | High | One-time |
| No shared attribute | Pattern matching or manual | Low–Medium | High |

---

## Key Takeaways

- Export and categorize every uncorrelated account — each one is a solvable problem, not a permanent gap
- Fix data at the source whenever possible; correlation rule workarounds accumulate and obscure root causes
- Employee ID is the most durable correlation attribute — advocate for populating it in every connected system
- Name changes are a signal to review whether correlation attributes are change-immune
- Duplicate identities must be re-correlated (not just deleted) to preserve account governance
- Pattern-matching correlation (derived from username format, full name) is approximate — always spot-check results

> 🔗 **CONNECTS TO:** Correlation problems surface during aggregation. Provisioning problems surface during account creation and modification. The next document covers the provisioning failures you'll encounter most often — and the diagnostic path for each one.
