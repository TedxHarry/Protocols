# 4.4.3 Provisioning Options within Lifecycle States

**Learning Objective:** Understand how lifecycle states control provisioning behavior in ISC.

---

## Lifecycle State and Provisioning

### **Core Concept**

**Provisioning is controlled by lifecycle state:**
```
Lifecycle state determines:
  - Can accounts be provisioned?
  - Can users request access?
  - What provisioning policies apply?
  - When de-provisioning occurs?
```

**Example:**
```
Lifecycle = active:
  ✓ Provisioning enabled
  ✓ User can request access
  ✓ Automated provisioning works
  
Lifecycle = inactive:
  ✗ Provisioning disabled
  ✗ User cannot request access
  ✗ De-provisioning triggered
```

**Exam Tip:** Lifecycle state acts as a gate for provisioning. Active = provision, Inactive = de-provision.

---

## Provisioning Configuration per Lifecycle State

### **Configuration Options**

**For each lifecycle state, configure:**

| Setting | Purpose | Options |
|---------|---------|---------|
| **Provisioning Enabled** | Can accounts be created? | Yes/No |
| **De-provisioning Action** | What happens on exit? | Disable/Delete/None |
| **Access Requests** | Can user request access? | Enabled/Disabled |
| **Auto-Provisioning** | Automatic provisioning? | Enabled/Disabled |

---

## State-Specific Provisioning Behavior

### **Lifecycle: prehire**

**Typical Configuration:**
```
Provisioning Enabled: Limited
De-provisioning on Exit: None
Access Requests: Disabled
Auto-Provisioning: Selective

Rationale:
  - Not yet started, limited needs
  - Prepare for day one
  - Don't provision full access yet
```

**Common Provisioning Patterns:**

**Pattern 1: No Provisioning (Most Common)**
```
Configuration:
  Provisioning Enabled: No
  
Result:
  - No accounts created
  - No access provisioned
  - Identity exists for tracking
  - Wait until lifecycle = active
  
Use case:
  - Standard pre-boarding
  - No early access needed
  - All provisioning on start date
```

**Pattern 2: Selective Provisioning**
```
Configuration:
  Provisioning Enabled: Conditional
  
What gets provisioned:
  ✓ HR system access (for forms)
  ✓ Learning management (pre-training)
  ✓ Equipment assignment (laptop order)
  
What does NOT get provisioned:
  ✗ Active Directory account
  ✗ Email
  ✗ Business applications
  
Use case:
  - Pre-boarding tasks
  - Complete paperwork before start
  - Early training
```

**Pattern 3: Pre-Created, Disabled**
```
Configuration:
  Provisioning Enabled: Yes (but accounts disabled)
  
What happens:
  ✓ AD account created (disabled)
  ✓ Email created (disabled)
  ✓ Groups assigned
  ✓ Ready for day one
  
On start date (prehire → active):
  ✓ Accounts enabled (not created)
  ✓ Immediate access
  ✓ Faster onboarding
  
Use case:
  - IT roles with complex setup
  - C-level executives
  - Critical day-one access
```

**Transition: prehire → active**
```
When lifecycle changes to active:
  
If Pattern 1 (No Provisioning):
  ✓ Joiner workflow creates all accounts
  ✓ All access provisioned
  ✓ Takes 5-15 minutes
  
If Pattern 2 (Selective):
  ✓ Joiner workflow provisions remaining access
  ✓ AD, email, apps created
  ✓ Pre-existing access retained
  
If Pattern 3 (Pre-Created):
  ✓ Joiner workflow enables accounts
  ✓ Immediate access (already created)
  ✓ Takes 1-2 minutes
```

---

### **Lifecycle: active**

**Typical Configuration:**
```
Provisioning Enabled: Yes (Full)
De-provisioning on Exit: Varies
Access Requests: Enabled
Auto-Provisioning: Enabled

Rationale:
  - Normal operations
  - Full provisioning capability
  - Users can request access
```

**Provisioning Behavior:**

**Account Provisioning:**
```
When lifecycle = active:
  
Automatic provisioning:
  ✓ Joiner workflow (if new to active)
  ✓ Birthright access profiles
  ✓ Role-based assignments
  ✓ Department-based access
  
Manual provisioning:
  ✓ User submits access request
  ✓ Approval workflow (if required)
  ✓ Provisioning executes
  ✓ User granted access
  
Lifecycle movements (within active):
  ✓ Department change → Mover workflow
  ✓ Role change → Access adjustment
  ✓ Promotion → Additional access
```

