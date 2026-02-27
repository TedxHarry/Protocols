# Real Provisioning Failures and How to Fix Them

A provisioning failure is not an abstract error — it's a specific person who doesn't have access they've been approved for, or a terminated employee whose account wasn't disabled. Either way, someone is waiting and you're the one who needs to diagnose it.

Provisioning failures have a reputation for being opaque. The error message says `INVALID_CROSS_REFERENCE_KEY` and you're left to figure out what that means in context. This document gives you the diagnostic path for the six failure types you'll encounter most often — from the error message to the root cause to the fix.

---

## Before Diagnosing: Find the Right Log

When a provisioning operation fails, there are two places to look:

**ISC Provisioning Activity** (Admin → Provisioning Activity): Shows every provisioning operation — pending, successful, failed. Click a failed operation to see the error details, the provisioning plan that was attempted, and the timestamp. Start here.

**VA Log** (for on-premises connectors): If the error is connector-level (auth failure, network timeout, AD write failure), the VA log at `/home/sailpoint/log/ccg.log` has the raw error from the connector before ISC processed it. Use this when the ISC error message is generic.

**Target system logs**: Some systems log provisioning attempts on their own side (Salesforce setup audit, AD security event log). When ISC says the operation failed but doesn't explain why, check the target system's own logs.

---

## Failure 1: Authentication Failed

**Error message** (from ISC or VA log):
```
ConnectorException: Authentication failed for source 'Active Directory Corp'
LDAP Error Code: 49 - Invalid Credentials
```
Or for SaaS systems:
```
HTTP 401 Unauthorized - OAuth token rejected
```

**What it means**: The credentials ISC is using to connect to the target system are wrong or expired. The connector got as far as attempting authentication but the target system rejected it.

**Causes by system type**:

| System | Common Cause |
|---|---|
| Active Directory | Service account password expired; account locked after failed attempts |
| Salesforce / SaaS | OAuth token expired; connected app credentials rotated |
| REST API systems | API key rotated; service account disabled |
| LDAP | Certificate expired (for LDAPS); bind DN changed |

**Diagnostic steps**:
1. Try connecting to the target system manually with the same credentials (test AD login, try the API key in Postman). If manual auth fails, the credentials are wrong.
2. For AD: check if the service account password expired (default AD policy = 90 days). Unlock if locked.
3. For OAuth: generate a new token and update in ISC source configuration.
4. Retest the connection in ISC (Source → Test Connection).

**Prevention**: Automate credential rotation reminders. ISC doesn't automatically detect expired AD service account passwords — it fails at the next aggregation or provisioning attempt. Set a calendar reminder 2 weeks before service account password expiration, or configure the service account's password to never expire and document that policy exception.

> ⚠️ **COMMON MISTAKE:** Changing the service account password in AD without updating it in ISC. This immediately breaks all aggregations and provisioning for that source. Always update ISC first (or simultaneously) when rotating credentials.

---

## Failure 2: Required Field Missing

**Error message**:
```
ProvisioningException: Required field 'ProfileId' is not set in provisioning plan
```
Or from Salesforce:
```
REQUIRED_FIELD_MISSING: Required fields are missing: [ProfileId]
```

**What it means**: The provisioning plan was built, but a field the target system requires before it will create the account was absent from the plan. The target system rejected the create operation.

**Causes**:
- Access profile doesn't include the required entitlement (Salesforce Profile, a required AD group)
- Provisioning policy doesn't map the required attribute
- The identity attribute that feeds the required field is null (transform returned null, source data missing)

**Diagnostic path**:
1. Open the failed provisioning operation in ISC Activity → view the Provisioning Plan that was attempted. Does it include the missing field?
2. If the field isn't in the plan: go to the access profile → provisioning policy → verify the required field is mapped
3. If the field is in the plan but has a null value: go to the provisioning policy attribute mapping → check the source of that attribute value → check the identity's current attribute value
4. Trace the null upstream: identity attribute is null → the source aggregation didn't populate it → check the source system's data

