# Aggregation — How ISC Gets Data FROM Systems

Identity governance requires data. ISC needs to know who has accounts on which systems, which entitlements those accounts carry, and whether any of that has changed since the last time it checked.

ISC doesn't get this data by magic. It gets it through **aggregation** — the process of reading account and entitlement data from connected systems and loading it into ISC's internal database.

Aggregation is a READ operation. ISC sends a request to the source system's connector, the connector queries the system (via API, LDAP, JDBC, SCIM, or file), transforms the returned data into ISC's schema, and stores the results in ISC. The source system doesn't change. ISC just reads what's there and builds its governance picture from it.

## The Aggregation Flow: What Actually Happens

Here's the exact sequence for Active Directory aggregation at GlobalTech (8,400 accounts, 340 groups):

**Step 1 — Connection**: ISC's Virtual Appliance (running inside GlobalTech's network) opens a connection to the domain controller. Connection uses LDAP/S over port 636, authenticated with the service account `svc-isc-ad` (a dedicated read-only service account).

**Step 2 — Schema read**: ISC reads the AD schema to understand what attributes are available. For GlobalTech's AD, the configured attributes to read per account: `samAccountName`, `displayName`, `mail`, `userPrincipalName`, `memberOf`, `userAccountControl`, `distinguishedName`, `department`, `title`, `manager`, `employeeID`, `passwordLastSet`.

**Step 3 — Account enumeration**: ISC queries AD for all accounts within the configured OUs (Organizational Units). GlobalTech has ISC configured to read from `OU=GlobalTech-Users,DC=globaltech,DC=com` and its sub-OUs. Result: 8,400 account objects returned.

**Step 4 — Entitlement enumeration**: ISC queries AD separately for all group objects within scope. GlobalTech has 340 groups configured for governance. ISC reads each group's name, description, members, and `distinguishedName`.

**Step 5 — Transformation**: The connector maps AD attributes to ISC's internal schema. `samAccountName` → ISC `nativeIdentity` (the account's unique identifier in ISC). `memberOf` → ISC entitlement list. `userAccountControl` bit flags → ISC account status (enabled/disabled).

**Step 6 — Delta detection**: ISC compares the new data against what it stored in the previous aggregation. New accounts get created in ISC. Changed attributes get updated. Missing accounts get flagged (either deleted or out of scope).

**Step 7 — Correlation**: ISC attempts to link each account to an identity in its database (covered in depth in the next document). Matched accounts become linked to the identity. Unmatched accounts become orphaned/uncorrelated.

**Step 8 — Results**: Aggregation completes. ISC generates a summary: 8,400 accounts processed, 8,388 correlated, 12 uncorrelated, 47 accounts updated, 3 new accounts created, 1 account disabled (detected change), 0 errors.

> 🎯 **KEY POINT:** Aggregation is ISC's source of truth update. After every aggregation run, ISC's picture of that system reflects the current state of that system — up to the moment the aggregation started. Between aggregations, ISC's data is stale by definition. The frequency of aggregation determines the freshness of governance data.

## Full Aggregation vs. Delta Aggregation

Running a full aggregation of 8,400 AD accounts takes time — reading every account and every entitlement, transforming the data, comparing against stored records. For GlobalTech's AD, it takes about 23 minutes.

Running this every hour would strain both ISC and the domain controller. Running it once a week means governance data is potentially seven days stale.

ISC solves this with two aggregation modes:

**Full Aggregation**: Reads every account and entitlement in scope. Detects all changes including deletions. Takes longer but gives a complete, authoritative picture. Typically scheduled nightly or weekly for large directories.

**Delta Aggregation**: Reads only accounts and entitlements that have changed since the last aggregation. Uses the source system's change tracking capability (Active Directory Change Log, Salesforce's Modified date field, etc.). Much faster — GlobalTech's delta aggregation of AD takes about 2 minutes and picks up accounts changed in the last 24 hours.

**GlobalTech's aggregation schedule for AD**:
- Full aggregation: Every Sunday at 2:00 AM
- Delta aggregation: Every 4 hours (6 AM, 10 AM, 2 PM, 6 PM, 10 PM)

This means governance data is never more than 4 hours stale for changes, with a complete reconciliation weekly.

> ⚠️ **COMMON MISTAKE:** Assuming delta aggregation catches everything that full aggregation catches. It doesn't. Delta aggregation relies on the source system correctly tracking and exposing changes. If a change happens between delta runs, if the change tracking mechanism has a bug, or if a record was deleted (deletions are often harder to detect in deltas than additions/updates), the delta might miss it. Full aggregation is the safety net that catches what deltas miss.

