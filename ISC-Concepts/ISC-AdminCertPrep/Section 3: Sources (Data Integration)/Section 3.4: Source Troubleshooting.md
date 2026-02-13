# Section 3.4: Source Troubleshooting

**Master identifying and resolving common source connectivity and aggregation issues**

---

## Table of Contents

- [3.4.1 Searching for Source Errors](#341-searching-for-source-errors)
- [3.4.2 Common Source Issues and Resolutions](#342-common-source-issues-and-resolutions)

---

# 3.4.1 Searching for Source Errors

**Learning Objective:** Learn how to locate and interpret source errors in ISC and VA logs.

---

## Where Source Errors Appear

### **Error Locations**

| Location | Type of Errors | When to Check |
|----------|---------------|---------------|
| **ISC UI - Source Status** | Connection, aggregation status | First place to check |
| **ISC UI - Activity Log** | Aggregation failures, provisioning errors | Review recent operations |
| **VA Connector Logs** | Detailed operation errors | Deep troubleshooting |
| **Source System Logs** | Source-side issues | When ISC/VA logs unclear |

**Exam Tip:** Always start with ISC UI, then check VA logs for details.

---

## Checking Source Status in ISC

### **Quick Source Health Check**

**Navigation:**
```
Admin → Connections → Sources
```

**What you'll see:**
```
Sources List:

Name                Status      Last Aggregation    Accounts
Active Directory    ● Healthy   2024-02-13 02:00   10,247
Workday            ● Healthy   2024-02-13 01:00   10,500
Salesforce         ● Healthy   2024-02-13 00:30   5,230
HR Database        ✗ Error     2024-02-12 02:00   0
```

**Status indicators:**
- **● Green (Healthy)**: Source working normally
- **⚠ Yellow (Warning)**: Some issues, partial functionality
- **✗ Red (Error)**: Source failing, needs attention

### **Source Detail View**

**Click on source for details:**
```
Source: Active Directory

Status: ● Healthy
Last Successful Aggregation: 2024-02-13 02:00:15
Accounts: 10,247
Entitlements: 250

Recent Activity:
  2024-02-13 02:00 - Account Aggregation - Success (10,247 accounts)
  2024-02-12 02:00 - Account Aggregation - Success (10,245 accounts)
  2024-02-11 03:00 - Entitlement Aggregation - Success (250 groups)
  2024-02-10 02:00 - Account Aggregation - Failed ✗
    Error: Connection timeout
```

**Click on failed event for details:**
```
Aggregation Details:

Start Time: 2024-02-10 02:00:00
End Time: 2024-02-10 02:00:45
Status: FAILED
Duration: 45 seconds
Accounts Processed: 0

Error Message:
  "Connection timeout connecting to dc01.company.com:389"
  
Error Code: CONNECTION_TIMEOUT

Suggested Actions:
  - Verify domain controller is online
  - Check network connectivity
  - Review firewall rules
  - Check VA status
```

---

## Searching for Errors in ISC

### **Using Activity Search**

**Navigate to:**
```
Activity → All Activity (or Provisioning Activity)
```

**Filter for errors:**
```
Filters:
  Status: Failed
  Source: Active Directory
  Date Range: Last 7 days
  Type: Aggregation

Results:
  2024-02-10 02:00 - Account Aggregation - Failed
  2024-02-08 14:30 - Account Provisioning - Failed
```

**Click each to view details**

### **Using Search Bar**

**Search queries:**
```
Find failed aggregations:
  @activity(type:aggregation AND status:failed)

Find errors for specific source:
  @activity(source.name:"Active Directory" AND status:failed)

Find errors in date range:
  @activity(status:failed AND created:[2024-02-10 TO 2024-02-13])
```

---

## Checking VA Logs

### **Primary Log Files**

**Location:** `/opt/sailpoint/logs/`

**Key files for source troubleshooting:**

**1. connector.log - Most Important**
```bash
# Real-time monitoring
tail -f /opt/sailpoint/logs/connector.log

# Search for errors
grep -i "error" /opt/sailpoint/logs/connector.log

# Search for specific source
grep "Active Directory" /opt/sailpoint/logs/connector.log | grep -i "error"

# Last 100 lines
tail -100 /opt/sailpoint/logs/connector.log
```

**2. va.log - VA Service Issues**
```bash
# Check for VA-level problems
grep -i "error" /opt/sailpoint/logs/va.log

# Check source initialization
grep "Active Directory" /opt/sailpoint/logs/va.log
```

**3. ccg.log - Communication Issues**
```bash
# Check ISC communication
grep -i "error" /opt/sailpoint/logs/ccg.log

# Check if commands received
grep "aggregation" /opt/sailpoint/logs/ccg.log
```

### **Reading Connector Logs**

**Example - Successful aggregation:**
```
2024-02-13 02:00:00 INFO  [Connector] Account aggregation started: Active Directory
2024-02-13 02:00:01 INFO  [AD-Connector] Connecting to dc01.company.com:389
2024-02-13 02:00:02 INFO  [AD-Connector] Connection successful
2024-02-13 02:00:02 INFO  [AD-Connector] Authenticating as svc_sailpoint@company.com
2024-02-13 02:00:03 INFO  [AD-Connector] Authentication successful
2024-02-13 02:00:03 INFO  [AD-Connector] Executing LDAP search
2024-02-13 02:00:03 DEBUG [AD-Connector] Filter: (&(objectClass=user)(!(objectClass=computer)))
2024-02-13 02:00:03 DEBUG [AD-Connector] Base DN: DC=company,DC=com
2024-02-13 02:02:45 INFO  [AD-Connector] Retrieved 10,247 user accounts
2024-02-13 02:02:50 INFO  [Connector] Sending accounts to ISC (batch 1/11)
2024-02-13 02:03:20 INFO  [Connector] Sending accounts to ISC (batch 2/11)
...
2024-02-13 02:05:42 INFO  [Connector] All batches sent successfully
2024-02-13 02:05:43 INFO  [Connector] Aggregation completed: 10,247 accounts in 5m 43s
```

**Example - Failed aggregation:**
```
2024-02-10 02:00:00 INFO  [Connector] Account aggregation started: Active Directory
2024-02-10 02:00:01 INFO  [AD-Connector] Connecting to dc01.company.com:389
2024-02-10 02:00:31 ERROR [AD-Connector] Connection timeout after 30 seconds
2024-02-10 02:00:31 ERROR [AD-Connector] Failed to connect to domain controller
2024-02-10 02:00:32 ERROR [Connector] Aggregation failed: Active Directory
2024-02-10 02:00:32 ERROR [Connector] Error: CONNECTION_TIMEOUT
2024-02-10 02:00:32 INFO  [CCG] Reporting failure to ISC
```

**Key information to extract:**
- **When**: Timestamp of error
- **What**: Type of operation (aggregation, provisioning)
- **Where**: Which source, which server
- **Why**: Error message and code
- **How far**: What step failed

---

## Common Log Patterns

### **Connection Errors**

**Pattern:**
```
ERROR [Connector] Connection timeout
ERROR [Connector] Connection refused
ERROR [Connector] Network unreachable
ERROR [Connector] Unknown host
```

**What to look for:**
- Hostname/IP mentioned
- Port number
- Timeout duration
- When it occurred

### **Authentication Errors**

**Pattern:**
```
ERROR [AD-Connector] LDAP bind failed: Invalid credentials
ERROR [AD-Connector] Authentication failed for user svc_sailpoint
ERROR [JDBC-Connector] SQL authentication failed
ERROR [Connector] Access denied
```

**What to look for:**
- Username attempting to authenticate
- Authentication method
- Specific error code (LDAP error, SQL error)

### **Permission Errors**

**Pattern:**
```
ERROR [AD-Connector] Insufficient permissions to read OU=Users
ERROR [JDBC-Connector] SELECT permission denied on table employees
ERROR [Connector] Access denied to resource
```

**What to look for:**
- What resource was being accessed
- What operation was attempted (read, write)
- Which account was used

### **Data/Schema Errors**

**Pattern:**
```
ERROR [Connector] Required attribute 'email' not found
ERROR [Connector] Invalid attribute value for 'telephoneNumber'
ERROR [Connector] Schema validation failed
ERROR [Connector] Unexpected data type for attribute 'employeeID'
```

**What to look for:**
- Which attribute caused the issue
- What was expected vs received
- Which account/record had the problem

---

## Searching Logs Effectively

### **Time-Based Searches**

**Find errors in specific timeframe:**
```bash
# Errors between 2 AM and 3 AM today
grep "2024-02-13 02:" /opt/sailpoint/logs/connector.log | grep -i error

# Errors from yesterday
grep "2024-02-12" /opt/sailpoint/logs/connector.log.1 | grep -i error

# Last hour of errors
tail -1000 /opt/sailpoint/logs/connector.log | grep -i error
```

### **Source-Specific Searches**

**Find all errors for a specific source:**
```bash
# Active Directory errors
grep "Active Directory" /opt/sailpoint/logs/connector.log | grep -i error

# Show context (20 lines before and after)
grep -A 20 -B 20 "Active Directory.*error" /opt/sailpoint/logs/connector.log

# Count of errors
grep "Active Directory" /opt/sailpoint/logs/connector.log | grep -c -i error
```

### **Operation-Specific Searches**

**Find aggregation failures:**
```bash
# Failed aggregations
grep -i "aggregation failed" /opt/sailpoint/logs/connector.log

# Failed provisioning
grep -i "provisioning failed" /opt/sailpoint/logs/connector.log

# All failed operations
grep -i "failed" /opt/sailpoint/logs/connector.log
```

### **Error Code Searches**

**Find specific error types:**
```bash
# Connection timeouts
grep "CONNECTION_TIMEOUT" /opt/sailpoint/logs/connector.log

# Authentication failures
grep "LDAP_INVALID_CREDENTIALS\|Authentication failed" /opt/sailpoint/logs/connector.log

# Permission errors
grep "INSUFFICIENT_ACCESS\|Access denied" /opt/sailpoint/logs/connector.log
```

---

## Correlating Errors Across Systems

### **ISC → VA → Source**

**Troubleshooting flow:**

**Step 1: Check ISC UI**
```
ISC shows: "Aggregation failed at 2024-02-13 02:00"
Error: "Connection timeout"
```

**Step 2: Check VA Logs (connector.log)**
```bash
grep "2024-02-13 02:" /opt/sailpoint/logs/connector.log | grep -i "active directory"

Output:
2024-02-13 02:00:00 INFO  Aggregation started: Active Directory
2024-02-13 02:00:01 INFO  Connecting to dc01.company.com:389
2024-02-13 02:00:31 ERROR Connection timeout after 30 seconds
```

**Step 3: Check Source (if accessible)**
```
Domain Controller: dc01.company.com
Check:
  - Is DC online? (ping dc01.company.com)
  - Is LDAP service running?
  - Any errors in DC event logs?
  - Firewall blocking port 389?
```

**Step 4: Check Network**
```bash
# From VA, test connectivity
ping dc01.company.com
telnet dc01.company.com 389
nslookup dc01.company.com
```

**Step 5: Correlate findings**
```
ISC: Connection timeout
VA: Timeout after 30 seconds to dc01.company.com:389
Network: Cannot reach dc01.company.com
Source: DC is offline

Root Cause: Domain controller down
Solution: Restart DC or fail over to backup DC
```

---

## Log Retention and Rotation

### **Understanding Log Rotation**

**Default retention:**
```
/opt/sailpoint/logs/
├── connector.log          (Current - today)
├── connector.log.1        (Yesterday)
├── connector.log.2.gz     (2 days ago - compressed)
├── connector.log.3.gz     (3 days ago)
...
└── connector.log.30.gz    (30 days ago - will be deleted tomorrow)
```

**Searching rotated logs:**
```bash
# Current log
grep "error" /opt/sailpoint/logs/connector.log

# Yesterday's log
grep "error" /opt/sailpoint/logs/connector.log.1

# Compressed logs (use zgrep)
zgrep "error" /opt/sailpoint/logs/connector.log.*.gz

# All logs (current + rotated)
grep "error" /opt/sailpoint/logs/connector.log*
zgrep "error" /opt/sailpoint/logs/connector.log.*.gz
```

**Exam Tip:** Use `zgrep` for compressed (.gz) log files, regular `grep` for uncompressed.

---

## Exporting Logs for Support

### **When to Export Logs**

**Situations requiring log export:**
- Persistent errors not resolved by troubleshooting
- Need to share with SailPoint support
- Complex issues requiring deep analysis
- Documentation for audit/compliance

### **How to Export**

**Option 1: Via ISC UI (if available)**
```
Source → Settings → Download Logs
  - Last 24 hours
  - Last 7 days
  - Custom date range
```

**Option 2: Directly from VA**
```bash
# Create archive of logs
cd /opt/sailpoint/logs
tar -czf sailpoint-logs-$(date +%Y%m%d).tar.gz *.log *.log.* 

# Copy to local machine
scp sailpoint@va-server:/opt/sailpoint/logs/sailpoint-logs-20240213.tar.gz .
```

**What to include:**
```
Logs to export:
  ✓ connector.log (and recent rotations)
  ✓ va.log
  ✓ ccg.log
  ✓ Specific time range covering the issue
  
Context to provide:
  ✓ Source name
  ✓ When error occurred
  ✓ What operation failed
  ✓ Steps already taken
  ✓ ISC tenant name
  ✓ VA version
```

---

## Practice Exercise

**Question:** A source aggregation failed at 2 AM today. Walk through the steps to find and diagnose the error.

<details>
<summary>Answer</summary>

**Systematic Error Investigation Process:**

**Step 1: Check ISC UI (Quick Assessment)**

**Navigate:**
```
Admin → Connections → Sources
```

**Look for:**
```
Source list shows:

Active Directory    ✗ Error    2024-02-13 02:00    0 accounts
```

**Click on source:**
```
Active Directory Details:

Status: ✗ Error
Last Successful: 2024-02-12 02:00 (yesterday)
Last Failed: 2024-02-13 02:00 (today - 12 hours ago)

Recent Activity:
  2024-02-13 02:00 - Account Aggregation - FAILED ✗
```

**Click on failed aggregation:**
```
Aggregation Details:

Start Time: 2024-02-13 02:00:00
End Time: 2024-02-13 02:00:45
Status: FAILED
Duration: 45 seconds
Accounts: 0

Error Message:
  "LDAP bind failed: Invalid credentials"
  
Error Code: LDAP_INVALID_CREDENTIALS

Connector: Active Directory
VA: VA-PROD-01
```

**Initial findings:**
- ✓ Identified: Authentication failure
- ✓ When: 2024-02-13 02:00
- ✓ Where: Active Directory via VA-PROD-01
- ✓ Error: Invalid credentials
- ✓ Duration: Failed quickly (45 sec = didn't even try to aggregate)

**Step 2: Check VA Logs (Detailed Information)**

**SSH to VA:**
```bash
ssh sailpoint@va-prod-01
```

**Check connector log for exact timeframe:**
```bash
# View logs from 2 AM
grep "2024-02-13 02:0" /opt/sailpoint/logs/connector.log | grep -i "active directory"
```

**Output:**
```
2024-02-13 02:00:00 INFO  [Connector] Account aggregation started: Active Directory
2024-02-13 02:00:01 INFO  [AD-Connector] Connecting to dc01.company.com:389
2024-02-13 02:00:02 INFO  [AD-Connector] Connection to DC successful
2024-02-13 02:00:03 INFO  [AD-Connector] Attempting LDAP bind
2024-02-13 02:00:03 INFO  [AD-Connector] Bind DN: svc_sailpoint@company.com
2024-02-13 02:00:04 ERROR [AD-Connector] LDAP bind failed
2024-02-13 02:00:04 ERROR [AD-Connector] Error code: 49 (LDAP_INVALID_CREDENTIALS)
2024-02-13 02:00:04 ERROR [AD-Connector] Error message: 80090308: LdapErr: DSID-0C09044E, comment: AcceptSecurityContext error, data 52e, v3839
2024-02-13 02:00:05 ERROR [Connector] Authentication failed for Active Directory
2024-02-13 02:00:05 ERROR [Connector] Aggregation aborted: Cannot proceed without authentication
2024-02-13 02:00:05 INFO  [CCG] Reporting failure to ISC
```

**Get context around error:**
```bash
# Show 10 lines before and after the error
grep -A 10 -B 10 "LDAP bind failed" /opt/sailpoint/logs/connector.log | grep "2024-02-13 02:"
```

**Detailed findings:**
- ✓ Connected to DC successfully (network OK)
- ✓ LDAP bind failed (authentication problem)
- ✓ Account: svc_sailpoint@company.com
- ✓ Error code: 49 (LDAP_INVALID_CREDENTIALS)
- ✓ AD error data: 52e (invalid credentials)

**Step 3: Verify Current Configuration**

**Check ISC source configuration:**
```
Admin → Connections → Sources → Active Directory → Configuration

Service Account:
  Username: svc_sailpoint@company.com
  Password: ******** (last updated: 2024-01-15)
```

**Check VA can reach DC:**
```bash
# Test connectivity
ping dc01.company.com

# Test LDAP port
telnet dc01.company.com 389
# or
nc -zv dc01.company.com 389
```

**Output:**
```
dc01.company.com [10.10.1.50] 389 (ldap) open
```

**Network is fine - DC is reachable**

**Step 4: Test Credentials Manually**

**From VA, test LDAP bind with service account:**
```bash
# Test authentication (will prompt for password)
ldapsearch -x -H ldap://dc01.company.com -D "svc_sailpoint@company.com" -W -b "DC=company,DC=com" "(objectClass=user)" sAMAccountName | head -5
```

**If password entered correctly but fails:**
```
ldap_bind: Invalid credentials (49)
```

**This confirms: Password is wrong or account locked**

**Step 5: Check Active Directory (Source System)**

**On Domain Controller, check service account:**
```powershell
# Check if account exists and is enabled
Get-ADUser -Identity svc_sailpoint -Properties *

Output:
  Enabled: True
  LockedOut: False
  PasswordExpired: True ← HERE'S THE PROBLEM!
  PasswordLastSet: 2023-11-15 (90 days ago)
```

**Root Cause Found: Password expired!**

**Step 6: Resolution**

**Fix the issue:**
```powershell
# On DC, reset password
Set-ADAccountPassword -Identity svc_sailpoint -Reset

# Set password to never expire (for service accounts)
Set-ADUser -Identity svc_sailpoint -PasswordNeverExpires $true
```

**Update password in ISC:**
```
1. Admin → Connections → Sources → Active Directory
2. Click "Edit"
3. Update password field
4. Click "Test Connection" 
   Result: ✓ Connection successful
5. Save configuration
```

**Step 7: Verify Fix**

**Trigger manual aggregation:**
```
1. Admin → Connections → Sources → Active Directory
2. Click "Aggregate Now"
3. Select "Account Aggregation"
4. Monitor progress
```

**Check logs:**
```bash
# Watch real-time
tail -f /opt/sailpoint/logs/connector.log
```

**Output:**
```
2024-02-13 14:30:00 INFO  [Connector] Account aggregation started: Active Directory
2024-02-13 14:30:01 INFO  [AD-Connector] Connecting to dc01.company.com:389
2024-02-13 14:30:02 INFO  [AD-Connector] Connection successful
2024-02-13 14:30:03 INFO  [AD-Connector] LDAP bind successful ✓
2024-02-13 14:30:04 INFO  [AD-Connector] Executing search
2024-02-13 14:32:45 INFO  [AD-Connector] Retrieved 10,247 accounts
2024-02-13 14:35:42 INFO  [Connector] Aggregation completed successfully ✓
```

**Verify in ISC:**
```
Active Directory:
  Status: ● Healthy
  Last Aggregation: 2024-02-13 14:35:42 (just now)
  Accounts: 10,247 ✓
```

**Step 8: Prevention**

**Implement preventive measures:**
```
1. Set password to never expire (already done)
   
2. Create alert for expiring passwords:
   - Monitor service account passwords
   - Alert 30 days before expiration
   
3. Document the service account:
   - Purpose: SailPoint ISC aggregation
   - Owner: IAM Team
   - Password location: Password vault
   - Password policy: Never expires
   
4. Regular password rotation:
   - Rotate every 180 days (even if never expires)
   - Test before updating in ISC
   - Update ISC immediately after rotation
   
5. Create runbook:
   - Document this issue and resolution
   - Add to troubleshooting guide
   - Train team on password management
```

**Step 9: Document**

**Create incident report:**
```
Incident: AD Aggregation Failure
Date: 2024-02-13
Duration: 02:00 - 14:35 (12 hours 35 minutes)

Root Cause:
  Service account password expired in AD

Impact:
  - No AD aggregations for 12+ hours
  - Account changes not detected
  - New hires not correlated
  - Reports potentially stale

Resolution:
  1. Identified password expiration via logs
  2. Reset password in AD
  3. Set to never expire
  4. Updated password in ISC
  5. Verified aggregation successful

Prevention:
  - Password set to never expire
  - Monitoring implemented
  - Documentation updated
  - Team trained

Ticket: INC-12345
Resolved By: IAM Team
```

**Summary of Investigation Process:**
```
1. ISC UI: Quick identification (authentication error)
2. VA Logs: Detailed error (LDAP bind failed, error 49)
3. Network Test: Confirmed connectivity OK
4. Manual Test: Confirmed credentials invalid
5. AD Check: Found password expired
6. Resolution: Reset password, update ISC
7. Verification: Aggregation successful
8. Prevention: Measures implemented
9. Documentation: Incident recorded

Time to Resolution: 35 minutes (from start of investigation)
```

**Exam Tip:** Always follow logs from ISC → VA → Source to pinpoint root cause. Check credentials expiration for authentication failures.
</details>

---

# 3.4.2 Common Source Issues and Resolutions

**Learning Objective:** Identify and resolve frequent source connectivity and aggregation problems.

---

## Connection Issues

### **Issue 1: Connection Timeout**

**Symptoms:**
```
Error: Connection timeout to source
Duration: Operation times out after 30-60 seconds
Status: Aggregation or provisioning fails immediately
```

**Common Causes:**

**1. Source System Down**
```
Problem: Domain controller offline, database server down
Test: ping <hostname>
Solution: Start the source system or fail over to backup
```

**2. Network Connectivity**
```
Problem: Network path broken, routing issues
Test: 
  ping <hostname>
  traceroute <hostname>
  telnet <hostname> <port>
Solution: Fix network connectivity, check routing
```

**3. Firewall Blocking**
```
Problem: Firewall blocking required ports
Test: telnet <hostname> <port>
Common ports:
  - LDAP: 389, 636
  - SQL Server: 1433
  - Oracle: 1521
  - MySQL: 3306
Solution: Open firewall ports, update rules
```

**4. VA Network Issues**
```
Problem: VA network interface down, DNS not working
Test:
  ip addr (check interface status)
  nslookup <hostname> (test DNS)
Solution: Fix VA network configuration
```

**Resolution Steps:**
```
1. Verify source system online
   ping dc01.company.com
   
2. Test port connectivity
   telnet dc01.company.com 389
   
3. Check from VA perspective
   SSH to VA, test from there
   
4. Review firewall rules
   Check both VA and source firewalls
   
5. Check DNS resolution
   nslookup dc01.company.com
   
6. Test with IP address
   If DNS fails, try IP directly
```

**Exam Tip:** Connection timeout = network/connectivity issue. Test ping and port connectivity first.

---

### **Issue 2: Connection Refused**

**Symptoms:**
```
Error: Connection refused
Status: Immediate failure (no timeout)
```

**Common Causes:**

**1. Service Not Running**
```
Problem: LDAP service stopped, database not started
Check (on source):
  - Windows: Services console
  - Linux: systemctl status <service>
Solution: Start the service
```

**2. Wrong Port**
```
Problem: Connecting to port 389 but service on 636
Check: Source configuration, service listening ports
Solution: Update ISC configuration with correct port
```

**3. Binding to Localhost Only**
```
Problem: Service listening on 127.0.0.1, not network interface
Check: Service configuration
Solution: Configure service to listen on network interface
```

**Resolution:**
```
1. Verify service running on source
   
2. Check port service is listening on
   netstat -an | grep <port>
   
3. Verify firewall allows connection
   
4. Update ISC configuration if needed
```

---

## Authentication Issues

### **Issue 3: Invalid Credentials**

**Symptoms:**
```
Error: LDAP bind failed: Invalid credentials
Error: SQL authentication failed
Status: Connection succeeds, authentication fails
```

**Common Causes:**

**1. Wrong Password**
```
Problem: Password changed in source, not updated in ISC
Test: Manual login with same credentials
Solution: Update password in ISC
```

**2. Password Expired**
```
Problem: Service account password expired
Check (AD): Get-ADUser -Identity svc_account -Properties PasswordExpired
Solution: Reset password, set to never expire
```

**3. Account Locked**
```
Problem: Too many failed login attempts
Check (AD): Get-ADUser -Identity svc_account -Properties LockedOut
Solution: Unlock account, fix underlying issue
```

**4. Account Disabled**
```
Problem: Service account disabled by admin
Check (AD): Get-ADUser -Identity svc_account -Properties Enabled
Solution: Enable account
```

**5. Wrong Username Format**
```
Problem: Using wrong format (UPN vs DN vs sAMAccountName)
Examples:
  ✓ svc_sailpoint@company.com (UPN)
  ✓ CN=svc_sailpoint,OU=ServiceAccounts,DC=company,DC=com (DN)
  ✗ company\svc_sailpoint (may not work)
Solution: Use correct format for connector type
```

**Resolution Steps:**
```
1. Test credentials manually
   ldapsearch (for LDAP/AD)
   sqlplus (for Oracle)
   mysql (for MySQL)
   
2. Check account status in source
   Expired? Locked? Disabled?
   
3. Verify username format correct
   
4. Update credentials in ISC if changed
   
5. Set password to never expire (service accounts)
```

---

### **Issue 4: Insufficient Permissions**

**Symptoms:**
```
Error: Access denied
Error: Insufficient permissions to read OU
Error: SELECT permission denied on table
```

**Common Causes:**

**1. Missing Read Permissions**
```
Problem: Service account can't read required OUs/tables
Test: Manual query with service account
Solution: Grant read permissions
```

**2. Missing Write Permissions (Provisioning)**
```
Problem: Can read but can't create/modify accounts
Test: Try manual account creation with service account
Solution: Grant write permissions
```

**3. Wrong OU Permissions**
```
Problem: Permissions on wrong OU
Example: Permission on OU=Users, but querying OU=Contractors
Solution: Grant permissions on correct OUs
```

**Resolution (Active Directory):**
```powershell
# Grant read permissions to service account
$ServiceAccount = Get-ADUser svc_sailpoint
$OU = "OU=Users,DC=company,DC=com"

# Delegate control wizard or:
dsacls $OU /G "$($ServiceAccount.DistinguishedName):GR"
```

**Resolution (Database):**
```sql
-- Oracle
GRANT SELECT ON hr_schema.employees TO sailpoint_svc;

-- SQL Server
GRANT SELECT ON hr.dbo.employees TO sailpoint_svc;

-- MySQL
GRANT SELECT ON hr.employees TO 'sailpoint_svc'@'%';
```

---

## Aggregation Issues

### **Issue 5: No Accounts Returned**

**Symptoms:**
```
Status: Aggregation completes successfully
Accounts: 0 (but should be thousands)
```

**Common Causes:**

**1. Wrong Base DN / Search Base**
```
Problem: Searching in wrong location
Example: Base DN set to OU=IT but users in OU=Users
Check: Review base DN configuration
Solution: Update to correct base DN
```

**2. Overly Restrictive Filter**
```
Problem: LDAP filter or SQL WHERE clause too restrictive
Example: 
  Filter: (&(objectClass=user)(department=NonexistentDept))
Check: Review filter/query
Solution: Broaden filter or fix logic
```

**3. Empty OU/Table**
```
Problem: Actually no data in that location
Check: Manual query
Solution: Update configuration or populate data
```

**4. Permission Scope Too Narrow**
```
Problem: Can query but no results due to permissions
Check: Service account permissions
Solution: Grant broader permissions
```

**Resolution:**
```
1. Run manual query with same parameters
   ldapsearch -x -H ldap://dc01.company.com -D "svc@domain" -W -b "OU=Users,DC=company,DC=com" "(objectClass=user)"
   
2. Start with broad filter, narrow down
   (&(objectClass=user)) - Get all users
   Then add restrictions one at a time
   
3. Check if data exists in source
   
4. Review permission scope
```

---

### **Issue 6: Slow Aggregation**

**Symptoms:**
```
Status: Aggregation succeeds but takes very long
Previous: 10 minutes
Current: 2 hours
```

**Common Causes:**

**1. Source Performance**
```
Problem: Source system overloaded (high CPU, memory)
Check: Resource usage on source
Solution: 
  - Run during off-peak hours
  - Optimize source system
  - Add resources to source
```

**2. Network Latency**
```
Problem: Slow network between VA and source
Check: ping times, network bandwidth
Solution:
  - Move VA closer to source (network)
  - Upgrade network connection
  - Optimize data transfer
```

**3. Too Many Attributes**
```
Problem: Fetching unnecessary attributes
Example: Fetching 50 attributes when only need 10
Solution: Reduce attributes in schema configuration
```

**4. Inefficient Query**
```
Problem: Query not optimized, no indexes
Check: Query execution plan
Solution:
  - Add database indexes
  - Optimize LDAP filters
  - Use more specific queries
```

**5. VA Undersized**
```
Problem: VA doesn't have enough resources
Check: VA CPU/memory during aggregation
Solution:
  - Increase VA resources
  - Add additional VA
  - Optimize aggregation schedule
```

**Resolution:**
```
1. Identify bottleneck (source, network, or VA)
   Monitor during aggregation
   
2. Optimize query
   - Fewer attributes
   - Better filters
   - Add indexes
   
3. Use delta aggregation
   Instead of full every time
   
4. Run during off-peak hours
   
5. Scale resources (source or VA)
```

---

### **Issue 7: Duplicate Accounts**

**Symptoms:**
```
Status: Aggregation finds same account twice
Result: Two records for same person
```

**Common Causes:**

**1. Multiple OUs Searched**
```
Problem: Same user in multiple OUs
Example: OU=Users and OU=Contractors both searched
Solution: Exclude duplicates or search one OU only
```

**2. Correlation Issues**
```
Problem: Same account correlated to two identities
Check: Correlation rules
Solution: Fix correlation logic
```

**3. Source Has Duplicates**
```
Problem: Actual duplicate accounts in source
Check: Manual query for duplicates
Solution: Clean up source system
```

**Resolution:**
```
1. Check if duplicates in source
   SELECT sAMAccountName, COUNT(*)
   FROM accounts
   GROUP BY sAMAccountName
   HAVING COUNT(*) > 1
   
2. Review base DN/search scope
   
3. Dedup logic in aggregation config
```

---

## Provisioning Issues

### **Issue 8: Account Creation Fails**

**Symptoms:**
```
Error: Failed to create account
Status: Provisioning fails
```

**Common Causes:**

**1. Account Already Exists**
```
Error: LDAP_ENTRY_ALREADY_EXISTS
Problem: Account exists in source
Solution:
  - Correlate existing account
  - Delete ghost account
  - Use different username
```

**2. Invalid Attributes**
```
Error: LDAP_INVALID_ATTRIBUTE_VALUE
Problem: Attribute doesn't meet schema requirements
Example: Phone number with letters
Solution: Fix attribute values or mapping
```

**3. Required Attribute Missing**
```
Error: LDAP_OBJECT_CLASS_VIOLATION
Problem: Missing required field
Solution: Provide required attributes in provisioning
```

**4. OU Doesn't Exist**
```
Error: LDAP_NO_SUCH_OBJECT
Problem: Target OU not found
Solution: Create OU or fix path
```

**5. Insufficient Permissions**
```
Error: LDAP_INSUFFICIENT_ACCESS_RIGHTS
Problem: Service account can't create in target OU
Solution: Grant create permissions
```

**Resolution:**
```
1. Check error code for specific issue
   
2. For ENTRY_EXISTS:
   - Verify account doesn't exist
   - Correlate if it does
   
3. For INVALID_ATTRIBUTE:
   - Review attribute values
   - Fix mapping or transforms
   
4. For MISSING_REQUIRED:
   - Check source schema requirements
   - Provide required attributes
   
5. For PERMISSION_DENIED:
   - Verify service account permissions
   - Grant necessary rights
```

---

## Data Quality Issues

### **Issue 9: Missing Attributes**

**Symptoms:**
```
Status: Aggregation succeeds
Problem: Some attributes empty/null for all accounts
```

**Common Causes:**

**1. Attribute Not in Schema**
```
Problem: Attribute not selected in connector
Solution: Add attribute to schema mapping
```

**2. Attribute Name Wrong**
```
Problem: Trying to fetch "email" but AD uses "mail"
Solution: Use correct attribute name
```

**3. Source Doesn't Have Data**
```
Problem: Attribute empty in source system
Check: Manual query
Solution: Populate data in source
```

**Resolution:**
```
1. Verify attribute name correct for source type
   
2. Check schema configuration in ISC
   
3. Test manual query to see if attribute populated
   
4. Update schema mapping if needed
```

---

## Quick Troubleshooting Guide

### **Systematic Approach**
```
1. IDENTIFY the symptom
   - What's failing? (aggregation, provisioning, connection)
   - When did it start?
   - What changed recently?

2. CHECK the obvious
   - Source online?
   - VA running?
   - Network connected?
   - Credentials valid?

3. READ the logs
   - ISC UI activity
   - VA connector.log
   - Source system logs

4. TEST components
   - Network connectivity (ping, telnet)
   - Authentication (manual login)
   - Queries (manual LDAP/SQL)

5. ISOLATE the cause
   - Source issue?
   - Network issue?
   - VA issue?
   - Configuration issue?

6. FIX and verify
   - Apply solution
   - Test immediately
   - Monitor next scheduled run
```

---

## Common Error Codes Reference

### **LDAP Errors (Active Directory)**

| Code | Error | Meaning | Solution |
|------|-------|---------|----------|
| **49** | INVALID_CREDENTIALS | Wrong password/username | Update credentials |
| **50** | INSUFFICIENT_ACCESS | Missing permissions | Grant permissions |
| **32** | NO_SUCH_OBJECT | Object/OU not found | Fix DN/path |
| **68** | ENTRY_ALREADY_EXISTS | Account exists | Correlate or use different name |
| **19** | CONSTRAINT_VIOLATION | Schema validation failed | Fix attribute values |
| **21** | INVALID_ATTRIBUTE_SYNTAX | Wrong data type | Fix attribute format |

### **Database Errors**

| Code | Error | Meaning | Solution |
|------|-------|---------|----------|
| **1045** | Access denied (MySQL) | Authentication failed | Check username/password |
| **2002** | Can't connect (MySQL) | Server unreachable | Check network/service |
| **ORA-01017** | Invalid username/password (Oracle) | Authentication failed | Update credentials |
| **ORA-12170** | TNS timeout (Oracle) | Network/connectivity | Check network |
| **18456** | Login failed (SQL Server) | Authentication failed | Check credentials |

---

## Practice Exercise

**Question:** Aggregation was working fine yesterday but failed today with "LDAP_INSUFFICIENT_ACCESS_RIGHTS". Nothing changed in ISC configuration. What would you check?

<details>
<summary>Answer</summary>

**Investigation Process:**

**Given Information:**
- Yesterday: Working fine ✓
- Today: LDAP_INSUFFICIENT_ACCESS_RIGHTS ✗
- ISC config: No changes

**Key Insight:** If ISC didn't change, something changed in the source (AD) or with the service account.

**Step 1: Confirm the Error**

**Check ISC:**
```
Admin → Connections → Sources → Active Directory → Activity

Last Failed Aggregation:
  Error: LDAP_INSUFFICIENT_ACCESS_RIGHTS
  Error Code: 50
  Message: "Insufficient permissions to read OU=Users,DC=company,DC=com"
```

**Check VA logs:**
```bash
grep "2024-02-13 02:" /opt/sailpoint/logs/connector.log | grep -i "active directory"

Output:
2024-02-13 02:00:00 INFO  Aggregation started: Active Directory
2024-02-13 02:00:01 INFO  Connecting to dc01.company.com:389
2024-02-13 02:00:02 INFO  Connection successful
2024-02-13 02:00:03 INFO  LDAP bind successful
2024-02-13 02:00:04 INFO  Executing LDAP search
2024-02-13 02:00:05 ERROR LDAP Error: 50 - INSUFFICIENT_ACCESS_RIGHTS
2024-02-13 02:00:05 ERROR Cannot read OU=Users,DC=company,DC=com
2024-02-13 02:00:06 ERROR Aggregation failed
```

**Analysis:**
- ✓ Connection successful (network OK)
- ✓ LDAP bind successful (credentials OK)
- ✗ LDAP search failed (permissions problem)

**Step 2: What Could Have Changed in AD?**

**Possible causes:**
```
1. Service account permissions removed/changed
2. OU permissions changed
3. Group Policy changed permissions
4. Service account moved to different OU (inherited permissions changed)
5. Security group membership changed
```

**Step 3: Check Service Account Status**

**On Domain Controller:**
```powershell
# Check if account exists and enabled
Get-ADUser -Identity svc_sailpoint -Properties *

Output:
  DistinguishedName: CN=svc_sailpoint,OU=ServiceAccounts,DC=company,DC=com
  Enabled: True ✓
  PasswordExpired: False ✓
  LockedOut: False ✓
  MemberOf: CN=ServiceAccounts-Group,OU=Groups,DC=company,DC=com
```

**Account is fine - Not disabled, locked, or expired**

**Step 4: Check Account Permissions on OU**

**Check current permissions:**
```powershell
# View permissions on OU=Users
$OU = "OU=Users,DC=company,DC=com"
$ServiceAccount = "CN=svc_sailpoint,OU=ServiceAccounts,DC=company,DC=com"

# Check effective permissions
dsacls $OU
```

**Look for svc_sailpoint in output - NOT FOUND!**

**Step 5: Check Change History**

**Review AD change audit logs:**
```powershell
# Check recent changes to OU permissions
Get-EventLog -LogName Security -InstanceId 4670 -Newest 100 | 
  Where-Object {$_.Message -like "*OU=Users*"}

Output shows:
  Date: 2024-02-12 18:30:00
  User: DOMAIN\john.admin
  Action: Permissions modified on OU=Users
  Details: Removed delegated permissions for ServiceAccounts-Group
```

**ROOT CAUSE FOUND: Admin removed permissions yesterday evening**

**Step 6: Contact Admin**

**Contact john.admin:**
```
Question: "Did you change permissions on OU=Users yesterday?"

Answer: "Yes, we were doing security cleanup and removed some delegated permissions. I didn't realize SailPoint used that group."
```

**Step 7: Resolution**

**Re-grant permissions:**
```powershell
# Method 1: Delegate control wizard (GUI)
# Active Directory Users and Computers
# Right-click OU=Users → Delegate Control
# Add ServiceAccounts-Group
# Grant: Read all properties

# Method 2: PowerShell
$OU = "OU=Users,DC=company,DC=com"
$ServiceGroup = "CN=ServiceAccounts-Group,OU=Groups,DC=company,DC=com"

# Grant read permissions
dsacls $OU /G "$ServiceGroup:GR"

# Verify
dsacls $OU | Select-String "ServiceAccounts-Group"
```

**Verify fix:**
```
Output: ServiceAccounts-Group has read permissions ✓
```

**Step 8: Test in ISC**

**Trigger manual aggregation:**
```
Admin → Connections → Sources → Active Directory → Aggregate Now
```

**Monitor VA logs:**
```bash
tail -f /opt/sailpoint/logs/connector.log
```

**Output:**
```
2024-02-13 15:30:00 INFO  Aggregation started
2024-02-13 15:30:01 INFO  Connection successful
2024-02-13 15:30:02 INFO  LDAP bind successful
2024-02-13 15:30:03 INFO  LDAP search successful ✓
2024-02-13 15:32:45 INFO  Retrieved 10,247 accounts ✓
2024-02-13 15:35:42 INFO  Aggregation completed successfully ✓
```

**Success!**

**Step 9: Prevention**

**Preventive measures:**
```
1. Document service account dependencies
   - Create documentation: "OU=Users requires read by ServiceAccounts-Group"
   - Share with AD admin team
   
2. Add monitoring
   - Alert if permissions change on critical OUs
   - Monitor daily aggregation success
   
3. Implement change control
   - Require approval for permission changes
   - Test in non-prod first
   
4. Create runbook
   - Document this issue
   - Add to troubleshooting guide
   - Include how to re-grant permissions
   
5. Regular permission audits
   - Verify service account permissions quarterly
   - Document expected permissions
```

**Summary:**
```
Issue: LDAP_INSUFFICIENT_ACCESS_RIGHTS
Root Cause: Admin removed delegated permissions on OU
Detection: Connector logs showed permission error after successful bind
Resolution: Re-granted read permissions to service account group
Time to Resolution: 45 minutes
Prevention: Documentation, monitoring, change control
```

**Exam Tip:** INSUFFICIENT_ACCESS_RIGHTS after successful bind = permissions removed/changed on target OU. Check AD audit logs for recent permission changes.
</details>

---

## Section Summary

**Core Concepts for Certification:**

✅ **Error Investigation:**
- Start with ISC UI for quick assessment
- Check VA connector.log for detailed errors
- Correlate across ISC → VA → Source
- Follow systematic troubleshooting process

✅ **Common Issues:**
- **Connection timeout**: Network/firewall/source down
- **Invalid credentials**: Password expired/changed/wrong
- **Insufficient permissions**: Service account permissions removed
- **No accounts returned**: Wrong base DN or overly restrictive filter
- **Slow aggregation**: Source performance, network latency, inefficient queries

✅ **Error Codes:**
- **LDAP 49**: Invalid credentials
- **LDAP 50**: Insufficient permissions
- **LDAP 68**: Entry already exists
- **Connection timeout**: Network/connectivity
- **Connection refused**: Service not running

✅ **Resolution Process:**
1. Identify symptom (what's failing)
2. Check obvious causes (source up, credentials valid)
3. Read logs (ISC UI, VA logs)
4. Test components (network, authentication, queries)
5. Isolate cause (source, network, VA, config)
6. Fix and verify

✅ **Best Practices:**
- Service account passwords: Set to never expire
- Monitor permissions: Alert on changes
- Document dependencies: What needs what permissions
- Test manually: Verify credentials and queries work
- Preserve logs: Keep for troubleshooting history

**Exam Focus Areas:**
- Know where to find errors (ISC UI, VA logs)
- Understand common error codes and meanings
- Systematic troubleshooting approach
- Connection vs authentication vs permission issues
- How to test components individually

---

**End of Section 3: Sources (Data Integration)**

**Completed Sections:**
- ✅ 3.1: Source Fundamentals
- ✅ 3.2: Aggregation Process
- ✅ 3.3: Account Correlation and Management
- ✅ 3.4: Source Troubleshooting

**Next:** Section 4: Identity and Lifecycle Management

---

**Study Tips:**
- Practice reading logs to identify errors quickly
- Memorize common LDAP error codes (49, 50, 68)
- Always test network connectivity first (ping, telnet)
- Remember: Successful bind + permission error = OU permissions issue
- Keep systematic approach: ISC → VA → Source
