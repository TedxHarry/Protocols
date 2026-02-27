# Aggregation — Full, Delta, and Everything That Can Go Wrong

Aggregation is the process by which ISC reads data from a connected source system and updates its internal records to reflect what that system contains. It sounds simple. In practice, it's the operation that generates the most support tickets, the most midnight pages, and the most "why is this identity wrong in ISC?" conversations.

Understanding what aggregation actually does — mechanically, step by step — is what separates engineers who can debug aggregation problems systematically from engineers who change random settings hoping something helps.

---

## What Aggregation Does

When ISC aggregates a source system, it runs through five phases:

**Phase 1: Connect**
ISC (or the VA, for on-premises systems) authenticates to the source system and establishes connectivity. This is where connection refused, credential failure, and network timeout errors surface.

**Phase 2: Enumerate**
ISC reads all account records from the source system. For a full aggregation, this is every account in scope. For a delta aggregation, this is only accounts that changed since the last run.

What "enumerate" means concretely depends on the connector:
- AD connector: ISC runs an LDAP query against domain controllers, retrieving account objects in specified OUs
- Workday connector: ISC calls the Workday Workers API, paging through results until all workers are read
- Flat file: ISC reads every row from the CSV file

**Phase 3: Transform**
For each account record returned, ISC applies the attribute transforms configured for that source. `Worker_ID` → strip prefix → store as `employeeId`. Department code → lookup table → store as human-readable department name. This is where transform errors surface — null inputs, unexpected data formats, missing lookup values.

**Phase 4: Correlate**
For each account record, ISC attempts to find the matching identity in its identity cube. The correlation rule runs: does this account's `employeeId` match any identity's `employeeId`? Match found → account is associated with that identity. No match → account is uncorrelated.

**Phase 5: Update**
ISC updates its internal records to reflect the aggregation results. Correlated accounts update their associated identity attributes (if those attributes have ISC-side authority from this source). New accounts that didn't exist before are created in ISC. Accounts in ISC that weren't present in the aggregation (i.e., deleted from the source) are flagged.

After aggregation completes, ISC evaluates access policies against the updated data — role assignments, access profile membership, SoD policy evaluations. This is what turns raw data into governed access decisions.

---

## Full vs Delta Aggregation

### Full Aggregation

Full aggregation reads every record from the source system. ISC compares the complete set of current records against its previous state to identify: new accounts, deleted accounts, changed attributes.

**When to use full aggregation**:
- Initial aggregation (first time connecting a source)
- After suspected data quality issues (delta may have missed changes)
- Weekly or monthly validation runs to catch drift between ISC and the source
- After a source system migration that affected many records

**Cost of full aggregation**: For large sources (100,000+ accounts), full aggregation takes time — reading all records, running all transforms, updating all matching records. For GlobalTech's 12,000-employee Workday, a full aggregation takes about 40 minutes. For an organization with 500,000 identities, a full aggregation might run for hours. Schedule full aggregations during off-peak windows.

### Delta Aggregation

Delta aggregation reads only accounts that changed since the last aggregation run. "Changed" means: created, updated, or deleted.

How ISC knows what changed depends on the connector and the source system:
- **Timestamp-based**: ISC stores the last-run timestamp and asks the system for records modified after that time. Works for systems with reliable `modified_timestamp` fields (many HR systems, Workday, AD with change tracking).
- **Watermark-based**: For systems that maintain a change log or version counter, ISC uses the last-processed change ID to read only new changes.
- **Token-based**: Some systems (Okta, Microsoft Graph API) provide a delta token — a cursor representing the point in the change stream where you last stopped. ISC stores the token and uses it for the next delta run.

**When to use delta aggregation**:
- Frequent update cycles (every 1–4 hours) where full aggregation would be too slow
- High-volume sources where reading all records repeatedly is prohibitive
- Between weekly full aggregations for ongoing attribute updates

**The delta catch-up problem**: Delta aggregation relies on the source system's change tracking being reliable. If the source system's change log has gaps (planned maintenance, crash, export failure), ISC's delta will miss those changes. For this reason, most production configurations run:
- Delta aggregation every 4 hours (or on a schedule based on latency requirements)
- Full aggregation once per week (or after any known data issues)

> ⚠️ **COMMON MISTAKE:** Running delta aggregation only, never full. The assumption is that delta catches everything. It doesn't — timestamp gaps, time zone mismatches between ISC and the source system, change log failures, and daylight saving time rollbacks (yes, really) all create scenarios where changes are missed. Full aggregation is the safety net. Run it regularly.

---

## Entitlement Aggregation vs Account Aggregation

These are separate aggregation types with different purposes and often different schedules.

**Account aggregation**: Reads user accounts and their attributes. Who has an account in this system? What attributes do they have? Runs the correlation logic.

**Entitlement aggregation**: Reads groups, roles, profiles, or permission sets — the entitlements that accounts can be assigned. What entitlements exist in this system? This runs independently of account aggregation.

**Why they're separate**: Entitlements change less frequently than accounts. In Active Directory, security groups might be created or deleted monthly. AD accounts change daily. Running entitlement aggregation nightly is overkill; running it weekly is usually sufficient. Account aggregation runs more frequently because attributes and membership change more often.

**Ordering matters**: If you're configuring a new source for the first time, run entitlement aggregation before account aggregation. Account aggregation needs to know what entitlements exist so it can correctly record which groups an account belongs to. Run them out of order and you'll see group membership data that doesn't match anything in the entitlement catalog.

---

## Aggregation Scheduling

ISC uses cron-style scheduling for aggregation jobs. The admin console provides a visual scheduler, but understanding the cron syntax helps you set up precise schedules:

