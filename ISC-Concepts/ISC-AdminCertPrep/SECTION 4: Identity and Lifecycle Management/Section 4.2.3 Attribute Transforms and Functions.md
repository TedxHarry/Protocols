# 4.2.3 Attribute Transforms and Functions

**Learning Objective:** Master common transform functions and their practical applications in identity attribute mappings.

---

## Transform Functions Overview

### **Categories of Functions**

| Category | Functions | Use Cases |
|----------|-----------|-----------|
| **String** | UPPER, LOWER, SUBSTRING, REPLACE | Format names, clean data |
| **Conditional** | IF, ISNULL, ISBLANK | Handle nulls, defaults |
| **Date** | DATEFORMAT, NOW, DATEADD | Format dates, calculate dates |
| **Logical** | AND, OR, NOT | Complex conditions |
| **Utility** | CONCATENATE, TRIM, LENGTH | String operations |

**Exam Tip:** Know the most common functions - LOWER, UPPER, IF, SUBSTRING, DATEFORMAT.

---

## String Functions

### **LOWER - Convert to Lowercase**

**Syntax:**
```
LOWER(string)
```

**Use cases:**
```
Email generation (emails should be lowercase):
  LOWER(firstName + "." + lastName + "@company.com")
  
Username generation:
  LOWER(firstName + lastName)
  
Standardizing data:
  LOWER(department) for consistent searches
```

**Examples:**
```
LOWER("JOHN SMITH")
→ "john smith"

LOWER("John.Smith@Company.COM")
→ "john.smith@company.com"

LOWER(firstName + "." + lastName + "@company.com")
  firstName = "John", lastName = "Smith"
→ "john.smith@company.com"
```

---

### **UPPER - Convert to Uppercase**

**Syntax:**
```
UPPER(string)
```

**Use cases:**
```
System requirements (some systems require uppercase usernames):
  UPPER(username)
  
Department codes:
  UPPER(deptCode)
  
Standardizing codes:
  UPPER(countryCode)
```

**Examples:**
```
UPPER("john smith")
→ "JOHN SMITH"

UPPER(lastName)
  lastName = "Smith"
→ "SMITH"

UPPER(SUBSTRING(firstName, 0, 1) + SUBSTRING(lastName, 0, 1))
  firstName = "John", lastName = "Smith"
→ "JS"
```

---

### **SUBSTRING - Extract Part of String**

**Syntax:**
```
SUBSTRING(string, startIndex, length)
```

**Parameters:**
- `string`: Text to extract from
- `startIndex`: Starting position (0-based)
- `length`: Number of characters to extract

**Use cases:**
```
Extract first initial:
  SUBSTRING(firstName, 0, 1)
  
Extract first N characters:
  SUBSTRING(employeeId, 0, 5)
  
Create initials:
  SUBSTRING(firstName, 0, 1) + SUBSTRING(lastName, 0, 1)
```

**Examples:**
```
SUBSTRING("John Smith", 0, 4)
→ "John"

SUBSTRING("John Smith", 5, 5)
→ "Smith"

SUBSTRING(firstName, 0, 1)
  firstName = "John"
→ "J"

SUBSTRING(firstName, 0, 1) + SUBSTRING(lastName, 0, 1)
  firstName = "John", lastName = "Smith"
→ "JS"

Generate username (first initial + last name):
  LOWER(SUBSTRING(firstName, 0, 1) + lastName)
  firstName = "John", lastName = "Smith"
→ "jsmith"
```

---

### **REPLACE - Replace Characters**

**Syntax:**
```
REPLACE(string, searchFor, replaceWith)
```

**Use cases:**
```
Remove spaces:
  REPLACE(fullName, " ", "")
  
Remove apostrophes:
  REPLACE(lastName, "'", "")
  
Replace characters in username:
  REPLACE(REPLACE(username, " ", ""), "'", "")
```

**Examples:**
```
REPLACE("John Smith", " ", "")
→ "JohnSmith"

REPLACE("O'Connor", "'", "")
→ "OConnor"

REPLACE(firstName, "-", "")
  firstName = "Mary-Ann"
→ "MaryAnn"

Multiple replacements:
  REPLACE(REPLACE(name, " ", ""), "'", "")
  name = "Mary Ann O'Connor"
→ "MaryAnnOConnor"

Clean username:
  LOWER(REPLACE(REPLACE(firstName + lastName, " ", ""), "'", ""))
  firstName = "Mary Ann", lastName = "O'Connor"
→ "maryannoconnor"
```