## What Gets Aggregated: Accounts vs. Entitlements

Aggregation reads two distinct types of objects:

**Account Aggregation**: Reads user account objects from the source system. For each account, reads the configured attributes (username, display name, status, department, etc.). Creates or updates account records in ISC. This is the primary aggregation operation.

**Entitlement Aggregation**: Reads the entitlement objects themselves — AD groups, Salesforce profiles, GitHub teams, AWS IAM policies. This populates ISC's entitlement catalog with descriptions, metadata, and risk ratings. Without entitlement aggregation, ISC can see that Sarah belongs to an AD group called `CN=Finance-ReadWrite` — but it doesn't know what that group is or what it grants until the group object itself has been aggregated.

For most systems, you run both types. Account aggregation updates the governance picture for each person. Entitlement aggregation keeps the entitlement catalog current (new groups created, old groups deleted, descriptions updated).

> 💡 **REMEMBER:** Entitlement aggregation frequency matters for certification quality. If a new high-risk AD group is created on Monday and the entitlement aggregation runs weekly on Sunday, that entitlement is invisible to ISC for the better part of a week. For high-risk systems, run entitlement aggregation at least as frequently as account aggregation.

## A Real Aggregation at Scale

GlobalTech has 47 connected systems. Here's their complete aggregation schedule overview:

| System Type | Frequency | Rationale |
|---|---|---|
| Active Directory | Full weekly, delta 4x daily | Identity backbone, changes frequently |
| Workday (HR) | Full nightly | Authoritative source, must be current |
| Salesforce | Full nightly | Business-critical CRM |
| GitHub | Full nightly | Developer access changes frequently |
| AWS | Full twice daily | Cloud infrastructure, high risk |
| Jira | Full weekly | Low volatility, low risk |
| Slack | Full weekly | Low governance priority |
| SAP Finance | Full nightly | High risk, regulatory requirement |
| Legacy payroll | Flat file, weekly | No API connector, batch file |

Designing aggregation schedules is a judgment call: risk level of the system, volatility of its data, capacity of the connector and VA, and the governance SLAs your organization commits to.

## Aggregation Errors: What They Mean

Aggregation errors are governance signals. They're not just technical failures — each error type tells you something specific about the state of your integration.

**Connection timeout**: The VA can't reach the source system. Network issue, firewall change, or system downtime. Aggregation stops entirely. Previous data remains in ISC — stale, but not deleted. This is why ISC doesn't automatically remove accounts just because a system is unreachable.

**Authentication failure**: The service account credentials are wrong or the account is locked. Common cause: password expiration on the ISC service account. ISC admins notoriously forget to update service account passwords. Use non-expiring service accounts where policy allows, or build alerting for upcoming password expirations.

**Schema mismatch**: The source system's attribute schema changed (new field, renamed field, required field removed) and the connector configuration doesn't match. Example: Salesforce adds a required field during an API upgrade, and the ISC Salesforce connector wasn't updated to handle it. Fix: update the connector configuration.

**Partial aggregation / skipped accounts**: Some accounts were read successfully but others were skipped due to format errors, missing required fields, or permission issues. The aggregation completes but with gaps. Review the error log for specific account identifiers that failed.

> ⚠️ **COMMON MISTAKE:** Accepting aggregation errors as normal background noise. Every aggregation error means something wasn't read. If 15 accounts error consistently every night, those 15 accounts have governance gaps — their data isn't being updated, their entitlements might be stale, and they won't receive certification updates. Every aggregation error needs a root cause and a fix.

## Key Takeaways

- Aggregation is ISC's mechanism for reading data FROM source systems — it's always a read-only operation from the source system's perspective
- Two modes: full aggregation (reads everything, slower, authoritative) and delta aggregation (reads changes only, faster, supplementary)
- Both accounts and entitlements need to be aggregated — account aggregation updates the governance picture, entitlement aggregation populates the catalog
- Aggregation schedule design balances data freshness against system load — risk level and data volatility drive the decision
- Aggregation errors are governance gaps, not just technical inconveniences — every error needs a root cause, not just acknowledgment

> 🔗 **CONNECTS TO:** Aggregation reads accounts from systems. But how does ISC know which accounts belong to which person? That's the correlation challenge — covered next.
