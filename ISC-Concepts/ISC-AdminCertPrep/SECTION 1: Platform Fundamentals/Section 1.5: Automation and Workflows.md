# Section 1.5: Automation and Workflows

**Master event triggers, workflow components, and common automation patterns in SailPoint Identity Security Cloud**

---

## Table of Contents

- [1.5.1 Event Triggers: Purpose and Use Cases](#151-event-triggers-purpose-and-use-cases)
- [1.5.2 Workflow Components and Structure](#152-workflow-components-and-structure)
- [1.5.3 Common Workflow Patterns](#153-common-workflow-patterns)

---

# 1.5.1 Event Triggers: Purpose and Use Cases

**Learning Objective:** Understand event triggers and how they enable automation in SailPoint ISC.

---

## What are Event Triggers?

Event triggers are automated responses to events happening in ISC.

**Think of event triggers as:**
- "If this happens, then do that"
- Webhooks for ISC events
- Automation starting points
- Integration points

**Simple example:**
```
When: Identity is created
Then: Send notification to manager
```

**Why use event triggers?**
- Automate repetitive tasks
- Real-time notifications
- Integration with external systems
- Custom business logic
- Improved efficiency

---

## How Event Triggers Work

### **Event Trigger Flow**
```
1. Event Occurs in ISC
   ↓
2. ISC Detects Event
   ↓
3. Event Trigger Fires
   ↓
4. Action Executed (Workflow, Webhook, etc.)
   ↓
5. Result Logged
```

### **Example Flow: New Hire**
```
Event: Identity Created
  ↓
Trigger: "New Employee Created"
  ↓
Actions:
  - Send welcome email
  - Notify manager
  - Create ticket in ServiceNow
  - Start workflow for access provisioning
```

---

## Types of Events

### **Identity Events**

Events related to identity changes.

| Event | When It Fires | Example Use Case |
|-------|---------------|------------------|
| **Identity Created** | New identity appears in ISC | Welcome email, manager notification |
| **Identity Updated** | Identity attributes change | Department transfer notification |
| **Identity Deleted** | Identity removed | Cleanup tasks, archival |
| **Lifecycle State Changed** | Active to inactive, etc. | Termination workflow |

### **Access Events**

Events related to access changes.

| Event | When It Fires | Example Use Case |
|-------|---------------|------------------|
| **Access Request Submitted** | User requests access | Notify approvers |
| **Access Request Approved** | Request approved | Notify requester |
| **Access Request Denied** | Request rejected | Notify requester, log reason |
| **Access Granted** | Access provisioned | Update CMDB |
| **Access Revoked** | Access removed | Audit log entry |

### **Account Events**

Events related to account operations.

| Event | When It Fires | Example Use Case |
|-------|---------------|------------------|
| **Account Created** | New account in source | Notify app owner |
| **Account Updated** | Account modified | Track changes |
| **Account Deleted** | Account removed | Audit logging |
| **Account Uncorrelated** | Orphaned account found | Alert for remediation |
| **Account Locked** | Too many failed logins | Security notification |

### **Certification Events**

Events related to access reviews.

| Event | When It Fires | Example Use Case |
|-------|---------------|------------------|
| **Campaign Started** | Certification launched | Notify reviewers |
| **Campaign Completed** | All reviews done | Generate report |
| **Decision Made** | Reviewer makes decision | Update tracking |
| **Campaign Deadline** | Campaign due soon | Send reminders |

### **Source Events**

Events related to source systems.

| Event | When It Fires | Example Use Case |
|-------|---------------|------------------|
| **Aggregation Completed** | Source sync finished | Notify admins of results |
| **Aggregation Failed** | Source sync error | Alert support team |
| **Source Created** | New source added | Security review |
| **Source Deleted** | Source removed | Audit logging |

---

## Accessing Event Triggers

### **Navigation**

**Location:** Admin → Workflows → Event Triggers
(Path may vary slightly by tenant configuration)

**What you'll see:**
- List of available event types
- Configured triggers
- Trigger status (Enabled/Disabled)
- Edit/Create options

---

## Creating an Event Trigger

### **Step-by-Step: Simple Event Trigger**

**Scenario:** Send email when new identity is created

**Step 1: Navigate to Event Triggers**
1. Go to **Admin** → **Workflows** → **Event Triggers**
2. Click **Create Event Trigger** or **New Trigger**

**Step 2: Select Event Type**
```
Event Type: Identity Created
Description: Trigger when new employee identity is created
```

**Step 3: Configure Trigger**
```
Trigger Name: New Employee Notification
Enabled: Yes
Filter: (optional) Only for department:IT
```

**Step 4: Choose Action**
```
Action Type: Send Email

Email Configuration:
  To: manager@company.com
  Subject: New Employee Created - {identity.name}
  Body: 
    A new employee has been created:
    
    Name: {identity.displayName}
    Email: {identity.email}
    Department: {identity.department}
    Start Date: {identity.startDate}
```

**Step 5: Test Trigger**
- Use test function if available
- Create test identity
- Verify email sent

**Step 6: Enable Trigger**
- Activate trigger
- Monitor for first real event

---

## Event Trigger Actions

### **Available Actions**

**1. Execute Workflow**
```
Action: Run SailPoint Workflow
Workflow: "New Hire Onboarding"
Input: Pass identity details to workflow
```

**2. Send Webhook**
```
Action: HTTP POST to external URL
URL: https://company.com/api/new-employee
Headers: 
  Authorization: Bearer {token}
  Content-Type: application/json
Body: JSON with event details
```

**3. Send Email**
```
Action: Send email notification
To: Configurable recipients
Subject: Event-based
Body: Customizable with variables
```

**4. Create Ticket**
```
Action: Integration with ITSM
System: ServiceNow, Jira, etc.
Ticket Type: Incident, Request, Task
Details: Auto-populated from event
```

---

## Event Filtering

### **When to Use Filters**

Filters limit which events trigger actions.

**Without filter:**
```
Event: Identity Created
Triggers for: ALL new identities
```

**With filter:**
```
Event: Identity Created
Filter: department equals "IT"
Triggers for: Only IT department identities
```

### **Filter Examples**

**Filter by department:**
```
Filter Expression:
  identity.department == "IT"
  
Result: Only IT employees trigger this
```

**Filter by lifecycle state:**
```
Filter Expression:
  identity.cloudLifecycleState == "active"
  
Result: Only active identities trigger
```

**Filter by attribute:**
```
Filter Expression:
  identity.attributes.employeeType == "Contractor"
  
Result: Only contractors trigger
```

**Complex filter:**
```
Filter Expression:
  identity.department == "IT" AND 
  identity.cloudLifecycleState == "active" AND
  identity.location == "NYC"
  
Result: Only active IT employees in NYC
```

---

## Event Trigger Use Cases

### **Use Case 1: New Hire Welcome**

**Scenario:** Automatically welcome new employees

**Configuration:**
```
Event: Identity Created
Filter: cloudLifecycleState equals "active"

Action: Execute Workflow
Workflow: "New Hire Welcome"

Workflow Steps:
  1. Send welcome email to employee
  2. Send notification to manager
  3. Create ticket for equipment setup
  4. Schedule 30-day check-in
```

### **Use Case 2: Termination Cleanup**

**Scenario:** Ensure all access removed when employee leaves

**Configuration:**
```
Event: Lifecycle State Changed
Filter: New state equals "inactive"

Action: Execute Workflow
Workflow: "Employee Termination"

Workflow Steps:
  1. Disable all accounts
  2. Revoke all access
  3. Notify manager
  4. Create ticket for equipment return
  5. Update HRIS system
```

### **Use Case 3: Orphaned Account Alert**

**Scenario:** Alert when uncorrelated accounts found

**Configuration:**
```
Event: Account Uncorrelated

Action: Send Email
To: iam-team@company.com
Subject: Uncorrelated Account Found
Body:
  An orphaned account was detected:
  
  Account: {account.name}
  Source: {account.source.name}
  Created: {account.created}
  
  Please correlate or investigate.
```

### **Use Case 4: High-Risk Access Notification**

**Scenario:** Alert security when privileged access granted

**Configuration:**
```
Event: Access Granted
Filter: access.privileged equals true

Action: Send Webhook
URL: https://siem.company.com/api/alerts
Body:
  {
    "type": "privileged_access_granted",
    "identity": "{identity.name}",
    "access": "{access.name}",
    "timestamp": "{event.timestamp}"
  }
```

### **Use Case 5: Manager Notification on Team Changes**

**Scenario:** Notify manager when their team changes

**Configuration:**
```
Event: Identity Updated
Filter: manager attribute changed

Action: Send Email
To: {identity.manager.email}
Subject: Team Update - {identity.name}

Body:
  Your team member has been updated:
  
  Employee: {identity.displayName}
  Previous Manager: {event.oldValue.manager}
  New Manager: {event.newValue.manager}
```

---

## Event Data and Variables

### **Available Event Data**

Each event provides specific data you can use.

**Identity Created Event:**
```
Available Variables:
  {identity.id}
  {identity.name}
  {identity.displayName}
  {identity.email}
  {identity.department}
  {identity.manager}
  {identity.cloudLifecycleState}
  {identity.startDate}
  {identity.attributes.*}
```

**Access Request Event:**
```
Available Variables:
  {request.id}
  {request.requestedFor.name}
  {request.requestedBy.name}
  {request.requestedItems[].name}
  {request.status}
  {request.created}
  {request.comment}
```

**Account Event:**
```
Available Variables:
  {account.id}
  {account.name}
  {account.nativeIdentity}
  {account.source.name}
  {account.identity.name}
  {account.disabled}
  {account.uncorrelated}
```

### **Using Variables in Actions**

**In email subject:**
```
Subject: New Employee - {identity.displayName} - {identity.department}

Result: New Employee - John Smith - IT
```

**In email body:**
```
Dear {identity.manager.displayName},

A new team member has joined:
Name: {identity.displayName}
Email: {identity.email}
Start Date: {identity.startDate}

Please ensure they have appropriate access.
```

**In webhook payload:**
```json
{
  "event_type": "identity_created",
  "identity": {
    "name": "{identity.name}",
    "email": "{identity.email}",
    "department": "{identity.department}"
  },
  "timestamp": "{event.timestamp}"
}
```

---

## Testing Event Triggers

### **Test Before Production**

**Always test triggers before enabling:**

**Method 1: Manual Test**
1. Create test identity/account
2. Verify trigger fires
3. Check action executed correctly
4. Review logs
5. Delete test data

**Method 2: Test Function**
Some ISC versions have built-in test:
1. Configure trigger
2. Click "Test"
3. Provide sample data
4. Review test results

**Method 3: Non-Prod Testing**
1. Configure in sandbox/dev tenant
2. Test thoroughly
3. Document results
4. Replicate in production

### **Test Checklist**
```
☐ Event fires when expected
☐ Event doesn't fire when not expected
☐ Filter works correctly
☐ Action executes successfully
☐ Variables populate correctly
☐ Recipients receive notifications
☐ External systems receive webhooks
☐ Errors handled gracefully
☐ Logs show correct information
```

---

## Monitoring Event Triggers

### **Checking Trigger Execution**

**View trigger history:**
1. Navigate to Event Triggers
2. Select trigger
3. View **Execution History** or **Logs**

**Execution log shows:**
```
Timestamp: 2024-02-12 10:30:15
Event: Identity Created
Identity: john.smith
Trigger: New Employee Notification
Action: Send Email
Status: Success
Recipient: manager@company.com
Duration: 2.3 seconds
```

### **Troubleshooting Failed Triggers**

**Common failures:**

**Issue 1: Trigger not firing**
```
Possible Causes:
  - Trigger disabled
  - Filter too restrictive
  - Event type mismatch

Check:
  ✓ Trigger enabled
  ✓ Filter logic correct
  ✓ Event type matches scenario
```

**Issue 2: Action failing**
```
Possible Causes:
  - Invalid email address
  - Webhook URL unreachable
  - Workflow error

Check:
  ✓ Email addresses valid
  ✓ External systems accessible
  ✓ Workflow tested separately
```

**Issue 3: Wrong data in output**
```
Possible Causes:
  - Incorrect variables
  - Data not available
  - Syntax error

Check:
  ✓ Variable names correct
  ✓ Data exists for event
  ✓ Syntax matches documentation
```

---

## Event Trigger Best Practices

### **Design Principles**

✅ **Do:**
- Use specific event types
- Apply filters to limit scope
- Test thoroughly before production
- Document trigger purpose
- Use descriptive names
- Monitor execution regularly
- Handle errors gracefully
- Keep actions simple and focused

❌ **Don't:**
- Create too many triggers for same event
- Use overly complex filters
- Skip testing
- Leave triggers unnamed/undocumented
- Ignore failed executions
- Create circular triggers (trigger that triggers itself)
- Perform long-running operations directly

### **Performance Considerations**

**Keep triggers lightweight:**
```
✅ Good:
  - Send simple notification
  - Call external API (quick)
  - Start workflow

❌ Avoid:
  - Complex data processing in trigger
  - Multiple sequential API calls
  - Long-running operations
```

**Use workflows for complex logic:**
```
Instead of:
  Trigger → Complex logic → Multiple actions

Use:
  Trigger → Start Workflow → Workflow handles complexity
```

---

## Security Considerations

### **Event Trigger Security**

**Protect sensitive data:**
```
❌ Don't include in webhooks:
  - Passwords
  - SSN/sensitive PII
  - API keys
  - Full account details

✅ Do include:
  - Identity names
  - Non-sensitive attributes
  - Event metadata
  - Timestamps
```

**Authentication for webhooks:**
```
Always use:
  - API keys in headers
  - OAuth tokens
  - Certificate-based auth
  - IP allowlisting

Never:
  - Send to unauthenticated endpoints
  - Include credentials in URL
  - Send over HTTP (use HTTPS)
```

**Audit logging:**
```
All trigger executions logged:
  - What event occurred
  - When trigger fired
  - What action taken
  - Success/failure status
  - Any errors
```

---

## Quick Reference

### **Common Event Triggers**

| Scenario | Event Type | Common Action |
|----------|-----------|---------------|
| **New hire** | Identity Created | Send welcome email |
| **Termination** | Lifecycle State Changed | Execute offboarding workflow |
| **Department change** | Identity Updated | Notify managers |
| **Access request** | Access Request Submitted | Notify approvers |
| **Orphaned account** | Account Uncorrelated | Alert IAM team |
| **Certification due** | Campaign Deadline | Send reminders |
| **Failed aggregation** | Aggregation Failed | Alert support team |

### **Event Trigger Syntax**
```
Basic Structure:
  Event: [Event Type]
  Filter: [Optional Condition]
  Action: [What to do]

Example:
  Event: Identity Created
  Filter: department == "IT"
  Action: Send Email to manager
```

### **Variable Usage**
```
Email Subject:
  New Employee: {identity.displayName}

Email Body:
  Name: {identity.name}
  Email: {identity.email}
  Department: {identity.department}

Webhook:
  {
    "identity": "{identity.id}",
    "event": "{event.type}",
    "timestamp": "{event.timestamp}"
  }
```

---

## Practice Exercises

**Exercise 1:** You want to notify a manager when someone joins their team. What event type should you use?

<details>
<summary>Answer</summary>

**Event Type: Identity Created**

Configuration:
```
Event: Identity Created
Filter: cloudLifecycleState equals "active"
Action: Send Email
To: {identity.manager.email}
Subject: New Team Member - {identity.displayName}
Body: [Welcome message with new employee details]
```

Alternatively:
**Event Type: Identity Updated**
If you want to catch when someone's manager field changes to this manager.
</details>

---

**Exercise 2:** You need to alert the security team whenever privileged access is granted. How would you configure this?

<details>
<summary>Answer</summary>
```
Event: Access Granted
Filter: access.privileged equals true

Action: Send Email
To: security@company.com
Subject: ALERT: Privileged Access Granted
Body:
  Privileged access has been granted:
  
  Identity: {identity.displayName}
  Access: {access.name}
  Source: {access.source.name}
  Timestamp: {event.timestamp}
  
  Please review for compliance.
```

Additional action:
- Send webhook to SIEM
- Create audit log entry
- Start approval workflow for verification
</details>

---

**Exercise 3:** What's wrong with this filter: `identity.department == IT`?

<details>
<summary>Answer</summary>

**Missing quotes around the string value.**

❌ **Wrong:**
```
identity.department == IT
```

✅ **Correct:**
```
identity.department == "IT"
```

String values must be in quotes. Without quotes, ISC thinks IT is a variable name, not a string value.
</details>

---

# 1.5.2 Workflow Components and Structure

**Learning Objective:** Understand the building blocks of SailPoint workflows and how they're structured.

---

## What are Workflows?

Workflows are automated business processes built in ISC.

**Think of workflows as:**
- Step-by-step automation
- Business process automation
- Complex logic execution
- Multi-step operations

**Difference from Event Triggers:**
- **Event Trigger:** "When X happens, do Y"
- **Workflow:** "Do steps A, B, C, D in sequence with logic"

**Example workflow:**
```
New Hire Onboarding Workflow:
  1. Create AD account
  2. Wait 5 minutes
  3. Grant email access
  4. Send welcome email
  5. Create onboarding ticket
  6. Notify manager
```

---

## Workflow Components

### **Basic Building Blocks**

**1. Trigger**
- What starts the workflow
- Event trigger, manual start, schedule

**2. Steps**
- Individual actions in workflow
- Execute in sequence
- Can have conditions

**3. Data**
- Variables passed between steps
- Input from trigger
- Output from previous steps

**4. Logic**
- Conditions (if/then)
- Loops (repeat)
- Branches (different paths)

**5. Output**
- Final result
- Data returned
- Status (success/failure)

---

## Workflow Structure

### **Linear Workflow**

Simplest structure - steps execute one after another.
```
Start
  ↓
Step 1: Create account
  ↓
Step 2: Grant access
  ↓
Step 3: Send email
  ↓
End
```

**Example: Simple new hire**
```
Workflow: Basic New Hire Setup

Input: identity object

Steps:
  1. Create AD Account
     Input: identity details
     Output: account created
     
  2. Grant Email Access
     Input: account ID
     Output: access granted
     
  3. Send Welcome Email
     Input: identity email
     Output: email sent

End
```

### **Conditional Workflow**

Workflow with decision points.
```
Start
  ↓
Check: Is employee full-time?
  ├─ Yes → Grant full benefits
  └─ No → Grant contractor access
  ↓
Send appropriate welcome email
  ↓
End
```

**Example: Employee vs Contractor**
```
Workflow: Conditional Onboarding

Input: identity object

Steps:
  1. Check Employee Type
     Condition: identity.employeeType
     
  2a. If "Employee":
      - Grant full access package
      - Assign all roles
      - Enable benefits
      
  2b. If "Contractor":
      - Grant limited access
      - Time-limited access
      - No benefits
      
  3. Send Welcome Email
     Template: Based on employee type

End
```

### **Parallel Workflow**

Multiple steps execute simultaneously.
```
Start
  ↓
Parallel Execution:
  ├─ Create AD account
  ├─ Create Office 365 account
  └─ Create Salesforce account
  ↓
Wait for all to complete
  ↓
Send completion email
  ↓
End
```

**Example: Multi-system provisioning**
```
Workflow: Multi-System Provisioning

Input: identity object

Steps:
  1. Start Parallel Provisioning
     
     Branch A: Active Directory
       - Create account
       - Add to groups
       
     Branch B: Office 365
       - Create mailbox
       - Assign license
       
     Branch C: Salesforce
       - Create user
       - Assign profile
     
  2. Wait for All Branches
  
  3. Check All Succeeded
     - If all successful: Send welcome
     - If any failed: Alert admin

End
```

---

## Common Workflow Steps

### **1. HTTP Request**

Call external APIs.
```
Step: HTTP Request
Type: POST
URL: https://api.company.com/create-user
Headers:
  Authorization: Bearer {token}
  Content-Type: application/json
Body:
  {
    "name": "{identity.name}",
    "email": "{identity.email}",
    "department": "{identity.department}"
  }
Output: API response
```

### **2. Send Email**

Send notifications.
```
Step: Send Email
To: {identity.manager.email}
CC: hr@company.com
Subject: New Team Member - {identity.displayName}
Body:
  Dear {identity.manager.displayName},
  
  {identity.displayName} has joined your team.
  
  Start Date: {identity.startDate}
  Department: {identity.department}
  
  Please ensure they have appropriate access.
```

### **3. Transform**

Manipulate data.
```
Step: Transform
Operation: Concatenate
Input:
  firstName: {identity.firstName}
  lastName: {identity.lastName}
Output:
  fullName: "{firstName} {lastName}"

Or:

Operation: Format Date
Input: {identity.startDate}
Format: "YYYY-MM-DD"
Output: formattedDate
```

### **4. Condition**

Make decisions.
```
Step: Condition
If: {identity.department} == "IT"
Then: Grant admin access
Else: Grant standard access
```

### **5. Loop**

Repeat operations.
```
Step: Loop
For Each: account in {identity.accounts}
Do:
  - Check account status
  - If disabled: Enable it
  - Update last modified
```

### **6. Wait/Delay**

Pause execution.
```
Step: Wait
Duration: 5 minutes

Purpose: Allow AD replication before next step
```

### **7. Approval**

Human decision point.
```
Step: Approval
Approver: {identity.manager}
Subject: Approve access for {identity.name}
Description: Requesting admin access
Timeout: 3 days
If Approved: Continue workflow
If Rejected: End workflow
If Timeout: Escalate to senior manager
```

---

## Workflow Variables

### **Input Variables**

Data passed into workflow when it starts.
```
Workflow: New Hire Onboarding

Input Variables:
  - identity (object): The new employee
  - startDate (date): When they start
  - department (string): Their department
  - manager (object): Their manager
```

### **Step Variables**

Data created during workflow execution.
```
Step 1: Create AD Account
Output: adAccount

Step 2: Generate Email
Input: adAccount.username
Output: emailAddress

Step 3: Send Welcome
Input: emailAddress
```

### **Variable Scope**

**Global variables:**
- Available throughout entire workflow
- Set at workflow start
- Example: identity, requestedBy

**Local variables:**
- Available only in current step/branch
- Created by step
- Example: tempResult, loopCounter

**Example:**
```
Workflow Variables:

Global:
  identity: {full identity object}
  workflowStartTime: "2024-02-12T10:30:00"

Step 1 Local:
  accountCreated: true
  accountID: "CN=jsmith,OU=Users"

Step 2 Local:
  emailSent: true
  messageID: "msg-12345"
```

---

## Error Handling

### **Try-Catch Pattern**

Handle errors gracefully.
```
Step: Try
  Action: Create AD Account
  On Success: Continue to next step
  On Error: Go to catch block

Catch:
  Log error
  Send alert to admin
  Retry with different parameters OR
  Mark workflow as failed
  End gracefully
```

**Example:**
```
Try:
  Create Salesforce Account
  
Catch: If account creation fails
  1. Log error details
  2. Send email to Salesforce admin
  3. Create ticket: "Salesforce account creation failed"
  4. Set workflow status: "Partial Success"
  5. Continue with other steps (don't fail entire workflow)
```

### **Retry Logic**

Automatically retry failed operations.
```
Step: Create Account
Retries: 3
Retry Delay: 30 seconds
Retry On:
  - Timeout
  - Connection error
  - Rate limit

Don't Retry On:
  - Invalid data
  - Account already exists
  - Permission denied
```

---

## Workflow Timing

### **Synchronous Workflows**

Workflow completes before returning.
```
Event Trigger → Start Workflow → Wait for completion → Return

Use When:
  - Quick operations (< 30 seconds)
  - Result needed immediately
  - Simple, linear steps
```

### **Asynchronous Workflows**

Workflow runs in background.
```
Event Trigger → Start Workflow → Return immediately
                     ↓
                Workflow continues
                     ↓
                  Complete

Use When:
  - Long-running operations
  - Multiple parallel steps
  - External approvals needed
  - Result not needed immediately
```

### **Scheduled Workflows**

Workflow runs on schedule.
```
Schedule: Daily at 2:00 AM

Workflow: Daily Account Cleanup
  1. Find inactive accounts
  2. Check last login > 90 days
  3. Disable inactive accounts
  4. Send report to admins
```

---

## Workflow Testing

### **Test Workflow Before Production**

**Testing steps:**
1. Create test workflow (copy of production)
2. Use test data
3. Execute workflow
4. Verify each step
5. Check outputs
6. Review logs
7. Fix issues
8. Repeat until working
9. Deploy to production

**Test checklist:**
```
☐ Workflow triggers correctly
☐ All steps execute in order
☐ Variables populate correctly
☐ Conditions work as expected
☐ Error handling catches failures
☐ Outputs are correct
☐ External systems receive data
☐ Emails sent to right people
☐ Timing is acceptable
☐ Logs show useful information
```

---

## Workflow Best Practices

### **Design Principles**

✅ **Do:**
- Keep workflows simple and focused
- Use descriptive step names
- Comment complex logic
- Handle errors gracefully
- Test thoroughly
- Document workflow purpose
- Use variables instead of hardcoding
- Build modularly (reusable steps)

❌ **Don't:**
- Create overly complex workflows
- Hardcode values
- Ignore error handling
- Skip testing
- Leave workflows undocumented
- Nest workflows too deeply
- Create workflows for everything (sometimes simple is better)

### **Performance Tips**

**Optimize workflow execution:**
```
✅ Good:
  - Use parallel steps where possible
  - Minimize wait times
  - Cache frequently-used data
  - Use conditional logic to skip unnecessary steps

❌ Avoid:
  - Sequential steps that could be parallel
  - Unnecessary waits
  - Repeated API calls for same data
  - Processing one item at a time when batch possible
```

---

## Quick Reference

### **Workflow Components**

| Component | Purpose | Example |
|-----------|---------|---------|
| **Trigger** | Starts workflow | Event, manual, schedule |
| **HTTP Request** | Call external API | Create ticket, update system |
| **Send Email** | Notify people | Manager notification, welcome email |
| **Transform** | Manipulate data | Format names, calculate values |
| **Condition** | Make decisions | If employee then... else... |
| **Loop** | Repeat operations | For each account, do... |
| **Approval** | Human decision | Manager approves access |
| **Wait** | Delay execution | Wait 5 minutes for replication |

### **Workflow Patterns**
```
Linear:
  Step 1 → Step 2 → Step 3 → End

Conditional:
  Start → If/Else → Different paths → End

Parallel:
  Start → Fork → Multiple branches → Join → End

With Approval:
  Start → Steps → Approval → If approved continue → End
```

---

## Practice Exercises

**Exercise 4:** What workflow component would you use to call an external ticketing system API?

<details>
<summary>Answer</summary>

**HTTP Request** component

Configuration:
```
Step: HTTP Request
Method: POST
URL: https://ticketing.company.com/api/tickets
Headers:
  Authorization: Bearer {api_token}
  Content-Type: application/json
Body:
  {
    "title": "New Hire Setup - {identity.name}",
    "description": "Setup access for new employee",
    "assignee": "it-team",
    "priority": "medium"
  }
Output: ticket object (with ticket ID, etc.)
```
</details>

---

**Exercise 5:** You need a workflow that creates AD account, waits for replication, then creates email. What components do you need?

<details>
<summary>Answer</summary>

**Components needed:**

1. **HTTP Request or Provisioning Step**
   - Create AD account
   
2. **Wait/Delay Step**
   - Wait for AD replication (5 minutes)
   
3. **HTTP Request or Provisioning Step**
   - Create email account

**Workflow structure:**
```
Start
  ↓
Create AD Account
  Input: identity details
  Output: adAccount
  ↓
Wait 5 Minutes
  Purpose: Allow AD replication
  ↓
Create Email Account
  Input: adAccount.username
  Output: emailAccount
  ↓
Send Welcome Email
  To: emailAccount.address
  ↓
End
```
</details>

---

# 1.5.3 Common Workflow Patterns

**Learning Objective:** Learn proven workflow patterns for common IAM scenarios.

---

## What are Workflow Patterns?

Workflow patterns are reusable solutions to common automation needs.

**Think of patterns as:**
- Templates for common scenarios
- Proven solutions
- Best practice implementations
- Starting points for customization

**Why use patterns:**
- Save time (don't reinvent)
- Proven to work
- Consistent approach
- Easier to maintain

---

## Joiner Workflow Pattern

### **New Hire Onboarding**

**When to use:** Employee joins company

**Trigger:** Identity Created + cloudLifecycleState = "active"

**Workflow steps:**
```
1. Validate Data
   - Check all required fields present
   - Validate email format
   - Confirm start date
   
2. Create Accounts (Parallel)
   Branch A: Active Directory
     - Generate username
     - Create account
     - Add to default groups
     
   Branch B: Office 365
     - Create mailbox
     - Assign license
     - Configure mailbox settings
     
   Branch C: Other Systems
     - Create accounts in approved systems
     
3. Wait for Account Creation
   - Ensure all parallel branches complete
   
4. Assign Access (Based on Role)
   - If Department = IT:
     → Grant IT access bundle
   - If Department = HR:
     → Grant HR access bundle
   - Else:
     → Grant standard access
     
5. Notifications
   - Send welcome email to employee
   - Notify manager
   - Notify IT for equipment setup
   - Create onboarding ticket
   
6. Schedule Follow-ups
   - 7-day check-in reminder
   - 30-day access review
   - 90-day probation review
   
7. Log Completion
   - Record all accounts created
   - Update HRIS system
   - Mark onboarding complete
```

**Error handling:**
```
If account creation fails:
  - Log error
  - Continue with other accounts
  - Alert admin
  - Create remediation ticket
  
If access assignment fails:
  - Retry 3 times
  - If still fails: alert manager
  - Employee can manually request
```

---

## Mover Workflow Pattern

### **Department Transfer**

**When to use:** Employee changes departments

**Trigger:** Identity Updated + department attribute changed

**Workflow steps:**
```
1. Detect Change
   - Capture old department
   - Capture new department
   - Validate change is legitimate
   
2. Revoke Old Access
   - Remove department-specific access
   - Remove old department groups
   - Keep common access (email, etc.)
   
3. Wait for Revocation
   - Ensure removals complete
   - Verify access actually removed
   
4. Grant New Access
   - Add new department groups
   - Assign department-specific access
   - Apply new access profiles
   
5. Update Systems
   - Update manager if changed
   - Update location if changed
   - Update cost center
   - Update HR system
   
6. Notifications
   - Notify old manager
   - Notify new manager
   - Notify employee
   - Notify IT if equipment change needed
   
7. Schedule Review
   - 30-day access review
   - Ensure transition complete
```

**Special considerations:**
```
Preserve:
  - Email address
  - File access (may need migration)
  - Historical data access
  
Remove:
  - Old department-specific applications
  - Old team access
  - Old location-based access
  
Add:
  - New department applications
  - New team access
  - New location-based access
```

---

## Leaver Workflow Pattern

### **Employee Termination**

**When to use:** Employee leaves company

**Trigger:** cloudLifecycleState changed to "inactive"

**Workflow steps:**
```
1. Immediate Actions (Within 15 minutes)
   Parallel:
     - Disable all accounts
     - Revoke VPN access
     - Disable email (forward to manager)
     - Revoke all privileged access
     
2. Access Removal (Within 24 hours)
   - Revoke all access profiles
   - Remove from all groups
   - Delete OAuth tokens
   - Revoke API access
   
3. Manager Handoff
   - Email access forwarded to manager (30 days)
   - Files transferred to manager
   - Calendar shared with manager
   - Manager notified of all actions
   
4. Create Tickets
   - Equipment return (laptop, badge, etc.)
   - Final paycheck processing
   - Benefits termination
   - Exit interview scheduling
   
5. Data Retention
   - Archive user data per policy
   - Preserve for compliance period
   - Mark for eventual deletion
   
6. Final Cleanup (After retention period)
   - Delete archived data
   - Remove accounts completely
   - Update historical records
   
7. Compliance Logging
   - Record all actions taken
   - Timestamp each step
   - Store audit trail
```

**Security priorities:**
```
Immediate (< 15 min):
  ✓ Disable accounts
  ✓ Revoke remote access
  ✓ Revoke privileged access

Same Day (< 24 hours):
  ✓ Revoke all access
  ✓ Remove group memberships
  ✓ Collect equipment

Within Week:
  ✓ Complete data archival
  ✓ Manager handoff complete
  ✓ Exit interview
```

---

## Access Request Workflow Pattern

### **Standard Access Request**

**When to use:** User requests access

**Trigger:** Access Request Submitted

**Workflow steps:**
```
1. Validate Request
   - Check requester is active
   - Verify access exists
   - Check for conflicts
   - Validate business justification
   
2. Determine Approval Path
   - If Standard Access:
     → Manager approval only
   - If Privileged Access:
     → Manager + Security approval
   - If Sensitive Data:
     → Manager + Data Owner + Compliance
     
3. Send for Approval(s)
   - Notify approvers
   - Set timeout (3 days)
   - Send reminders at 2 days, 1 day
   
4. Process Approval Decision
   
   If Approved:
     → Continue to provisioning
     
   If Rejected:
     → Notify requester
     → Log reason
     → End workflow
     
   If Timeout:
     → Escalate to senior manager
     → Wait for decision
     
5. Provision Access
   - Execute provisioning
   - Verify successful
   - Update identity record
   
6. Notifications
   - Notify requester (approved + provisioned)
   - Notify approvers (for their records)
   - Update ticket system
   
7. Schedule Review
   - Set access review date
   - Create reminder for recertification
```

**Approval matrix example:**
```
Access Type          | Approvers
---------------------|---------------------------
Standard Access      | Manager
Privileged Access    | Manager + Security Team
Financial Systems    | Manager + Finance Director
HR Systems          | Manager + HR Director
PII Data Access     | Manager + Data Owner + Compliance
```

---

## Certification Workflow Pattern

### **Quarterly Access Review**

**When to use:** Regular access certification

**Trigger:** Scheduled (quarterly)

**Workflow steps:**
```
1. Prepare Certification Data
   - Identify all active identities
   - Gather current access
   - Identify reviewers (managers)
   - Generate certification items
   
2. Create Certification Campaign
   - Configure campaign settings
   - Set deadline (14 days)
   - Assign reviewers
   - Generate review items
   
3. Launch Campaign
   - Send notifications to reviewers
   - Provide review instructions
   - Set reminder schedule
   
4. Monitor Progress
   Daily:
     - Check completion rate
     - Send reminders to inactive reviewers
     - Escalate overdue items
     
5. Handle Decisions
   
   Approved Items:
     - No action needed
     - Log approval
     
   Revoked Items:
     - Queue for revocation
     - Execute removal
     - Verify removal successful
     
   No Decision (timeout):
     - Escalate to senior manager
     - Extend deadline if approved
     - Or auto-certify based on policy
     
6. Remediation
   - Execute all revocations
   - Verify completion
   - Handle failures
   - Create remediation tickets
   
7. Campaign Closure
   - Generate completion report
   - Calculate metrics
   - Archive results
   - Schedule next campaign
   
8. Compliance Reporting
   - Export certification data
   - Generate audit report
   - Store for compliance period
```

---

## Orphaned Account Workflow Pattern

### **Uncorrelated Account Remediation**

**When to use:** Orphaned account detected

**Trigger:** Account Uncorrelated event

**Workflow steps:**
```
1. Account Discovered
   - Capture account details
   - Source system
   - Account attributes
   - When created
   
2. Attempt Auto-Correlation
   - Search for matching identity
   - Check email matches
   - Check name matches
   - Check employee ID
   
   If Match Found:
     → Correlate automatically
     → Send notification
     → End workflow
     
3. Manual Review Required
   - Create investigation ticket
   - Assign to source admin
   - Include account details
   - Set SLA (3 days)
   
4. Investigation
   Admin options:
     A. Correlate to existing identity
     B. Delete (not needed)
     C. Keep uncorrelated (service account)
     D. Create new identity
     
5. Execute Decision
   Based on admin choice:
     - Correlate account
     - Delete account
     - Mark as exception
     - Create identity
     
6. Documentation
   - Log decision and reason
   - Update inventory
   - Close ticket
   
7. Prevention
   - If pattern detected:
     → Improve correlation rules
     → Update procedures
     → Train admins
```

---

## Privileged Access Workflow Pattern

### **Temporary Privileged Access**

**When to use:** Need time-limited elevated access

**Trigger:** Privileged Access Request

**Workflow steps:**
```
1. Request Validation
   - Verify requester identity
   - Check business justification
   - Validate duration request
   - Confirm break-glass not needed
   
2. Risk Assessment
   - Check requester's current access
   - Identify conflicts
   - Calculate risk score
   - Determine approval requirements
   
3. Multi-Level Approval
   Level 1: Manager approval
     → Business justification valid?
     
   Level 2: Security team approval
     → Risk acceptable?
     → Mitigation in place?
     
   Level 3: (If high risk) CISO approval
     → Exceptional access justified?
     
4. Grant Access
   - Provision privileged access
   - Set expiration date/time
   - Enable monitoring
   - Log grant action
   
5. Active Monitoring
   While access is active:
     - Monitor usage
     - Alert on suspicious activity
     - Log all actions
     - Send daily reports to security
     
6. Access Expiration
   When time limit reached:
     - Auto-revoke access
     - Verify revocation successful
     - Archive activity logs
     - Generate usage report
     
7. Post-Access Review
   - Review activity during access period
   - Check for policy violations
   - Update risk profile
   - Document for audit
```

---

## Automated Lifecycle Workflow Pattern

### **Continuous Access Validation**

**When to use:** Ongoing access hygiene

**Trigger:** Scheduled (daily)

**Workflow steps:**
```
1. Daily Scan (Runs at 2 AM)
   - Check all active identities
   - Review access assignments
   - Identify anomalies
   
2. Detect Issues
   
   Orphaned Access:
     - User left, access remains
     → Queue for removal
     
   Excessive Access:
     - User has >20 entitlements
     → Alert manager for review
     
   Dormant Access:
     - Access granted, never used (90 days)
     → Consider removal
     
   Stale Access:
     - Access not used in 180 days
     → Queue for review
     
   Conflicting Access:
     - Segregation of duties violation
     → Alert security immediately
     
3. Automated Remediation
   Low Risk Issues (auto-fix):
     - Remove orphaned access
     - Clean up test accounts
     - Remove expired access
     
   Medium Risk Issues (alert):
     - Notify manager of excess
     - Request access review
     - Create review ticket
     
   High Risk Issues (immediate):
     - Alert security team
     - Suspend conflicting access
     - Create incident ticket
     
4. Generate Daily Report
   - Issues found: 127
   - Auto-remediated: 43
   - Awaiting review: 71
   - High priority: 13
   
5. Send Notifications
   - Summary to IAM team
   - High risk to security
   - Review requests to managers
```

---

## Integration Workflow Pattern

### **HR System Integration**

**When to use:** Sync with external HR system

**Trigger:** Scheduled (every 4 hours)

**Workflow steps:**
```
1. Connect to HR System
   - Authenticate to API
   - Verify connection
   - Check last sync timestamp
   
2. Fetch Changed Records
   - Get employees updated since last sync
   - Get new employees
   - Get terminated employees
   - Get department changes
   
3. Process Each Record
   
   For New Employee:
     → Trigger Joiner workflow
     
   For Updated Employee:
     → Update identity attributes
     → Check if department changed
     → If yes: Trigger Mover workflow
     
   For Terminated Employee:
     → Trigger Leaver workflow
     
4. Validate Data
   - Check required fields present
   - Validate email format
   - Verify manager exists
   - Confirm department valid
   
5. Update ISC
   - Create/update identities
   - Trigger appropriate workflows
   - Log all changes
   
6. Error Handling
   If data validation fails:
     - Log error
     - Send to exception queue
     - Alert admin
     - Don't process record
     
7. Reconciliation
   - Compare HR count vs ISC count
   - Identify discrepancies
   - Generate reconciliation report
   
8. Update Sync Timestamp
   - Record successful sync time
   - Use for next sync
```

---

## Quick Reference

### **Pattern Selection Guide**

| Scenario | Pattern | Key Trigger |
|----------|---------|-------------|
| **New employee** | Joiner | Identity Created |
| **Employee transfers** | Mover | Identity Updated (department) |
| **Employee leaves** | Leaver | Lifecycle State → inactive |
| **User requests access** | Access Request | Access Request Submitted |
| **Quarterly review** | Certification | Scheduled |
| **Orphaned account** | Account Remediation | Account Uncorrelated |
| **Temp admin access** | Privileged Access | Privileged Request |
| **Daily cleanup** | Automated Lifecycle | Scheduled daily |
| **HR sync** | Integration | Scheduled (hourly/daily) |

### **Pattern Components**
```
Every pattern should include:
  ✓ Clear trigger
  ✓ Validation step
  ✓ Main processing logic
  ✓ Error handling
  ✓ Notifications
  ✓ Logging/audit
  ✓ Cleanup/completion
```

---

## Practice Exercises

**Exercise 6:** An employee is transferring from IT to HR department. Which workflow pattern should you use?

<details>
<summary>Answer</summary>

**Mover Workflow Pattern**

This pattern handles:
- ✅ Department changes
- ✅ Revoking old department access
- ✅ Granting new department access
- ✅ Updating manager if changed
- ✅ Notifying relevant people

**Trigger:**
```
Event: Identity Updated
Filter: department attribute changed
Workflow: Department Transfer (Mover Pattern)
```

**Key steps:**
1. Capture old and new department
2. Revoke IT department access
3. Grant HR department access
4. Update systems
5. Notify managers
6. Schedule access review
</details>

---

**Exercise 7:** You need a workflow that runs every night to find and remove access that hasn't been used in 6 months. What pattern is this?

<details>
<summary>Answer</summary>

**Automated Lifecycle Workflow Pattern**

Specifically: **Stale Access Cleanup**

**Configuration:**
```
Trigger: Scheduled
Frequency: Daily at 2:00 AM

Workflow: Stale Access Cleanup
Steps:
  1. Scan all identities
  2. Find access not used in 180 days
  3. Check if access is still needed
  4. If dormant:
     - Remove access
     - Log removal
     - Notify manager
  5. Generate report
  6. Send summary to IAM team
```

This is a **scheduled, automated lifecycle** workflow for access hygiene.
</details>

---

**Exercise 8:** Create a simple workflow outline for requesting VPN access that requires manager approval.

<details>
<summary>Answer</summary>

**Access Request Workflow Pattern - VPN Access**
```
Workflow: VPN Access Request

Trigger: Access Request Submitted (VPN access profile)

Steps:
  1. Validate Request
     - Check requester is active employee
     - Verify VPN access profile exists
     - Validate business justification provided
     
  2. Manager Approval
     - Send to requester's manager
     - Subject: "VPN Access Request for {requester.name}"
     - Timeout: 3 days
     - Reminder: After 2 days
     
  3. Process Decision
     
     If Approved:
       → Continue to step 4
       
     If Rejected:
       → Notify requester with reason
       → End workflow
       
     If Timeout:
       → Escalate to senior manager
       → Wait for decision
       
  4. Provision VPN Access
     - Grant VPN access profile
     - Verify provisioning successful
     - Update identity record
     
  5. Send Notifications
     - To requester: "VPN access granted"
     - To manager: "Request approved and provisioned"
     - To IT security: "New VPN user"
     
  6. Schedule Review
     - Set 90-day review reminder
     - Create certification item

End Workflow
```
</details>

---

## Section Summary

You've learned:

✅ **Event Triggers** - Automate responses to ISC events  
✅ **Workflow Components** - Building blocks of automation  
✅ **Workflow Patterns** - Proven solutions for common scenarios  
✅ **Best Practices** - Design, test, and implement workflows effectively  

**Key Patterns Covered:**
- Joiner (new hire)
- Mover (transfers)
- Leaver (terminations)
- Access requests
- Certifications
- Orphaned accounts
- Privileged access
- Automated lifecycle
- System integrations

**Next Steps:**
- Review workflow examples in your tenant
- Identify automation opportunities
- Start with simple event triggers
- Build workflows incrementally
- Test thoroughly before production

---

## Additional Resources

📚 **Official Documentation:**
- [Event Triggers Guide](https://documentation.sailpoint.com/saas/help/workflows/triggers/event_triggers.html)
- [Workflows Documentation](https://documentation.sailpoint.com/saas/help/workflows/index.html)
- [Workflow Steps Reference](https://documentation.sailpoint.com/saas/help/workflows/steps/index.html)

💡 **Pro Tips:**
- Start simple, add complexity gradually
- Document every workflow's purpose
- Test with real scenarios
- Monitor workflow execution
- Keep workflows maintainable
- Reuse proven patterns

---

**End of Section 1.5: Automation and Workflows**

**Next Section:** [1.6 Authentication and Configuration →](#16-authentication-and-configuration)
