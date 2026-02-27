# Building Your First Source Integration

You've learned the theory. This document is where you actually build something.

A source integration is the foundation of every ISC deployment. It's the connection that brings identity data — employee records, account attributes, organizational structure — into ISC where governance decisions can be made. Every access certification, every role assignment, every provisioning decision downstream depends on source data being accurate, current, and well-correlated.

This document walks through building an Active Directory source integration from end to end. AD is the most common source engineers encounter — it exists in over 95% of enterprise environments — and it surfaces almost every decision point you'll face with any source. Master this, and the pattern applies to Workday, SAP, LDAP, or any other source you connect.

---

## Step 1: Requirements Gathering

Before touching ISC, answer these questions. Skipping this step turns a 4-hour configuration task into a 3-week rework project.

**What AD domains and forests are in scope?**
GlobalTech has three AD domains: `corp.globaltech.com` (headquarters), `mfg.globaltech.com` (manufacturing division), and `eu.globaltech.com` (European subsidiary). Each is a separate ISC source — three connectors, three VA-connected aggregations. If you design for one and discover the others later, you rebuild.

**Which OUs contain accounts you want to govern?**
GlobalTech's `corp` domain has OUs for Employees, Contractors, Service Accounts, and Disabled Users. Only Employees and Contractors are in scope — Service Accounts and Disabled Users have separate governance processes. Document the OU paths before configuring the search base: `OU=Employees,DC=corp,DC=globaltech,DC=com`.

**What attributes does ISC need from AD?**
Build the attribute list explicitly. Common required attributes for governance:

| AD Attribute | ISC Purpose | Notes |
|---|---|---|
| `sAMAccountName` | Unique account identifier | Used in correlation |
| `userPrincipalName` | Email-format identifier | May differ from actual email |
| `mail` | Email address | Use for correlation to identity |
| `givenName` / `sn` | First / Last name | Identity display |
| `department` | Role assignment rules | Often dirty — plan transforms |
| `title` | Job classification | Often dirty |
| `manager` | DN of manager object | Requires lookup to get email |
| `memberOf` | Group memberships | Entitlement source |
| `employeeID` | Employee number | Primary correlation attribute |
| `accountExpires` | Account expiry | For contractor lifecycle |
| `userAccountControl` | Account enabled/disabled | Status derivation |

