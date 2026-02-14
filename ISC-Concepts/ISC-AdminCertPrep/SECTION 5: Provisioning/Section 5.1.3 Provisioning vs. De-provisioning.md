# 5.1.3 Provisioning vs. De-provisioning

**Learning Objective:** Understand the differences between provisioning and de-provisioning, and when each occurs.

---

## Provisioning vs. De-provisioning Overview

### **Definitions**

**Provisioning:**
```
Creating or granting access

Actions:
  - Create account
  - Add to group
  - Assign license
  - Grant permissions
  - Enable access

Direction: Adding access
```

**De-provisioning:**
```
Removing or revoking access

Actions:
  - Delete account
  - Remove from group
  - Revoke license
  - Remove permissions
  - Disable access

Direction: Removing access
```

**Key difference:**
```
Provisioning = Give access
De-provisioning = Take away access
```

**Exam Tip:** Provisioning adds access, de-provisioning removes access. Both are automated based on lifecycle and policies.

---

## When Provisioning Occurs

### **Provisioning Scenarios**

**Scenario 1: New Hire (Joiner)**
```
Event: Employee starts (prehire → active)

Provisioning Actions:
  ✓ Create AD account
  ✓ Create email mailbox
  ✓ Grant VPN access (add to group)
  ✓ Create Salesforce account
  ✓ Assign Office 365 license
  ✓ Grant building access

Result: User has all necessary access to start working
```

**Scenario 2: Access Request Approval**
```
Event: User requests Salesforce, manager approves

Provisioning Actions:
  ✓ Create Salesforce account
  ✓ Assign Sales Cloud license
  ✓ Set user profile
  ✓ Grant permissions

Result: User can access Salesforce
```

**Scenario 3: Role Assignment**
```
Event: User assigned "Developer" role

Provisioning Actions:
  ✓ Create GitHub account
  ✓ Add to Dev-Team AD group
  ✓ Grant JIRA access
  ✓ Provision dev server access

Result: User has developer tools and access
```

**Scenario 4: Department Transfer (Mover)**
```
Event: User moves from IT to Sales

Provisioning Actions (new access):
  ✓ Create Salesforce account
  ✓ Add to Sales-Team group
  ✓ Grant sales tools access
  ✓ Add to sales distribution lists

Also includes de-provisioning (covered next)
```

**Scenario 5: Return from Leave**
```
Event: Employee returns (leave → active)

Provisioning Actions (re-enable):
  ✓ Enable AD account
  ✓ Enable email
  ✓ Re-activate application licenses
  ✓ Restore group memberships

Result: Access restored as before leave
```

---

## When De-provisioning Occurs

### **De-provisioning Scenarios**

**Scenario 1: Termination (Leaver)**
```
Event: Employee terminated (active → inactive)

De-provisioning Actions:
  ✓ Disable AD account
  ✓ Disable email (forward to manager)
  ✓ Remove from all groups
  ✓ Revoke VPN access
  ✓ Disable Salesforce account
  ✓ Revoke Office 365 license
  ✓ Disable building access
  ✓ Schedule account deletion (30-90 days)

Result: User has no access to any systems
```

**Scenario 2: Access Revocation**
```
Event: Access review - manager revokes Salesforce access

De-provisioning Actions:
  ✓ Disable Salesforce account
  ✓ Revoke Sales Cloud license
  ✓ Remove permissions

Result: User no longer has Salesforce access
```

**Scenario 3: Role Removal**
```
Event: "Developer" role removed from user

De-provisioning Actions:
  ✓ Remove GitHub access
  ✓ Remove from Dev-Team AD group
  ✓ Revoke JIRA access
  ✓ Remove dev server access

Result: User no longer has developer access
```

**Scenario 4: Department Transfer (Mover)**
```
Event: User moves from Sales to IT

De-provisioning Actions (old access):
  ✓ Disable Salesforce account
  ✓ Remove from Sales-Team group
  ✓ Revoke sales tools access
  ✓ Remove from sales distribution lists

Combined with provisioning new IT access

Result: Sales access removed, IT access added
```

**Scenario 5: Leave of Absence**
```
Event: Employee on leave (active → leave)

De-provisioning Actions (suspend):
  ✓ Disable AD account
  ✓ Disable email (forward to manager)
  ✓ Suspend application licenses
  ✓ Revoke VPN access

Note: Accounts suspended, not deleted (for return)

Result: Temporary access suspension
```