---

### **TRIM - Remove Whitespace**

**Syntax:**
```
TRIM(string)
```

**Use cases:**
```
Clean data from source:
  TRIM(firstName)
  
Remove leading/trailing spaces:
  TRIM(department)
```

**Examples:**
```
TRIM("  John Smith  ")
→ "John Smith"

TRIM("John  ")
→ "John"

TRIM(firstName)
  firstName = "  John  "
→ "John"
```

---

### **CONCATENATE - Join Strings**

**Syntax:**
```
CONCATENATE(string1, string2, ...)
```

**Note:** Usually use `+` operator instead

**Examples:**
```
CONCATENATE(firstName, " ", lastName)
→ Same as: firstName + " " + lastName

firstName + " " + lastName
  firstName = "John", lastName = "Smith"
→ "John Smith"

firstName + "." + lastName + "@company.com"
  firstName = "John", lastName = "Smith"
→ "John.Smith@company.com"
```

---

### **LENGTH - String Length**

**Syntax:**
```
LENGTH(string)
```

**Use cases:**
```
Validate data:
  IF(LENGTH(password) >= 8, "Valid", "Too short")
  
Check if value exists:
  IF(LENGTH(middleName) > 0, middleName, "")
```

**Examples:**
```
LENGTH("John Smith")
→ 10

LENGTH(firstName)
  firstName = "John"
→ 4

IF(LENGTH(firstName) > 0, firstName, "Unknown")
  firstName = "John"
→ "John"
```

---

## Conditional Functions

### **IF - Conditional Logic**

**Syntax:**
```
IF(condition, trueValue, falseValue)
```

**Use cases:**
```
Provide defaults:
  IF(department != null, department, "Unassigned")
  
Handle empty values:
  IF(middleName != "", firstName + " " + middleName + " " + lastName, firstName + " " + lastName)
  
Conditional formatting:
  IF(employeeType == "FTE", "Full-Time", "Part-Time")
```

**Examples:**
```
IF(department != null, department, "Unassigned")
  department = "IT"
→ "IT"

  department = null
→ "Unassigned"

IF(employeeType == "FTE", "Employee", "Contractor")
  employeeType = "FTE"
→ "Employee"

  employeeType = "CTR"
→ "Contractor"

Nested IF:
  IF(status == "Active", "active", IF(status == "Terminated", "inactive", "unknown"))
  status = "Active"
→ "active"

  status = "Terminated"
→ "inactive"

  status = "Leave"
→ "unknown"
```

---

### **ISNULL - Check for Null**

**Syntax:**
```
ISNULL(value, defaultValue)
```

**Use cases:**
```
Provide default for null values:
  ISNULL(department, "Unassigned")
  
Handle missing data:
  ISNULL(middleName, "")
```

**Examples:**
```
ISNULL(department, "Unassigned")
  department = "IT"
→ "IT"

  department = null
→ "Unassigned"

ISNULL(middleName, "")
  middleName = "Alexander"
→ "Alexander"

  middleName = null
→ ""
```

---

### **ISBLANK - Check for Empty String**

**Syntax:**
```
ISBLANK(value, defaultValue)
```

**Difference from ISNULL:**
```
ISNULL: Checks for null
ISBLANK: Checks for null OR empty string ""

ISBLANK catches both:
  - value = null
  - value = ""
```

**Examples:**
```
ISBLANK(department, "Unassigned")
  department = "IT"
→ "IT"

  department = ""
→ "Unassigned"

  department = null
→ "Unassigned"
```

---

## Date Functions

### **DATEFORMAT - Format Dates**

**Syntax:**
```
DATEFORMAT(date, "format")
```

**Format patterns:**
```
yyyy - 4-digit year (2024)
MM - 2-digit month (01-12)
dd - 2-digit day (01-31)

Common formats:
  "yyyy-MM-dd" → 2024-02-13 (ISO format)
  "MM/dd/yyyy" → 02/13/2024 (US format)
  "dd/MM/yyyy" → 13/02/2024 (EU format)
```

**Use cases:**
```
Standardize date format:
  DATEFORMAT(hireDate, "yyyy-MM-dd")
  
Convert between formats:
  DATEFORMAT(sourceDate, "MM/dd/yyyy")
```

