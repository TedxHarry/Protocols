# Section 5.1: Provisioning Fundamentals

**Master how provisioning works, what triggers it, and the provisioning lifecycle**

---

## Table of Contents

- [5.1.1 How Provisioning is Triggered](#511-how-provisioning-is-triggered)
- [5.1.2 Provisioning Lifecycle Overview](#512-provisioning-lifecycle-overview)


---

# 5.1.1 How Provisioning is Triggered

**Learning Objective:** Understand the different ways provisioning is initiated in ISC.

---

## What is Provisioning?

**Definition:** The automated process of creating, modifying, or deleting accounts and access in target systems

**Simple concept:**
```
Provisioning = ISC makes changes in target systems

Examples:
  - Create AD account for new employee
  - Add user to VPN group
  - Grant Salesforce license
  - Delete terminated employee's accounts
```

**Key point:**
```
Provisioning is HOW ISC enforces access decisions

Decision: "User needs Salesforce access"
Provisioning: ISC creates Salesforce account
```

**Exam Tip:** Provisioning = ISC executing changes in target systems (create, modify, delete accounts).

---

## Provisioning Triggers Overview

### **Five Main Trigger Types**

| Trigger Type | How Initiated | Example | Use Case |
|--------------|---------------|---------|----------|
| **Lifecycle Change** | Identity lifecycle state changes | active → inactive | Joiner/Leaver |
| **Access Request** | User requests access | Request Salesforce | Self-service |
| **Workflow** | Automated workflow executes | Birthright access | Automated provisioning |
| **Manual** | Admin manually provisions | Admin creates account | Exception handling |
| **Role/Profile Assignment** | Role or access profile assigned | Assign "Sales" role | Role-based access |

---

## Trigger 1: Lifecycle State Changes

### **How It Works**

**Lifecycle change triggers provisioning:**
```
Lifecycle Change → Workflow → Provisioning

Example:
  prehire → active
    ↓
  Joiner Workflow
    ↓
  Provision accounts (AD, email, apps)
```

### **Common Lifecycle Triggers**

**New Hire (prehire → active):**
```
Scenario: Employee starts today

Lifecycle Change:
  cloudLifecycleState: prehire → active
  
Joiner Workflow Triggers:
  1. Create AD account
     Username: john.smith
     Password: Auto-generated
     OU: OU=Users,DC=company,DC=com
     
  2. Create email mailbox
     Email: john.smith@company.com
     License: Exchange Online
     
  3. Provision VPN access
     Add to: VPN-Users group
     
  4. Grant Salesforce access (if in Sales)
     Create account
     Assign license
     Set profile: Standard User
     
  5. Provision department-specific access
     Based on: department attribute
```

**Termination (active → inactive):**
```
Scenario: Employee terminated

Lifecycle Change:
  cloudLifecycleState: active → inactive
  
Leaver Workflow Triggers:
  1. Disable AD account
     Status: Disabled
     Effective: Immediately
     
  2. Disable email
     Forward to: manager
     Auto-reply: Set
     
  3. Revoke VPN access
     Remove from: VPN-Users group
     
  4. Revoke Salesforce access
     Disable account
     Remove license
     
  5. Revoke all other access
     Systematic de-provisioning
```

**Leave of Absence (active → leave):**
```
Scenario: Employee on maternity leave

Lifecycle Change:
  cloudLifecycleState: active → leave
  
Leave Workflow Triggers:
  1. Suspend AD account
     Status: Disabled (not deleted)
     
  2. Suspend email
     Forward to: manager
     
  3. Suspend application access
     Preserve for return
     
  4. Keep benefits access
     Employee needs benefits portal
```

**Return from Leave (leave → active):**
```
Scenario: Employee returns from leave

Lifecycle Change:
  cloudLifecycleState: leave → active
  
Return Workflow Triggers:
  1. Re-enable AD account
     Status: Enabled
     
  2. Re-enable email
     Remove forwarding
     
  3. Restore application access
     Re-activate accounts
     
  4. Welcome back notification
```

---

## Trigger 2: Access Requests

### **How It Works**

**User-initiated access request:**
```
User Request → Approval (if required) → Provisioning

Example:
  User requests Salesforce access
    ↓
  Manager approves
    ↓
  ISC provisions Salesforce account
```

### **Access Request Flow**

**Step-by-step:**
```
1. User logs into ISC
   Navigate to: Request Access
   
2. User browses catalog
   Finds: Salesforce - Sales Cloud User
   
3. User submits request
   Reason: "Need access for new sales role"
   
4. Approval workflow (if configured)
   Routed to: Manager
   Manager: Approves
   
5. Provisioning triggered
   ISC provisions Salesforce account
   
6. User notified
   Email: "Your Salesforce access has been granted"
   
7. User can access
   Log into Salesforce
```

**Example scenarios:**

**Scenario 1: No approval required**
```
Request: VPN Access
Approval: None (auto-approved)

Flow:
  User requests VPN
    ↓
  No approval needed
    ✓ Auto-approved
    ↓
  Provisioning executes immediately
    ✓ Add to VPN-Users group in AD
    ↓
  User notified
    "VPN access granted"
    
Duration: 2-5 minutes
```

**Scenario 2: Manager approval required**
```
Request: Salesforce Access
Approval: Manager

Flow:
  User requests Salesforce
    ↓
  Routed to manager
    Manager reviews
    Manager approves (or denies)
    ↓
  If approved:
    Provisioning executes
    ✓ Create Salesforce account
    ✓ Assign license
    ↓
  User notified
    "Salesforce access granted"
    
Duration: Hours to days (depends on approval)
```

**Scenario 3: Multi-level approval**
```
Request: Admin Access (high-risk)
Approval: Manager + IT Security

Flow:
  User requests admin access
    ↓
  Level 1: Manager approves
    ↓
  Level 2: IT Security reviews
    IT Security approves
    ↓
  Provisioning executes
    ✓ Add to Admin group
    ↓
  User notified
    
Duration: Days (multiple approvals)
```

---

## Trigger 3: Automated Workflows

### **How It Works**

**Workflows provision automatically based on rules:**
```
Condition Met → Workflow → Provisioning

Example:
  New employee in Sales department
    ↓
  Sales onboarding workflow
    ↓
  Provision Salesforce, CRM, sales tools
```

### **Common Workflow Triggers**

**Department-based provisioning:**
```
Trigger: Identity created/updated with department = "Sales"

Workflow: Sales Onboarding
  
Provisions:
  ✓ Salesforce (Sales Cloud)
  ✓ LinkedIn Sales Navigator
  ✓ Zoom
  ✓ Sales-Tools group in AD
  ✓ Sales shared drive access
  
Automatic: No request needed
```

**Role-based provisioning:**
```
Trigger: Identity assigned role = "Developer"

Workflow: Developer Onboarding
  
Provisions:
  ✓ GitHub access
  ✓ JIRA access
  ✓ Dev-Team group in AD
  ✓ VPN access
  ✓ Development servers access
  
Automatic: Based on role
```

**Location-based provisioning:**
```
Trigger: Identity created with location = "New York Office"

Workflow: NY Office Access
  
Provisions:
  ✓ NY-Office WiFi group
  ✓ NY building access badge
  ✓ NY conference room booking
  ✓ Regional shared resources
  
Automatic: Based on location
```

**Birthright access:**
```
Trigger: Identity lifecycle = active

Workflow: Standard Employee Access
  
Provisions (for all active employees):
  ✓ Active Directory account
  ✓ Email
  ✓ VPN
  ✓ Office 365
  ✓ Company intranet
  ✓ Standard applications
  
Automatic: Universal baseline
```

---

## Trigger 4: Manual Provisioning

### **How It Works**

**Admin manually creates provisioning task:**
```
Admin Decision → Manual Provisioning → Account Created

Example:
  Admin manually provisions test account
  Admin handles exception case
  Admin fixes failed auto-provisioning
```

### **When Manual Provisioning Used**

**Use Case 1: Exceptions**
```
Scenario: Contractor needs temporary admin access

Why manual:
  - Exception to normal process
  - High-risk access
  - Temporary (1 week)
  - Doesn't fit automated workflow
  
Process:
  1. Admin receives ticket
  2. Admin verifies authorization
  3. Admin manually provisions access
  4. Admin documents exception
  5. Admin sets expiration reminder
```

**Use Case 2: Troubleshooting**
```
Scenario: Auto-provisioning failed, user needs access urgently

Why manual:
  - Automated workflow failed
  - User needs access now
  - Business critical
  
Process:
  1. Admin investigates why auto-provisioning failed
  2. Admin manually provisions account (workaround)
  3. Admin fixes automated workflow
  4. Prevents future manual interventions
```

**Use Case 3: Testing**
```
Scenario: Testing new application integration

Why manual:
  - Test environment
  - Not in production workflow yet
  - Validation needed
  
Process:
  1. Admin manually creates test account
  2. Admin validates provisioning works
  3. Admin tests account attributes
  4. Later: Automate in workflow
```

**How to manually provision:**
```
In ISC:
  1. Admin → Provisioning
  2. Select identity: John Smith
  3. Select source: Active Directory
  4. Select action: Create Account
  5. Enter attributes:
     - Username: john.smith
     - Groups: VPN-Users, IT-All
  6. Click: Provision
  7. ISC executes provisioning
  8. Verify success
```

**Exam Tip:** Manual provisioning is for exceptions and troubleshooting. Automated provisioning should be the norm.

---

## Trigger 5: Role and Access Profile Assignment

### **How It Works**

**Assigning role/profile triggers provisioning:**
```
Role Assignment → Provisioning of Role Contents

Example:
  Assign "Sales Representative" role to user
    ↓
  ISC provisions all access in that role
    - Salesforce
    - Sales tools
    - CRM access
```

### **Access Profile Assignment**

**What happens:**
```
Access Profile: "Standard Employee Access"

Contains:
  - AD account
  - Email
  - VPN
  - Office 365
  
Assignment:
  Assigned to: All identities with lifecycle = active
  
Provisioning:
  When identity becomes active:
    ✓ Access profile assigned
    ✓ ISC provisions all contents (AD, email, VPN, O365)
    ✓ User has all standard access
```

**Example:**
```
New employee: Sarah Johnson
  cloudLifecycleState: prehire → active
  
Access Profile "Standard Employee Access" assigned:
  (because lifecycle = active)
  
Provisioning triggered:
  1. Create AD account
     Result: sarah.johnson account created
     
  2. Create email
     Result: sarah.johnson@company.com created
     
  3. Grant VPN
     Result: Added to VPN-Users group
     
  4. Provision Office 365
     Result: License assigned
     
All from single access profile assignment
```

### **Role Assignment**

**What happens:**
```
Role: "Sales Representative"

Contains:
  - Access Profile: Salesforce Access
  - Access Profile: Sales Tools
  - Entitlement: Sales-Team AD group
  
Assignment:
  Assign to: Sarah Johnson
  
Provisioning triggered:
  1. Provision Salesforce Access profile
     ✓ Create Salesforce account
     ✓ Assign license
     
  2. Provision Sales Tools profile
     ✓ Grant CRM access
     ✓ Grant LinkedIn Sales Navigator
     
  3. Grant Sales-Team AD group
     ✓ Add to group
     
All from single role assignment
```

---

## Provisioning Trigger Priority

### **What Happens with Multiple Triggers?**

**Scenario: Multiple triggers at once**
```
New employee starts:
  - Lifecycle: prehire → active (Trigger 1)
  - Access profile assigned (Trigger 5)
  - Department = Sales (Trigger 3)
  
All trigger provisioning simultaneously
```

**ISC handling:**
```
ISC consolidates provisioning:
  
Step 1: Collect all provisioning tasks
  - From lifecycle change
  - From access profile
  - From department workflow
  
Step 2: De-duplicate
  - Don't create AD account twice
  - Combine group memberships
  - Consolidate operations
  
Step 3: Execute once
  - Single AD account creation
  - All group memberships added
  - All apps provisioned
  - Efficient execution
```

**Example consolidation:**
```
Triggers:
  1. Lifecycle change: Create AD account
  2. Access profile: Create AD account + add to VPN-Users
  3. Department workflow: Add to Sales-Team
  
ISC consolidates:
  Single operation:
    ✓ Create AD account (once, not three times)
    ✓ Add to groups: VPN-Users, Sales-Team
    
Result:
  - One account created
  - Multiple groups assigned
  - Efficient provisioning
```

---

## Practice Exercise

**Question:** A new employee starts today in the Sales department. Their lifecycle changes from prehire to active. What provisioning is likely triggered and through which trigger types?

<details>
<summary>Answer</summary>

**Scenario Analysis:**

**Employee Details:**
```
Name: John Smith
Start Date: Today (2024-02-14)
Department: Sales
Previous Lifecycle: prehire
Current Lifecycle: active (as of today)
```

**Provisioning Triggered:**

---

**Trigger 1: Lifecycle State Change (prehire → active)**
```
What happens:
  cloudLifecycleState: prehire → active
  
Joiner Workflow executes:
  
1. Create Active Directory Account
   Username: john.smith
   Password: Auto-generated, must change on first login
   OU: OU=Users,DC=company,DC=com
   Enabled: Yes
   
2. Create Email Mailbox
   Email: john.smith@company.com
   Mailbox: 50 GB
   License: Exchange Online Plan 1
   
3. Provision VPN Access
   Action: Add to VPN-Users group in AD
   Result: Can connect to VPN
   
4. Basic Office 365
   License: Microsoft 365 E3
   Apps: Teams, OneDrive, SharePoint
   
5. Company Intranet Access
   Create account in intranet portal
   Grant: Employee self-service

Trigger Type: Lifecycle Change
Automatic: Yes
```

---

**Trigger 2: Access Profile Assignment (Birthright Access)**
```
Access Profile: "Standard Employee Access"

Assigned to: All identities with lifecycle = active

Provisions:
  1. Core AD groups
     - All-Employees
     - Company-Wide-Resources
     
  2. Printing Access
     Add to: Print-Users group
     
  3. WiFi Access
     Add to: WiFi-Users group
     
  4. Standard Applications
     - Zoom (basic license)
     - Slack (standard member)
     - Company knowledge base

Trigger Type: Access Profile Assignment
Automatic: Yes (assigned when lifecycle = active)
```

---

**Trigger 3: Department-Based Workflow (Sales)**
```
Because department = "Sales"

Sales Onboarding Workflow executes:
  
1. Salesforce Provisioning
   Create Salesforce account
   Username: john.smith@company.com
   Profile: Standard User
   License: Sales Cloud
   
2. Sales Tools
   - LinkedIn Sales Navigator (license)
   - SalesLoft (user account)
   - Gong (viewer access)
   
3. Sales-Specific AD Groups
   - Sales-Team (all sales staff)
   - Sales-Shared-Drive (access to sales files)
   - Sales-Distribution-List (email distribution)
   
4. CRM Access
   Create account in CRM
   Assign: Sales representative role
   Territory: TBD by manager

Trigger Type: Automated Workflow (department-based)
Automatic: Yes (department = Sales)
```

---

**Trigger 4: Role Assignment (if applicable)**
```
If John is also assigned role: "Sales Representative"

Role: Sales Representative

Contains:
  - Access Profile: Salesforce Advanced
  - Access Profile: Sales Enablement Tools
  - Entitlement: Regional sales team
  
Additional Provisioning:
  1. Salesforce Advanced Features
     - Salesforce CPQ
     - Advanced reporting
     - Mobile app
     
  2. Sales Enablement
     - Sales training portal
     - Proposal generator
     - Pricing calculator
     
  3. Regional Access
     - Regional shared resources
     - Regional team collaboration

Trigger Type: Role Assignment
Automatic: If role assigned
```

---

**Complete Provisioning Summary:**

**Accounts Created:**
```
1. Active Directory
   ✓ Username: john.smith
   ✓ Groups: VPN-Users, All-Employees, Sales-Team, etc.
   
2. Email
   ✓ john.smith@company.com
   ✓ Exchange Online mailbox
   
3. Salesforce
   ✓ john.smith@company.com
   ✓ Sales Cloud license
   
4. Sales Tools
   ✓ LinkedIn Sales Navigator
   ✓ SalesLoft
   ✓ CRM system
   
5. Standard Apps
   ✓ Zoom
   ✓ Slack
   ✓ Office 365
```

**Timeline:**
```
2:00 AM: Workday aggregation runs
  - Detects start date = today
  - Lifecycle changes: prehire → active
  
2:05 AM: Joiner workflow triggers
  - Creates AD account
  - Creates email
  - Provisions VPN
  
2:10 AM: Access profiles assigned
  - Standard Employee Access provisions
  - Birthright access granted
  
2:15 AM: Department workflow triggers
  - Sales onboarding provisions
  - Salesforce created
  - Sales tools provisioned
  
2:30 AM: All provisioning complete
  - Total duration: ~30 minutes
  - All access ready

8:00 AM: John arrives for first day
  ✓ Can log into computer (AD account works)
  ✓ Email is ready
  ✓ Salesforce access available
  ✓ All tools accessible
  ✓ Smooth first-day experience
```

**Trigger Type Summary:**
```
Trigger Types Used:

1. ✓ Lifecycle Change (prehire → active)
   - Primary trigger
   - Joiner workflow
   - Core account creation
   
2. ✓ Access Profile Assignment
   - Birthright access
   - Standard employee access
   - Automatic based on lifecycle
   
3. ✓ Automated Workflow (Department = Sales)
   - Sales-specific provisioning
   - Salesforce and sales tools
   - Automatic based on department
   
4. Possibly: Role Assignment
   - If "Sales Representative" role assigned
   - Additional sales access
   
5. ✗ Access Request
   - Not used (automatic provisioning)
   - User didn't request anything
   - All granted automatically
   
6. ✗ Manual Provisioning
   - Not needed (automation worked)
   - Everything provisioned automatically
```

**What John Didn't Have to Do:**
```
✗ Request AD account (automatic)
✗ Request email (automatic)
✗ Request Salesforce (automatic - department based)
✗ Request VPN (automatic - birthright)
✗ Request any standard access (all automatic)

Result:
  - Zero-touch onboarding
  - Everything ready day one
  - No manual requests needed
  - No IT tickets required
```

**Exam Tip:** New hire provisioning typically uses multiple triggers: lifecycle change (primary), access profiles (birthright), and department-based workflows (role-specific). Consolidates into single efficient provisioning operation.
</details>

---

# 5.1.2 Provisioning Lifecycle Overview

**Learning Objective:** Understand the complete lifecycle of a provisioning operation from trigger to completion.

---

## Provisioning Lifecycle Phases

### **Six Phases**
```
1. Trigger → 2. Planning → 3. Execution → 4. Verification → 5. Notification → 6. Audit
```

**Visual flow:**
```
Trigger Event
  ↓
ISC Plans Provisioning
  ↓
ISC Executes on Target System
  ↓
ISC Verifies Success
  ↓
ISC Notifies Stakeholders
  ↓
ISC Logs for Audit
```

---

## Phase 1: Trigger

**What happens:**
```
Event occurs that requires provisioning

Examples:
  - Lifecycle change (prehire → active)
  - Access request approved
  - Workflow executes
  - Manual admin action
  - Role assigned
```

**ISC detects:**
```
Event: New employee lifecycle = active

ISC determines:
  - What needs to be provisioned?
  - Which sources/systems?
  - Which accounts to create?
  - Which access to grant?
```

---

## Phase 2: Planning

**What happens:**
```
ISC determines exactly what provisioning operations to perform
```

**Planning steps:**

**Step 1: Identify Target Sources**
```
Determine which sources need provisioning:
  ✓ Active Directory (create account)
  ✓ Exchange Online (create mailbox)
  ✓ Salesforce (create account)
  ✓ VPN (add to group)
```

**Step 2: Determine Operations**
```
For each source, what operation:
  
Active Directory:
  Operation: Create
  Account: john.smith
  Attributes: firstName, lastName, email, department
  Groups: VPN-Users, All-Employees
  
Salesforce:
  Operation: Create
  Username: john.smith@company.com
  License: Sales Cloud
  Profile: Standard User
```

**Step 3: Check Prerequisites**
```
Verify prerequisites met:
  ✓ Identity has required attributes (email, employeeId)
  ✓ Source is available (not in maintenance)
  ✓ Provisioning enabled for this identity
  ✓ No conflicting operations in progress
```

**Step 4: Consolidate Operations**
```
If multiple triggers:
  Combine into single efficient operation
  Avoid duplicates
  Optimize execution
```

**Step 5: Create Provisioning Plan**
```
Provisioning Plan Created:
  
  Task 1: Create AD Account
    Source: Active Directory
    Operation: Create
    Username: john.smith
    Attributes: [...]
    Groups: [...]
    Priority: High
    
  Task 2: Create Email
    Source: Exchange Online
    Operation: Create
    Email: john.smith@company.com
    Mailbox size: 50 GB
    Priority: High
    
  Task 3: Create Salesforce Account
    Source: Salesforce
    Operation: Create
    Username: john.smith@company.com
    License: Sales Cloud
    Priority: Medium
```

---

## Phase 3: Execution

**What happens:**
```
ISC executes provisioning plan on target systems
```

**Execution steps:**

**Step 1: Queue Provisioning**
```
Provisioning tasks queued:
  - Organized by priority
  - Grouped by source
  - Scheduled for execution
```

**Step 2: Execute on Target System**
```
For each provisioning task:
  
AD Account Creation:
  1. ISC sends command to VA (if on-prem)
  2. VA connects to Active Directory
  3. VA executes: New-ADUser cmdlet
  4. Account created in AD
  5. Groups assigned
  6. VA reports success to ISC
  
OR (for cloud sources):
  1. ISC calls API directly
  2. Salesforce API: Create user
  3. Salesforce responds: Success
  4. ISC records success
```

**Step 3: Handle Dependencies**
```
Some provisioning has dependencies:
  
Example: Email requires AD account
  1. Create AD account first
  2. Wait for AD account creation success
  3. Then create email (uses AD account)
  
ISC manages dependencies automatically
```

**Step 4: Retry on Failure**
```
If provisioning fails:
  1. ISC logs error
  2. ISC retries (if configured)
  3. Retry count: 1, 2, 3
  4. If still fails after retries:
     - Mark as failed
     - Alert admin
     - User notified
```

**Execution methods:**

**Real-time (immediate):**
```
Used for: High-priority provisioning
  - New hire day one
  - Security revocations
  - Manager-approved requests
  
Execution: Immediately
Duration: Minutes
```

**Batch (scheduled):**
```
Used for: Lower-priority, bulk operations
  - Bulk role assignments
  - Periodic group updates
  - Non-urgent changes
  
Execution: Next batch window (e.g., hourly)
Duration: Up to 1 hour delay
```

---

## Phase 4: Verification

**What happens:**
```
ISC verifies provisioning succeeded
```

**Verification methods:**

**Method 1: Immediate Confirmation**
```
Target system responds with success/failure:
  
Salesforce Create User API:
  Request: Create user john.smith@company.com
  Response: 
    {
      "id": "005xx000001234",
      "success": true,
      "username": "john.smith@company.com"
    }
    
ISC records: ✓ Provisioning successful
```

**Method 2: Subsequent Aggregation**
```
Provisioning executed:
  - AD account created
  - ISC marks: Pending verification
  
Next AD aggregation (e.g., daily):
  - ISC aggregates AD
  - Finds: john.smith account
  - Verifies: Account exists
  - Updates: ✓ Provisioning confirmed
```

**Method 3: Error Detection**
```
Provisioning fails:
  Error: "User already exists"
  
ISC records:
  ✗ Provisioning failed
  Error code: ENTRY_ALREADY_EXISTS
  Message: "Account john.smith already exists in AD"
  
ISC can:
  - Retry with different username
  - Alert admin
  - Correlate existing account
```

---

## Phase 5: Notification

**What happens:**
```
ISC notifies relevant parties about provisioning results
```

**Notifications sent:**

**To User:**
```
Email: "Your Salesforce access has been granted"

Content:
  "Hi John,
  
  Your request for Salesforce access has been approved and provisioned.
  
  Details:
  - Username: john.smith@company.com
  - URL: https://company.salesforce.com
  - Initial password sent separately
  
  If you have any issues, contact IT support.
  
  Best regards,
  IT Team"
```

**To Manager:**
```
Email: "Access granted to your direct report"

Content:
  "Hi Manager,
  
  Salesforce access has been granted to John Smith.
  
  Access details:
  - System: Salesforce
  - Profile: Standard User
  - Granted: 2024-02-14
  
  This access was auto-approved based on department.
  
  If you have concerns, please contact IT."
```

**To Admin (if needed):**
```
Email: "Provisioning failed - action required"

Content:
  "Provisioning Task Failed
  
  Identity: John Smith
  Target: Active Directory
  Operation: Create Account
  Error: Account name already exists
  
  Action Required:
  - Review conflicting account
  - Resolve and retry provisioning
  - Ticket: PROV-12345"
```

**To Audit Log:**
```
Audit Entry:
  Timestamp: 2024-02-14 10:30:15
  Action: Account Provisioned
  Identity: John Smith (EMP12345)
  Target: Salesforce
  Operation: Create
  Requested by: Workflow (Joiner)
  Status: Success
  Account: john.smith@company.com
```

---

## Phase 6: Audit and Logging

**What happens:**
```
ISC records all provisioning activity for compliance and troubleshooting
```

**What's logged:**

**Provisioning Event:**
```
Event Log Entry:
  
Event ID: PROV-20240214-001
Date/Time: 2024-02-14 10:30:15
Type: Provisioning
Action: Create Account
Identity: John Smith
  Employee ID: EMP12345
  Email: john.smith@company.com
Target System: Active Directory
Operation: Create
Account Created: john.smith
Status: Success
Triggered By: Lifecycle Change (prehire → active)
Executed By: System (Joiner Workflow)
Duration: 15 seconds
Retries: 0
```

**Attributes Provisioned:**
```
Account Attributes:
  sAMAccountName: john.smith
  displayName: John Smith
  mail: john.smith@company.com
  department: Sales
  manager: Jane Doe
  employeeID: EMP12345
  
Groups Added:
  - VPN-Users
  - All-Employees
  - Sales-Team
```

**Change History:**
```
Identity: John Smith

Change History:
  2024-02-14 10:30:15 - Account created: AD (john.smith)
  2024-02-14 10:32:45 - Account created: Salesforce
  2024-02-14 10:35:12 - Group added: VPN-Users
  2024-02-14 10:35:13 - Group added: Sales-Team
```

---

## Complete Lifecycle Example

### **End-to-End Provisioning Flow**

**Scenario: New employee starts**
Timeline: New hire provisioning from trigger to completion
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
PHASE 1: TRIGGER (02:00:00)
Event: Workday aggregation completes

Employee: John Smith
Start date: Today
Status: Active
Lifecycle: prehire → active

Trigger detected: Lifecycle state change
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
PHASE 2: PLANNING (02:00:15 - 02:00:45)
ISC Planning Engine:
Step 1: Identify what needs provisioning
✓ Joiner workflow applies
✓ Access profiles assigned (birthright)
✓ Department = Sales (sales workflow)
Step 2: Determine target systems
✓ Active Directory
✓ Exchange Online
✓ Salesforce
✓ Slack
✓ Zoom
Step 3: Create provisioning plan
Task 1: AD Account (priority: high)
Task 2: Email (priority: high, depends on AD)
Task 3: Salesforce (priority: medium)
Task 4: Slack (priority: low)
Task 5: Zoom (priority: low)
Step 4: Verify prerequisites
✓ Identity has email: john.smith@company.com
✓ Identity has employeeID: EMP12345
✓ All sources available
✓ No conflicts detected
Provisioning plan created: 5 tasks
Duration: 30 seconds
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
PHASE 3: EXECUTION (02:01:00 - 02:05:30)
Task 1: Active Directory Account (02:01:00)
✓ ISC sends command to VA
✓ VA creates AD account
✓ Username: john.smith
✓ Groups: VPN-Users, All-Employees, Sales-Team
✓ Success (15 seconds)
Task 2: Exchange Online Mailbox (02:01:30)
✓ ISC calls Microsoft Graph API
✓ Mailbox created
✓ Email: john.smith@company.com
✓ License: Exchange Online Plan 1
✓ Success (45 seconds)
Task 3: Salesforce Account (02:02:30)
✓ ISC calls Salesforce API
✓ User created
✓ Username: john.smith@company.com
✓ License: Sales Cloud
✓ Profile: Standard User
✓ Success (30 seconds)
Task 4: Slack Account (02:03:15)
✓ ISC calls Slack API
✓ User created
✓ Email: john.smith@company.com
✓ User type: Member
✓ Success (20 seconds)
Task 5: Zoom Account (02:03:45)
✓ ISC calls Zoom API
✓ User created
✓ Email: john.smith@company.com
✓ License: Basic
✓ Success (15 seconds)
All tasks completed successfully
Total duration: 4 minutes 30 seconds
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
PHASE 4: VERIFICATION (02:05:30 - 02:06:00)
ISC verifies each provisioning:
✓ AD account: Success response received
✓ Email: Success response received
✓ Salesforce: Account ID returned
✓ Slack: User ID returned
✓ Zoom: User created confirmation
All provisioning verified successful
Duration: 30 seconds
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
PHASE 5: NOTIFICATION (02:06:00 - 02:06:30)
Notifications sent:
✓ Email to john.smith@company.com (personal)
Subject: "Welcome to Company - Your Accounts Are Ready"
✓ Email to manager (jane.doe@company.com)
Subject: "John Smith accounts provisioned"
✓ Email to IT (if configured)
Subject: "New hire provisioning complete"
✓ Slack notification to #new-hires channel
"Welcome John Smith to the team!"
Duration: 30 seconds
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
PHASE 6: AUDIT (02:06:30)
Audit logs created:
✓ Provisioning event logged
✓ All accounts recorded
✓ All attributes documented
✓ Timeline captured
✓ Compliance record created
Available for:

Compliance reporting
Access reviews
Troubleshooting
Audit trail

Duration: Immediate (async logging)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
COMPLETE
Total Time: 6 minutes 30 seconds
Trigger detection: Immediate
Planning: 30 seconds
Execution: 4 minutes 30 seconds
Verification: 30 seconds
Notification: 30 seconds
Audit: Async
Status: ✓ All provisioning successful
Result: John ready for day one at 8:00 AM

**Exam Tip:** Provisioning lifecycle has six phases: Trigger, Planning, Execution, Verification, Notification, Audit. Typical duration: 5-10 minutes for standard provisioning.