**Access Request Capability:**
```
User experience:
  1. User logs into ISC
  2. Navigates to "Request Access"
  3. Browses available access
  4. Selects desired access
  5. Submits request
  6. Approval workflow (if needed)
  7. Provisioning executes
  8. User receives access
  
Available because lifecycle = active
```

**De-provisioning Triggers:**
```
When leaving active state:

active → leave:
  Action: Suspend (not delete)
  - Accounts disabled
  - Access revoked
  - Data preserved
  
active → inactive:
  Action: Disable or Delete
  - Accounts disabled immediately
  - Data archived
  - Scheduled deletion (30-90 days)
```

---

### **Lifecycle: leave**

**Typical Configuration:**
```
Provisioning Enabled: No
De-provisioning on Exit: Suspend/Disable
Access Requests: Disabled
Auto-Provisioning: Disabled

Rationale:
  - Temporary absence
  - No access during leave
  - Preserve for return
```

**Provisioning Behavior:**

**On Entry (active → leave):**
```
Leave workflow executes:
  
Account Actions:
  ✓ Disable accounts (don't delete)
  ✓ Revoke access
  ✓ Suspend licenses
  ✓ Preserve group memberships
  
Email Actions:
  ✓ Disable access
  ✓ Forward to manager
  ✓ Set auto-reply
  
Applications:
  ✓ Suspend licenses (save cost)
  ✓ Disable access
  ✓ Preserve data
```

**During Leave (lifecycle = leave):**
```
Provisioning disabled:
  ✗ Cannot provision new access
  ✗ Cannot request access
  ✗ No automatic provisioning
  
Why:
  - User not working
  - Shouldn't have access
  - On temporary absence
```

**On Exit (leave → active):**
```
Return workflow executes:
  
Account Actions:
  ✓ Re-enable accounts
  ✓ Restore access
  ✓ Re-activate licenses
  ✓ Restore group memberships
  
Email Actions:
  ✓ Enable access
  ✓ Remove forwarding
  ✓ Remove auto-reply
  
Applications:
  ✓ Re-activate licenses
  ✓ Restore access
  ✓ Same access as before
```

**Key Point - Preserve vs Delete:**
```
Leave state preserves everything:
  ✓ Accounts suspended (not deleted)
  ✓ Groups retained
  ✓ Data preserved
  ✓ Easy to restore
  
Why not delete:
  - Temporary absence
  - Will return
  - Restoration should be seamless
  - Preserve context
```

---

### **Lifecycle: inactive**

**Typical Configuration:**
```
Provisioning Enabled: No
De-provisioning on Exit: N/A (permanent state)
Access Requests: Disabled
Auto-Provisioning: Disabled

Rationale:
  - Terminated employee
  - No access permitted
  - Permanent de-provisioning
```

**Provisioning Behavior:**

**On Entry (active → inactive):**
```
Leaver workflow executes:
  
Immediate Actions (within minutes):
  ✓ Disable AD account
  ✓ Revoke VPN access
  ✓ Disable email (forward to manager)
  ✓ Disable building access
  ✓ Revoke application access
  ✓ Disable all systems
  
Same Day:
  ✓ Manager handoff (email, files)
  ✓ Equipment collection
  ✓ Access reviews updated
  ✓ Certifications updated
```

**De-provisioning Schedule:**
```
Typical timeline:

Day 0 (Termination):
  - All accounts DISABLED
  - Access REVOKED
  - Immediate security
  
Day 1-30:
  - Data archived
  - Email converted to shared mailbox
  - Files transferred to manager
  - Equipment processed
  
Day 30-90:
  - Accounts DELETED (per policy)
  - Retention period complete
  - Final cleanup
  
Day 90+:
  - All accounts removed
  - Only ISC identity preserved (audit)
```

**During Inactive State:**
```
Provisioning locked:
  ✗ Cannot provision access
  ✗ Cannot request access
  ✗ No access to any systems
  ✗ Cannot authenticate
  
Identity preserved:
  ✓ ISC identity retained
  ✓ Audit trail maintained
  ✓ Historical access visible
  ✓ Compliance reporting
```

**De-provisioning Configuration Options:**

**Option 1: Disable Only**
```
Configuration:
  De-provision Action: Disable
  
Result:
  - Accounts disabled
  - Accounts NOT deleted
  - Preserved indefinitely
  - Manual deletion required
  
Use case:
  - Compliance requires long retention
  - May need to re-enable
  - Manual cleanup process
```