**Examples:**
```
DATEFORMAT(hireDate, "yyyy-MM-dd")
  hireDate = "01/15/2023"
→ "2023-01-15"

DATEFORMAT(hireDate, "MM/dd/yyyy")
  hireDate = "2023-01-15"
→ "01/15/2023"

DATEFORMAT(NOW(), "yyyy-MM-dd")
→ "2024-02-13" (today's date in ISO format)
```

---

### **NOW - Current Date/Time**

**Syntax:**
```
NOW()
```

**Use cases:**
```
Set current date:
  NOW()
  
Calculate based on today:
  IF(endDate < NOW(), "inactive", "active")
```

**Examples:**
```
NOW()
→ 2024-02-13T14:30:00Z (current timestamp)

DATEFORMAT(NOW(), "yyyy-MM-dd")
→ "2024-02-13" (today's date)

IF(endDate < NOW(), "Expired", "Active")
  endDate = "2024-01-01"
→ "Expired" (date is in the past)
```

---

### **DATEADD - Add/Subtract Days**

**Syntax:**
```
DATEADD(date, days)
```

**Use cases:**
```
Calculate future date:
  DATEADD(NOW(), 90) (90 days from now)
  
Calculate expiration:
  DATEADD(startDate, 365) (1 year from start)
```

**Examples:**
```
DATEADD(NOW(), 90)
  Current date: 2024-02-13
→ 2024-05-13 (90 days later)

DATEADD(startDate, 365)
  startDate = "2024-01-01"
→ "2025-01-01" (1 year later)

DATEADD(NOW(), -30)
→ 30 days ago
```

---

## Logical Functions

### **AND - Logical AND**

**Syntax:**
```
condition1 AND condition2
```

**Use cases:**
```
Multiple conditions must be true:
  IF(employeeType == "FTE" AND department == "IT", "IT Employee", "Other")
  
Validation:
  firstName != null AND lastName != null
```

**Examples:**
```
IF(employeeType == "FTE" AND status == "Active", "Active FTE", "Other")
  employeeType = "FTE", status = "Active"
→ "Active FTE"

  employeeType = "FTE", status = "Terminated"
→ "Other"

  employeeType = "CTR", status = "Active"
→ "Other"
```

---

### **OR - Logical OR**

**Syntax:**
```
condition1 OR condition2
```

**Use cases:**
```
Any condition can be true:
  IF(status == "Active" OR status == "Leave", "active", "inactive")
  
Multiple acceptable values:
  title == "VP" OR title == "SVP" OR title == "EVP"
```

**Examples:**
```
IF(status == "Active" OR status == "Leave", "active", "inactive")
  status = "Active"
→ "active"

  status = "Leave"
→ "active"

  status = "Terminated"
→ "inactive"
```

---

### **NOT - Logical NOT**

**Syntax:**
```
NOT condition
```

**Use cases:**
```
Negate condition:
  NOT(status == "Terminated")
  
Check for non-empty:
  NOT(department == "")
```

**Examples:**
```
IF(NOT(status == "Terminated"), "active", "inactive")
  status = "Active"
→ "active"

  status = "Terminated"
→ "inactive"

Same as:
  IF(status != "Terminated", "active", "inactive")
```

---

## Practical Transform Examples

### **Example 1: Email Generation**

**Requirements:**
- Generate from firstName and lastName
- Lowercase
- Format: firstname.lastname@company.com
- Handle special characters

**Transform:**
```
LOWER(
  REPLACE(
    REPLACE(firstName + "." + lastName, " ", ""),
    "'",
    ""
  ) + "@company.com"
)
```

**Test cases:**
```
firstName="John", lastName="Smith"
→ "john.smith@company.com"

firstName="Mary Ann", lastName="O'Connor"
→ "maryann.oconnor@company.com"
  (spaces and apostrophes removed)

firstName="José", lastName="García"
→ "josé.garcía@company.com"
  (accents preserved - depends on system)
```

---

### **Example 2: Username Generation**

**Requirements:**
- First initial + last name
- Lowercase
- No special characters

**Transform:**
```
LOWER(
  REPLACE(
    REPLACE(
      SUBSTRING(firstName, 0, 1) + lastName,
      " ",
      ""
    ),
    "'",
    ""
  )
)
```

**Test cases:**
```
firstName="John", lastName="Smith"
→ "jsmith"

firstName="Mary", lastName="O'Connor"
→ "moconnor"

firstName="José", lastName="García"
→ "jgarcía" or "jgarcia" (depending on system handling)
```

