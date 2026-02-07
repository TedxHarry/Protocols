# DAY 4: DATE/TIME & LIST OPERATIONS - MORNING SESSION

## Table of Contents
- [Overview](#overview)
- [What You'll Learn Today](#what-youll-learn-today)
- [Understanding Dates in SailPoint](#understanding-dates-in-sailpoint)
- [Transform 1: Date Format](#transform-1-date-format-30-minutes)
- [Transform 2: Date Math](#transform-2-date-math-30-minutes)
- [Transform 3: Date Compare](#transform-3-date-compare-45-minutes)
- [Combining Date Transforms](#combining-date-transforms-15-minutes)
- [Morning Session Review](#morning-session-review)
- [Checkpoint Quiz](#checkpoint-quiz)

---

## Overview

**Session Duration:** 2 hours

**Prerequisites:** 
- Completed Days 1-3
- Understand transform chaining
- Comfortable with Conditional transforms
- Understand FirstValid

**Goal:** Master date/time transforms and build proper lifecycle state logic

---

## What You'll Learn Today

By the end of this morning session, you'll be able to:

- ✅ Understand ISO 8601 date format requirements
- ✅ Convert between different date formats
- ✅ Perform date arithmetic (add/subtract days, months, years)
- ✅ Compare dates for decision making
- ✅ Build proper lifecycle state calculations
- ✅ Handle timezone considerations
- ✅ Work with date-based conditionals

---

## Understanding Dates in SailPoint

### The Critical Requirement: ISO 8601 Format

SailPoint IdentityNow requires dates in **ISO 8601 format** for any date operations:
```
ISO 8601 Format: YYYY-MM-DDTHH:MM:SSZ

Examples:
  2024-01-15T00:00:00Z
  2024-12-31T23:59:59Z
  2026-06-01T06:00:00Z
```

#### Format Breakdown
```
2024-01-15T00:00:00Z
│    │  │  │ │  │  │ │
│    │  │  │ │  │  │ └─ Timezone indicator (Z = UTC)
│    │  │  │ │  │  └─── Seconds (00-59)
│    │  │  │ │  └────── Minutes (00-59)
│    │  │  │ └───────── Hours (00-23)
│    │  │  └─────────── Separator (literal "T")
│    │  └────────────── Day (01-31)
│    └───────────────── Month (01-12)
└────────────────────── Year (4 digits)
```

---

### Common Source Date Formats

Your source systems rarely provide ISO 8601 format:

| Source System | Format | Example |
|---------------|--------|---------|
| Workday | MM/dd/yyyy | 01/15/2024 |
| PeopleSoft | dd-MMM-yyyy | 15-Jan-2024 |
| Active Directory | yyyyMMdd | 20240115 |
| SAP | dd.MM.yyyy | 15.01.2024 |
| Custom CSV | yyyy-MM-dd | 2024-01-15 |

**You must convert these to ISO 8601 before any date operations!**

---

### Why This Matters
```
If you try date comparison WITHOUT proper format:

Source: hire_date = "01/15/2024"

Conditional: hire_date > today
Result: ERROR or unexpected behavior

Why: String comparison, not date comparison
  "01/15/2024" compared as string to "2024-02-06"
  String "0" < String "2", so appears "less than" (wrong!)
```

**The fix:**
```
Transform chain:
1. Date Format: "01/15/2024" --> "2024-01-15T00:00:00Z"
2. Date Compare: Now properly compares dates
```

---

## Transform 1: Date Format (30 minutes)

### What Date Format Does

**Date Format** converts dates from one format to another.

#### Basic Usage
```
INPUT: Date in source format
TRANSFORM: Date Format
  Input Format: MM/dd/yyyy
  Output Format: ISO8601
OUTPUT: 2024-01-15T00:00:00Z
```

---

### Exercise 1A: Convert MM/dd/yyyy to ISO 8601

**Scenario:** Workday provides hire_date as "01/15/2024", convert to ISO 8601.

#### UI Style: Steps
```
1. Go to Identity Profile > Mappings
2. Create attribute "hireDateISO"
3. Source: Transform
4. Transform: Date Format
5. Configuration:

   Input Value: hire_date (source attribute)
   Input Format: MM/dd/yyyy
   Output Format: ISO8601

6. Preview with identities
7. Save
```

#### JSON Style: Transform Definition
```json
{
  "name": "hireDateISO",
  "type": "dateFormat",
  "attributes": {
    "input": {
      "type": "identityAttribute",
      "attributes": {
        "name": "hire_date"
      }
    },
    "inputFormat": "MM/dd/yyyy",
    "outputFormat": "ISO8601"
  }
}
```

---

#### Test Cases
```
Test Case 1:
  Input: hire_date = "01/15/2024"
  Output: "2024-01-15T00:00:00Z"

Test Case 2:
  Input: hire_date = "12/31/2023"
  Output: "2023-12-31T00:00:00Z"

Test Case 3:
  Input: hire_date = null
  Output: null

Test Case 4:
  Input: hire_date = "13/45/2024" (invalid date)
  Output: Error or null (depends on ISC version)
```

---

### Exercise 1B: Convert dd-MMM-yyyy to ISO 8601

**Scenario:** PeopleSoft provides term_date as "15-Jan-2024".

#### UI Style: Steps
```
1. Create attribute "termDateISO"
2. Transform: Date Format
3. Configuration:

   Input Value: term_date
   Input Format: dd-MMM-yyyy
   Output Format: ISO8601

4. Preview
5. Save
```

#### JSON Style: Transform Definition
```json
{
  "name": "termDateISO",
  "type": "dateFormat",
  "attributes": {
    "input": {
      "type": "identityAttribute",
      "attributes": {
        "name": "term_date"
      }
    },
    "inputFormat": "dd-MMM-yyyy",
    "outputFormat": "ISO8601"
  }
}
```

---

#### Test Cases
```
Test Case 1:
  Input: term_date = "15-Jan-2024"
  Output: "2024-01-15T00:00:00Z"

Test Case 2:
  Input: term_date = "31-Dec-2024"
  Output: "2024-12-31T00:00:00Z"
```

---

### Exercise 1C: Convert yyyyMMdd to ISO 8601

**Scenario:** Active Directory stores dates as "20240115".

#### UI Style: Steps
```
1. Create attribute "adDateISO"
2. Transform: Date Format
3. Configuration:

   Input Value: ad_date_field
   Input Format: yyyyMMdd
   Output Format: ISO8601

4. Preview
5. Save
```

#### JSON Style: Transform Definition
```json
{
  "name": "adDateISO",
  "type": "dateFormat",
  "attributes": {
    "input": {
      "type": "identityAttribute",
      "attributes": {
        "name": "ad_date_field"
      }
    },
    "inputFormat": "yyyyMMdd",
    "outputFormat": "ISO8601"
  }
}
```

---

#### Common Date Format Patterns

| Pattern | Example | Meaning |
|---------|---------|---------|
| yyyy | 2024 | 4-digit year |
| MM | 01 | 2-digit month (01-12) |
| dd | 15 | 2-digit day (01-31) |
| MMM | Jan | 3-letter month name |
| HH | 23 | 2-digit hour (00-23) |
| mm | 59 | 2-digit minute |
| ss | 59 | 2-digit second |

---

### Key Learning Points

- ✅ Date Format converts between formats
- ✅ ISO8601 required for date operations
- ✅ Must specify both input and output formats
- ✅ Invalid dates return null or error
- ✅ Case matters: MM = month, mm = minute
- ✅ Always convert source dates to ISO8601 first

---

## Transform 2: Date Math (30 minutes)

### What Date Math Does

**Date Math** adds or subtracts time from a date.

#### Basic Usage
```
INPUT: 2024-01-15T00:00:00Z
TRANSFORM: Date Math
  Operation: Add 90 days
OUTPUT: 2024-04-14T00:00:00Z
```

---

### Date Math Operations

| Operation | Example | Result |
|-----------|---------|--------|
| Add days | +90 days | 90 days later |
| Subtract days | -30 days | 30 days earlier |
| Add months | +3 months | 3 months later |
| Add years | +1 year | 1 year later |
| Subtract years | -5 years | 5 years earlier |

---

### Exercise 2A: Calculate Probation End Date

**Scenario:** Probation ends 90 days after hire date.

#### UI Style: Steps
```
1. First, ensure hire_date is in ISO8601 format
   (Use hireDateISO from Exercise 1A)

2. Create attribute "probationEndDate"
3. Transform: Date Math
4. Configuration:

   Input Date: hireDateISO
   Operation: Add
   Amount: 90
   Unit: days

5. Preview
6. Save
```

#### JSON Style: Transform Definition
```json
{
  "name": "probationEndDate",
  "type": "dateMath",
  "attributes": {
    "input": {
      "type": "identityAttribute",
      "attributes": {
        "name": "hireDateISO"
      }
    },
    "expression": "+90d"
  }
}
```

---

#### Test Cases
```
Test Case 1:
  Input: hireDateISO = "2024-01-15T00:00:00Z"
  Operation: Add 90 days
  Output: "2024-04-14T00:00:00Z"

Test Case 2:
  Input: hireDateISO = "2024-12-01T00:00:00Z"
  Operation: Add 90 days
  Output: "2025-02-28T00:00:00Z" (crosses year boundary!)

Test Case 3:
  Input: hireDateISO = null
  Output: null
```

---

### Exercise 2B: Calculate Account Expiration (1 Year Later)

**Scenario:** Contractor accounts expire 1 year after hire date.

#### UI Style: Steps
```
1. Create attribute "accountExpirationDate"
2. Transform: Date Math
3. Configuration:

   Input Date: hireDateISO
   Operation: Add
   Amount: 1
   Unit: years

4. Preview
5. Save
```

#### JSON Style: Transform Definition
```json
{
  "name": "accountExpirationDate",
  "type": "dateMath",
  "attributes": {
    "input": {
      "type": "identityAttribute",
      "attributes": {
        "name": "hireDateISO"
      }
    },
    "expression": "+1y"
  }
}
```

---

#### Test Cases
```
Test Case 1:
  Input: "2024-01-15T00:00:00Z"
  Operation: Add 1 year
  Output: "2025-01-15T00:00:00Z"

Test Case 2:
  Input: "2024-02-29T00:00:00Z" (leap year)
  Operation: Add 1 year
  Output: "2025-02-28T00:00:00Z" (2025 not leap year)
```

---

### Exercise 2C: Calculate 90 Days Ago (for Archive Logic)

**Scenario:** Archive identities if terminated more than 90 days ago.

#### UI Style: Steps
```
1. Create attribute "ninetyDaysAgo"
2. Transform: Date Math
3. Configuration:

   Input Date: [Current Date - we'll use a reference]
   Operation: Subtract
   Amount: 90
   Unit: days

4. Save
```

> **Note:** Getting "current date" in transforms requires special handling. Some ISC versions have a "now" or "today" reference. If not available, you may need to use a rule or scheduled processing.

For this exercise, understand the concept: subtracting days moves backwards in time.

#### JSON Style: Transform Definition
```json
{
  "name": "ninetyDaysAgo",
  "type": "dateMath",
  "attributes": {
    "input": {
      "type": "now"
    },
    "expression": "-90d"
  }
}
```

---

### Exercise 2D: Add Time Component to Date

**Scenario:** Set processing time to 6:00 AM on hire date.

#### UI Style: Steps
```
1. Create attribute "hireDateWithTime"
2. Transform chain:

   Transform 1: Date Format
     Input: hire_date
     Input Format: MM/dd/yyyy
     Output Format: ISO8601
     Output: "2024-01-15T00:00:00Z"
   
   Transform 2: Date Math
     Operation: Add
     Amount: 6
     Unit: hours
     Output: "2024-01-15T06:00:00Z"

3. Save
```

#### JSON Style: Transform Definition
```json
{
  "name": "hireDateWithTime",
  "type": "dateFormat",
  "attributes": {
    "input": {
      "type": "identityAttribute",
      "attributes": {
        "name": "hire_date"
      }
    },
    "inputFormat": "MM/dd/yyyy",
    "outputFormat": "ISO8601",
    "next": {
      "type": "dateMath",
      "attributes": {
        "expression": "+6h"
      }
    }
  }
}
```

---

### Key Learning Points

- ✅ Date Math adds or subtracts time
- ✅ Input must be ISO8601 format
- ✅ Can add/subtract days, months, years, hours
- ✅ Handles month/year boundaries correctly
- ✅ Handles leap years automatically
- ✅ Null input returns null output

---

## Transform 3: Date Compare (45 minutes)

### What Date Compare Does

**Date Compare** compares two dates and returns true/false.

#### Basic Usage
```
INPUT: 
  Date 1: 2024-01-15T00:00:00Z
  Date 2: 2024-02-06T00:00:00Z (today)

TRANSFORM: Date Compare
  Operation: Greater Than (Date 1 > Date 2)

OUTPUT: false (Jan 15 is NOT greater than Feb 6)
```

---

### Date Compare Operations

| Operation | Symbol | When True |
|-----------|--------|-----------|
| Greater Than | > | Date 1 is after Date 2 |
| Less Than | < | Date 1 is before Date 2 |
| Equals | = | Dates are the same |
| Greater Than or Equal | >= | Date 1 is after or same as Date 2 |
| Less Than or Equal | <= | Date 1 is before or same as Date 2 |

---

### Exercise 3A: Check if Hire Date is in Future

**Scenario:** Determine if employee is pre-hire (hire date > today).

#### UI Style: Steps
```
1. Create attribute "isPreHire"
2. Transform chain:

   Transform 1: Date Compare
     First Date: hireDateISO
     Comparison: Greater Than
     Second Date: [Current Date/Today]
     Output: true or false
   
   (Configuration varies by ISC version for "today" reference)

3. Preview
4. Save
```

> **Note:** Getting current date varies by ISC version:
> - Some have "now" or "today" built-in reference
> - Some require using a special transform
> - For this exercise, understand the concept

#### JSON Style: Transform Definition
```json
{
  "name": "isPreHire",
  "type": "dateCompare",
  "attributes": {
    "firstDate": {
      "type": "identityAttribute",
      "attributes": {
        "name": "hireDateISO"
      }
    },
    "secondDate": {
      "type": "now"
    },
    "operator": "gt",
    "positiveCondition": "true",
    "negativeCondition": "false"
  }
}
```

---

#### Test Cases
```
Assume today is: 2024-02-06

Test Case 1:
  hireDateISO = "2024-06-01T00:00:00Z" (future)
  Compare: 2024-06-01 > 2024-02-06
  Output: true (is pre-hire)

Test Case 2:
  hireDateISO = "2024-01-15T00:00:00Z" (past)
  Compare: 2024-01-15 > 2024-02-06
  Output: false (not pre-hire)

Test Case 3:
  hireDateISO = "2024-02-06T00:00:00Z" (today)
  Compare: 2024-02-06 > 2024-02-06
  Output: false (equal, not greater)
```

---

### Exercise 3B: Check if Terminated

**Scenario:** Determine if employee is terminated (term_date exists and <= today).

#### UI Style: Steps
```
1. Create attribute "isTerminated"
2. Transform chain:

   Transform 1: Conditional (check if term_date exists)
     Condition: termDateISO ne null
     If True: Continue to date compare
     If False: Return false
   
   Transform 2: Date Compare
     First Date: termDateISO
     Comparison: Less Than or Equal
     Second Date: [Today]
     Output: true or false

3. Save
```

#### JSON Style: Transform Definition
```json
{
  "name": "isTerminated",
  "type": "conditional",
  "attributes": {
    "expression": "termDateISO ne null",
    "positiveCondition": {
      "type": "dateCompare",
      "attributes": {
        "firstDate": {
          "type": "identityAttribute",
          "attributes": {
            "name": "termDateISO"
          }
        },
        "secondDate": {
          "type": "now"
        },
        "operator": "lte",
        "positiveCondition": "true",
        "negativeCondition": "false"
      }
    },
    "negativeCondition": "false"
  }
}
```

---

#### Test Cases
```
Assume today is: 2024-02-06

Test Case 1:
  termDateISO = "2024-01-15T00:00:00Z" (past)
  Compare: 2024-01-15 <= 2024-02-06
  Output: true (is terminated)

Test Case 2:
  termDateISO = "2024-03-15T00:00:00Z" (future)
  Compare: 2024-03-15 <= 2024-02-06
  Output: false (not yet terminated)

Test Case 3:
  termDateISO = null
  Output: false (no term date = not terminated)
```

---

### Exercise 3C: Check if Account Should be Archived

**Scenario:** Archive if terminated more than 90 days ago.

#### UI Style: Steps
```
1. Create helper attribute: ninetyDaysAgoDate
   Transform: Date Math
     Input: [Today]
     Operation: Subtract 90 days
     Output: Date 90 days ago

2. Create attribute "shouldBeArchived"
3. Transform chain:

   Transform 1: Conditional (check term_date exists)
     Condition: termDateISO ne null
     If True: Continue
     If False: Return false
   
   Transform 2: Date Compare
     First Date: termDateISO
     Comparison: Less Than or Equal
     Second Date: ninetyDaysAgoDate
     Output: true or false

3. Save
```

#### JSON Style: Helper Attribute
```json
{
  "name": "ninetyDaysAgoDate",
  "type": "dateMath",
  "attributes": {
    "input": {
      "type": "now"
    },
    "expression": "-90d"
  }
}
```

#### JSON Style: Final Attribute
```json
{
  "name": "shouldBeArchived",
  "type": "conditional",
  "attributes": {
    "expression": "termDateISO ne null",
    "positiveCondition": {
      "type": "dateCompare",
      "attributes": {
        "firstDate": {
          "type": "identityAttribute",
          "attributes": {
            "name": "termDateISO"
          }
        },
        "secondDate": {
          "type": "identityAttribute",
          "attributes": {
            "name": "ninetyDaysAgoDate"
          }
        },
        "operator": "lte",
        "positiveCondition": "true",
        "negativeCondition": "false"
      }
    },
    "negativeCondition": "false"
  }
}
```

---

#### How It Works
```
Assume today is: 2024-02-06
90 days ago = 2023-11-08

Test Case 1:
  termDateISO = "2023-10-01T00:00:00Z" (terminated 4 months ago)
  Compare: 2023-10-01 <= 2023-11-08
  Output: true (should archive)

Test Case 2:
  termDateISO = "2023-12-01T00:00:00Z" (terminated 2 months ago)
  Compare: 2023-12-01 <= 2023-11-08
  Output: false (too recent to archive)

Test Case 3:
  termDateISO = null
  Output: false (not terminated)
```

---

### Exercise 3D: Compare Two Date Fields

**Scenario:** Check if end_date is after start_date (validation).

#### UI Style: Steps
```
1. Create attribute "isValidDateRange"
2. Transform chain:

   Transform 1: Conditional (both dates exist)
     Condition: startDateISO ne null AND endDateISO ne null
     If True: Continue
     If False: Return true (no dates = valid by default)
   
   Transform 2: Date Compare
     First Date: endDateISO
     Comparison: Greater Than
     Second Date: startDateISO
     Output: true or false

3. Save
```

#### JSON Style: Transform Definition
```json
{
  "name": "isValidDateRange",
  "type": "conditional",
  "attributes": {
    "expression": "startDateISO ne null && endDateISO ne null",
    "positiveCondition": {
      "type": "dateCompare",
      "attributes": {
        "firstDate": {
          "type": "identityAttribute",
          "attributes": {
            "name": "endDateISO"
          }
        },
        "secondDate": {
          "type": "identityAttribute",
          "attributes": {
            "name": "startDateISO"
          }
        },
        "operator": "gt",
        "positiveCondition": "true",
        "negativeCondition": "false"
      }
    },
    "negativeCondition": "true"
  }
}
```

---

#### Test Cases
```
Test Case 1:
  startDateISO = "2024-01-15T00:00:00Z"
  endDateISO = "2024-12-31T00:00:00Z"
  Compare: 2024-12-31 > 2024-01-15
  Output: true (valid range)

Test Case 2:
  startDateISO = "2024-12-31T00:00:00Z"
  endDateISO = "2024-01-15T00:00:00Z"
  Compare: 2024-01-15 > 2024-12-31
  Output: false (invalid - end before start)

Test Case 3:
  startDateISO = null
  endDateISO = "2024-12-31T00:00:00Z"
  Output: true (no start date = valid by default)
```

---

### Key Learning Points

- ✅ Date Compare returns true/false
- ✅ Useful in Conditional transforms
- ✅ Both dates must be ISO8601 format
- ✅ Always check for null dates first
- ✅ Can compare to current date or other fields
- ✅ Supports >, <, =, >=, <= operations

---

## Combining Date Transforms (15 minutes)

### Real-World Pattern: Lifecycle State with Proper Dates

Now let's build a PROPER lifecycle state calculator using date transforms!

#### Business Logic
```
States: Pre-hire, Active, Leave of Absence, Terminated, Archived

Logic:
- Leave of Absence: leave_flag = true
- Pre-hire: hire_date > today
- Archived: term_date <= (today - 90 days)
- Terminated: term_date <= today AND term_date > (today - 90 days)
- Active: hire_date <= today AND (term_date null OR term_date > today)
```

---

### Step 1: Prepare Date Fields

#### UI Style: Steps
```
1. Attribute: hireDateISO
   Transform: Date Format
     Input: hire_date
     Input Format: MM/dd/yyyy
     Output Format: ISO8601

2. Attribute: termDateISO
   Transform: Date Format
     Input: term_date
     Input Format: MM/dd/yyyy
     Output Format: ISO8601

3. Attribute: ninetyDaysAgo
   Transform: Date Math
     Input: [Today]
     Operation: Subtract 90 days
```

#### JSON Style: Transform Definitions

**hireDateISO:**
```json
{
  "name": "hireDateISO",
  "type": "dateFormat",
  "attributes": {
    "input": {
      "type": "identityAttribute",
      "attributes": {
        "name": "hire_date"
      }
    },
    "inputFormat": "MM/dd/yyyy",
    "outputFormat": "ISO8601"
  }
}
```

**termDateISO:**
```json
{
  "name": "termDateISO",
  "type": "dateFormat",
  "attributes": {
    "input": {
      "type": "identityAttribute",
      "attributes": {
        "name": "term_date"
      }
    },
    "inputFormat": "MM/dd/yyyy",
    "outputFormat": "ISO8601"
  }
}
```

**ninetyDaysAgo:**
```json
{
  "name": "ninetyDaysAgo",
  "type": "dateMath",
  "attributes": {
    "input": {
      "type": "now"
    },
    "expression": "-90d"
  }
}
```

---

### Step 2: Build Lifecycle State Logic

#### UI Style: Steps
```
Attribute: lifecycleState
Transform chain:

LEVEL 1: Check Leave of Absence
Transform 1: Conditional
  Condition: leave_flag eq true
  If True: "Leave of Absence"
  If False: Continue

LEVEL 2: Check Pre-hire
Transform 2: Conditional
  Condition: [Date Compare: hireDateISO > today]
  If True: "Pre-hire"
  If False: Continue

LEVEL 3: Check Archived
Transform 3: Conditional
  Condition: termDateISO ne null AND [Date Compare: termDateISO <= ninetyDaysAgo]
  If True: "Archived"
  If False: Continue

LEVEL 4: Check Terminated
Transform 4: Conditional
  Condition: termDateISO ne null AND [Date Compare: termDateISO <= today]
  If True: "Terminated"
  If False: "Active"
```

> **Note:** Exact syntax for embedding Date Compare in Conditional varies by ISC version. You may need to create helper attributes for each date comparison result.

---

### Better Approach: Helper Attributes

Create boolean helpers for clarity:

#### UI Style: Create Helpers
```
Helper 1: isHireDateFuture
  Transform: Date Compare (hireDateISO > today)
  Output: true/false

Helper 2: isTermDatePast
  Transform: Date Compare (termDateISO <= today)
  Output: true/false

Helper 3: isTermDateOld
  Transform: Date Compare (termDateISO <= ninetyDaysAgo)
  Output: true/false

Final: lifecycleState
  Conditional chain using these helpers:
  - If leave_flag = true --> "Leave of Absence"
  - If isHireDateFuture = true --> "Pre-hire"
  - If termDateISO ne null AND isTermDateOld = true --> "Archived"
  - If termDateISO ne null AND isTermDatePast = true --> "Terminated"
  - Else --> "Active"
```

#### JSON Style: Helper Attributes

**isHireDateFuture:**
```json
{
  "name": "isHireDateFuture",
  "type": "dateCompare",
  "attributes": {
    "firstDate": {
      "type": "identityAttribute",
      "attributes": {
        "name": "hireDateISO"
      }
    },
    "secondDate": {
      "type": "now"
    },
    "operator": "gt",
    "positiveCondition": "true",
    "negativeCondition": "false"
  }
}
```

**isTermDatePast:**
```json
{
  "name": "isTermDatePast",
  "type": "conditional",
  "attributes": {
    "expression": "termDateISO ne null",
    "positiveCondition": {
      "type": "dateCompare",
      "attributes": {
        "firstDate": {
          "type": "identityAttribute",
          "attributes": {
            "name": "termDateISO"
          }
        },
        "secondDate": {
          "type": "now"
        },
        "operator": "lte",
        "positiveCondition": "true",
        "negativeCondition": "false"
      }
    },
    "negativeCondition": "false"
  }
}
```

**isTermDateOld:**
```json
{
  "name": "isTermDateOld",
  "type": "conditional",
  "attributes": {
    "expression": "termDateISO ne null",
    "positiveCondition": {
      "type": "dateCompare",
      "attributes": {
        "firstDate": {
          "type": "identityAttribute",
          "attributes": {
            "name": "termDateISO"
          }
        },
        "secondDate": {
          "type": "identityAttribute",
          "attributes": {
            "name": "ninetyDaysAgo"
          }
        },
        "operator": "lte",
        "positiveCondition": "true",
        "negativeCondition": "false"
      }
    },
    "negativeCondition": "false"
  }
}
```

**Final lifecycleState:**
```json
{
  "name": "lifecycleState",
  "type": "conditional",
  "attributes": {
    "expression": "leave_flag eq true",
    "positiveCondition": "Leave of Absence",
    "negativeCondition": {
      "type": "conditional",
      "attributes": {
        "expression": "isHireDateFuture eq \"true\"",
        "positiveCondition": "Pre-hire",
        "negativeCondition": {
          "type": "conditional",
          "attributes": {
            "expression": "isTermDateOld eq \"true\"",
            "positiveCondition": "Archived",
            "negativeCondition": {
              "type": "conditional",
              "attributes": {
                "expression": "isTermDatePast eq \"true\"",
                "positiveCondition": "Terminated",
                "negativeCondition": "Active"
              }
            }
          }
        }
      }
    }
  }
}
```

---

### Test Complete Lifecycle State
```
Assume today = 2024-02-06
90 days ago = 2023-11-08

Test Case 1: Pre-hire
  leave_flag = false
  hire_date = "06/01/2024"
  hireDateISO = "2024-06-01T00:00:00Z"
  isHireDateFuture = true
  Output: "Pre-hire"

Test Case 2: Active
  leave_flag = false
  hire_date = "01/15/2024"
  term_date = null
  Output: "Active"

Test Case 3: Terminated (recent)
  leave_flag = false
  hire_date = "01/15/2023"
  term_date = "12/15/2023"
  termDateISO = "2023-12-15T00:00:00Z"
  isTermDatePast = true
  isTermDateOld = false (Dec 15 is after Nov 08)
  Output: "Terminated"

Test Case 4: Archived
  leave_flag = false
  hire_date = "01/15/2020"
  term_date = "10/01/2023"
  termDateISO = "2023-10-01T00:00:00Z"
  isTermDatePast = true
  isTermDateOld = true (Oct 1 is before Nov 08)
  Output: "Archived"

Test Case 5: Leave of Absence
  leave_flag = true
  (all other fields ignored)
  Output: "Leave of Absence"
```

---

### Key Learning Points

- ✅ Combine Date Format + Date Math + Date Compare
- ✅ Always convert to ISO8601 first
- ✅ Use helper attributes for complex logic
- ✅ Date comparisons enable proper lifecycle states
- ✅ Check for null dates before comparing
- ✅ Real date logic replaces simplified string checks

---

## Morning Session Review

### Transforms Learned

| Transform | Purpose | Key Use Cases |
|-----------|---------|---------------|
| **Date Format** | Convert date formats | Source format to ISO8601 |
| **Date Math** | Add/subtract time | Expiration dates, probation, archival |
| **Date Compare** | Compare two dates | Pre-hire check, termination check |

---

### What You Built

- [ ] Date format conversions (3 formats)
- [ ] Probation end date calculator
- [ ] Account expiration date
- [ ] 90 days ago calculator
- [ ] Pre-hire checker
- [ ] Termination checker
- [ ] Archive checker
- [ ] Date range validator
- [ ] Complete lifecycle state with real dates

**Total: 9+ date-based transforms!**

---

### Key Concepts Mastered

- ✅ ISO 8601 format requirement
- ✅ Converting source dates to ISO8601
- ✅ Common date format patterns
- ✅ Date arithmetic operations
- ✅ Date comparison operators
- ✅ Combining date transforms with conditionals
- ✅ Helper attributes for date logic
- ✅ Null date handling
- ✅ Production-ready lifecycle state calculation
- ✅ Both UI and JSON configuration

---

## Checkpoint Quiz

### Question 1: ISO 8601 Format

Which of these is valid ISO 8601 format?

A) `01/15/2024`  
B) `2024-01-15`  
C) `2024-01-15T00:00:00Z`  
D) `15-Jan-2024`

<details>
<summary>Click to reveal answer</summary>

**Answer: C) 2024-01-15T00:00:00Z**

**Why:**
- A) MM/dd/yyyy (common source format, not ISO)
- B) Missing time component (yyyy-MM-dd only)
- C) CORRECT - Full ISO 8601 with date, time, timezone
- D) dd-MMM-yyyy (another source format)

