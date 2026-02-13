# 4.2.2 Mapping Configuration Options (Static, Complex, Rules)

**Learning Objective:** Master the three mapping configuration types and when to use each.

---

## Three Mapping Types

### **Overview**

| Type | Use Case | Complexity | Example |
|------|----------|------------|---------|
| **Static** | Direct copy | Simple | firstName ← source.firstName |
| **Complex** | Transform/calculate | Medium | email ← firstName + "@company.com" |
| **Rule** | Conditional logic | Advanced | If dept="IT" → "Information Technology" |

**Exam Tip:** Static = copy, Complex = transform, Rule = if/then logic.

---

# Static Mapping

## What is Static Mapping?

**Definition:** Direct one-to-one copy from source attribute to identity attribute

**Characteristics:**
```
✓ No transformation
✓ No logic
✓ Value copied as-is
✓ Simplest mapping type
```

**When to use:**
```
Use static when:
  - Source attribute matches ISC attribute format
  - No transformation needed
  - Value can be used directly
  
Examples:
  ✓ firstName (already correct)
  ✓ lastName (already correct)
  ✓ email (if in correct format)
  ✓ employeeNumber (unique ID)
```

---

## Static Mapping Configuration

### **Basic Setup**

**Configuration:**
```
Identity Attribute: firstName
Mapping Type: Static
Source Attribute: workday.firstName
```

**Visual flow:**
```
Workday.firstName = "John"
        ↓ (static mapping - direct copy)
Identity.firstName = "John"
```

### **Common Static Mappings**

**Example 1: First Name**
```
Configuration:
  ISC Attribute: firstName
  Type: Static
  Source: workday.firstName

Processing:
  Source value: "John"
  Identity value: "John"
  
No changes, direct copy
```

**Example 2: Last Name**
```
Configuration:
  ISC Attribute: lastName
  Type: Static
  Source: workday.lastName

Processing:
  Source value: "Smith"
  Identity value: "Smith"
```

**Example 3: Email (if already formatted)**
```
Configuration:
  ISC Attribute: email
  Type: Static
  Source: workday.workEmail

Processing:
  Source value: "john.smith@company.com"
  Identity value: "john.smith@company.com"
  
Works because email already in correct format
```

**Example 4: Employee ID**
```
Configuration:
  ISC Attribute: employeeNumber
  Type: Static
  Source: workday.employeeID

Processing:
  Source value: "EMP12345"
  Identity value: "EMP12345"
  
Critical for correlation - always use static (no transform)
```

**Example 5: Department (if full name)**
```
Configuration:
  ISC Attribute: department
  Type: Static
  Source: workday.department

Processing:
  Source value: "Information Technology"
  Identity value: "Information Technology"
  
Only if source already has full department name
If source has code (IT), use rule instead
```

---

## Static Mapping Advantages
```
✓ Simple to configure
✓ Easy to understand
✓ Fast processing (no transforms)
✓ Reliable (no logic to break)
✓ Easy to troubleshoot
```

## Static Mapping Limitations
```
✗ Cannot transform data
✗ Cannot combine fields
✗ Cannot apply logic
✗ Cannot format values
✗ Source must match ISC format exactly
```

**When static doesn't work:**
```
❌ Source: "JOHN" → Need: "John" (capitalization)
   Solution: Use complex mapping to format

❌ Source: firstName="John", lastName="Smith" → Need: "John Smith"
   Solution: Use complex mapping to combine

❌ Source: deptCode="IT" → Need: "Information Technology"
   Solution: Use rule mapping to translate

❌ Source: "01/15/2023" → Need: "2023-01-15"
   Solution: Use complex mapping to reformat
```

---

# Complex Mapping

## What is Complex Mapping?

**Definition:** Uses transforms to calculate, combine, or format attribute values

**Characteristics:**
```
✓ Can transform data
✓ Can combine multiple fields
✓ Can format/calculate values
✓ Uses SailPoint transform syntax
```

**When to use:**
```
Use complex when:
  - Need to combine fields (firstName + lastName)
  - Need to format data (UPPER, LOWER)
  - Need to calculate values
  - Need to manipulate strings
  
Examples:
  ✓ displayName (firstName + lastName)
  ✓ email (generate from name)
  ✓ uid (username from name)
  ✓ Formatting (UPPER, LOWER, SUBSTRING)
```

---

## Complex Mapping Configuration

### **Basic Setup**

**Configuration:**
```
Identity Attribute: displayName
Mapping Type: Complex
Transform: firstName + " " + lastName
```

**Visual flow:**
```
Workday.firstName = "John"
Workday.lastName = "Smith"
        ↓ (complex mapping - transform)
        firstName + " " + lastName
        ↓
Identity.displayName = "John Smith"
```

