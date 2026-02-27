# Building Your First Target Integration

A source integration gets data into ISC. A target integration is where ISC acts — creating accounts, assigning roles, disabling access, and enforcing the governance decisions that make ISC more than a reporting tool.

Building a target integration is more complex than building a source. It requires not just reading data, but knowing how to create an account correctly in a system you may not own, handling edge cases in provisioning, and designing the full account lifecycle from first creation to eventual deprovisioning. Mistakes in a source integration produce wrong data in ISC. Mistakes in a target integration affect real people's access in production systems.

This document builds a Salesforce target integration end to end — one of the most common enterprise target integrations and a representative example of the decisions you'll face with any SaaS target.

---

## Step 1: Requirements

The requirements for a target integration go deeper than for a source. You're not just reading data — you're designing how ISC creates, modifies, and removes access in a live system.

**Questions to answer before configuration**:

**Who should get a Salesforce account?**
At GlobalTech, Salesforce is the CRM. The Sales department (220 people), Sales Operations (15 people), and 8 executives who need pipeline visibility get accounts. Nobody else. This scope definition drives your role membership criteria.

**What Salesforce profiles and permission sets are in scope?**
Salesforce has a complex access model: every user needs exactly one Profile (determines object access and UI layout), and can have zero or more Permission Sets (additive permissions). ISC needs to govern both.

Profiles in scope: `Sales User`, `Sales Manager`, `Sales Operations`, `Exec Read-Only`. Each becomes an access profile in ISC.

Permission Sets in scope: `Salesforce Inbox`, `Einstein Analytics`, `Territory Management`. Some are add-ons available to specific user types.

**What attribute values does Salesforce need when creating an account?**
Salesforce requires: `Username` (must be unique across all Salesforce — not just your org), `Email`, `FirstName`, `LastName`, `Alias`, `TimeZoneSidKey`, `LocaleSidKey`, `EmailEncodingKey`, `LanguageLocaleKey`, and a Profile ID. Missing any required field causes provisioning failure.

**What's the deprovisioning behavior?**
Salesforce does not delete users — it freezes or deactivates them. Deactivated users can't log in but their data ownership is preserved. ISC should deactivate (not delete) on termination. At GlobalTech, Salesforce user data retention is 90 days post-termination before the Salesforce admin manually reassigns owned records.

**What's the timeline for access?**
Sales managers want new hire access ready on day 1. ISC provisioning via birthright role on lifecycle state = Active, with pre-hire pre-provisioning for new accounts expected first day.

> ⚠️ **COMMON MISTAKE:** Not confirming deprovisioning behavior with the application owner before configuring the provisioning policy. Configuring "Delete" on a Salesforce user deletes their record permanently — including any owned accounts, contacts, opportunities. This is almost certainly not what the business wants. Confirm whether Deactivate, Freeze, or a custom flow is required.

---

## Step 2: Connector Selection and Configuration

Salesforce has a pre-built ISC connector that operates as a direct connection (no VA). It uses the Salesforce REST API with OAuth 2.0 authentication.

**Authentication setup**:
1. In Salesforce: Setup → Apps → Connected Apps → New Connected App
2. Enable OAuth, set callback URL to ISC's callback endpoint, grant API scopes: `api`, `refresh_token`, `offline_access`
3. Note the Consumer Key and Consumer Secret
4. In ISC: configure the Salesforce source with Consumer Key, Consumer Secret, and a service account credential for the refresh token

**Test the connection**: ISC provides a Test Connection button. If it fails, common causes are OAuth scope misconfiguration in Salesforce, incorrect Consumer Key/Secret, or the connected app not yet activated (takes up to 10 minutes after creation in Salesforce).

**Selecting the right Salesforce connector version**: SailPoint maintains multiple Salesforce connector versions. Use the latest stable version. Older connector versions may not support all Salesforce API versions — check connector release notes for the minimum Salesforce API version supported.

---

## Step 3: Aggregation Configuration (Entitlements First)

For target systems, run entitlement aggregation before account aggregation. ISC needs to know what Profiles and Permission Sets exist in Salesforce before it can correctly record which profiles your accounts have.

**Entitlement aggregation**: Reads all Profiles and Permission Sets from Salesforce. After this runs, ISC's entitlement catalog includes `Sales User`, `Sales Manager`, and all the Permission Sets. Access profiles you configure can reference these by exact name.

**Account aggregation**: Reads existing Salesforce user accounts. For most fresh target integrations, Salesforce already has accounts that predate ISC — accounts that IT provisioned manually, that Salesforce admins created, or that came over from an acquisition. The first aggregation surfaces all existing accounts, and correlation links them to ISC identities.