**Key:** ISO 8601 requires: YYYY-MM-DDTHH:MM:SSZ

</details>

---

### Question 2: Date Math
```
Input: "2024-01-15T00:00:00Z"
Operation: Add 90 days

What is the output?
```

<details>
<summary>Click to reveal answer</summary>

**Answer: 2024-04-14T00:00:00Z**

**Calculation:**
- January: 15 + 16 days left = 31 total (16 days used)
- February: 29 days (2024 is leap year) - 16 = 13 days remaining, 74 days left to add
- March: 31 days - 74 = end of March (43 days left)
- April: 43 days into April = April 14

**Actually:** Let the transform calculate it! Just know that:
- It handles month boundaries
- It handles leap years
- 90 days from Jan 15 lands in mid-April

</details>

---

### Question 3: Date Compare Logic
```
Today: 2024-02-06
hire_date: "2024-06-01T00:00:00Z"

Date Compare: hire_date > today
Result: ?
```

<details>
<summary>Click to reveal answer</summary>

**Answer: true**

**Why:**
- June 1, 2024 is AFTER February 6, 2024
- 2024-06-01 > 2024-02-06 = true
- This person is PRE-HIRE (hire date in future)

</details>

---

### Question 4: Building Lifecycle State

Build lifecycle state logic:
```
If hire_date > today --> "Pre-hire"
If term_date <= today --> "Terminated"
Otherwise --> "Active"

What transforms do you need?
```

