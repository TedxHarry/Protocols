# 🎓 FOUNDATION PHASE - Lesson 5
## Date and Numeric Operations

---

## What You'll Master
- ✅ Work with dates in transforms
- ✅ Convert date formats between systems
- ✅ Calculate dates (add/subtract days, months, years)
- ✅ Compare dates for lifecycle decisions
- ✅ Handle numeric comparisons
- ✅ Build time-based business logic

---

## Why Date Operations Matter

Identity management is about **lifecycle**:
- When was someone hired? (hireDate)
- When do they get reviewed? (calculation)
- When did they get terminated? (terminationDate)
- Are they in onboarding? (hire date < 30 days)
- Should they be deprovisioned? (termination date > 90 days)

All of these require working with dates.

---

## Important: Understanding Date Formats

Different systems use different date formats. This is the #1 cause of date problems.

### Common Date Formats

| Format | Example | System |
|--------|---------|--------|
| ISO 8601 | 2020-01-15 | Most databases, APIs |
| US Format | 01/15/2020 | US-based systems |
| European | 15/01/2020 | European systems |
| Text Month | January 15, 2020 | HR documents |
| Unix Timestamp | 1579046400 | Technical systems |

**Problem:** 01/15/2020 could mean January 15 OR 15th of January depending on the system.

**Solution:** Use date operations to convert between formats.

---

## Operation #1: DateFormat

### What It Does
Converts a date from one format to another.

### JSON Structure
```json
{
  "type": "dateFormat",
  "attributes": {
    "input": "source_of_date",
    "inputFormat": "source_format",
    "outputFormat": "target_format"
  }
}
```

### Format Syntax

Dates use pattern codes:

| Code | Meaning | Example |
|------|---------|---------|
| `yyyy` | 4-digit year | 2020 |
| `MM` | 2-digit month | 01 (January) |
| `dd` | 2-digit day | 15 |
| `HH` | 2-digit hour (24-hour) | 14 (2 PM) |
| `mm` | 2-digit minute | 30 |
| `ss` | 2-digit second | 45 |

### Real Example #1: ISO to US Format

**Scenario:** Workday provides dates as "2020-01-15" but AD needs "01/15/2020"

```json
{
  "name": "HireDate_Workday_to_AD",
  "type": "dateFormat",
  "attributes": {
    "input": {
      "type": "accountAttribute",
      "attributes": {
        "sourceName": "Workday",
        "attributeName": "hireDate"
      }
    },
    "inputFormat": "yyyy-MM-dd",
    "outputFormat": "MM/dd/yyyy"
  }
}
```

**Execution:**

```
Input:  "2020-01-15" (Workday format)
Pattern: "yyyy-MM-dd" means: year-month-day
Parse:   2020=year, 01=month, 15=day
Convert: MM/dd/yyyy means: month/day/year
Output:  "01/15/2020" (AD format)
```

### Real Example #2: Full Datetime Conversion

**Scenario:** Source has "2020-01-15 14:30:45" but system needs "01/15/2020 2:30 PM"

```json
{
  "name": "DateTime_Full_Convert",
  "type": "dateFormat",
  "attributes": {
    "input": {
      "type": "accountAttribute",
      "attributes": {
        "sourceName": "JDBC_HR",
        "attributeName": "createdDateTime"
      }
    },
    "inputFormat": "yyyy-MM-dd HH:mm:ss",
    "outputFormat": "MM/dd/yyyy hh:mm a"
  }
}
```

**Format Codes Used:**
- `HH` = 24-hour format (14)
- `hh` = 12-hour format (02, plus 'a' for AM/PM)
- `a` = AM/PM indicator

**Execution:**

```
Input:  "2020-01-15 14:30:45"
Output: "01/15/2020 02:30 PM"
```

### Real Example #3: Extract Just the Year

```json
{
  "name": "HireDate_Extract_Year",
  "type": "dateFormat",
  "attributes": {
    "input": {
      "type": "identityAttribute",
      "attributes": { "name": "hireDate" }
    },
    "inputFormat": "yyyy-MM-dd",
    "outputFormat": "yyyy"
  }
}
```

**Execution:**

```
Input:  "2020-01-15"
Output: "2020"
```

### Common Date Format Mistakes

**Mistake #1: Wrong format pattern**

```json
// WRONG: Using lowercase 'mm' for month (that's minutes!)
{
  "inputFormat": "yyyy-mm-dd"  // WRONG
}

// CORRECT: Use 'MM' for months
{
  "inputFormat": "yyyy-MM-dd"  // CORRECT
}
```