**Option 2: Scheduled Deletion**
```
Configuration:
  De-provision Action: Delete
  Schedule: 90 days after termination
  
Result:
  Day 0: Accounts disabled
  Day 90: Accounts deleted
  
Automated process:
  ✓ Consistent enforcement
  ✓ Reduces account sprawl
  ✓ Compliance-aligned
  
Use case:
  - Standard practice
  - Automated cleanup
  - Balance retention and security
```

**Option 3: Immediate Deletion**
```
Configuration:
  De-provision Action: Delete Immediately
  
Result:
  - Accounts disabled AND deleted
  - Same day removal
  - No retention period
  
Use case:
  - Termination for cause
  - Security incident
  - Legal requirement
  - High-risk separation
```

---

## Provisioning Policy by Lifecycle

### **Access Profile Assignment**

**Configuration determines what gets provisioned:**

**Access Profile: "Standard Employee Access"**
```
Assigned to:
  Lifecycle States: active
  
Contains:
  - Active Directory account
  - Email
  - VPN access
  - Office apps
  
Provisioning Behavior:
  When lifecycle = active: ✓ Provision
  When lifecycle = prehire: ✗ Don't provision
  When lifecycle = leave: ✗ Don't provision
  When lifecycle = inactive: ✗ Don't provision
```

**Access Profile: "Pre-Hire Access"**
```
Assigned to:
  Lifecycle States: prehire
  
Contains:
  - HR portal access
  - Training portal
  - Document portal
  
Provisioning Behavior:
  When lifecycle = prehire: ✓ Provision
  When lifecycle = active: Auto-removed (may keep or revoke)
  When lifecycle = leave: ✗ Don't provision
  When lifecycle = inactive: ✗ Don't provision
```

**Access Profile: "Employee Benefits"**
```
Assigned to:
  Lifecycle States: active, leave
  
Contains:
  - Benefits portal
  - 401k access
  - Health insurance portal
  
Provisioning Behavior:
  When lifecycle = active: ✓ Provision
  When lifecycle = leave: ✓ Keep access (for benefits management)
  When lifecycle = inactive: ✗ Remove (per policy)
```

---

## Access Request Behavior

### **Request Capability by Lifecycle**

**Lifecycle = active:**
```
User can:
  ✓ Browse access catalog
  ✓ Submit access requests
  ✓ View pending requests
  ✓ Self-service access
  
Process:
  1. User logs in
  2. Request Access → Browse
  3. Select desired access
  4. Submit request
  5. Approval workflow (if required)
  6. Provisioning executes
  7. Access granted
```

**Lifecycle = prehire:**
```
User can:
  ✗ Cannot request access (typically)
  ✗ No catalog access
  
Why:
  - Not started yet
  - No business need
  - Wait until active
  
Exception:
  Some organizations allow limited requests
  Example: Request laptop configuration
```

**Lifecycle = leave:**
```
User can:
  ✗ Cannot request access
  ✗ Cannot authenticate (suspended)
  
Why:
  - On leave, not working
  - Shouldn't need new access
  - Access suspended
```

**Lifecycle = inactive:**
```
User can:
  ✗ Cannot request access
  ✗ Cannot authenticate
  
Why:
  - Terminated
  - No access permitted
  - Security requirement
```

---

## Conditional Provisioning

### **Lifecycle-Based Rules**

**Rule Example 1: Email Provisioning**
```
Provisioning Policy: Email Account

Condition:
  IF lifecycle = "active" OR lifecycle = "leave"
    THEN provision email
  ELSE
    Do not provision
    
Behavior:
  prehire: No email (wait until active)
  active: ✓ Email provisioned
  leave: ✓ Email kept (forwarded to manager)
  inactive: ✗ Email disabled/deleted
```

**Rule Example 2: VPN Access**
```
Provisioning Policy: VPN Access

Condition:
  IF lifecycle = "active"
    THEN provision VPN
  ELSE
    Revoke VPN
    
Behavior:
  prehire: No VPN
  active: ✓ VPN provisioned
  leave: ✗ VPN revoked (security)
  inactive: ✗ VPN revoked
```

**Rule Example 3: Salesforce License**
```
Provisioning Policy: Salesforce License

Condition:
  IF lifecycle = "active" AND department = "Sales"
    THEN provision Salesforce
  ELSE
    Revoke Salesforce
    
Behavior:
  prehire (Sales): No Salesforce yet
  active (Sales): ✓ Salesforce provisioned
  active (IT): ✗ Not provisioned (wrong dept)
  leave: ✗ License revoked (save cost)
  inactive: ✗ License revoked
```

---

## Practice Exercise

**Question:** An employee goes on leave. They need to continue accessing the benefits portal to manage insurance during leave, but all other access should be suspended. How would you configure provisioning for the leave lifecycle state?