---

## Common Transform Operations

### **String Concatenation**

**Combine multiple fields:**
```
Transform: firstName + " " + lastName

Input:
  firstName = "John"
  lastName = "Smith"
  
Output:
  "John Smith"
```

**Generate email:**
```
Transform: firstName + "." + lastName + "@company.com"

Input:
  firstName = "John"
  lastName = "Smith"
  
Output:
  "John.Smith@company.com"
  
Problem: Capital letters in email
Solution: Add LOWER function
```

### **Case Conversion**

**LOWER - Convert to lowercase:**
```
Transform: LOWER(firstName + "." + lastName + "@company.com")

Input:
  firstName = "John"
  lastName = "Smith"
  
Output:
  "john.smith@company.com"
```

**UPPER - Convert to uppercase:**
```
Transform: UPPER(firstName)

Input:
  firstName = "John"
  
Output:
  "JOHN"
  
Use case: Some systems require uppercase usernames
```

**PROPERCASE - Capitalize first letter:**
```
Transform: PROPERCASE(firstName)

Input:
  firstName = "JOHN"
  
Output:
  "John"
  
Use case: Fix all-caps data from source
```

### **Substring Operations**

**SUBSTRING - Extract part of string:**
```
Transform: SUBSTRING(firstName, 0, 1) + SUBSTRING(lastName, 0, 1)

Input:
  firstName = "John"
  lastName = "Smith"
  
Breakdown:
  SUBSTRING("John", 0, 1) = "J"
  SUBSTRING("Smith", 0, 1) = "S"
  "J" + "S" = "JS"
  
Output:
  "JS"
  
Use case: Generate initials
```

**Extract first initial:**
```
Transform: SUBSTRING(firstName, 0, 1)

Input:
  firstName = "John"
  
Output:
  "J"
```

### **Conditional Operations**

**If-then-else in transform:**
```
Transform: IF(department != null, department, "Unassigned")

Input (has department):
  department = "IT"
Output:
  "IT"

Input (no department):
  department = null
Output:
  "Unassigned"
  
Use case: Provide default value when source field empty
```

**Nested conditions:**
```
Transform: IF(employeeType == "FTE", "Employee", IF(employeeType == "CTR", "Contractor", "Other"))

Input (FTE):
  employeeType = "FTE"
Output:
  "Employee"

Input (CTR):
  employeeType = "CTR"
Output:
  "Contractor"

Input (Other):
  employeeType = "TEMP"
Output:
  "Other"
```

### **Date Formatting**

**DATEFORMAT - Convert date formats:**
```
Transform: DATEFORMAT(hireDate, "yyyy-MM-dd")

Input:
  hireDate = "01/15/2023" (MM/DD/YYYY)
  
Output:
  "2023-01-15" (ISO format)
  
Use case: Standardize date format for ISC
```

---

## Complex Mapping Examples

### **Example 1: Display Name**

**Requirement:** Combine first and last name with space

**Configuration:**
```
Attribute: displayName
Type: Complex
Transform: firstName + " " + lastName

Test Cases:
  firstName="John", lastName="Smith" → "John Smith"
  firstName="Mary", lastName="Johnson" → "Mary Johnson"
```

### **Example 2: Email Generation**

**Requirement:** Generate email from name in lowercase

**Configuration:**
```
Attribute: email
Type: Complex
Transform: LOWER(firstName + "." + lastName + "@company.com")

Test Cases:
  firstName="John", lastName="Smith" → "john.smith@company.com"
  firstName="Mary-Ann", lastName="O'Connor" → "mary-ann.o'connor@company.com"
```

**Handle duplicates (advanced):**
```
Problem: Two John Smiths

Transform with logic:
  LOWER(firstName + "." + lastName + IF(isDuplicate, "2", "") + "@company.com")
  
If duplicate detected:
  john.smith2@company.com
```

### **Example 3: Username (uid)**

**Requirement:** Generate username from name, lowercase, no special chars

**Configuration:**
```
Attribute: uid
Type: Complex
Transform: LOWER(firstName + "." + lastName)

Test Cases:
  firstName="John", lastName="Smith" → "john.smith"
  firstName="Mary", lastName="Johnson" → "mary.johnson"
```

**Handle special characters:**
```
Transform: REPLACE(REPLACE(LOWER(firstName + "." + lastName), " ", ""), "'", "")

Removes spaces and apostrophes:
  firstName="Mary Ann", lastName="O'Connor"
  → mary.ann.oconnor (spaces and apostrophe removed)
```

