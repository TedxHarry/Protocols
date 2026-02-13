# Section 4.4: Lifecycle States

**Master lifecycle states, their use cases, and provisioning configuration**

---

## Table of Contents

- [4.4.1 Understanding Lifecycle States](#441-understanding-lifecycle-states)
- [4.4.2 Common Lifecycle State Use Cases](#442-common-lifecycle-state-use-cases)
- [4.4.3 Provisioning Options within Lifecycle States](#443-provisioning-options-within-lifecycle-states)
- [4.4.4 Cloud Lifecycle State Attribute Purpose](#444-cloud-lifecycle-state-attribute-purpose)

---

# 4.4.1 Understanding Lifecycle States

**Learning Objective:** Understand what lifecycle states are and their role in identity management.

---

## What is a Lifecycle State?

**Definition:** The current status of an identity that determines their access rights and system behavior

**Simple concept:**
```
Lifecycle State = Where someone is in their employment journey

Examples:
  - prehire: Hired but hasn't started yet
  - active: Currently employed, full access
  - leave: On leave (medical, maternity, sabbatical)
  - inactive: Terminated, no access
```

**Key point:**
```
Lifecycle state drives automation:
  - What access they should have
  - What workflows trigger
  - What provisioning occurs
```

**Exam Tip:** Lifecycle state = current status of identity that drives automated access decisions.

---

## Standard Lifecycle States

### **Common States**

| State | Meaning | Access | Workflows |
|-------|---------|--------|-----------|
| **prehire** | Hired, not started | Limited/none | Pre-boarding |
| **active** | Currently employed | Full access | Provisioning |
| **leave** | On leave | Reduced/suspended | Suspend access |
| **inactive** | Terminated | No access | De-provisioning |

**Visual timeline:**
```
Hired → prehire → Start Date → active → End Date → inactive
                                   ↓
                              (temporary leave)
```

---

## How Lifecycle States Work

### **State Determination**

**Lifecycle state set by Identity Profile:**
```
Identity Profile maps source data to lifecycle state:

Example - Workday to ISC:
  When workday.employeeStatus = "Active" → lifecycle = active
  When workday.employeeStatus = "Terminated" → lifecycle = inactive
  When workday.employeeStatus = "Leave" → lifecycle = leave
  When workday.employeeStatus = "Pre-Hire" → lifecycle = prehire
```

**Example mapping:**
```
Source Data → Identity Profile Logic → Lifecycle State

Workday:
  employeeStatus: "Active"
  startDate: 2024-01-15 (in past)
  endDate: null
  
Identity Profile:
  IF employeeStatus = "Active" AND startDate <= TODAY
    → lifecycle = active

Result:
  identity.cloudLifecycleState = "active"
```

### **State Transitions**

**Lifecycle changes trigger workflows:**
```
State Change: prehire → active
Trigger: Joiner workflow
  - Create AD account
  - Create email
  - Provision standard access
  - Send welcome email

State Change: active → leave
Trigger: Leave workflow
  - Suspend accounts (don't delete)
  - Disable access
  - Notify manager
  - Set return date

State Change: active → inactive
Trigger: Leaver workflow
  - Disable all accounts
  - Revoke all access
  - Notify manager
  - Transfer data
  - Schedule deletion

State Change: leave → active
Trigger: Return from leave workflow
  - Re-enable accounts
  - Restore access
  - Update status
```

**Exam Tip:** Lifecycle state changes trigger workflows. active → inactive = leaver workflow.

---

## Lifecycle State Attribute

### **Technical Details**

**Attribute name:**
```
cloudLifecycleState (internal ISC attribute)
```

**Values:**
```
Standard values:
  - prehire
  - active
  - leave
  - inactive

Custom values:
  - Can define custom states (advanced)
  - Examples: "contractor", "pending", "suspended"
```

**Where it's stored:**
```
On identity record:

Identity: John Smith
  employeeNumber: EMP12345
  email: john.smith@company.com
  department: IT
  cloudLifecycleState: active ← Here
```

**How it's set:**
```
Identity Profile determines lifecycle:
  
Source aggregation:
  Workday aggregates
  Employee record: status = "Active"
  
Identity Profile processes:
  Maps status to lifecycle
  Sets cloudLifecycleState = "active"
  
Result:
  Identity now in "active" state
```

---

## Lifecycle vs Employment Status

### **Key Difference**

**Employment Status (Source):**
```
Source system value:
  - Workday.employeeStatus = "Active"
  - HR system's field
  - Authoritative source data
```

**Lifecycle State (ISC):**
```
ISC internal state:
  - cloudLifecycleState = "active"
  - Standardized across all sources
  - Drives ISC automation
```

**Mapping:**
```
Source Status → Identity Profile → Lifecycle State

Different sources, same lifecycle:
  Workday: "Active" → active
  Contractor System: "Current" → active
  Result: Both map to standard "active" state
```

**Why this matters:**
```
Different sources use different terminology:
  Workday: "Active", "Terminated", "Leave of Absence"
  ADP: "A", "T", "L"
  Custom HR: "1", "0", "2"

Identity Profile normalizes all to standard lifecycle:
  All → active, inactive, leave
  ISC workflows work consistently
```

---

## Lifecycle State Scope

### **What Lifecycle Controls**

**Access decisions:**
```
Lifecycle = active:
  ✓ Can request access
  ✓ Accounts provisioned
  ✓ Access grants processed
  ✓ Appears in certifications

Lifecycle = inactive:
  ✗ Cannot request access
  ✗ Provisioning disabled
  ✗ Access revoked
  ✗ Excluded from active certifications
```

**Workflow triggers:**
```
Based on lifecycle transitions:
  
  prehire → active: Joiner workflow
  active → leave: Leave workflow
  leave → active: Return workflow
  active → inactive: Leaver workflow
```

**Provisioning behavior:**
```
Lifecycle determines:
  - If provisioning allowed
  - What gets provisioned
  - When de-provisioning occurs
```

---

## Practice Exercise

**Question:** An employee in Workday has employeeStatus = "Active" and terminationDate = "2024-02-13" (today). What lifecycle state should they have and why?

<details>
<summary>Answer</summary>

**Answer: lifecycle = inactive (terminated today)**

**Reasoning:**

**Source Data:**
```
Workday:
  employeeStatus: "Active"
  terminationDate: "2024-02-13"
  Today's date: 2024-02-13
```

**Analysis:**

**Option 1: Simple status mapping (INCORRECT)**
```
If identity profile only checks employeeStatus:
  IF employeeStatus = "Active" → lifecycle = active
  
Problem:
  - Employee terminated today
  - Still shows as "Active" in Workday
  - Would incorrectly remain active in ISC
  - Would keep all access (security issue!)
```

**Option 2: Date-aware mapping (CORRECT)**
```
If identity profile checks both status AND termination date:
  
  IF terminationDate IS NOT NULL 
    AND terminationDate <= TODAY
    → lifecycle = inactive
  ELSE IF employeeStatus = "Active"
    → lifecycle = active
    
Result:
  terminationDate = 2024-02-13
  TODAY = 2024-02-13
  terminationDate <= TODAY ✓
  
  → lifecycle = inactive
```

**Why this is correct:**

**Reason 1: Termination date reached**
```
terminationDate = 2024-02-13 (today)
  → Employee's last day is today
  → Should be terminated as of today
  → Access should be revoked today
  → lifecycle = inactive
```

**Reason 2: Security**
```
If lifecycle remained "active":
  ✗ Terminated employee keeps all access
  ✗ Can still use systems
  ✗ Security risk
  ✗ Compliance violation

If lifecycle = inactive:
  ✓ Leaver workflow triggers
  ✓ All access revoked
  ✓ Accounts disabled
  ✓ Secure
```

**Reason 3: Timing of HR updates**
```
HR systems often work this way:
  - Set termination date in advance
  - employeeStatus doesn't change until HR processes it
  - Status may stay "Active" for days after term date
  
ISC should use termination date, not just status
```

**Proper Identity Profile Configuration:**
```
Lifecycle State Mapping:

Priority 1 - Check termination date:
  IF terminationDate != null 
    AND terminationDate <= NOW()
    THEN lifecycle = inactive

Priority 2 - Check leave status:
  ELSE IF employeeStatus = "Leave"
    THEN lifecycle = leave

Priority 3 - Check pre-hire:
  ELSE IF startDate != null 
    AND startDate > NOW()
    THEN lifecycle = prehire

Priority 4 - Default to active:
  ELSE IF employeeStatus = "Active"
    THEN lifecycle = active

Priority 5 - Fallback:
  ELSE
    lifecycle = inactive (safe default)
```

**Applying to this scenario:**
```
Employee data:
  employeeStatus: "Active"
  terminationDate: "2024-02-13"
  Today: 2024-02-13

Process:
  1. Check Priority 1:
     terminationDate = "2024-02-13" (not null) ✓
     terminationDate <= NOW() → 2024-02-13 <= 2024-02-13 ✓
     
     Match! → lifecycle = inactive

Result:
  identity.cloudLifecycleState = "inactive"
  
Triggers:
  - Leaver workflow executes
  - All accounts disabled
  - All access revoked
  - Manager notified
```

**What happens today:**
```
Morning (before Workday aggregation):
  Employee status: active
  Access: Full access
  
After Workday aggregation (e.g., 2 AM):
  ISC detects: terminationDate = today
  Sets: lifecycle = inactive
  Triggers: Leaver workflow
  
Within minutes:
  - AD account disabled
  - Email disabled (forwarded to manager)
  - Salesforce disabled
  - VPN revoked
  - All access removed
  
Employee arrives to work:
  - Cannot log in (accounts disabled)
  - Expected behavior
```

**Edge Case - Future termination date:**
```
What if terminationDate = "2024-02-20" (future)?

Data:
  employeeStatus: "Active"
  terminationDate: "2024-02-20"
  Today: 2024-02-13

Process:
  1. Check Priority 1:
     terminationDate = "2024-02-20" (not null) ✓
     terminationDate <= NOW() → 2024-02-20 <= 2024-02-13 ✗
     
     No match.
     
  2. Check Priority 2-4:
     employeeStatus = "Active" ✓
     
     Match! → lifecycle = active

Result:
  lifecycle = active (until Feb 20)
  
On Feb 20:
  Aggregation runs
  terminationDate <= NOW() becomes true
  lifecycle changes: active → inactive
  Leaver workflow triggers
```

**Summary:**
```
Termination date = today → lifecycle = inactive

Why:
  ✓ Termination date is authoritative
  ✓ Takes precedence over status
  ✓ Security: revoke access immediately
  ✓ Compliance: term date = access end date
  
Identity Profile should:
  ✓ Check termination date first
  ✓ Use date logic, not just status
  ✓ Set inactive when term date reached
  ✓ Trigger leaver workflows
```

**Exam Tip:** When termination date exists and is today or past, lifecycle should be inactive regardless of current status field. Date-based logic is critical for security.
</details>

---

# 4.4.2 Common Lifecycle State Use Cases

**Learning Objective:** Understand practical applications of each lifecycle state.

---

## Lifecycle State: prehire

### **Purpose**

**Used for:** Employees who are hired but haven't started yet

**Characteristics:**
```
Timeline: Between offer acceptance and start date
Access: Limited or none
Duration: Days to weeks
Goal: Prepare for day one
```

### **Common Use Cases**

**Use Case 1: Pre-boarding**
```
Scenario: New hire starts in 2 weeks

Lifecycle: prehire

Actions triggered:
  ✓ Create identity in ISC (early)
  ✓ Send pre-boarding email
  ✓ Assign equipment order
  ✓ Create building access request
  ✓ Initiate background check
  ✓ Trigger training enrollment
  ✓ Manager notification
  
Do NOT yet:
  ✗ Create AD account (wait until start date)
  ✗ Create email (wait until start date)
  ✗ Provision applications
```

**Use Case 2: Early Access for Setup**
```
Scenario: IT role needs laptop before start date

Lifecycle: prehire

Actions:
  ✓ Create limited AD account (disabled)
  ✓ Assign laptop to employee
  ✓ Employee configures laptop at home
  ✓ Email created but disabled
  
On start date:
  prehire → active transition
  ✓ Enable AD account
  ✓ Enable email
  ✓ Full access granted
```

### **State Transition**

**Entry: Hired**
```
HR creates employee in Workday:
  employeeStatus: "Pre-Hire"
  startDate: "2024-03-01" (future)
  
Workday aggregates to ISC
  
Identity Profile evaluates:
  IF startDate > NOW()
    → lifecycle = prehire
    
Result:
  New identity created
  cloudLifecycleState = "prehire"
  Pre-boarding workflow triggers
```

**Exit: Start Date Reached**
```
On start date (2024-03-01):
  
Workday aggregates (daily at 2 AM)
  
Identity Profile evaluates:
  IF startDate <= NOW() 
    AND employeeStatus = "Active"
    → lifecycle = active
    
Lifecycle change: prehire → active
  
Joiner workflow triggers:
  ✓ Create/enable AD account
  ✓ Create/enable email
  ✓ Provision applications
  ✓ Grant standard access
  ✓ Welcome email sent
```

---

## Lifecycle State: active

### **Purpose**

**Used for:** Currently employed, full access

**Characteristics:**
```
Timeline: Start date through employment
Access: Full access
Duration: Months to years
Goal: Normal operations
```

### **Common Use Cases**

**Use Case 1: Standard Employee**
```
Scenario: Active employee, normal operations

Lifecycle: active

Access enabled:
  ✓ Can request access
  ✓ Provisioning enabled
  ✓ All accounts active
  ✓ Applications accessible
  ✓ Included in certifications
  ✓ Normal operations
```

**Use Case 2: Department Transfer**
```
Scenario: Employee moves IT → Marketing

Lifecycle: active (stays active)

Workday updated:
  department: IT → Marketing
  manager: John → Jane
  
ISC processes:
  Lifecycle: active (no change)
  Attributes updated: department, manager
  
Mover workflow triggers:
  ✓ Revoke IT-specific access
  ✓ Grant Marketing-specific access
  ✓ Update AD attributes
  ✓ Notify new/old managers
```

**Use Case 3: Promotion/Role Change**
```
Scenario: Developer promoted to Manager

Lifecycle: active (stays active)

Workday updated:
  title: Developer → Engineering Manager
  
ISC processes:
  Lifecycle: active (no change)
  Title updated
  
Workflow triggers:
  ✓ Grant management access
  ✓ Add to manager groups
  ✓ Grant approval permissions
```

### **State Transitions**

**Entry from prehire:**
```
Start date reached → prehire → active
Joiner workflow executes
```

**Entry from leave:**
```
Return from leave → leave → active
Return workflow executes
```

**Exit to leave:**
```
Leave of absence → active → leave
Leave workflow executes
```

**Exit to inactive:**
```
Termination → active → inactive
Leaver workflow executes
```

---

## Lifecycle State: leave

### **Purpose**

**Used for:** Employees temporarily not working (medical, maternity, sabbatical)

**Characteristics:**
```
Timeline: Temporary absence
Access: Suspended or limited
Duration: Weeks to months
Goal: Suspend access, preserve for return
```

### **Common Use Cases**

**Use Case 1: Maternity Leave**
```
Scenario: 12-week maternity leave

Lifecycle: active → leave

Leave workflow:
  ✓ Suspend AD account (don't delete)
  ✓ Disable email access
  ✓ Forward email to manager
  ✓ Revoke application access
  ✓ Preserve data/files
  ✓ Set return date
  ✓ Notify manager/HR

During leave (lifecycle = leave):
  ✗ Cannot log in
  ✗ No application access
  ✗ Email forwarded
  ✓ Identity preserved
  ✓ Data retained
  
Return date reached:
  leave → active
  
Return workflow:
  ✓ Re-enable AD account
  ✓ Re-enable email
  ✓ Restore access
  ✓ Welcome back email
```

**Use Case 2: Medical Leave**
```
Scenario: Short-term disability (4 weeks)

Lifecycle: active → leave

Actions:
  ✓ Suspend access (similar to maternity)
  ✓ Preserve benefits access (if needed)
  ✓ Keep HR system access (for forms)
  ✓ Set tentative return date
  
Note: Return date may be uncertain
  Can extend leave if needed
  Workflow handles date changes
```

**Use Case 3: Sabbatical**
```
Scenario: 6-month sabbatical

Lifecycle: active → leave

Actions:
  ✓ Full access suspension
  ✓ Data archived
  ✓ Equipment collected
  ✓ Building access suspended
  ✓ Return date documented
  
Return:
  ✓ Full access restoration
  ✓ Equipment reissued
  ✓ Building access restored
```

### **Leave vs Termination**

**Key differences:**

| Aspect | Leave | Inactive (Terminated) |
|--------|-------|----------------------|
| **Duration** | Temporary | Permanent |
| **Return** | Expected | No return |
| **Data** | Preserved | Archived/deleted |
| **Accounts** | Suspended | Disabled/deleted |
| **Email** | Forwarded | Disabled |
| **Re-enable** | Automatic on return | N/A |

**Why separate leave state:**
```
Benefits of "leave" state:
  ✓ Preserves context (no data loss)
  ✓ Easy restoration (one workflow)
  ✓ Tracks temporary absences
  ✓ Different handling than termination
  
If only had active/inactive:
  ✗ Leave would = inactive (wrong!)
  ✗ Looks like termination
  ✗ Data might be deleted
  ✗ Harder to restore access
```

---

## Lifecycle State: inactive

### **Purpose**

**Used for:** Terminated employees, no access

**Characteristics:**
```
Timeline: After employment ends
Access: None
Duration: Permanent
Goal: Revoke all access, preserve audit trail
```

### **Common Use Cases**

**Use Case 1: Standard Termination**
```
Scenario: Employee terminated

Lifecycle: active → inactive

Leaver workflow:
  Immediate (within minutes):
    ✓ Disable AD account
    ✓ Revoke VPN access
    ✓ Disable email (forward to manager)
    ✓ Revoke application access
    ✓ Disable building access
    
  Same day:
    ✓ Manager handoff (email, files)
    ✓ IT equipment collection
    ✓ Access reviews updated
    ✓ Certifications updated
    
  30-90 days later:
    ✓ Archive data
    ✓ Delete accounts (per policy)
    ✓ Remove from systems
```

**Use Case 2: Voluntary Resignation**
```
Scenario: Employee resigns, 2-week notice

Process:
  Today: Resignation submitted
    terminationDate set: 2 weeks from now
    Lifecycle: active (until term date)
    
  During notice period:
    - Still has access
    - Completes transition
    - Knowledge transfer
    
  Termination date:
    Lifecycle: active → inactive
    Leaver workflow executes
    All access revoked
```

**Use Case 3: Immediate Termination**
```
Scenario: Termination for cause, effective immediately

Process:
  HR updates Workday:
    employeeStatus: "Terminated"
    terminationDate: TODAY
    
  ISC aggregates (may trigger immediate or next run)
    
  Lifecycle: active → inactive
    
  Leaver workflow (priority/urgent):
    ✓ IMMEDIATE account disable
    ✓ IMMEDIATE VPN revoke
    ✓ IMMEDIATE building access disable
    ✓ Security team notified
    ✓ Manager notified
    ✓ Equipment collection scheduled
    
Note: Some organizations trigger immediate aggregation
  for urgent terminations via API call
```

### **Inactive Data Retention**

**Identity preservation:**
```
Identity record: Kept in ISC
  Purpose:
    ✓ Audit trail
    ✓ Historical access reviews
    ✓ Compliance reporting
    ✓ Reference for re-hire
    
  Retention: Typically 7 years

Attributes preserved:
  ✓ Name, employee ID
  ✓ Department, manager (at time of term)
  ✓ Termination date
  ✓ Access history
  ✓ Certification history
```

**Account deletion:**
```
Typical timeline:
  Day 0: Accounts disabled
  Day 30-90: Accounts deleted
  
Varies by system:
  - AD: Often 90 days
  - Email: 30 days (mailbox converted to shared)
  - Applications: Immediate to 90 days
  
ISC identity: Preserved (not deleted)
```

---

## Practice Exercise

**Question:** An employee goes on 12-week maternity leave. Walk through the lifecycle state changes and what happens at each stage.

<details>
<summary>Answer</summary>

**Complete Maternity Leave Lifecycle Journey:**

**Stage 1: Pre-Leave (Lifecycle = active)**

**Timeline: 2 weeks before leave**
```
Current state:
  Lifecycle: active
  Employee: Working normally
  Access: Full access
  
HR actions:
  - Employee submits leave request
  - HR approves 12-week maternity leave
  - Leave dates entered in Workday:
    * Start: March 1, 2024
    * End: May 24, 2024 (12 weeks later)
```

**Stage 2: Leave Begins (Lifecycle: active → leave)**

**Timeline: March 1, 2024 (leave start date)**
```
Workday updated:
  employeeStatus: "Leave of Absence"
  leaveStartDate: "2024-03-01"
  leaveEndDate: "2024-05-24"
  
Daily aggregation runs (2 AM):
  ISC receives updated employee data
  
Identity Profile evaluates:
  IF employeeStatus = "Leave of Absence"
    → lifecycle = leave
    
Lifecycle change detected:
  Previous: active
  Current: leave
  Change: active → leave
  
Leave Workflow Triggers:
  [Immediate actions - within 5 minutes]
```

**Leave Workflow Execution:**

**Immediate Actions (5-10 minutes):**
```
1. Suspend AD Account
   Action: Set account to disabled
   Result:
     ✗ Cannot log in
     ✗ Password no longer works
     ✓ Account preserved (not deleted)
     ✓ Groups retained
     ✓ Data preserved

2. Configure Email
   Action: Set email forwarding
   Configuration:
     - Disable direct access
     - Forward to manager (jane.doe@company.com)
     - Auto-reply: "I'm on maternity leave until May 24"
   Result:
     ✗ Cannot access email directly
     ✓ Manager receives emails
     ✓ Senders notified of leave

3. Revoke Application Access
   Applications affected:
     - Salesforce: License suspended
     - Slack: Account deactivated
     - VPN: Access revoked
     - Other apps: As configured
   Result:
     ✗ Cannot access applications
     ✓ Data preserved for return

4. Suspend Building Access
   Action: Deactivate badge
   Result:
     ✗ Cannot access building
     ✓ Badge preserved for re-activation
```

**Same Day Actions:**
```
5. Notifications Sent
   Email to manager:
     "Sarah Johnson's maternity leave started today.
      Email forwarded to you.
      Expected return: May 24, 2024."
   
   Email to HR:
     "Sarah Johnson leave workflow completed.
      All access suspended.
      Return date: May 24, 2024."
   
   Email to IT:
     "Sarah Johnson on leave.
      Equipment collected.
      Laptop stored: Locker 42."

6. Documentation Updated
   ISC records:
     - Lifecycle state: leave
     - Leave start: March 1, 2024
     - Leave end: May 24, 2024
     - Manager notified: Yes
     - Access suspended: Yes
```

**Stage 3: During Leave (Lifecycle = leave)**

**Timeline: March 1 - May 23, 2024 (12 weeks)**
```
Identity status:
  cloudLifecycleState: leave
  Status: All access suspended
  
What's disabled:
  ✗ AD account (cannot log in)
  ✗ Email access (forwarded to manager)
  ✗ All applications
  ✗ VPN
  ✗ Building access
  
What's preserved:
  ✓ Identity in ISC
  ✓ AD account (suspended, not deleted)
  ✓ Email mailbox
  ✓ Files/data
  ✓ Group memberships
  ✓ Access history
  
Daily aggregations:
  - Workday continues to aggregate
  - Status remains "Leave of Absence"
  - Lifecycle remains "leave"
  - No changes until return date
```

**Stage 4: Leave Extended (Optional Scenario)**

**Timeline: Week 10 of leave (May 10)**
```
If leave extended by 4 weeks:

HR updates Workday:
  leaveEndDate: "2024-05-24" → "2024-06-21"
  
Next aggregation:
  ISC receives new end date
  Identity updated:
    Expected return: June 21, 2024
  
  Lifecycle: Still "leave" (no change)
  
Actions:
  - Manager notified of extension
  - Return workflow rescheduled
  - No access changes (still suspended)
```

**Stage 5: Return Preparation (1 day before return)**

**Timeline: May 23, 2024**
```
Automated checks:
  System scans for employees returning tomorrow
  
Identifies:
  Sarah Johnson
    leaveEndDate: May 24, 2024
    Current lifecycle: leave
    Action needed: Prepare for return
  
Preparation workflow:
  - Notify manager: "Sarah returns tomorrow"
  - Notify IT: "Prepare equipment"
  - Queue return workflow
```

**Stage 6: Return from Leave (Lifecycle: leave → active)**

**Timeline: May 24, 2024 (return date)**
```
Workday updated:
  employeeStatus: "Active"
  (Leave end date reached)
  
Daily aggregation runs (2 AM):
  ISC receives updated status
  
Identity Profile evaluates:
  IF employeeStatus = "Active" 
    AND leaveEndDate <= NOW()
    → lifecycle = active
    
Lifecycle change detected:
  Previous: leave
  Current: active
  Change: leave → active
  
Return from Leave Workflow Triggers:
```

**Return Workflow Execution:**

**Immediate Actions (5-10 minutes):**
```
1. Re-enable AD Account
   Action: Set account to enabled
   Result:
     ✓ Can log in
     ✓ Password works (same as before leave)
     ✓ Groups restored
     ✓ Same access as before

2. Restore Email Access
   Action:
     - Remove email forwarding
     - Re-enable direct access
     - Remove auto-reply
   Result:
     ✓ Can access email directly
     ✓ Emails no longer forwarded
     ✓ Normal email operations

3. Re-enable Application Access
   Applications restored:
     - Salesforce: License re-activated
     - Slack: Account re-activated
     - VPN: Access restored
     - Other apps: As configured
   Result:
     ✓ Full application access restored

4. Re-enable Building Access
   Action: Reactivate badge
   Result:
     ✓ Can access building
     ✓ Same badge as before
```

**Same Day Actions:**
```
5. Notifications Sent
   Welcome back email to employee:
     "Welcome back, Sarah!
      Your access has been restored.
      If you have any issues, contact IT."
   
   Email to manager:
     "Sarah Johnson has returned from leave.
      All access restored.
      Welcome her back!"
   
   Email to HR:
     "Sarah Johnson return workflow completed.
      Status: Active."

6. Verification Checks
   System validates:
     ✓ AD account enabled
     ✓ Email working
     ✓ Applications accessible
     ✓ Building access active
     
   If any failures:
     - Alert IT
     - Create ticket
     - Notify manager
```

**Stage 7: Post-Return (Lifecycle = active)**

**Timeline: After May 24, 2024**
```
Identity status:
  cloudLifecycleState: active
  Access: Full access restored
  
Employee experience:
  ✓ Logs in successfully
  ✓ Email access working
  ✓ Applications working
  ✓ Badge works
  ✓ Everything as before leave
  
Operations return to normal:
  - Regular provisioning enabled
  - Can request access
  - Included in certifications
  - Normal workflows apply
```

**Complete Timeline Summary:**
```
Feb 15: Leave request approved
  Lifecycle: active
  Action: HR enters dates in Workday

Mar 1 (2 AM): Leave begins
  Lifecycle: active → leave
  Actions:
    - Account suspended
    - Email forwarded
    - Apps disabled
    - Building access off
  
Mar 1 - May 23: During leave
  Lifecycle: leave
  Status: All access suspended
  
May 24 (2 AM): Return from leave
  Lifecycle: leave → active
  Actions:
    - Account re-enabled
    - Email restored
    - Apps re-enabled
    - Building access on
  
May 24 onwards: Normal operations
  Lifecycle: active
  Status: Full access
```

**Key Benefits of Leave State:**
```
✓ Temporary suspension (not termination)
✓ Data/account preservation
✓ Easy restoration (automatic)
✓ Tracks leave periods
✓ Different handling than termination
✓ Smooth return experience
✓ Manager gets emails during leave
✓ Security maintained (no access during leave)
```

**What Would Happen Without Leave State:**
If only had active/inactive:
Option 1: Keep as active
✗ Employee has access while on leave
✗ Security risk
✗ Compliance issue
Option 2: Set to inactive
✗ Looks like termination
✗ May trigger deletions
✗ Harder to restore
✗ Loses context
Leave state solves both problems:
✓ Access suspended (secure)
✓ Preserved for return (not terminated)
✓ Automatic restoration

**Exam Tip:** Leave state temporarily suspends access while preserving everything for easy restoration. Uses different workflows than termination. Active → leave → active.
</details>