**Mistake #2: Inconsistent separators**

```json
// WRONG: Input has dashes, pattern has slashes
{
  "input": "2020-01-15",
  "inputFormat": "yyyy/MM/dd"  // Pattern says slashes, but input has dashes
}

// CORRECT: Match the separator
{
  "input": "2020-01-15",
  "inputFormat": "yyyy-MM-dd"  // Matches the dash separator
}
```

---

## Operation #2: DateMath

### What It Does
Adds or subtracts time from a date. Also evaluates "now".

### JSON Structure
```json
{
  "type": "dateMath",
  "attributes": {
    "input": "source_of_date",
    "expression": "calculation_expression"
  }
}
```

### Expression Syntax

| Expression | Meaning | Example |
|-----------|---------|---------|
| `now` | Current date/time | "now" → 2024-02-22 |
| `now+Xd` | Now plus X days | "now+30d" → 30 days from today |
| `now-Xd` | Now minus X days | "now-30d" → 30 days ago |
| `now+Xm` | Now plus X months | "now+6m" → 6 months from now |
| `now-Xm` | Now minus X months | "now-6m" → 6 months ago |
| `now+Xy` | Now plus X years | "now+1y" → 1 year from now |
| `date+Xd` | Date plus X days | "$hireDate+90d" → hire date + 90 days |

### Real Example #1: Calculate Review Date

**Scenario:** Employee review is 1 year after hire date

```json
{
  "name": "ReviewDate_One_Year_After_Hire",
  "type": "dateMath",
  "attributes": {
    "input": {
      "type": "identityAttribute",
      "attributes": { "name": "hireDate" }
    },
    "expression": "$hireDate+1y"
  }
}
```

**Execution:**

```
User's hireDate: "2023-02-22"
Expression: $hireDate+1y
Add 1 year: 2023 + 1 = 2024
Output: "2024-02-22"
```

### Real Example #2: Calculate Contract End Date

**Scenario:** Contract lasts 2 years from hire date

```json
{
  "name": "ContractEndDate",
  "type": "dateMath",
  "attributes": {
    "input": {
      "type": "identityAttribute",
      "attributes": { "name": "hireDate" }
    },
    "expression": "$hireDate+2y"
  }
}
```

**Execution:**

```
hireDate: "2022-06-15"
Add 2 years: 2022 + 2 = 2024
Output: "2024-06-15"
```

### Real Example #3: Calculate 90-Day Mark

**Scenario:** Probation ends 90 days after hire

```json
{
  "name": "ProbationEndDate",
  "type": "dateMath",
  "attributes": {
    "input": {
      "type": "identityAttribute",
      "attributes": { "name": "hireDate" }
    },
    "expression": "$hireDate+90d"
  }
}
```

**Execution:**

```
hireDate: "2024-01-01"
Add 90 days: January 1 + 90 days = April 1
Output: "2024-04-01"
```

### Real Example #4: Work with Current Date

**Scenario:** Mark contract renewals that happen within next 30 days

```json
{
  "name": "UpcomingRenewal_Date",
  "type": "dateMath",
  "attributes": {
    "input": {
      "type": "static",
      "attributes": { "value": "now" }
    },
    "expression": "now+30d"
  }
}
```

**Execution:**

```
Today: "2024-02-22"
Add 30 days: February 22 + 30 = March 23
Output: "2024-03-23"

(Used to compare against contract renewal dates)
```

---

## Operation #3: DateCompare

### What It Does
Compares two dates and returns different values based on the comparison.

### JSON Structure
```json
{
  "type": "dateCompare",
  "attributes": {
    "firstDate": "first_date_to_compare",
    "secondDate": "second_date_to_compare",
    "operator": "comparison_operator",
    "positiveCondition": "return_if_true",
    "negativeCondition": "return_if_false"
  }
}
```

### Operators

| Operator | Meaning | Example |
|----------|---------|---------|
| `lt` | Less than (<) | hireDate < probationEndDate |
| `lte` | Less than or equal (<=) | hireDate <= probationEndDate |
| `gt` | Greater than (>) | hireDate > terminationDate |
| `gte` | Greater than or equal (>=) | hireDate >= startDate |

### Real Example #1: Identify Terminated Users >30 Days Ago

**Scenario:** If someone was terminated more than 30 days ago, they should be deprovisioned.

Deprovisioning should only happen for users where:
`terminationDate < (today - 30 days)`

