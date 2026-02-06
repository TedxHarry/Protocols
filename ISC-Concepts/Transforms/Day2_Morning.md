# DAY 2: CONDITIONAL LOGIC & DECISION MAKING - MORNING SESSION

## Table of Contents
- [Overview](#overview)
- [What You'll Learn Today](#what-youll-learn-today)
- [Transform 1: Conditional](#transform-1-conditional-45-minutes)
  - [Understanding Conditional Logic](#understanding-conditional-logic)
  - [Operators You Need to Know](#operators-you-need-to-know)
  - [Exercise 1A: Simple Conditional](#exercise-1a-simple-conditional)
  - [Exercise 1B: Multi-Condition Lifecycle State](#exercise-1b-multi-condition-lifecycle-state)
  - [Exercise 1C: Complex Nested Conditionals](#exercise-1c-complex-nested-conditionals)
- [Transform 2: FirstValid](#transform-2-firstvalid-30-minutes)
  - [Understanding FirstValid](#understanding-firstvalid)
  - [Exercise 2A: Simple Fallback](#exercise-2a-simple-fallback)
  - [Exercise 2B: Multi-Level Fallback Chain](#exercise-2b-multi-level-fallback-chain)
  - [Exercise 2C: Combining FirstValid with Other Transforms](#exercise-2c-combining-firstvalid-with-other-transforms)
- [Transform 3: Static with Conditions](#transform-3-static-with-conditions-15-minutes)
- [Morning Session Review](#morning-session-review)
- [Checkpoint Quiz](#checkpoint-quiz)

---

## Overview

**Session Duration:** 2 hours

**Prerequisites:** 
- Completed Day 1 (Morning & Afternoon)
- Comfortable with basic string transforms
- Understand transform chaining

**Goal:** Master conditional logic and decision-making in transforms

---

## What You'll Learn Today

By the end of this morning session, you'll be able to:

- ✅ Build if-then-else logic with Conditional transforms
- ✅ Use comparison operators (eq, ne, gt, lt, contains, etc.)
- ✅ Create nested conditions for complex decision trees
- ✅ Handle null values gracefully with FirstValid
- ✅ Build multi-level fallback chains
- ✅ Combine conditional logic with string transforms

---

## Transform 1: Conditional (45 minutes)

### Understanding Conditional Logic

The **Conditional** transform is your if-then-else decision maker.

#### Basic Structure
```
IF (condition is true)
  THEN return this value
ELSE
  return that value
```

#### Real-World Example
```
IF employee_type equals "FTE"
  THEN "Full-Time Employee"
ELSE "Contractor"
```

#### In Transform Terms
```
INPUT: employee_type = "FTE"

TRANSFORM: Conditional
  Condition: employee_type equals "FTE"
  True Value: "Full-Time Employee"
  False Value: "Contractor"

OUTPUT: "Full-Time Employee"
```

---

### Operators You Need to Know

The Conditional transform supports these comparison operators:

| Operator | Meaning | Example |
|----------|---------|---------|
| `eq` | Equals | `status eq "Active"` |
| `ne` | Not equals | `status ne "Terminated"` |
| `gt` | Greater than | `age gt 18` |
| `lt` | Less than | `salary lt 100000` |
| `ge` | Greater than or equal | `years ge 5` |
| `le` | Less than or equal | `hours le 40` |
| `contains` | Contains substring | `email contains "@company.com"` |
| `startsWith` | Starts with | `dept startsWith "ENG"` |
| `endsWith` | Ends with | `title endsWith "Manager"` |
| `in` | In a list | `status in ["Active", "Leave"]` |

#### Important Notes

- Operators are **case-sensitive** for the operator itself
- String comparisons are **case-sensitive** for values
- `"Active"` is NOT equal to `"active"`

---

### Exercise 1A: Simple Conditional

**Scenario:** Set employee type description based on code

#### Business Requirement
```
If employee_type is "FTE", set description to "Full-Time Employee"
Otherwise, set description to "Contractor"
```

#### Steps
```
1. Go to Identity Profile > Mappings
2. Create new attribute "employeeTypeDescription"
3. Click on the attribute
4. Source: Transform
5. Transform: Conditional
6. Configuration:
   
   Condition Expression:
   - Attribute: employee_type
   - Operator: eq (equals)
   - Value: "FTE"
   
   Value if True: "Full-Time Employee"
   Value if False: "Contractor"

7. Preview with different identities
8. Save
```

#### Expected Results
```
Test Case 1:
  Input: employee_type = "FTE"
  Output: "Full-Time Employee"

Test Case 2:
  Input: employee_type = "CTR"
  Output: "Contractor"

Test Case 3:
  Input: employee_type = null
  Output: "Contractor" (falls to else)
```

#### Key Learning

- The condition is a **binary decision** (true or false)
- **True path** executes when condition matches
- **False path** (else) executes for everything else
- **Null values** typically go to false path

---

### Exercise 1B: Multi-Condition Lifecycle State

**Scenario:** Calculate lifecycle state based on employment dates and status

#### Business Logic
```
IF status equals "On Leave"
  THEN "Leave of Absence"
ELSE IF term_date is not null
  THEN "Terminated"
ELSE IF hire_date > today
  THEN "Pre-hire"
ELSE
  "Active"
```

This requires **nested conditionals**!

#### Understanding Nested Conditionals
```
Conditional 1 (Outer):
  IF status eq "On Leave"
    TRUE --> "Leave of Absence"
    FALSE --> Conditional 2 (Inner)
    
Conditional 2 (Inner):
  IF term_date is not null
    TRUE --> "Terminated"
    FALSE --> Conditional 3
    
Conditional 3:
  IF hire_date > today
    TRUE --> "Pre-hire"
    FALSE --> "Active"
```

#### Steps

> **Note:** This creates a chain of conditionals. Each "False" path leads to the next conditional.
```
1. Create attribute "calculatedLifecycleState"
2. Transform: Conditional (First Level)
   
   Condition 1:
   - Attribute: status
   - Operator: eq
   - Value: "On Leave"
   - If True: "Leave of Absence"
   - If False: [Add another Conditional transform]

3. Add Transform: Conditional (Second Level)
   
   Condition 2:
   - Attribute: term_date
   - Operator: ne (not equals)
   - Value: null
   - If True: "Terminated"
   - If False: [Add another Conditional transform]

4. Add Transform: Conditional (Third Level)
   
   Condition 3:
   - This requires Date Compare (we'll learn this afternoon)
   - For now, use a simplified version:
   - Attribute: hire_date
   - Operator: contains (placeholder)
   - Value: "2026" (future year)
   - If True: "Pre-hire"
   - If False: "Active"

5. Preview with different scenarios
6. Save
```

> **Note:** We'll improve this with proper date comparisons in the afternoon session. This exercise focuses on the nesting concept.

#### Expected Results
```
Test Case 1:
  status = "On Leave"
  Output: "Leave of Absence"

Test Case 2:
  status = "Active", term_date = "2024-12-31"
  Output: "Terminated"

Test Case 3:
  status = "Active", term_date = null, hire_date = "2026-06-01"
  Output: "Pre-hire"

Test Case 4:
  status = "Active", term_date = null, hire_date = "2024-01-15"
  Output: "Active"
```

#### Key Learning

- **Nested conditionals** create decision trees
- Each level handles one decision
- Order matters! First match wins
- Can nest **3-5 levels** typically (more gets hard to maintain)

---

### Exercise 1C: Complex Nested Conditionals

**Scenario:** Set country based on office location

#### Business Logic
```
IF office_location contains "Austin" OR contains "Dallas"
  THEN "USA"
ELSE IF office_location contains "London"
  THEN "UK"
ELSE IF office_location contains "Tokyo"
  THEN "Japan"
ELSE
  "Unknown"
```

#### Challenge

The Conditional transform doesn't support **OR** directly in a single condition!

**Solution:** Use `in` operator with a list, or nest conditionals.

#### Steps - Method 1: Using "in" operator (if location is exact)
```
1. Create attribute "country"
2. Transform: Conditional
   
   Condition:
   - Attribute: office_location
   - Operator: in
   - Value: ["Austin", "Dallas", "Houston"]
   - If True: "USA"
   - If False: [Next Conditional]

3. Add Conditional:
   - Attribute: office_location
   - Operator: in
   - Value: ["London", "Manchester"]
   - If True: "UK"
   - If False: [Next Conditional]

4. Add Conditional:
   - Attribute: office_location
   - Operator: in
   - Value: ["Tokyo", "Osaka"]
   - If True: "Japan"
   - If False: "Unknown"

5. Preview
6. Save
```

#### Steps - Method 2: Using "contains" (if location has extra text)
```
1. Create attribute "country"
2. Transform: Conditional
   
   Condition 1:
   - Attribute: office_location
   - Operator: contains
   - Value: "Austin"
   - If True: "USA"
   - If False: [Next Conditional]

3. Add Conditional:
   - Attribute: office_location
   - Operator: contains
   - Value: "Dallas"
   - If True: "USA"
   - If False: [Next Conditional]

4. Add Conditional:
   - Attribute: office_location
   - Operator: contains
   - Value: "London"
   - If True: "UK"
   - If False: [Next Conditional]

5. Continue pattern...
6. Final Else: "Unknown"
```

#### Expected Results
```
Test Case 1:
  office_location = "Austin"
  Output: "USA"

Test Case 2:
  office_location = "Austin Office - Building 2"
  Output: "USA" (contains "Austin")

Test Case 3:
  office_location = "London"
  Output: "UK"

Test Case 4:
  office_location = "Sydney"
  Output: "Unknown"
```

#### Key Learning

- For **OR logic**, use `in` operator with list when possible
- For **complex OR**, nest multiple conditionals
- **Order matters** - first match wins, so order by priority
- For many options, consider using **Lookup** transform instead (more efficient)

---

### When to Use Conditional vs Lookup

| Use Conditional When | Use Lookup When |
|---------------------|-----------------|
| 2-5 options | 10+ options |
| Logic-based decisions | Simple value mapping |
| Complex conditions | Static key-value pairs |
| Dynamic calculations | Standardization |

**Example:**
```
Conditional: Good for "if status = X and date > Y then Z"
Lookup: Good for "ENG" --> "Engineering" (50+ dept codes)
```

---

## Transform 2: FirstValid (30 minutes)

### Understanding FirstValid

**FirstValid** returns the **first non-null value** from a list of inputs.

#### Why This Matters

Source data is messy! You often need fallback logic:
```
Try preferred_email
If that's null, try work_email
If that's null, try personal_email
If that's null, use default_email
```

#### Basic Structure
```
INPUT:
  preferred_email = null
  work_email = "john@company.com"
  personal_email = "jsmith@gmail.com"

TRANSFORM: FirstValid
  Check in order:
  1. preferred_email (null - skip)
  2. work_email ("john@company.com" - FOUND!)
  
OUTPUT: "john@company.com"
```

---

### Exercise 2A: Simple Fallback

**Scenario:** Use preferred email if available, otherwise use work email

#### Steps
```
1. Create attribute "primaryEmail"
2. Transform: FirstValid
3. Configuration:
   
   Values to check (in order):
   - Add: preferred_email
   - Add: work_email
   - Add: Static value "noemail@company.com"

4. Preview with identities that have:
   - Both emails
   - Only work email
   - Neither email
5. Save
```

#### Expected Results
```
Test Case 1:
  preferred_email = "john.smith@company.com"
  work_email = "jsmith@company.com"
  Output: "john.smith@company.com" (first non-null)

Test Case 2:
  preferred_email = null
  work_email = "jsmith@company.com"
  Output: "jsmith@company.com" (second option)

Test Case 3:
  preferred_email = null
  work_email = null
  Output: "noemail@company.com" (fallback to static)
```

#### Key Learning

- FirstValid evaluates **in order**
- Returns **first non-null** value
- Always include a **static fallback** at the end to guarantee a value
- **Order is critical** - most preferred first!

---

### Exercise 2B: Multi-Level Fallback Chain

**Scenario:** Generate email with multiple fallback strategies

#### Business Logic
```
Priority 1: Use preferred_email if exists
Priority 2: Use work_email if exists
Priority 3: Generate from firstname.lastname if both exist
Priority 4: Use employeeID@company.com if ID exists
Priority 5: Use "noemail@company.com"
```

This requires **combining FirstValid with other transforms**!

#### Steps
```
1. Create attribute "smartEmail"
2. Transform: FirstValid
3. Configuration:
   
   Values to check:
   - Add: preferred_email (attribute)
   - Add: work_email (attribute)
   - Add: [Generated email - see below]
   - Add: [Generated ID email - see below]
   - Add: Static "noemail@company.com"
```

**For the generated email (option 3):**

You need a **separate transform chain** that creates it:
```
Option 3: Generated Email Transform
Create as a separate attribute first:

1. Create attribute "generatedEmail"
2. Transform chain:
   - Trim firstname
   - Trim lastname
   - Concatenation (firstname + "." + lastname + "@company.com")
   - Lower
3. Save

Now reference this in your FirstValid!
```

**Updated FirstValid:**
```
Values to check:
- preferred_email
- work_email
- generatedEmail (the attribute you just created)
- [ID email - create similarly]
- Static "noemail@company.com"
```

#### Alternative: Conditional + FirstValid

You can also use Conditional to check if firstname/lastname exist before generating:
```
1. Conditional transform:
   IF firstname is not null AND lastname is not null
     THEN Concatenate (firstname.lastname@company.com)
     ELSE null

2. Use this conditional result in FirstValid
```

#### Expected Results
```
Test Case 1:
  preferred_email = "john@preferred.com"
  Output: "john@preferred.com" (priority 1)

Test Case 2:
  preferred_email = null
  work_email = "john@work.com"
  Output: "john@work.com" (priority 2)

Test Case 3:
  preferred_email = null
  work_email = null
  firstname = "John"
  lastname = "Smith"
  Output: "john.smith@company.com" (priority 3)

Test Case 4:
  All nulls except employeeID = "E12345"
  Output: "e12345@company.com" (priority 4)

Test Case 5:
  Everything null
  Output: "noemail@company.com" (fallback)
```

#### Key Learning

- FirstValid can reference **other attributes** (that are themselves calculated)
- Build **complex fallback chains** with multiple strategies
- Always test with **all null** scenario
- **Order = priority**

---

### Exercise 2C: Combining FirstValid with Other Transforms

**Scenario:** Get manager email with fallback and formatting

#### Business Logic
```
1. Try to get manager's email from their AD account
2. If not found, generate from manager's firstname.lastname
3. If no manager name, use department head email
4. If no department head, use "no-manager@company.com"
```

#### Steps
```
1. Create attribute "managerEmail"
2. Transform: FirstValid
3. Values:
   - Account Attribute (manager's AD email) - if available
   - managerGeneratedEmail (create this as separate attribute)
   - Lookup (department --> department_head_email)
   - Static "no-manager@company.com"

4. Preview
5. Save
```

**For managerGeneratedEmail (create separately):**
```
1. Create attribute "managerGeneratedEmail"
2. Transform chain:
   - Conditional:
     IF manager_firstname is not null AND manager_lastname is not null
       THEN continue to next transform
       ELSE return null
   - Trim manager_firstname
   - Trim manager_lastname
   - Concatenation (manager_firstname + "." + manager_lastname + "@company.com")
   - Lower
3. Save
```

#### Expected Results
```
Test Case 1:
  Manager has AD account with email = "jane.doe@company.com"
  Output: "jane.doe@company.com" (from AD)

Test Case 2:
  No AD email, but manager_firstname = "Jane", manager_lastname = "Doe"
  Output: "jane.doe@company.com" (generated)

Test Case 3:
  No manager info, but department = "Engineering"
  Lookup table has: Engineering --> "eng-lead@company.com"
  Output: "eng-lead@company.com"

Test Case 4:
  No manager, no department head
  Output: "no-manager@company.com"
```

#### Key Learning

- Combine **multiple transform types** in FirstValid
- Account Attribute + Generated values + Lookup + Static
- Build **sophisticated fallback strategies**
- Always have **final static fallback**

---

## Transform 3: Static with Conditions (15 minutes)

### Understanding Context-Aware Static Values

Sometimes you want a **static value**, but **conditionally applied**.

#### Example Scenario
```
All employees in the "Employee" Identity Profile get country = "United States"
But you want to conditionally set it based on another attribute
```

#### Combining Static + Conditional
```
Transform: Conditional
  IF office_location contains "London"
    THEN Static "United Kingdom"
  ELSE Static "United States"
```

### Quick Exercise

**Scenario:** Set timezone based on office

#### Steps
```
1. Create attribute "timezone"
2. Transform: Conditional
   - IF office_location contains "Austin"
     THEN Static "America/Chicago"
   - ELSE IF office_location contains "London"
     THEN Static "Europe/London"
   - ELSE Static "America/New_York" (default)
3. Preview
4. Save
```

#### Key Learning

- Static values can be **conditional**
- Use when you have **fixed values** for each condition
- Alternative to Lookup for **small number** of options

---

## Morning Session Review

### Transforms Learned

| Transform | Purpose | Key Use Cases |
|-----------|---------|---------------|
| **Conditional** | If-then-else logic | Decision making, calculations |
| **FirstValid** | First non-null value | Fallback chains, null handling |
| **Static (conditional)** | Fixed values conditionally | Simple mappings |

---

### What You Built

- [ ] Simple conditional (employee type)
- [ ] Nested conditional (lifecycle state)
- [ ] Complex nested conditional (country from location)
- [ ] Simple FirstValid (email fallback)
- [ ] Multi-level FirstValid (smart email generator)
- [ ] Combined transforms (manager email with fallbacks)
- [ ] Conditional static (timezone)

**Total: 7+ conditional transforms with real-world logic!**

---

### Key Concepts Mastered

- ✅ If-then-else logic with Conditional transform
- ✅ Comparison operators (eq, ne, gt, lt, contains, in, etc.)
- ✅ Nested conditionals for decision trees
- ✅ FirstValid for null handling
- ✅ Multi-level fallback strategies
- ✅ Combining conditionals with string transforms
- ✅ Order of evaluation matters

---

## Checkpoint Quiz

### Question 1: Simple Conditional

Build a transform:
```
If status equals "Active", return "Employed"
Otherwise, return "Not Employed"
```

<details>
<summary>Click to reveal answer</summary>
```
Transform: Conditional
  Condition: status eq "Active"
  If True: "Employed"
  If False: "Not Employed"
```

</details>

---

### Question 2: FirstValid

Build a transform with fallback:
```
Priority 1: Use mobile_phone if exists
Priority 2: Use work_phone if exists
Priority 3: Use "555-0000"
```

<details>
<summary>Click to reveal answer</summary>
```
Transform: FirstValid
  Values:
  1. mobile_phone
  2. work_phone
  3. Static "555-0000"
```

</details>

---

### Question 3: Nested Conditional

Build a transform:
```
If employee_type = "FTE" and years_of_service > 5
  Return "Senior Employee"
Else if employee_type = "FTE"
  Return "Employee"
Else
  Return "Contractor"
```

<details>
<summary>Click to reveal answer</summary>

**Challenge:** You need TWO conditions checked together (FTE AND years > 5)

**Solution:** Nest the conditions:
```
Conditional 1 (Check type first):
  IF employee_type eq "FTE"
    TRUE --> Conditional 2 (Check years)
    FALSE --> "Contractor"

Conditional 2:
  IF years_of_service gt 5
    TRUE --> "Senior Employee"
    FALSE --> "Employee"
```

**Alternative:** Create a helper attribute first that combines the logic, then use in main conditional.

</details>

---

### Question 4: What's Wrong?
```
Transform: FirstValid
  Values:
  1. Static "default@company.com"
  2. work_email
  3. personal_email
```

What will ALWAYS be returned?

<details>
<summary>Click to reveal answer</summary>

**Answer:** "default@company.com" will ALWAYS be returned!

**Why:** Static values are never null, so FirstValid stops at the first value (the static).

**Fix:** Put static value LAST:
```
Transform: FirstValid
  Values:
  1. work_email
  2. personal_email
  3. Static "default@company.com"
```

**Key Learning:** Order matters! Most preferred first, fallback last.

</details>

---

### Question 5: Real-World Scenario

Build an email generator:
```
Priority 1: Use preferred_email if it exists
Priority 2: If firstname and lastname exist, generate firstname.lastname@company.com (lowercase)
Priority 3: Use "noemail@company.com"
```

<details>
<summary>Click to reveal answer</summary>

**Answer:**

**Step 1:** Create generated email as separate attribute:
```
Attribute: generatedEmailFromName
Transform chain:
1. Conditional:
   IF firstname ne null AND lastname ne null
     TRUE --> continue
     FALSE --> null
2. Trim firstname
3. Trim lastname
4. Concatenation (firstname + "." + lastname + "@company.com")
5. Lower
```

**Step 2:** Use in FirstValid:
```
Attribute: primaryEmail
Transform: FirstValid
  Values:
  1. preferred_email
  2. generatedEmailFromName
  3. Static "noemail@company.com"
```

**Alternative:** Use Conditional inside FirstValid if your ISC version supports it.

</details>

---

## Morning Session Complete!

### You've Mastered

- ✅ Conditional transform with all major operators
- ✅ Nested conditionals for complex logic
- ✅ FirstValid for fallback chains
- ✅ Combining transforms for sophisticated logic
- ✅ Null handling strategies
- ✅ Order of evaluation

---

## What's Next: Afternoon Session Preview

In the afternoon, you'll learn:

### Advanced Conditional Patterns

- **Date comparisons** for lifecycle state calculations
- **Combining multiple conditions** (AND/OR logic)
- **Country assignment** with complex location logic
- **Manager email resolution** with multiple fallback strategies

### Real-World Complete Solutions

- Smart email generator (handling all edge cases)
- Complete lifecycle state calculator
- Department hierarchy resolution
- Phone number validation with fallbacks

**You'll build production-ready transforms that handle real-world complexity!**

---

## Practice Before Afternoon

Try building these:

### Challenge 1: Access Level
```
If title contains "VP" or "Director"
  Return "Executive"
Else if title contains "Manager"
  Return "Manager"
Else if title contains "Senior"
  Return "Senior"
Else
  Return "Staff"
```

### Challenge 2: Smart Display Name
```
Priority 1: Use displayName if it exists
Priority 2: Generate "Lastname, Firstname" if both exist
Priority 3: Use firstname only if lastname is null
Priority 4: Use lastname only if firstname is null
Priority 5: Use username
```

---

**Great work! Take a break, then continue to the Afternoon Session!**

---

*Part of the SailPoint ISC Transforms Mastery 7-Day Study Plan*
