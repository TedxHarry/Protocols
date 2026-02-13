# 4.1.3 Priority and Precedence Rules

**Learning Objective:** Understand how ISC handles multiple identity profiles and determines which profile takes precedence.

---

## Why Priority Matters

### **The Problem**

**Scenario:**
```
Your organization has:
  - Employee data in Workday
  - Contractor data in Contractor Management System
  - Some people in BOTH systems (employee who also contracts)

Question: Which system is authoritative if person appears in both?
```

**Without priority:**
```
Chaos scenario:
  - John Smith in Workday: Department = IT
  - John Smith in Contractor System: Department = Marketing
  
Which department should ISC use?
  - Without priority rules → Unpredictable
  - Could flip-flop between values
  - Data inconsistency
```

**With priority:**
```
Controlled scenario:
  Priority 1: Employee Identity Profile (Workday)
  Priority 2: Contractor Identity Profile (Contractor System)
  
Result:
  - If John is in Workday → Use Workday data (higher priority)
  - Department = IT (from Workday)
  - Contractor data ignored for attributes
```

**Exam Tip:** Priority determines which identity profile wins when a person appears in multiple authoritative sources.

---

## How Priority Works

### **Priority Levels**

**Priority is a number:**
```
Lower number = Higher priority

Priority 1 = Highest priority (most authoritative)
Priority 2 = Second highest
Priority 3 = Third highest
... and so on
```

**Example configuration:**
```
Identity Profile: Employee Profile
  Source: Workday
  Priority: 1 (Highest)
  
Identity Profile: Contractor Profile
  Source: Contractor System
  Priority: 2 (Lower)
  
Identity Profile: Partner Profile
  Source: Partner System
  Priority: 3 (Lowest)
```

### **Processing Order**

**When aggregations run:**
```
1. All sources aggregate (in any order)
   - Workday aggregates
   - Contractor System aggregates
   - Partner System aggregates

2. Identity profiles process in priority order
   Priority 1 (Employee) processes first
   Priority 2 (Contractor) processes second
   Priority 3 (Partner) processes third

3. Each profile evaluates its data
   - Does this person match existing identity?
   - Create new or update existing?
   - Which attributes to set?
```

---

## Priority Scenarios

### **Scenario 1: Single Identity Profile**

**Configuration:**
```
Identity Profile: Employee Profile
  Source: Workday
  Priority: 1
  Population: All employees
```

**Result:**
```
Simple case:
  - Only one profile
  - Priority doesn't matter
  - All identities from Workday
  - No conflicts possible
```

### **Scenario 2: Multiple Profiles, No Overlap**

**Configuration:**
```
Identity Profile: Employee Profile
  Source: Workday
  Priority: 1
  Filter: employeeType = "Employee"
  
Identity Profile: Contractor Profile
  Source: Contractor System
  Priority: 2
  Filter: All records
```

**Result:**
```
No conflicts:
  - Employees only in Workday
  - Contractors only in Contractor System
  - No person in both
  - Priority doesn't affect anything
  - Each profile creates separate identities
```

### **Scenario 3: Multiple Profiles, WITH Overlap (Priority Matters!)**

**Configuration:**
```
Identity Profile: Employee Profile
  Source: Workday
  Priority: 1
  
Identity Profile: Contractor Profile
  Source: Contractor System
  Priority: 2
```

**Person appears in both systems:**
```
Workday has:
  employeeId: EMP12345
  name: John Smith
  department: IT
  email: john.smith@company.com
  
Contractor System has:
  contractorId: CTR99001
  name: John Smith
  department: Marketing
  email: john.smith@company.com
```

**How ISC processes with priority:**

**Step 1: Employee Profile (Priority 1) processes first**
```
1. Employee Profile processes Workday record
2. Finds no existing identity with employeeId = EMP12345
3. Creates new identity:
   Identity: John Smith
     employeeId: EMP12345
     department: IT
     email: john.smith@company.com
     source: Employee Profile (Priority 1)
```