```json
{
  "name": "ShouldDeprovision_Over30Days",
  "type": "dateCompare",
  "attributes": {
    "firstDate": {
      "type": "identityAttribute",
      "attributes": { "name": "terminationDate" }
    },
    "secondDate": {
      "type": "dateMath",
      "attributes": {
        "input": { "type": "static", "attributes": { "value": "now" } },
        "expression": "now-30d"
      }
    },
    "operator": "lt",
    "positiveCondition": "YES_DEPROVISION",
    "negativeCondition": "NO_KEEP_ACCESS"
  }
}
```

**How It Works:**

```
firstDate:   User's terminationDate (e.g., "2023-12-01")
secondDate:  30 days ago (today - 30d = 2024-01-23)
operator:    "lt" (less than)
Check:       Is 2023-12-01 < 2024-01-23?  YES
Output:      "YES_DEPROVISION"
```

**Scenarios:**

```
User 1: terminationDate = "2023-12-01" (over 30 days ago)
  Check: 2023-12-01 < 2024-01-23  →  YES
  Output: "YES_DEPROVISION" ✓

User 2: terminationDate = "2024-02-01" (recent)
  Check: 2024-02-01 < 2024-01-23  →  NO
  Output: "NO_KEEP_ACCESS" ✓

User 3: No terminationDate (active)
  Check: NULL < 2024-01-23  →  FALSE
  Output: "NO_KEEP_ACCESS" ✓
```

### Real Example #2: Identify New Hires (< 30 Days)

**Scenario:** If someone was hired less than 30 days ago, they're in ONBOARDING

```json
{
  "name": "Status_Onboarding_Check",
  "type": "dateCompare",
  "attributes": {
    "firstDate": {
      "type": "dateMath",
      "attributes": {
        "input": { "type": "static", "attributes": { "value": "now" } },
        "expression": "now-30d"
      }
    },
    "secondDate": {
      "type": "identityAttribute",
      "attributes": { "name": "hireDate" }
    },
    "operator": "lt",
    "positiveCondition": "ONBOARDING",
    "negativeCondition": "ACTIVE"
  }
}
```

**How It Works:**

```
firstDate:   30 days ago (today - 30d)
secondDate:  User's hireDate
operator:    "lt" (is 30-days-ago less than hireDate?)
             If YES, hireDate is AFTER 30 days ago (recent hire)

Check:       Is 2024-01-23 < 2024-02-15?  YES (hired recently)
Output:      "ONBOARDING"
```

### Real Example #3: Complex Lifecycle Status

**Scenario:** Multiple lifecycle stages:
- ONBOARDING if hired < 30 days ago
- NEW_EMPLOYEE if hired 30-90 days ago
- ACTIVE otherwise

```json
{
  "name": "EmployeeLifecycleStatus_Complex",
  "type": "dateCompare",
  "attributes": {
    "firstDate": {
      "type": "dateMath",
      "attributes": {
        "input": { "type": "static", "attributes": { "value": "now" } },
        "expression": "now-30d"
      }
    },
    "secondDate": {
      "type": "identityAttribute",
      "attributes": { "name": "hireDate" }
    },
    "operator": "lt",
    "positiveCondition": "ONBOARDING",
    "negativeCondition": {
      "type": "dateCompare",
      "attributes": {
        "firstDate": {
          "type": "dateMath",
          "attributes": {
            "input": { "type": "static", "attributes": { "value": "now" } },
            "expression": "now-90d"
          }
        },
        "secondDate": {
          "type": "identityAttribute",
          "attributes": { "name": "hireDate" }
        },
        "operator": "lt",
        "positiveCondition": "NEW_EMPLOYEE",
        "negativeCondition": "ACTIVE"
      }
    }
  }
}
```

**Execution Examples:**