---

## Provisioning vs. De-provisioning Comparison

### **Side-by-Side Comparison**

| Aspect | Provisioning | De-provisioning |
|--------|-------------|-----------------|
| **Action** | Grant access | Revoke access |
| **Direction** | Add | Remove |
| **Trigger** | Joiner, request approval, role add | Leaver, access revoke, role remove |
| **Examples** | Create account, add to group | Delete account, remove from group |
| **Urgency** | Medium (hours acceptable) | HIGH (immediate for security) |
| **Reversibility** | Can be revoked later | Can be re-granted if needed |
| **Risk** | Low (granting access) | High (security if delayed) |

---

## De-provisioning Timing and Priority

### **Critical Timing Differences**

**Provisioning Timing:**
```
Urgency: Medium to Low

Acceptable delays:
  - New hire: Hours to 1 day acceptable
    Employee can wait briefly for access
    
  - Access request: Hours to days acceptable
    Depends on approval workflow
    
  - Role assignment: Hours acceptable
    Not usually urgent

Typical execution: 
  Within hours, sometimes batched
```

**De-provisioning Timing:**
```
Urgency: HIGH (especially terminations)

Required speed:
  - Termination: IMMEDIATE (minutes)
    Security risk if delayed
    
  - Access revocation: Same day
    Compliance requirement
    
  - Role removal: Hours
    Remove unnecessary access promptly

Typical execution:
  Real-time, high priority
```

**Why de-provisioning is more urgent:**
```
Security risk:
  Terminated employee with access = security breach
  Can cause damage
  Can steal data
  Can sabotage systems
  
Compliance:
  Regulations require timely access removal
  Audit findings if delayed
  
Business risk:
  Unauthorized access
  Data leakage
  Legal liability
```

---

## De-provisioning Methods

### **Method 1: Disable (Soft De-provisioning)**

**What it is:**
```
Account disabled but not deleted

Example - Active Directory:
  Account: john.smith
  Status: Disabled
  Data: Preserved
  Can be re-enabled: Yes
```

**When to use:**
```
✓ Initial termination (disable immediately)
✓ Leave of absence (temporary suspension)
✓ Access suspension pending investigation
✓ Compliance requires retention

Advantages:
  ✓ Reversible
  ✓ Data preserved
  ✓ Quick to re-enable
  ✓ Audit trail intact
  
Disadvantages:
  ✗ Account still exists (license cost)
  ✗ Accumulates disabled accounts
  ✗ Requires cleanup process
```

**Example:**
```
Day 0 (Termination):
  Action: Disable AD account
  Result:
    - Account exists
    - User cannot log in
    - Password doesn't work
    - Groups retained
    - Data preserved
    
  Status: Disabled, not deleted
```

---

### **Method 2: Delete (Hard De-provisioning)**

**What it is:**
```
Account permanently removed

Example - Active Directory:
  Account: john.smith
  Status: Deleted
  Data: Gone
  Can be re-enabled: No (must recreate)
```

**When to use:**
```
✓ After retention period (30-90 days post-termination)
✓ Cleanup of old disabled accounts
✓ Compliance requires deletion
✓ Test accounts no longer needed

Advantages:
  ✓ Clean (no account sprawl)
  ✓ Reduces license costs
  ✓ Definitive removal
  ✓ Compliance satisfied
  
Disadvantages:
  ✗ Irreversible
  ✗ Data lost
  ✗ Cannot re-enable
  ✗ Must recreate if needed
```

**Example:**
```
Day 90 (Post-termination):
  Action: Delete AD account
  Result:
    - Account removed from AD
    - All data deleted
    - Groups memberships gone
    - Cannot recover
    
  Status: Deleted permanently
```

---

### **Recommended De-provisioning Strategy**

**Two-stage de-provisioning:**
```
Stage 1: Immediate Disable (Day 0)
  When: Employee terminated
  Action: Disable all accounts
  Result: No access, data preserved
  
Stage 2: Scheduled Deletion (Day 30-90)
  When: Retention period complete
  Action: Delete accounts
  Result: Complete removal
```