**Step 2: Contractor Profile (Priority 2) processes second**
```
1. Contractor Profile processes contractor record
2. Tries to correlate: email = john.smith@company.com
3. Finds existing identity (created by Employee Profile)
4. Identity already exists from HIGHER priority profile
5. Decision: Does NOT override attributes
6. May link contractor account but doesn't change identity attributes
```

**Result:**
```
Final identity:
  Name: John Smith
  employeeId: EMP12345 (from Workday)
  department: IT (from Workday - higher priority)
  email: john.smith@company.com
  
  Managed by: Employee Profile (Priority 1)
  
  Contractor data not used for identity attributes
  (Contractor account may exist but linked to this identity)
```

**Exam Tip:** Higher priority profile (lower number) wins. Lower priority profiles cannot override attributes set by higher priority profiles.

---

## Attribute-Level Priority

### **How Attributes Are Managed**

**Rule:**
```
Once an attribute is set by a priority profile,
lower priority profiles cannot override it.
```

**Example:**
```
Priority 1 Profile sets:
  department = IT
  
Priority 2 Profile has:
  department = Marketing
  
Result:
  department remains = IT
  (Priority 1 wins, Priority 2 ignored for this attribute)
```

### **Exceptions: Unmapped Attributes**

**If higher priority doesn't set an attribute:**
```
Priority 1 Profile (Employee):
  Maps: firstName, lastName, email, department
  Does NOT map: location
  
Priority 2 Profile (Contractor):
  Maps: firstName, lastName, email, location
  
Result:
  firstName, lastName, email, department: From Priority 1
  location: From Priority 2 (Priority 1 didn't set it)
```

**Example:**
```
Employee Profile (Priority 1):
  firstName: John
  lastName: Smith
  email: john.smith@company.com
  department: IT
  location: (not mapped)
  
Contractor Profile (Priority 2):
  firstName: John
  lastName: Smith
  email: john.smith@company.com
  location: New York Office
  
Final Identity:
  firstName: John (from Priority 1)
  lastName: Smith (from Priority 1)
  email: john.smith@company.com (from Priority 1)
  department: IT (from Priority 1)
  location: New York Office (from Priority 2 - Priority 1 didn't set it)
```

---

## Lifecycle State and Priority

### **Lifecycle Precedence**

**Lifecycle follows same priority rules:**
```
Priority 1 Profile sets: lifecycle = active
Priority 2 Profile sets: lifecycle = inactive

Result: lifecycle = active (Priority 1 wins)
```

**Why this matters:**
```
Scenario: Employee in Workday terminated, but still active contractor

Workday (Priority 1):
  status: Terminated → lifecycle = inactive
  
Contractor System (Priority 2):
  status: Active → lifecycle = active
  
Result with Priority 1 winning:
  lifecycle = inactive (from Workday)
  
Impact:
  - Identity marked inactive (even though still contracting)
  - May trigger leaver workflows
  - May disable all access (including contractor access)
```

**Solution: Careful priority design**
```
Option 1: Make contractor data higher priority if they should remain active
  Not recommended - defeats purpose of employee data being authoritative
  
Option 2: Different lifecycle logic
  Use: If employee OR contractor active → lifecycle = active
  Custom rule combining both statuses
  
Option 3: Separate identities
  Don't correlate employee and contractor identities
  Two separate identities for same person
```

---

## Priority in Practice

### **Common Priority Patterns**

**Pattern 1: Employee First**
```
Priority 1: Employees (Workday)
  - Full-time employees
  - Most authoritative
  - Complete lifecycle management
  
Priority 2: Contractors
  - Contractors/temps
  - Less authoritative
  - Separate or linked identities

Why:
  - Employees are primary workforce
  - HR data most complete
  - If someone is both, employee data preferred
```

**Pattern 2: Regional Priority**
```
Priority 1: US Employees (Workday US)
  - US employment data
  
Priority 2: EU Employees (Workday EU)
  - EU employment data
  
Priority 3: APAC Employees (Workday APAC)
  - APAC employment data

Why:
  - Rarely overlaps
  - Priority handles edge cases (transfers between regions)
  - US data takes precedence if person in multiple systems during transfer
```