<details>
<summary>Answer</summary>

**Configuration Solution:**

**Lifecycle State: leave**

**Overall Configuration:**
```
Admin → Identities → Identity Profiles → Employee Profile

Lifecycle State: leave
  Provisioning Enabled: Conditional
  Access Requests: Disabled
  Auto-Provisioning: Selective
```

**Access Profile 1: Standard Employee Access**
```
Name: Standard Employee Access

Contains:
  - Active Directory account
  - Email
  - VPN
  - Slack
  - Office 365
  - Other standard apps

Lifecycle Assignment:
  Assigned to: active
  NOT assigned to: leave
  
Provisioning Behavior:
  When lifecycle = active: ✓ Provisioned
  
  When lifecycle changes active → leave:
    ✗ De-provision (disable accounts)
    ✗ Revoke access
    
  Specific actions on leave:
    - AD account: DISABLED
    - Email: DISABLED (forwarded to manager)
    - VPN: REVOKED
    - Slack: DISABLED
    - Office 365: DISABLED
```

**Access Profile 2: Employee Benefits Access**
```
Name: Employee Benefits Access

Contains:
  - Benefits portal account
  - 401k portal access
  - Health insurance portal

Lifecycle Assignment:
  Assigned to: active, leave
  
Provisioning Behavior:
  When lifecycle = active: ✓ Provisioned
  
  When lifecycle changes active → leave:
    ✓ KEEP access (don't de-provision)
    ✓ Accounts remain active
    
  When lifecycle changes leave → active:
    ✓ Continue access (no change)
    
  When lifecycle changes to inactive:
    ✗ De-provision (per policy)
```

**Detailed Workflow Configuration:**

**Leave Workflow (active → leave):**
```
Trigger: Lifecycle change from active to leave

Actions:

1. Disable Standard Access
   For each access in "Standard Employee Access" profile:
     ✓ Disable AD account
     ✓ Disable email (forward to manager)
     ✓ Revoke VPN
     ✓ Disable Slack
     ✓ Disable Office 365
     ✓ Revoke application access

2. Preserve Benefits Access
   For each access in "Employee Benefits Access" profile:
     ✓ SKIP (do not disable)
     ✓ Keep benefits portal active
     ✓ Keep 401k access active
     ✓ Keep health insurance portal active

3. Notifications
   ✓ Email manager: "Employee on leave, email forwarded"
   ✓ Email HR: "Leave workflow completed"
   ✓ Email employee (at personal email):
     "Your work access is suspended.
      Benefits portal remains available at:
      https://benefits.company.com
      Contact HR if you need assistance."
```

**Technical Implementation:**

**Option 1: Separate Access Profiles (Recommended)**
```
Why this works:
  ✓ Benefits profile assigned to both active AND leave
  ✓ Standard profile assigned to active ONLY
  ✓ Lifecycle controls provisioning automatically
  ✓ Clean separation of concerns

Configuration:
  Access Profile: Standard Employee Access
    Lifecycle: active
    
  Access Profile: Employee Benefits Access
    Lifecycle: active, leave
```

**Option 2: Conditional Provisioning Rules**
```
Alternative approach using rules:

Provisioning Policy: Benefits Portal

Rule:
  IF lifecycle IN ["active", "leave"]
    THEN provision
  ELSE
    de-provision
    
Provisioning Policy: Active Directory

Rule:
  IF lifecycle = "active"
    THEN provision
  ELSE
    de-provision (disable)
```

**Option 3: Lifecycle-Specific De-provisioning**
```
Leave Workflow Configuration:

De-provisioning Exceptions:
  Do NOT de-provision:
    - Benefits Portal
    - 401k Portal
    - Health Insurance Portal
    
  De-provision everything else:
    - AD account
    - Email
    - VPN
    - Applications
```

**User Experience:**

**During Leave:**
```
Employee attempts to access work systems:

Work Email (Outlook):
  ✗ Cannot log in
  Message: "Account disabled"
  
VPN:
  ✗ Cannot connect
  Message: "Access denied"
  
Slack:
  ✗ Cannot access
  Message: "Account deactivated"

Benefits Portal:
  ✓ CAN log in
  ✓ Full access
  ✓ Manage insurance
  ✓ Update beneficiaries
  ✓ View 401k
  
Employee experience:
  - Work access blocked (expected)
  - Benefits access works (as needed)
  - Can manage benefits during leave
```