**Timeline example:**
```
Day 0 (Termination Date):
  ✓ Disable AD account
  ✓ Disable email (forward to manager)
  ✓ Disable all applications
  ✓ Revoke all access
  Security: Immediate
  Data: Preserved

Day 1-30:
  Status: Disabled accounts remain
  Purpose: Data access for transition
  Manager can: Access forwarded email, retrieve files

Day 30:
  ✓ Email mailbox converted to shared
  ✓ No longer consuming license
  Status: Email archived

Day 90:
  ✓ Delete AD account
  ✓ Delete application accounts
  ✓ Remove all traces
  Status: Permanently deleted
  
ISC Identity:
  ✓ Preserved indefinitely (audit trail)
  Status: inactive
  Purpose: Compliance, reporting, history
```

**Exam Tip:** Best practice is disable immediately (Day 0), then delete after retention period (Day 30-90). Never delete immediately unless required.

---

## Provisioning and De-provisioning in Workflows

### **Joiner Workflow (Provisioning)**

**Trigger:** Lifecycle = active (new hire)

**Provisioning actions:**
```
1. Create Accounts:
   ✓ AD account
   ✓ Email mailbox
   ✓ Application accounts

2. Grant Access:
   ✓ Add to groups
   ✓ Assign licenses
   ✓ Grant permissions

3. Configure Attributes:
   ✓ Set department
   ✓ Set manager
   ✓ Set location

4. Enable Services:
   ✓ VPN access
   ✓ Building access
   ✓ Phone/mobile

All actions = Provisioning
```

---

### **Leaver Workflow (De-provisioning)**

**Trigger:** Lifecycle = inactive (termination)

**De-provisioning actions:**
```
1. Disable Accounts:
   ✓ Disable AD account
   ✓ Disable email
   ✓ Disable applications

2. Revoke Access:
   ✓ Remove from groups
   ✓ Revoke licenses
   ✓ Remove permissions

3. Secure Data:
   ✓ Forward email to manager
   ✓ Transfer files
   ✓ Archive mailbox

4. Revoke Physical:
   ✓ Revoke VPN
   ✓ Disable building access
   ✓ Deactivate phone

All actions = De-provisioning
```

---

### **Mover Workflow (Both)**

**Trigger:** Department change (IT → Sales)

**De-provisioning (remove old access):**
```
1. Remove IT Access:
   ✗ Remove from IT-All group
   ✗ Revoke IT-specific apps
   ✗ Remove dev server access
   ✗ Remove IT distribution lists
```

**Provisioning (add new access):**
```
2. Add Sales Access:
   ✓ Add to Sales-Team group
   ✓ Create Salesforce account
   ✓ Grant sales tools
   ✓ Add to sales distribution lists
```

**Result:**
```
Old access removed (de-provisioning)
New access granted (provisioning)
Seamless transition
```

---

## Common De-provisioning Errors

### **Error 1: Delayed De-provisioning**

**Problem:**
```
Termination occurs but de-provisioning delayed

Risk:
  - Terminated employee still has access
  - Can log in for hours/days
  - Security breach potential
  - Compliance violation
```

**Solution:**
```
✓ Real-time de-provisioning
✓ Immediate on lifecycle change
✓ High-priority execution
✓ Monitoring and alerts
```

---

### **Error 2: Incomplete De-provisioning**

**Problem:**
```
Some access revoked, but not all

Example:
  ✓ AD account disabled
  ✓ Email disabled
  ✗ Salesforce still active
  ✗ VPN still works

Risk:
  - Partial access remains
  - Backdoor entry
  - Compliance gap
```

**Solution:**
```
✓ Comprehensive de-provisioning policy
✓ Verify all systems
✓ Automated checking
✓ Manual verification for critical terminations
```

---

### **Error 3: Premature Deletion**

**Problem:**
```
Account deleted immediately (Day 0)

Issues:
  - Manager can't access forwarded email
  - Files can't be retrieved
  - Data lost
  - May violate retention policy
```

**Solution:**
```
✓ Disable first (Day 0)
✓ Delete later (Day 30-90)
✓ Follow retention policy
✓ Allow data transition period
```

---

## Practice Exercise

**Question:** An employee is terminated today. Walk through the de-provisioning process from termination through final account deletion, including timing and what happens at each stage.

<details>
<summary>Answer</summary>

**Complete De-provisioning Process:**

