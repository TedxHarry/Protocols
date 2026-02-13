# Section 4.1: Identity Profiles

**Master identity profiles, their purpose, configuration, and priority rules in ISC**

---

## Table of Contents

- [4.1.1 Purpose and Use of Identity Profiles](#411-purpose-and-use-of-identity-profiles)
- [4.1.2 Creating and Configuring Identity Profiles](#412-creating-and-configuring-identity-profiles)
- [4.1.3 Priority and Precedence Rules](#413-priority-and-precedence-rules)

---

# 4.1.1 Purpose and Use of Identity Profiles

**Learning Objective:** Understand what identity profiles are and why they're essential in ISC.

---

## What is an Identity Profile?

**Definition:** A configuration that defines how ISC creates and manages identities from authoritative sources

**Think of it as:**
- A template for creating identities
- Instructions for ISC: "How to build an identity from this source"
- The bridge between source data and ISC identities

**Simple concept:**
```
Source Data (Workday) → Identity Profile → Identity in ISC
```

**Example:**
```
Workday has employee data:
  First Name: John
  Last Name: Smith
  Email: john.smith@company.com
  Department: IT
  
Identity Profile says:
  "Take Workday data and create ISC identity with these attributes"
  
Result - Identity in ISC:
  Name: John Smith
  Email: john.smith@company.com
  Department: IT
  Status: Active
```

**Exam Tip:** Identity Profile = configuration that creates identities from authoritative source data.

---

## Why Identity Profiles Are Needed

### **The Core Problem**

**ISC needs to know:**
```
Questions:
  - Which source is authoritative for identity data?
  - What attributes create the identity?
  - How to map source attributes to ISC attributes?
  - When to create, update, or disable identities?
  - What lifecycle states to use?
```

**Identity Profiles provide answers**

### **What Identity Profiles Do**

**1. Define Authoritative Source**
```
Identity Profile specifies:
  "Workday is the authoritative source for employees"
  
Result:
  - ISC trusts Workday for identity data
  - Other sources provide accounts, not identities
  - Workday data overrides others
```

**2. Map Source Attributes to ISC**
```
Identity Profile maps:
  Workday.firstName → identity.firstName
  Workday.lastName → identity.lastName
  Workday.workEmail → identity.email
  Workday.department → identity.department
  
Result:
  - ISC knows which source fields to use
  - Creates consistent identity structure
```

**3. Define Lifecycle Management**
```
Identity Profile determines:
  When Workday.status = "Active" → Identity lifecycle = active
  When Workday.status = "Terminated" → Identity lifecycle = inactive
  
Result:
  - Automatic lifecycle state management
  - Triggers workflows based on state changes
```

**4. Set Identity Attributes**
```
Identity Profile can:
  - Copy attributes directly (firstName = source.firstName)
  - Transform attributes (email = LOWER(firstName + "." + lastName + "@company.com"))
  - Set static values (type = "Employee")
  - Use rules for complex logic
  
Result:
  - Flexible identity attribute population
  - Consistent data format
```

---

## Identity Profile Architecture

### **How It Works**
```
┌─────────────────────────────────┐
│   Authoritative Source          │
│   (Workday, HR Database)        │
│                                 │
│   Employee records              │
└─────────────┬───────────────────┘
              │
              │ Aggregation
              ↓
┌─────────────────────────────────┐
│   Identity Profile              │
│                                 │
│   Configuration:                │
│   - Source: Workday             │
│   - Attribute mappings          │
│   - Lifecycle rules             │
│   - Transforms                  │
└─────────────┬───────────────────┘
              │
              │ Creates/Updates
              ↓
┌─────────────────────────────────┐
│   ISC Identities                │
│                                 │
│   - John Smith                  │
│   - Jane Doe                    │
│   - Bob Wilson                  │
└─────────────────────────────────┘
```

### **Processing Flow**
```
1. Source Aggregation
   Workday aggregates → ISC receives employee data
   
2. Identity Profile Evaluation
   For each employee record:
     - Check if identity exists (by unique ID)
     - If new: Create identity
     - If exists: Update identity
     
3. Attribute Mapping
   Apply mappings:
     firstName = Workday.firstName
     lastName = Workday.lastName
     email = Workday.email
     
4. Lifecycle Determination
   Check Workday.status:
     If "Active" → lifecycle = active
     If "Terminated" → lifecycle = inactive
     
5. Identity Created/Updated
   Identity in ISC with all attributes set
   
6. Workflows Triggered (if configured)
   If lifecycle changed → trigger joiner/leaver workflows
```

---

## Identity Profile vs Source

### **Key Differences**

| Aspect | Source | Identity Profile |
|--------|--------|------------------|
| **Purpose** | Provides data (accounts/identities) | Defines how to create identities |
| **What it is** | Connection to system | Configuration/template |
| **Examples** | Workday, Active Directory, Salesforce | Employee Profile, Contractor Profile |
| **Creates** | Account records | Identity records |
| **Authoritative** | Can be authoritative or not | Always uses authoritative source |

**Example:**
```
Source: Workday
  - Connection to Workday
  - Aggregates employee data
  - Can be used by identity profile
  
Identity Profile: Employee Identity Profile
  - Uses Workday source
  - Maps Workday data to identities
  - Creates ISC identities for employees
```

---

## Common Identity Profile Scenarios

### **Scenario 1: Single Identity Profile (Simple)**

**Configuration:**
```
Organization: Small company, one employee type

Identity Profile: "Employees"
  Source: Workday
  Population: All Workday employees
  
Result:
  - All employees get same identity profile
  - Simple, straightforward
```

### **Scenario 2: Multiple Identity Profiles (Common)**

**Configuration:**
```
Organization: Company with employees and contractors

Identity Profile 1: "Employees"
  Source: Workday
  Filter: employeeType = "Employee"
  Lifecycle: Full employee lifecycle
  
Identity Profile 2: "Contractors"
  Source: Workday (or separate contractor system)
  Filter: employeeType = "Contractor"
  Lifecycle: Simplified contractor lifecycle
  
Result:
  - Different identity profiles for different populations
  - Different lifecycle management
  - Different attribute mappings if needed
```

### **Scenario 3: Regional Identity Profiles**

**Configuration:**
```
Organization: Global company, different regions

Identity Profile 1: "US Employees"
  Source: Workday
  Filter: country = "US"
  Attributes: Include SSN
  
Identity Profile 2: "EU Employees"
  Source: Workday
  Filter: country in ["UK", "France", "Germany"]
  Attributes: GDPR-compliant attributes
  
Result:
  - Regional compliance requirements
  - Different attribute sets
  - Localized lifecycle rules
```

---

## Identity Attributes

### **Core Identity Attributes**

**Standard ISC attributes:**
```
Required:
  - uid (unique identifier)
  - email
  - firstName
  - lastName
  
Common:
  - displayName
  - department
  - manager
  - employeeId
  - startDate
  - endDate
  - location
  - phone
```

**Where they come from:**
```
Identity Profile defines for each attribute:

Option 1: Direct mapping
  identity.firstName = source.firstName
  
Option 2: Transform
  identity.email = LOWER(source.firstName + "." + source.lastName + "@company.com")
  
Option 3: Static value
  identity.type = "Employee"
  
Option 4: Rule-based
  Complex logic using rules
```

---

## Lifecycle States

### **What are Lifecycle States?**

**Definition:** The current status of an identity (active, inactive, etc.)

**Common lifecycle states:**
```
active: Active employee, full access
inactive: Terminated, no access
prehire: Hired but not started yet
leave: On leave (maternity, medical, sabbatical)
```

**Identity Profile determines lifecycle:**
```
Example mapping:
  When Workday.employeeStatus = "Active" → lifecycle = active
  When Workday.employeeStatus = "Terminated" → lifecycle = inactive
  When Workday.employeeStatus = "Leave" → lifecycle = leave
```

**Lifecycle drives automation:**
```
Lifecycle change: active → inactive
  
Triggers:
  - Leaver workflow
  - Disable all accounts
  - Revoke all access
  - Notify manager
```

**Exam Tip:** Identity Profile sets lifecycle state based on authoritative source data. Lifecycle changes trigger workflows.

---

## Identity Profile Use Cases

### **Use Case 1: New Hire Onboarding**

**Process:**
```
Day 1: HR creates employee in Workday
  First Name: Sarah
  Last Name: Johnson
  Email: sarah.johnson@company.com
  Department: Marketing
  Status: Active
  Start Date: 2024-03-01
  
Day 2: Workday aggregation runs
  ISC receives new employee record
  
Identity Profile processes:
  1. Detects new employee (no existing identity)
  2. Maps attributes from Workday
  3. Sets lifecycle = active (status is Active)
  4. Creates identity in ISC
  
Result - New identity created:
  Name: Sarah Johnson
  Email: sarah.johnson@company.com
  Department: Marketing
  Lifecycle: active
  
Triggers:
  Joiner workflow → Provisions accounts (AD, email, apps)
```

### **Use Case 2: Employee Transfer**

**Process:**
```
Event: Sarah transfers IT → Marketing

Workday updated:
  Department: IT → Marketing
  Manager: John Smith → Jane Doe
  
Workday aggregation runs:
  ISC receives updated employee record
  
Identity Profile processes:
  1. Finds existing identity (Sarah Johnson)
  2. Updates attributes from Workday
     department: IT → Marketing
     manager: John Smith → Jane Doe
  3. Lifecycle remains: active
  
Result - Identity updated:
  Department changed to Marketing
  Manager changed to Jane Doe
  
Triggers:
  Mover workflow → Revoke IT access, grant Marketing access
```

### **Use Case 3: Termination**

**Process:**
```
Event: Sarah terminated

Workday updated:
  Status: Active → Terminated
  End Date: 2024-02-13
  
Workday aggregation runs:
  ISC receives updated employee record
  
Identity Profile processes:
  1. Finds existing identity (Sarah Johnson)
  2. Detects lifecycle change (status changed)
  3. Sets lifecycle: active → inactive
  
Result - Identity updated:
  Lifecycle state = inactive
  End date = 2024-02-13
  
Triggers:
  Leaver workflow → Disable all accounts, revoke access
```

---

## Practice Exercise

**Question:** Your company has employees in Workday and contractors in a separate contractor management system. How would you structure identity profiles?

<details>
<summary>Answer</summary>

**Recommended Structure: Two Identity Profiles**

**Identity Profile 1: Employees**
```
Configuration:
  Name: "Employee Identity Profile"
  Authoritative Source: Workday
  
  Population/Filter:
    Use all records from Workday
    (Workday only contains employees)
  
  Attribute Mappings:
    firstName: workday.firstName
    lastName: workday.lastName
    email: workday.workEmail
    employeeId: workday.employeeNumber
    department: workday.department
    manager: workday.managerName
    startDate: workday.hireDate
    endDate: workday.terminationDate
    location: workday.location
    type: Static value "Employee"
  
  Lifecycle State Mapping:
    When workday.employeeStatus = "Active" → lifecycle = active
    When workday.employeeStatus = "Leave" → lifecycle = leave
    When workday.employeeStatus = "Terminated" → lifecycle = inactive
  
  Correlation:
    employeeId (unique identifier from Workday)
```

**Identity Profile 2: Contractors**
```
Configuration:
  Name: "Contractor Identity Profile"
  Authoritative Source: Contractor Management System
  
  Population/Filter:
    Use all records from contractor system
  
  Attribute Mappings:
    firstName: contractor.firstName
    lastName: contractor.lastName
    email: contractor.email
    employeeId: contractor.contractorId
    department: contractor.department
    manager: contractor.projectManager
    startDate: contractor.contractStartDate
    endDate: contractor.contractEndDate
    location: contractor.workLocation
    type: Static value "Contractor"
  
  Lifecycle State Mapping:
    When contractor.status = "Active" AND today < contractEndDate → lifecycle = active
    When contractor.status = "Active" AND today >= contractEndDate → lifecycle = inactive
    When contractor.status = "Terminated" → lifecycle = inactive
  
  Correlation:
    employeeId (contractorId from contractor system)
```

**Why Two Separate Identity Profiles?**
```
Reason 1: Different authoritative sources
  - Employees: Workday
  - Contractors: Contractor system
  - Can't use one profile for two sources

Reason 2: Different attribute mappings
  - Employees: Use employeeNumber
  - Contractors: Use contractorId
  - Different field names in sources

Reason 3: Different lifecycle logic
  - Employees: Based on employment status
  - Contractors: Based on contract dates
  - Different business rules

Reason 4: Different identity attributes
  - Employees: May have additional fields (benefits, etc.)
  - Contractors: Contract-specific fields (end date, agency)
  - Type field distinguishes them

Reason 5: Different workflows
  - Employees: Full joiner/mover/leaver workflows
  - Contractors: Simplified workflows, expiration handling
  - Different provisioning needs
```

**Implementation:**
```
Step 1: Create Employee Identity Profile
  1. Admin → Identities → Identity Profiles → Create
  2. Name: Employee Identity Profile
  3. Authoritative Source: Workday
  4. Configure attribute mappings
  5. Configure lifecycle states
  6. Save

Step 2: Create Contractor Identity Profile
  1. Admin → Identities → Identity Profiles → Create
  2. Name: Contractor Identity Profile
  3. Authoritative Source: Contractor Management System
  4. Configure attribute mappings
  5. Configure lifecycle states
  6. Save

Step 3: Test Each Profile
  - Trigger Workday aggregation → Verify employee identities created
  - Trigger contractor system aggregation → Verify contractor identities created
  - Check that identities have correct "type" attribute

Step 4: Configure Workflows
  - Employee joiner workflow: Triggered when Employee profile creates identity
  - Contractor joiner workflow: Triggered when Contractor profile creates identity
  - Different provisioning for each type
```

**Result:**
```
Identities in ISC:

Employee Identities:
  John Smith
    type: Employee
    source: Workday
    employeeId: EMP12345
    lifecycle: active
    
  Jane Doe
    type: Employee
    source: Workday
    employeeId: EMP12346
    lifecycle: active

Contractor Identities:
  Bob Wilson
    type: Contractor
    source: Contractor System
    employeeId: CTR99001
    lifecycle: active
    endDate: 2024-06-30
    
  Alice Brown
    type: Contractor
    source: Contractor System
    employeeId: CTR99002
    lifecycle: active
    endDate: 2024-12-31
```

**Benefits:**
```
✓ Clear separation of employee vs contractor identities
✓ Different lifecycle management rules
✓ Different workflows based on type
✓ Easy reporting (filter by type)
✓ Compliance (different handling for each type)
✓ Scalable (can add more profiles for other populations)
```

**Alternative Approach (Not Recommended):**
```
❌ Single Identity Profile with complex logic:
  - Would need complex transforms to handle two sources
  - Difficult to maintain
  - Prone to errors
  - Less clear what applies to whom

✓ Two separate profiles is cleaner and more maintainable
```

**Exam Tip:** Create separate identity profiles for different populations with different authoritative sources or different business logic.
</details>

---

# 4.1.2 Creating and Configuring Identity Profiles

**Learning Objective:** Learn how to create and configure identity profiles in ISC.

---

## Creating an Identity Profile

### **Basic Setup**

**Navigation:**
```
Admin → Identities → Identity Profiles → Create Identity Profile
```

**Required Information:**

**1. Basic Information**
```
Name: Employee Identity Profile
Description: Identity profile for all employees from Workday
Owner: IAM Team
```

**2. Authoritative Source**
```
Select source: Workday

This determines:
  - Where identity data comes from
  - Which source is "source of truth"
  - What data is available for mapping
```

**3. Identity Attribute Mappings**
```
Configure how source attributes map to identity attributes

Examples:
  firstName ← workday.firstName
  lastName ← workday.lastName
  email ← workday.workEmail
  employeeId ← workday.employeeNumber
```

---

## Attribute Mapping Configuration

### **Mapping Types**

**Type 1: Static Mapping (Direct Copy)**

**Configuration:**
```
Identity Attribute: firstName
Mapping Type: Static
Value: source.firstName

Result:
  identity.firstName = workday.firstName
  
Example:
  Workday: firstName = "John"
  Identity: firstName = "John"
```

**Type 2: Complex Mapping (Transform)**

**Configuration:**
```
Identity Attribute: email
Mapping Type: Complex
Transform: 
  LOWER(firstName + "." + lastName + "@company.com")

Result:
  identity.email = LOWER(workday.firstName + "." + workday.lastName + "@company.com")
  
Example:
  Workday: firstName = "John", lastName = "Smith"
  Identity: email = "john.smith@company.com"
```

**Type 3: Rule-Based Mapping**

**Configuration:**
```
Identity Attribute: department
Mapping Type: Rule
Rule: DepartmentMappingRule

Rule logic:
  If source.deptCode = "IT" → "Information Technology"
  If source.deptCode = "HR" → "Human Resources"
  If source.deptCode = "FIN" → "Finance"
  Else → source.department

Result:
  Maps department codes to full names
```

**Exam Tip:** Static = direct copy, Complex = transform, Rule = conditional logic.

---

## Common Attribute Mappings

### **Standard Mappings**

**Core attributes:**
```
firstName:
  Mapping: Static
  Value: source.firstName

lastName:
  Mapping: Static
  Value: source.lastName

displayName:
  Mapping: Complex
  Transform: firstName + " " + lastName

email:
  Mapping: Static (if available)
  Value: source.workEmail
  OR Complex: firstName + "." + lastName + "@company.com"

employeeId:
  Mapping: Static
  Value: source.employeeNumber
  (Used for correlation, must be unique)
```

**Organizational attributes:**
```
department:
  Mapping: Static or Rule
  Value: source.department
  (May use rule for code-to-name translation)

manager:
  Mapping: Static or Complex
  Value: source.managerEmployeeId
  (ISC can lookup manager identity)

location:
  Mapping: Static
  Value: source.location

costCenter:
  Mapping: Static
  Value: source.costCenter
```

**Date attributes:**
```
startDate:
  Mapping: Static
  Value: source.hireDate

endDate:
  Mapping: Static
  Value: source.terminationDate
  (Empty if still employed)
```

---

## Lifecycle State Configuration

### **Defining Lifecycle States**

**Configuration:**
```
Lifecycle State Attribute: source.employeeStatus

Mappings:
  When source.employeeStatus = "Active" → lifecycle = active
  When source.employeeStatus = "Leave" → lifecycle = leave
  When source.employeeStatus = "Terminated" → lifecycle = inactive
  When source.employeeStatus = "Pre-Hire" → lifecycle = prehire
```

**Using Complex Logic:**
```
Lifecycle can use transforms:

Example - Date-based:
  If source.startDate > TODAY → lifecycle = prehire
  Else if source.endDate != null AND source.endDate < TODAY → lifecycle = inactive
  Else if source.employeeStatus = "Active" → lifecycle = active
  Else → lifecycle = inactive
```

### **Lifecycle Provisioning Policies**

**Configuration options:**
```
For each lifecycle state, configure:

State: active
  Provisioning: Enabled
  Allow requests: Yes
  Deprovisioning: On lifecycle change to inactive
  
State: inactive
  Provisioning: Disabled
  Allow requests: No
  Deprovisioning: Immediate
  
State: leave
  Provisioning: Partially enabled
  Allow requests: Limited
  Deprovisioning: On return or termination
```

**Example:**
```
Lifecycle State: active
  ✓ User can request access
  ✓ Automatic provisioning enabled
  ✓ Can use applications
  
Lifecycle State: inactive
  ✗ User cannot request access
  ✗ Provisioning disabled
  ✗ All access revoked
```

---

## Identity Correlation

### **What is Identity Correlation in Profiles?**

**Purpose:** Link accounts from non-authoritative sources to identities created by the profile

**How it works:**
```
Identity Profile creates identity from Workday:
  Identity: John Smith
    employeeId: EMP12345
    email: john.smith@company.com
  
Account exists in Active Directory:
  Account: jsmith
    mail: john.smith@company.com
  
Correlation:
  Match identity.email to account.mail
  Link account to identity
  
Result:
  Identity: John Smith
    Accounts:
      - Active Directory: jsmith (correlated)
```

**Configuration:**
```
Identity Profile → Correlation

Correlation Attribute: employeeId
  Match account.employeeId to identity.employeeId
  
Fallback: email
  If employeeId doesn't match, try email
```

---

## Identity Refresh Schedule

### **When Identities Update**

**Automatic refresh:**
```
Trigger: Authoritative source aggregation

Process:
  1. Workday aggregation runs (e.g., daily 1 AM)
  2. ISC receives employee data
  3. Identity profile processes each record
  4. Identities created/updated based on mappings
  5. Lifecycle states evaluated
  6. Workflows triggered if lifecycle changed
```

**Manual refresh:**
```
Admin can trigger:
  1. Admin → Identities → Identity Profiles
  2. Select profile
  3. Click "Refresh Identities"
  4. All identities in this profile refreshed
```

---

## Identity Profile Settings

### **Additional Configuration Options**

**Identity Attribute Configuration:**
```
For each attribute:
  - Searchable: Include in search index
  - Multi-valued: Can have multiple values
  - Required: Must have a value
  - Sensitive: Mark as confidential
```

**Example:**
```
Attribute: email
  ✓ Searchable (users can search by email)
  ✗ Multi-valued (only one email)
  ✓ Required (all identities must have email)
  ✗ Sensitive (not confidential)

Attribute: SSN
  ✗ Searchable (don't index for security)
  ✗ Multi-valued
  ✗ Required (contractors may not have)
  ✓ Sensitive (confidential data)
```

**Identity Generation:**
```
Username generation:
  Pattern: firstName.lastName
  Uniqueness: Add number if duplicate (john.smith2)
  
Display name generation:
  Pattern: firstName + " " + lastName
  Example: "John Smith"
```

---

## Testing Identity Profiles

### **Before Production**

**Test process:**
```
1. Create test identity profile in sandbox/test tenant

2. Use small test data set
   - 10-20 test employee records
   - Cover different scenarios (new, update, terminate)

3. Trigger aggregation

4. Verify results:
   - Identities created correctly?
   - Attributes mapped correctly?
   - Lifecycle states correct?
   - Correlation working?

5. Adjust configuration based on results

6. Test again until perfect

7. Deploy to production
```

**Validation checklist:**
```
☐ All expected identities created
☐ No duplicate identities
☐ Attributes populated correctly
☐ Email format correct
☐ Department mapped correctly
☐ Manager relationships correct
☐ Lifecycle states accurate
☐ Correlation working
☐ No errors in logs
☐ Performance acceptable
```

---

## Common Configuration Mistakes

### **Mistake 1: Wrong Authoritative Source**
```
❌ Wrong:
  Using Active Directory as authoritative source
  (AD doesn't have employee hire date, termination, etc.)

✓ Correct:
  Use HR system (Workday, SuccessFactors) as authoritative
  (Has complete employee lifecycle data)
```

### **Mistake 2: Missing Required Attributes**
```
❌ Wrong:
  Identity profile doesn't map employeeId
  Result: Correlation fails

✓ Correct:
  Always map unique identifier (employeeId)
  Used for correlation and identity lookup
```

### **Mistake 3: Complex Transforms Without Testing**
```
❌ Wrong:
  Transform: UPPER(SUBSTRING(firstName, 0, 1)) + lastName + "@company.com"
  Deployed to production without testing
  Result: Incorrect emails (JSmith@company.com instead of john.smith@company.com)

✓ Correct:
  Test all transforms with sample data
  Verify output matches expectations
  Test edge cases (names with spaces, special characters)
```

### **Mistake 4: No Lifecycle Management**
```
❌ Wrong:
  Identity profile doesn't configure lifecycle states
  Result: Terminated employees remain active

✓ Correct:
  Configure lifecycle states based on employment status
  Triggers leaver workflows automatically
```

---

## Practice Exercise

**Question:** You're creating an identity profile for employees from Workday. Workday has these fields: employeeNumber, firstName, lastName, workEmail, dept, managerEmpNumber. Configure basic attribute mappings.

<details>
<summary>Answer</summary>

**Identity Profile Configuration: Employees from Workday**

**Basic Information:**
```
Name: Employee Identity Profile
Description: Identity profile for all full-time employees from Workday
Authoritative Source: Workday
Owner: IAM Team
```

**Attribute Mappings:**

**1. Unique Identifier (Critical)**
```
Identity Attribute: employeeId
Mapping Type: Static
Source Attribute: workday.employeeNumber

Why:
  - Unique per employee
  - Never changes (even if name changes)
  - Used for correlation to accounts
  - Required for identity lookup
```

**2. First Name**
```
Identity Attribute: firstName
Mapping Type: Static
Source Attribute: workday.firstName

Example:
  Workday: firstName = "John"
  Identity: firstName = "John"
```

**3. Last Name**
```
Identity Attribute: lastName
Mapping Type: Static
Source Attribute: workday.lastName

Example:
  Workday: lastName = "Smith"
  Identity: lastName = "Smith"
```

**4. Display Name**
```
Identity Attribute: displayName
Mapping Type: Complex (Transform)
Transform: firstName + " " + lastName

Example:
  Workday: firstName = "John", lastName = "Smith"
  Identity: displayName = "John Smith"
  
Why complex:
  - Combines two fields
  - Consistent format
```

**5. Email**
```
Option 1: If Workday has email (Recommended)
  Identity Attribute: email
  Mapping Type: Static
  Source Attribute: workday.workEmail
  
  Example:
    Workday: workEmail = "john.smith@company.com"
    Identity: email = "john.smith@company.com"

Option 2: If Workday doesn't have email (Generate)
  Identity Attribute: email
  Mapping Type: Complex (Transform)
  Transform: LOWER(firstName + "." + lastName + "@company.com")
  
  Example:
    Workday: firstName = "John", lastName = "Smith"
    Identity: email = "john.smith@company.com"
```

**6. Department**
```
Identity Attribute: department
Mapping Type: Static (if dept is full name)
Source Attribute: workday.dept

OR

Mapping Type: Rule (if dept is code)
Rule: DepartmentCodeToName

Rule logic:
  If workday.dept = "IT" → "Information Technology"
  If workday.dept = "HR" → "Human Resources"
  If workday.dept = "FIN" → "Finance"
  Else → workday.dept

Example:
  Workday: dept = "IT"
  Identity: department = "Information Technology"
```

**7. Manager**
```
Identity Attribute: manager
Mapping Type: Static
Source Attribute: workday.managerEmpNumber

How it works:
  - ISC looks up identity with employeeId = managerEmpNumber
  - Links manager identity to this identity
  - Creates manager-employee relationship

Example:
  Workday: managerEmpNumber = "EMP99999"
  ISC finds: Identity with employeeId = "EMP99999" (Jane Doe)
  Result: John Smith's manager = Jane Doe
```

**8. Additional Recommended Attributes**
```
Identity Attribute: uid
Mapping Type: Complex
Transform: LOWER(firstName + "." + lastName)
Purpose: Username for account provisioning

Example:
  Workday: firstName = "John", lastName = "Smith"
  Identity: uid = "john.smith"
  Used when creating AD account: jsmith or john.smith
```

**Complete Mapping Table:**

| ISC Attribute | Mapping Type | Source/Transform | Example Result |
|---------------|--------------|------------------|----------------|
| **employeeId** | Static | workday.employeeNumber | EMP12345 |
| **firstName** | Static | workday.firstName | John |
| **lastName** | Static | workday.lastName | Smith |
| **displayName** | Complex | firstName + " " + lastName | John Smith |
| **email** | Static | workday.workEmail | john.smith@company.com |
| **department** | Static/Rule | workday.dept | Information Technology |
| **manager** | Static | workday.managerEmpNumber | EMP99999 (→ Jane Doe) |
| **uid** | Complex | LOWER(firstName + "." + lastName) | john.smith |

**Lifecycle Configuration:**
```
Since Workday fields not specified, assume:

If Workday has employeeStatus field:
  When workday.employeeStatus = "Active" → lifecycle = active
  When workday.employeeStatus = "Terminated" → lifecycle = inactive
  When workday.employeeStatus = "Leave" → lifecycle = leave

If Workday has terminationDate field:
  If workday.terminationDate is null → lifecycle = active
  If workday.terminationDate is not null → lifecycle = inactive
```

**Correlation Configuration:**
```
Primary Correlation: employeeId
  Account.employeeId = Identity.employeeId

Fallback Correlation: email
  Account.email = Identity.email
  (In case account doesn't have employeeId)
```

**Testing Plan:**
```
Test Case 1: New Employee
  Workday Data:
    employeeNumber: EMP12345
    firstName: John
    lastName: Smith
    workEmail: john.smith@company.com
    dept: IT
    managerEmpNumber: EMP99999
  
  Expected Identity:
    employeeId: EMP12345
    firstName: John
    lastName: Smith
    displayName: John Smith
    email: john.smith@company.com
    department: Information Technology
    manager: Jane Doe (EMP99999)
    uid: john.smith

Test Case 2: Name with Special Characters
  Workday Data:
    firstName: Mary-Ann
    lastName: O'Connor
  
  Expected:
    displayName: Mary-Ann O'Connor
    email: mary-ann.oconnor@company.com (if generated)
    uid: mary-ann.oconnor

Test Case 3: Manager Lookup
  Workday Data:
    managerEmpNumber: EMP00001
  
  Expected:
    manager field links to identity with employeeId EMP00001
    Manager relationship visible in ISC
```

**Validation:**
After aggregation, verify:
✓ Identity created with all attributes
✓ employeeId populated (critical for correlation)
✓ Email in correct format
✓ Manager relationship correct
✓ Department mapped correctly (if using rule)
✓ Display name formatted correctly
✓ No duplicate identities

