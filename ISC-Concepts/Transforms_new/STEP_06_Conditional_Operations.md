# 🎓 FOUNDATION PHASE - Lesson 4
## Conditional Operations: Making Transforms Intelligent

---

## What You'll Master
- ✅ Understand conditional logic (if/then/else)
- ✅ Build simple and complex conditions
- ✅ Know the difference between conditional and firstValid
- ✅ Avoid common conditional mistakes
- ✅ Build decision trees for complex business rules

---

## Why Conditional Operations Matter

So far, you've learned to:
- Clean data (trim, replace, lowercase)
- Extract data (substring)
- Build data (concatenation)

But you haven't made **decisions**. Conditional operations let transforms ask questions:

```
"Is this person a manager?"
"Did they get hired more than 30 days ago?"
"Is their email valid?"
```

Based on the answer, take different actions. This is where transforms become smart.

---

## Before We Start: String Comparison

Conditional operations use **expressions** to evaluate conditions. You need to understand how conditions work.

### How Conditions Evaluate

A condition is a question that returns TRUE or FALSE:

```
"firstName == 'John'"        → TRUE if firstName is John, FALSE otherwise
"department != 'IT'"         → TRUE if department is not IT, FALSE otherwise
"email.contains('@')"        → TRUE if email contains @, FALSE otherwise
"salary > 50000"             → TRUE if salary greater than 50000
```

### Comparison Operators

| Operator | Meaning | Example |
|----------|---------|---------|
| `==` | Equals | `$firstName == 'John'` |
| `!=` | Not equals | `$department != 'Contractor'` |
| `>` | Greater than | `$salary > 50000` |
| `<` | Less than | `$salary < 30000` |
| `>=` | Greater or equal | `$yearsOfService >= 5` |
| `<=` | Less or equal | `$days <= 30` |

### Special Methods

| Method | Meaning | Example |
|--------|---------|---------|
| `.contains()` | String contains substring | `$email.contains('@')` |
| `.startsWith()` | Starts with | `$jobCode.startsWith('M')` |
| `.endsWith()` | Ends with | `$email.endsWith('.com')` |
| `.length()` | String length | `$firstName.length() > 2` |

### Logical Operators

Combine multiple conditions:

| Operator | Meaning | Example |
|----------|---------|---------|
| `&&` | AND (both must be true) | `$department == 'IT' && $salary > 50000` |
| `\|\|` | OR (at least one true) | `$status == 'Active' \|\| $status == 'Onboarding'` |
| `!` | NOT (opposite) | `!$email.contains('@')` (email does NOT contain @) |

### Real Condition Examples

```
$department == 'Manager'                      → Is department exactly 'Manager'?
$jobCode.startsWith('1')                      → Does jobCode start with '1'?
$hireDate > 2020-01-01                        → Was hire date after Jan 1, 2020?
$firstName.length() > 1                       → Is first name longer than 1 character?
$email.contains('@company.com')               → Does email contain company domain?
$status == 'Active' && $department == 'IT'    → Active AND in IT?
$contractType == 'Full-Time' || employmentType == 'Contractor' → Full-time OR contractor?
```

**Important:** You'll see `$variableName` in conditions. The `$` means "take the value of this attribute".

---

## Operation #1: Conditional

### What It Does
Evaluates a condition. If TRUE, returns one value. If FALSE, returns another value.

Think of it as: **If/Then/Else**

### JSON Structure
```json
{
  "type": "conditional",
  "attributes": {
    "expression": "condition_to_check",
    "positiveCondition": "return_this_if_true",
    "negativeCondition": "return_this_if_false"
  }
}
```

### Real Example #1: Classify by Department

**Requirement:** If someone is in the "Manager" department, mark them as "MANAGER", otherwise "STAFF"

```json
{
  "name": "EmployeeRole_Classification",
  "type": "conditional",
  "attributes": {
    "expression": "$department == 'Manager'",
    "positiveCondition": "MANAGER",
    "negativeCondition": "STAFF"
  }
}
```

**Execution:**

```
User 1: department = "Manager"
  Expression: "Manager" == "Manager"  →  TRUE
  Output: "MANAGER" ✓

User 2: department = "Engineering"
  Expression: "Engineering" == "Manager"  →  FALSE
  Output: "STAFF" ✓
```