### **Example 4: Initial-Based Username**

**Requirement:** First initial + last name

**Configuration:**
```
Attribute: uid
Type: Complex
Transform: LOWER(SUBSTRING(firstName, 0, 1) + lastName)

Test Cases:
  firstName="John", lastName="Smith" → "jsmith"
  firstName="Mary", lastName="Johnson" → "mjohnson"
```

### **Example 5: Department with Default**

**Requirement:** Use department or "Unassigned" if empty

**Configuration:**
```
Attribute: department
Type: Complex
Transform: IF(department != null AND department != "", department, "Unassigned")

Test Cases:
  department="IT" → "IT"
  department="" → "Unassigned"
  department=null → "Unassigned"
```

---

## Complex Mapping Best Practices

### **Testing Transforms**

**Always test with edge cases:**
```
Test Cases to Consider:

Names with special characters:
  - O'Connor (apostrophe)
  - Mary-Ann (hyphen)
  - José (accent)
  - Van Der Berg (multiple words)

Names with spaces:
  - "Mary Ann Smith"
  - "De La Cruz"

Empty/null values:
  - firstName = null
  - lastName = ""

Very long names:
  - "Wolfeschlegelsteinhausenbergerdorff"

Duplicate names:
  - Two John Smiths
```

**Example test results:**
```
Transform: LOWER(firstName + "." + lastName + "@company.com")

Good cases:
  ✓ "John Smith" → "john.smith@company.com"
  ✓ "Mary Johnson" → "mary.johnson@company.com"

Edge cases to handle:
  ✗ "Mary Ann Smith" → "mary ann.smith@company.com" (space in email!)
  ✗ "John O'Connor" → "john.o'connor@company.com" (apostrophe acceptable)
  
Fix for spaces:
  REPLACE(LOWER(firstName + "." + lastName), " ", "") + "@company.com"
  ✓ "Mary Ann Smith" → "maryann.smith@company.com"
```

### **Common Mistakes**

**Mistake 1: Not handling nulls**
```
❌ Wrong:
  Transform: firstName + "." + lastName + "@company.com"
  
  If firstName = null:
    Result: "null.Smith@company.com"

✓ Correct:
  Transform: IF(firstName != null AND lastName != null, 
              LOWER(firstName + "." + lastName + "@company.com"), 
              "")
```

**Mistake 2: Not testing in production-like data**
```
❌ Wrong:
  Test only with: "John Smith"
  Deploy to production
  Fails with: "Mary-Ann O'Connor"

✓ Correct:
  Test with diverse names
  Include special characters
  Test edge cases before deployment
```

**Mistake 3: Overly complex transforms**
```
❌ Wrong:
  Transform with 10+ nested IFs and complex logic
  Hard to read, harder to debug

✓ Correct:
  Keep transforms simple
  If too complex, use Rule instead
  Break into multiple steps if needed
```

---

# Rule Mapping

## What is Rule Mapping?

**Definition:** Uses conditional logic (if/then) to map values based on rules

**Characteristics:**
```
✓ Conditional logic (if/then/else)
✓ Complex decision trees
✓ Reusable across attributes
✓ More powerful than complex transforms
```

**When to use:**
```
Use rules when:
  - Need complex conditional logic
  - Translating codes to values (IT → Information Technology)
  - Multiple conditions to evaluate
  - Logic too complex for transforms
  - Want reusable logic
  
Examples:
  ✓ Department code translation
  ✓ Employee type determination
  ✓ Region mapping
  ✓ Access level calculation
```

---

## Rule Configuration

### **Basic Structure**

**Rule example:**
```
Rule Name: DepartmentCodeToName

Logic:
  If source.deptCode == "IT"
    Return "Information Technology"
  Else If source.deptCode == "HR"
    Return "Human Resources"
  Else If source.deptCode == "FIN"
    Return "Finance"
  Else If source.deptCode == "ENG"
    Return "Engineering"
  Else
    Return source.deptCode (use code as-is)
```

**Applying rule to attribute:**
```
Identity Attribute: department
Mapping Type: Rule
Rule: DepartmentCodeToName
```

---

## Rule Examples

### **Example 1: Department Translation**

**Scenario:** Source has department codes, need full names

**Rule:**
```
Rule Name: DepartmentCodeMapping

Input: source.deptCode

Logic:
  Switch (source.deptCode):
    Case "IT":
      Return "Information Technology"
    Case "HR":
      Return "Human Resources"
    Case "FIN":
      Return "Finance"
    Case "MKT":
      Return "Marketing"
    Case "ENG":
      Return "Engineering"
    Case "OPS":
      Return "Operations"
    Default:
      Return "Other"

Test Cases:
  deptCode="IT" → "Information Technology"
  deptCode="HR" → "Human Resources"
  deptCode="XYZ" → "Other"
```