**Pattern 3: Type-Based Priority**
```
Priority 1: Full-Time Employees
Priority 2: Part-Time Employees  
Priority 3: Contractors
Priority 4: Partners

Why:
  - Clear hierarchy
  - Full-time most important
  - Priority resolves conflicts if someone moves between types
```

---

## Setting Priority in ISC

### **Configuration**

**Navigation:**
```
Admin → Identities → Identity Profiles
```

**View current priorities:**
```
Identity Profiles List:

Name                    Priority    Source          Identities
Employee Profile        1           Workday         10,500
Contractor Profile      2           Contractor Sys  1,200
Partner Profile         3           Partner Portal  350
```

**Change priority:**
```
1. Click on Identity Profile
2. Edit → Priority
3. Set priority number (1, 2, 3, etc.)
4. Save
5. Refresh identities to apply new priority
```

**Important:**
```
When changing priority:
  ✓ Test in sandbox first
  ✓ Understand impact on existing identities
  ✓ May need to refresh all identities
  ✓ Could change attribute values if priorities flip
```

---

## Priority and Identity Refresh

### **What Happens During Refresh**

**Process:**
```
1. Source aggregations run
   All sources aggregate independently
   
2. Identity profiles process in priority order
   Priority 1 first, then 2, then 3, etc.
   
3. For each identity profile:
   a. Process each record from authoritative source
   b. Try to correlate to existing identity
   c. If exists and managed by higher priority: Skip attribute updates
   d. If exists and managed by SAME priority: Update attributes
   e. If exists and managed by LOWER priority: Take over (update attributes)
   f. If new: Create identity
```

**Example refresh:**
```
Identity: John Smith (currently managed by Priority 1)

Priority 1 Profile processes:
  - Finds John in Workday
  - Updates attributes (same priority, can update)
  - department: IT → Engineering (changed in Workday)
  - Result: department = Engineering

Priority 2 Profile processes:
  - Finds John in Contractor System
  - Identity managed by Priority 1 (higher)
  - Does NOT update attributes
  - Result: department stays = Engineering
```

---

## Troubleshooting Priority Issues

### **Issue 1: Wrong Profile Managing Identity**

**Symptom:**
```
Identity managed by wrong profile
Expected: Employee Profile (Priority 1)
Actual: Contractor Profile (Priority 2)
```

**Causes:**
```
1. Person not in higher priority source
   - Not in Workday yet
   - Contractor profile created identity first
   
2. Correlation not working
   - Higher priority can't correlate to existing identity
   - Creates duplicate instead
   
3. Higher priority profile disabled
   - Only lower priority active
```

**Solution:**
```
1. Verify person in higher priority source (Workday)

2. Check correlation configuration
   - Does email/employeeId match?
   - Fix correlation rules
   
3. If needed, manually reassign:
   - Delete identity created by Priority 2
   - Trigger Priority 1 aggregation
   - Identity recreated by Priority 1
```

### **Issue 2: Attributes Not Updating**

**Symptom:**
```
Department changed in Workday
Identity department not updating
```

**Causes:**
```
1. Identity managed by different profile
   Check which profile manages this identity
   
2. Aggregation not running
   Source not aggregating
   
3. Attribute mapping incorrect
   Profile not mapping department attribute
```

**Solution:**
```
1. Check managing profile:
   View identity → See which profile manages it
   
2. Trigger aggregation manually
   Verify source aggregates successfully
   
3. Check attribute mapping
   Verify department mapped in profile
```

### **Issue 3: Lifecycle State Wrong**

**Symptom:**
```
Employee terminated in Workday
Identity still active
```

**Causes:**
```
1. Lower priority profile overriding
   Check if another profile setting lifecycle = active
   
2. Lifecycle mapping incorrect
   Profile not mapping terminated status to inactive
   
3. Aggregation lag
   Workday not yet aggregated
```

