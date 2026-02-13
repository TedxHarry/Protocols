# Section 3.3: Account Correlation and Management

**Master account correlation, handle uncorrelated accounts, and understand account deletion processes**

---

## Table of Contents

- [3.3.1 Understanding Uncorrelated Accounts](#331-understanding-uncorrelated-accounts)
- [3.3.2 Correlation Rules and Logic](#332-correlation-rules-and-logic)
- [3.3.3 Account Deletion Process](#333-account-deletion-process)
- [3.3.4 Impact of Deletion on Aggregation](#334-impact-of-deletion-on-aggregation)

---

# 3.3.1 Understanding Uncorrelated Accounts

**Learning Objective:** Understand what uncorrelated accounts are, why they occur, and how to manage them.

---

## What is an Uncorrelated Account?

**Definition:** An account that exists in a source system but isn't linked to any identity in ISC

**Visual representation:**
```
ISC Identities          Source Accounts
┌──────────────┐       ┌──────────────┐
│ John Smith   │◄──────│ jsmith       │ ✓ Correlated
│              │       │              │
│ Jane Doe     │◄──────│ jdoe         │ ✓ Correlated
│              │       │              │
│ Bob Wilson   │◄──────│ bwilson      │ ✓ Correlated
└──────────────┘       │              │
                       │ orphan123    │ ✗ Uncorrelated (no link)
                       │              │
                       │ svc_backup   │ ✗ Uncorrelated (no link)
                       └──────────────┘
```

**What this means:**
- ISC found the account during aggregation
- ISC couldn't match it to any existing identity
- Account exists but is "orphaned" in ISC

**Exam Tip:** Uncorrelated = account exists but not linked to an identity.

---

## Why Accounts Become Uncorrelated

### **Reason 1: Account Created Outside ISC**

**Scenario:**
```
Admin manually creates account in Active Directory
  ↓
Account bypasses ISC provisioning process
  ↓
No identity created in ISC
  ↓
Aggregation finds account but no matching identity
  ↓
Result: Uncorrelated account
```

**Example:**
```
AD Admin creates:
  Account: temp_contractor
  For: Temporary contractor
  Manually in AD (urgent need)
  
ISC doesn't know:
  - Who this person is
  - No identity exists
  - Account shows as uncorrelated
```

### **Reason 2: User Left Company**

**Scenario:**
```
Employee terminated
  ↓
Identity disabled/removed in ISC
  ↓
Account still active in AD (not yet disabled)
  ↓
Aggregation finds account with no matching identity
  ↓
Result: Uncorrelated (orphaned) account
```

**Example:**
```
Employee: Sarah Johnson terminated
  
In ISC:
  Identity "Sarah Johnson" → Disabled or deleted
  
In AD:
  Account "sjohnson" → Still active (not synced yet)
  
Aggregation result:
  sjohnson → Uncorrelated (identity gone)
```

### **Reason 3: Service Accounts**

**Scenario:**
```
Service account created for application/service
  ↓
Not a real person → No identity needed
  ↓
Aggregation finds account
  ↓
Result: Uncorrelated (by design)
```

**Examples:**
```
Service accounts:
  - svc_backup
  - svc_monitoring  
  - svc_sqlserver
  - app_webserver
  
These are NOT people:
  - No identity in HR system
  - No identity in ISC
  - Uncorrelated is expected
```

### **Reason 4: Correlation Rule Doesn't Match**

**Scenario:**
```
Correlation rule: Match on email
  ↓
Account email: john.smith@company.com
Identity email: jsmith@company.com (typo)
  ↓
Emails don't match
  ↓
Result: Uncorrelated (rule failed to match)
```

**Example:**
```
Account (from AD):
  sAMAccountName: jsmith
  mail: john.smith@oldcompany.com (old email domain)
  
Identity (in ISC):
  Name: John Smith
  email: john.smith@company.com (new email domain)
  
Correlation rule matches on email:
  john.smith@oldcompany.com ≠ john.smith@company.com
  → NO MATCH → Uncorrelated
```

### **Reason 5: Shared/Generic Accounts**

**Scenario:**
```
Shared account used by multiple people
  ↓
Not tied to single person
  ↓
Hard to correlate to one identity
  ↓
Result: Uncorrelated
```

**Examples:**
```
Shared accounts:
  - admin_shared
  - helpdesk_queue
  - reception_desk
  - info@company.com
  
Can't map to single identity:
  - Multiple people use it
  - Not individual account
  - Uncorrelated by nature
```

### **Reason 6: Test/Temporary Accounts**

**Scenario:**
```
Test account created for testing
  ↓
Not a real user
  ↓
No identity created
  ↓
Result: Uncorrelated
```

**Examples:**
```
Test accounts:
  - test_user1
  - demo_account
  - training_user
  - qa_test
```

---

## Impact of Uncorrelated Accounts

### **Security Risks**

**1. Unknown Access**
```
Problem:
  - Don't know who has the account
  - Can't track what they're accessing
  - Can't review in certifications
  
Risk:
  - Unauthorized access
  - No accountability
  - Compliance violations
```

**2. Orphaned Access**
```
Problem:
  - Ex-employee account still active
  - Still has access to systems
  - No one responsible for it
  
Risk:
  - Data breach potential
  - Terminated employees retaining access
  - Audit findings
```

### **Operational Issues**

**1. Cannot Manage**
```
Can't provision because:
  - No identity to provision to
  - Workflows don't apply
  - Access requests don't work
```

**2. Incomplete Reporting**
```
Reports show incomplete data:
  - Access reports missing these accounts
  - Certification campaigns can't include them
  - Analytics incomplete
```

### **Compliance Concerns**
```
Audit questions:
  "Who has access to this sensitive system?"
  
If uncorrelated accounts exist:
  - Can't definitively answer
  - Shows lack of control
  - Compliance violation
  - Failed audit finding
```

**Exam Tip:** Uncorrelated accounts are security and compliance risks. Must be investigated.

---

## Identifying Uncorrelated Accounts

### **In ISC UI**

**Location:**
```
Admin → Accounts → Filter by "Uncorrelated"

Or:

Admin → Connections → Sources → [Source] → Accounts → Uncorrelated
```

**What you'll see:**
```
Uncorrelated Accounts (150)

Account Name    Source              Created        Last Aggregation
orphan123       Active Directory    2023-05-10     2024-02-13
svc_backup      Active Directory    2020-01-15     2024-02-13
temp_user       Active Directory    2024-02-01     2024-02-13
sjohnson        Active Directory    2022-08-20     2024-02-13
admin_shared    Active Directory    2019-03-12     2024-02-13
...
```

**Account details:**
```
Click on account to see:
  Account Name: orphan123
  Source: Active Directory
  Status: UNCORRELATED
  Created: 2023-05-10
  Attributes:
    - displayName: Orphaned Account
    - mail: orphan123@company.com
    - department: IT
    - enabled: True
  
  Correlation Status: No matching identity found
  Suggested Actions:
    - Correlate to existing identity
    - Create new identity
    - Mark as service account
    - Disable account
```

### **Via Search**

**Search for uncorrelated accounts:**
```
Search query:
  @accounts(uncorrelated:true)

Returns:
  All accounts across all sources with uncorrelated status
```

### **Via Reports**

**Create uncorrelated accounts report:**
```
Report: Uncorrelated Accounts by Source

Columns:
  - Account Name
  - Source
  - Created Date
  - Last Login
  - Status (enabled/disabled)
  - Suggested Category

Filters:
  - By source
  - By date range
  - By status
```

---

## Managing Uncorrelated Accounts

### **Decision Tree**
```
Is this account for a real person?
├─ Yes → Correlate to identity
│   ├─ Identity exists? → Manual correlation
│   └─ No identity? → Create identity, then correlate
│
├─ No → What type?
│   ├─ Service account → Tag and exclude from reviews
│   ├─ Shared account → Document ownership, special handling
│   ├─ Test account → Disable or delete
│   └─ Orphaned (user left) → Disable or delete
```

### **Action 1: Correlate to Identity**

**When to use:**
- Real person's account
- Identity exists in ISC
- Correlation just failed

**Steps:**
```
1. Find account in ISC
   Admin → Accounts → orphan123

2. Click "Correlate"

3. Search for identity
   Search: "John Smith" or "john.smith@company.com"

4. Select matching identity
   Identity: John Smith (ID: 2c918...)

5. Confirm correlation
   Click "Correlate Account"

6. Verify
   Account now shows: Correlated
   Identity now shows: Has account "orphan123"
```

### **Action 2: Create Identity and Correlate**

**When to use:**
- Real person but no identity exists
- Missed in HR feed
- External contractor

**Steps:**
```
1. Create identity manually
   Admin → Identities → Create
   
2. Enter details:
   First Name: John
   Last Name: Smith
   Email: john.smith@company.com
   Employee ID: (if applicable)

3. Save identity

4. Correlate account to new identity
   Follow correlation steps above
```

### **Action 3: Mark as Service Account**

**When to use:**
- Application service account
- Not a person
- Needs to remain uncorrelated

**Steps:**
```
1. Tag account
   Account → Edit → Tags
   Add tag: "Service Account"

2. Add to group (if available)
   Create group: "Service Accounts"
   Add account to group

3. Exclude from reports
   In certification campaigns:
     Exclude group: "Service Accounts"
   
   In reports:
     Filter out: tags contains "Service Account"

4. Document
   Account → Notes
   "Service account for backup service. Owner: IT Ops"
```

### **Action 4: Disable or Delete**

**When to use:**
- Orphaned account (user left)
- Test account no longer needed
- Temporary account expired

**Steps:**
```
1. Verify account should be removed
   - Check with account owner
   - Verify user really left
   - Confirm no longer needed

2. Disable account first (safer)
   In source system (e.g., AD):
     Disable account
   
   Or via ISC (if provisioning enabled):
     Manual provisioning → Disable

3. Monitor (30-90 days)
   Ensure nothing breaks
   Verify not needed

4. Delete if still not needed
   Permanently remove from source
```

---

## Best Practices for Uncorrelated Accounts

### **Prevention**

**1. Enforce ISC Provisioning**
```
Policy:
  ✓ All accounts created via ISC
  ✓ No manual account creation in sources
  ✓ Exceptions require approval
  
Result:
  - Every account has identity
  - No uncorrelated accounts (except service)
```

**2. Regular Reviews**
```
Schedule:
  - Weekly: Quick scan
  - Monthly: Full review
  - Quarterly: Deep dive with source owners
  
Process:
  1. Export uncorrelated list
  2. Categorize each account
  3. Take appropriate action
  4. Track trends
```

**3. Improve Correlation Rules**
```
Instead of just email:
  - Use employee ID (more reliable)
  - Use multiple attributes
  - Handle edge cases
  
Result:
  - Fewer false uncorrelated
  - Better automatic correlation
```

**4. Service Account Management**
```
Process:
  1. Naming convention: svc_*, app_*, etc.
  2. Auto-tag based on name pattern
  3. Regular inventory
  4. Documented ownership
  
Result:
  - Expected uncorrelated accounts
  - Well-managed service accounts
```

### **Handling**

**Triage approach:**
```
Priority 1 - High Risk (Immediate):
  - Accounts with admin/privileged access
  - Accounts accessing sensitive data
  - Recently active accounts
  
Priority 2 - Medium Risk (This Week):
  - Standard user accounts
  - Accounts not recently active
  - Non-sensitive access
  
Priority 3 - Low Risk (This Month):
  - Disabled accounts
  - Test/temp accounts
  - Service accounts (known)
```

---

## Practice Exercise

**Question:** During your monthly review, you find 75 uncorrelated accounts in Active Directory. How would you categorize and handle them?

<details>
<summary>Answer</summary>

**Step-by-Step Approach:**

**Step 1: Export and Analyze**
```
Export uncorrelated accounts to spreadsheet:
  - Account name
  - Display name
  - Email
  - Created date
  - Last login
  - Enabled/disabled status
  - Group memberships
```

**Step 2: Categorize by Pattern**

**Category A: Service Accounts (identify by naming)**
```
Pattern: svc_*, app_*, sql*, backup*, etc.

Examples found:
  - svc_backup
  - svc_monitoring
  - app_webserver
  - sqlserver_svc
  
Count: 25 accounts

Action:
  ✓ Verify with IT team (legitimate?)
  ✓ Tag as "Service Account"
  ✓ Document ownership
  ✓ Exclude from user reviews
  ✓ Create separate service account review process
```

**Category B: Orphaned Accounts (user left)**
```
Pattern: 
  - Created >6 months ago
  - Last login >90 days ago
  - OR no login ever
  - Email matches ex-employee pattern

Examples found:
  - sjohnson (Sarah Johnson - terminated)
  - mlee (Mike Lee - left company)
  
Count: 30 accounts

Action:
  ✓ Verify with HR (really terminated?)
  ✓ Disable immediately in AD
  ✓ Document in ticket
  ✓ Schedule deletion after 90 days
  ✓ Review what access they had
```

**Category C: Correlation Failures (real users)**
```
Pattern:
  - Recent creation
  - Active last login
  - Looks like real person
  - Email doesn't match exactly

Examples found:
  - jsmith (John Smith - email typo in AD)
  - agarcia (Ana Garcia - new hire, HR hasn't synced)
  
Count: 15 accounts

Action:
  ✓ Correlate to existing identity (if exists)
  ✓ Create identity (if doesn't exist)
  ✓ Fix correlation issue
  ✓ Review why correlation failed
```

**Category D: Shared/Generic Accounts**
```
Pattern:
  - admin, shared, generic, dept names
  - Multiple people use
  
Examples found:
  - admin_shared
  - helpdesk_queue
  - reception_desk
  
Count: 5 accounts

Action:
  ✓ Document ownership
  ✓ Tag as "Shared Account"
  ✓ Assign to manager/owner
  ✓ Include in special review
  ✓ Consider if still needed
```

**Step 3: Execute Actions**

**Week 1: High Priority (Orphaned)**
```
Day 1-2: Disable 30 orphaned accounts
  - Verify with HR each person left
  - Disable in AD
  - Document reason
  - Create tickets for deletion
  
Result: Immediate security improvement
```

**Week 2: Service Accounts**
```
Day 3-5: Process 25 service accounts
  - Meet with IT teams
  - Verify legitimacy
  - Document each account:
    * Purpose
    * Owner
    * Application/service
    * Required access
  - Tag appropriately
  - Exclude from user certifications
  
Result: Service accounts properly managed
```

**Week 3: Correlate Real Users**
```
Day 6-8: Fix 15 correlation failures
  - 10 correlate to existing identities
  - 5 create new identities (contractors, missed in HR)
  - Fix root cause:
    * Update correlation rule
    * Fix email typos in AD
    * Improve HR feed
  
Result: Accounts properly linked
```

**Week 4: Shared Accounts**
```
Day 9-10: Handle 5 shared accounts
  - Document purpose and ownership
  - Determine if still needed
  - Disable 2 (no longer used)
  - Keep 3 with proper documentation
  
Result: Shared accounts managed
```

**Step 4: Prevention Plan**

**Improve Processes:**
```
1. Block manual AD account creation
   - Require ISC provisioning
   - Exception process for emergencies
   
2. Auto-tag service accounts
   - Naming convention enforcement
   - Auto-detect and tag svc_*, app_*
   
3. Better correlation
   - Use employee ID + email
   - Handle email domain changes
   - Regular correlation rule review
   
4. Faster offboarding
   - Automate based on HR termination
   - Same-day account disabling
   
5. Monthly reviews
   - Scheduled task
   - Owner assigned
   - Metrics tracked
```

**Step 5: Track Progress**

**Metrics to monitor:**
```
Month 1 (Before):
  Uncorrelated accounts: 75
  
Month 2 (After cleanup):
  Uncorrelated accounts: 25 (service accounts only)
  
Ongoing:
  New uncorrelated per month: <5
  Time to resolution: <1 week
  
Success criteria:
  ✓ <30 uncorrelated (excluding service accounts)
  ✓ All categorized and documented
  ✓ No orphaned accounts >7 days
```

**Final Summary:**
```
75 Uncorrelated Accounts:
  ✓ 25 Service accounts → Tagged, documented
  ✓ 30 Orphaned → Disabled, scheduled deletion
  ✓ 15 Real users → Correlated
  ✓ 5 Shared → 2 disabled, 3 documented

Result:
  - Zero unknown accounts
  - All categorized
  - Security improved
  - Compliance satisfied
  - Prevention measures in place
```

**Exam Tip:** Categorize uncorrelated accounts: service, orphaned, correlation failures, or shared. Handle each type appropriately.
</details>

---

# 3.3.2 Correlation Rules and Logic

**Learning Objective:** Understand how correlation rules work and how to configure them effectively.

---

## What are Correlation Rules?

**Definition:** Logic that determines how ISC matches accounts to identities

**Purpose:**
```
During aggregation:
  Account found: jsmith
  
Correlation rule asks:
  "Which identity does this account belong to?"
  
Correlation rule uses:
  Account attributes + Identity attributes
  
If match found:
  Link account to identity (correlated)
  
If no match:
  Flag as uncorrelated
```

---

## Common Correlation Methods

### **Method 1: Email Address Matching (Most Common)**

**Rule:**
```
IF account.email == identity.email
THEN correlate
```

**Example:**
```
Account (from AD):
  sAMAccountName: jsmith
  mail: john.smith@company.com
  
Identity (in ISC):
  Name: John Smith
  email: john.smith@company.com
  
Comparison:
  john.smith@company.com == john.smith@company.com
  ✓ MATCH
  
Result: Account correlated to identity
```

**Advantages:**
- Email is usually unique
- Available in most systems
- Users don't change email often

**Disadvantages:**
- Email changes when person changes name
- Email domain migrations cause mismatches
- External users may have different domains

### **Method 2: Employee ID Matching (Most Reliable)**

**Rule:**
```
IF account.employeeID == identity.employeeId
THEN correlate
```

**Example:**
```
Account (from AD):
  sAMAccountName: jsmith
  employeeID: EMP12345
  
Identity (in ISC):
  Name: John Smith
  employeeId: EMP12345
  
Comparison:
  EMP12345 == EMP12345
  ✓ MATCH
  
Result: Account correlated
```

**Advantages:**
- Employee ID never changes
- Unique per person
- Survives name changes, email changes
- Most reliable method

**Disadvantages:**
- Not all systems store employee ID
- External contractors may not have employee ID
- Requires synchronization from HR

**Exam Tip:** Employee ID is most reliable. Email is most common. Use employee ID when available.

### **Method 3: Username Matching**

**Rule:**
```
IF account.username == identity.uid
THEN correlate
```

**Example:**
```
Account (from AD):
  sAMAccountName: jsmith
  
Identity (in ISC):
  uid: jsmith
  
Comparison:
  jsmith == jsmith
  ✓ MATCH
```

**Advantages:**
- Simple
- Direct match

**Disadvantages:**
- Different systems may use different usernames
- Username may not be unique across systems
- Not reliable across multiple sources

### **Method 4: Composite Matching**

**Rule:**
```
IF account.firstName == identity.firstName
AND account.lastName == identity.lastName
AND account.department == identity.department
THEN correlate
```

**Example:**
```
Account:
  firstName: John
  lastName: Smith
  department: IT
  
Identity:
  firstName: John
  lastName: Smith
  department: IT
  
Comparison:
  John == John ✓
  Smith == Smith ✓
  IT == IT ✓
  
Result: All match → Correlate
```

**Advantages:**
- Can work without unique identifier
- Multiple attributes increase confidence

**Disadvantages:**
- Not unique (multiple John Smiths in IT)
- Attributes change (department, name)
- Complex to maintain
- False positives possible

---

## Correlation Rule Configuration

### **Basic Rule Structure**

**In ISC correlation configuration:**
```
Source: Active Directory
Correlation Attribute: email

Mapping:
  Account Attribute → Identity Attribute
  mail → email

Logic:
  IF account.mail equals identity.email
  THEN link account to that identity
```

### **Multi-Attribute Rules**

**Configuration:**
```
Source: Active Directory

Primary Correlation: Employee ID
  Account: employeeNumber
  Identity: employeeId
  Logic: Exact match

Fallback Correlation: Email
  Account: mail
  Identity: email
  Logic: Exact match (if employee ID not available)

Process:
  1. Try employee ID match first
  2. If no match, try email
  3. If still no match, uncorrelated
```

### **Case Sensitivity**

**Important consideration:**
```
Account email: John.Smith@company.com
Identity email: john.smith@company.com

Case-sensitive match:
  John.Smith != john.smith
  ✗ NO MATCH (different case)

Case-insensitive match:
  john.smith == john.smith
  ✓ MATCH (ignore case)
```

**Best practice:**
```
Always use case-insensitive matching for:
  - Email addresses
  - Usernames
  - Any text fields

Keep case-sensitive for:
  - Employee IDs (if case matters)
  - Numeric IDs
```

---

## Handling Correlation Edge Cases

### **Edge Case 1: Multiple Matches**

**Problem:**
```
Account: jsmith
Account email: john.smith@company.com

Two identities found:
  1. John Smith (IT) - email: john.smith@company.com
  2. John Smith (HR) - email: john.smith@company.com

Which one to correlate to?
```

**Solution:**
```
Use additional attributes to disambiguate:

Add secondary match:
  Account.department == Identity.department
  
Account department: IT
  
Identity 1: Department = IT ✓ MATCH
Identity 2: Department = HR ✗ NO MATCH

Result: Correlate to Identity 1 (John Smith IT)
```

### **Edge Case 2: No Match Due to Data Quality**

**Problem:**
```
Account email: john.smith@company.com
Identity email: jsmith@company.com (typo in HR system)

Correlation fails due to data quality issue
```

**Solutions:**
```
Option 1: Fix data at source
  - Correct email in HR system
  - Re-aggregate
  - Should correlate

Option 2: Manual correlation
  - Admin correlates manually
  - Documents issue
  - Fixes root cause

Option 3: Flexible matching
  - Use employee ID instead
  - More tolerant of email typos
```

### **Edge Case 3: Changed Attributes**

**Problem:**
```
User gets married, changes last name:
  Before: Jane Smith → jane.smith@company.com
  After: Jane Wilson → jane.wilson@company.com

Account still has old email:
  Account: jane.smith@company.com
  Identity: jane.wilson@company.com
  
Correlation breaks!
```

**Solution:**
```
Use employee ID (doesn't change):
  Account: employeeID = EMP54321
  Identity: employeeId = EMP54321
  ✓ Still matches after name change

Or: Update account email via provisioning
  ISC detects email change in identity
  Provisions new email to account
  Correlation maintained
```

---

## Testing Correlation Rules

### **Before Production**

**Test process:**
```
1. Configure correlation rule in test/sandbox

2. Run test aggregation
   - Small sample (100 accounts)
   - Known good data
   
3. Review results:
   - How many correlated?
   - How many uncorrelated?
   - Any false positives?
   - Any missed matches?

4. Adjust rule if needed

5. Test again with larger sample

6. Deploy to production when confident
```

### **Validation Queries**

**Check correlation results:**
```
Search: All accounts from AD, correlated
  @accounts(source.name:"Active Directory" AND correlated:true)
  
Expected: Most accounts should be correlated

Search: All accounts from AD, uncorrelated
  @accounts(source.name:"Active Directory" AND uncorrelated:true)
  
Expected: Only service accounts, shared accounts
```

---

## Correlation Rule Best Practices

### **Design Principles**

**1. Use Most Reliable Attribute**
```
Priority Order:
  1st choice: Employee ID (never changes)
  2nd choice: Email (changes sometimes)
  3rd choice: Username (varies by system)
  Last resort: Composite (name + dept)
```

**2. Have Fallback Method**
```
Primary: Employee ID
Fallback: Email
Last resort: Manual

Example:
  Try employee ID → No match
  Try email → Match found ✓
  Correlate successfully
```

**3. Handle Case Sensitivity**
```
✓ Case-insensitive for text:
  john.smith@company.com == John.Smith@company.com

✓ Trim whitespace:
  "john.smith@company.com " == "john.smith@company.com"
```

**4. Document Rules**
```
Documentation:
  Source: Active Directory
  Correlation: Employee ID (employeeNumber → employeeId)
  Fallback: Email (mail → email)
  Case: Insensitive
  Updated: 2024-02-01
  Owner: IAM Team
```

### **Maintenance**

**Regular reviews:**
```
Quarterly:
  ✓ Check correlation success rate
  ✓ Review uncorrelated accounts
  ✓ Analyze trends
  ✓ Update rules if needed

After changes:
  ✓ Email domain migration
  ✓ HR system change
  ✓ New source added
  ✓ Test and adjust rules
```

---

## Practice Exercise

**Question:** You're configuring correlation for a new Active Directory source. Your HR system provides employee ID and email. AD has employeeID and mail attributes. What correlation rule would you configure and why?

<details>
<summary>Answer</summary>

**Recommended Correlation Rule:**

**Primary Method: Employee ID**
```
Configuration:
  Source: Active Directory
  
  Primary Correlation:
    Account Attribute: employeeID (from AD)
    Identity Attribute: employeeId (from ISC/HR)
    Match Type: Exact match
    Case Sensitive: Yes (employee IDs are usually alphanumeric)
```

**Why Employee ID Primary:**
```
✓ Most reliable identifier
✓ Never changes (even with name changes, marriage, etc.)
✓ Unique per person
✓ Available in both systems (AD and HR)
✓ Survives email migrations
✓ Survives username changes
```

**Fallback Method: Email**
```
Configuration:
  Fallback Correlation (if employee ID not present):
    Account Attribute: mail (from AD)
    Identity Attribute: email (from ISC/HR)
    Match Type: Exact match
    Case Sensitive: No (emails are case-insensitive)
    Trim Whitespace: Yes
```

**Why Email Fallback:**
```
✓ Handles accounts without employee ID:
  - External contractors
  - Service accounts with email
  - Legacy accounts

✓ Second-most reliable:
  - Usually unique
  - Available in most accounts

✓ Catches edge cases:
  - New employees (employee ID not synced yet)
  - Accounts created before employee ID policy
```

**Complete Rule Logic:**
```
STEP 1: Try Employee ID Match
  IF account.employeeID exists
    AND account.employeeID == identity.employeeId
  THEN
    Correlate account to identity
    STOP

STEP 2: Try Email Match (fallback)
  IF account.mail exists
    AND LOWERCASE(TRIM(account.mail)) == LOWERCASE(TRIM(identity.email))
  THEN
    Correlate account to identity
    STOP

STEP 3: No Match
  Mark account as UNCORRELATED
```

**Implementation Steps:**
```
1. Configure in ISC:
   Admin → Connections → Sources → Active Directory → Correlation
   
2. Set primary correlation:
   Account Attribute: employeeID
   Identity Attribute: employeeId
   
3. Set fallback correlation:
   Account Attribute: mail
   Identity Attribute: email
   Case Insensitive: Yes
   
4. Save configuration

5. Test with small aggregation:
   - Manually trigger aggregation
   - Review ~100 accounts
   - Check correlation results
   - Verify success rate

6. Review results:
   Correlate via employee ID: 95%
   Correlated via email: 3%
   Uncorrelated: 2% (service accounts)
   
7. Investigate uncorrelated:
   - Check if service accounts (expected)
   - Check if data quality issues
   - Fix or document each

8. Deploy to production:
   - Enable scheduled aggregation
   - Monitor first few runs
   - Document configuration
```

**Why NOT Other Methods:**

**Username Matching:**
```
❌ Don't use: account.sAMAccountName == identity.uid

Problems:
  - AD username may not match ISC uid
  - Different naming conventions
  - Not consistent across systems
  - Less reliable than employee ID or email
```

**Name Matching:**
```
❌ Don't use: firstName + lastName

Problems:
  - Not unique (multiple John Smiths)
  - Names change (marriage, legal name change)
  - High risk of false matches
  - Maintenance nightmare
```

**Expected Outcomes:**
After Full Aggregation (10,000 AD accounts):
Correlated via Employee ID: 9,500 (95%)

Standard employees
Clean employee ID in both systems

Correlated via Email: 200 (2%)

External contractors (no employee ID)
Legacy accounts
Accounts created before employee ID policy

Uncorrelated: 300 (3%)

Service accounts (svc_, app_) - Expected
Shared accounts (admin_shared, etc.) - Expected
Test accounts (test_*) - Need cleanup
Data quality issues - Need investigation

Action Items:
✓ Tag 200 service accounts
✓ Document 50 shared accounts
✓ Delete 30 test accounts
✓ Fix 20 data quality issues
✓ Result: <1% truly problematic uncorrelated

**Exam Tip:** Use employee ID as primary correlation method when available. Email as fallback. Document the rule and test before production.
</details>

# 3.3.3 Account Deletion Process

**Learning Objective:** Understand how account deletion works in ISC and source systems.

---

## What is Account Deletion?

**Definition:** The process of removing an account record from a source system and ISC

**Two types of deletion:**

| Type | What Happens | Reversible? |
|------|-------------|-------------|
| **Soft Delete** | Account disabled/marked inactive | Yes - can re-enable |
| **Hard Delete** | Account permanently removed | No - must recreate |

**Exam Tip:** Most organizations use soft delete (disable) rather than hard delete for compliance and audit purposes.

---

## Account Deletion Methods

### **Method 1: Deletion in Source System**

**What happens:**
```
1. Account deleted in source (e.g., AD)
   Admin deletes jsmith account in Active Directory
   
2. Next aggregation runs
   ISC doesn't find jsmith account anymore
   
3. ISC detects account missing
   Account existed before, now gone
   
4. ISC marks account as deleted
   Account status changes to "deleted" in ISC
   
5. Account removed from identity
   Identity no longer shows this account
```

**Example - Active Directory:**
```
Before:
  Identity: John Smith
    Accounts:
      - Active Directory: jsmith
      - Email: john.smith@company.com
      
AD Admin deletes jsmith account
  
After Next Aggregation:
  Identity: John Smith
    Accounts:
      - Email: john.smith@company.com
    
  Account jsmith: Status = DELETED (in ISC history)
```

### **Method 2: Deletion via ISC (De-provisioning)**

**What happens:**
```
1. Trigger in ISC (termination workflow, manual action)
   Employee terminated
   
2. ISC sends delete command to source
   Via provisioning: "Delete account jsmith"
   
3. Source executes deletion
   Active Directory deletes the account
   
4. ISC updates record
   Marks account as deleted
   
5. Next aggregation confirms
   Account no longer appears in source
```

**Example - Termination Workflow:**
```
Employee: John Smith terminated

Workflow executes:
  1. Disable identity in ISC
  2. Send delete command to AD source
  3. AD deletes account jsmith
  4. ISC marks account deleted
  5. Next aggregation: jsmith not found (as expected)
```

### **Method 3: Manual Deletion in ISC**

**What happens:**
```
1. Admin manually deletes account record in ISC
   Admin → Accounts → jsmith → Delete
   
2. Account removed from ISC database
   No longer visible in ISC UI
   
3. Account STILL EXISTS in source
   AD still has jsmith account
   
4. Next aggregation finds it again
   Account reappears in ISC (unless deleted in source too)
```

**Important:** Deleting account in ISC doesn't delete it in source system!

---

## Soft Delete vs Hard Delete

### **Soft Delete (Disable)**

**What it is:**
```
Account not actually deleted
Account marked as disabled/inactive
Account remains in system but can't be used
```

**Active Directory Example:**
```
Disable Account:
  - Set userAccountControl flag to "disabled"
  - Account still exists in AD
  - User cannot log in
  - Account attributes preserved
  - Can be re-enabled later

PowerShell:
  Disable-ADAccount -Identity jsmith
```

**ISC Behavior:**
```
After aggregation of disabled account:
  - Account shows in ISC as "disabled"
  - Still linked to identity
  - Can be tracked and reported
  - Can be re-enabled via provisioning
```

**Advantages:**
- Reversible (can re-enable)
- Preserves data for audit
- Meets compliance requirements
- Safer (no accidental data loss)

**Disadvantages:**
- Disabled accounts accumulate
- Still count in license counts (some systems)
- Requires cleanup process

### **Hard Delete (Remove)**

**What it is:**
```
Account completely removed from system
Account object deleted
No recovery possible
```

**Active Directory Example:**
```
Delete Account:
  - Account object removed from AD
  - All data lost
  - Cannot be recovered (unless from backup)
  - OU membership, groups, attributes all gone

PowerShell:
  Remove-ADUser -Identity jsmith
```

**ISC Behavior:**
```
After aggregation (account not found):
  - ISC detects account missing
  - Marks account as deleted
  - Removes from identity's account list
  - Keeps in history (audit trail)
```

**Advantages:**
- Cleaner (no accumulation)
- Reduces license count
- Definitive removal

**Disadvantages:**
- Irreversible
- Data lost
- May violate compliance (retention policies)
- Risk of accidental deletion

---

## Best Practices for Account Deletion

### **Recommended Approach**

**Step 1: Disable First**
```
When employee leaves:
  1. Disable account immediately
     - Terminates access instantly
     - Preserves data
     - Safe and reversible
  
  Time: Day 0 (termination date)
```

**Step 2: Retention Period**
```
Keep disabled account for retention period:
  - Legal holds: As required
  - Compliance: 90 days to 7 years (varies)
  - Business need: 30-90 days typical
  
  Time: Days 1-90 (example)
```

**Step 3: Delete After Retention**
```
After retention period expires:
  1. Verify no longer needed
  2. Check no legal holds
  3. Hard delete from source
  4. Purge from backups (if required)
  
  Time: After day 90
```

**Example Timeline:**
```
Day 0 (Termination):
  ✓ Disable AD account (soft delete)
  ✓ Disable email (retain mailbox)
  ✓ Revoke all access
  
Day 30:
  ✓ Archive mailbox
  ✓ Verify still disabled
  
Day 90:
  ✓ Check legal holds (none)
  ✓ Delete AD account (hard delete)
  ✓ Delete mailbox
  ✓ Remove from all systems
  
Day 91+ (Next aggregation):
  ✓ ISC detects account gone
  ✓ Marks as deleted
  ✓ Audit trail preserved in ISC
```

### **Industry-Specific Requirements**

**Healthcare (HIPAA):**
```
Retention: Minimum 6 years
Process:
  - Disable immediately
  - Retain all audit logs
  - Delete after retention period
  - Document all deletions
```

**Financial Services (SOX, SEC):**
```
Retention: 7 years typical
Process:
  - Disable on termination
  - Archive data
  - Legal hold support
  - Controlled deletion process
```

**Standard Corporate:**
```
Retention: 30-90 days
Process:
  - Disable on termination
  - 30-90 day hold for business continuity
  - Delete after period
  - Document in ticket
```

---

## Account Deletion Triggers

### **Automatic Triggers**

**1. Lifecycle State Change**
```
Trigger: Identity lifecycle state changes to "inactive"

Workflow:
  1. HR system marks employee as terminated
  2. ISC aggregates termination
  3. Identity lifecycle → inactive
  4. Workflow triggers: "Leaver Workflow"
  5. Workflow disables all accounts
  
Example:
  Employee: John Smith
  HR Status: Terminated
  → ISC lifecycle: active → inactive
  → Disable AD account
  → Disable email
  → Disable Salesforce
```

**2. Identity Deletion**
```
Trigger: Identity deleted in ISC

Workflow:
  1. Identity removed from ISC
  2. All linked accounts processed
  3. Accounts disabled or deleted (per policy)
  
Use case: Rare - usually only for cleanup or data corrections
```

**3. Scheduled Cleanup**
```
Trigger: Scheduled job finds old disabled accounts

Workflow:
  Daily/Weekly job:
    1. Find accounts disabled >90 days
    2. Verify no holds
    3. Delete from source
    4. Document deletion
  
Example:
  Job runs: Every Sunday 3 AM
  Finds: 15 accounts disabled >90 days
  Deletes: All 15 accounts
  Logs: Deletion audit trail
```

### **Manual Triggers**

**1. Admin Initiated**
```
Use case: Immediate removal needed

Process:
  1. Admin identifies account to delete
  2. Verifies authorization
  3. Manually triggers deletion
  4. Documents reason
  
Example:
  Security incident: Compromised account
  → Immediate deletion required
  → Admin manually deletes
  → Incident documented
```

**2. Ticket-Based**
```
Use case: Formal process via ticketing

Process:
  1. Ticket created: "Delete account jsmith"
  2. Approvals obtained
  3. Admin executes deletion
  4. Ticket closed with documentation
```

---

## Handling Special Account Types

### **Service Accounts**

**Consideration:**
```
Service accounts are NOT deleted when "user" leaves
  
Reason:
  - Not tied to individual person
  - Application/service still needs it
  - Managed separately

Management:
  - Regular review (quarterly)
  - Verify still needed
  - Delete only when service retired
```

### **Shared Accounts**

**Consideration:**
```
Shared accounts remain active
  
Process:
  - Remove individual user access
  - Account stays active for others
  - Track who has access
  - Regular access reviews
```

### **Privileged Accounts**

**Consideration:**
```
Extra scrutiny for deletion

Process:
  1. Immediate disable (security)
  2. Extended audit (90 days minimum)
  3. Document all access performed
  4. Secure deletion after retention
  5. Alert security team
```

---

## Account Deletion and Compliance

### **Audit Trail Requirements**

**What to log:**
```
Deletion Event:
  - Who deleted (person or workflow)
  - What was deleted (account name, source)
  - When deleted (timestamp)
  - Why deleted (reason/ticket)
  - Method (manual, workflow, scheduled)
  
Example Log Entry:
  Timestamp: 2024-02-13 10:30:15
  Action: Account Deleted
  Account: jsmith (Active Directory)
  Identity: John Smith
  Deleted By: Leaver Workflow
  Reason: Employee Terminated
  Ticket: TERM-12345
```

**Retention of logs:**
```
Keep deletion logs longer than account data:
  - Account deleted after 90 days
  - Deletion logs kept 7 years
  
Reason: Prove account was properly deleted for compliance
```

### **Right to be Forgotten (GDPR)**

**GDPR deletion requirements:**
```
When user requests deletion:
  1. Disable account immediately
  2. Delete personal data
  3. Anonymize audit logs (where possible)
  4. Confirm deletion to user
  5. Document process
  
Timeline: 30 days maximum

Exceptions:
  - Legal holds
  - Regulatory requirements
  - Legitimate business needs
```

---

## Troubleshooting Account Deletion

### **Issue 1: Account Won't Delete**

**Symptom:**
```
Delete command sent, account still exists in source
```

**Common causes:**
```
1. Insufficient permissions
   - Service account can't delete
   - Need elevated privileges
   
2. Protected account
   - Admin account
   - Built-in account
   - Protected by policy
   
3. Dependencies
   - Account owns objects
   - Manager of other users
   - Must resolve dependencies first
```

**Solution:**
```
1. Check error logs
2. Verify service account permissions
3. Check account protection status
4. Resolve dependencies
5. Retry deletion
```

### **Issue 2: Deleted Account Reappears**

**Symptom:**
```
Account deleted in ISC
Next aggregation: Account back in ISC
```

**Cause:**
```
Account deleted in ISC but NOT in source
  
ISC deletion ≠ Source deletion

Next aggregation finds it in source
Recreates in ISC
```

**Solution:**
```
Delete in source system, not just ISC:
  1. Delete account in AD (or source)
  2. Next aggregation: Account not found
  3. ISC marks as deleted
  4. Account stays gone
```

### **Issue 3: Account Deleted Too Soon**

**Symptom:**
```
Account needed for audit/recovery
Account already deleted
Data lost
```

**Prevention:**
```
Use soft delete (disable) first:
  1. Immediate: Disable account
  2. Retention period: Keep disabled
  3. After retention: Hard delete

Never hard delete immediately unless:
  - Security incident requires it
  - Explicitly documented exception
```

---

## Practice Exercise

**Question:** An employee is terminated today. Walk through the account deletion process from start to finish, including timing and what happens at each stage.

<details>
<summary>Answer</summary>

**Employee Termination: Complete Account Deletion Process**

**Scenario:**
```
Employee: Sarah Johnson
Accounts:
  - Active Directory: sjohnson
  - Email: sarah.johnson@company.com
  - Salesforce: sarah.johnson@company.com
  - VPN: sjohnson

Termination Date: February 13, 2024
```

**Day 0 - Termination Day (February 13)**

**Morning (HR notification):**
```
08:00 - HR marks Sarah as terminated in Workday
  Status: Active → Terminated
  Last Day: February 13, 2024
```

**Midday (ISC aggregation):**
```
12:00 - Workday aggregation runs (scheduled)
  1. ISC aggregates from Workday
  2. Detects Sarah's status = Terminated
  3. Updates identity in ISC:
     cloudLifecycleState: active → inactive
```

**Afternoon (Leaver workflow triggers):**
```
12:05 - Leaver Workflow detects lifecycle change
  
  Step 1: Immediate Disabling (Security Critical)
    ✓ Disable AD account (sjohnson)
      - Account cannot log in
      - Password no longer works
      - VPN access terminated
      
    ✓ Disable email (forward to manager)
      - Can't send/receive new email
      - Manager receives forwarded emails
      - Mailbox preserved
      
    ✓ Disable Salesforce
      - Cannot access Salesforce
      - Data preserved
      - License freed
      
    ✓ Revoke VPN access
      - Remove from VPN group
      - Active sessions terminated
  
  Step 2: Notifications
    ✓ Email to manager: "Sarah Johnson terminated"
    ✓ Email to IT: "Collect equipment"
    ✓ Ticket created: TERM-54321
    
  Step 3: Documentation
    ✓ All actions logged
    ✓ Timestamp recorded
    ✓ Workflow completion confirmed
    
  Duration: ~5 minutes
  Status at end of day: All accounts DISABLED
```

**Day 1-7 (Week 1 - Immediate Post-Termination)**
```
Daily aggregations run:
  - Accounts show as disabled in ISC
  - No access possible
  - Audit trail building

Manager actions:
  - Retrieves important emails from forwarded mailbox
  - Documents ongoing work
  - Identifies knowledge gaps

IT actions:
  - Equipment collected (laptop, phone, badge)
  - Data backed up if needed
  - Access verified disabled across all systems
```

**Day 8-30 (Weeks 2-4 - Initial Retention)**
```
Weekly checks:
  ✓ Verify accounts still disabled
  ✓ No re-enablement requests
  ✓ No legal holds placed

Manager may request:
  - Access to specific files: Granted via backup/archive
  - Historical emails: Provided from mailbox
  - Salesforce data: Exported and provided
  
All requests documented in ticket TERM-54321
```

**Day 30 (End of Month 1)**
```
Automated 30-day review:
  
  Check:
    ☐ Legal holds? No
    ☐ Ongoing litigation? No
    ☐ Manager needs? Resolved
    ☐ IT needs? Complete
    
  Action:
    ✓ Email mailbox archived
      - Mailbox converted to inactive
      - No longer consuming license
      - Searchable for 7 years (compliance)
      
    ✓ Files archived
      - OneDrive/home folder backed up
      - Available if needed
      
    ✓ Accounts remain disabled
      - AD account: Still disabled
      - Salesforce: Still disabled
      - No changes yet
```

**Day 60 (Month 2)**
```
60-day review:
  
  Verify:
    ☐ No outstanding issues
    ☐ Manager hasn't requested access in 30 days
    ☐ No legal holds
    
  Prepare for deletion:
    ✓ Final verification with manager
    ✓ Review ticket for any holds
    ✓ Confirm ready for deletion
```

**Day 90 (Month 3 - Scheduled Deletion)**
```
Automated cleanup job runs (Sunday 3 AM):
  
  Job: "Delete accounts disabled >90 days"
  
  Finds:
    Account: sjohnson (AD)
    Disabled: February 13, 2024
    Days disabled: 90
    Legal hold: No
    
  Actions:
    1. Verify no holds (automated check)
       ✓ No legal holds
       ✓ No active tickets
       ✓ Retention period complete
       
    2. Delete AD account
       PowerShell: Remove-ADUser -Identity sjohnson
       Result: Account deleted from Active Directory
       
    3. Delete Salesforce account
       API call: DELETE /User/sjohnson
       Result: Account permanently removed
       
    4. Delete VPN account
       Result: Account removed
       
    5. Email mailbox
       Already archived (day 30)
       Archive retained for 7 years
       
    6. Update ISC
       Mark accounts as DELETED
       Preserve audit trail
       
    7. Close ticket
       TERM-54321 → Closed
       Resolution: "All accounts deleted per policy"
       
  Notifications:
    ✓ Email to audit team: Deletion completed
    ✓ Log entry created
    ✓ Compliance report updated
```

**Day 91+ (Post-Deletion)**
```
Next AD aggregation runs:
  1. ISC queries Active Directory
  2. Account sjohnson not found
  3. ISC confirms account deleted (as expected)
  4. No action needed
  
Identity in ISC:
  Name: Sarah Johnson
  Status: Inactive (preserved for audit)
  Accounts: None (all deleted)
  History: All account activity preserved
  Retention: 7 years for compliance
```

**Summary Timeline:**
```
Day 0: DISABLE (immediate)
  ✓ All accounts disabled
  ✓ Access terminated
  ✓ Security ensured

Day 30: ARCHIVE (data preservation)
  ✓ Email archived
  ✓ Files backed up
  ✓ Still disabled

Day 90: DELETE (permanent removal)
  ✓ Accounts hard deleted
  ✓ Source systems cleaned
  ✓ Audit trail preserved

Day 91+: CONFIRMED (verification)
  ✓ Aggregation confirms deletion
  ✓ No reappearance
  ✓ Process complete
```

**Audit Trail Preserved:**
```
ISC retains for 7 years:
  - Sarah Johnson identity record
  - All account history
  - All access history
  - Termination date and reason
  - All workflow actions
  - All deletion events
  - All approvals and tickets
  
Available for:
  - Compliance audits
  - Legal discovery
  - HR inquiries
  - Security investigations
```

**Exception Handling:**
```
If legal hold received on Day 45:
  ✓ Stop deletion process
  ✓ Preserve all data
  ✓ Keep accounts disabled
  ✓ Maintain indefinitely
  ✓ Delete only after hold lifted
  
If manager needs access on Day 75:
  ✓ Provide from archives/backups
  ✓ Do NOT re-enable accounts
  ✓ Document request
  ✓ Proceed with deletion on Day 90
```

**Exam Tip:** Standard process is disable immediately, retain 30-90 days, then hard delete. Always preserve audit trail.
</details>

---

# 3.3.4 Impact of Deletion on Aggregation

**Learning Objective:** Understand how account deletion affects aggregation and ISC processing.

---

## How Aggregation Detects Deletions

### **Delta Aggregation Limitations**

**The challenge:**
```
Delta aggregation query:
  "Give me accounts CHANGED since last aggregation"

Deleted accounts:
  - Don't exist anymore
  - Can't be "changed"
  - Won't appear in delta results

Problem: Delta aggregation may miss deletions
```

**Example:**
```
Last Aggregation: Monday 2 AM (10,000 accounts)

Tuesday at 10 AM:
  Admin deletes account "jsmith" in AD

Delta Aggregation: Tuesday 2 AM
  Query: Accounts changed since Monday 2 AM
  
  Changed accounts found: 50
    - 30 modified
    - 20 created
    - 0 deleted (jsmith not in results!)
    
  ISC doesn't know jsmith was deleted
  ISC still shows jsmith as active
```

### **Full Aggregation Detects Deletions**

**How it works:**
```
Full aggregation:
  1. Get ALL current accounts from source
  2. Compare with previous aggregation
  3. Find accounts that were there before, now missing
  4. Mark those as deleted

Example:
  Previous: 10,000 accounts (including jsmith)
  Current: 9,999 accounts (jsmith missing)
  
  ISC detects: jsmith was deleted
  Action: Mark jsmith as deleted in ISC
```

**Comparison:**
```
❌ Delta Aggregation:
  Query: Changed accounts
  Deleted accounts: NOT detected
  
✅ Full Aggregation:
  Query: ALL accounts
  Deleted accounts: Detected (missing from results)
```

**Exam Tip:** Full aggregation required to reliably detect deleted accounts. Delta misses deletions.

---

## Deletion Detection Methods

### **Method 1: Full Aggregation (Most Reliable)**

**Process:**
```
1. ISC retrieves ALL accounts from source
   Current count: 9,999

2. ISC compares with previous aggregation
   Previous count: 10,000
   
3. ISC identifies missing accounts
   Missing: jsmith, bwilson
   
4. ISC marks as deleted
   Accounts flagged as DELETED
   Removed from identity
   Logged in audit trail
```

**Schedule recommendation:**
```
Weekly full aggregation:
  - Runs Sunday night
  - Detects all deletions from past week
  - Acceptable delay for most organizations
```

### **Method 2: Tombstone Detection**

**What is tombstoning:**
```
Instead of deleting records, mark as deleted

Active Directory example:
  When account deleted:
    - Account moved to "Deleted Objects" container
    - Marked with "isDeleted" attribute
    - Remains for tombstone lifetime (default: 180 days)
```

**Delta aggregation can detect:**
```
Query: Accounts with isDeleted=true AND changed since last aggregation

Finds:
  - Deleted accounts (tombstoned)
  - Even in delta aggregation

Result: Delta can catch deletions (with tombstone support)
```

**Limitations:**
```
Not all sources support tombstoning:
  ✓ Active Directory: Yes (tombstone)
  ✓ Azure AD: Yes (soft delete)
  ❌ LDAP: Usually no
  ❌ Databases: Usually no (depends on schema)
  ❌ CSV files: No
```

### **Method 3: Event-Based Notification**

**How it works:**
```
Source notifies ISC of deletions:
  1. Account deleted in source
  2. Source sends event/notification to ISC
  3. ISC immediately processes deletion
  4. No waiting for aggregation

Example (Workday):
  Employee terminated in Workday
  → Workday sends termination event to ISC
  → ISC processes immediately
  → Accounts disabled in real-time
```

**Advantages:**
- Real-time or near real-time
- No waiting for aggregation
- Immediate security

**Limitations:**
- Only available for some SaaS sources
- Requires source to support events
- On-premises sources usually don't support

---

## Handling Deletion Lag

### **The Delay Problem**

**Scenario:**
```
Monday 10 AM: Account deleted in AD
Tuesday 2 AM: Delta aggregation (doesn't detect)
Wednesday 2 AM: Delta aggregation (doesn't detect)
Thursday 2 AM: Delta aggregation (doesn't detect)
Sunday 2 AM: Full aggregation (DETECTS deletion)

Delay: 6 days between deletion and detection
```

**Risks during delay:**
```
ISC still shows account as active:
  - Appears in reports (incorrect)
  - Shows in certifications (incorrect)
  - Identity shows account exists (incorrect)
  - Compliance reports wrong

User confusion:
  - "Why does it show I have this account?"
  - "The report says I still have access"
```

### **Mitigation Strategies**

**Strategy 1: More Frequent Full Aggregations**
```
Instead of: Weekly full
Use: Daily full (for critical sources)

Trade-off:
  ✓ Faster deletion detection (24 hours)
  ✗ Higher load on source and VA
  ✗ Longer aggregation time
  
When to use:
  - Small environments (<5,000 accounts)
  - Critical sources (HR, AD)
  - Security-sensitive systems
```

**Strategy 2: Bi-Weekly Full**
```
Schedule:
  Sunday: Full aggregation
  Wednesday: Full aggregation
  Mon/Tue/Thu/Fri/Sat: Delta aggregation

Result:
  - Max delay: 3-4 days
  - Balanced load
  - Acceptable for most organizations
```

**Strategy 3: Provisioning-Based Deletion**
```
Don't wait for aggregation:
  
  Leaver workflow:
    1. Detect termination in ISC
    2. Immediately send delete command to source
    3. Source deletes account
    4. ISC immediately updates (no wait)
    
  Result:
    - Real-time deletion in ISC
    - No delay
    - Accurate immediately
```

**Strategy 4: Tombstone Queries (If Supported)**
```
For AD and sources with tombstone support:
  
  Delta aggregation includes:
    - Changed accounts
    - Deleted accounts (tombstoned)
    
  Result:
    - Daily detection of deletions
    - Best of both worlds
```

---

## Impact on Identity Records

### **When Account Deleted**

**What happens to identity:**
```
Before deletion:
  Identity: John Smith
    Accounts:
      - Active Directory: jsmith
      - Email: john.smith@company.com
      - Salesforce: john.smith@company.com

After aggregation detects deletion:
  Identity: John Smith
    Accounts:
      - Email: john.smith@company.com
      - Salesforce: john.smith@company.com
    
    Deleted Accounts (history):
      - Active Directory: jsmith (deleted 2024-02-13)
```

**Identity not deleted:**
- Identity remains in ISC
- Other accounts still linked
- History preserved
- Can view past accounts

### **When All Accounts Deleted**

**Scenario:**
```
Identity with all accounts deleted:

Before:
  Identity: John Smith
    Accounts:
      - AD: jsmith
      - Email: john.smith@company.com

Both deleted:
  Identity: John Smith
    Accounts: None
    Status: Typically remains active (unless workflow changes it)
```

**Best practice:**
```
Lifecycle management:
  When all accounts deleted:
    1. Workflow detects no accounts remaining
    2. Sets lifecycle state to "inactive"
    3. Identity disabled but preserved
    4. Audit trail maintained
```

---

## Deletion and Certifications

### **Impact on Active Certifications**

**Scenario:**
```
Certification campaign running:
  "Review all VPN-Users group members"
  
  Campaign started: February 1
  Items: 500 user access items to review
  
  February 10: 10 users terminated, accounts deleted
  
  What happens to their certification items?
```

**Options:**

**Option 1: Items Remain in Campaign**
```
Deleted accounts still in review:
  - Reviewer sees terminated user
  - Can revoke (already gone)
  - Or approve (pointless)
  
Issue: Confusing to reviewers
```

**Option 2: Items Automatically Revoked**
```
ISC detects account deleted:
  - Automatically marks as "Revoked"
  - Removes from reviewer's queue
  - Logs automatic revocation
  
Better: Cleaner for reviewers
```

**Option 3: Items Removed**
```
ISC removes items from campaign:
  - No longer appears for review
  - Campaign count adjusted
  - Logged as "Not applicable - account deleted"
  
Best: Clean and accurate
```

**ISC typical behavior:**
```
Account deleted during certification:
  → Item automatically marked as "Revoked"
  → Removed from pending reviews
  → Logged in audit trail
  → Campaign statistics updated
```

---

## Deletion and Reporting

### **Impact on Reports**

**Access reports:**
```
Report: "All users with VPN access"

If based on:
  Current accounts: Deleted accounts not included ✓
  Historical data: May include deleted accounts ✗

Important: Specify report scope
  - Current access only
  - Or historical access
```

**Compliance reports:**
```
Report: "Access certifications - last 12 months"

Should include:
  ✓ Current accounts
  ✓ Deleted accounts (for complete picture)
  ✓ When accounts deleted
  ✓ Who approved deletion

Reason: Auditors need complete history
```

---

## Best Practices Summary

### **Deletion Detection**
```
✓ Run full aggregation at least weekly
  - Reliable deletion detection
  - Catches missed deletions
  
✓ Use tombstone queries if supported
  - Enables delta to detect deletions
  - Best for Active Directory
  
✓ Combine with provisioning-based deletion
  - Real-time deletion via workflows
  - Don't rely solely on aggregation
  
✓ Monitor deletion lag
  - Track time between deletion and detection
  - Alert if lag too long
```

### **Deletion Management**
```
✓ Soft delete (disable) before hard delete
  - Immediate: Disable
  - Retention: 30-90 days
  - Final: Hard delete
  
✓ Document all deletions
  - Who, what, when, why
  - Approval/ticket reference
  - Retention logs
  
✓ Preserve audit trail
  - Keep identity records
  - Keep deletion history
  - Comply with retention policies
  
✓ Handle special accounts appropriately
  - Service accounts: Different process
  - Shared accounts: Remove individual access only
  - Privileged accounts: Extra scrutiny
```

---

## Practice Exercise

**Question:** Your organization runs delta aggregation daily and full aggregation weekly (Sundays). An account is deleted in AD on Monday. When will ISC detect it? How could you improve this?

<details>
<summary>Answer</summary>

**Current Situation Analysis:**

**Schedule:**
```
Daily (Mon-Sat): Delta aggregation at 2 AM
Weekly (Sunday): Full aggregation at 2 AM
```

**Deletion Timeline:**
```
Monday 10 AM: Account deleted in Active Directory

Monday 2 AM (Already passed): Delta ran before deletion
Tuesday 2 AM: Delta aggregation runs
  Query: Accounts changed since Monday 2 AM
  Result: Doesn't find deleted account (not "changed")
  ISC status: Still shows account as active ✗

Wednesday-Saturday 2 AM: Delta aggregation
  Result: Still doesn't detect deletion
  ISC status: Still shows account as active ✗

Sunday 2 AM: Full aggregation runs
  Query: ALL accounts from AD
  Comparison: Account present last week, missing now
  Detection: ISC detects account deleted ✓
  ISC status: Marks account as deleted

Detection delay: 6 days (Monday to Sunday)
```

**Problems with 6-Day Delay:**
```
Issues:
  ✗ Reports show incorrect access for 6 days
  ✗ Certifications include deleted account
  ✗ Identity shows account exists (wrong)
  ✗ Compliance reports inaccurate
  ✗ User confusion ("Why does it show I have this?")
```

**Improvement Options:**

**Option 1: More Frequent Full Aggregations**
```
Change: Run full aggregation twice weekly

Schedule:
  Sunday 2 AM: Full aggregation
  Wednesday 2 AM: Full aggregation
  Mon/Tue/Thu/Fri/Sat: Delta aggregation

Result:
  Monday 10 AM: Account deleted
  Wednesday 2 AM: Full aggregation detects it
  Detection delay: 2 days maximum

Trade-offs:
  ✓ Faster deletion detection (2 days vs 6 days)
  ✗ More load on AD and VA (2x full aggregations)
  ✗ Longer aggregation time (full vs delta)

When to use:
  - Medium environments (<25K accounts)
  - Acceptable load increase
  - Need better accuracy
```

**Option 2: Daily Full Aggregations**
```
Change: Run full aggregation every day

Schedule:
  Daily: Full aggregation at 2 AM

Result:
  Monday 10 AM: Account deleted
  Tuesday 2 AM: Full aggregation detects it
  Detection delay: <24 hours

Trade-offs:
  ✓ Very fast detection (next day)
  ✗ Highest load on AD and VA
  ✗ No delta efficiency benefit
  ✗ Longer nightly aggregations

When to use:
  - Small environments (<5K accounts)
  - Critical source (must be accurate)
  - Can handle daily full aggregation load
```

**Option 3: Use Tombstone Detection (AD Only)**
```
Change: Configure delta to include deleted objects

Setup:
  Enable AD tombstone detection in connector
  
  Delta query includes:
    - Changed accounts (modified since last run)
    - Deleted objects (tombstoned since last run)

Result:
  Monday 10 AM: Account deleted (tombstoned in AD)
  Tuesday 2 AM: Delta aggregation
    Query includes: isDeleted=true AND changed since yesterday
    Finds: Deleted account in tombstone
    Detection: ISC detects deletion ✓
  Detection delay: <24 hours

Trade-offs:
  ✓ Fast detection with delta efficiency
  ✓ Lower load than daily full
  ✓ Best of both worlds
  ✗ Only works for AD (not all sources)
  ✗ Requires tombstone configuration

When to use:
  - Active Directory source
  - Want fast detection without full aggregation load
  - Recommended for most AD deployments
```

**Option 4: Provisioning-Based Deletion (Best)**
```
Change: Don't wait for aggregation

Implementation:
  Leaver Workflow:
    1. HR marks employee terminated
    2. ISC aggregates termination from HR
    3. Identity lifecycle → inactive
    4. Workflow immediately sends delete to AD
    5. AD deletes account
    6. ISC immediately marks deleted
    7. No waiting for aggregation

Result:
  Monday 10 AM: HR marks terminated
  Monday 12 PM: Workday aggregation (if scheduled)
  Monday 12:05 PM: Workflow triggers
    - Deletes AD account immediately
    - ISC updates immediately
  Detection delay: <1 hour (same day)

Trade-offs:
  ✓ Fastest detection (real-time)
  ✓ No aggregation wait
  ✓ Consistent process
  ✓ Works for all sources with provisioning
  ✗ Requires workflow configuration
  ✗ Only works for ISC-triggered deletions

When to use:
  - Should be standard for all environments
  - Handles planned terminations perfectly
  - Combine with aggregation for manual deletions
```

**Recommended Solution (Combination Approach):**
```
Best Practice: Use Multiple Methods

1. Provisioning-Based (Primary):
   - Leaver workflow deletes immediately
   - Handles 90% of deletions (planned terminations)
   - Detection: Real-time

2. Tombstone Detection (Secondary):
   - Delta includes deleted objects
   - Catches manual deletions in AD
   - Detection: Next day

3. Full Aggregation (Backup):
   - Weekly full on Sunday
   - Catches anything missed
   - Detection: Within 7 days

Result:
  Planned terminations: Real-time (workflow)
  Manual AD deletions: Next day (tombstone)
  Edge cases: Within 7 days (full)
  
Typical detection time: <24 hours for 99% of deletions
```

**Implementation Steps:**
```
Week 1: Enable Tombstone Detection
  1. Configure AD connector for tombstone
  2. Test delta aggregation
  3. Verify deleted accounts detected
  Result: Daily detection of manual deletions

Week 2: Implement Leaver Workflow
  1. Create/update leaver workflow
  2. Configure AD account deletion
  3. Test with test termination
  Result: Real-time deletion for terminations

Week 3: Optimize Full Aggregation
  1. Keep weekly full (Sunday)
  2. Review effectiveness
  3. Adjust if needed
  Result: Safety net in place

Ongoing: Monitor and Refine
  - Track deletion detection times
  - Measure improvement
  - Adjust as needed
```

**Expected Results:**
```
Before (Delta daily, Full weekly):
  Average detection time: 3-4 days
  Maximum detection time: 6 days
  
After (Tombstone + Workflow + Full):
  Average detection time: 2-4 hours
  Maximum detection time: 24 hours (manual deletions)
  
Improvement: 90% reduction in detection time
```

**Exam Tip:** Delta aggregation misses deletions. Use full aggregation weekly, tombstone detection if available, and provisioning-based deletion for planned terminations.
</details>

---

## Section Summary

**Core Concepts for Certification:**

✅ **Uncorrelated Accounts:**
- Account exists but not linked to identity
- Causes: Manual creation, user left, service accounts, correlation failures
- Must investigate and categorize appropriately
- Security and compliance risk if not managed

✅ **Correlation Rules:**
- Match accounts to identities using attributes
- Employee ID most reliable, email most common
- Use fallback methods for robustness
- Test before production deployment

✅ **Account Deletion:**
- **Soft delete**: Disable (reversible, preserves data)
- **Hard delete**: Remove (permanent, data lost)
- Best practice: Disable → Retain → Delete
- Typical retention: 30-90 days

✅ **Deletion Detection:**
- **Delta aggregation**: Doesn't detect deletions
- **Full aggregation**: Reliably detects deletions
- **Tombstone**: Allows delta to detect (AD only)
- **Provisioning**: Real-time deletion via workflows

✅ **Best Practices:**
- Use employee ID for correlation when available
- Disable accounts immediately on termination
- Retain disabled accounts per policy (30-90 days)
- Run full aggregation at least weekly
- Combine provisioning-based deletion with aggregation
- Preserve audit trails for compliance

**Exam Focus Areas:**
- Understand uncorrelated account causes and handling
- Know correlation methods (employee ID vs email)
- Understand soft delete vs hard delete
- Know that delta aggregation misses deletions
- Understand retention periods and compliance
- Know how to improve deletion detection

---

**End of Section 3.3: Account Correlation and Management**

**Next:** Section 3.4: Source Troubleshooting

---

**Study Tips:**
- Remember: Uncorrelated = investigate (service account, orphaned, or error)
- Employee ID > Email > Username for correlation
- Disable first, delete after retention period
- Delta misses deletions, full detects them
- Always preserve audit trail