**Employee Details:**
```
Name: John Smith
Employee ID: EMP12345
Department: Sales
Termination Date: February 14, 2024 (today)
Reason: Voluntary resignation
Notice: 2 weeks given
Last Day: Today
```

---

**STAGE 1: IMMEDIATE DE-PROVISIONING (Day 0 - Today)**

**Timeline: Within minutes of lifecycle change**

**Morning (HR Action):**
```
Time: 9:00 AM

HR updates Workday:
  employeeStatus: "Active" → "Terminated"
  terminationDate: 2024-02-14
  lastWorkDay: 2024-02-14
```

**Midday (Aggregation and Detection):**
```
Time: 12:00 PM (scheduled aggregation)

Workday aggregates to ISC:
  - Employee data received
  - terminationDate = today
  
Identity Profile evaluates:
  IF terminationDate <= NOW()
    → cloudLifecycleState = "inactive"
    
Lifecycle Change Detected:
  Previous: active
  Current: inactive
  Change: active → inactive
```

**Immediate (Leaver Workflow Execution):**
```
Time: 12:05 PM (5 minutes after detection)

Leaver Workflow Triggers:

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

PRIORITY 1: SECURITY-CRITICAL (Immediate - 0-2 minutes)

1. Disable Active Directory Account
   Time: 12:05 PM
   Action: Set userAccountControl to disabled
   Result:
     ✓ Account disabled
     ✓ Password no longer works
     ✓ Cannot log in
     ✓ Groups retained (not removed yet)
     ✓ Data preserved
   
2. Revoke VPN Access
   Time: 12:05 PM
   Action: Remove from VPN-Users group
   Result:
     ✓ VPN access revoked
     ✓ Active VPN sessions terminated
     ✓ Cannot reconnect
   
3. Disable Building Access Badge
   Time: 12:05 PM
   Action: Deactivate badge in access control system
   Result:
     ✓ Badge won't open doors
     ✓ Cannot enter building
     ✓ Security notified

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

PRIORITY 2: COMMUNICATION SYSTEMS (0-5 minutes)

4. Configure Email Account
   Time: 12:06 PM
   Actions:
     ✓ Disable direct access (cannot log in)
     ✓ Forward to manager: jane.doe@company.com
     ✓ Set automatic reply:
       "I am no longer with Company.
        For assistance, please contact Jane Doe."
   Result:
     ✗ John cannot access email
     ✓ Manager receives forwarded emails
     ✓ Senders notified
   
5. Disable Slack Account
   Time: 12:07 PM
   Action: Deactivate user
   Result:
     ✓ Cannot send messages
     ✓ Cannot access channels
     ✓ Profile shows: Deactivated

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

PRIORITY 3: BUSINESS APPLICATIONS (0-10 minutes)

6. Disable Salesforce Account
   Time: 12:08 PM
   Actions:
     ✓ Set account to inactive
     ✓ Revoke Sales Cloud license
     ✓ Transfer open opportunities to manager
   Result:
     ✗ Cannot access Salesforce
     ✓ License freed (cost savings)
     ✓ Data transferred
   
7. Revoke Office 365 Access
   Time: 12:09 PM
   Actions:
     ✓ Disable account
     ✓ Revoke licenses (E3 suite)
     ✓ Preserve mailbox (litigation hold if needed)
   Result:
     ✗ Cannot access Teams, OneDrive, etc.
     ✓ Mailbox preserved for transition
   
8. Disable Other Applications
   Time: 12:10 PM
   Applications:
     ✓ Zoom: Account deactivated
     ✓ LinkedIn Sales Navigator: License revoked
     ✓ CRM tools: Access removed
     ✓ Sales tools: Accounts disabled

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

PRIORITY 4: NOTIFICATIONS (10-15 minutes)

9. Notifications Sent
   Time: 12:15 PM
   
   To Manager (Jane Doe):
     Subject: "John Smith Termination - Action Required"
     Content:
       - John's last day: Today
       - Email forwarded to you
       - Please collect equipment
       - File transfer needed
       - Ticket: TERM-12345
   
   To IT Department:
     Subject: "Termination Completed - John Smith"
     Content:
       - All access disabled
       - Equipment collection needed
       - Laptop location: Desk 4B
       - Ticket: TERM-12345
   
   To HR:
     Subject: "IT Termination Complete - John Smith"
     Content:
       - All access revoked
       - Equipment pending collection
       - Exit interview can proceed
   
   To Security:
     Subject: "Badge Deactivated - John Smith"
     Content:
       - Building access revoked
       - Badge should be collected

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

SUMMARY - Day 0 (Complete by 12:15 PM):

Status: ALL ACCESS DISABLED
Duration: 15 minutes from lifecycle change
Security: Immediate - no access to any systems
Data: Preserved for transition
```