```
[minute] [hour] [day of month] [month] [day of week]
   0       2         *             *          *         → 2:00 AM daily
   0     */4         *             *          *         → Every 4 hours
   0       3         *             *          0         → 3:00 AM every Sunday
```

**GlobalTech's aggregation schedule (representative)**:

| Source | Aggregation Type | Schedule | Reasoning |
|---|---|---|---|
| Workday | Delta | Every 4 hours | Attribute changes, mid-day org changes |
| Workday | Full | Sundays 2:00 AM | Weekly validation |
| Active Directory | Delta | Every 2 hours | High-change environment |
| Active Directory | Full | Saturdays 1:00 AM | Weekly cleanup catch |
| Salesforce | Delta | Daily 3:00 AM | Slower-moving entitlements |
| GitHub | Full | Sundays 4:00 AM | Weekly (low change volume) |
| Legacy payroll | Full | Daily 3:00 AM | File arrives at 2:00 AM |

**Time zone consideration**: ISC operates in UTC. When you schedule an aggregation for "2:00 AM," that's 2:00 AM UTC — which may be 9 PM Eastern the previous evening. Configure schedules in the time zone your operations team monitors, and document the UTC equivalent. More than one production incident has started with "the aggregation didn't run last night" when it actually ran at 9 PM local time as configured.

---

## Performance Tuning

### Batch Size

Most connectors process accounts in batches — ISC reads 500 accounts, processes them, reads the next 500, etc. The default batch size is usually fine. For very large sources (500,000+ accounts), increasing batch size can reduce API round-trips. For slow API sources (rate limits, throttling), decreasing batch size may be more reliable.

### Concurrent Aggregations

Running 20 aggregations simultaneously strains both the VA and the connected systems. ISC allows you to configure maximum concurrent jobs per VA cluster. A reasonable default for a 2-VA cluster is 5–8 concurrent jobs during off-peak hours. If you're seeing aggregation slowdowns or timeouts, check if concurrent jobs are exceeding the VA's capacity.

### Rate Limiting

Cloud APIs have rate limits. Workday, Salesforce, and ServiceNow all enforce API call limits — too many calls per minute and the API starts returning 429 errors. ISC connectors handle this with retry logic and backoff, but persistent rate limiting indicates your aggregation frequency is too high for the API tier your organization has licensed.

---

## Reading the Aggregation Log

After any aggregation completes, ISC generates an aggregation status record in the admin console. This is your primary tool for understanding what happened.

**Key fields in the aggregation log**:

| Field | What It Tells You |
|---|---|
| `totalAccounts` | Total accounts read from the source |
| `processed` | Accounts successfully processed (transformed, correlation attempted) |
| `created` | New ISC accounts created (accounts that didn't exist before) |
| `updated` | Existing accounts updated with new attribute values |
| `deleted` | Accounts present in ISC but absent from the source (deleted) |
| `errors` | Accounts that failed to process — investigate every time this is non-zero |
| `optimisticProvisioning` | Accounts automatically updated in connected targets based on new data |
| `startTime / endTime` | Duration — trending duration helps catch degradation before it becomes failure |

**The errors field**: If `errors` is non-zero, ISC provides an error file — downloadable from the admin console — that lists every account that failed to process and why. Review this file after every aggregation. A few errors in a large aggregation might be data quality issues; hundreds of errors means the aggregation has a systemic problem.

**Duration trending**: If your Workday aggregation normally takes 40 minutes and today it took 2 hours, something changed. Common causes: source system performance degradation, VA resource pressure, increased record volume (large batch of new hires), API rate limiting. Track aggregation duration over time and alert when it exceeds 150% of normal.

---

## What Happens When an Account Disappears

When an account is present in ISC but absent from the latest aggregation, ISC needs to decide what to do. Three options, configured per source:

**Delete**: Remove the account from ISC entirely. Appropriate when accounts being absent definitively means they've been deleted from the source. Risk: if the aggregation had a data issue and returned partial results, accounts could be incorrectly deleted from ISC.

**Disable**: Mark the account as inactive in ISC but keep the record. Conservative option — preserves governance history even if the account is gone from the source. Good for auditable deprovisioning workflows.

**Leave Unchanged**: Do nothing. The account remains in ISC even though it's gone from the source. Useful during aggregation debugging when you don't want unexpected state changes.

**GlobalTech's configuration**: For Workday (authoritative HR source), a missing account triggers a lifecycle state evaluation — ISC checks if the employee ID still exists in any other source before marking the identity as terminated. For Active Directory, missing accounts are marked Inactive. For flat file sources, accounts are left unchanged until the next full aggregation validates the state.

> ⚠️ **COMMON MISTAKE:** Setting account deletion on a source with unreliable aggregation. If your flat file aggregation occasionally delivers an incomplete file (file truncated, SFTP transfer interrupted), "delete" mode will remove thousands of accounts from ISC. "Disable" mode is safer — it still reflects the data change but doesn't permanently remove governance history.

---

## Key Takeaways

- Aggregation has five phases: connect, enumerate, transform, correlate, update — each phase has its own failure modes
- Full aggregation reads everything; delta reads only changes since last run — run both on a schedule, not just delta
- Run entitlement aggregation before account aggregation when setting up a new source for the first time
- Aggregation scheduling uses UTC time — document the local-time equivalent to avoid confusion
- The aggregation error file lists every failed account — review it after every aggregation, even in production
- Account deletion on missing records is risky — prefer disable unless you're certain about data completeness

> 🔗 **CONNECTS TO:** Aggregation brings data in. Transforms are what shape that data into the form ISC needs. The next document covers every built-in transform in detail — the tools you'll use to handle 80% of data manipulation without writing code.
