# Section 3.2: Aggregation Process

**Master how ISC gathers identity and access data from sources through aggregation**

---

## Table of Contents

- [3.2.1 Types of Aggregation (Entitlement, Account)](#321-types-of-aggregation-entitlement-account)
- [3.2.2 Aggregation Scheduling and Execution](#322-aggregation-scheduling-and-execution)
- [3.2.3 Obtaining Account and Entitlement Information](#323-obtaining-account-and-entitlement-information)

---

# 3.2.1 Types of Aggregation (Entitlement, Account)

**Learning Objective:** Understand the different types of aggregation and what data each type collects.

---

## What is Aggregation?

**Aggregation** = The process of reading data from a source into ISC

**Think of it as:**
- Data import
- Synchronization
- Pulling data from source to ISC
- Keeping ISC up-to-date with source changes

**Simple flow:**
```
Source (has data) → Aggregation → ISC (receives data)
```

**Exam Tip:** Aggregation is READ-ONLY. It doesn't change data in the source, only reads it.

---

## Two Main Aggregation Types

### **Overview**

| Type | What It Gets | Example | Frequency |
|------|-------------|---------|-----------|
| **Account Aggregation** | User accounts and attributes | Get all AD users | Daily |
| **Entitlement Aggregation** | Groups, roles, permissions | Get all AD groups | Weekly |

**Both are needed for complete picture:**
```
Account Aggregation:
  "Who has accounts in this system?"
  
Entitlement Aggregation:
  "What access is available in this system?"
```

---

## Account Aggregation

### **Purpose**

**Collects:**
- Individual user accounts
- Account attributes (username, email, status)
- Account status (enabled/disabled)
- Last login information
- Account metadata

**Example - Active Directory:**
```
Account Aggregation Returns:

Account 1:
  sAMAccountName: jsmith
  displayName: John Smith
  mail: john.smith@company.com
  department: IT
  enabled: True
  lastLogon: 2024-02-13

Account 2:
  sAMAccountName: jdoe
  displayName: Jane Doe
  mail: jane.doe@company.com
  department: HR
  enabled: True
  lastLogon: 2024-02-12

... (continues for all user accounts)
```

### **What ISC Does with Account Data**

**1. Correlation**
```
ISC tries to match account to identity:

Account jsmith → Correlates to → Identity: John Smith
  Based on: email match (john.smith@company.com)
  
Account jdoe → Correlates to → Identity: Jane Doe
  Based on: email match (jane.doe@company.com)

Account orphan123 → No match → Flagged as uncorrelated
  No identity with matching email
```

**2. Updates Identity Records**
```
Identity: John Smith

Accounts:
  - Active Directory: jsmith
  - Email: john.smith@company.com
  - Salesforce: john.smith@company.com
  
Account Details Updated:
  - Last login dates
  - Account status (enabled/disabled)
  - Attribute changes
```

**3. Detects Changes**
```
Previous Aggregation: Account jsmith enabled
Current Aggregation: Account jsmith disabled

ISC detects:
  - Account was disabled
  - May trigger workflow
  - May alert admin
  - Updates identity record
```

### **Account Aggregation Types**

**Full Aggregation:**
```
What: Get ALL accounts from source
When: First aggregation, periodic full sync
Duration: Longer (processes all accounts)

Example:
  - Read all 10,000 AD users
  - Process each one
  - Update ISC with all data
```

**Delta (Incremental) Aggregation:**
```
What: Get ONLY changed accounts since last aggregation
When: Regular scheduled aggregations
Duration: Faster (processes only changes)

Example:
  - Last aggregation: Yesterday 2 AM
  - Delta aggregation: Today 2 AM
  - Get only accounts changed in last 24 hours
  - Process 50 changed accounts instead of 10,000
```

**Exam Tip:** Delta aggregation is faster and more efficient than full aggregation.

---

## Entitlement Aggregation

### **Purpose**

**Collects:**
- Groups
- Roles
- Permissions
- Access rights
- Organizational units
- Resource pools

**Example - Active Directory:**
```
Entitlement Aggregation Returns:

Group 1:
  CN: VPN-Users
  DN: CN=VPN-Users,OU=Groups,DC=company,DC=com
  Description: VPN access for remote workers
  Members: 245 users
  
Group 2:
  CN: IT-All
  DN: CN=IT-All,OU=Groups,DC=company,DC=com
  Description: All IT department staff
  Members: 52 users

Group 3:
  CN: HR-Sensitive
  DN: CN=HR-Sensitive,OU=Groups,DC=company,DC=com
  Description: Access to sensitive HR data
  Members: 8 users

... (continues for all groups)
```

**Example - Salesforce:**
```
Entitlement Aggregation Returns:

Permission Set 1:
  Name: Sales Cloud User
  Description: Standard sales user permissions
  
Permission Set 2:
  Name: Marketing Cloud User
  Description: Marketing automation access

Profile 1:
  Name: Standard User
  Description: Basic platform access
```

### **What ISC Does with Entitlement Data**

**1. Catalogs Available Access**
```
ISC knows what access exists:

Active Directory:
  - 250 groups available
  - Users can request group membership
  
Salesforce:
  - 15 permission sets available
  - 8 profiles available
  - Users can request access
```

**2. Links to Account Memberships**
```
During Account Aggregation, ISC also gets:
  Account: jsmith
  Member of groups:
    - VPN-Users
    - IT-All
    - Email-Users
    
ISC records:
  John Smith has these entitlements via AD account
```

**3. Enables Access Requests**
```
In ISC access request catalog:

Available from Active Directory:
  - VPN-Users group (requestable)
  - IT-All group (auto-assigned to IT)
  - HR-Sensitive group (requires approval)
  
User can request:
  "I need VPN access"
  → ISC grants VPN-Users group
```

**4. Supports Access Reviews**
```
Certification Campaign:
  "Review all VPN-Users group members"
  
ISC knows:
  - VPN-Users has 245 members
  - Each member listed for review
  - Managers approve/revoke
```

### **Entitlement vs Account Aggregation**

**Key Differences:**

| Aspect | Account Aggregation | Entitlement Aggregation |
|--------|-------------------|------------------------|
| **What** | Individual accounts | Groups, roles, permissions |
| **Why** | Know who has accounts | Know what access exists |
| **Frequency** | Daily/Hourly | Weekly/Monthly |
| **Volume** | High (thousands of users) | Lower (hundreds of groups) |
| **Changes** | Frequent (users come/go) | Infrequent (groups stable) |
| **Updates** | Identity records | Entitlement catalog |

**Example - Complete Picture:**
```
Account Aggregation tells ISC:
  - jsmith account exists
  - jsmith is enabled
  - jsmith last logged in yesterday

Entitlement Aggregation tells ISC:
  - VPN-Users group exists
  - IT-All group exists
  - HR-Sensitive group exists

Account Aggregation ALSO tells ISC:
  - jsmith is member of: VPN-Users, IT-All
  
Combined, ISC knows:
  - John Smith has AD account (jsmith)
  - John Smith has VPN access (via VPN-Users)
  - John Smith has IT access (via IT-All)
```

**Exam Tip:** Account aggregation gets accounts AND their entitlement memberships. Entitlement aggregation gets the available entitlements themselves.

---

## Aggregation Process Flow

### **Typical Aggregation Sequence**

**Step-by-step:**
```
1. Scheduled Time Arrives
   Example: 2:00 AM daily
   
2. ISC Sends Command to Source/VA
   "Run account aggregation for Active Directory"
   
3. Source/VA Executes Query
   LDAP query: Get all users in domain
   
4. Source Returns Data
   10,000 user accounts with attributes
   
5. VA Sends to ISC (if using VA)
   Data compressed and encrypted
   Sent in batches (e.g., 1,000 at a time)
   
6. ISC Processes Each Account
   - Correlate to identity (match)
   - Update identity attributes
   - Detect changes
   - Flag uncorrelated accounts
   
7. ISC Updates Records
   - Identity records updated
   - Account records updated
   - Changes logged
   
8. Aggregation Completes
   Status: Success
   Accounts processed: 10,000
   Duration: 6 minutes
```

---

## Practice Exercise

**Question:** What's the difference between account aggregation and entitlement aggregation? When would you run each?

<details>
<summary>Answer</summary>

**Account Aggregation:**

**What it does:**
```
Gets individual user accounts from source

Example - Active Directory:
  - jsmith (account)
  - jdoe (account)
  - bwilson (account)
  
With attributes:
  - Name, email, department
  - Enabled/disabled status
  - Group memberships
  - Last login
```

**When to run:**
```
Frequency: Daily or more often

Why frequently:
  - Users join/leave frequently
  - Account status changes daily
  - Need current membership data
  - Support timely provisioning/deprovisioning
  
Typical schedule:
  - Daily at 2 AM (standard)
  - Every 6 hours (active environments)
  - Hourly (real-time needs)
```

**Entitlement Aggregation:**

**What it does:**
```
Gets available groups/roles/permissions

Example - Active Directory:
  - VPN-Users (group)
  - IT-All (group)
  - Finance-Reports (group)
  
With details:
  - Description
  - Membership count
  - Permissions granted
```

**When to run:**
```
Frequency: Weekly or monthly

Why less frequently:
  - Groups don't change often
  - New groups rarely created
  - Existing groups stable
  - Less time-sensitive
  
Typical schedule:
  - Weekly (Sunday night)
  - Monthly (first of month)
  - After major changes
```

**When to Run Both:**
```
First-Time Setup:
  1. Run entitlement aggregation first
     → Get all available groups
  2. Run account aggregation second
     → Get accounts and their group memberships
     → ISC can now correlate memberships to known groups

Regular Operations:
  - Account: Daily (keeps account data current)
  - Entitlement: Weekly (keeps group catalog current)

After Major Changes:
  - Created many new AD groups?
    → Run entitlement aggregation
  - Onboarded many new users?
    → Run account aggregation
```

**Real-World Example:**
```
Monday 2 AM:
  ✓ Account aggregation runs (scheduled daily)
    - Processes 10,000 users
    - Detects 5 new accounts
    - Detects 3 disabled accounts
    - Updates 47 changed accounts
    - Duration: 6 minutes

Sunday 3 AM:
  ✓ Entitlement aggregation runs (scheduled weekly)
    - Processes 250 groups
    - Detects 2 new groups
    - Updates 1 group description
    - Duration: 2 minutes
```

**Exam Tip:** Account aggregation = frequent (daily), Entitlement aggregation = infrequent (weekly).
</details>

---

# 3.2.2 Aggregation Scheduling and Execution

**Learning Objective:** Understand how to schedule and execute aggregations effectively.

---

## Aggregation Scheduling Options

### **Scheduled Aggregation**

**Most common method** - Runs automatically on schedule

**Configuration:**
```
Source: Active Directory
Aggregation Schedule:

Account Aggregation:
  Type: Recurring
  Frequency: Daily
  Time: 02:00 (2 AM)
  Timezone: America/New_York
  Enabled: Yes

Entitlement Aggregation:
  Type: Recurring
  Frequency: Weekly
  Day: Sunday
  Time: 03:00 (3 AM)
  Timezone: America/New_York
  Enabled: Yes
```

**Frequency Options:**
- Hourly
- Daily
- Weekly
- Monthly
- Custom (cron expression)

### **Manual (On-Demand) Aggregation**

**Triggered by administrator**

**When to use:**
- Testing new source
- After making configuration changes
- Troubleshooting issues
- Need immediate update
- Before important operation (certification, provisioning)

**How to trigger:**
```
In ISC:
  1. Admin → Connections → Sources
  2. Select source (e.g., Active Directory)
  3. Click "Aggregate Now" or similar
  4. Choose type:
     - Account Aggregation
     - Entitlement Aggregation
  5. Confirm
  6. Aggregation starts immediately
```

### **Event-Driven Aggregation**

**Triggered by events** (less common in ISC)

**Examples:**
- File appears in directory → Trigger aggregation
- Webhook received → Trigger aggregation
- After provisioning → Trigger delta aggregation

**Exam Tip:** Scheduled aggregation is most common. Manual for testing/troubleshooting.

---

## Scheduling Best Practices

### **Timing Considerations**

**Choose off-peak hours:**
```
✅ Good Times:
  - 2-6 AM (night)
  - Weekends
  - Low-activity periods

❌ Bad Times:
  - 8-5 PM (business hours)
  - During backups
  - Peak usage times
  - During other aggregations
```

**Why off-peak matters:**
```
Source Impact:
  - Aggregation queries source system
  - Can increase load on source (AD, database)
  - May slow down source for users
  
Network Impact:
  - Transfers large amounts of data
  - Consumes bandwidth
  - Especially for large environments

VA Impact:
  - High CPU/memory during aggregation
  - Concurrent aggregations strain VA
```

### **Stagger Multiple Sources**

**Don't run all aggregations simultaneously:**
```
❌ Bad Schedule (All at 2 AM):
  02:00 - Active Directory
  02:00 - HR Database
  02:00 - LDAP
  02:00 - Email System
  
  Problem:
    - All hit VA at once
    - VA overloaded
    - Aggregations slow/fail
    - Source systems stressed

✅ Good Schedule (Staggered):
  02:00 - Active Directory (largest)
  03:00 - HR Database
  04:00 - LDAP
  05:00 - Email System
  
  Benefit:
    - One at a time
    - VA handles load
    - Sources not overwhelmed
    - Reliable completion
```

### **Full vs Delta Scheduling**

**Recommended pattern:**
```
Account Aggregation:
  Daily: Delta (incremental)
    - Fast, only changed accounts
    - Monday-Saturday
    
  Weekly: Full
    - Complete refresh
    - Sunday
    - Catches anything delta missed

Entitlement Aggregation:
  Weekly: Full
    - Groups don't change often
    - No need for delta
    - Sunday night
```

**Example Schedule:**
```
Active Directory:

Monday-Saturday 2 AM:
  Account Aggregation (Delta)
  Duration: ~3 minutes
  Processes: ~100 changed accounts

Sunday 2 AM:
  Account Aggregation (Full)
  Duration: ~8 minutes
  Processes: 10,000 accounts

Sunday 3 AM:
  Entitlement Aggregation (Full)
  Duration: ~2 minutes
  Processes: 250 groups
```

---

## Aggregation Execution Process

### **Execution Flow**
```
1. Schedule Triggers
   ↓
2. ISC Creates Aggregation Job
   Job ID: agg-20240213-020000-ad
   ↓
3. ISC Sends Command
   If cloud source: Direct to source
   If on-prem: Via VA
   ↓
4. Source Queried
   LDAP search, SQL query, API call, etc.
   ↓
5. Data Retrieved
   Accounts/entitlements returned
   ↓
6. Data Sent to ISC
   Batched, compressed, encrypted
   ↓
7. ISC Processes
   Correlation, updates, change detection
   ↓
8. Completion
   Status: Success/Failed
   Results logged
```

### **What Happens During Execution**

**Phase 1: Initialization**
```
- ISC validates source configuration
- Checks VA availability (if needed)
- Authenticates to source
- Prepares for data collection
```

**Phase 2: Data Collection**
```
- Executes query/API call
- Retrieves accounts or entitlements
- Handles pagination (large datasets)
- Manages rate limits
```

**Phase 3: Data Transfer**
```
- Batches data for efficiency
- Compresses data
- Encrypts for transmission
- Sends to ISC
```

**Phase 4: Processing**
```
For each account/entitlement:
  - Parse data
  - Apply transforms
  - Correlate (for accounts)
  - Update ISC records
  - Log changes
```

**Phase 5: Completion**
```
- Finalize aggregation
- Update statistics
- Log results
- Send notifications (if configured)
- Clean up temporary data
```

---

## Monitoring Aggregation Execution

### **Check Aggregation Status**

**In ISC UI:**
```
Location: Admin → Connections → Sources → [Source Name] → Activity

Recent Aggregations:

Date/Time           Type        Status    Accounts  Duration
2024-02-13 02:00   Account     Success   10,247    6m 23s
2024-02-12 02:00   Account     Success   10,245    6m 18s
2024-02-11 03:00   Entitlement Success   250       2m 05s
2024-02-11 02:00   Account     Success   10,243    6m 15s
2024-02-10 02:00   Account     Failed    0         0m 45s
```

**Aggregation Details:**
```
Click on aggregation to see:
  - Start time
  - End time
  - Duration
  - Records processed
  - Errors encountered
  - Changes detected (new, updated, deleted)
  - Uncorrelated accounts
```

### **In VA Logs (if using VA)**
```bash
# Check connector log
tail -f /opt/sailpoint/logs/connector.log

# Sample output:
[INFO] Account aggregation started: Active Directory
[INFO] Connecting to dc01.company.com
[INFO] Query: (&(objectClass=user)(!(objectClass=computer)))
[INFO] Retrieved 10,247 accounts
[INFO] Sending batch 1/11 to ISC (1,000 accounts)
[INFO] Sending batch 2/11 to ISC (1,000 accounts)
...
[INFO] All batches sent to ISC
[INFO] Account aggregation completed: 10,247 accounts in 6m 23s
```

---

## Aggregation Performance

### **Factors Affecting Speed**

| Factor | Impact | Example |
|--------|--------|---------|
| **Dataset Size** | Larger = slower | 10K users vs 100K users |
| **Attributes** | More attributes = slower | 5 attributes vs 50 attributes |
| **Network** | Slower network = slower | VA to source latency |
| **Source Performance** | Slow source = slow aggregation | Overloaded domain controller |
| **VA Resources** | Low resources = slower | 4 CPU vs 16 CPU |
| **Aggregation Type** | Full vs delta | Full: 10 min, Delta: 2 min |

### **Typical Performance Metrics**

**Active Directory (via VA):**
```
Small (1,000-5,000 users):
  Full Aggregation: 2-5 minutes
  Delta Aggregation: 30-60 seconds

Medium (5,000-25,000 users):
  Full Aggregation: 5-15 minutes
  Delta Aggregation: 1-3 minutes

Large (25,000-100,000 users):
  Full Aggregation: 15-60 minutes
  Delta Aggregation: 3-10 minutes

Enterprise (100,000+ users):
  Full Aggregation: 1-3 hours
  Delta Aggregation: 10-30 minutes
```

**Cloud SaaS Sources:**
```
Workday, Salesforce:
  Typically faster (optimized APIs)
  Limited by API rate limits
  10,000 users: 3-10 minutes
```

### **Optimizing Aggregation Performance**

**1. Use Delta Aggregation**
```
✅ Delta: Only changed records
   10,000 users, 50 changed
   Process: 50 records
   Time: 1 minute

❌ Full: All records every time
   10,000 users, 50 changed
   Process: 10,000 records
   Time: 10 minutes
```

**2. Reduce Attributes**
```
✅ Only needed attributes:
   sAMAccountName, mail, displayName, department
   Faster queries, less data transfer

❌ All possible attributes:
   50+ attributes per account
   Slower queries, large data transfer
```

**3. Optimize Queries**
```
✅ Efficient filters:
   (&(objectClass=user)(!(objectClass=computer)))
   Returns only user accounts

❌ Broad searches:
   (objectClass=*)
   Returns everything, slow
```

**4. Schedule During Off-Peak**
```
✅ 2-6 AM when source is idle
   Source can dedicate resources

❌ 2 PM when source is busy
   Source slow to respond
```

**5. Adequate VA Resources**
```
✅ Properly sized VA:
   16 CPU, 32 GB RAM for 50K users
   Handles load easily

❌ Undersized VA:
   4 CPU, 8 GB RAM for 50K users
   Struggles, slow processing
```

**Exam Tip:** Delta aggregation is faster than full. Use delta daily, full weekly.

---

## Handling Aggregation Failures

### **Common Failure Reasons**

**1. Source Unavailable**
```
Error: Connection timeout to source
Cause: Source system down or unreachable
Fix: Verify source availability, check network
```

**2. Authentication Failed**
```
Error: Invalid credentials
Cause: Password changed, account locked
Fix: Update credentials in ISC
```

**3. Permission Denied**
```
Error: Access denied to query users
Cause: Service account lacks permissions
Fix: Grant necessary permissions
```

**4. Timeout**
```
Error: Operation timeout after 30 minutes
Cause: Query too slow, dataset too large
Fix: Optimize query, use delta, increase timeout
```

### **Retry Behavior**

**Automatic retry (if configured):**
```
Aggregation fails at 2:00 AM
  ↓
ISC waits 15 minutes
  ↓
Retry at 2:15 AM
  ↓
If still fails, retry at 2:30 AM
  ↓
If still fails, give up
  ↓
Send alert (if configured)
```

**Manual retry:**
```
Admin sees failed aggregation
Admin investigates cause
Admin fixes issue (e.g., updates password)
Admin triggers manual aggregation
Aggregation succeeds
```

---

## Practice Exercise

**Question:** You have 50,000 users in Active Directory. Currently running full account aggregation daily at 2 AM, taking 45 minutes. How would you optimize this?

<details>
<summary>Answer</summary>

**Current Situation:**
```
Environment: 50,000 AD users
Schedule: Daily full aggregation at 2 AM
Duration: 45 minutes
Issue: Too slow, processing all 50K users daily
```

**Optimization Strategy:**

**1. Switch to Delta Aggregation (Primary Fix)**
```
Change Schedule:
  Monday-Saturday: Delta aggregation
    - Only processes changed accounts
    - Typical daily changes: 100-500 users (1%)
    - Expected duration: 2-5 minutes
    
  Sunday: Full aggregation
    - Complete refresh weekly
    - Catches anything delta missed
    - Duration: 45 minutes (acceptable weekly)

Expected Improvement:
  - 6 days: 2-5 minutes each
  - 1 day: 45 minutes
  - Average: ~10 minutes per day vs 45 minutes
  - 78% time savings
```

**2. Reduce Attributes (if applicable)**
```
Review what attributes are needed:

Currently fetching:
  ✓ sAMAccountName (needed)
  ✓ displayName (needed)
  ✓ mail (needed)
  ✓ department (needed)
  ✓ manager (needed)
  ? thumbnailPhoto (needed?)
  ? homePhone (needed?)
  ? mobile (needed?)
  ... 20+ more attributes

Optimize:
  - Remove unnecessary attributes
  - Reduces query time
  - Reduces data transfer
  - Faster processing
```

**3. Optimize LDAP Query**
```
Current query (example):
  (objectClass=user)
  → Returns all users AND computers

Optimized query:
  (&(objectClass=user)(!(objectClass=computer)))
  → Returns only user accounts
  → Faster, less data
```

**4. Verify VA Resources**
```
Check VA specs:
  50,000 users = large environment
  
Recommended:
  - 16 CPU minimum
  - 32 GB RAM
  - Fast network to DC

If undersized:
  - Increase VA resources
  - Consider additional VA
  - Distribute load across VAs
```

**5. Check Source Performance**
```
Verify Domain Controller:
  - Is DC overloaded during aggregation?
  - Is network latency acceptable?
  - Are there replication delays?
  
If DC slow:
  - Use dedicated DC for ISC queries
  - Schedule during DC off-peak
  - Optimize AD indexes
```

**Implementation Plan:**
```
Week 1: Enable Delta Aggregation
  - Configure delta schedule
  - Test delta aggregation
  - Monitor results
  - Expected: 2-5 min daily, 45 min Sunday

Week 2: Optimize Attributes
  - Review attribute list
  - Remove unnecessary attributes
  - Test aggregation
  - Expected: Additional 10-20% improvement

Week 3: Monitor and Tune
  - Monitor delta performance
  - Verify data accuracy
  - Check for missed changes
  - Adjust as needed

Success Criteria:
  ✓ Daily aggregations: <5 minutes
  ✓ Weekly full: <45 minutes
  ✓ No missed account changes
  ✓ Reduced VA load
```

**Expected Results:**
```
Before Optimization:
  Monday: 45 min (full)
  Tuesday: 45 min (full)
  Wednesday: 45 min (full)
  Thursday: 45 min (full)
  Friday: 45 min (full)
  Saturday: 45 min (full)
  Sunday: 45 min (full)
  Total: 315 minutes/week

After Optimization:
  Monday: 3 min (delta)
  Tuesday: 3 min (delta)
  Wednesday: 3 min (delta)
  Thursday: 3 min (delta)
  Friday: 3 min (delta)
  Saturday: 3 min (delta)
  Sunday: 45 min (full)
  Total: 63 minutes/week
  
Improvement: 80% reduction in aggregation time
```

**Exam Tip:** For large environments, delta aggregation is essential. Use full aggregation weekly only.
</details>

---

# 3.2.3 Obtaining Account and Entitlement Information

**Learning Objective:** Understand what specific information ISC obtains during aggregation and how it's used.

---

## Account Information Collected

### **Standard Account Attributes**

**Core attributes every source should provide:**

| Attribute | Purpose | Example |
|-----------|---------|---------|
| **Account ID** | Unique identifier | jsmith, 12345 |
| **Account Name** | Display name | John Smith |
| **Status** | Enabled/disabled | Enabled, Active, Locked |
| **Created Date** | When account created | 2023-01-15 |
| **Modified Date** | Last change | 2024-02-10 |
| **Last Login** | Activity indicator | 2024-02-13 |

**Active Directory Example:**
```
Account Attributes Retrieved:

sAMAccountName: jsmith (Account ID)
displayName: John Smith (Account Name)
distinguishedName: CN=jsmith,OU=Users,DC=company,DC=com
mail: john.smith@company.com
department: IT
title: Systems Administrator
manager: CN=Jane Doe,OU=Users,DC=company,DC=com
memberOf: 
  - CN=VPN-Users,OU=Groups,DC=company,DC=com
  - CN=IT-All,OU=Groups,DC=company,DC=com
userAccountControl: 512 (enabled)
whenCreated: 2023-01-15T08:00:00Z
whenChanged: 2024-02-10T14:30:00Z
lastLogonTimestamp: 2024-02-13T09:15:00Z
```

**Salesforce Example:**
```
Account Attributes Retrieved:

Id: 005xx0000012345AAA (Account ID)
Username: john.smith@company.com
Email: john.smith@company.com
FirstName: John
LastName: Smith
IsActive: true
ProfileId: 00exx000000abcd
UserRoleId: 00Exx000000efgh
CreatedDate: 2023-01-15
LastLoginDate: 2024-02-13
```

### **Custom Attributes**

**Source-specific attributes:**
```
Active Directory Specific:
  - employeeID
  - homeDirectory
  - profilePath
  - scriptPath
  - telephoneNumber
  - mobile

Salesforce Specific:
  - LanguageLocaleKey
  - TimeZoneSidKey
  - UserPermissions
  - License assignments

Database Custom:
  - Cost center
  - Badge number
  - Office location
  - Custom fields
```

---

## Entitlement Information Collected

### **Group/Role Attributes**

**Standard entitlement attributes:**

| Attribute | Purpose | Example |
|-----------|---------|---------|
| **Entitlement ID** | Unique identifier | VPN-Users, Admin-Role |
| **Name** | Display name | VPN Users Group |
| **Description** | What it grants | Access to corporate VPN |
| **Type** | Kind of entitlement | Group, Role, Permission Set |
| **Members** | Who has it | Count or list |
| **Privileged** | High-risk flag | True/False |

**Active Directory Group Example:**
```
Entitlement: VPN-Users

DN: CN=VPN-Users,OU=Groups,DC=company,DC=com (ID)
cn: VPN-Users (Name)
description: Corporate VPN access for remote workers
groupType: -2147483646 (Security Group)
member: 
  - CN=jsmith,OU=Users,DC=company,DC=com
  - CN=jdoe,OU=Users,DC=company,DC=com
  ... (245 members total)
whenCreated: 2020-05-10T10:00:00Z
whenChanged: 2024-02-12T16:45:00Z
```

**Salesforce Permission Set Example:**
```
Entitlement: Sales Cloud User

Id: 0PSxx000000abcd (ID)
Name: Sales_Cloud_User
Label: Sales Cloud User
Description: Standard permissions for sales team
IsCustom: false
Type: PermissionSet
Assignments: 127 users
```

---

## How ISC Uses Aggregated Data

### **Account Data Usage**

**1. Identity Correlation**
```
Process:
  Account jsmith retrieved from AD
    ↓
  ISC searches for matching identity
    Search by: email = john.smith@company.com
    ↓
  Match found: Identity "John Smith"
    ↓
  ISC links account to identity
  
Result:
  Identity: John Smith
    Accounts:
      - AD: jsmith (linked)
```

**2. Account Status Tracking**
```
ISC monitors:
  - Is account enabled or disabled?
  - When was last login?
  - Is account locked?
  - Has account been modified?

Uses:
  - Detect inactive accounts
  - Alert on status changes
  - Compliance reporting
  - Lifecycle management
```

**3. Access Tracking**
```
From account attributes:
  memberOf: VPN-Users, IT-All, Email-Users

ISC records:
  John Smith has:
    - VPN access (via VPN-Users)
    - IT department access (via IT-All)
    - Email access (via Email-Users)

Used for:
  - Access reviews/certifications
  - Compliance reporting
  - Access request decisions
```

### **Entitlement Data Usage**

**1. Access Catalog**
```
ISC builds catalog of available access:

Active Directory offers:
  - VPN-Users (245 members, requestable)
  - IT-All (52 members, auto-assigned)
  - Finance-Reports (15 members, requires approval)
  - HR-Sensitive (8 members, high-risk)

Users can:
  - Browse catalog
  - Request access
  - See who has what
```

**2. Access Reviews**
```
Certification campaign uses entitlement data:

Review: "All VPN-Users members"
  
ISC knows:
  - VPN-Users has 245 members
  - Lists each member
  - Manager reviews each
  - Approves or revokes

Without entitlement aggregation:
  - ISC wouldn't know group exists
  - Couldn't review memberships
```

**3. Role Assignments**
```
Access Profile uses entitlements:

Access Profile: "Standard Employee"
  Includes:
    - Email-Users (AD group)
    - VPN-Users (AD group)
    - Salesforce Standard User (SF profile)

ISC knows these exist from entitlement aggregation
Can provision them when needed
```

**4. Privileged Access Detection**
```
ISC flags high-risk entitlements:

Marked as privileged:
  - Domain Admins (AD)
  - Enterprise Admins (AD)
  - Salesforce System Administrator
  - Database Admin role

Used for:
  - Extra approval requirements
  - Audit logging
  - Certification priority
  - Access analytics
```

---

## Account Correlation Process

### **What is Correlation?**

**Correlation** = Matching accounts to identities

**The challenge:**
```
ISC has identity: "John Smith"
AD has account: "jsmith"

How does ISC know they're the same person?
→ Correlation
```

### **Correlation Methods**

**1. Automatic Correlation (Rule-Based)**
```
Correlation Rule: Match on email

Account (from AD):
  sAMAccountName: jsmith
  mail: john.smith@company.com

Identity (in ISC):
  Name: John Smith
  email: john.smith@company.com

ISC compares:
  Account.mail == Identity.email
  john.smith@company.com == john.smith@company.com
  → MATCH

Result: Account correlated to identity
```

**2. Manual Correlation**
```
Admin manually links:
  1. View uncorrelated account "jsmith"
  2. Search for identity "John Smith"
  3. Click "Correlate"
  4. Account now linked to identity
```

**Common Correlation Attributes:**
- Email address (most common)
- Employee ID
- Username
- SSN or national ID (if available)
- Custom unique identifier

### **Uncorrelated Accounts**

**What are they:**
```
Accounts that don't match any identity

Example:
  Account: orphan123
  No identity with matching email/ID
  
ISC flags as: UNCORRELATED
```

**Why accounts become uncorrelated:**
- Account created manually in source (not through ISC)
- User left company (identity removed, account remains)
- Correlation rule doesn't match (typo in email)
- Service accounts (not real people)
- Shared accounts (multiple users)

**What to do:**
```
Review uncorrelated accounts:
  1. Is it a real person?
     → Correlate to existing identity
     → Or create identity if missing
     
  2. Is it a service account?
     → Mark as service account
     → Exclude from reviews
     
  3. Is it orphaned (user left)?
     → Disable or delete account
     → Document reason
```

**Exam Tip:** Correlation links accounts to identities. Email is most common correlation attribute.

---

## Delta Aggregation Details

### **How Delta Works**

**Concept:**
```
Only process accounts that changed since last aggregation

Last aggregation: Yesterday 2 AM
Current aggregation: Today 2 AM
  
Query: "Give me accounts changed in last 24 hours"
  
Result: 50 changed accounts instead of 10,000 total
```

**Delta Query Examples:**

**Active Directory:**
```
LDAP filter with timestamp:
  (&
    (objectClass=user)
    (whenChanged>=20240212020000.0Z)
  )

Returns only accounts modified since Feb 12, 2 AM
```

**SQL Database:**
```
SELECT * FROM employees
WHERE last_modified > '2024-02-12 02:00:00'

Returns only rows changed since last aggregation
```

**Salesforce (API):**
```
GET /services/data/v58.0/query
  ?q=SELECT Id, Name, Email, LastModifiedDate
     FROM User
     WHERE LastModifiedDate > 2024-02-12T02:00:00Z
```

### **Delta Aggregation Advantages**
```
✅ Faster: 
   Process 50 records vs 10,000

✅ Less load:
   Lower impact on source system

✅ Less network:
   Transfer less data

✅ More frequent:
   Can run hourly instead of daily

✅ Better VA performance:
   Lower CPU/memory usage
```

### **Delta Aggregation Limitations**
```
❌ Doesn't catch deletions well:
   Deleted accounts not "changed"
   Need periodic full to detect
   
❌ Requires source support:
   Source must track "last modified"
   Not all sources support delta
   
❌ May miss some changes:
   Clock skew issues
   Timestamp rounding
   Need periodic full to reconcile
```

**Best Practice:**
```
Use BOTH delta and full:

Monday-Saturday: Delta aggregation
  - Fast, efficient
  - Catches most changes
  
Sunday: Full aggregation
  - Complete refresh
  - Catches deletions
  - Reconciles any missed changes
```

---

## Practice Exercise

**Question:** During account aggregation from AD, you get 10,000 accounts but 150 show as "uncorrelated." What does this mean and what should you do?

<details>
<summary>Answer</summary>

**What "Uncorrelated" Means:**
```
Uncorrelated = Account exists in AD but doesn't match any identity in ISC

Status:
  ✓ Successfully aggregated: 10,000 accounts
  ✓ Correlated (matched to identity): 9,850 accounts
  ❌ Uncorrelated (no match): 150 accounts

What it looks like:
  Account: orphan123
  Status: UNCORRELATED
  No identity match found
```

**Why Accounts Are Uncorrelated:**

**1. Accounts created outside ISC:**
```
Admin manually created account in AD
Account bypassed ISC workflows
No corresponding identity exists in ISC
```

**2. Orphaned accounts (users left):**
```
User terminated
Identity disabled/removed in ISC
AD account still active
Now orphaned
```

**3. Service accounts:**
```
svc_backup
svc_monitoring
svc_application
These aren't real people → No identities
```

**4. Correlation rule doesn't match:**
```
Account email: john.smith@company.com
Identity email: jsmith@company.com (typo)
Emails don't match → No correlation
```

**5. Shared accounts:**
```
admin_shared
helpdesk_generic
Multiple people use → Hard to correlate to one identity
```

**What You Should Do:**

**Step 1: Review Uncorrelated List**
```
In ISC:
  Admin → Accounts → Filter: Uncorrelated
  Review list of 150 accounts
```

**Step 2: Categorize Each Account**
```
For each uncorrelated account, determine:

Category A: Real users (should be correlated)
  Example: jsmith exists in AD but not matched
  Action: Correlate to existing identity or create identity

Category B: Service accounts
  Example: svc_backup, svc_monitoring
  Action: Mark as service account, exclude from reviews

Category C: Orphaned (user left)
  Example: Account for ex-employee still active
  Action: Disable/delete account in AD

Category D: Shared/admin accounts
  Example: admin_shared, helpdesk_queue
  Action: Mark as shared, manage separately
```

**Step 3: Take Action**

**For Real Users (Correlate):**
```
Option 1: Manual correlation
  1. Find uncorrelated account: jsmith
  2. Search for identity: John Smith
  3. Click "Correlate Account"
  4. Account now linked

Option 2: Fix correlation rule
  If many uncorrelated due to rule:
    1. Review correlation rule
    2. Fix matching logic
    3. Re-run aggregation
    4. Accounts should correlate
```

**For Service Accounts:**
```
1. In ISC, tag account as "Service Account"
2. Create governance group: "Service Accounts"
3. Add these accounts to group
4. Exclude from:
   - Access certifications
   - Inactive account reports
   - Termination workflows
```

**For Orphaned Accounts:**
```
1. Verify user really left (check HR)
2. Disable account in AD
3. Document reason
4. Schedule for deletion (after retention)
```

**For Shared Accounts:**
```
1. Document purpose and ownership
2. Tag as "Shared Account"
3. Assign to manager/owner
4. Include in special review process
```

**Step 4: Prevent Future Uncorrelated Accounts**
```
Prevention strategies:

1. Enforce ISC provisioning:
   - All AD accounts created via ISC
   - No manual account creation
   - Exceptions require approval

2. Regular cleanup:
   - Monthly review of uncorrelated
   - Quarterly service account review
   - Disable orphaned accounts promptly

3. Improve correlation:
   - Use employee ID (more reliable than email)
   - Multiple correlation rules
   - Better data quality from HR

4. Automated categorization:
   - Auto-detect service accounts (naming pattern)
   - Auto-flag orphaned (no login 90 days + uncorrelated)
```

**Example Resolution:**
```
150 Uncorrelated Accounts Breakdown:

45 Real users (correlation rule issue):
   → Fix correlation rule (use employee ID)
   → Re-aggregate
   → Now correlated

60 Service accounts:
   → Tag as service accounts
   → Exclude from reviews
   → Documented

30 Orphaned (ex-employees):
   → Disable in AD
   → Document
   → Schedule deletion

15 Shared/admin accounts:
   → Tag appropriately
   → Assign owners
   → Special review process

Result: 
  ✓ 0 uncorrelated (all categorized)
  ✓ Proper handling for each type
  ✓ Prevention measures in place
```

**Exam Tip:** Uncorrelated accounts need investigation. Could be service accounts, orphaned accounts, or correlation rule issues.
</details>

---

## Section Summary

**Core Concepts for Certification:**

✅ **Aggregation Types:**
- **Account Aggregation**: Gets user accounts and attributes (daily)
- **Entitlement Aggregation**: Gets groups/roles/permissions (weekly)
- Both needed for complete picture

✅ **Aggregation Methods:**
- **Full**: All accounts (slower, complete)
- **Delta**: Only changed accounts (faster, efficient)
- Best practice: Delta daily, full weekly

✅ **Scheduling:**
- Schedule during off-peak hours (2-6 AM)
- Stagger multiple sources (don't all at once)
- Account aggregation: Daily or more frequent
- Entitlement aggregation: Weekly or monthly

✅ **Data Collected:**
- **Accounts**: Username, status, attributes, group memberships
- **Entitlements**: Groups, roles, permissions, descriptions
- Used for correlation, access reviews, provisioning

✅ **Correlation:**
- Links accounts to identities
- Usually by email or employee ID
- Uncorrelated accounts need investigation
- Can be service accounts, orphaned, or errors

✅ **Performance:**
- Delta much faster than full
- Optimize queries and attributes
- Proper VA sizing important
- Monitor and tune regularly

**Exam Focus Areas:**
- Difference between account and entitlement aggregation
- When to use full vs delta aggregation
- Understanding correlation and uncorrelated accounts
- Aggregation scheduling best practices
- What data is collected and how it's used

---

**End of Section 3.2: Aggregation Process**

**Next:** Section 3.3: Account Correlation and Management

---

**Study Tips:**
- Remember: Account = people, Entitlement = access
- Delta = faster (daily), Full = complete (weekly)
- Uncorrelated accounts need investigation
- Email is most common correlation attribute
- Off-peak scheduling prevents performance issues