---

**STAGE 2: TRANSITION PERIOD (Day 1-30)**

**Purpose: Data handoff and transition**

**Day 0-7 (Week 1):**
```
Access Status:
  ✗ John: No access to anything
  ✓ Manager: Receives forwarded emails
  ✓ IT: Can access John's files for transfer
  
Activities:
  - Manager reviews forwarded emails
  - Manager retrieves important files
  - IT transfers files to shared drive
  - Equipment collected (laptop, phone, badge)
  
Accounts Status:
  - AD account: Disabled (not deleted)
  - Email: Disabled, forwarded
  - Apps: Disabled
  - Data: Fully accessible for transition
```

**Day 8-30 (Weeks 2-4):**
```
Access Status:
  Status unchanged (still disabled)
  
Activities:
  - Final file transfers completed
  - Email forwarding may continue
  - Any remaining data needs addressed
  
Accounts Status:
  - Still disabled
  - Data still accessible if needed
  - Preparing for next stage
```

---

**STAGE 3: RETENTION PERIOD END (Day 30)**

**Email Conversion:**
```
Time: Day 30 (March 15, 2024)

Email Mailbox Actions:
  1. Stop forwarding to manager
  2. Convert mailbox to "shared mailbox"
     - No longer assigned to user
     - No license cost
     - Searchable for compliance
     - Accessible to authorized admins
  3. Set retention policy
     - Keep for 7 years (compliance)
     - Auto-delete after retention
     
Result:
  ✓ Email preserved for compliance
  ✓ No license cost
  ✓ Available if needed
  ✓ Forwarding stopped
```

---

**STAGE 4: ACCOUNT DELETION (Day 90)**

**Final Cleanup:**
```
Time: Day 90 (May 14, 2024)

Automated Cleanup Job Runs:

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

1. Pre-Deletion Checks
   ✓ Verify 90 days since termination
   ✓ Check for legal holds: None
   ✓ Check for pending tickets: None
   ✓ Verify retention requirements met
   ✓ Confirm no active dependencies
   
2. Active Directory Account Deletion
   Action: Remove-ADUser -Identity john.smith
   Result:
     ✓ Account deleted from AD
     ✓ All attributes removed
     ✓ Group memberships removed
     ✓ Cannot be recovered (except from backup)
     
3. Application Account Deletion
   Salesforce:
     ✓ Account permanently deleted
     ✓ Data transferred/archived
     ✓ License freed
     
   Slack:
     ✓ Account removed
     ✓ Message history preserved (workspace-level)
     
   Other Apps:
     ✓ Systematic deletion
     ✓ Per application policy
     
4. File System Cleanup
   Home Drive:
     ✓ Files archived to backup
     ✓ Home folder deleted
     
   OneDrive:
     ✓ Files transferred to manager
     ✓ OneDrive deleted
     
5. Physical Cleanup
   ✓ Badge permanently deactivated
   ✓ Phone number released
   ✓ Equipment wiped and returned to inventory

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

SUMMARY - Day 90:

Status: ALL ACCOUNTS DELETED
Data: Archived per retention policy
Physical: All equipment processed
Compliance: Retention requirements met
```

---

**STAGE 5: LONG-TERM RETENTION (Day 90+)**

**What Remains:**
```
ISC Identity Record:
  Status: Preserved indefinitely
  Purpose: Audit trail, compliance, reporting
  Contains:
    ✓ Name: John Smith
    ✓ Employee ID: EMP12345
    ✓ Department: Sales (at termination)
    ✓ Termination date: 2024-02-14
    ✓ cloudLifecycleState: inactive
    ✓ Account history (what accounts existed)
    ✓ Access history (what access had)
    ✓ Certification history
  
Email Archive:
  Status: Retained for 7 years
  Location: Shared mailbox / compliance archive
  Purpose: Legal, compliance, discovery
  Auto-delete: After 7 years (2031)
  
Audit Logs:
  Status: Retained per policy (typically 7 years)
  Contains:
    ✓ All de-provisioning actions
    ✓ Timestamps
    ✓ Who performed actions
    ✓ What was changed
  Purpose: Compliance, audit, investigations
```

