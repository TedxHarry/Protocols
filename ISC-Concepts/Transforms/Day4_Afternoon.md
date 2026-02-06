# DAY 4: DATE/TIME & LIST OPERATIONS - AFTERNOON SESSION

## Table of Contents
- [Overview](#overview)
- [What You'll Learn Today](#what-youll-learn-today)
- [Understanding List/Array Operations](#understanding-listarrayoperations)
- [Transform 1: Split](#transform-1-split-25-minutes)
- [Transform 2: Index](#transform-2-index-25-minutes)
- [Transform 3: Join](#transform-3-join-20-minutes)
- [Combining List Transforms](#combining-list-transforms-30-minutes)
- [Advanced Pattern: Next Processing Date](#advanced-pattern-next-processing-date-30-minutes)
- [Day 4 Final Assessment](#day-4-final-assessment)
- [Homework Challenges](#homework-challenges)

---

## Overview

**Session Duration:** 2.5 hours

**Prerequisites:** 
- Completed Day 4 Morning Session
- Understand date transforms
- Comfortable with transform chaining
- Understand conditional logic

**Goal:** Master list/array operations and complete your transform toolkit

---

## What You'll Learn Today

By the end of this afternoon session, you'll be able to:

- ✅ Split strings into arrays
- ✅ Extract items from arrays by position
- ✅ Join arrays back into strings
- ✅ Parse complex string formats
- ✅ Work with multi-value attributes
- ✅ Extract email domains
- ✅ Parse names from various formats
- ✅ Build next processing date logic
- ✅ Handle AD group lists

---

## Understanding List/Array Operations

### What Are Multi-Value Attributes?

Some attributes contain **multiple values** instead of single values:
```
Single-Value Attribute:
  email = "john.smith@company.com"

Multi-Value Attribute:
  memberOf = [
    "CN=Admins,OU=Groups,DC=company,DC=com",
    "CN=Developers,OU=Groups,DC=company,DC=com",
    "CN=Engineering,OU=Groups,DC=company,DC=com"
  ]
```

### Common Multi-Value Attributes

| Source | Attribute | Example Values |
|--------|-----------|----------------|
| Active Directory | memberOf | AD group DNs |
| Active Directory | proxyAddresses | Email aliases |
| HR System | skills | ["Java", "Python", "SQL"] |
| Custom App | certifications | ["PMP", "CISSP", "AWS"] |

---

### The Three Core List Transforms
```
SPLIT: String --> Array
  "John,Smith,Engineer" --> ["John", "Smith", "Engineer"]

INDEX: Array --> Single Item
  ["John", "Smith", "Engineer"] --> "John" (position 0)

JOIN: Array --> String
  ["John", "Smith", "Engineer"] --> "John; Smith; Engineer"
```

---

## Transform 1: Split (25 minutes)

### What Split Does

**Split** breaks a string into an array based on a delimiter.

#### Basic Usage
```
INPUT: "John,Smith,Engineer"
TRANSFORM: Split
  Delimiter: ","
OUTPUT: ["John", "Smith", "Engineer"]
```

---

### Exercise 1A: Split Full Name

**Scenario:** Source provides "Lastname, Firstname" and you need to separate them.

#### Steps
```
1. Go to Identity Profile > Mappings
2. Create attribute "nameParts"
3. Transform: Split
4. Configuration:

   Input: full_name
   Delimiter: "," (comma)

5. Preview
6. Save
```

> **Note:** Verify Split transform exists in your ISC version and note exact configuration options.

---

#### Test Cases
```
Test Case 1:
  Input: full_name = "Smith, John"
  Delimiter: ","
  Output: ["Smith", " John"] (note space before John)

Test Case 2:
  Input: full_name = "O'Brien, Mary-Ann"
  Delimiter: ","
  Output: ["O'Brien", " Mary-Ann"]

Test Case 3:
  Input: full_name = "SingleName"
  Delimiter: ","
  Output: ["SingleName"] (no delimiter found, returns single-item array)

Test Case 4:
  Input: full_name = null
  Output: null
```

---

### Exercise 1B: Split Email Address

**Scenario:** Extract username and domain from email.

#### Steps
```
1. Create attribute "emailParts"
2. Transform: Split
3. Configuration:

   Input: email
   Delimiter: "@"

4. Preview
5. Save
```

---

#### Test Cases
```
Test Case 1:
  Input: email = "john.smith@company.com"
  Delimiter: "@"
  Output: ["john.smith", "company.com"]

Test Case 2:
  Input: email = "user@subdomain.company.com"
  Delimiter: "@"
  Output: ["user", "subdomain.company.com"]

Test Case 3:
  Input: email = "invalid-email-no-at-sign"
  Delimiter: "@"
  Output: ["invalid-email-no-at-sign"] (no @ found)
```

---

### Exercise 1C: Split AD Distinguished Name

**Scenario:** Parse AD group DN to extract group name.

#### Steps
```
1. Input: "CN=Admins,OU=Groups,DC=company,DC=com"
2. Create attribute "dnParts"
3. Transform: Split
4. Configuration:

   Input: ad_group_dn
   Delimiter: ","

5. Output: ["CN=Admins", "OU=Groups", "DC=company", "DC=com"]
6. Save
```

---

### Exercise 1D: Split with Multiple Delimiters

**Scenario:** Parse location string "Austin, TX - Building 2".

**Challenge:** Has both "," and "-" delimiters.

**Solution:** Split twice (chain splits):
```
Transform chain:
1. Split by ","
   Input: "Austin, TX - Building 2"
   Output: ["Austin", " TX - Building 2"]

2. Get second item, split by "-"
   Input: " TX - Building 2"
   Output: [" TX ", " Building 2"]
```

We'll build this complete chain in the Combining Transforms section.

---

### Key Learning Points

- ✅ Split converts string to array
- ✅ Delimiter can be any character(s)
- ✅ Returns array even if delimiter not found
- ✅ Preserves spaces (may need Trim after)
- ✅ Can chain multiple Splits
- ✅ Null input returns null

---

## Transform 2: Index (25 minutes)

### What Index Does

**Index** extracts a specific item from an array by position.

#### Basic Usage
```
INPUT: ["John", "Smith", "Engineer"]
TRANSFORM: Index
  Position: 0
OUTPUT: "John"
```

#### Array Positions Start at 0!
```
Array: ["John", "Smith", "Engineer"]
Index:    0       1         2

Position 0 = "John"
Position 1 = "Smith"
Position 2 = "Engineer"
Position 3 = Error (out of bounds)
```

---

### Exercise 2A: Extract Firstname from Split Name

**Scenario:** After splitting "Smith, John", get the firstname.

#### Steps
```
1. First, create the split (from Exercise 1A)
   Attribute: nameParts
   Transform: Split
     Input: full_name ("Smith, John")
     Delimiter: ","
     Output: ["Smith", " John"]

2. Create attribute "firstname"
3. Transform chain:

   Transform 1: Index
     Input: nameParts
     Position: 1
     Output: " John"
   
   Transform 2: Trim
     Input: " John"
     Output: "John"

4. Save
```

---

#### Test Cases
```
Test Case 1:
  full_name = "Smith, John"
  After Split: ["Smith", " John"]
  Index 1: " John"
  After Trim: "John"

Test Case 2:
  full_name = "O'Brien, Mary-Ann"
  After Split: ["O'Brien", " Mary-Ann"]
  Index 1: " Mary-Ann"
  After Trim: "Mary-Ann"
```

---

### Exercise 2B: Extract Lastname

**Scenario:** Get lastname from same split.
```
Attribute: lastname
Transform chain:
1. Index
   Input: nameParts
   Position: 0
   Output: "Smith"

2. Trim (just to be safe)
   Output: "Smith"
```

---

### Exercise 2C: Extract Email Domain

**Scenario:** Get domain from email address.

#### Steps
```
1. First, split email (from Exercise 1B)
   emailParts = ["john.smith", "company.com"]

2. Create attribute "emailDomain"
3. Transform: Index
   Input: emailParts
   Position: 1
   Output: "company.com"

4. Save
```

---

#### Test Cases
```
Test Case 1:
  email = "john.smith@company.com"
  After Split: ["john.smith", "company.com"]
  Index 1: "company.com"

Test Case 2:
  email = "user@mail.subdomain.company.com"
  After Split: ["user", "mail.subdomain.company.com"]
  Index 1: "mail.subdomain.company.com"
```

---

### Exercise 2D: Extract Username from Email

**Scenario:** Get username part before @.
```
Attribute: emailUsername
Transform: Index
  Input: emailParts (from split)
  Position: 0
  Output: "john.smith"
```

---

### Exercise 2E: Get First AD Group

**Scenario:** User has multiple AD groups, get the first one.

#### Steps
```
1. Get AD groups (multi-value attribute)
   Attribute: adGroups
   Transform: Account Attribute
     Source: Active Directory
     Attribute: memberOf
     Output: ["CN=Admins,...", "CN=Developers,...", ...]

2. Create attribute "primaryGroup"
3. Transform: Index
   Input: adGroups
   Position: 0
   Output: "CN=Admins,OU=Groups,DC=company,DC=com"

4. Save
```

---

### Exercise 2F: Handle Out of Bounds

**Scenario:** What happens if you try to access position that doesn't exist?
```
Array: ["John", "Smith"]
Index: 2

Result: null or error (depends on ISC version)
```

**Best Practice:** Always ensure array has enough items before accessing position.
```
Use Conditional:
  IF array_length >= 3
    THEN Index(2)
    ELSE null
```

> **Note:** Check your ISC version documentation for array_length or similar function availability.

---

### Key Learning Points

- ✅ Index extracts item by position
- ✅ Positions start at 0 (not 1!)
- ✅ Works on arrays from Split or multi-value attributes
- ✅ Out of bounds returns null or error
- ✅ Combine with Trim to clean extracted values
- ✅ Can chain multiple Index operations

---

## Transform 3: Join (20 minutes)

### What Join Does

**Join** combines array items into a single string with a delimiter.

#### Basic Usage
```
INPUT: ["John", "Smith", "Engineer"]
TRANSFORM: Join
  Delimiter: ", "
OUTPUT: "John, Smith, Engineer"
```

---

### Exercise 3A: Join Name Parts

**Scenario:** After splitting and processing, rejoin with different format.

#### Steps
```
1. Have array: ["John", "Smith"]
2. Create attribute "fullNameFormatted"
3. Transform: Join
4. Configuration:

   Input: namePartsArray
   Delimiter: " " (space)

5. Output: "John Smith"
6. Save
```

---

#### Test Cases
```
Test Case 1:
  Input: ["John", "Smith"]
  Delimiter: " "
  Output: "John Smith"

Test Case 2:
  Input: ["Mary-Ann", "O'Brien"]
  Delimiter: " "
  Output: "Mary-Ann O'Brien"

Test Case 3:
  Input: ["John", "Michael", "Smith"]
  Delimiter: " "
  Output: "John Michael Smith"
```

---

### Exercise 3B: Join AD Groups with Semicolon

**Scenario:** Display all AD groups as semicolon-separated string.

#### Steps
```
1. Input: adGroups = ["CN=Admins,...", "CN=Developers,...", "CN=Engineering,..."]

2. Create attribute "adGroupsString"
3. Transform: Join
   Input: adGroups
   Delimiter: "; "

4. Output: "CN=Admins,...; CN=Developers,...; CN=Engineering,..."
5. Save
```

---

#### Test Cases
```
Test Case 1:
  Input: ["CN=Admins,OU=Groups", "CN=Developers,OU=Groups"]
  Delimiter: "; "
  Output: "CN=Admins,OU=Groups; CN=Developers,OU=Groups"

Test Case 2:
  Input: ["Group1", "Group2", "Group3"]
  Delimiter: ", "
  Output: "Group1, Group2, Group3"

Test Case 3:
  Input: ["SingleGroup"]
  Delimiter: "; "
  Output: "SingleGroup" (no delimiter added for single item)
```

---

### Exercise 3C: Join Skills List

**Scenario:** Combine skills from multi-value attribute.
```
Input: skills = ["Java", "Python", "SQL", "AWS"]

Attribute: skillsSummary
Transform: Join
  Input: skills
  Delimiter: " | "

Output: "Java | Python | SQL | AWS"
```

---

### Key Learning Points

- ✅ Join converts array to string
- ✅ Delimiter inserted between items
- ✅ No delimiter for single-item array
- ✅ Useful for display purposes
- ✅ Can use any delimiter (", ", "; ", " | ", etc.)
- ✅ Opposite of Split

---

## Combining List Transforms (30 minutes)

### Pattern 1: Parse and Extract Group Name from AD DN

**Goal:** From "CN=Admins,OU=Groups,DC=company,DC=com" get "Admins".

#### Complete Transform Chain
```
Input: "CN=Admins,OU=Groups,DC=company,DC=com"

Transform 1: Split by ","
  Output: ["CN=Admins", "OU=Groups", "DC=company", "DC=com"]

Transform 2: Index (position 0)
  Output: "CN=Admins"

Transform 3: Split by "="
  Output: ["CN", "Admins"]

Transform 4: Index (position 1)
  Output: "Admins"

Final Output: "Admins"
```

---

#### Steps to Build
```
1. Create attribute "primaryGroupName"
2. Build transform chain:

   Step 1: Account Attribute
     Source: AD
     Attribute: memberOf
     Output: Array of DNs
   
   Step 2: Index
     Position: 0
     Output: "CN=Admins,OU=Groups,DC=company,DC=com"
   
   Step 3: Split
     Delimiter: ","
     Output: ["CN=Admins", "OU=Groups", "DC=company", "DC=com"]
   
   Step 4: Index
     Position: 0
     Output: "CN=Admins"
   
   Step 5: Split
     Delimiter: "="
     Output: ["CN", "Admins"]
   
   Step 6: Index
     Position: 1
     Output: "Admins"

3. Save
```

---

#### Test Cases
```
Test Case 1:
  Input DN: "CN=Admins,OU=Groups,DC=company,DC=com"
  Final Output: "Admins"

Test Case 2:
  Input DN: "CN=Domain Users,OU=Groups,DC=company,DC=com"
  Final Output: "Domain Users"

Test Case 3:
  Input DN: "CN=Engineering-Team,OU=Departments,DC=company,DC=com"
  Final Output: "Engineering-Team"
```

---

### Pattern 2: Parse Complex Location String

**Goal:** From "Austin, TX - Building 2" extract city, state, and building.

#### Complete Transform Chain
```
Input: "Austin, TX - Building 2"

Extract City:
  Transform 1: Split by ","
    Output: ["Austin", " TX - Building 2"]
  Transform 2: Index (position 0)
    Output: "Austin"
  Transform 3: Trim
    Output: "Austin"

Extract State:
  Transform 1: Split by ","
    Output: ["Austin", " TX - Building 2"]
  Transform 2: Index (position 1)
    Output: " TX - Building 2"
  Transform 3: Split by "-"
    Output: [" TX ", " Building 2"]
  Transform 4: Index (position 0)
    Output: " TX "
  Transform 5: Trim
    Output: "TX"

Extract Building:
  Transform 1: Split by "-"
    Output: ["Austin, TX ", " Building 2"]
  Transform 2: Index (position 1)
    Output: " Building 2"
  Transform 3: Trim
    Output: "Building 2"
```

---

### Pattern 3: Parse Email Username and Format

**Goal:** From "john.smith@company.com" create "SMITH, John".

#### Complete Transform Chain
```
Input: "john.smith@company.com"

Transform 1: Split by "@"
  Output: ["john.smith", "company.com"]

Transform 2: Index (position 0)
  Output: "john.smith"

Transform 3: Split by "."
  Output: ["john", "smith"]

Transform 4: Index (position 1)
  Output: "smith"

Transform 5: Upper
  Output: "SMITH"

Transform 6: Index (back to position 0 of previous split)
  Output: "john"

Transform 7: Upper + Substring (first letter only)
  Output: "J"

Transform 8: Concatenation
  SMITH + ", " + J + remainder of firstname
  Output: "SMITH, John"
```

> **Note:** This gets complex. May be easier to build separate attributes for each part then concatenate.

---

### Pattern 4: Get All Group Names (Not Just First)

**Scenario:** Extract clean group names from all AD groups.

**Challenge:** Can't easily loop through array in transforms.

**Solution:** This may require a rule or custom transform depending on ISC capabilities.

**Alternative:** Just get first group (which we did), or join them all and display as-is.

---

### Key Learning Points

- ✅ Split + Index + Split + Index = parse complex strings
- ✅ Build in stages - test each step
- ✅ Use helper attributes for intermediate results
- ✅ Trim is your friend after splits
- ✅ Some patterns easier with separate attributes
- ✅ Complex parsing may need rules (not just transforms)

---

## Advanced Pattern: Next Processing Date (30 minutes)

### Business Requirement

Calculate when identity should be processed next based on dates:
```
Logic:
- If startDate is in future, process on startDate at 6:00 AM
- Else if endDate is in future, process on endDate at 3:00 PM
- Else return null (no future processing needed)
```

This is a **common SailPoint pattern** for time-based identity processing.

---

### Step 1: Prepare Dates (15 min)

#### Convert Source Dates to ISO8601
```
1. Attribute: startDateISO
   Transform: Date Format
     Input: start_date
     Input Format: MM/dd/yyyy
     Output Format: ISO8601

2. Attribute: endDateISO
   Transform: Date Format
     Input: end_date
     Input Format: MM/dd/yyyy
     Output Format: ISO8601
```

---

### Step 2: Create Date Comparison Helpers (5 min)
```
1. Attribute: isStartDateFuture
   Transform: Date Compare
     First Date: startDateISO
     Comparison: Greater Than
     Second Date: [Today]
     Output: true/false

2. Attribute: isEndDateFuture
   Transform: Date Compare
     First Date: endDateISO
     Comparison: Greater Than
     Second Date: [Today]
     Output: true/false
```

---

### Step 3: Add Time to Dates (5 min)
```
1. Attribute: startDateWith6AM
   Transform: Date Math
     Input: startDateISO
     Operation: Add
     Amount: 6
     Unit: hours
     Output: "2024-06-01T06:00:00Z"

2. Attribute: endDateWith3PM
   Transform: Date Math
     Input: endDateISO
     Operation: Add
     Amount: 15
     Unit: hours
     Output: "2024-12-31T15:00:00Z"
```

---

### Step 4: Build Next Processing Date Logic (5 min)
```
Attribute: nextProcessing
Transform: Conditional chain

Level 1: Check start date
  IF startDateISO ne null AND isStartDateFuture eq true
    THEN startDateWith6AM
    ELSE Continue

Level 2: Check end date
  IF endDateISO ne null AND isEndDateFuture eq true
    THEN endDateWith3PM
    ELSE null
```

---

### Complete Test Cases
```
Assume today = 2024-02-06

Test Case 1: Future start date
  start_date = "06/01/2024"
  end_date = "12/31/2024"
  startDateISO = "2024-06-01T00:00:00Z"
  isStartDateFuture = true
  Result: "2024-06-01T06:00:00Z" (start date + 6am)

Test Case 2: Past start, future end
  start_date = "01/15/2024"
  end_date = "12/31/2024"
  isStartDateFuture = false
  isEndDateFuture = true
  Result: "2024-12-31T15:00:00Z" (end date + 3pm)

Test Case 3: Both dates in past
  start_date = "01/15/2023"
  end_date = "12/31/2023"
  isStartDateFuture = false
  isEndDateFuture = false
  Result: null (no future processing)

Test Case 4: No dates
  start_date = null
  end_date = null
  Result: null
```

---

### Why This Matters

The **nextProcessing** attribute is used by SailPoint's **time-based processing**:

- When this date/time arrives, identity processing runs for this identity
- Useful for:
  - Processing on hire date (grant access)
  - Processing on termination date (revoke access)
  - Processing on contract expiration
  - Periodic reviews on specific dates

---

### Key Learning Points

- ✅ Next processing date is common SailPoint pattern
- ✅ Combines Date Format + Date Math + Date Compare + Conditional
- ✅ Add specific times (6am, 3pm) with Date Math hours
- ✅ Use conditionals to select which date
- ✅ Return null if no future processing needed
- ✅ This enables time-based identity lifecycle

---

## Day 4 Final Assessment

### What You Built Today

**Morning Session:**
- [ ] Date format conversions (multiple formats)
- [ ] Date math calculations (probation, expiration)
- [ ] Date comparisons (pre-hire, terminated, archived)
- [ ] Complete lifecycle state with real dates

**Afternoon Session:**
- [ ] String splitting (names, emails, DNs)
- [ ] Array indexing (extract specific items)
- [ ] Array joining (combine items)
- [ ] AD group name extraction (complex parsing)
- [ ] Location string parsing
- [ ] Next processing date calculator

**Total: 15+ date and list transforms!**

---

### Concepts Mastered

- ✅ ISO 8601 date format
- ✅ Date Format transform
- ✅ Date Math operations
- ✅ Date Compare logic
- ✅ Split transform
- ✅ Index transform
- ✅ Join transform
- ✅ Complex string parsing
- ✅ Multi-value attribute handling
- ✅ Transform chaining (10+ steps)
- ✅ Production-ready patterns

---

### Final Challenge Quiz

#### Challenge 1: Parse Full Name

From "Smith, John Michael" extract:
- Lastname: "Smith"
- Firstname: "John"
- Middle: "Michael"

<details>
<summary>Click to reveal solution</summary>
```
Step 1: Split by ","
  Output: ["Smith", " John Michael"]

Step 2: Extract lastname
  Index(0) + Trim
  Output: "Smith"

Step 3: Extract firstname + middle
  Index(1) + Trim
  Output: "John Michael"

Step 4: Split firstname+middle by space
  Split(" ")
  Output: ["John", "Michael"]

Step 5: Extract firstname
  Index(0)
  Output: "John"

Step 6: Extract middle
  Index(1)
  Output: "Michael"
```

</details>

---

#### Challenge 2: Extract Email Domain Levels

From "user@mail.subdomain.company.com" extract:
- Top domain: "com"
- Company: "company"
- Subdomain: "subdomain"

<details>
<summary>Click to reveal solution</summary>
```
Step 1: Split by "@"
  Output: ["user", "mail.subdomain.company.com"]

Step 2: Index(1) - get domain part
  Output: "mail.subdomain.company.com"

Step 3: Split by "."
  Output: ["mail", "subdomain", "company", "com"]

Step 4: Extract each level
  Index(3) --> "com" (top domain)
  Index(2) --> "company"
  Index(1) --> "subdomain"
  Index(0) --> "mail"
```

</details>

---

#### Challenge 3: Calculate 30-60-90 Day Review Dates

Calculate three review dates:
- 30 days after hire
- 60 days after hire
- 90 days after hire

<details>
<summary>Click to reveal solution</summary>
```
Prerequisite: hireDateISO (in ISO8601)

Attribute: review30Days
  Transform: Date Math
    Input: hireDateISO
    Add 30 days

Attribute: review60Days
  Transform: Date Math
    Input: hireDateISO
    Add 60 days

Attribute: review90Days
  Transform: Date Math
    Input: hireDateISO
    Add 90 days
```

</details>

---

## Homework Challenges

### Challenge 1: Phone Number Parser

Parse "(214) 555-1234" into:
- Area code: "214"
- Exchange: "555"
- Number: "1234"

Use Split and Index transforms.

---

### Challenge 2: Multi-Source Date Priority

Calculate "effectiveDate" with priority:
1. Use contract_start_date if exists and in future
2. Else use hire_date if exists
3. Else use created_date
4. Format as ISO8601 with 9:00 AM time

Combine FirstValid + Date Format + Date Math.

---

### Challenge 3: Extract All Group Types

From AD groups, categorize into:
- Admin groups (contain "Admin")
- Manager groups (contain "Manager")
- Department groups (contain "Engineering", "Finance", etc.)

This may require multiple conditionals checking if joined group string contains keywords.

---

## Day 4 Complete!

### You've Mastered

- ✅ ISO 8601 date format
- ✅ Date Format, Date Math, Date Compare
- ✅ Split, Index, Join transforms
- ✅ Complex string parsing
- ✅ Multi-value attribute handling
- ✅ Complete lifecycle state with dates
- ✅ Next processing date pattern
- ✅ Transform chains (15+ steps possible!)
- ✅ Production-ready date and list patterns

---

## What's Next: Day 5 Preview

Tomorrow you'll learn:

### Transform Chaining Mastery & Real-World Patterns

**Morning:**
- Advanced chaining strategies
- Performance optimization
- When to use transforms vs rules
- Debugging complex chains
- Best practices for production

**Afternoon:**
- 5 complete production-ready patterns:
  1. Smart username generator (handles all edge cases)
  2. International phone normalizer
  3. Complete employee profile enrichment
  4. Dynamic access level calculator
  5. Intelligent manager resolution

**You'll build enterprise-grade transform solutions!**

---

## Study Tips for Tomorrow

### Review These Concepts

- All transforms learned so far
- Transform chaining principles
- Helper attribute patterns
- Error handling strategies

### Prepare Questions

Think about:
- How do you debug a 10-step transform chain?
- When is a transform chain too complex?
- How do you optimize performance?
- When should you use a rule instead?

---

**Excellent work on Day 4! You've completed your transform toolkit!**

---

*Part of the SailPoint ISC Transforms Mastery 7-Day Study Plan*