**Solution:**
```
1. Review priority order
   Ensure terminated employee's profile is highest priority
   
2. Check lifecycle configuration
   Verify status → lifecycle mapping
   
3. Trigger aggregation
   Manual Workday aggregation to force update
```

---

## Best Practices

### **Priority Design**
```
✓ Clear hierarchy
  - Most authoritative source = Priority 1
  - Less authoritative sources = lower priorities
  - Document reasoning

✓ Minimize overlap
  - Design profiles to minimize people in multiple sources
  - Use filters to separate populations
  - Overlaps should be intentional, not accidental

✓ Test priority changes thoroughly
  - Test in sandbox before production
  - Understand impact on existing identities
  - Have rollback plan

✓ Document priority decisions
  - Why this priority order?
  - What happens with overlaps?
  - Edge cases considered?
```

### **Monitoring**
```
✓ Regular reviews
  - Check which profiles manage identities
  - Verify priority still appropriate
  - Look for unexpected profile assignments

✓ Alert on conflicts
  - Monitor for same person in multiple sources
  - Investigate unexpected overlaps
  - Verify priority handling correctly

✓ Audit changes
  - Track when identities change managing profile
  - Investigate why priority changed
  - Ensure intentional, not error
```

---

## Practice Exercise

**Question:** You have Employee Profile (Priority 1, Workday) and Contractor Profile (Priority 2, Contractor System). John Smith appears in both. Workday says department=IT, Contractor System says department=Marketing. What will John's identity show for department and why?

<details>
<summary>Answer</summary>

**Answer: department = IT (from Workday)**

**Detailed Explanation:**

**Configuration:**
```
Identity Profile: Employee Profile
  Source: Workday
  Priority: 1 (Highest)
  Attribute Mapping: department ← workday.department
  
Identity Profile: Contractor Profile
  Source: Contractor System
  Priority: 2 (Lower)
  Attribute Mapping: department ← contractor.department
```

**Data in Sources:**
```
Workday:
  name: John Smith
  email: john.smith@company.com
  department: IT
  
Contractor System:
  name: John Smith
  email: john.smith@company.com
  department: Marketing
```

**Processing Steps:**

**Step 1: Aggregation**
```
Both sources aggregate:
  - Workday aggregates (John Smith in IT)
  - Contractor System aggregates (John Smith in Marketing)
  
Order doesn't matter - both aggregate independently
```

**Step 2: Identity Profile Processing (Priority Order)**

**Employee Profile (Priority 1) processes first:**
```
1. Employee Profile receives John Smith record from Workday
   
2. Tries to correlate:
   - Searches for existing identity with email = john.smith@company.com
   - No existing identity found (first time processing)
   
3. Creates new identity:
   Identity: John Smith
     email: john.smith@company.com
     department: IT (from Workday)
     managedBy: Employee Profile
     priority: 1
```

**Contractor Profile (Priority 2) processes second:**
```
1. Contractor Profile receives John Smith record from Contractor System
   
2. Tries to correlate:
   - Searches for existing identity with email = john.smith@company.com
   - FOUND: Identity already exists (created by Employee Profile)
   
3. Checks managing profile:
   - Existing identity managed by Employee Profile (Priority 1)
   - Current profile is Contractor Profile (Priority 2)
   - Priority 2 < Priority 1 (lower priority)
   
4. Decision:
   - Cannot override attributes set by higher priority profile
   - department attribute already set to "IT" by Priority 1
   - Priority 2 cannot change it to "Marketing"
   
5. Result:
   - Identity NOT updated
   - department remains "IT"
   - Contractor Profile may link contractor account but doesn't change identity attributes
```

**Final Identity:**
```
Identity: John Smith
  email: john.smith@company.com
  department: IT ← From Workday (Priority 1)
  managedBy: Employee Profile (Priority 1)
  
Attributes:
  - All attributes from Employee Profile (Priority 1)
  - Contractor Profile (Priority 2) data ignored for identity attributes
```