---

**COMPLETE TIMELINE SUMMARY:**
```
Day 0 (Feb 14): IMMEDIATE DE-PROVISIONING
  Time: 12:05-12:15 PM
  Actions:
    ✓ All access disabled (10 minutes)
    ✓ Accounts suspended
    ✓ Data preserved
    ✓ Notifications sent
  Result: Complete access revocation
  
Day 1-30: TRANSITION PERIOD
  Status: Disabled, not deleted
  Purpose: Data handoff to manager
  Activities: File transfer, email review
  
Day 30 (Mar 15): EMAIL CONVERSION
  Action: Convert to shared mailbox
  Result: License freed, data preserved
  
Day 90 (May 14): ACCOUNT DELETION
  Action: Permanent account removal
  Result: All accounts deleted
  
Day 90+: LONG-TERM RETENTION
  Preserved: ISC identity, email archive, audit logs
  Purpose: Compliance, legal, audit trail
  Retention: 7 years typical
```

**Key Principles:**
✓ Security First:

Disable immediately (Day 0)
No delays acceptable
Within minutes of detection

✓ Data Preservation:

Don't delete immediately
Allow transition period
Support business continuity

✓ Scheduled Cleanup:

Delete after retention (Day 90)
Automated process
Consistent enforcement

✓ Audit Trail:

Preserve identity record
Keep audit logs
Compliance documentation


**Exam Tip:** De-provisioning is two-stage: Disable immediately (security), Delete after retention period (cleanup). Day 0 = disable all access, Day 90 = delete accounts, Always preserve audit trail in ISC.
</details>

---

## Section Summary

**Core Concepts for Certification:**

✅ **Provisioning Triggers:**
- **Lifecycle change**: prehire → active (joiner), active → inactive (leaver)
- **Access request**: User requests and gets approved
- **Workflow**: Automated based on rules (department, role)
- **Manual**: Admin provisions (exceptions)
- **Role/Profile**: Assignment triggers provisioning

✅ **Provisioning Lifecycle:**
1. **Trigger**: Event requiring provisioning
2. **Planning**: ISC determines what to provision
3. **Execution**: ISC provisions on target systems
4. **Verification**: Confirm success or detect failure
5. **Notification**: Inform stakeholders
6. **Audit**: Log for compliance

✅ **Provisioning vs. De-provisioning:**
- **Provisioning**: Grant access (create, add, enable)
- **De-provisioning**: Revoke access (delete, remove, disable)
- **De-provisioning urgency**: Immediate (security critical)
- **Provisioning urgency**: Hours acceptable

✅ **De-provisioning Methods:**
- **Disable (Soft)**: Account suspended, data preserved, reversible
- **Delete (Hard)**: Account removed, data gone, permanent
- **Best Practice**: Disable Day 0, Delete Day 30-90

✅ **Timeline:**
- **Day 0**: Disable all access (immediate)
- **Day 1-30**: Transition period (data preserved)
- **Day 30**: Email converted to shared mailbox
- **Day 90**: Accounts permanently deleted
- **Long-term**: ISC identity and logs preserved

**Exam Focus Areas:**
- Know five provisioning triggers
- Understand six-phase provisioning lifecycle
- Know de-provisioning is more urgent than provisioning
- Understand disable vs. delete
- Remember: Disable immediately, delete after retention
- Know typical timeline: Day 0 disable, Day 90 delete

---

**End of Section 5.1: Provisioning Fundamentals**

**Completed:**
- ✅ 5.1.1: How Provisioning is Triggered
- ✅ 5.1.2: Provisioning Lifecycle Overview
- ✅ 5.1.3: Provisioning vs. De-provisioning

**Next:** Section 5.2: Source Provisioning Components

---

**Study Tips:**
- Remember: 5 triggers (lifecycle, request, workflow, manual, role/profile)
- Lifecycle: 6 phases (trigger, plan, execute, verify, notify, audit)
- De-provisioning: More urgent than provisioning (security)
- Two-stage de-provisioning: Disable then delete
- Timeline: Day 0 = disable, Day 90 = delete