**Mapping configuration:**
```
Attribute: department
Type: Rule
Rule: DepartmentCodeMapping
```

### **Example 2: Employee Type Determination**

**Scenario:** Determine employee type from multiple source attributes

**Rule:**
```
Rule Name: EmployeeTypeRule

Inputs: 
  - source.workerType
  - source.contractEndDate

Logic:
  If source.workerType == "FTE":
    Return "Full-Time Employee"
  Else If source.workerType == "PTE":
    Return "Part-Time Employee"
  Else If source.workerType == "CTR" AND source.contractEndDate != null:
    Return "Contractor"
  Else If source.workerType == "INT":
    Return "Intern"
  Else:
    Return "Other"

Test Cases:
  workerType="FTE" → "Full-Time Employee"
  workerType="CTR", contractEndDate="2024-12-31" → "Contractor"
  workerType="INT" → "Intern"
```

### **Example 3: Location Code to Office Name**

**Scenario:** Map location codes to full office names

**Rule:**
```
Rule Name: LocationMapping

Input: source.locationCode

Logic:
  Switch (source.locationCode):
    Case "NYC":
      Return "New York Office"
    Case "SF":
      Return "San Francisco Office"
    Case "LON":
      Return "London Office"
    Case "TKY":
      Return "Tokyo Office"
    Case "SYD":
      Return "Sydney Office"
    Default:
      Return "Remote"

Test Cases:
  locationCode="NYC" → "New York Office"
  locationCode="SF" → "San Francisco Office"
  locationCode=null → "Remote"
```

### **Example 4: Access Level Based on Title**

**Scenario:** Determine access level from job title

**Rule:**
```
Rule Name: AccessLevelRule

Input: source.title

Logic:
  If source.title contains "CEO" OR source.title contains "President":
    Return "Executive"
  Else If source.title contains "VP" OR source.title contains "Vice President":
    Return "Executive"
  Else If source.title contains "Director":
    Return "Management"
  Else If source.title contains "Manager":
    Return "Management"
  Else If source.title contains "Senior":
    Return "Senior"
  Else:
    Return "Standard"

Test Cases:
  title="CEO" → "Executive"
  title="VP of Sales" → "Executive"
  title="Engineering Manager" → "Management"
  title="Senior Developer" → "Senior"
  title="Developer" → "Standard"
```

---

## Rules vs Complex Transforms

### **When to Use Which**

**Use Complex Transform when:**
```
✓ Simple string manipulation
✓ Combining fields
✓ Formatting values
✓ Single-step logic

Examples:
  - firstName + lastName
  - LOWER(email)
  - SUBSTRING operations
```

**Use Rule when:**
```
✓ Multiple conditions (if/then/else)
✓ Code-to-value translation
✓ Complex decision trees
✓ Reusable logic across multiple attributes

Examples:
  - Department code → Full name
  - Multiple inputs → Single output
  - Complex eligibility determination
```

**Comparison:**

| Scenario | Complex Transform | Rule |
|----------|-------------------|------|
| **Combine name** | ✓ Good fit | ❌ Overkill |
| **Generate email** | ✓ Good fit | ❌ Overkill |
| **Translate dept code** | ❌ Gets messy | ✓ Perfect |
| **Multi-condition logic** | ❌ Hard to read | ✓ Perfect |
| **Format date** | ✓ Good fit | ❌ Overkill |

---

## Practice Exercise

**Question:** You need to map these attributes from Workday: (1) firstName + lastName → displayName, (2) deptCode (IT, HR, FIN) → department (full names), (3) email (already correct format). What mapping type for each?

<details>
<summary>Answer</summary>

**Attribute 1: displayName (firstName + lastName)**

**Mapping Type: Complex (Transform)**
```
Configuration:
  Attribute: displayName
  Type: Complex
  Transform: firstName + " " + lastName

Reasoning:
  - Need to combine two fields
  - Simple string concatenation
  - No conditional logic needed
  - Perfect for complex transform

Example:
  Input:
    firstName = "John"
    lastName = "Smith"
  Transform:
    firstName + " " + lastName
  Output:
    displayName = "John Smith"

Why NOT Static:
  ❌ Static can't combine fields
  
Why NOT Rule:
  ❌ Overkill for simple concatenation
  ❌ Rules for conditional logic, not simple joins
```

**Attribute 2: department (deptCode translation)**