**Return from Leave (leave → active):**
```
Return Workflow:

1. Re-enable Standard Access
   ✓ Enable AD account
   ✓ Enable email
   ✓ Restore VPN
   ✓ Enable Slack
   ✓ Enable Office 365

2. Benefits Access
   ✓ No change needed (already active)
   ✓ Continue access

3. Result
   All access restored
   Back to normal operations
```

**Complete Configuration Example:**
```yaml
Identity Profile: Employee Profile

Lifecycle State Configurations:

  active:
    provisioning_enabled: true
    access_profiles:
      - Standard Employee Access
      - Employee Benefits Access
    access_requests: enabled
    
  leave:
    provisioning_enabled: false
    access_profiles:
      - Employee Benefits Access  # Only this one!
    access_requests: disabled
    de_provisioning:
      standard_access: disable
      benefits_access: preserve
      
  inactive:
    provisioning_enabled: false
    access_profiles: []  # None
    access_requests: disabled
    de_provisioning:
      all_access: disable
```

**Benefits Portal Configuration:**
```
Benefits Portal Source:
  Type: Web Services (API)
  Authentication: SAML or OAuth
  
Provisioning Policy:
  Create Account:
    Lifecycle: active, leave
    
  Disable Account:
    Lifecycle: inactive
    
  Account Attributes:
    - employeeId
    - email (personal email for notifications)
    - enrollmentStatus
```

**Why This Approach Works:**
```
✓ Benefits access preserved during leave
  - Employee can manage insurance
  - Can change beneficiaries
  - Can view 401k
  - Can handle life events

✓ Work access suspended
  - Cannot access corporate systems
  - Security maintained
  - Compliance satisfied
  
✓ Automatic handling
  - Lifecycle change triggers workflows
  - Access profiles control provisioning
  - No manual intervention needed
  
✓ Seamless return
  - All access restored automatically
  - Benefits continue uninterrupted
  - Normal operations resume
```

**Alternative: Keep Email for Benefits**
```
Some organizations also keep email:

Reasoning:
  - Benefits portal sends emails
  - Need to receive notifications
  - Insurance documents via email
  
Configuration:
  Email Access:
    Lifecycle: active, leave
    During leave: Read-only
    Forward: No (keep for notifications)
    
Benefits:
  ✓ Receive benefits communications
  ✓ Access important documents
  ✓ Respond to HR benefits questions
  
Risks:
  ⚠ Could be contacted about work
  ⚠ May feel obligated to respond
  
Mitigation:
  - Auto-reply: "On leave, contact manager for work items"
  - Clear communication about leave policy
```

**Exam Tip:** Assign access profiles to multiple lifecycle states when access should persist across states. Benefits access typically spans active AND leave states.
</details>

---

# 4.4.4 Cloud Lifecycle State Attribute Purpose

**Learning Objective:** Understand the cloudLifecycleState attribute and its role in ISC.

---

## What is cloudLifecycleState?

### **Technical Overview**

**Attribute name:**
```
cloudLifecycleState
```

**Data type:**
```
String (enum)
```

**Standard values:**
```
- prehire
- active
- leave
- inactive
```

**Where it lives:**
```
Identity attribute:

Identity: John Smith
  employeeNumber: EMP12345
  firstName: John
  lastName: Smith
  email: john.smith@company.com
  department: IT
  cloudLifecycleState: "active" ← Here
```

**Exam Tip:** cloudLifecycleState is the ISC attribute that stores lifecycle state. It's set by identity profiles based on source data.

---

## Purpose of cloudLifecycleState

### **Core Functions**

**1. Drives Provisioning Decisions**
```
ISC checks cloudLifecycleState to determine:
  - Should this identity have access?
  - What access profiles apply?
  - Can user request access?
  - Should accounts be provisioned?
```

**Example:**
```
User requests Salesforce access:

ISC checks:
  identity.cloudLifecycleState = ?
  
If "active":
  ✓ Allow request
  ✓ Process approval
  ✓ Provision access
  
If "inactive":
  ✗ Deny request
  ✗ User cannot request
  Message: "Your account is inactive"
```

**2. Triggers Workflows**
```
Workflow triggers based on lifecycle changes:

Trigger: Lifecycle State Change
  When: cloudLifecycleState changes
  From: prehire
  To: active
  
  Action: Execute Joiner Workflow
    - Create AD account
    - Create email
    - Provision standard access
```

**3. Controls Access Certification Scope**
```
Certification campaigns filter by lifecycle:

Campaign: "Review Active Employee Access"
  Include identities where:
    cloudLifecycleState = "active"
  
  Exclude:
    - cloudLifecycleState = "inactive" (terminated)
    - cloudLifecycleState = "leave" (on leave)
    - cloudLifecycleState = "prehire" (not started)
```