<details>
<summary>Click to reveal answer</summary>

**Answer:**

**Step 1: Format dates**

**JSON Style:**
```json
{
  "name": "hireDateISO",
  "type": "dateFormat",
  "attributes": {
    "input": {
      "type": "identityAttribute",
      "attributes": {
        "name": "hire_date"
      }
    },
    "inputFormat": "MM/dd/yyyy",
    "outputFormat": "ISO8601"
  }
}
```
```json
{
  "name": "termDateISO",
  "type": "dateFormat",
  "attributes": {
    "input": {
      "type": "identityAttribute",
      "attributes": {
        "name": "term_date"
      }
    },
    "inputFormat": "MM/dd/yyyy",
    "outputFormat": "ISO8601"
  }
}
```

**Step 2: Create date comparison helpers**
```json
{
  "name": "isPreHire",
  "type": "dateCompare",
  "attributes": {
    "firstDate": {
      "type": "identityAttribute",
      "attributes": {
        "name": "hireDateISO"
      }
    },
    "secondDate": {
      "type": "now"
    },
    "operator": "gt",
    "positiveCondition": "true",
    "negativeCondition": "false"
  }
}
```
```json
{
  "name": "isTerminated",
  "type": "conditional",
  "attributes": {
    "expression": "termDateISO ne null",
    "positiveCondition": {
      "type": "dateCompare",
      "attributes": {
        "firstDate": {
          "type": "identityAttribute",
          "attributes": {
            "name": "termDateISO"
          }
        },
        "secondDate": {
          "type": "now"
        },
        "operator": "lte",
        "positiveCondition": "true",
        "negativeCondition": "false"
      }
    },
    "negativeCondition": "false"
  }
}
```