### Real Example #2: Classify by Job Code

**Requirement:** If jobCode starts with "M", they're a manager. Otherwise, they're not.

```json
{
  "name": "IsManager_Check",
  "type": "conditional",
  "attributes": {
    "expression": "$jobCode.startsWith('M')",
    "positiveCondition": "YES",
    "negativeCondition": "NO"
  }
}
```

**Execution:**

```
User 1: jobCode = "M1001"
  Expression: "M1001".startsWith('M')  →  TRUE
  Output: "YES" ✓

User 2: jobCode = "E2002"
  Expression: "E2002".startsWith('M')  →  FALSE
  Output: "NO" ✓
```

### Real Example #3: Validate Email Format

**Requirement:** If email contains "@", mark as "VALID", otherwise "INVALID"

```json
{
  "name": "Email_Format_Check",
  "type": "conditional",
  "attributes": {
    "expression": "$email.contains('@')",
    "positiveCondition": "VALID",
    "negativeCondition": "INVALID"
  }
}
```

**Execution:**

```
User 1: email = "john@company.com"
  Expression: contains('@')  →  TRUE
  Output: "VALID" ✓

User 2: email = "johncompany.com"
  Expression: contains('@')  →  FALSE
  Output: "INVALID" ✓
```

### Important: What Goes in positiveCondition and negativeCondition?

These can be:
- **Static values** (simple case above)
- **Other data sources** (more complex)
- **Even other transforms** (advanced nesting)

### Example: Return Different Data Based on Condition

```json
{
  "name": "Email_By_Status",
  "type": "conditional",
  "attributes": {
    "expression": "$employmentType == 'Contractor'",
    "positiveCondition": {
      "type": "static",
      "attributes": { "value": "contractor@external.com" }
    },
    "negativeCondition": {
      "type": "identityAttribute",
      "attributes": { "name": "email" }
    }
  }
}
```

**Meaning:**
- If employmentType is "Contractor" → return static value "contractor@external.com"
- Otherwise → return the identity's email attribute

---

## Multiple Conditions: Nested Conditionals

What if you have MORE than two options?

You nest conditionals inside each other.

### Real Example: Three Classifications

**Requirement:**
- If department = "Executive" → "EXECUTIVE"
- Else if jobCode starts with "M" → "MANAGER"  
- Else → "STAFF"

```json
{
  "name": "EmployeeRole_Complex",
  "type": "conditional",
  "attributes": {
    "expression": "$department == 'Executive'",
    "positiveCondition": "EXECUTIVE",
    "negativeCondition": {
      "type": "conditional",
      "attributes": {
        "expression": "$jobCode.startsWith('M')",
        "positiveCondition": "MANAGER",
        "negativeCondition": "STAFF"
      }
    }
  }
}
```

**How It Works:**

```
Step 1: Is department == 'Executive'?
  YES  → Return "EXECUTIVE"
  NO   → Check next condition (Step 2)

Step 2: Is jobCode.startsWith('M')?
  YES  → Return "MANAGER"
  NO   → Return "STAFF"
```

**Execution Examples:**

```
User 1: department = "Executive", jobCode = "E1001"
  Step 1: "Executive" == "Executive" → TRUE
  Output: "EXECUTIVE"

User 2: department = "Engineering", jobCode = "M1001"
  Step 1: "Engineering" == "Executive" → FALSE
  Step 2: "M1001".startsWith('M') → TRUE
  Output: "MANAGER"

User 3: department = "Engineering", jobCode = "E2002"
  Step 1: "Engineering" == "Executive" → FALSE
  Step 2: "E2002".startsWith('M') → FALSE
  Output: "STAFF"
```

### Visual: Nested Conditionals as a Decision Tree

```
Start
  │
  ├─ Is department == 'Executive'?
  │   ├─ YES → EXECUTIVE ✓
  │   └─ NO
  │       │
  │       └─ Is jobCode.startsWith('M')?
  │           ├─ YES → MANAGER ✓
  │           └─ NO → STAFF ✓
```

---

## Real Production Example: Four-Level Classification

