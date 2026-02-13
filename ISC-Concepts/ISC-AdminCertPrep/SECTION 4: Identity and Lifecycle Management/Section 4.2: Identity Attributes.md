# Section 4.2: Identity Attributes

**Master identity attribute mappings, configuration options, and transforms**

---

## Table of Contents

- [4.2.1 Identity Attribute Mappings](#421-identity-attribute-mappings)
- [4.2.2 Mapping Configuration Options (Static, Complex, Rules)](#422-mapping-configuration-options-static-complex-rules)
---

# 4.2.1 Identity Attribute Mappings

**Learning Objective:** Understand how source attributes map to identity attributes in ISC.

---

## What is Attribute Mapping?

**Definition:** The process of linking source system attributes to ISC identity attributes

**Simple concept:**
```
Source Attribute → Mapping → ISC Identity Attribute

Example:
  Workday.firstName → Mapping → identity.firstName
  Workday.workEmail → Mapping → identity.email
```

**Why mapping is needed:**
```
Problem:
  - Source uses "workEmail"
  - ISC uses "email"
  - Different names for same thing
  
Solution:
  - Mapping connects them
  - ISC knows workEmail = email
  - Data flows correctly
```

**Exam Tip:** Attribute mapping connects source field names to ISC identity attribute names.

---

## Standard ISC Identity Attributes

### **Core Attributes**

**Required for all identities:**

| ISC Attribute | Purpose | Example |
|---------------|---------|---------|
| **uid** | Unique username | john.smith |
| **email** | Email address | john.smith@company.com |
| **firstName** | First name | John |
| **lastName** | Last name | Smith |

**Common additional attributes:**

| ISC Attribute | Purpose | Example |
|---------------|---------|---------|
| **displayName** | Full name for display | John Smith |
| **employeeNumber** | Unique employee ID | EMP12345 |
| **department** | Department/org unit | Information Technology |
| **manager** | Manager relationship | Jane Doe |
| **location** | Office location | New York Office |
| **title** | Job title | Senior Developer |
| **phone** | Phone number | +1-555-0100 |
| **startDate** | Start date | 2023-01-15 |
| **endDate** | End date (if terminated) | 2024-02-13 |

### **Extended Attributes**

**Organization-specific:**
```
Custom attributes you can define:
  - costCenter
  - division
  - region
  - businessUnit
  - employeeType (FTE, contractor, etc.)
  - workSchedule
  - Any business-specific field
```

**Example custom attributes:**
```
Company ABC's custom attributes:
  - division: "North America Sales"
  - region: "Northeast"
  - costCenter: "CC-1234"
  - employeeType: "Full-Time Employee"
  - securityClearance: "Level 2"
```

---

## How Mapping Works

### **Basic Mapping Flow**
```
Step 1: Source Aggregation
  Workday aggregates → ISC receives data
  
  Example record:
    firstName: "John"
    lastName: "Smith"
    workEmail: "john.smith@company.com"
    employeeNumber: "EMP12345"
    dept: "IT"

Step 2: Identity Profile Evaluates
  Profile has mapping configuration
  
Step 3: Apply Mappings
  Source.firstName → Identity.firstName
  Source.lastName → Identity.lastName
  Source.workEmail → Identity.email
  Source.employeeNumber → Identity.employeeNumber
  Source.dept → Identity.department

Step 4: Identity Created/Updated
  Identity attributes populated:
    firstName: "John"
    lastName: "Smith"
    email: "john.smith@company.com"
    employeeNumber: "EMP12345"
    department: "IT"
```

### **Mapping Example - Workday to ISC**

**Source attributes (Workday):**
```json
{
  "employeeNumber": "EMP12345",
  "firstName": "John",
  "lastName": "Smith",
  "workEmail": "john.smith@company.com",
  "dept": "Information Technology",
  "managerEmployeeNumber": "EMP99999",
  "hireDate": "2023-01-15",
  "location": "New York Office"
}
```

**Mapping configuration:**
```
employeeNumber → identity.employeeNumber
firstName → identity.firstName
lastName → identity.lastName
workEmail → identity.email
dept → identity.department
managerEmployeeNumber → identity.manager (lookup by employeeNumber)
hireDate → identity.startDate
location → identity.location
```

**Resulting identity:**
```json
{
  "employeeNumber": "EMP12345",
  "firstName": "John",
  "lastName": "Smith",
  "email": "john.smith@company.com",
  "department": "Information Technology",
  "manager": "Jane Doe",
  "startDate": "2023-01-15",
  "location": "New York Office"
}
```

---

## Mapping Source Attributes

### **Identifying Available Attributes**

**In ISC, view source schema:**
```
Admin → Connections → Sources → [Source] → Schema

Shows all available attributes from source:
  - Attribute name
  - Data type
  - Sample values
  - Whether multi-valued
```

**Example - Active Directory Schema:**
```
Available Attributes:

sAMAccountName (String)
  Sample: jsmith
  
displayName (String)
  Sample: John Smith
  
mail (String)
  Sample: john.smith@company.com
  
department (String)
  Sample: Information Technology
  
memberOf (Multi-valued String)
  Sample: CN=VPN-Users,OU=Groups,DC=company,DC=com
```

### **Choosing Attributes to Map**

**Questions to ask:**
```
1. Is this attribute needed for identity management?
   - Employee ID: Yes (correlation)
   - Thumbnail photo: Maybe not
   
2. Is this attribute available in the source?
   - email: Check if source has it
   - If not, can generate it
   
3. Is this attribute used for:
   - Correlation? (employeeId, email)
   - Provisioning? (department, location)
   - Workflows? (manager, department)
   - Reporting? (department, location, title)
   
4. Does this attribute change frequently?
   - department: Yes, map it
   - Shoe size: Probably not needed
```

---

## Mapping Strategies

### **Strategy 1: Minimal Mapping**

**Map only essential attributes:**
```
Attributes:
  ✓ employeeNumber (correlation)
  ✓ firstName
  ✓ lastName
  ✓ email
  ✓ department (for basic provisioning)
  
Advantages:
  - Simple configuration
  - Faster aggregation
  - Less data to maintain
  
Disadvantages:
  - Limited functionality
  - May need to add more later
```

### **Strategy 2: Comprehensive Mapping**

**Map all useful attributes:**
```
Attributes:
  ✓ All core attributes
  ✓ Organizational attributes (dept, location, title)
  ✓ Manager relationships
  ✓ Dates (start, end)
  ✓ Custom attributes (cost center, division)
  
Advantages:
  - Rich identity data
  - Better reporting
  - More workflow options
  
Disadvantages:
  - More complex
  - Longer aggregation
  - More to maintain
```

### **Strategy 3: Purpose-Driven Mapping**

**Map based on specific needs:**
```
For Provisioning:
  ✓ department (determines access)
  ✓ location (for regional apps)
  ✓ title (for role-based access)
  
For Reporting:
  ✓ department
  ✓ manager
  ✓ location
  ✓ employeeType
  
For Compliance:
  ✓ startDate
  ✓ endDate
  ✓ manager
  ✓ department
```

**Recommended approach:**
```
Start with essentials:
  - Core attributes (name, email, ID)
  - Critical for provisioning (department)
  - Required for workflows (manager)
  
Add as needed:
  - New use case requires new attribute?
  - Add to mapping incrementally
  - Test and validate
```

---

## Special Mapping Cases

### **Multi-Valued Attributes**

**Scenario: One attribute with multiple values**

**Example - AD group memberships:**
```
Source attribute: memberOf (multi-valued)
  Values:
    - CN=VPN-Users,OU=Groups,DC=company,DC=com
    - CN=Email-Users,OU=Groups,DC=company,DC=com
    - CN=IT-All,OU=Groups,DC=company,DC=com

Mapping options:

Option 1: Don't map to identity
  - Group memberships tracked via accounts
  - Not needed on identity directly
  
Option 2: Map to custom multi-valued attribute
  - Create custom attribute: adGroups (multi-valued)
  - Maps all groups to identity
  - Used for reporting/visibility
```

### **Manager Relationships**

**Mapping manager attribute:**

**Source provides manager ID:**
```
Source attribute: managerEmployeeNumber = "EMP99999"

Mapping:
  identity.manager ← source.managerEmployeeNumber
  
ISC processing:
  1. Receives managerEmployeeNumber = EMP99999
  2. Looks up identity with employeeNumber = EMP99999
  3. Finds: Jane Doe
  4. Sets manager relationship: John Smith → manager → Jane Doe
  
Result:
  Identity shows: Manager = Jane Doe
  Relationship established for workflows/approvals
```

**Source provides manager name (less ideal):**
```
Source attribute: managerName = "Jane Doe"

Challenge:
  - Name might not be unique (two Jane Does)
  - Hard to correlate reliably
  
Better approach:
  - Use manager employee ID if available
  - Or manager email address
```

### **Date Attributes**

**Mapping date fields:**
```
Source formats vary:
  - "2023-01-15" (ISO format)
  - "01/15/2023" (US format)
  - "15/01/2023" (EU format)
  - "January 15, 2023" (text)

ISC expects:
  - ISO 8601 format: YYYY-MM-DD
  - Example: 2023-01-15

Mapping:
  If source uses ISO format:
    identity.startDate ← source.hireDate (direct)
    
  If source uses other format:
    Use transform to convert format
    identity.startDate ← DATEFORMAT(source.hireDate, "YYYY-MM-DD")
```

### **Null/Empty Attributes**

**Handling missing data:**
```
Source attribute: middleName (not always present)

Mapping options:

Option 1: Map anyway (allow null)
  identity.middleName ← source.middleName
  Result: middleName = null (if not present)
  
Option 2: Map with default
  identity.middleName ← source.middleName OR ""
  Result: middleName = "" (empty string if not present)
  
Option 3: Conditional mapping
  If source.middleName exists:
    Map it
  Else:
    Don't set attribute
```

---

## Attribute Data Types

### **Common Data Types**

| Type | Description | Example |
|------|-------------|---------|
| **String** | Text | "John Smith" |
| **Integer** | Whole number | 12345 |
| **Date** | Date value | 2023-01-15 |
| **Boolean** | True/False | true |
| **Multi-valued** | Array/List | ["IT", "Engineering"] |

**Mapping considerations:**
```
Source type must match ISC type:

✓ Compatible:
  Source: String → ISC: String
  Source: Date → ISC: Date
  
✗ Incompatible (need transform):
  Source: String "12345" → ISC: Integer
  Need: PARSEINT(source.attribute)
  
  Source: String "true" → ISC: Boolean
  Need: PARSEBOOL(source.attribute)
```

---

## Practice Exercise

**Question:** You're mapping Workday attributes to ISC identities. Workday has: employeeID, firstName, lastName, email, deptCode (like "IT", "HR"), managerID, hireDate. What attributes would you map and how?

<details>
<summary>Answer</summary>

**Recommended Attribute Mappings:**

**1. Employee ID (Critical - Correlation)**
```
Source: workday.employeeID
ISC Attribute: employeeNumber
Mapping Type: Static (direct copy)

Why:
  - Unique identifier
  - Never changes
  - Used for correlation
  - Required for identity lookup
  
Configuration:
  identity.employeeNumber ← workday.employeeID
```

**2. First Name**
```
Source: workday.firstName
ISC Attribute: firstName
Mapping Type: Static

Configuration:
  identity.firstName ← workday.firstName
  
Example:
  Workday: firstName = "John"
  Identity: firstName = "John"
```

**3. Last Name**
```
Source: workday.lastName
ISC Attribute: lastName
Mapping Type: Static

Configuration:
  identity.lastName ← workday.lastName
  
Example:
  Workday: lastName = "Smith"
  Identity: lastName = "Smith"
```

**4. Display Name (Generated)**
```
Source: Combine firstName + lastName
ISC Attribute: displayName
Mapping Type: Complex (Transform)

Configuration:
  identity.displayName ← firstName + " " + lastName
  
Example:
  Workday: firstName = "John", lastName = "Smith"
  Identity: displayName = "John Smith"
  
Why generate:
  - Consistent format across all identities
  - Workday might not have it
  - Useful for UI display
```

**5. Email**
```
Source: workday.email
ISC Attribute: email
Mapping Type: Static

Configuration:
  identity.email ← workday.email
  
Example:
  Workday: email = "john.smith@company.com"
  Identity: email = "john.smith@company.com"
  
Note: If Workday doesn't have email, generate it:
  identity.email ← LOWER(firstName + "." + lastName + "@company.com")
```

**6. Username (Generated)**
```
Source: Generated from name
ISC Attribute: uid
Mapping Type: Complex (Transform)

Configuration:
  identity.uid ← LOWER(firstName + "." + lastName)
  
Example:
  Workday: firstName = "John", lastName = "Smith"
  Identity: uid = "john.smith"
  
Why:
  - Used for account provisioning
  - Consistent username format
  - AD account name will be john.smith
```

**7. Department (With Translation)**
```
Source: workday.deptCode
ISC Attribute: department
Mapping Type: Rule (since it's a code that needs translation)

Rule Logic:
  If workday.deptCode = "IT" → "Information Technology"
  If workday.deptCode = "HR" → "Human Resources"
  If workday.deptCode = "FIN" → "Finance"
  If workday.deptCode = "ENG" → "Engineering"
  If workday.deptCode = "MKT" → "Marketing"
  Else → workday.deptCode (use as-is if unknown)

Example:
  Workday: deptCode = "IT"
  Identity: department = "Information Technology"
  
Why use rule:
  - Codes not user-friendly
  - Full names better for display/reporting
  - Standardized department names
  
Alternative (if Workday has full names):
  If Workday already has full department names, use static mapping:
    identity.department ← workday.department
```

**8. Manager (Relationship)**
```
Source: workday.managerID
ISC Attribute: manager
Mapping Type: Static (ISC handles lookup)

Configuration:
  identity.manager ← workday.managerID
  
How it works:
  1. Workday: managerID = "EMP99999"
  2. ISC looks up identity with employeeNumber = "EMP99999"
  3. Finds: Jane Doe (identity)
  4. Creates relationship: John Smith → manager → Jane Doe
  
Result:
  - Manager relationship established
  - Used in approval workflows
  - Manager sees direct reports
  - Organizational hierarchy built
```

**9. Start Date**
```
Source: workday.hireDate
ISC Attribute: startDate
Mapping Type: Static (if format compatible) or Complex (if conversion needed)

Configuration:
  If Workday uses ISO format (YYYY-MM-DD):
    identity.startDate ← workday.hireDate
    
  If Workday uses different format (MM/DD/YYYY):
    identity.startDate ← DATEFORMAT(workday.hireDate, "YYYY-MM-DD")
    
Example:
  Workday: hireDate = "2023-01-15"
  Identity: startDate = "2023-01-15"
  
Why important:
  - Track employment start
  - Calculate tenure
  - Compliance reporting
  - Access reviews (how long had access)
```

**10. Employee Type (Helpful for Workflows)**
```
Source: Could be derived from Workday field or set statically
ISC Attribute: employeeType (custom attribute)
Mapping Type: Static

Configuration:
  identity.employeeType ← "Employee"
  
Why:
  - Distinguish employees from contractors
  - Used in workflow conditions
  - Filtering in reports
  - Different provisioning rules
```

**Complete Mapping Table:**

| ISC Attribute | Source | Mapping Type | Transform/Rule | Example Result |
|---------------|--------|--------------|----------------|----------------|
| employeeNumber | workday.employeeID | Static | - | EMP12345 |
| firstName | workday.firstName | Static | - | John |
| lastName | workday.lastName | Static | - | Smith |
| displayName | firstName + lastName | Complex | firstName + " " + lastName | John Smith |
| email | workday.email | Static | - | john.smith@company.com |
| uid | firstName + lastName | Complex | LOWER(firstName + "." + lastName) | john.smith |
| department | workday.deptCode | Rule | Code-to-Name mapping | Information Technology |
| manager | workday.managerID | Static | ISC lookup | Jane Doe |
| startDate | workday.hireDate | Static/Complex | DATEFORMAT if needed | 2023-01-15 |
| employeeType | - | Static | "Employee" | Employee |

**Attributes NOT Mapped (Explanation):**
```
Could add but not essential:
  - endDate: Not in Workday (employee still active)
  - location: Not provided (could add if available)
  - title: Not provided (could add if available)
  - phone: Not provided (could add if available)
  - costCenter: Not provided (could add if available)
  
These can be added later if:
  - Workday provides them
  - Use cases require them
  - Start simple, expand as needed
```

**Testing Approach:**
```
Test Case 1: Standard Employee
  Workday Input:
    employeeID: EMP12345
    firstName: John
    lastName: Smith
    email: john.smith@company.com
    deptCode: IT
    managerID: EMP99999
    hireDate: 2023-01-15
    
  Expected Identity:
    employeeNumber: EMP12345
    firstName: John
    lastName: Smith
    displayName: John Smith
    email: john.smith@company.com
    uid: john.smith
    department: Information Technology
    manager: Jane Doe
    startDate: 2023-01-15
    employeeType: Employee

Test Case 2: Name with Hyphen
  Workday Input:
    firstName: Mary-Ann
    lastName: O'Connor
    
  Expected Identity:
    displayName: Mary-Ann O'Connor
    uid: mary-ann.oconnor
    
Test Case 3: Unknown Department Code
  Workday Input:
    deptCode: XYZ (unknown)
    
  Expected Identity:
    department: XYZ (use code as-is)

Test Case 4: No Manager
  Workday Input:
    managerID: (null/empty)
    
  Expected Identity:
    manager: (null) - CEO/top level has no manager
```

**Implementation Steps:**