**Step 3: Build lifecycle state**
```json
{
  "name": "lifecycleState",
  "type": "conditional",
  "attributes": {
    "expression": "isPreHire eq \"true\"",
    "positiveCondition": "Pre-hire",
    "negativeCondition": {
      "type": "conditional",
      "attributes": {
        "expression": "isTerminated eq \"true\"",
        "positiveCondition": "Terminated",
        "negativeCondition": "Active"
      }
    }
  }
}
```

</details>

---

### Question 5: What's Wrong?
```
Source: hire_date = "01/15/2024"

Transform: Date Compare
  hire_date > today
  
Result: ERROR
```

What's the problem?

<details>
<summary>Click to reveal answer</summary>

**Problem:** hire_date is NOT in ISO 8601 format!

Source provides: "01/15/2024" (MM/dd/yyyy)  
Date Compare requires: ISO 8601 format

**Fix:**

**JSON Style:**
```json
{
  "name": "hireDateISO",
  "type": "dateFormat",
  "attributes": {
    "input": {
      "type": "identityAttribute",
      "attributes": {
        "name": "hire_date"
      }
    },
    "inputFormat": "MM/dd/yyyy",
    "outputFormat": "ISO8601",
    "next": {
      "type": "dateCompare",
      "attributes": {
        "firstDate": {
          "type": "reference"
        },
        "secondDate": {
          "type": "now"
        },
        "operator": "gt",
        "positiveCondition": "true",
        "negativeCondition": "false"
      }
    }
  }
}
```