**4. Enables Reporting and Analytics**
```
Reports use cloudLifecycleState:

Report: "Active Employees by Department"
  Filter: cloudLifecycleState = "active"
  Group by: department
  
Report: "Terminated Employees This Quarter"
  Filter: cloudLifecycleState = "inactive"
  Filter: terminationDate >= Q1_START_DATE
  
Report: "Pre-Hire Status"
  Filter: cloudLifecycleState = "prehire"
  Show: startDate, department, manager
```

---

## How cloudLifecycleState is Set

### **Identity Profile Determines Lifecycle**

**Process:**
```
1. Source aggregates (e.g., Workday)
   Employee data received
   
2. Identity Profile evaluates
   Applies lifecycle mapping logic
   
3. cloudLifecycleState set
   Attribute updated on identity
   
4. Workflows trigger (if changed)
   Provisioning/de-provisioning executes
```

**Example Mapping:**
```
Identity Profile: Employee Profile

Lifecycle Mapping Logic:

IF source.terminationDate != null 
   AND source.terminationDate <= NOW()
   THEN cloudLifecycleState = "inactive"

ELSE IF source.employeeStatus = "Leave of Absence"
   THEN cloudLifecycleState = "leave"

ELSE IF source.startDate != null 
   AND source.startDate > NOW()
   THEN cloudLifecycleState = "prehire"

ELSE IF source.employeeStatus = "Active"
   THEN cloudLifecycleState = "active"

ELSE
   cloudLifecycleState = "inactive" (safe default)
```

---

## cloudLifecycleState vs Other Attributes

### **Comparison**

**cloudLifecycleState (ISC standard):**
```
Purpose: Control ISC automation
Set by: Identity Profile
Values: Standardized (prehire, active, leave, inactive)
Used for: Provisioning, workflows, reporting
```

**employeeStatus (source attribute):**
```
Purpose: Employment status in source (HR)
Set by: HR system (Workday, ADP, etc.)
Values: Source-specific ("Active", "Terminated", "LOA", etc.)
Used for: Source data, mapping to lifecycle
```

**Relationship:**
```
Source Status → Identity Profile → cloudLifecycleState

Workday.employeeStatus = "Active"
  → Identity Profile maps
  → cloudLifecycleState = "active"
  
Workday.employeeStatus = "Terminated"
  → Identity Profile maps
  → cloudLifecycleState = "inactive"
```

**Why separate attributes:**
```
Different sources, different terminology:
  Workday: "Active", "Terminated", "LOA"
  ADP: "A", "T", "L"
  Custom: "1", "0", "2"
  
cloudLifecycleState normalizes all:
  All map to: active, inactive, leave
  ISC workflows work consistently
  Reports use standard values
```

---

## Viewing cloudLifecycleState

### **In ISC UI**

**Identity Details:**
```
Admin → Identities → Search → [Select Identity]

Identity: John Smith
  Employee Number: EMP12345
  Email: john.smith@company.com
  Department: IT
  Lifecycle State: active ← cloudLifecycleState displayed
```

**Search and Filter:**
```
Search: All active identities
  Query: @identities(cloudLifecycleState:active)
  
Search: All inactive identities
  Query: @identities(cloudLifecycleState:inactive)
  
Search: Identities on leave
  Query: @identities(cloudLifecycleState:leave)
```

**Reports:**
```
Create Report: Identity Status

Columns:
  - Name
  - Employee Number
  - Lifecycle State (cloudLifecycleState)
  - Department
  
Filter:
  cloudLifecycleState = "active"
  
Result:
  All active employees listed
```

---

## cloudLifecycleState in Workflows

### **Workflow Triggers**

**Trigger on Lifecycle Change:**
```
Workflow: Joiner Workflow

Trigger:
  Event: Lifecycle State Change
  From: prehire
  To: active
  
Actions:
  1. Create AD account
  2. Create email
  3. Provision access profiles
  4. Send welcome email
  5. Notify manager
```

**Workflow Conditions:**
```
Workflow: Quarterly Access Review

Trigger: Scheduled (quarterly)

Condition:
  IF identity.cloudLifecycleState = "active"
    THEN include in review
  ELSE
    SKIP (don't review inactive employees)
```

**Multiple State Triggers:**
```
Workflow: Benefits Portal Access

Trigger:
  Event: Lifecycle State Change
  To: active OR leave
  
Action:
  Provision benefits portal access
  
Rationale:
  - Both active and leave need benefits access
  - Workflow handles both transitions
```

---

## cloudLifecycleState in Access Profiles

### **Lifecycle-Based Assignment**