**Requirement:** Complex employee classification:
- If department = "Director" → "DIRECTOR"
- Else if department = "Manager" → "MANAGER"
- Else if yearsOfService >= 10 → "SENIOR_STAFF"
- Else → "JUNIOR_STAFF"

```json
{
  "name": "EmployeeLevel_Complex",
  "type": "conditional",
  "attributes": {
    "expression": "$department == 'Director'",
    "positiveCondition": "DIRECTOR",
    "negativeCondition": {
      "type": "conditional",
      "attributes": {
        "expression": "$department == 'Manager'",
        "positiveCondition": "MANAGER",
        "negativeCondition": {
          "type": "conditional",
          "attributes": {
            "expression": "$yearsOfService >= 10",
            "positiveCondition": "SENIOR_STAFF",
            "negativeCondition": "JUNIOR_STAFF"
          }
        }
      }
    }
  }
}
```

**Execution Example:**

```
User 1: department = "Director", yearsOfService = 5
  Check: Director? YES
  Output: "DIRECTOR"

User 2: department = "Manager", yearsOfService = 5
  Check: Director? NO
  Check: Manager? YES
  Output: "MANAGER"

User 3: department = "Engineering", yearsOfService = 15
  Check: Director? NO
  Check: Manager? NO
  Check: yearsOfService >= 10? YES
  Output: "SENIOR_STAFF"

User 4: department = "Engineering", yearsOfService = 2
  Check: Director? NO
  Check: Manager? NO
  Check: yearsOfService >= 10? NO
  Output: "JUNIOR_STAFF"
```

---

## Complex Expressions: AND, OR, NOT

You can combine conditions:

### Example 1: AND (Both Must Be True)

**Requirement:** If they're in IT AND salary > 50000, mark as "SENIOR_IT"

```json
{
  "name": "Senior_IT_Check",
  "type": "conditional",
  "attributes": {
    "expression": "$department == 'IT' && $salary > 50000",
    "positiveCondition": "SENIOR_IT",
    "negativeCondition": "NOT_SENIOR_IT"
  }
}
```

**Meaning:** Both conditions must be TRUE for positive result

```
User 1: department = "IT", salary = 60000
  Check: IT? YES, AND salary > 50000? YES
  Output: "SENIOR_IT" ✓

User 2: department = "IT", salary = 40000
  Check: IT? YES, AND salary > 50000? NO
  Output: "NOT_SENIOR_IT" ✓

User 3: department = "Finance", salary = 60000
  Check: IT? NO, AND salary > 50000? YES (doesn't matter)
  Output: "NOT_SENIOR_IT" ✓
```

### Example 2: OR (At Least One Must Be True)

**Requirement:** If they're in IT OR Finance, give them "BUSINESS_CRITICAL"

```json
{
  "name": "BusinessCritical_Check",
  "type": "conditional",
  "attributes": {
    "expression": "$department == 'IT' || $department == 'Finance'",
    "positiveCondition": "BUSINESS_CRITICAL",
    "negativeCondition": "REGULAR"
  }
}
```

**Meaning:** At least ONE condition must be TRUE

```
User 1: department = "IT"
  Check: IT? YES, OR Finance? (doesn't matter, already YES)
  Output: "BUSINESS_CRITICAL" ✓

User 2: department = "Finance"
  Check: IT? NO, OR Finance? YES
  Output: "BUSINESS_CRITICAL" ✓

User 3: department = "HR"
  Check: IT? NO, OR Finance? NO
  Output: "REGULAR" ✓
```

### Example 3: NOT (Opposite)

**Requirement:** If they're NOT a contractor, give them "EMPLOYEE_BENEFITS"

```json
{
  "name": "EmployeeBenefits_Check",
  "type": "conditional",
  "attributes": {
    "expression": "!($employmentType == 'Contractor')",
    "positiveCondition": "EMPLOYEE_BENEFITS",
    "negativeCondition": "NO_BENEFITS"
  }
}
```

**Meaning:** The opposite of the condition

```
User 1: employmentType = "Full-Time"
  Check: NOT Contractor? YES
  Output: "EMPLOYEE_BENEFITS" ✓

User 2: employmentType = "Contractor"
  Check: NOT Contractor? NO
  Output: "NO_BENEFITS" ✓
```