**Key Learning:** ALWAYS convert to ISO8601 before date operations!

</details>

---

## Morning Session Complete!

### You've Mastered

- ✅ ISO 8601 date format
- ✅ Date Format transform
- ✅ Date Math transform
- ✅ Date Compare transform
- ✅ Converting source dates
- ✅ Date arithmetic
- ✅ Date-based conditionals
- ✅ Proper lifecycle state calculation
- ✅ Helper attribute pattern for dates
- ✅ Both UI and JSON configuration

---

## What's Next: Afternoon Session Preview

In the afternoon, you'll learn:

### List/Array Operations

- **Split** - String to array
- **Index** - Get item from array
- **Join** - Array to string
- **First Valid** with arrays

### Real-World Applications

- Parsing complex strings (names, locations)
- Extracting from multi-value attributes
- Processing AD group lists
- Email domain extraction
- Advanced lifecycle state patterns
- Next processing date calculation

**You'll complete your transform toolkit with array manipulation!**

---

## Practice Before Afternoon

Try building these:

### Challenge 1: Review Date Calculator

Calculate next performance review date (6 months after hire):
```
hire_date: "01/15/2024"
Expected output: "2024-07-15T00:00:00Z"
```

### Challenge 2: Contract Expiration

Calculate contractor end date (1 year from start, plus 30 day grace period):
```
start_date: "01/15/2024"
Expected: start + 1 year + 30 days = "2025-02-14T00:00:00Z"
```

### Challenge 3: Eligibility Checker

Check if eligible for benefits (must be hired more than 90 days ago):
```
hire_date: "10/01/2023"
Today: "02/06/2024"
Is eligible? true (more than 90 days)
```

---

**Great work! Take a break, then continue to the Afternoon Session!**

---

*Part of the SailPoint ISC Transforms Mastery 7-Day Study Plan*