**Access Profile Configuration:**
```
Access Profile: Standard Employee Access

Assignment Rules:
  Assign when:
    cloudLifecycleState = "active"
  
  Remove when:
    cloudLifecycleState != "active"
    
Result:
  - Provisioned only for active employees
  - Auto-removed when lifecycle changes
```

**Multiple Lifecycle Assignment:**
```
Access Profile: Benefits Access

Assignment Rules:
  Assign when:
    cloudLifecycleState IN ["active", "leave"]
  
  Remove when:
    cloudLifecycleState = "inactive"
    
Result:
  - Active employees: Have access
  - Leave employees: Retain access
  - Inactive employees: Access removed
```

---

## Best Practices

### **Lifecycle Mapping**
```
✓ Use date-based logic
  - Check terminationDate first
  - More reliable than status field
  - Immediate termination on date

✓ Provide safe defaults
  - If uncertain, use "inactive"
  - Safer to deny access by default
  - Require explicit "active"

✓ Handle edge cases
  - Null dates
  - Future dates
  - Missing status
  - Data quality issues

✓ Document mapping logic
  - Clear rules
  - Business justification
  - Easy to troubleshoot
```

### **Workflow Design**
```
✓ Trigger on lifecycle changes
  - Automate provisioning/de-provisioning
  - Consistent handling
  - Timely access changes

✓ Test all transitions
  - prehire → active
  - active → leave
  - leave → active
  - active → inactive
  - Each should work correctly

✓ Handle failures gracefully
  - Retry logic
  - Error notifications
  - Manual fallback
```

### **Reporting**
```
✓ Use lifecycle in reports
  - Filter by active for current employees
  - Track inactive for terminations
  - Monitor prehire for onboarding

✓ Monitor transitions
  - How many joined this month?
  - How many terminated?
  - How many on leave?
  - Trend analysis

✓ Compliance reporting
  - Active employees with access
  - Inactive employees without access
  - Leave handling compliance
```

---

## Practice Exercise

**Question:** You need to create a report showing all employees who should currently have access to systems. What cloudLifecycleState values should you include and which should you exclude?

<details>
<summary>Answer</summary>

**Report Configuration:**

**Report Name:** "Employees with Active System Access"

**Include (Should Have Access):**
```
cloudLifecycleState = "active"
```

**Reasoning:**
```
Active employees:
  ✓ Currently employed
  ✓ Working full-time
  ✓ Need access to do their job
  ✓ Should appear in access reports
  ✓ Subject to access reviews
  
This is the primary state for system access
```

**Exclude (Should NOT Have Access):**

**1. cloudLifecycleState = "inactive"**
```
Exclude because:
  ✗ Terminated employees
  ✗ No longer with company
  ✗ Should not have any access
  ✗ Security requirement
  ✗ Compliance requirement
  
If inactive employees appear with access:
  - Security issue (orphaned access)
  - Compliance violation
  - Requires immediate remediation
```

**2. cloudLifecycleState = "prehire"**
```
Exclude because:
  ✗ Haven't started yet
  ✗ Not yet performing job duties
  ✗ Shouldn't have production access
  ✗ Pre-boarding only
  
Exception:
  - May have limited access (HR portal, training)
  - But not full production access
  - Separate report for pre-hire if needed
```

**3. cloudLifecycleState = "leave"**
```
Exclude because:
  ✗ On leave (not working)
  ✗ Temporarily absent
  ✗ Access should be suspended
  ✗ Not performing job duties
  
Exception scenario:
  Some organizations keep limited access:
    - Benefits portal
    - HR self-service
  But NOT general system access
  
For this report (system access):
  Exclude leave employees
```

**Complete Report Configuration:**
```
Report: Employees with Active System Access

Filter:
  cloudLifecycleState = "active"

Columns:
  - Employee Name
  - Employee Number
  - Department
  - Manager
  - Start Date
  - Accounts (AD, Email, etc.)
  - Access Entitlements
  
Sort By:
  Department, Last Name

Result:
  Only currently active employees
  Those who should have access
  Baseline for access reviews
```

**Use Cases for This Report:**

**Use Case 1: Access Certification**
```
Purpose: Review who has access

Process:
  1. Generate report (active employees only)
  2. Compare with actual access
  3. Certify access is appropriate
  4. Revoke inappropriate access
  
Why filter to active:
  - Only active employees should have access
  - Inactive/leave already de-provisioned
  - Focus review on current employees
```