**Why IT and Not Marketing:**
```
Reason 1: Priority order
  Employee Profile (1) > Contractor Profile (2)
  Lower number = higher priority = wins conflicts
  
Reason 2: First to set attribute wins
  Employee Profile set department = IT first
  Contractor Profile tried to set department = Marketing later
  Higher priority profile's value takes precedence
  
Reason 3: Protection mechanism
  Prevents lower priority sources from overriding authoritative data
  Ensures data integrity
  HR system (Workday) is more authoritative than contractor system
```

**What About Contractor Account:**
```
Identity: John Smith
  Managed by: Employee Profile
  
Linked Accounts:
  - Workday: john.smith (employee account)
  - Contractor System: john.smith (contractor account - may be linked)
  
The contractor account may still be linked to this identity, but:
  - It doesn't control identity attributes
  - It's just an account associated with the identity
  - Identity attributes come from Employee Profile (Workday)
```

**If Priorities Were Reversed:**
```
Hypothetical: Contractor Profile = Priority 1, Employee Profile = Priority 2

Then:
  department = Marketing (from Contractor System)
  managedBy: Contractor Profile
  
Workday data would be ignored (lower priority)

But this is NOT recommended:
  - HR data (Workday) should be authoritative
  - Employee status more important than contractor status
  - Standard practice: HR = highest priority
```

**Edge Case: Workday Doesn't Have Department:**
```
If Workday record had:
  department: (null or not mapped)
  
And Contractor System had:
  department: Marketing
  
Then:
  department = Marketing (from Contractor Profile)
  
Why:
  - Employee Profile didn't set department (null/unmapped)
  - Contractor Profile can populate unmapped attributes
  - Only mapped attributes protected by priority
```

**Summary:**
```
John Smith's department = IT because:
  ✓ Employee Profile (Priority 1) processed first
  ✓ Set department = IT from Workday
  ✓ Contractor Profile (Priority 2) processed second
  ✓ Cannot override Priority 1 attributes
  ✓ department remains IT
  
Priority rule: Higher priority (lower number) wins attribute conflicts
```

**Exam Tip:** Priority determines attribute values when same person appears in multiple sources. Priority 1 always wins over Priority 2, 3, etc.
</details>

---

## Section Summary

**Core Concepts for Certification:**

✅ **Identity Profiles:**
- Configuration that creates identities from authoritative sources
- Defines attribute mappings, lifecycle states, correlation
- Bridge between source data and ISC identities
- Each profile has one authoritative source

✅ **Profile Configuration:**
- **Static mapping**: Direct copy (firstName ← source.firstName)
- **Complex mapping**: Transform (email ← firstName + "." + lastName + "@domain.com")
- **Rule mapping**: Conditional logic (if/then)
- **Lifecycle mapping**: Source status → ISC lifecycle state

✅ **Priority Rules:**
- Lower number = Higher priority (Priority 1 > Priority 2)
- Higher priority profile wins attribute conflicts
- Lower priority cannot override higher priority attributes
- Priority determines which profile manages the identity

✅ **Best Practices:**
- Use HR system as Priority 1 (most authoritative)
- Map unique identifier (employeeId) for correlation
- Test transforms before production
- Configure lifecycle states for automation
- Document priority decisions and reasoning

✅ **Common Patterns:**
- Single profile: Simple, one employee type
- Multiple profiles: Employees vs contractors
- Priority hierarchy: Employee > Contractor > Partner
- Regional profiles: Different regions, same priority level

**Exam Focus Areas:**
- Understand identity profile purpose and components
- Know mapping types (static, complex, rule)
- Understand priority precedence rules
- Know that Priority 1 > Priority 2 > Priority 3
- Understand lifecycle state configuration
- Know when to use multiple profiles

---

**End of Section 4.1: Identity Profiles**

**Next:** Section 4.2: Identity Attributes

---

**Study Tips:**
- Remember: Identity Profile creates identities, Source provides data
- Priority 1 = highest priority (not Priority 10)
- Higher priority can't be overridden by lower priority
- Always map employeeId or unique identifier
- Test transforms with sample data before production