---

## Conditional vs FirstValid: When to Use Each

You now know TWO ways to handle multiple options:
1. **Conditional** - Make decisions based on logic
2. **FirstValid** - Try sources in order, use first non-null

### When to Use Conditional

Use when you're **making a decision** based on data values:

```
"Is this person a manager?" (YES/NO decision)
"What's their access level?" (logic-based decision)
"Are they eligible for benefits?" (rules-based decision)
```

**Conditional decides based on VALUES**

### When to Use FirstValid

Use when you're **finding data** from multiple sources:

```
"Get email from AD, if missing try Workday, if missing use default"
(Priority-based search, not logic)
```

**FirstValid searches based on AVAILABILITY**

### Real Example Showing the Difference

**Scenario:** You need to classify users AND you need their email

```json
{
  "name": "User_Classification_and_Email",
  "type": "concatenation",
  "attributes": {
    "values": [
      {
        "type": "conditional",  // DECISION MAKING
        "attributes": {
          "expression": "$department == 'Manager'",
          "positiveCondition": "MANAGER",
          "negativeCondition": "STAFF"
        }
      },
      { "type": "static", "attributes": { "value": ":" } },
      {
        "type": "firstValid",  // DATA SEARCHING
        "attributes": {
          "values": [
            { "type": "accountAttribute", "attributes": { "sourceName": "AD", "attributeName": "mail" } },
            { "type": "accountAttribute", "attributes": { "sourceName": "Workday", "attributeName": "email" } },
            { "type": "static", "attributes": { "value": "unknown@company.com" } }
          ]
        }
      }
    ]
  }
}
```

**Result:** Classification (conditional) + Email (firstValid)
- User 1 (Manager, AD email): "MANAGER:john@company.com"
- User 2 (Staff, Workday email): "STAFF:jane@workday.com"
- User 3 (Manager, no email): "MANAGER:unknown@company.com"

---

## Important: NULL Handling in Conditionals

What if the attribute you're checking is NULL?

### Example: Missing Department

```json
{
  "type": "conditional",
  "attributes": {
    "expression": "$department == 'Manager'",
    "positiveCondition": "MANAGER",
    "negativeCondition": "STAFF"
  }
}
```

**If department is NULL:**
```
Check: NULL == 'Manager'  →  FALSE
Output: "STAFF"
```

**Is this correct?** Maybe. NULL usually means "not manager", so STAFF is safe.

But if you want to be explicit about missing data:

```json
{
  "type": "conditional",
  "attributes": {
    "expression": "$department != null && $department == 'Manager'",
    "positiveCondition": "MANAGER",
    "negativeCondition": "STAFF"
  }
}
```

**Meaning:** Check that department exists AND equals "Manager"

---

## Common Mistakes with Conditionals

### Mistake #1: Case Sensitivity

```json
// WRONG: Won't match if case is different
{
  "expression": "$department == 'manager'"
}

User has: department = "Manager"
Check: "Manager" == "manager"  →  FALSE ❌
Output: Wrong result
```

**Fix:** Use .equalsIgnoreCase() or standardize case:

```json
{
  "expression": "$department.toLowerCase() == 'manager'"
}
```

---

### Mistake #2: Forgetting Quotes

```json
// WRONG: No quotes around string value
{
  "expression": "$department == Manager"
}

// CORRECT: Quotes around string
{
  "expression": "$department == 'Manager'"
}
```

---

### Mistake #3: Using = Instead of ==

```json
// WRONG: = is assignment, not comparison
{
  "expression": "$department = 'Manager'"
}

// CORRECT: == is comparison
{
  "expression": "$department == 'Manager'"
}
```

---

### Mistake #4: Too Many Nesting Levels

```json
// HARD TO READ: 5+ levels deep
{
  "type": "conditional",
  "attributes": {
    "expression": "...",
    "positiveCondition": "A",
    "negativeCondition": {
      "type": "conditional",
      "attributes": {
        "expression": "...",
        "positiveCondition": "B",
        "negativeCondition": {
          "type": "conditional",
          "attributes": {
            ...
          }
        }
      }
    }
  }
}
```

**Solution:** If you need more than 3-4 levels, consider:
1. Breaking into multiple transforms
2. Using a Rule instead