```
User 1: hireDate = "2024-02-15" (8 days ago)
  First check: Is 2024-01-23 < 2024-02-15?  YES
  Output: "ONBOARDING" ✓

User 2: hireDate = "2023-11-15" (100 days ago)
  First check: Is 2024-01-23 < 2023-11-15?  NO
  Second check: Is 2023-12-23 < 2023-11-15?  NO
  Output: "ACTIVE" ✓

User 3: hireDate = "2023-12-01" (83 days ago)
  First check: Is 2024-01-23 < 2023-12-01?  NO
  Second check: Is 2023-12-23 < 2023-12-01?  NO, wait... 
               Is 2023-12-23 < 2023-12-01?  NO (December 23 is after December 1)
  Output: "ACTIVE" ✓

Wait, let me recalculate User 3:
Today: 2024-02-22
Hire: 2023-12-01 (83 days ago)
now-30d = 2024-01-23
now-90d = 2023-12-23
Check: Is 2024-01-23 < 2023-12-01? NO (Jan 23 is AFTER Dec 1)
Check: Is 2023-12-23 < 2023-12-01? NO (Dec 23 is AFTER Dec 1)
Output: "ACTIVE" ✓

Actually let me use a simpler example:
User 3: hireDate = "2024-01-05" (48 days ago)
now-30d = 2024-01-23
Check: Is 2024-01-23 < 2024-01-05? NO (Jan 23 is AFTER Jan 5)
now-90d = 2023-12-23
Check: Is 2023-12-23 < 2024-01-05? YES (Dec 23 is BEFORE Jan 5)
Output: "NEW_EMPLOYEE" ✓
```

---

## Date Operations in Practice: Real Workflow

**Complete Example: Employee Lifecycle Management**

```json
{
  "name": "EmployeeStatus_FullLifecycle",
  "type": "conditional",
  "attributes": {
    "expression": "$terminationDate != null",
    "positiveCondition": {
      "type": "dateCompare",
      "attributes": {
        "firstDate": {
          "type": "identityAttribute",
          "attributes": { "name": "terminationDate" }
        },
        "secondDate": {
          "type": "dateMath",
          "attributes": {
            "input": { "type": "static", "attributes": { "value": "now" } },
            "expression": "now-30d"
          }
        },
        "operator": "lt",
        "positiveCondition": "TERMINATED_ACTIVE",
        "negativeCondition": "TERMINATED_DEPROVISION"
      }
    },
    "negativeCondition": {
      "type": "dateCompare",
      "attributes": {
        "firstDate": {
          "type": "dateMath",
          "attributes": {
            "input": { "type": "static", "attributes": { "value": "now" } },
            "expression": "now-30d"
          }
        },
        "secondDate": {
          "type": "identityAttribute",
          "attributes": { "name": "hireDate" }
        },
        "operator": "lt",
        "positiveCondition": "ONBOARDING",
        "negativeCondition": "ACTIVE"
      }
    }
  }
}
```

**Decision Tree:**

```
Start
  │
  ├─ Has terminationDate?
  │   ├─ YES: Was terminated > 30 days ago?
  │   │   ├─ YES → "TERMINATED_DEPROVISION"
  │   │   └─ NO → "TERMINATED_ACTIVE"
  │   │
  │   └─ NO: Was hired < 30 days ago?
  │       ├─ YES → "ONBOARDING"
  │       └─ NO → "ACTIVE"
```

---

## Numeric Operations: Basics

Date operations often involve numeric comparisons. You use the same comparison operators:

### Simple Numeric Comparisons in Conditionals

```json
// Greater than
{
  "expression": "$salary > 50000",
  "positiveCondition": "HIGH_PAY",
  "negativeCondition": "REGULAR_PAY"
}

// Less than or equal
{
  "expression": "$yearsOfService <= 2",
  "positiveCondition": "JUNIOR",
  "negativeCondition": "SENIOR"
}

// Equals
{
  "expression": "$daysUntilReview == 0",
  "positiveCondition": "REVIEW_TODAY",
  "negativeCondition": "NOT_TODAY"
}
```

### Numeric Math in DateMath

You can also use numeric math in date expressions:

```json
// Add multiple units
{
  "expression": "$hireDate+1y+6m+15d"
}

// Result: hire date + 1 year + 6 months + 15 days
```

---

## Important: NULL Handling with Dates

Dates are often NULL (missing). Handle this explicitly.

### Mistake: Not Checking for NULL

```json
// WRONG: terminationDate might be NULL
{
  "type": "dateCompare",
  "attributes": {
    "firstDate": {
      "type": "identityAttribute",
      "attributes": { "name": "terminationDate" }
    },
    ...
  }
}
```

**If terminationDate is NULL:** Comparison fails or returns unexpected result.

### Correct: Check for NULL First

```json
// CORRECT: Check null first, then compare
{
  "type": "conditional",
  "attributes": {
    "expression": "$terminationDate != null",
    "positiveCondition": {
      "type": "dateCompare",
      "attributes": { ... }
    },
    "negativeCondition": "ACTIVE"
  }
}
```

---

## Date Operations Quick Reference