**Fix**: Either populate the required attribute in the source system's data, or add a static default value in the provisioning policy for required fields that can have a sensible default.

> 💡 **REMEMBER:** Required field failures are always traceable to a specific attribute mapping. Work backward from the missing field → provisioning policy → identity attribute → source system data. Don't guess; follow the chain.

---

## Failure 3: Duplicate Account

**Error message**:
```
ProvisioningException: Account already exists in target system
DUPLICATE_VALUE: duplicate value found: Username duplicates value on record
```

**What it means**: ISC attempted to create an account, but an account with the same unique identifier (username, email, or other unique field) already exists in the target system.

**Why this happens**:
- The person previously had an account (rehire scenario) that was never deleted
- Another system created an account for this person before ISC's provisioning ran
- The username generation logic produces the same username for two different people (collision)

**Diagnostic steps**:
1. Check the target system directly: does an account exist with the generated username?
2. If yes: is it this person's old account (rehire) or someone else's account (collision)?
3. For rehire: the existing account should be reactivated and linked. Update conflict resolution in the provisioning policy from "Create New" to "Link to Existing"
4. For collision: your username generation needs a uniqueness check. Configure the `UniqueValue` generator in the provisioning policy — it automatically appends a counter (`john.smith1`, `john.smith2`) when the base username is taken.

**Fix for rehire scenario**:
Change the provisioning policy conflict resolution to "Link to Existing Account" (match by email or another stable attribute). On next provisioning attempt, ISC finds the existing account, enables it, and adds the required entitlements instead of creating a duplicate.

> 🎯 **KEY POINT:** Rehires are common enough in enterprise environments that "Link to Existing" should be the default conflict resolution for most target systems, not "Create New." Design for the expected case, not just the ideal case.

---

## Failure 4: Permission Denied

**Error message**:
```
ConnectorException: Insufficient privileges to create user
Salesforce error: INSUFFICIENT_ACCESS_ON_CROSS_REFERENCE_ENTITY
```
Or for AD:
```
LDAP Error Code: 50 - Insufficient Access Rights
```

**What it means**: ISC authenticated successfully (credentials are correct) but the service account doesn't have the permissions needed to perform the provisioning operation in the target system.

