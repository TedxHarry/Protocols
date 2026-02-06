# DAY 3: LOOKUPS & REFERENCE DATA - AFTERNOON SESSION

## Table of Contents
- [Overview](#overview)
- [What You'll Build Today](#what-youll-build-today)
- [Pattern 1: Department Hierarchy Resolution](#pattern-1-department-hierarchy-resolution-45-minutes)
- [Pattern 2: Cross-Source Identity Enrichment](#pattern-2-cross-source-identity-enrichment-45-minutes)
- [Pattern 3: Manager Chain Resolution](#pattern-3-manager-chain-resolution-40-minutes)
- [Pattern 4: Access Level from Multiple Sources](#pattern-4-access-level-from-multiple-sources-40-minutes)
- [Day 3 Final Assessment](#day-3-final-assessment)
- [Homework Challenges](#homework-challenges)

---

## Overview

**Session Duration:** 2.5 hours

**Prerequisites:** 
- Completed Day 3 Morning Session
- Understand Lookup transforms
- Understand Account Attribute
- Comfortable with transform chaining

**Goal:** Build production-ready data integration patterns using lookups and cross-references

---

## What You'll Build Today

By the end of this afternoon session, you'll have built:

- ✅ Complete department hierarchy system (4 levels deep)
- ✅ Cross-source identity enrichment (combining AD + HR + app data)
- ✅ Manager chain resolution with multiple fallbacks
- ✅ Access level calculator using multiple sources and logic
- ✅ Production-ready patterns handling real-world complexity

These patterns demonstrate **enterprise-level** transform architectures!

---

## Pattern 1: Department Hierarchy Resolution (45 minutes)

### Business Requirements

Build a complete department hierarchy system:
```
Department Code --> Department Name --> Division --> Business Unit --> Executive Sponsor

Example flow:
"ENG-SW" --> "Software Engineering" --> "Engineering" --> "Technology" --> "CTO"

Requirements:
- Support 50+ department codes
- Map to 20 departments
- Roll up to 5 divisions
- Roll up to 3 business units
- Assign to 3 executives
- Handle unknown departments gracefully
```

---

### Step 1: Design the Lookup Table Strategy (10 min)

We'll create **4 lookup tables** for the hierarchy:
```
Table 1: DeptCodeToName
  Input: Department Code (ENG-SW, ENG-HW, FIN-AP, etc.)
  Output: Department Name (Software Engineering, etc.)

Table 2: DeptNameToDivision
  Input: Department Name
  Output: Division (Engineering, Finance, etc.)

Table 3: DivisionToBusinessUnit
  Input: Division
  Output: Business Unit (Technology, Corporate, Revenue)

Table 4: BusinessUnitToExecutive
  Input: Business Unit
  Output: Executive (CTO, CFO, CRO)
```

---

### Step 2: Create Lookup Tables (15 min)

#### Table 1: DeptCodeToName
```
Create lookup table: DeptCodeToName

Entries:
ENG-SW --> Software Engineering
ENG-HW --> Hardware Engineering
ENG-QA --> Quality Assurance
ENG-DEV --> DevOps Engineering
FIN-AP --> Accounts Payable
FIN-AR --> Accounts Receivable
FIN-TAX --> Tax & Compliance
HR-REC --> Recruiting
HR-COMP --> Compensation & Benefits
HR-DEV --> Learning & Development
IT-INFRA --> Infrastructure
IT-SEC --> Information Security
IT-SUPPORT --> IT Support
SALES-ENT --> Enterprise Sales
SALES-SMB --> SMB Sales
MKT-DIG --> Digital Marketing
MKT-PROD --> Product Marketing
... (add 30+ more codes)

Default: Unknown Department
```

---

#### Table 2: DeptNameToDivision
```
Create lookup table: DeptNameToDivision

Entries:
Software Engineering --> Engineering
Hardware Engineering --> Engineering
Quality Assurance --> Engineering
DevOps Engineering --> Engineering
Accounts Payable --> Finance
Accounts Receivable --> Finance
Tax & Compliance --> Finance
Recruiting --> Human Resources
Compensation & Benefits --> Human Resources
Learning & Development --> Human Resources
Infrastructure --> Information Technology
Information Security --> Information Technology
IT Support --> Information Technology
Enterprise Sales --> Sales
SMB Sales --> Sales
Digital Marketing --> Marketing
Product Marketing --> Marketing
... (continue for all departments)

Default: Other
```

---

#### Table 3: DivisionToBusinessUnit
```
Create lookup table: DivisionToBusinessUnit

Entries:
Engineering --> Technology
Information Technology --> Technology
Product Management --> Technology
Finance --> Corporate
Human Resources --> Corporate
Legal --> Corporate
Sales --> Revenue
Marketing --> Revenue
Customer Success --> Revenue
Other --> General

Default: General
```

---

#### Table 4: BusinessUnitToExecutive
```
Create lookup table: BusinessUnitToExecutive

Entries:
Technology --> CTO
Corporate --> CFO
Revenue --> CRO
General --> CEO

Default: CEO
```

---

### Step 3: Build the Transform Chain (20 min)

Now create attributes using these lookups in sequence:

#### Attribute 1: departmentName
```
1. Create attribute "departmentName"
2. Transform chain:

   Transform 1: Upper (normalize input)
     Input: dept_code
     Output: "ENG-SW" (from "eng-sw")
   
   Transform 2: Lookup
     Input: Output from Transform 1
     Table: DeptCodeToName
     Default: "Unknown Department"
     Output: "Software Engineering"

3. Save
```

---

#### Attribute 2: division
```
1. Create attribute "division"
2. Transform: Lookup
   Input: departmentName (attribute we just created)
   Table: DeptNameToDivision
   Default: "Other"
   Output: "Engineering"

3. Save
```

---

#### Attribute 3: businessUnit
```
1. Create attribute "businessUnit"
2. Transform: Lookup
   Input: division (attribute we created)
   Table: DivisionToBusinessUnit
   Default: "General"
   Output: "Technology"

3. Save
```

---

#### Attribute 4: executiveSponsor
```
1. Create attribute "executiveSponsor"
2. Transform: Lookup
   Input: businessUnit (attribute we created)
   Table: BusinessUnitToExecutive
   Default: "CEO"
   Output: "CTO"

3. Save
```

---

### Step 4: Test the Complete Chain (10 min)

#### Test Case 1: Complete Path
```
Input: dept_code = "eng-sw"

Transform Flow:
1. Upper: "eng-sw" --> "ENG-SW"
2. Lookup (DeptCodeToName): "ENG-SW" --> "Software Engineering"
   (departmentName = "Software Engineering")
3. Lookup (DeptNameToDivision): "Software Engineering" --> "Engineering"
   (division = "Engineering")
4. Lookup (DivisionToBusinessUnit): "Engineering" --> "Technology"
   (businessUnit = "Technology")
5. Lookup (BusinessUnitToExecutive): "Technology" --> "CTO"
   (executiveSponsor = "CTO")

Final Result:
  departmentName = "Software Engineering"
  division = "Engineering"
  businessUnit = "Technology"
  executiveSponsor = "CTO"
```

---

#### Test Case 2: Unknown Department Code
```
Input: dept_code = "UNKNOWN-XYZ"

Transform Flow:
1. Upper: "UNKNOWN-XYZ" --> "UNKNOWN-XYZ"
2. Lookup: "UNKNOWN-XYZ" --> "Unknown Department" (default)
   (departmentName = "Unknown Department")
3. Lookup: "Unknown Department" --> "Other" (default)
   (division = "Other")
4. Lookup: "Other" --> "General"
   (businessUnit = "General")
5. Lookup: "General" --> "CEO"
   (executiveSponsor = "CEO")

Final Result:
  departmentName = "Unknown Department"
  division = "Other"
  businessUnit = "General"
  executiveSponsor = "CEO"
```

**Notice:** Defaults cascade through the chain!

---

#### Test Case 3: Finance Department
```
Input: dept_code = "FIN-AP"

Transform Flow:
1. Upper: "FIN-AP"
2. Lookup: "FIN-AP" --> "Accounts Payable"
3. Lookup: "Accounts Payable" --> "Finance"
4. Lookup: "Finance" --> "Corporate"
5. Lookup: "Corporate" --> "CFO"

Final Result:
  departmentName = "Accounts Payable"
  division = "Finance"
  businessUnit = "Corporate"
  executiveSponsor = "CFO"
```

---

### Key Learning Points

- ✅ **Hierarchical lookups** create organizational structure
- ✅ **Each level builds on previous** - creates dependencies
- ✅ **Defaults cascade** through the chain
- ✅ **Normalization** (Upper) prevents lookup misses
- ✅ **Intermediate attributes** make debugging easier
- ✅ **Maintainable** - update tables without changing transforms

---

## Pattern 2: Cross-Source Identity Enrichment (45 minutes)

### Business Requirements

Enrich identity with data from multiple sources:
```
Sources:
- Workday (HR): hire_date, job_title, department
- Active Directory: email, phone, office_location
- Salesforce: territory, quota, sales_region
- Custom App: certifications, skills, projects

Goal: Create complete identity profile combining all sources
```

---

### Step 1: Understanding Cross-Source Data (10 min)

#### The Challenge

Identity has accounts on multiple sources, each with different attributes:
```
Identity: John Smith

Account 1: Workday
  - title: "Senior Account Executive"
  - department: "Sales"
  - hire_date: "2020-01-15"

Account 2: Active Directory
  - mail: "john.smith@company.com"
  - telephoneNumber: "555-1234"
  - physicalDeliveryOfficeName: "Austin"

Account 3: Salesforce
  - Territory__c: "West Region"
  - Quota__c: "500000"
  - Sales_Team__c: "Enterprise"

We want ALL of this on the identity!
```

---

### Step 2: Extract Data from Each Source (15 min)

#### Get Data from Workday (Authoritative Source)

Workday is authoritative, so these are direct mappings or already in identity profile:
```
Already mapped from Workday:
- title (direct mapping)
- department (direct mapping)
- hire_date (direct mapping)
```

---

#### Get Email from Active Directory
```
1. Create attribute "emailFromAD"
2. Transform: Account Attribute
   Source: Active Directory
   Attribute: mail
3. Save
```

---

#### Get Phone from Active Directory
```
1. Create attribute "phoneFromAD"
2. Transform chain:

   Transform 1: Account Attribute
     Source: Active Directory
     Attribute: telephoneNumber
     Output: "555-1234" (or null)
   
   Transform 2: Replace (clean formatting)
     Find: "-"
     Replace: ""
   
   Transform 3: Replace
     Find: " "
     Replace: ""

3. Save
```

---

#### Get Office Location from AD
```
1. Create attribute "officeLocation"
2. Transform: Account Attribute
   Source: Active Directory
   Attribute: physicalDeliveryOfficeName
   Output: "Austin"
3. Save
```

---

#### Get Sales Territory from Salesforce
```
1. Create attribute "salesTerritory"
2. Transform: Account Attribute
   Source: Salesforce
   Attribute: Territory__c
   Output: "West Region"
3. Save
```

---

#### Get Sales Quota from Salesforce
```
1. Create attribute "salesQuota"
2. Transform: Account Attribute
   Source: Salesforce
   Attribute: Quota__c
   Output: "500000"
3. Save
```

---

### Step 3: Create Derived Attributes (10 min)

Now combine data from multiple sources:

#### Derive Primary Contact Method
```
1. Create attribute "primaryContact"
2. Transform: FirstValid
   Values:
   - emailFromAD
   - phoneFromAD
   - Static "No Contact Info"
3. Save
```

---

#### Derive Sales Region from Territory + Office
```
1. Create lookup table: TerritoryToRegion

   West Region --> Americas
   East Region --> Americas
   EMEA Territory --> EMEA
   APAC Territory --> APAC

2. Create attribute "salesRegion"
3. Transform: Lookup
   Input: salesTerritory
   Table: TerritoryToRegion
   Default: officeLocation (use office as fallback!)
4. Save
```

> **Note:** Some transforms allow using another attribute as default instead of static value. If not, use FirstValid:
```
Attribute: salesRegion
Transform: FirstValid
  Values:
  - [Lookup result]
  - officeLocation
  - Static "Unknown"
```

---

#### Derive Employee Level

Combine title + hire_date for seniority:
```
1. Create attribute "employeeLevel"
2. Transform: Conditional

   IF title contains "VP" OR title contains "Director"
     THEN "Leadership"
   ELSE IF title contains "Senior" OR hire_date < "2022-01-01"
     THEN "Senior"
   ELSE IF title contains "Manager"
     THEN "Management"
   ELSE
     "Individual Contributor"

3. Save
```

---

### Step 4: Test Cross-Source Enrichment (10 min)

#### Complete Test Case
```
Identity: John Smith

Source Data:
  Workday:
    title = "Senior Account Executive"
    department = "Sales"
    hire_date = "2020-01-15"
  
  Active Directory:
    mail = "john.smith@company.com"
    telephoneNumber = "555-1234"
    physicalDeliveryOfficeName = "Austin"
  
  Salesforce:
    Territory__c = "West Region"
    Quota__c = "500000"

Enriched Identity Attributes:
  title = "Senior Account Executive" (Workday)
  department = "Sales" (Workday)
  hire_date = "2020-01-15" (Workday)
  emailFromAD = "john.smith@company.com" (AD)
  phoneFromAD = "5551234" (AD, cleaned)
  officeLocation = "Austin" (AD)
  salesTerritory = "West Region" (Salesforce)
  salesQuota = "500000" (Salesforce)
  primaryContact = "john.smith@company.com" (derived)
  salesRegion = "Americas" (derived from lookup)
  employeeLevel = "Senior" (derived from title + hire date)
```

---

### Key Learning Points

- ✅ **Account Attribute** pulls data from any source
- ✅ **Multiple sources** enrich single identity
- ✅ **Each source** contributes different attributes
- ✅ **Derived attributes** combine cross-source data
- ✅ **FirstValid** prioritizes sources
- ✅ **Null handling** critical when accounts might not exist

---

## Pattern 3: Manager Chain Resolution (40 minutes)

### Business Requirements

Resolve manager information with sophisticated fallback:
```
Get Manager Email with this priority:

1. If manager has AD account, use AD email
2. If no AD, generate from manager's name (firstname.lastname@company.com)
3. If no name, use manager's employee ID email (empid@company.com)
4. If no manager at all, look up department head
5. If no department head, use "no-manager@company.com"

Also get:
- Manager's full name
- Manager's title
- Manager's department
- Skip level manager (manager's manager)
```

---

### Step 1: Get Manager's Basic Info (10 min)

First, ensure you have manager correlation configured. This is typically done at the source level.

Once configured, the identity has a `manager` attribute that references the manager's identity.

#### Get Manager's Display Name
```
1. Create attribute "managerDisplayName"
2. Transform: Identity Attribute
   Attribute: manager.displayName
   (This references the manager's displayName attribute)
3. Save
```

> **Note:** Syntax varies by ISC version. May be `manager.displayName` or require separate transform.

---

#### Get Manager's Title
```
1. Create attribute "managerTitle"
2. Transform: Identity Attribute
   Attribute: manager.title
3. Save
```

---

#### Get Manager's Department
```
1. Create attribute "managerDepartment"
2. Transform: Identity Attribute
   Attribute: manager.department
3. Save
```

---

### Step 2: Get Manager's Email with Fallbacks (15 min)

#### Helper Attribute 1: Manager Email from AD
```
1. Create attribute "managerEmailFromAD"
2. Transform: Account Attribute
   Source: Active Directory
   Identity: manager (get from manager's account, not user's)
   Attribute: mail
3. Save
```

> **Note:** Getting Account Attribute from manager's account requires specific syntax. May look like:
> - Some versions: Specify "Identity Context" as Manager
> - Other versions: Use manager.accounts["Active Directory"].mail

Consult your ISC version documentation for exact syntax.

---

#### Helper Attribute 2: Generated Manager Email from Name
```
1. Create attribute "managerEmailGenerated"
2. Transform chain:

   Transform 1: Identity Attribute
     Attribute: manager.firstname
   
   Transform 2: Identity Attribute
     Attribute: manager.lastname
   
   Transform 3: Conditional
     IF manager.firstname ne null AND manager.lastname ne null
       THEN continue
       ELSE return null
   
   Transform 4: Concatenation
     manager.firstname + "." + manager.lastname + "@company.com"
   
   Transform 5: Lower

3. Save
```

---

#### Helper Attribute 3: Manager Email from Employee ID
```
1. Create attribute "managerEmailFromID"
2. Transform chain:

   Transform 1: Identity Attribute
     Attribute: manager.employeeID
   
   Transform 2: Conditional
     IF manager.employeeID ne null
       THEN continue
       ELSE return null
   
   Transform 3: Concatenation
     manager.employeeID + "@company.com"
   
   Transform 4: Lower

3. Save
```

---

#### Helper Attribute 4: Department Head Email
```
1. Create lookup table: DepartmentHeadEmails

   Sales --> sales-head@company.com
   Engineering --> eng-head@company.com
   Finance --> finance-head@company.com
   ... (all departments)

2. Create attribute "deptHeadEmail"
3. Transform: Lookup
   Input: department (user's department)
   Table: DepartmentHeadEmails
   Default: null
4. Save
```

---

#### Final Attribute: Manager Email (Combined)
```
1. Create attribute "managerEmail"
2. Transform: FirstValid
   Values (in priority order):
   1. managerEmailFromAD
   2. managerEmailGenerated
   3. managerEmailFromID
   4. deptHeadEmail
   5. Static "no-manager@company.com"
3. Save
```

---

### Step 3: Get Skip-Level Manager (15 min)

Skip-level manager = manager's manager

#### Get Skip-Level Manager Name
```
1. Create attribute "skipLevelManagerName"
2. Transform: Identity Attribute
   Attribute: manager.manager.displayName
   (This chains through manager to their manager)
3. Save
```

---

#### Get Skip-Level Manager Email
```
1. Create attribute "skipLevelManagerEmail"
2. Transform: Account Attribute
   Source: Active Directory
   Identity: manager.manager
   Attribute: mail
3. Save
```

> **Note:** If manager or manager's manager doesn't exist, returns null.

---

### Step 4: Test Manager Chain (10 min)

#### Test Case 1: Complete Manager Chain
```
User: John Smith
Manager: Jane Doe (has AD account)
Skip-Level: Bob Johnson (Jane's manager, has AD account)

Results:
  managerDisplayName = "Jane Doe"
  managerTitle = "Sales Director"
  managerDepartment = "Sales"
  managerEmailFromAD = "jane.doe@company.com"
  managerEmail = "jane.doe@company.com" (Priority 1)
  skipLevelManagerName = "Bob Johnson"
  skipLevelManagerEmail = "bob.johnson@company.com"
```

---

#### Test Case 2: Manager Has No AD Account
```
User: John Smith
Manager: Jane Doe (NO AD account)
  firstname = "Jane"
  lastname = "Doe"

Results:
  managerDisplayName = "Jane Doe"
  managerEmailFromAD = null (no AD account)
  managerEmailGenerated = "jane.doe@company.com"
  managerEmail = "jane.doe@company.com" (Priority 2)
```

---

#### Test Case 3: No Manager Assigned
```
User: John Smith
Manager: null
Department: "Sales"

Results:
  managerDisplayName = null
  managerEmailFromAD = null
  managerEmailGenerated = null
  managerEmailFromID = null
  deptHeadEmail = "sales-head@company.com" (from lookup)
  managerEmail = "sales-head@company.com" (Priority 4)
```

---

### Key Learning Points

- ✅ **Identity Attribute** can chain through relationships (manager.manager)
- ✅ **Account Attribute** can get data from related identities' accounts
- ✅ **Multiple fallbacks** ensure always have manager contact
- ✅ **Department head** good backup when no direct manager
- ✅ **Skip-level** useful for escalations and org charts
- ✅ **Null handling** critical in manager chains (not everyone has manager)

---

## Pattern 4: Access Level from Multiple Sources (40 minutes)

### Business Requirements

Calculate access level based on multiple factors from different sources:
```
Determine access level using:

From HR (Workday):
- Job title
- Years of service
- Department

From AD:
- Group memberships
- Account status

From Security App:
- Security clearance level
- Training completion

Logic:
- If in "Admins" AD group --> "System Administrator"
- If VP/Director title --> "Executive"
- If security clearance = "Secret" --> "Confidential Access"
- If Manager title AND in "Managers" group --> "Manager"
- If years > 5 --> "Senior"
- Otherwise --> "Standard"
```

---

### Step 1: Get Data from Each Source (15 min)

#### From Workday (Already Mapped)
```
Attributes already available:
- title
- department
- hire_date
```

---

#### From Active Directory
```
1. Create attribute "adGroups"
2. Transform: Account Attribute
   Source: Active Directory
   Attribute: memberOf
   Output: ["CN=Admins,...", "CN=Managers,...", ...]
3. Save
```

---

#### From Security App
```
1. Create attribute "securityClearance"
2. Transform: Account Attribute
   Source: SecurityApp
   Attribute: clearance_level
   Output: "Secret" (or "Confidential", "Public", etc.)
3. Save
```
```
1. Create attribute "trainingComplete"
2. Transform: Account Attribute
   Source: SecurityApp
   Attribute: security_training_complete
   Output: "true" or "false"
3. Save
```

---

#### Calculate Years of Service
```
1. Create attribute "yearsOfService"
2. Transform chain:

   Transform 1: [Calculate difference between hire_date and today]
   (Simplified for now - we'll learn proper date math in Day 4)
   
   For now, use conditional:
   IF hire_date contains "2020" OR earlier
     THEN 5 (or more)
   ELSE IF hire_date contains "2021"
     THEN 4
   ... etc

3. Save
```

> **Note:** Proper date math coming in Day 4. This is simplified.

---

### Step 2: Check AD Group Membership (10 min)

Need to check if user is in specific groups.

#### Helper: Is in Admins Group
```
1. Create attribute "isInAdminsGroup"
2. Transform: Conditional
   
   Condition: adGroups contains "Admins"
   If True: "true"
   If False: "false"

3. Save
```

---

#### Helper: Is in Managers Group
```
1. Create attribute "isInManagersGroup"
2. Transform: Conditional
   
   Condition: adGroups contains "Managers"
   If True: "true"
   If False: "false"

3. Save
```

---

### Step 3: Calculate Access Level (15 min)

#### Build Decision Tree
```
1. Create attribute "accessLevel"
2. Build nested conditional chain:

LEVEL 1: Check for System Admin
Transform 1: Conditional
  Condition: isInAdminsGroup eq "true"
  If True: "System Administrator"
  If False: Continue

LEVEL 2: Check for Executive
Transform 2: Conditional
  Condition: title contains "VP" OR title contains "Director"
  If True: "Executive"
  If False: Continue

LEVEL 3: Check for Confidential Access
Transform 3: Conditional
  Condition: securityClearance eq "Secret"
  If True: "Confidential Access"
  If False: Continue

LEVEL 4: Check for Manager
Transform 4: Conditional
  Condition: title contains "Manager" AND isInManagersGroup eq "true"
  If True: "Manager"
  If False: Continue

LEVEL 5: Check for Senior
Transform 5: Conditional
  Condition: yearsOfService ge 5
  If True: "Senior"
  If False: "Standard"

3. Save
```

---

### Step 4: Test Access Level Calculation (10 min)

#### Test Case 1: System Administrator
```
User: Admin User

Data:
  adGroups contains "Admins" --> isInAdminsGroup = "true"
  title = "IT Manager"
  securityClearance = "Public"
  yearsOfService = 3

Result:
  accessLevel = "System Administrator" (stops at Level 1)
```

---

#### Test Case 2: Executive
```
User: VP Sales

Data:
  adGroups does NOT contain "Admins" --> isInAdminsGroup = "false"
  title = "VP Sales" --> contains "VP"
  securityClearance = "Confidential"

Result:
  accessLevel = "Executive" (stops at Level 2)
```

---

#### Test Case 3: Manager
```
User: Engineering Manager

Data:
  adGroups = ["Managers", "Engineering"]
  isInAdminsGroup = "false"
  isInManagersGroup = "true"
  title = "Engineering Manager"
  securityClearance = "Public"
  yearsOfService = 4

Result:
  accessLevel = "Manager" (stops at Level 4)
  (Skipped Level 2 because title doesn't have VP/Director)
  (Skipped Level 3 because clearance not Secret)
```

---

#### Test Case 4: Senior Employee
```
User: Long-time Employee

Data:
  adGroups = ["Engineering", "Developers"]
  isInAdminsGroup = "false"
  isInManagersGroup = "false"
  title = "Software Engineer" (no Manager/VP/Director)
  securityClearance = "Public"
  yearsOfService = 6

Result:
  accessLevel = "Senior" (stops at Level 5)
```

---

#### Test Case 5: Standard Employee
```
User: New Hire

Data:
  adGroups = ["Engineering"]
  isInAdminsGroup = "false"
  title = "Junior Engineer"
  securityClearance = "Public"
  yearsOfService = 1

Result:
  accessLevel = "Standard" (reaches final else)
```

---

### Key Learning Points

- ✅ **Multiple sources** contribute to single decision
- ✅ **Priority order** determines which rule wins
- ✅ **Helper attributes** simplify complex conditionals
- ✅ **Conditional chains** create decision trees
- ✅ **First match wins** - order is critical
- ✅ **Combining Account Attribute + Conditional** = powerful patterns

---

## Day 3 Final Assessment

### What You Built Today

**Morning Session:**
- [ ] Department code lookup
- [ ] Title standardization lookup
- [ ] Multi-column location mapping
- [ ] Hierarchical department lookup chain
- [ ] Email from AD (Account Attribute)
- [ ] AD groups extraction
- [ ] Identity Attribute references

**Afternoon Session:**
- [ ] Complete department hierarchy (4 levels)
- [ ] Cross-source identity enrichment (3 sources)
- [ ] Manager chain resolution (5 fallback levels)
- [ ] Access level from multiple sources (5-level decision tree)

**Total: 15+ production-ready lookup and reference patterns!**

---

### Concepts Mastered

- ✅ Creating and using lookup tables
- ✅ Multi-level hierarchical lookups
- ✅ Chaining lookups for organizational structure
- ✅ Account Attribute for cross-source data
- ✅ Identity Attribute for cross-referencing
- ✅ Manager chain traversal (manager.manager)
- ✅ Combining multiple sources in decisions
- ✅ FirstValid for multi-source priority
- ✅ Helper attributes for complex logic
- ✅ Production-ready data integration patterns

---

### Final Challenge Quiz

#### Challenge 1: Three-Level Lookup

Build a three-level lookup chain:
```
Location Code --> City --> State --> Country

Location codes:
  ATX --> Austin --> TX --> USA
  NYC --> New York --> NY --> USA
  LON --> London --> null --> UK
```

<details>
<summary>Click to reveal solution</summary>

**Create 3 Lookup Tables:**

**Table 1: LocationToCity**
```
ATX --> Austin
NYC --> New York
LON --> London
```

**Table 2: CityToState**
```
Austin --> TX
New York --> NY
London --> null (or empty)
```

**Table 3: StateToCountry**
```
TX --> USA
NY --> USA
null --> UK (if UK cities return null state)
```

**Alternative for Table 3:** Create CityToCountry for cities without states:
```
London --> UK
```

**Create Attributes:**
```
Attribute: city
Transform: Lookup
  Input: location_code
  Table: LocationToCity

Attribute: state
Transform: Lookup
  Input: city
  Table: CityToState

Attribute: country
Transform: FirstValid
  Values:
  1. [Lookup state --> StateToCountry]
  2. [Lookup city --> CityToCountry] (direct city to country)
  3. Static "Unknown"
```

</details>

---

#### Challenge 2: Multi-Source Contact Info

Get contact info with priority:
```
1. Mobile from HR system
2. Work phone from AD
3. Office phone from location lookup
4. Default "555-0000"

All phones should be cleaned (no formatting)
```

<details>
<summary>Click to reveal solution</summary>

**Solution:**
```
Helper 1: cleanedMobileFromHR
Transform chain:
1. Source attribute: mobile_phone
2. Replace "(" --> ""
3. Replace ")" --> ""
4. Replace " " --> ""
5. Replace "-" --> ""

Helper 2: workPhoneFromAD
Transform chain:
1. Account Attribute (AD, telephoneNumber)
2. Replace "(", ")", " ", "-" (multiple Replace transforms)

Helper 3: officePhoneFromLocation
Transform:
1. Lookup (office_location --> LocationToOfficePhone)

Final: primaryPhone
Transform: FirstValid
  Values:
  1. cleanedMobileFromHR
  2. workPhoneFromAD
  3. officePhoneFromLocation
  4. Static "555-0000"
```

</details>

---

#### Challenge 3: Complex Access Decision

Calculate access based on:
```
If (title = "CEO" OR title = "President")
  AND security_clearance = "Top Secret"
  --> "C-Level Confidential"

Else if in "BoardMembers" AD group
  --> "Board Access"

Else if (department = "Finance" OR department = "Legal")
  AND years_of_service > 3
  --> "Confidential"

Else
  --> "Standard"
```

<details>
<summary>Click to reveal solution</summary>

**Solution:**
```
Helper 1: isCLevelTitle
Transform: Conditional
  Condition: title eq "CEO" OR title eq "President"
  If True: "true"
  If False: "false"

Helper 2: hasTopSecret
Transform: Conditional
  Condition: security_clearance eq "Top Secret"
  If True: "true"
  If False: "false"

Helper 3: isFinanceOrLegal
Transform: Conditional
  Condition: department eq "Finance" OR department eq "Legal"
  If True: "true"
  If False: "false"

Helper 4: isInBoardGroup
Transform: Conditional
  Condition: adGroups contains "BoardMembers"
  If True: "true"
  If False: "false"

Final: accessLevel
Transform chain:

Level 1: Check C-Level
  Conditional:
    IF isCLevelTitle eq "true" AND hasTopSecret eq "true"
      TRUE --> "C-Level Confidential"
      FALSE --> Continue

Level 2: Check Board
  Conditional:
    IF isInBoardGroup eq "true"
      TRUE --> "Board Access"
      FALSE --> Continue

Level 3: Check Confidential Departments
  Conditional:
    IF isFinanceOrLegal eq "true" AND yearsOfService gt 3
      TRUE --> "Confidential"
      FALSE --> "Standard"
```

</details>

---

## Homework Challenges

### Challenge 1: Cost Center Hierarchy

Build a complete cost center system:
```
Employee --> Department --> Cost Center --> Budget Owner --> Approver Chain

Requirements:
- 50+ employees map to 20 departments
- 20 departments map to 10 cost centers
- Each cost center has budget owner
- Each budget owner has approval chain (2-3 levels)
```

Create lookup tables and transform chains.

---

### Challenge 2: Multi-Source Profile Completion

Calculate profile completion percentage:
```
Check if these exist (from different sources):
- Email (AD)
- Phone (AD)
- Office location (AD)
- Title (HR)
- Department (HR)
- Manager (correlation)
- Photo (AD - thumbnailPhoto attribute)
- Skills (custom app)

Profile Completion = (fields filled / total fields) * 100

Return: "75% Complete"
```

**Hint:** Create a conditional for each field (exists = 1, null = 0), sum them, calculate percentage.

---

### Challenge 3: Smart Department Assignment

Build intelligent department assignment:
```
Priority 1: Use HR department if exists
Priority 2: Infer from AD groups (if in "Engineering" group, dept = "Engineering")
Priority 3: Infer from email domain (if @sales.company.com, dept = "Sales")
Priority 4: Infer from manager's department
Priority 5: Use "Unassigned"

Then standardize through lookup table (50+ variations to 10 standard depts)
```

---

## Day 3 Complete!

### You've Mastered

- ✅ Lookup tables for value mapping
- ✅ Multi-level hierarchical lookups
- ✅ Account Attribute transform
- ✅ Identity Attribute references
- ✅ Manager chain traversal
- ✅ Cross-source data enrichment
- ✅ Multi-source decision making
- ✅ Complex transform chains (10+ steps)
- ✅ Production-ready integration patterns

---

## What's Next: Day 4 Preview

Tomorrow you'll learn:

### Date/Time & List Operations

- **Date Format** - Convert between date formats
- **Date Math** - Add/subtract days, months, years
- **Date Compare** - Compare dates for decisions
- **ISO8601** - Master the required format
- **Split** - String to array
- **Index** - Get items from arrays
- **Join** - Array to string

### Real-World Applications

- Proper lifecycle state calculation with real dates
- Account expiration dates
- Next processing date logic
- Parsing complex strings
- Multi-value attribute manipulation

**You'll complete your lifecycle state calculator with proper date logic!**

---

## Study Tips for Tomorrow

### Review These Concepts

- ISO8601 date format: `2024-01-15T00:00:00Z`
- Date comparison logic
- Array/list operations
- Multi-value attributes

### Prepare Questions

Think about:
- How do you calculate 90 days from today?
- How do you compare hire_date to current date?
- How do you extract first item from a list?
- How do you handle multi-value attributes?

---

**Excellent work on Day 3! You've built enterprise-level data integration skills!**

---

*Part of the SailPoint ISC Transforms Mastery 7-Day Study Plan*