**Use Case 2: Audit Compliance**
```
Audit Question:
  "Who currently has access to financial systems?"
  
Report:
  Filter: cloudLifecycleState = "active"
  AND has access to financial systems
  
Result:
  - All active employees with financial access
  - Audit can verify appropriateness
  - Inactive employees excluded (correct)
```

**Use Case 3: Access Analytics**
```
Metrics:
  - Total employees with access: 500
  - All have lifecycle = "active" ✓
  - Access aligned with employment status ✓
  
Red flags to investigate:
  - Inactive employee with access ✗
  - Leave employee with access ✗
  - Pre-hire with production access ✗
```

**Alternative: Comprehensive Access Report**
```
If you want to see ALL states:

Report: Complete Access Audit

Include ALL cloudLifecycleState values:
  - active
  - inactive
  - leave
  - prehire

Additional Column: Lifecycle State

Purpose:
  - See everyone
  - Identify anomalies
  - Verify lifecycle transitions
  
Expected Results:
  ✓ Active employees: Have access
  ✗ Inactive employees: NO access (investigate if they do)
  ✗ Leave employees: NO access (investigate if they do)
  ✗ Prehire employees: Limited/no access

This helps find orphaned access
```

**Query Examples:**
```
ISC Search Query:

Active employees only:
  @identities(cloudLifecycleState:active)

All employees with access to AD:
  @identities(cloudLifecycleState:active AND accounts.source.name:"Active Directory")

Active employees in IT department:
  @identities(cloudLifecycleState:active AND department:IT)

Employees who SHOULDN'T have access but might:
  @identities(cloudLifecycleState:inactive OR cloudLifecycleState:leave)
  
  Then check if they have any accounts:
    - Should be zero
    - If not zero → orphaned access to investigate
```

**Summary:**
For "Employees with Active System Access" report:
INCLUDE:
✓ cloudLifecycleState = "active"
EXCLUDE:
✗ cloudLifecycleState = "inactive" (terminated)
✗ cloudLifecycleState = "leave" (on leave)
✗ cloudLifecycleState = "prehire" (not started)
Rationale:

Active employees should have access
All others should NOT have access
Exceptions should be rare and documented
Report helps verify correct access state


**Exam Tip:** For reports on who should have access, include only cloudLifecycleState = "active". All other states should have access suspended or revoked.
</details>

---

## Section Summary

**Core Concepts for Certification:**

✅ **Lifecycle States:**
- **prehire**: Hired but not started, limited/no access
- **active**: Currently employed, full access
- **leave**: Temporary absence, access suspended
- **inactive**: Terminated, no access

✅ **Provisioning by Lifecycle:**
- **prehire**: Limited/no provisioning (wait for active)
- **active**: Full provisioning enabled
- **leave**: Provisioning disabled, access suspended
- **inactive**: De-provisioning executed

✅ **cloudLifecycleState:**
- ISC attribute storing lifecycle state
- Set by identity profile based on source data
- Drives provisioning, workflows, reporting
- Standard values: prehire, active, leave, inactive

✅ **State Transitions:**
- **prehire → active**: Joiner workflow (provision access)
- **active → leave**: Leave workflow (suspend access)
- **leave → active**: Return workflow (restore access)
- **active → inactive**: Leaver workflow (revoke access)

✅ **Best Practices:**
- Use date-based logic for lifecycle mapping
- Trigger workflows on lifecycle changes
- Configure access profiles per lifecycle
- Filter reports by cloudLifecycleState
- Test all lifecycle transitions

**Exam Focus Areas:**
- Know the four standard lifecycle states
- Understand provisioning behavior per state
- Know that cloudLifecycleState drives automation
- Recognize state transitions trigger workflows
- Understand active = provision, inactive = de-provision
- Know leave preserves (suspend), inactive removes (delete)

---

**End of Section 4.4: Lifecycle States**

**Section 4: Identity and Lifecycle Management - COMPLETE**

**Completed Sections:**
- ✅ 4.1: Identity Profiles
- ✅ 4.2: Identity Attributes  
- ✅ 4.3: Authentication in Identity Profiles
- ✅ 4.4: Lifecycle States

---

**Excellent progress! You've completed all of Section 4 on Identity and Lifecycle Management.** This covered identity profiles, attribute mappings, authentication (SAML), and lifecycle states - critical topics for the certification exam.

**Study Tips for Section 4:**
- **Identity Profiles**: Define how identities are created from authoritative sources
- **Attributes**: Static = copy, Complex = transform, Rule = conditional logic
- **Authentication**: SAML for production (SSO), not internal passwords
- **Lifecycle**: active = provision, inactive = de-provision, leave = suspend
- **cloudLifecycleState**: ISC attribute that drives all automation
