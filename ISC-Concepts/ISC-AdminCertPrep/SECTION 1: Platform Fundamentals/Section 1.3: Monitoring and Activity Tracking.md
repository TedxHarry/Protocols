# Section 1.3: Monitoring and Activity Tracking

**Master monitoring provisioning activities, using dashboards, and reading logs in SailPoint Identity Security Cloud**

---

## Table of Contents

- [1.3.1 Monitoring Provisioning Activity](#131-monitoring-provisioning-activity)
- [1.3.2 Using Activity Dashboards](#132-using-activity-dashboards)
- [1.3.3 Reading Provisioning Logs](#133-reading-provisioning-logs)

---

# 1.3.1 Monitoring Provisioning Activity

**Learning Objective:** Understand how to monitor provisioning operations and track their status in real-time.

---

## What is Provisioning Activity?

Provisioning activity refers to the actions ISC takes to create, modify, or delete accounts and access in connected systems.

**Examples of provisioning activities:**
- Creating new user account in Active Directory
- Granting access to Salesforce
- Disabling account when user leaves
- Updating email address across systems
- Removing access after approval

**Why monitor provisioning?**
- Ensure changes complete successfully
- Identify failures quickly
- Troubleshoot issues
- Meet SLA requirements
- Audit compliance

---

## Understanding the Provisioning Lifecycle

### **Provisioning Flow**
```
1. Trigger Event
   ↓
2. ISC Creates Provisioning Request
   ↓
3. Request Queued
   ↓
4. Provisioning Executes
   ↓
5. Result (Success/Failure)
   ↓
6. Identity Updated
```

### **What Triggers Provisioning?**

| Trigger Type | Example |
|-------------|---------|
| **Lifecycle Event** | New hire starts, employee leaves |
| **Access Request** | User requests VPN access |
| **Joiner/Mover/Leaver** | Department change, promotion |
| **Manual Action** | Admin enables account |
| **Scheduled Job** | Daily account sync |
| **Identity Refresh** | Attribute change propagation |

---

## Where to Monitor Provisioning

### **Primary Monitoring Locations**

**1. Activity Page**
- Real-time provisioning events
- Recent operations
- Quick status overview

**2. Search**
- Search for specific operations
- Filter by status or type
- Historical analysis

**3. Identity Profile**
- View provisioning for specific person
- See account changes
- Track access history

**4. Source Management**
- View provisioning to specific source
- Monitor connector status
- Check aggregation results

---

## Accessing Activity Monitoring

### **Step 1: Navigate to Activity**

**Method 1: Main Menu**
1. Click main menu (top navigation)
2. Select **Activity** or **Monitoring**
3. Choose **Provisioning** or **Activity List**

**Method 2: Dashboard Shortcut**
1. From home dashboard
2. Click **Recent Activity** widget
3. View latest provisioning events

### **What You'll See**

**Activity List displays:**
- Operation type (Create, Modify, Delete)
- Target account/identity
- Source system
- Status (Pending, Complete, Failed)
- Timestamp
- Details link

---

## Understanding Provisioning Status

### **Common Status Values**

| Status | Meaning | What to Do |
|--------|---------|------------|
| **Queued** | Waiting to execute | Normal - wait for processing |
| **Pending** | Currently executing | Normal - in progress |
| **Complete** | Successfully finished | No action needed |
| **Success** | Completed successfully | No action needed |
| **Failed** | Error occurred | Investigate and fix |
| **Cancelled** | Manually stopped | Review why cancelled |
| **Retry** | Retrying after failure | Monitor retry attempts |

### **Status Indicators**

**Visual indicators:**
- ✅ Green checkmark = Success
- ⏳ Yellow/orange = Pending/In Progress
- ❌ Red X = Failed
- ⏸️ Gray = Queued/Waiting

---

## Viewing Provisioning Details

### **Opening Provisioning Event**

1. Navigate to Activity list
2. Find the provisioning event
3. Click on the event row
4. Provisioning details open

### **Details Page Shows**

**Event Information:**
```
Operation: Create Account
Target: Active Directory - jsmith
Identity: John Smith
Status: Complete
Started: 2024-02-12 10:30:15
Completed: 2024-02-12 10:30:42
Duration: 27 seconds
```

**Operation Details:**
```
Requested Attributes:
  - accountName: jsmith
  - firstName: John
  - lastName: Smith
  - email: john.smith@company.com
  - department: IT
  
Result:
  - Account Created Successfully
  - Account ID: CN=jsmith,OU=Users,DC=company,DC=com
```

**Approval Chain (if applicable):**
```
Requester: John Smith
Approver 1: Jane Doe (Approved - 2024-02-12 10:15:00)
Approver 2: IT Manager (Approved - 2024-02-12 10:20:00)
```

---

## Filtering Provisioning Activity

### **Common Filters**

**Filter by Status:**
- Show only failed operations
- Show pending items
- Show completed items

**Filter by Type:**
- Account creation
- Account modification
- Account deletion
- Entitlement changes

**Filter by Source:**
- Active Directory
- Salesforce
- Workday
- Specific source

**Filter by Date:**
- Last 24 hours
- Last 7 days
- Last 30 days
- Custom date range

**Filter by Identity:**
- Specific user
- Department
- Location

### **Applying Filters**

**Example 1: Find Failed Provisioning Today**

1. Go to Activity page
2. Apply filter: Status = Failed
3. Apply filter: Date = Today
4. Review results

**Example 2: Find AD Account Creations This Week**

1. Go to Activity page
2. Apply filter: Source = Active Directory
3. Apply filter: Type = Create
4. Apply filter: Date = Last 7 days
5. Review results

---

## Monitoring Real-Time Provisioning

### **Watching Provisioning Execute**

**For immediate operations:**

1. Submit access request or trigger provisioning
2. Navigate to Activity page
3. Look for newest entry (top of list)
4. Watch status change:
   - Queued → Pending → Complete

**Refresh frequency:**
- Page auto-refreshes every 30-60 seconds
- Manual refresh: Click refresh button
- Real-time updates may vary by tenant

### **Expected Timing**

| Operation Type | Typical Duration |
|---------------|------------------|
| **Create Account** | 10-60 seconds |
| **Modify Account** | 5-30 seconds |
| **Delete Account** | 10-45 seconds |
| **Add Entitlement** | 5-20 seconds |
| **Multiple Operations** | 1-5 minutes |

**Note:** Timing varies by:
- Source system responsiveness
- Network latency
- Complexity of operation
- Current system load

---

## Identifying Provisioning Issues

### **Red Flags to Watch For**

❌ **Failed status**
- Immediate attention needed
- Check error message
- Review logs

❌ **Stuck in Pending (>10 minutes)**
- May indicate timeout
- Check source connectivity
- Review connector logs

❌ **Repeated retries**
- Underlying issue exists
- Won't succeed without fix
- Review error pattern

❌ **Queued for long time (>30 minutes)**
- System may be overloaded
- Check for bulk operations
- Review queue depth

### **Common Failure Patterns**

**Pattern 1: All operations to one source failing**
- **Likely cause:** Source system down or unreachable
- **Check:** Source connector status
- **Action:** Verify source system availability

**Pattern 2: Specific operation type failing**
- **Likely cause:** Permission issue or schema mismatch
- **Check:** Connector permissions
- **Action:** Review provisioning policy

**Pattern 3: Random failures**
- **Likely cause:** Network intermittent issues
- **Check:** Connectivity logs
- **Action:** Monitor pattern, may self-resolve

---

## Provisioning Queues

### **Understanding the Queue**

ISC processes provisioning operations through queues.

**Queue behavior:**
- Operations processed in order (FIFO - First In, First Out)
- Multiple operations can run simultaneously
- Priority may affect order
- Failed operations don't block queue

### **Viewing Queue Status**

**Location:** Activity → Provisioning Queue

**Queue displays:**
```
Queued Operations: 23
Processing: 5
Average Wait Time: 2 minutes
Estimated Completion: 10 minutes
```

### **Queue Management**

**Normal queue:**
- <50 items queued = Normal
- Processing steadily
- Items complete within minutes

**Heavy queue:**
- >100 items queued = Busy
- May experience delays
- Consider off-peak hours for bulk operations

**Stuck queue:**
- Items not processing
- Same items in queue for hours
- Contact support

---

## Monitoring Specific Scenarios

### **Scenario 1: New Hire Onboarding**

**What to monitor:**
1. Identity creation
2. Account provisioning to each system
3. Access profile assignments
4. Entitlement grants

**Monitoring steps:**
1. Search for new hire by name
2. View their provisioning activity
3. Verify all accounts created
4. Check for any failures

**Success criteria:**
- All accounts created ✅
- All access granted ✅
- No failed operations ✅
- Completed within SLA ✅

### **Scenario 2: Employee Termination**

**What to monitor:**
1. Account disablements
2. Access revocations
3. Group membership removals
4. Manager reassignments

**Monitoring steps:**
1. Search for terminated employee
2. Verify cloudLifecycleState changed to inactive
3. Check all accounts disabled
4. Confirm access removed

**Success criteria:**
- Identity status = inactive ✅
- All accounts disabled ✅
- All access revoked ✅
- Completed within SLA ✅

### **Scenario 3: Department Transfer**

**What to monitor:**
1. Attribute updates
2. Old access removals
3. New access grants
4. Manager changes

**Monitoring steps:**
1. Search for transferred employee
2. View recent provisioning
3. Check attribute changes propagated
4. Verify access changes

**Success criteria:**
- Attributes updated ✅
- Old access removed ✅
- New access granted ✅
- No orphaned access ✅

---

## Provisioning Metrics

### **Key Metrics to Track**

**Success Rate:**
```
Success Rate = (Successful Operations / Total Operations) × 100

Target: >95% success rate
```

**Average Duration:**
```
Track how long operations take
Compare against baseline
Identify slowdowns
```

**Failure Rate by Source:**
```
Which sources have most failures?
Focus troubleshooting efforts
```

**Queue Depth:**
```
How many operations waiting?
Identify peak times
Plan maintenance windows
```

### **Using Metrics**

**Daily checks:**
- Overall success rate
- Any critical failures
- Queue depth trends

**Weekly review:**
- Success rate by source
- Average operation duration
- Failure pattern analysis

**Monthly analysis:**
- Long-term trends
- Capacity planning
- Performance optimization

---

## Automated Monitoring

### **Setting Up Alerts**

Some ISC tenants support automated alerts.

**Alert types:**
- Provisioning failures
- Long-running operations
- Queue depth thresholds
- Specific error patterns

**Alert delivery:**
- Email notifications
- Webhook to monitoring system
- Integration with ITSM tools

**Example alert configuration:**
```
Alert: High Failure Rate
Condition: >10 failures in 1 hour
Recipients: iam-ops@company.com
Action: Email notification
```

---

## Best Practices for Monitoring

### **Daily Monitoring Tasks**

**Morning check (Start of day):**
1. Review overnight provisioning
2. Check for any failures
3. Verify queue is clear
4. Address any critical issues

**During day:**
1. Monitor real-time operations
2. Respond to failures promptly
3. Track SLA compliance

**End of day:**
1. Review day's activity
2. Document any issues
3. Plan next-day actions

### **Proactive Monitoring**

✅ **Do:**
- Check Activity page regularly
- Set up filters for common searches
- Monitor during peak times
- Track trends over time
- Document patterns
- Address issues promptly

❌ **Don't:**
- Only check when problems reported
- Ignore minor failures
- Let queue grow unchecked
- Delay troubleshooting
- Skip regular reviews

---

## Quick Reference

### **Monitoring Locations**

| What to Monitor | Where to Look |
|----------------|---------------|
| **Real-time activity** | Activity → Provisioning |
| **Specific identity** | Identity → Activity tab |
| **Source operations** | Sources → Activity |
| **Queue status** | Activity → Queue |
| **Failed operations** | Activity → Filter: Failed |

### **Status Quick Guide**

| Status | Action Needed |
|--------|---------------|
| ✅ **Complete** | None |
| ⏳ **Pending** | Wait (normal) |
| ⏸️ **Queued** | Wait (normal) |
| ❌ **Failed** | Investigate immediately |
| 🔄 **Retry** | Monitor progress |

### **Common Filters**
```
Failed Today:
  Status = Failed
  Date = Today

Pending AD Operations:
  Source = Active Directory
  Status = Pending

This Week's Creations:
  Type = Create
  Date = Last 7 days
```

---

## Practice Exercises

**Exercise 1:** Where do you go to see real-time provisioning activity?

<details>
<summary>Answer</summary>

Navigate to: **Activity** → **Provisioning** or **Activity List**

This shows real-time and recent provisioning operations.
</details>

---

**Exercise 2:** A new hire account creation has been "Pending" for 15 minutes. Is this normal?

<details>
<summary>Answer</summary>

No, this is not normal. Account creation typically takes 10-60 seconds.

Actions to take:
1. Check provisioning details for errors
2. Verify source system is accessible
3. Check connector status
4. Review provisioning logs
</details>

---

**Exercise 3:** How would you find all failed provisioning operations from yesterday?

<details>
<summary>Answer</summary>

1. Go to Activity page
2. Apply filter: Status = Failed
3. Apply filter: Date = Yesterday
4. Review results

Or use Search to find failed provisioning events from that date range.
</details>

---

# 1.3.2 Using Activity Dashboards

**Learning Objective:** Navigate and utilize ISC dashboards to monitor identity and provisioning activities.

---

## What are Activity Dashboards?

Activity dashboards provide visual summaries of identity and access activities in ISC.

**Purpose of dashboards:**
- Quick overview of system health
- Visual representation of data
- Trend analysis
- Performance monitoring
- Executive reporting

**Types of dashboards:**
- Home dashboard
- Provisioning dashboard
- Access request dashboard
- Certification dashboard
- Custom dashboards

---

## Accessing Dashboards

### **Home Dashboard**

**How to access:**
1. Log into ISC
2. Default landing page is Home dashboard

**What you see:**
- Recent activity summary
- Quick action tiles
- Pending work items
- System notifications
- Key metrics

### **Activity Dashboard**

**How to access:**
1. Navigate to **Activity** from main menu
2. Select **Dashboard** or **Overview**

**What you see:**
- Provisioning statistics
- Success/failure rates
- Operation trends
- Source health status
- Recent events

---

## Home Dashboard Components

### **Common Dashboard Widgets**

**1. Pending Work**
```
My Pending Approvals: 5
  - 3 Access Requests
  - 2 Certifications
  
My Access Requests: 2
  - VPN Access (Pending)
  - SharePoint Access (Approved)
```

**2. Recent Activity**
```
Last 24 Hours:
  - 45 Accounts Created
  - 23 Accounts Modified
  - 12 Access Grants
  - 2 Failed Operations
```

**3. Quick Actions**
```
Common Tasks:
  - Request Access
  - Start Certification
  - Search Identities
  - Run Reports
```

**4. System Health**
```
Connected Sources: 15/15 Healthy
Virtual Appliances: 3/3 Running
Last Aggregation: 2 hours ago
```

### **Widget Interaction**

**Clicking widgets:**
- Opens detailed view
- Filters to relevant data
- Provides more context

**Example:**
- Click "5 Pending Approvals" → Opens approval queue
- Click "2 Failed Operations" → Shows failure details

---

## Provisioning Dashboard

### **Key Metrics Displayed**

**Success Rate Chart**
```
Today's Success Rate: 98.5%
  ✅ Successful: 457
  ❌ Failed: 7
  ⏳ Pending: 12
```

**Operations by Type**
```
Create: 234 (51%)
Modify: 189 (41%)
Delete: 36 (8%)
```

**Operations by Source**
```
Active Directory: 256
Salesforce: 124
Workday: 78
Other: 66
```

**Timeline Chart**
```
Operations over last 7 days:
Mon: 520
Tue: 485
Wed: 502
Thu: 498
Fri: 467
Sat: 145
Sun: 89
```

### **Interpreting Dashboard Metrics**

**Healthy indicators:**
- Success rate >95%
- Steady operation volume
- Low failure count
- Quick operation completion
- All sources healthy

**Warning signs:**
- Success rate <90%
- Spike in failures
- Growing queue
- Operations taking longer
- Source connectivity issues

---

## Access Request Dashboard

### **Request Metrics**

**Request Status Summary**
```
Total Requests (This Month): 234

By Status:
  Completed: 198 (85%)
  Pending: 23 (10%)
  Rejected: 13 (5%)
```

**Request Types**
```
By Request Type:
  Access Profiles: 145
  Roles: 67
  Entitlements: 22
```

**Approval Times**
```
Average Approval Time: 4.2 hours

By Approver:
  Manager: 2.5 hours avg
  Security Team: 6.8 hours avg
  Application Owner: 5.1 hours avg
```

### **Using Request Dashboard**

**Monitor:**
- Pending request backlog
- Approval bottlenecks
- Request trends
- Rejection patterns

**Actions:**
- Click pending requests to review
- Identify approval delays
- Follow up on stalled requests

---

## Certification Dashboard

### **Campaign Metrics**

**Active Certifications**
```
Active Campaigns: 3

Q1 Access Review:
  Progress: 67% Complete
  Due Date: 2024-03-31
  Items Remaining: 456
  
Manager Reviews:
  Progress: 45% Complete
  Due Date: 2024-02-29
  Items Remaining: 892
```

**Certification Results**
```
Last Campaign Results:

Total Items: 2,340
  Approved: 2,156 (92%)
  Revoked: 184 (8%)
  
Completion Rate: 100%
Completed On Time: 95%
```

### **Campaign Monitoring**

**Track:**
- Completion percentage
- Items remaining
- Days until deadline
- Reviewer activity
- Remediation status

**Alert on:**
- Low completion rate near deadline
- Inactive reviewers
- High revocation rate
- Overdue campaigns

---

## Custom Dashboard Views

### **Creating Custom Views**

Some ISC configurations allow custom dashboards.

**Customization options:**
- Add/remove widgets
- Resize widgets
- Arrange layout
- Filter data displayed
- Set refresh intervals

**Example custom dashboard:**
```
IAM Operations Dashboard:

Row 1:
  - Failed Provisioning (Last 24h)
  - Queue Depth
  - Source Health

Row 2:
  - Operations by Source (Chart)
  - Success Rate Trend (Chart)

Row 3:
  - Recent Failures (List)
```

### **Saving Dashboard Layouts**

**To save custom layout:**
1. Arrange widgets as desired
2. Click **Save Layout** or **Save Dashboard**
3. Name your custom view
4. Set as default (optional)

**To switch between views:**
1. Click dashboard selector
2. Choose saved layout
3. Dashboard reconfigures

---

## Dashboard Drill-Down

### **Going from Dashboard to Details**

Dashboards provide high-level overview; click for details.

**Example drill-down flow:**

**Level 1: Dashboard**
```
Failed Operations: 7
```
↓ Click

**Level 2: Filtered List**
```
7 Failed Operations:
1. Create account - jsmith - AD
2. Add entitlement - mjones - Salesforce
...
```
↓ Click specific failure

**Level 3: Operation Details**
```
Operation: Create Account
Target: Active Directory - jsmith
Error: User already exists
Timestamp: 2024-02-12 10:45:23
Full Error Log: [View]
```

---

## Dashboard Refresh and Updates

### **Auto-Refresh**

**Default behavior:**
- Dashboards auto-refresh every 1-5 minutes
- Shows near real-time data
- No manual refresh needed

**Manual refresh:**
- Click refresh button
- Force immediate update
- Useful after known changes

### **Data Lag**

**Understanding timing:**
- Dashboard shows processed data
- May lag 1-2 minutes behind real-time
- Normal operational delay
- Not an issue for monitoring

---

## Exporting Dashboard Data

### **Export Options**

**Available formats:**
- Screenshot (image)
- CSV (data tables)
- PDF (formatted report)

**How to export:**
1. Navigate to desired dashboard
2. Click **Export** or **Download**
3. Select format
4. Choose data range
5. Download file

**Use cases:**
- Executive reports
- Trend analysis
- Historical records
- Compliance documentation

---

## Mobile Dashboard Access

### **Mobile App**

Some ISC features available via mobile app.

**Mobile dashboard features:**
- View key metrics
- Approve requests
- Monitor critical alerts
- Basic provisioning status

**Limitations:**
- Reduced functionality vs desktop
- Limited customization
- Smaller data views

---

## Dashboard Best Practices

### **Daily Dashboard Review**

**Morning routine:**
1. Check Home dashboard
2. Review overnight activity
3. Check pending work
4. Address critical issues

**Throughout day:**
1. Monitor provisioning dashboard
2. Watch for failure spikes
3. Check request backlog
4. Track certification progress

**End of day:**
1. Review day's metrics
2. Document issues
3. Plan next day

### **Weekly Dashboard Analysis**

**Weekly tasks:**
1. Review trend charts
2. Compare week-over-week
3. Identify patterns
4. Document observations
5. Share with team

### **Dashboard Optimization**

✅ **Do:**
- Focus on actionable metrics
- Remove unused widgets
- Set appropriate refresh rates
- Use meaningful date ranges
- Share important dashboards

❌ **Don't:**
- Overcrowd with widgets
- Ignore data trends
- Rely solely on dashboards (use detailed views too)
- Forget to export important data
- Skip regular reviews

---

## Troubleshooting Dashboard Issues

### **Dashboard Not Loading**

**Possible causes:**
- Browser cache issues
- Network connectivity
- Session timeout
- Permissions

**Solutions:**
1. Refresh browser (Ctrl+F5)
2. Clear cache and cookies
3. Try different browser
4. Verify network connection
5. Log out and back in

### **Data Not Updating**

**Check:**
- Auto-refresh enabled
- Manual refresh button
- Data timestamp
- Known system issues

**Solution:**
- Wait for next refresh cycle
- Manual refresh
- Check system status page

### **Missing Widgets**

**Possible causes:**
- Insufficient permissions
- Feature not enabled
- Custom layout removed widget

**Solutions:**
- Check user permissions
- Verify feature enabled
- Reset to default layout

---

## Dashboard for Different Roles

### **Administrator Dashboard Focus**

**Key widgets:**
- System health
- Source connectivity
- Failed operations
- Queue depth
- Performance metrics

**Daily actions:**
- Monitor source health
- Address failures
- Manage queue
- Review performance

### **Manager Dashboard Focus**

**Key widgets:**
- Pending approvals
- Team certifications
- Team access requests
- Compliance status

**Daily actions:**
- Approve requests
- Review certifications
- Monitor team activity
- Track compliance

### **Help Desk Dashboard Focus**

**Key widgets:**
- Pending tickets
- Recent provisioning
- Failed operations
- User activity

**Daily actions:**
- Monitor user issues
- Track provisioning status
- Respond to failures
- Update tickets

---

## Quick Reference

### **Dashboard Locations**

| Dashboard Type | Navigation |
|---------------|------------|
| **Home** | Default landing page |
| **Activity** | Activity → Dashboard |
| **Provisioning** | Activity → Provisioning → Dashboard |
| **Requests** | Access → Requests → Dashboard |
| **Certifications** | Governance → Certifications → Dashboard |

### **Key Metrics to Monitor**

| Metric | Target | Action If Below |
|--------|--------|-----------------|
| **Success Rate** | >95% | Investigate failures |
| **Queue Depth** | <50 items | Review if growing |
| **Avg Operation Time** | <60 sec | Check performance |
| **Approval Time** | <24 hours | Follow up with approvers |
| **Certification Progress** | On track | Send reminders |

### **Common Dashboard Actions**
```
View Details: Click widget or metric
Refresh Data: Click refresh button
Export Report: Click export button
Filter Data: Use date/status filters
Drill Down: Click chart elements
```

---

## Practice Exercises

**Exercise 4:** Where can you see a quick summary of pending work items?

<details>
<summary>Answer</summary>

**Home Dashboard** - Look for the "Pending Work" or "My Work" widget which shows:
- Pending approvals
- Access requests
- Certifications
- Other assigned tasks
</details>

---

**Exercise 5:** Your dashboard shows success rate dropped to 85%. What should you do?

<details>
<summary>Answer</summary>

1. Click on the success rate metric to drill down
2. Filter to show only failed operations
3. Identify which source(s) are failing
4. Review error messages
5. Check source connectivity
6. Address root cause
7. Monitor for improvement
</details>

---

**Exercise 6:** You need to create a weekly report showing provisioning trends. Which dashboard should you use?

<details>
<summary>Answer</summary>

**Provisioning Dashboard** - It shows:
- Operations over time (trend charts)
- Success/failure rates
- Operations by source
- Can be exported to PDF/CSV for reporting
</details>

---

# 1.3.3 Reading Provisioning Logs

**Learning Objective:** Understand how to access, read, and interpret provisioning logs for troubleshooting.

---

## What are Provisioning Logs?

Provisioning logs are detailed records of every provisioning operation ISC performs.

**What logs contain:**
- Step-by-step operation details
- API calls made to source systems
- Responses from source systems
- Error messages and stack traces
- Timing information
- Request/response data

**Why logs are important:**
- Detailed troubleshooting
- Root cause analysis
- Audit trail
- Compliance evidence
- Performance analysis

---

## Types of Logs in ISC

### **Log Categories**

| Log Type | What It Contains | When to Use |
|----------|------------------|-------------|
| **Provisioning Logs** | Account create/modify/delete operations | Troubleshoot provisioning failures |
| **Aggregation Logs** | Source data import operations | Troubleshoot sync issues |
| **Connector Logs** | Connector communication details | Debug connector issues |
| **System Logs** | General system events | Overall system health |
| **Audit Logs** | User actions and changes | Compliance and security |

---

## Accessing Provisioning Logs

### **Method 1: From Provisioning Event**

**Steps:**
1. Navigate to **Activity** → **Provisioning**
2. Find the provisioning operation
3. Click on the operation
4. Click **View Logs** or **Details**
5. Logs display

### **Method 2: From Identity**

**Steps:**
1. Navigate to specific identity
2. Go to **Activity** tab
3. Find provisioning event
4. Click **View Logs**

### **Method 3: From Source**

**Steps:**
1. Navigate to **Admin** → **Connections** → **Sources**
2. Select the source
3. Go to **Activity** or **Logs** tab
4. View source-specific logs

---

## Understanding Log Structure

### **Log Entry Components**

**Typical log entry:**
```
Timestamp: 2024-02-12 10:30:15.234
Level: INFO
Operation: CreateAccount
Source: Active Directory
Identity: john.smith
Message: Account creation initiated
Request ID: req-2024-02-12-abc123
```

**Key fields:**

| Field | Description |
|-------|-------------|
| **Timestamp** | When event occurred |
| **Level** | INFO, WARN, ERROR, DEBUG |
| **Operation** | What action was performed |
| **Source** | Target system |
| **Identity** | Affected user |
| **Message** | Human-readable description |
| **Request ID** | Unique operation identifier |

### **Log Levels**

| Level | Meaning | Example |
|-------|---------|---------|
| **DEBUG** | Detailed technical info | API request parameters |
| **INFO** | Normal operations | "Account created successfully" |
| **WARN** | Non-critical issues | "Retry attempted" |
| **ERROR** | Operation failures | "Failed to create account" |
| **FATAL** | Critical system errors | "Connector unavailable" |

---

## Reading Successful Provisioning Logs

### **Example: Successful Account Creation**
```
[2024-02-12 10:30:15] INFO - Starting account provisioning
  Identity: john.smith (2c9180887...)
  Source: Active Directory
  Operation: Create
  Request ID: req-abc123

[2024-02-12 10:30:16] DEBUG - Building provisioning request
  Account Name: jsmith
  Attributes:
    - firstName: John
    - lastName: Smith
    - email: john.smith@company.com
    - department: IT

[2024-02-12 10:30:17] DEBUG - Sending request to connector
  Endpoint: https://ad-connector.company.com/create
  Method: POST
  Payload: [See full request]

[2024-02-12 10:30:35] INFO - Received response from connector
  Status: 200 OK
  Account DN: CN=jsmith,OU=Users,DC=company,DC=com

[2024-02-12 10:30:36] INFO - Updating identity cube
  Added account reference
  Account ID: acc-xyz789

[2024-02-12 10:30:37] INFO - Provisioning completed successfully
  Duration: 22 seconds
  Status: Complete
```

**Reading this log:**
1. Operation started at 10:30:15
2. Request built with user attributes
3. Sent to Active Directory connector
4. Connector responded successfully (200 OK)
5. ISC updated identity record
6. Total time: 22 seconds

---

## Reading Failed Provisioning Logs

### **Example: Failed Account Creation**
```
[2024-02-12 11:15:20] INFO - Starting account provisioning
  Identity: jane.doe (2c9180887...)
  Source: Active Directory
  Operation: Create
  Request ID: req-def456

[2024-02-12 11:15:21] DEBUG - Building provisioning request
  Account Name: jdoe
  Attributes:
    - firstName: Jane
    - lastName: Doe
    - email: jane.doe@company.com

[2024-02-12 11:15:22] DEBUG - Sending request to connector
  Endpoint: https://ad-connector.company.com/create
  Method: POST

[2024-02-12 11:15:45] ERROR - Connector returned error
  Status: 400 Bad Request
  Error Code: LDAP_ENTRY_ALREADY_EXISTS
  Error Message: An account with sAMAccountName 'jdoe' already exists

[2024-02-12 11:15:46] ERROR - Provisioning failed
  Reason: Account already exists in target system
  Recommendation: Verify account uniqueness or correlate existing account
  
[2024-02-12 11:15:46] INFO - Provisioning completed with errors
  Duration: 26 seconds
  Status: Failed
  Error: LDAP_ENTRY_ALREADY_EXISTS
```

**Reading this log:**
1. Operation started normally
2. Request sent to Active Directory
3. **ERROR** received from AD: Account already exists
4. ISC marked provisioning as failed
5. Error clearly indicates: duplicate account
6. Recommendation provided: correlate or rename

---

## Common Error Messages

### **Error: Account Already Exists**

**Log excerpt:**
```
ERROR: LDAP_ENTRY_ALREADY_EXISTS
Message: An account with sAMAccountName 'jsmith' already exists
```

**Meaning:**
- Target system already has account with same username
- Cannot create duplicate

**Resolution:**
1. Check if account should be correlated
2. Verify account not already managed
3. Consider different username
4. Or correlate existing account to identity

### **Error: Insufficient Permissions**

**Log excerpt:**
```
ERROR: ACCESS_DENIED
Message: Insufficient permissions to create user in OU=Users
Error Code: 0x80070005
```

**Meaning:**
- Connector account lacks required permissions
- Cannot perform requested operation

**Resolution:**
1. Verify connector service account permissions
2. Grant necessary rights in target system
3. Test connector connectivity
4. Retry operation

### **Error: Connection Timeout**

**Log excerpt:**
```
ERROR: CONNECTION_TIMEOUT
Message: Failed to connect to target system
Timeout after: 30 seconds
```

**Meaning:**
- Cannot reach target system
- Network or system availability issue

**Resolution:**
1. Check target system is running
2. Verify network connectivity
3. Check firewall rules
4. Verify Virtual Appliance status
5. Retry operation

### **Error: Invalid Attribute Value**

**Log excerpt:**
```
ERROR: INVALID_ATTRIBUTE_VALUE
Message: Invalid value for attribute 'telephoneNumber'
Value: 'N/A'
Constraint: Must be valid phone number format
```

**Meaning:**
- Data doesn't meet target system requirements
- Validation failed

**Resolution:**
1. Review attribute mapping
2. Add data validation/transformation
3. Update source data
4. Modify provisioning policy

### **Error: Required Attribute Missing**

**Log excerpt:**
```
ERROR: MISSING_REQUIRED_ATTRIBUTE
Message: Required attribute 'employeeID' not provided
```

**Meaning:**
- Source didn't provide required field
- Cannot proceed without it

**Resolution:**
1. Check identity profile mappings
2. Verify source data has required attribute
3. Add static value if appropriate
4. Update provisioning policy

---

## Log Search and Filtering

### **Searching Logs**

**Common search criteria:**
- Error keyword: "ERROR", "FAILED"
- Operation type: "Create", "Modify", "Delete"
- Identity name: "john.smith"
- Source name: "Active Directory"
- Date/time range
- Request ID

### **Filter Examples**

**Find all errors today:**
```
Level: ERROR
Date: Today
```

**Find specific identity's operations:**
```
Identity: john.smith
Date: Last 7 days
```

**Find Active Directory failures:**
```
Source: Active Directory
Level: ERROR
Date: Last 24 hours
```

---

## Log Analysis Techniques

### **Technique 1: Timeline Analysis**

Track operation flow through timestamps.

**Example:**
```
10:30:15 - Operation started
10:30:16 - Request built (1 second)
10:30:17 - Sent to connector (1 second)
10:30:35 - Connector responded (18 seconds) ← Long delay
10:30:36 - Updated ISC (1 second)
10:30:37 - Completed (1 second)
```

**Analysis:** 18-second delay at connector = target system slow

### **Technique 2: Pattern Recognition**

Look for repeating errors.

**Example:**
```
11:00 - Create failed: Already exists
11:15 - Create failed: Already exists
11:30 - Create failed: Already exists
```

**Pattern:** All creates failing with "already exists" = correlation issue

### **Technique 3: Error Correlation**

Connect related errors across systems.

**Example:**
```
Source aggregation log: "Failed to fetch user data - timeout"
Provisioning log: "No data available for provisioning"
```

**Correlation:** Aggregation failure caused provisioning failure

---

## Downloading Logs

### **When to Download Logs**

**Download for:**
- Detailed offline analysis
- Sharing with support
- Long-term archival
- Compliance documentation
- Bulk log review

### **How to Download**

**Method 1: From UI**
1. Navigate to log view
2. Click **Export** or **Download**
3. Select date range
4. Choose format (TXT, CSV)
5. Save file

**Method 2: Via API**
```
GET /v3/provisioning-logs?start=2024-02-12&end=2024-02-13
```

Downloads logs for specified date range

---

## Log Retention

### **How Long Logs are Kept**

**Typical retention:**
- Online viewing: 30-90 days
- Archived: May vary by tenant
- Downloadable: While online

**Important:**
- Download critical logs for long-term storage
- Don't rely solely on ISC retention
- Archive important troubleshooting logs

---

## Advanced Log Analysis

### **Using Request IDs**

Every operation has unique Request ID.

**Tracing full operation:**
1. Find Request ID in failed operation
2. Search logs for that Request ID
3. See all related log entries
4. Follow complete operation flow

**Example:**
```
Search: Request ID "req-abc123"

Results show:
- Initial request
- Approval steps
- Provisioning attempts
- All retries
- Final result
```

### **Correlating Multiple Sources**

When operations span multiple sources.

**Example: Job change provisioning**
1. Identity attribute updated (Workday)
2. Access removed (Old department AD group)
3. Access added (New department AD group)
4. Email updated (Exchange)

**Trace through logs:**
- Start with identity change log
- Follow to each provisioning operation
- Verify all steps completed
- Identify any failures

---

## Troubleshooting with Logs

### **Troubleshooting Process**

**Step 1: Identify the failure**
- Navigate to failed operation
- Note error message

**Step 2: Open detailed logs**
- Click "View Logs"
- Read complete log

**Step 3: Find error section**
- Look for "ERROR" level entries
- Read error message and code

**Step 4: Understand context**
- Read entries before error
- What was being attempted?
- What data was sent?

**Step 5: Determine cause**
- Network issue?
- Permission issue?
- Data validation issue?
- System availability?

**Step 6: Plan resolution**
- Based on error type
- Fix root cause
- Retry operation

---

## Log Best Practices

### **For Administrators**

✅ **Do:**
- Review error logs daily
- Download important logs
- Track patterns over time
- Document recurring issues
- Share logs with team
- Keep logs for audits

❌ **Don't:**
- Ignore warning messages
- Delete logs prematurely
- Share logs publicly (may contain sensitive data)
- Overlook patterns
- Wait to review until major issue

### **For Troubleshooting**

**Effective log reading:**
1. Start at the beginning
2. Follow chronologically
3. Note each step
4. Identify where it failed
5. Read error carefully
6. Check related systems
7. Document findings

---

## Quick Reference

### **Log Locations**

| What to Find | Where to Look |
|-------------|---------------|
| **Provisioning logs** | Activity → Event → View Logs |
| **Source logs** | Sources → Source → Logs tab |
| **Identity activity** | Identity → Activity tab |
| **Connector logs** | Source → Advanced → Connector Logs |

### **Log Levels**

| Level | When to Review |
|-------|---------------|
| **DEBUG** | Deep troubleshooting |
| **INFO** | Normal monitoring |
| **WARN** | Potential issues |
| **ERROR** | Immediate attention |
| **FATAL** | Critical issues |

### **Common Error Codes**

| Error Code | Meaning | Action |
|-----------|---------|--------|
| **LDAP_ENTRY_ALREADY_EXISTS** | Duplicate account | Correlate or rename |
| **ACCESS_DENIED** | Permission issue | Check service account |
| **CONNECTION_TIMEOUT** | Network/system issue | Check connectivity |
| **INVALID_ATTRIBUTE_VALUE** | Data validation | Fix data or mapping |
| **MISSING_REQUIRED_ATTRIBUTE** | Missing data | Update mapping |

---

## Practice Exercises

**Exercise 7:** You see this error: "LDAP_ENTRY_ALREADY_EXISTS". What does it mean?

<details>
<summary>Answer</summary>

**Meaning:** An account with the same username already exists in Active Directory.

**Common causes:**
- Previous account not deleted
- Account created manually
- Correlation not working

**Solutions:**
1. Correlate the existing account to the identity
2. Use a different username
3. Delete the existing account (if appropriate)
4. Check correlation rules
</details>

---

**Exercise 8:** Where do you find detailed logs for a failed provisioning operation?

<details>
<summary>Answer</summary>

1. Go to **Activity** → **Provisioning**
2. Find the failed operation
3. Click on the operation
4. Click **View Logs** or **Details**
5. Read the detailed provisioning log
</details>

---

**Exercise 9:** A log shows "Duration: 180 seconds". Is this normal for account creation?

<details>
<summary>Answer</summary>

**No**, this is not normal. Typical account creation takes 10-60 seconds.

**Possible causes:**
- Target system is slow
- Network latency
- Complex provisioning rules
- System under heavy load

**Actions:**
1. Check target system performance
2. Review network connectivity
3. Check for bulk operations
4. Review provisioning policy complexity
</details>

---

**Exercise 10:** You need to troubleshoot why an account wasn't created. What log information should you look for?

<details>
<summary>Answer</summary>

Look for:
1. **ERROR level entries** - Main error message
2. **Error code** - Specific error identifier
3. **Timestamp** - When failure occurred
4. **Request data** - What was sent to target system
5. **Response** - What target system returned
6. **Previous entries** - What led to the failure
7. **Recommendations** - Suggested actions

Example error chain:
- Request built → Request sent → Error response → Failure logged
</details>

---

## Section Summary

You've learned:

✅ **Monitoring Provisioning** - Track operations in real-time  
✅ **Using Dashboards** - Visual overview of activities  
✅ **Reading Logs** - Detailed troubleshooting with logs  
✅ **Error Analysis** - Understanding and resolving errors  
✅ **Best Practices** - Effective monitoring techniques  

**Next Steps:**
- Practice monitoring daily provisioning
- Familiarize yourself with dashboards
- Review logs for sample failures
- Create monitoring routine

---

## Additional Resources

📚 **Official Documentation:**
- [Monitoring Guide](https://documentation.sailpoint.com/saas/help/common/monitoring.html)
- [Provisioning Logs](https://documentation.sailpoint.com/saas/help/provisioning/provisioning-logs.html)
- [Troubleshooting](https://documentation.sailpoint.com/saas/help/common/troubleshooting.html)

💡 **Pro Tips:**
- Check Activity page first thing each morning
- Set up filters for common searches
- Download logs before they expire
- Document recurring issues and resolutions
- Share knowledge with team

---

**End of Section 1.3: Monitoring and Activity Tracking**

**Next Section:** [1.4 Security Administration →](#14-security-administration)