**Causes**:
- Service account provisioned with read-only access (fine for source-only, wrong for provisioning target)
- Service account has create permission but not modify permission (can create accounts, can't add to groups)
- Target system scope restriction: service account can provision in some OUs but not others

**Diagnostic steps**:
1. Identify the exact operation that failed (create account? add to group? modify attribute?)
2. Check what permissions the ISC service account has in the target system
3. For AD: check which OUs the service account has Write permissions on. If the target OU isn't included, that's the gap.
4. For Salesforce: check the connected app permissions and the profile assigned to the service account

**Fix**: Grant the service account the minimum permissions needed for provisioning operations:
- AD: Write permission on target OUs (create/modify user objects), Write permission on target groups (add members)
- Salesforce: `Manage Users` system permission on the service account's Salesforce profile
- GitHub: `admin:org` scope on the OAuth token (for team membership management)

Document the permissions granted to the service account. When someone asks in six months why the service account has elevated permissions, the documentation should answer it.

> ⚠️ **COMMON MISTAKE:** Granting the service account full admin rights to fix a permission error. This violates least privilege. Identify the specific permission that's missing and grant exactly that. Full admin "fixes" everything immediately but creates a privileged account that's a security risk and an audit finding.

---

## Failure 5: Data Validation Error

**Error message**:
```
ConnectorException: Field validation failed: 'department' value 'Engineering-New' not in allowable values
```
Or:
```
INVALID_OR_NULL_FOR_RESTRICTED_PICKLIST: TimeZoneSidKey: bad value for restricted picklist field
```

**What it means**: The value ISC is trying to write to the target system doesn't pass the target system's own validation. The field may be a restricted picklist, a foreign key to a record that doesn't exist, or a field with a specific format requirement.

**Causes**:
- A new department, location, or classification was added in the HR system that hasn't been added to the target system's picklist
- A lookup transform references a value that exists in ISC but not in the target
- A date/format transform produces a value in the wrong format for the target

**Diagnostic steps**:
1. Identify which field failed and what value was attempted (from the error message or the provisioning plan)
2. Check what valid values the target system accepts for that field
3. Find where ISC derives that value (provisioning policy attribute mapping or transform)
4. Determine if it's a new value problem (add it to the target system) or a transform problem (fix the transform)

**Fix scenarios**:
- New department not in Salesforce: add the new department value to the Salesforce `Department` picklist
- TimeZoneSidKey wrong: update the static value to a valid Salesforce time zone identifier (e.g., `"America/New_York"` not `"Eastern Time"`)
- Date format wrong: update the DateFormat transform to match the target's expected format

---

## Failure 6: Network Timeout

**Error message**:
```
ConnectorException: Connection timed out after 30000ms
SocketTimeoutException: Read timed out
```

**What it means**: ISC (or the VA) attempted to connect to the target system, initiated the operation, but didn't get a response within the configured timeout window.

**Causes**:
- Target system is down or under heavy load
- Network path between VA and target system is congested or interrupted
- Provisioning operation triggers a long-running process in the target system that exceeds timeout
- VA itself is under resource pressure and couldn't process the response in time

**Diagnostic steps**:
1. Can the target system be reached at all? Test from the VA host: `curl -v https://targetapi.com/health` or LDAP search from VA to DC.
2. Is the target system responsive? Check its health dashboards (Salesforce Trust, ServiceNow status pages, AD performance counters).
3. Is the VA under load? Check VA health in ISC console — if Degraded, address VA resource pressure first.
4. Is the timeout threshold too low? Some complex provisioning operations (especially SAP, Oracle) legitimately take longer than default timeouts.

**Fix options**:
- If target system is down: wait for recovery, then retry
- If network issue: engage network team to investigate
- If timeout threshold too low: increase the connector timeout setting in the source configuration
- If VA is under load: add another VA to the cluster, redistribute work

**Retry behavior**: ISC has configurable retry logic for transient failures. Set up automatic retry (with exponential backoff) for network failures. Distinguish between transient failures (retry makes sense) and permanent failures (retry won't help — authentication failure, permission denied, data validation error).

---

## The Data Problem vs System Problem Decision

Every provisioning failure falls into one of two categories:

**Data problem**: The data driving the provisioning is wrong — null attribute, invalid value, wrong format. Fix: correct the data in the source or provisioning policy. The system is working correctly; it's rejecting bad input.

**System problem**: The system itself is misbehaving — authentication fails even with correct credentials, timeouts on simple operations, unexpected API errors. Fix: investigate the target system's health, credentials, or configuration. The provisioning logic is correct; the system isn't responding correctly.

Identifying which category you're in determines where to look. Authentication failures, permission errors, and timeouts are system problems. Required field missing, data validation errors, and format mismatches are data problems.

---

## Key Takeaways

- Start with ISC Provisioning Activity — it shows the plan and the error for every failed operation
- Authentication failures mean credentials have changed or expired — check service account status in the target system
- Required field missing is always traceable through: missing field → provisioning policy mapping → identity attribute → source system data
- Design conflict resolution for "Link to Existing" by default — rehires are common enough to plan for
- Permission denied: grant the specific missing permission, not broad admin rights
- Distinguish data problems (fix the input) from system problems (fix the connection/system) before investigating further

> 🔗 **CONNECTS TO:** Individual failure patterns are pieces of a larger diagnostic methodology. The next document provides a systematic approach to diagnosing any integration problem — source, target, or configuration — regardless of the specific error you're seeing.