| Operation | Use Case | Example |
|-----------|----------|---------|
| **dateFormat** | Convert between formats | Convert "2020-01-15" to "01/15/2020" |
| **dateMath** | Calculate future/past dates | hireDate + 90 days = probation end |
| **dateCompare** | Compare two dates | Is terminationDate > 30 days ago? |
| **In Conditional** | Make date-based decisions | If hired < 30 days, status = ONBOARDING |

---

## Mastery Checkpoint #5

### Question 1: Format Conversion

What will this transform return for input "2020-01-15"?

```json
{
  "type": "dateFormat",
  "attributes": {
    "input": { "type": "static", "attributes": { "value": "2020-01-15" } },
    "inputFormat": "yyyy-MM-dd",
    "outputFormat": "MM/dd/yyyy"
  }
}
```

<details>
<summary>Answer</summary>

**"01/15/2020"**

Pattern "yyyy-MM-dd" interprets 2020=year, 01=month, 15=day
Pattern "MM/dd/yyyy" outputs month/day/year
Result: 01/15/2020

</details>

---

### Question 2: Date Math

What will this return for someone hired on "2024-01-01"?

```json
{
  "type": "dateMath",
  "attributes": {
    "input": { "type": "static", "attributes": { "value": "2024-01-01" } },
    "expression": "$input+90d"
  }
}
```

<details>
<summary>Answer</summary>

**"2024-04-01"** (January 1 + 90 days = April 1)

January has 31 days, February has 29 days (2024 is leap year), March has 31 days
31 + 29 + 31 = 91 days takes you to April 1

</details>

---

### Question 3: Date Compare

What will this return if terminationDate = "2023-12-01" and today is "2024-02-22"?

```json
{
  "type": "dateCompare",
  "attributes": {
    "firstDate": { "type": "static", "attributes": { "value": "2023-12-01" } },
    "secondDate": { "type": "static", "attributes": { "value": "2024-01-23" } },
    "operator": "lt",
    "positiveCondition": "DEPROVISION",
    "negativeCondition": "KEEP"
  }
}
```

<details>
<summary>Answer</summary>

**"DEPROVISION"**

Check: Is 2023-12-01 < 2024-01-23? YES (December 1 is before January 23)
Output: "DEPROVISION" ✓

(This identifies users terminated more than 30 days ago)

</details>

---

### Question 4: Real Scenario

Build a dateCompare that returns:
- "ONBOARDING" if hired less than 30 days ago
- "ACTIVE" otherwise

(Today is 2024-02-22)

<details>
<summary>Answer</summary>

```json
{
  "name": "OnboardingStatus_Check",
  "type": "dateCompare",
  "attributes": {
    "firstDate": {
      "type": "dateMath",
      "attributes": {
        "input": { "type": "static", "attributes": { "value": "now" } },
        "expression": "now-30d"
      }
    },
    "secondDate": {
      "type": "identityAttribute",
      "attributes": { "name": "hireDate" }
    },
    "operator": "lt",
    "positiveCondition": "ONBOARDING",
    "negativeCondition": "ACTIVE"
  }
}
```

**How it works:**
- firstDate: now-30d = 2024-01-23
- Check: Is 2024-01-23 < hireDate?
- If YES (hireDate is after 2024-01-23): "ONBOARDING"
- If NO (hireDate is before 2024-01-23): "ACTIVE"

</details>

---

## Summary: Date and Numeric Operations

| Concept | Purpose |
|---------|---------|
| **dateFormat** | Convert dates between different formats |
| **inputFormat** | Pattern describing input date |
| **outputFormat** | Pattern describing desired output |
| **dateMath** | Add/subtract time from dates |
| **now** | Current date/time reference |
| **dateCompare** | Compare two dates, return values based on result |
| **Operators** | lt, lte, gt, gte for comparisons |
| **NULL Handling** | Always check if date exists before comparing |

---

## What's Next

In **Lesson 6**, you'll learn:
- ✅ Reference operations (lookup, find, reuse)
- ✅ Generator operations (create unique usernames, passwords)
- ✅ Encoding operations (Base64)

These are the specialized operations for specific needs.

---

## Homework Before Lesson 6

1. For your organization:
   - When do people start? (hireDate)
   - When should they be reviewed? (calculate)
   - When should they be deprovisioned after termination? (calculate)

2. Design dateCompare operations for each

3. Think about: What happens if hireDate is NULL?

**Next: Reference and Generator Operations** — specialized tools for specific problems.