---

### **Example 3: Display Name with Middle Initial**

**Requirements:**
- Format: FirstName MiddleInitial. LastName
- Only include middle initial if present
- Example: "John A. Smith" or "John Smith"

**Transform:**
```
firstName + IF(LENGTH(middleName) > 0, " " + SUBSTRING(middleName, 0, 1) + ".", "") + " " + lastName
```

**Test cases:**
```
firstName="John", middleName="Alexander", lastName="Smith"
→ "John A. Smith"

firstName="John", middleName="", lastName="Smith"
→ "John Smith"

firstName="John", middleName=null, lastName="Smith"
→ "John Smith"
```

---

### **Example 4: Lifecycle from Date**

**Requirements:**
- If termination date in past: inactive
- If termination date in future: active
- If no termination date: active

**Transform:**
```
IF(
  terminationDate != null AND terminationDate < NOW(),
  "inactive",
  "active"
)
```

**Test cases:**
```
terminationDate = "2024-01-01" (past)
→ "inactive"

terminationDate = "2024-12-31" (future)
→ "active"

terminationDate = null
→ "active"
```

---

### **Example 5: Department with Fallback**

**Requirements:**
- Use department from source
- If empty or null, use "Unassigned"
- If "TEMP", use "Temporary Assignment"

**Transform:**
```
IF(
  ISBLANK(department),
  "Unassigned",
  IF(department == "TEMP", "Temporary Assignment", department)
)
```

**Test cases:**
```
department = "IT"
→ "IT"

department = ""
→ "Unassigned"

department = null
→ "Unassigned"

department = "TEMP"
→ "Temporary Assignment"
```

---

### **Example 6: Formatted Phone Number**

**Requirements:**
- Source has: 5551234567
- Need: +1-555-123-4567

**Transform:**
```
"+1-" + SUBSTRING(phone, 0, 3) + "-" + SUBSTRING(phone, 3, 3) + "-" + SUBSTRING(phone, 6, 4)
```

**Test case:**
```
phone = "5551234567"
→ "+1-555-123-4567"
```

---

## Transform Best Practices

### **1. Handle Nulls**
```
❌ Bad:
  firstName + " " + lastName

  If firstName = null:
    Result: "null Smith"

✓ Good:
  IF(firstName != null AND lastName != null,
     firstName + " " + lastName,
     "")
```

### **2. Test Edge Cases**
```
Always test:
  - Null values
  - Empty strings
  - Special characters (apostrophes, hyphens)
  - Very long strings
  - Unexpected formats
```

### **3. Keep Transforms Readable**
```
❌ Bad (unreadable):
  LOWER(REPLACE(REPLACE(IF(ISBLANK(firstName),"",SUBSTRING(firstName,0,1))+IF(ISBLANK(lastName),"",lastName)," ",""),"'",""))

✓ Good (broken into steps):
  Step 1: Combine initial + last name
    initial = SUBSTRING(firstName, 0, 1)
    username = initial + lastName
    
  Step 2: Remove special chars
    cleaned = REPLACE(REPLACE(username, " ", ""), "'", "")
    
  Step 3: Lowercase
    result = LOWER(cleaned)
    
  Or use comments in transform if supported
```

### **4. Validate Results**
```
After creating transform:
  1. Test with sample data
  2. Test edge cases
  3. Deploy to sandbox first
  4. Verify results
  5. Then deploy to production
```

---

## Common Transform Patterns

### **Pattern 1: Email from Name**
```
Standard email:
  LOWER(firstName + "." + lastName + "@company.com")

With special char handling:
  LOWER(REPLACE(REPLACE(firstName + "." + lastName, " ", ""), "'", "") + "@company.com")
```

### **Pattern 2: Username Variations**
```
First initial + last name:
  LOWER(SUBSTRING(firstName, 0, 1) + lastName)
  → "jsmith"

First name + last initial:
  LOWER(firstName + SUBSTRING(lastName, 0, 1))
  → "johns"

First + Last (no dot):
  LOWER(firstName + lastName)
  → "johnsmith"
```

### **Pattern 3: Conditional Defaults**
```
With ISNULL:
  ISNULL(department, "Unassigned")

With IF:
  IF(department != null AND department != "", department, "Unassigned")

With ISBLANK:
  ISBLANK(department, "Unassigned")
```