What you explicitly do NOT need: `userPassword`, `badPasswordCount`, `logonHours`, `homeDirectory` (unless there's a governance reason). Minimize scope.

**How many accounts are in scope?**
GlobalTech's `corp` domain has 8,400 employee accounts and 1,200 contractor accounts. Knowing this upfront lets you size the aggregation window, set expectations on first-run duration, and plan VA capacity.

> ⚠️ **COMMON MISTAKE:** Discovering out-of-scope OUs mid-project. A forgotten OU containing 300 application service accounts triggers a design scramble during testing. Ask the AD team for a complete OU tree export before requirements sign-off.

---

## Step 2: Connector Selection

For on-premises AD: the SailPoint Active Directory connector is pre-built and VA-required. No evaluation needed — it's the right connector. Configure one source per AD domain. If you have three domains, you create three ISC sources.

For Azure AD (Entra ID): the Microsoft Entra ID connector is direct (no VA). It's a separate connector from the on-premises AD connector. Many organizations run both — on-prem AD remains authoritative for some attributes; Entra ID is authoritative for cloud-only users. Clarify this during requirements.

**VA placement decision**: The VA must have network path to the AD domain controllers on port 389 (LDAP) or 636 (LDAPS). GlobalTech places its VAs in the corporate datacenter with direct DC access. If your VAs are in a DMZ, you need firewall rules allowing LDAP from the VA subnet to the DC subnet — confirm with networking before configuration starts.

---

## Step 3: Connection Configuration

In the ISC admin console: Admin → Connections → Sources → Add Source → Active Directory.

**Authentication**: ISC connects to AD using a service account. Requirements for this account:
- Domain user account (not an admin account — principle of least privilege)
- Read permissions on target OUs (no write needed for source-only integration)
- Member of the `Domain Users` group (default, no elevation needed for read)
- Password set to never expire — or a credential rotation process in ISC

Document the service account name and where it lives (`svc-isc-ad-readonly@corp.globaltech.com`). Expired service account credentials are the number one cause of AD connector outages.

**Connection settings**:
- LDAP Host: domain controller FQDN or a round-robin DNS name for the domain (`corp.globaltech.com` resolves to DCs automatically)
- Port: 636 (LDAPS) — use encrypted LDAP in production; plain 389 only for testing
- Search Base: `DC=corp,DC=globaltech,DC=com` (start from domain root; filter by OU in the connector object filter)
- Object filter: `(&(objectClass=user)(!(objectClass=computer)))` — users only, no computer objects

**Test the connection**: Click Test Connection before proceeding. If it fails, check service account credentials, network path from VA to DC, and LDAP port reachability. Fix connectivity before attempting anything else.

---

## Step 4: Schema Definition

The schema defines which AD attributes ISC reads. The connector auto-discovers available attributes from your AD schema, but you select which ones to import.

**Steps**:
1. Click Discover Schema — ISC queries AD and returns all available attribute names
2. Select the attributes from your requirements list (Step 1)
3. Map each to an ISC identity attribute or mark as an account-level attribute only

**The `memberOf` attribute** (group memberships): Add this — it's how AD entitlements (group memberships) get aggregated. Without it, ISC sees accounts but not what groups they belong to.

**Multi-value attributes**: `memberOf` is multi-value (an account can be in many groups). Mark it multi-value in the schema. Single-value schema on a multi-value attribute returns only the first value — a common configuration mistake that causes entitlement aggregation failures.

**userAccountControl decoding**: This AD attribute is a bitmask flag field. An account with `userAccountControl = 512` is enabled; `66048` is enabled with password never expires; `514` is disabled. ISC's AD connector handles the decoding automatically — map it to `IIQDisabled` and the connector translates the bitmask to true/false.

> 💡 **REMEMBER:** ISC's AD connector schema is pre-populated with the most common attributes. Review the pre-populated list against your requirements — you'll likely add a few custom extension attributes (`extensionAttribute1`-`15`) and remove attributes you don't need.

---

## Step 5: Correlation Rules

For AD as a source, correlation links AD accounts to ISC identities. If AD is your first (and only) source, identities don't exist yet — the first full aggregation creates them. But if you're adding AD as a second source (after Workday, for example), you need correlation rules to link AD accounts to existing Workday-sourced identities.

**GlobalTech's AD correlation strategy** (adding AD after Workday is already the authoritative source):

Primary rule (priority 10):
- AD attribute `employeeID` equals Identity attribute `employeeId`

Secondary rule (priority 20):
- AD attribute `mail` equals Identity attribute `email`

Tertiary rule (priority 30):
- BeanShell rule: extract username from `userPrincipalName` (strip `@corp.globaltech.com`), match against Identity `uid`

Each subsequent rule catches accounts the previous rules missed — contractors without `employeeID` populated, executives with custom email formats, legacy accounts predating the `employeeID` standardization.

**Exclusion filter**: Add an exclusion for accounts in the Service Accounts OU: if `distinguishedName` contains `OU=Service Accounts`, skip correlation. This prevents 300 service accounts from generating uncorrelated account alerts.

---

## Step 6: Test Aggregation

Run aggregation in stages. Do not run directly against 9,600 production accounts on the first attempt.

**Stage 1 — Single account test**: Create a test AD account in a test OU. Configure the source search base to that test OU only. Run aggregation. Verify one account appears in ISC with correct attributes. Check each attribute value against what's in AD. Fix any attribute mapping issues before proceeding.

**Stage 2 — Small OU test**: Point the search base at one real employee OU (not the full domain). Pick one with 50–100 accounts. Run aggregation. Check:
- Account count matches what AD reports in that OU
- Attribute values look correct (no truncation, correct encoding)
- `memberOf` shows groups correctly
- Correlation results for any known identities in that OU

**Stage 3 — Full aggregation (report-only correlation)**: Expand to the full scope. Run with correlation in preview mode — ISC computes correlation results but makes no changes. Review: how many accounts correlate? How many are uncorrelated? Categorize uncorrelated accounts.

**Stage 4 — Full aggregation (live)**: After Stage 3 issues are resolved, run the full live aggregation. Verify final counts, review the aggregation log for errors, spot-check 10 known identities.

**Common problems at each stage**:

| Stage | Common Problem | Resolution |
|---|---|---|
| 1 | Connection timeout | VA can't reach DC — verify network path |
| 1 | Authentication failure | Service account credentials wrong or account locked |
| 2 | `memberOf` returns only one group | Multi-value not set in schema |
| 2 | `department` value truncated | Max length setting too short |
| 3 | High uncorrelated rate | Correlation rule logic; check attribute values on both sides |
| 4 | Aggregation job hangs | VA resource pressure; DC performance; timeout settings |

---

## Key Takeaways

- Requirements before configuration: know your OUs, attributes, account count, and domain structure before logging into ISC
- Service account credentials for the VA connection are a recurring operational risk — document them and build a rotation process
- Schema: select minimum necessary attributes; mark multi-value attributes (especially `memberOf`) correctly
- Correlation: build a primary rule on a persistent unique identifier, then fallback rules for edge cases; add exclusion filters for service accounts
- Stage aggregation testing — single account → small OU → full preview → full live — to catch issues before they affect all accounts
- The aggregation log tells you everything that went wrong; errors field non-zero means something needs investigation

> 🔗 **CONNECTS TO:** With your source integration running, the next step is connecting target systems — the applications where ISC will provision and manage accounts based on the identity data your source provides.