---

## Conditional Operations Quick Reference

| Scenario | Use This | Example |
|----------|----------|---------|
| True/False decision | Conditional | Is person a manager? |
| Multiple options | Nested Conditional | What level? (Director/Manager/Staff) |
| AND logic | `expression: "a && b"` | Department is IT AND salary > 50k |
| OR logic | `expression: "a \|\| b"` | Department is IT OR Finance |
| NOT logic | `expression: "!a"` | NOT a contractor |
| Null check | `expression: "a != null"` | Department exists |

---

## Mastery Checkpoint #4

### Question 1: Simple Conditional

What will this return for someone with department = "Engineering"?

```json
{
  "type": "conditional",
  "attributes": {
    "expression": "$department == 'Manager'",
    "positiveCondition": "MANAGER",
    "negativeCondition": "NOT_MANAGER"
  }
}
```

<details>
<summary>Answer</summary>

**"NOT_MANAGER"**

Because "Engineering" != "Manager", the expression is FALSE, so negativeCondition is returned.

</details>

---

### Question 2: Nested Conditional

What will this return for someone with status = "Active" and salary = 40000?

```json
{
  "type": "conditional",
  "attributes": {
    "expression": "$status == 'Active'",
    "positiveCondition": "ACTIVE_USER",
    "negativeCondition": {
      "type": "conditional",
      "attributes": {
        "expression": "$salary > 50000",
        "positiveCondition": "HIGH_PAY",
        "negativeCondition": "LOW_PAY"
      }
    }
  }
}
```

<details>
<summary>Answer</summary>

**"ACTIVE_USER"**

Step 1: Is status == 'Active'? YES
Returns: "ACTIVE_USER"

(The nested conditional is never checked because the first condition is TRUE)

</details>

---

### Question 3: AND Logic

What will this return for someone with department = "IT" and salary = 40000?

```json
{
  "type": "conditional",
  "attributes": {
    "expression": "$department == 'IT' && $salary > 50000",
    "positiveCondition": "SENIOR_IT",
    "negativeCondition": "NOT_SENIOR_IT"
  }
}
```

<details>
<summary>Answer</summary>

**"NOT_SENIOR_IT"**

Even though department = "IT" (TRUE), salary is 40000 which is NOT > 50000 (FALSE).

For AND logic, BOTH must be TRUE. Since one is FALSE, the whole expression is FALSE.

</details>

---

### Question 4: Real Scenario

Build a conditional that returns:
- "EXECUTIVE" if department = "Director"
- "MANAGER" if jobCode starts with "M"
- "STAFF" otherwise

<details>
<summary>Answer</summary>

```json
{
  "name": "RoleClassification_ThreeLevel",
  "type": "conditional",
  "attributes": {
    "expression": "$department == 'Director'",
    "positiveCondition": "EXECUTIVE",
    "negativeCondition": {
      "type": "conditional",
      "attributes": {
        "expression": "$jobCode.startsWith('M')",
        "positiveCondition": "MANAGER",
        "negativeCondition": "STAFF"
      }
    }
  }
}
```

</details>

---

## Summary: Conditional Operations

| Concept | What It Does |
|---------|--------------|
| **Conditional** | Evaluates a condition, returns different values based on TRUE/FALSE |
| **Expression** | The condition being checked (uses comparison operators) |
| **positiveCondition** | What to return if expression is TRUE |
| **negativeCondition** | What to return if expression is FALSE |
| **Nested** | Put conditionals inside conditionals for multiple options |
| **AND (&&)** | Both conditions must be TRUE |
| **OR (\|\|)** | At least one condition must be TRUE |
| **NOT (!)** | Opposite of the condition |

---

## What's Next

In **Lesson 5**, you'll learn:
- ✅ Data operations (working with dates and numbers)
- ✅ Date formatting and calculations
- ✅ Comparison operations

These let you make decisions based on dates and numbers.

---

## Homework Before Lesson 5

1. For 3 people in your environment, design the conditional that would classify them correctly
2. Write out the decision logic: "Is X true? If yes, then... if no, then..."
3. Think about edge cases: What if the data is NULL?

**Next: Date and Data Operations** — where transforms work with time-based logic.