### **Pattern 4: Date Standardization**
```
Convert to ISO format:
  DATEFORMAT(sourceDate, "yyyy-MM-dd")

Extract year:
  SUBSTRING(DATEFORMAT(date, "yyyy-MM-dd"), 0, 4)
```

---

## Practice Exercise

**Question:** Create a transform to generate email addresses with these requirements: lowercase, format firstname.lastname@company.com, remove spaces and apostrophes from names, handle null values gracefully.

<details>
<summary>Answer</summary>

**Complete Transform:**
```
LOWER(
  REPLACE(
    REPLACE(
      IF(firstName != null AND lastName != null,
         firstName + "." + lastName,
         "unknown"),
      " ",
      ""
    ),
    "'",
    ""
  ) + "@company.com"
)
```

**Step-by-Step Breakdown:**

**Step 1: Check for null values**
```
IF(firstName != null AND lastName != null,
   firstName + "." + lastName,
   "unknown")

Purpose:
  - Ensures both names exist
  - Provides "unknown" default if either is null
  - Prevents "null.Smith@company.com"

Test cases:
  firstName="John", lastName="Smith"
  → "John.Smith"
  
  firstName=null, lastName="Smith"
  → "unknown"
  
  firstName="John", lastName=null
  → "unknown"
```

**Step 2: Remove spaces**
```
REPLACE(
  [result from step 1],
  " ",
  ""
)

Purpose:
  - Removes spaces from names
  - "Mary Ann" becomes "MaryAnn"

Test cases:
  "Mary Ann.Smith"
  → "MaryAnn.Smith"
  
  "John.Van Der Berg"
  → "John.VanDerBerg"
```

**Step 3: Remove apostrophes**
```
REPLACE(
  [result from step 2],
  "'",
  ""
)

Purpose:
  - Removes apostrophes
  - "O'Connor" becomes "OConnor"

Test cases:
  "John.O'Connor"
  → "John.OConnor"
  
  "Mary.D'Angelo"
  → "Mary.DAngelo"
```

**Step 4: Convert to lowercase**
```
LOWER([result from step 3])

Purpose:
  - Email addresses should be lowercase
  - Consistent format

Test cases:
  "John.Smith"
  → "john.smith"
  
  "Mary.OConnor"
  → "mary.oconnor"
```

**Step 5: Add domain**
```
[result from step 4] + "@company.com"

Purpose:
  - Complete email address

Final result:
  "john.smith@company.com"
```

**Complete Test Cases:**
```
Test Case 1: Standard name
  Input:
    firstName = "John"
    lastName = "Smith"
  Transform execution:
    → IF check: Both exist → "John.Smith"
    → Remove spaces: "John.Smith" (no spaces)
    → Remove apostrophes: "John.Smith" (no apostrophes)
    → Lowercase: "john.smith"
    → Add domain: "john.smith@company.com"
  Output:
    "john.smith@company.com"

Test Case 2: Name with spaces
  Input:
    firstName = "Mary Ann"
    lastName = "Smith"
  Transform execution:
    → IF check: Both exist → "Mary Ann.Smith"
    → Remove spaces: "MaryAnn.Smith"
    → Remove apostrophes: "MaryAnn.Smith" (no apostrophes)
    → Lowercase: "maryann.smith"
    → Add domain: "maryann.smith@company.com"
  Output:
    "maryann.smith@company.com"

Test Case 3: Name with apostrophe
  Input:
    firstName = "John"
    lastName = "O'Connor"
  Transform execution:
    → IF check: Both exist → "John.O'Connor"
    → Remove spaces: "John.O'Connor" (no spaces)
    → Remove apostrophes: "John.OConnor"
    → Lowercase: "john.oconnor"
    → Add domain: "john.oconnor@company.com"
  Output:
    "john.oconnor@company.com"

Test Case 4: Name with both spaces and apostrophes
  Input:
    firstName = "Mary Ann"
    lastName = "O'Connor"
  Transform execution:
    → IF check: Both exist → "Mary Ann.O'Connor"
    → Remove spaces: "MaryAnn.O'Connor"
    → Remove apostrophes: "MaryAnn.OConnor"
    → Lowercase: "maryann.oconnor"
    → Add domain: "maryann.oconnor@company.com"
  Output:
    "maryann.oconnor@company.com"

Test Case 5: Null firstName
  Input:
    firstName = null
    lastName = "Smith"
  Transform execution:
    → IF check: firstName is null → "unknown"
    → Remove spaces: "unknown" (no spaces)
    → Remove apostrophes: "unknown" (no apostrophes)
    → Lowercase: "unknown"
    → Add domain: "unknown@company.com"
  Output:
    "unknown@company.com"

Test Case 6: Both null
  Input:
    firstName = null
    lastName = null
  Transform execution:
    → IF check: Both null → "unknown"
    → Remove spaces: "unknown"
    → Remove apostrophes: "unknown"
    → Lowercase: "unknown"
    → Add domain: "unknown@company.com"
  Output:
    "unknown@company.com"

Test Case 7: Multi-word last name
  Input:
    firstName = "John"
    lastName = "Van Der Berg"
  Transform execution:
    → IF check: Both exist → "John.Van Der Berg"
    → Remove spaces: "John.VanDerBerg"
    → Remove apostrophes: "John.VanDerBerg" (no apostrophes)
    → Lowercase: "john.vanderberg"
    → Add domain: "john.vanderberg@company.com"
  Output:
    "john.vanderberg@company.com"
```