**Correlation for existing Salesforce accounts**:
Salesforce accounts typically correlate by email — the Salesforce `Username` field (which looks like an email) or the `Email` field (the user's actual email). GlobalTech uses:
- `Email` field in Salesforce equals Identity `email` in ISC

For 220 Sales users already in Salesforce, this correlation should produce near-100% match rates if GlobalTech's email format is consistent. Investigate any that don't correlate — they may be test accounts, role-based mailboxes, or historical accounts for people who've already left.

---

## Step 4: Provisioning Policy Configuration

The provisioning policy specifies what ISC sets when creating or updating a Salesforce user.

**Required Salesforce field mappings**:

| Salesforce Field | Source | Value/Transform |
|---|---|---|
| `Username` | Transform | `Concat(Lower(email), ".sf")` — Salesforce requires globally unique usernames; appending `.sf` to email prevents conflicts |
| `Email` | Identity attribute | `email` |
| `FirstName` | Identity attribute | `firstname` |
| `LastName` | Identity attribute | `lastname` |
| `Alias` | Transform | `Substring(firstname, 0, 1) + Substring(lastname, 0, 7)` — max 8 chars |
| `TimeZoneSidKey` | Static | `"America/New_York"` (or derive from location attribute) |
| `LocaleSidKey` | Static | `"en_US"` |
| `EmailEncodingKey` | Static | `"UTF-8"` |
| `LanguageLocaleKey` | Static | `"en_US"` |
| `ProfileId` | Access profile entitlement | Set via entitlement assignment, not provisioning policy |

> 💡 **REMEMBER:** The Salesforce `Username` field must be globally unique across ALL Salesforce orgs worldwide — not just yours. Appending a company-specific suffix (`.globaltech` or `.sf`) to the email prevents future conflicts if users exist in other Salesforce orgs. Confirm the suffix convention with your Salesforce admin.

**Account conflict handling**: An existing Salesforce account matching the correlation attribute (email) → link and update, not create new. This is the right behavior for existing accounts being brought under governance.

**Plan operations**: Enable, Disable, Update (for attribute changes), and Modify Entitlements (for Profile and Permission Set changes). Do NOT enable Delete — use Disable only for terminations; deactivation is permanent data-safe offboarding.

---

## Step 5: Access Profile and Role Configuration

With the connector configured and entitlements aggregated, build the governance layer.

**Access profiles** (one per Salesforce access tier):
- `Salesforce Sales User` — entitlement: Profile = `Sales User`
- `Salesforce Sales Manager` — entitlement: Profile = `Sales Manager` + Permission Set = `Territory Management`
- `Salesforce Sales Operations` — entitlement: Profile = `Sales Operations` + Permission Set = `Einstein Analytics`
- `Salesforce Exec Read-Only` — entitlement: Profile = `Exec Read-Only`

**Business role**: `Sales Representative` role bundles:
- From ISC: Standard Employee Access IT Role (AD account)
- From Salesforce: `Salesforce Sales User` access profile
- From Slack: `Slack Standard` access profile
- From Jira: `Jira Software User` access profile

When a new Sales hire's lifecycle state activates, the `Sales Representative` role fires, and all four access profiles provision in parallel.

---

## Step 6: End-to-End Testing

Test the complete provisioning flow before go-live — not just the connector test.

**Test scenario 1 — New account provisioning**:
1. Create a test identity in ISC with Sales department attributes
2. Manually trigger the `Sales Representative` role assignment
3. Verify ISC generates a provisioning plan (review it in the Provisioning Preview)
4. Approve the plan execution in test mode
5. Verify account appears in Salesforce with correct Profile and attribute values
6. Verify Salesforce `Username` format is correct and unique
7. Verify the identity in ISC now shows the correlated Salesforce account

**Test scenario 2 — Attribute update**:
Change the test identity's `firstname` in ISC. Wait for propagation (or trigger manually). Verify Salesforce `FirstName` and `Alias` update correctly.

**Test scenario 3 — Deprovisioning**:
Change the test identity lifecycle state to Terminated. Verify Salesforce account is Deactivated (not deleted). Verify the account is marked as disabled in ISC's account view for this identity.

**Test scenario 4 — Existing account linking** (if Salesforce already had users):
Take a known existing Salesforce user. Verify ISC correlated them correctly during aggregation. Assign the matching access profile. Verify ISC updates the Profile assignment without creating a duplicate account.

**Common provisioning failures at each stage**:

| Failure | Root Cause | Fix |
|---|---|---|
| `INSUFFICIENT_ACCESS_ON_CROSS_REFERENCE_ENTITY` | Service account lacks Salesforce permission | Grant `Manage Users` permission in Salesforce |
| `FIELD_INTEGRITY_EXCEPTION: Username invalid` | Username not globally unique or wrong format | Check suffix convention; verify uniqueness |
| `REQUIRED_FIELD_MISSING: ProfileId` | Access profile missing Profile entitlement | Assign Profile entitlement to access profile |
| `INVALID_OR_NULL_FOR_RESTRICTED_PICKLIST` | Static field value not valid for Salesforce locale | Verify TimeZoneSidKey and LocaleSidKey are valid Salesforce values |
| Duplicate account created | Conflict resolution set to Create instead of Link | Change to Link to existing account |

---

## Key Takeaways

- Target integration requirements must cover: who gets access, what the access model looks like, what Salesforce needs when creating accounts, and how deprovisioning works — before configuration starts
- Run entitlement aggregation before account aggregation on any new target
- Every required field in the target system must be mapped in the provisioning policy — missing required fields cause provisioning failures
- Salesforce Username uniqueness is global — append a company suffix to email addresses
- Test all four scenarios: new account, attribute update, deprovisioning, and existing account linking
- Deprovisioning behavior must match the application's data model — Salesforce deactivates, it doesn't delete

> 🔗 **CONNECTS TO:** Both source and target integrations depend on correlation working correctly. The next document covers the specific correlation challenges that appear in practice — and how to solve them when your rules don't produce the results you expected.
