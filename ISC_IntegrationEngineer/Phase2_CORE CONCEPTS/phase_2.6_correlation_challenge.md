# The Correlation Challenge — Linking Accounts to Identities

ISC has aggregated 8,400 accounts from Active Directory. It has 8,200 identity records from Workday. Somewhere in those numbers, Sarah Chen's AD account (`s.chen`) needs to be linked to her Workday identity (`GT-48291`).

ISC won't do this by magic. It needs a rule — a **correlation rule** that says: "If this attribute on the account matches this attribute on the identity, they belong to the same person."

This sounds straightforward. In practice, correlation is one of the most technically interesting and operationally consequential problems in ISC integration. Getting it wrong creates orphaned accounts (accounts that belong to someone but ISC thinks they're anonymous), false merges (accounts from two different people linked to the same identity), and governance gaps that can persist for years without anyone noticing.

## The Core Problem: Mismatched Identifiers

Sarah exists in seven systems. Each system uses a different identifier:

| System | Sarah's Identifier |
|---|---|
| Workday (HR) | `GT-48291` (employee ID) |
| Active Directory | `s.chen` (samAccountName) |
| Salesforce | `sarah.chen@globaltech.com` (username) |
| GitHub | `schen-dev` (login handle) |
| AWS | `sarah_chen_dev` (IAM username) |
| Slack | `sarah.chen` (workspace handle) |
| Jira | `SChen` (username) |
| SAP Finance | `SCHEN001` (SAP user ID) |

None of these are the same. None of them directly match any single Workday attribute. ISC needs a strategy for each system that reliably answers: "This account, with this identifier — which identity does it belong to?"

## How Correlation Rules Work

A correlation rule is a matching logic definition: "For accounts from System X, find the identity where Identity.attributeA = Account.attributeB."

**Simple correlation** — one attribute, direct match:
- Rule: `Identity.email = Account.mail`
- Logic: Workday provides `sarah.chen@globaltech.com` as Sarah's identity email. AD account has `mail = sarah.chen@globaltech.com`. Match found → account correlated to Sarah's identity.

**Transformed correlation** — attribute match after transformation:
- Rule: `Identity.employeeId = Account.employeeId` (after stripping "GT-" prefix)
- Logic: Workday provides `GT-48291`. AD has `employeeID = GT-48291`. Direct match.
- Or: Workday provides `48291` (numeric only). AD has `GT-48291`. Correlation rule strips "GT-" from AD value before comparison.

**Concatenated correlation** — build an identifier from multiple attributes:
- Rule: `Account.samAccountName = LOWERCASE(Identity.firstName[0] + "." + Identity.lastName)`
- Logic: Sarah Chen → `s.chen`. Check if any AD account has `samAccountName = s.chen`. This finds Sarah's account.
- Problem: This breaks for common names. If there are two employees named Sarah Chen, they both generate `s.chen` → ambiguous match.

**Employee ID correlation** — most reliable when available:
- Rule: `Identity.employeeId = Account.employeeId`
- AD has an `employeeID` attribute. Workday provides employee ID. Direct match.
- This is the most reliable correlation method when both systems consistently populate employee ID.

> 🎯 **KEY POINT:** Email correlation is the most common fallback, but employee ID is the gold standard. Employee IDs are assigned by HR, unique, and persistent across name changes and system migrations. Email addresses change (marriages, legal name changes, domain migrations) and are sometimes shared or reused. If your systems support employee ID correlation, use it.

## The Seven Systems Problem

For GlobalTech's seven systems, here are the real correlation strategies the integration team uses:

**Workday → ISC**: Workday IS the identity source. No correlation needed — Workday accounts ARE the identities. Every Workday worker record creates an identity.

**Active Directory**: Correlation via `employeeID` attribute on AD accounts. GlobalTech's AD is configured to store Workday employee IDs in the `employeeID` field during provisioning. Reliable — 99.7% match rate.

**Salesforce**: Correlation via email. Salesforce usernames follow the format `firstname.lastname@globaltech.com`. ISC matches against identity email. Works well for employees; breaks for contractors who use personal email domains.

**GitHub**: No reliable employee ID. ISC uses email correlation (GitHub primary email). Problem: 8% of developers used personal GitHub accounts (pre-company account policy) and registered with personal emails. Custom correlation rule: check GitHub email first, then check GitHub profile name against `firstname.lastname` pattern.

**AWS**: Correlation via `Tags` on IAM user. GlobalTech's provisioning workflow tags every IAM user with `EmployeeID: GT-XXXXX`. ISC reads the tag value and matches against identity `employeeId`. Near-100% match rate for ISC-provisioned accounts. Pre-existing accounts created before ISC: correlated via `samAccountName` pattern matching against username.

**Slack**: Email correlation. Slack requires company email for workspace accounts. Reliable.

**Jira**: Username pattern correlation. Jira usernames follow `FirstnameLastname[0]` pattern (e.g., `SChen`). ISC uses: `Account.username = UPPERCASE(Identity.lastName[0..1])`. Ambiguous for common surnames — secondary fallback: email.

**SAP Finance**: Legacy system. SAP usernames follow `FIRSTLAST[XXX]` pattern (e.g., `SCHEN001`). ISC uses a combination: strip numeric suffix, match remaining against `lastname+firstname[0]` pattern. Lower reliability (~94%) — remaining 6% correlated manually by SAP admin team.

> ⚠️ **COMMON MISTAKE:** Designing correlation rules against clean test data without testing against the full production dataset. When you test with 50 accounts, the rule works perfectly. When you run against 8,400 accounts including contractors, service accounts, shared mailboxes, executive accounts with special formatting, and legacy accounts from an acquisition three years ago, you find 200 edge cases you didn't anticipate.

## What Breaks Correlation

Real-world correlation is messy. Here are the failure patterns you'll encounter:

**Name changes**: Jennifer Smith gets married, becomes Jennifer Rodriguez. Her Workday identity updates to `jennifer.rodriguez@globaltech.com`. Her AD account still has `mail = jennifer.smith@globaltech.com` (AD update is pending). Email correlation breaks. ISC tries to correlate the AD account, doesn't find a matching email, marks the account as uncorrelated. Solution: always have a secondary correlation attribute (employee ID) as fallback.

**Duplicate accounts**: Two employees in an acquired company both have the username `jsmith`. The acquiree's user directory didn't enforce uniqueness across subsidiary boundaries. Correlation attempts to link both accounts to the same identity. This creates incorrect access pictures for both people. Solution: import employee IDs from the acquired company's HR system as the correlation anchor.

**System mergers**: GlobalTech acquires Precision Dynamics. Precision runs their own Salesforce org. Merge creates duplicate Salesforce accounts for employees who had both. Correlation needs to handle the "one identity, multiple accounts on the same system" scenario.

**Contractors and consultants**: TechStaff consulting firm provides contractors who use `@techstaff.com` email in systems. ISC has identities for them from a contractor management system with contractor IDs, not employee IDs. Email correlation picks up the company-side email from their corporate accounts but misses their contractor accounts. Custom correlation rule needed: match contractor ID against a custom attribute field.

**Service accounts**: The 40 service accounts in AD don't match any human identity by design. ISC's correlation attempts fail for all of them. They appear as 40 uncorrelated accounts. Solution: configure a correlation exclusion — accounts matching pattern `svc-*` are excluded from standard correlation and processed separately.

> 💡 **REMEMBER:** Uncorrelated accounts are not just a data quality issue — they're a security risk. An active account that ISC can't associate with an identity is effectively ungoverned. It won't be included in certifications. Its entitlements won't be evaluated against person-level SoD policies. It could belong to a former employee whose identity was properly deprovisioned but whose account was missed. Always have a process for investigating and resolving uncorrelated accounts.

## Correlation Confidence and Manual Override

ISC supports correlation confidence levels — an account can be correlated automatically with high confidence (strong attribute match), suggested with medium confidence (partial match requiring human review), or flagged as uncorrelated (no match found).

The identity admin queue handles medium-confidence matches: ISC presents a candidate match and a human decides whether to accept it. For GlobalTech's SAP system with its lower correlation reliability, approximately 6% of SAP accounts require human review during initial onboarding. After the initial batch is processed, new accounts tend to follow the pattern and correlate automatically.

Manual correlation override is also available: an admin can manually link a specific account to a specific identity. This creates a permanent manual correlation that persists even if the attribute-based rule wouldn't match. Useful for service accounts assigned to an "owner" identity, or executive accounts with non-standard naming conventions.

## Key Takeaways

- Correlation is the process of linking aggregated accounts to identity records — it's the foundation for meaningful governance
- Different systems require different correlation strategies; there's no universal approach
- Employee ID correlation is the gold standard when available; email correlation is a common fallback with known failure modes
- Test correlation rules against the full production dataset, including edge cases: contractors, service accounts, name changes, legacy accounts
- Uncorrelated accounts are governance gaps — active accounts with no known owner that can't be certified, policy-evaluated, or properly deprovisioned

> 🔗 **CONNECTS TO:** Aggregation reads data in, correlation links it to identities. The next document covers provisioning — the opposite direction: how ISC sends data OUT to systems to create, update, and disable accounts.