**Alternative: Simplified Version (If Nulls Handled Elsewhere)**
```
If identity profile already ensures firstName and lastName exist:

Simplified Transform:
  LOWER(
    REPLACE(
      REPLACE(firstName + "." + lastName, " ", ""),
      "'",
      ""
    ) + "@company.com"
  )

Pros:
  - Simpler
  - Easier to read
  - Fewer nested functions

Cons:
  - Assumes nulls handled elsewhere
  - May fail if assumption wrong
  
Recommendation:
  Use full version with null check for safety
```

**Production Implementation Checklist:**
```
Before deploying:
  ☐ Test with at least 10 different name variations
  ☐ Test null/empty cases
  ☐ Test special characters (apostrophes, hyphens, accents)
  ☐ Test multi-word names
  ☐ Test very long names
  ☐ Verify lowercase conversion working
  ☐ Verify no spaces in output
  ☐ Deploy to sandbox first
  ☐ Verify first aggregation results
  ☐ Then deploy to production
```

**Exam Tip:** When building transforms, work inside-out: handle nulls first, then manipulate strings, then format/case conversion, finally add static parts like domain.
</details>

---

## Section Summary

**Core Concepts for Certification:**

✅ **String Functions:**
- **LOWER**: Convert to lowercase (emails, usernames)
- **UPPER**: Convert to uppercase (codes, standards)
- **SUBSTRING**: Extract part of string (initials, truncate)
- **REPLACE**: Replace characters (remove spaces, apostrophes)
- **TRIM**: Remove whitespace

✅ **Conditional Functions:**
- **IF**: Conditional logic (if/then/else)
- **ISNULL**: Handle null values (provide defaults)
- **ISBLANK**: Handle null or empty (more comprehensive)

✅ **Date Functions:**
- **DATEFORMAT**: Format dates (standardize to ISO)
- **NOW**: Current date/time
- **DATEADD**: Calculate future/past dates

✅ **Common Patterns:**
- Email generation: LOWER(firstName + "." + lastName + "@domain.com")
- Username: LOWER(SUBSTRING(firstName, 0, 1) + lastName)
- Display name: firstName + " " + lastName
- Handle nulls: IF(value != null, value, "default")
- Clean data: REPLACE(REPLACE(text, " ", ""), "'", "")

✅ **Best Practices:**
- Always handle nulls
- Test edge cases (special characters, long names)
- Keep transforms readable
- Test in sandbox before production
- Document complex transforms

**Exam Focus Areas:**
- Know common string functions (LOWER, UPPER, SUBSTRING, REPLACE)
- Understand IF for conditional logic
- Know date formatting with DATEFORMAT
- Recognize patterns (email generation, username creation)
- Remember to handle nulls and empty values

---

**End of Section 4.2: Identity Attributes**

**Completed:**
- ✅ 4.2.1: Identity Attribute Mappings
- ✅ 4.2.2: Mapping Configuration Options (Static, Complex, Rules)
- ✅ 4.2.3: Attribute Transforms and Functions

**Next:** Section 4.3: Authentication in Identity Profiles

---

**Study Tips:**
- Practice writing transforms for common scenarios
- Memorize key functions: LOWER, UPPER, IF, SUBSTRING
- Always test transforms with null and special character cases
- Remember the pattern: Check nulls → Manipulate → Format → Add static parts
- Know when to use static vs complex vs rule mappings