**Mapping Type: Rule**
```
Configuration:
  Attribute: department
  Type: Rule
  Rule: DepartmentCodeMapping

Rule Logic:
  If source.deptCode == "IT":
    Return "Information Technology"
  Else If source.deptCode == "HR":
    Return "Human Resources"
  Else If source.deptCode == "FIN":
    Return "Finance"
  Else:
    Return source.deptCode (unknown code, use as-is)

Reasoning:
  - Conditional logic (if/then)
  - Code-to-name translation
  - Multiple conditions
  - Perfect for rule mapping

Example:
  Input:
    deptCode = "IT"
  Rule evaluates:
    "IT" matches first condition
  Output:
    department = "Information Technology"

Test Cases:
  deptCode="IT" → "Information Technology"
  deptCode="HR" → "Human Resources"
  deptCode="FIN" → "Finance"
  deptCode="XYZ" → "XYZ" (unknown, use code)

Why NOT Static:
  ❌ Static just copies value
  ❌ Would give department = "IT" (code, not name)
  
Why NOT Complex:
  ⚠ Could work with nested IFs but messy:
    IF(deptCode=="IT", "Information Technology", 
       IF(deptCode=="HR", "Human Resources",
          IF(deptCode=="FIN", "Finance", deptCode)))
  ❌ Hard to read
  ❌ Hard to maintain
  ✓ Rule is cleaner and more maintainable
```

**Attribute 3: email (already correct format)**

**Mapping Type: Static**
```
Configuration:
  Attribute: email
  Type: Static
  Source: workday.email

Reasoning:
  - Email already in correct format
  - No transformation needed
  - No logic needed
  - Direct copy is perfect

Example:
  Input:
    workday.email = "john.smith@company.com"
  Static mapping (direct copy):
  Output:
    identity.email = "john.smith@company.com"

Why NOT Complex:
  ❌ No transform needed
  ❌ Email already correct format
  ❌ Don't add complexity when not needed
  
Why NOT Rule:
  ❌ No conditional logic needed
  ❌ Overkill for direct copy
```

**Summary Table:**

| Attribute | Source | Type | Configuration | Reasoning |
|-----------|--------|------|---------------|-----------|
| **displayName** | firstName + lastName | **Complex** | firstName + " " + lastName | Combine two fields |
| **department** | deptCode | **Rule** | DepartmentCodeMapping rule | Translate code to name |
| **email** | workday.email | **Static** | Direct copy | Already correct format |

**Key Decision Points:**
```
Ask yourself:

1. Is it a direct copy with no changes?
   → Static mapping

2. Does it need simple string manipulation (combine, format)?
   → Complex transform

3. Does it need conditional logic (if code = X then Y)?
   → Rule mapping

For this exercise:
  ✓ displayName: Combine fields → Complex
  ✓ department: Translate codes → Rule
  ✓ email: Direct copy → Static
```

**Additional Considerations:**

**If email needed to be generated (alternative scenario):**
```
Scenario: Workday doesn't have email field

Then:
  Attribute: email
  Type: Complex
  Transform: LOWER(firstName + "." + lastName + "@company.com")
  
Example:
  firstName="John", lastName="Smith"
  → email = "john.smith@company.com"
  
This would be Complex (transform), not Static
```

**If department codes were more complex (alternative scenario):**
Scenario: Department depends on both code AND region
Rule Logic:
If deptCode=="IT" AND region=="US":
Return "US Information Technology"
Else If deptCode=="IT" AND region=="EU":
Return "EU Information Technology"
... etc
Even more appropriate for Rule mapping
Multiple inputs, complex logic

**Exam Tip:** 
- Direct copy = Static
- String manipulation = Complex
- Conditional logic/translation = Rule
</details>

---

## Section Summary (4.2.2)

**Core Concepts:**

✅ **Three Mapping Types:**
- **Static**: Direct copy, no changes (firstName ← source.firstName)
- **Complex**: Transform/manipulate (email ← firstName + "@domain.com")
- **Rule**: Conditional logic (if dept="IT" → "Information Technology")

✅ **When to Use Each:**
- **Static**: Source matches ISC format exactly
- **Complex**: Combine/format/calculate values
- **Rule**: Translate codes, conditional logic

✅ **Common Complex Transforms:**
- Concatenation: firstName + " " + lastName
- Case conversion: LOWER(), UPPER()
- Substring: SUBSTRING(text, start, length)
- Conditional: IF(condition, trueValue, falseValue)

✅ **Rule Use Cases:**
- Department code → Full department name
- Employee type determination
- Location code → Office name
- Multi-condition logic

**Exam Focus:**
- Know when to use static vs complex vs rule
- Understand basic transform syntax
- Remember: static for copy, complex for manipulation, rule for conditions

---
