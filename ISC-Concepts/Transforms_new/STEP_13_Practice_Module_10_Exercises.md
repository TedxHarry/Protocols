# 🎓 FOUNDATION PHASE - Practice Module
## Build Real Transforms: From Simple to Complex

---

## What You'll Do
- ✅ Build 10 transforms from scratch (simple to expert-level)
- ✅ Combine string, conditional, and date operations
- ✅ Solve real business problems
- ✅ Test your understanding before moving forward

---

## How to Use This Module

**For each exercise:**
1. **Read the requirement** - Understand what problem you're solving
2. **Attempt to build it** - Write the transform JSON yourself
3. **Check the solution** - Compare with provided answer
4. **Understand the differences** - Why did they solve it that way?
5. **Modify and experiment** - Change parameters, add complexity

---

## EXERCISE 1: Standardize Email (String Operations)

### Requirement

Create a transform that:
- Takes an email from the identity
- Converts to lowercase
- Removes any spaces
- Ensures it never returns NULL (use default if missing)

### Your Task

Write the JSON for this transform.

**Hint:** You'll use: `firstValid`, `lower`, `replace`, and `static` operations.

### My Solution

```json
{
  "name": "Email_Standardize_Safe",
  "type": "lower",
  "attributes": {
    "input": {
      "type": "replace",
      "attributes": {
        "input": {
          "type": "firstValid",
          "attributes": {
            "values": [
              {
                "type": "identityAttribute",
                "attributes": { "name": "email" }
              },
              {
                "type": "static",
                "attributes": { "value": "noemail@company.com" }
              }
            ]
          }
        },
        "old": " ",
        "new": ""
      }
    }
  }
}
```

### Why This Works

```
Step 1: firstValid tries email, if NULL uses noemail@company.com
Step 2: replace removes all spaces
Step 3: lower converts to lowercase
Step 4: Never returns NULL (always has value)
```

### What You Should Learn

- firstValid provides safety net
- Operations nest inside out (innermost executes first)
- Order matters (if you lowercased BEFORE replacing spaces, results would be same, but logically replace should happen after finding data)

---

## EXERCISE 2: Build Username from Name (String + Concatenation)

### Requirement

Create a transform that:
- Gets firstName and lastName from identity
- Combines them as: firstname.lastname
- Converts to lowercase
- Handles missing lastName (use "User" as fallback)
- Limits to 20 characters

### Your Task

Write the JSON for this transform.

**Hint:** You'll use: `concatenation`, `firstValid`, `lower`, `substring`, and `identityAttribute`.

### My Solution

```json
{
  "name": "Username_Standard_Format",
  "type": "substring",
  "attributes": {
    "input": {
      "type": "lower",
      "attributes": {
        "input": {
          "type": "concatenation",
          "attributes": {
            "values": [
              {
                "type": "identityAttribute",
                "attributes": { "name": "firstName" }
              },
              {
                "type": "static",
                "attributes": { "value": "." }
              },
              {
                "type": "firstValid",
                "attributes": {
                  "values": [
                    {
                      "type": "identityAttribute",
                      "attributes": { "name": "lastName" }
                    },
                    {
                      "type": "static",
                      "attributes": { "value": "User" }
                    }
                  ]
                }
              }
            ]
          }
        }
      }
    },
    "begin": 0,
    "end": 20
  }
}
```

### Execution Examples

```
User 1: firstName = "John", lastName = "Smith"
  → "John.Smith" → "john.smith" → "john.smith" (20 chars limit OK)

User 2: firstName = "Alexander", lastName = "Worthington"
  → "Alexander.Worthington" → "alexander.worthington" → "alexander.wort" (limited to 20)

User 3: firstName = "Jane", lastName = NULL
  → "Jane.User" → "jane.user" → "jane.user"
```

### What You Should Learn

- Nesting multiple operations creates powerful transforms
- substring limits length without breaking on word boundaries
- firstValid prevents NULL propagation through the chain

---

## EXERCISE 3: Classify by Department (Conditional Logic)

### Requirement

Create a transform that classifies employees:
- If department = "Executive" → "EXECUTIVE"
- Else if department = "Manager" → "MANAGER"
- Else → "STAFF"

### Your Task

Write the JSON for this transform.

**Hint:** You'll use nested `conditional` operations.

### My Solution

```json
{
  "name": "EmployeeClassification_ThreeLevel",
  "type": "conditional",
  "attributes": {
    "expression": "$department == 'Executive'",
    "positiveCondition": "EXECUTIVE",
    "negativeCondition": {
      "type": "conditional",
      "attributes": {
        "expression": "$department == 'Manager'",
        "positiveCondition": "MANAGER",
        "negativeCondition": "STAFF"
      }
    }
  }
}
```

### Execution Examples

```
User 1: department = "Executive"
  Check: "Executive" == "Executive"? YES
  Output: "EXECUTIVE"

User 2: department = "Manager"
  Check: "Manager" == "Executive"? NO
  Check: "Manager" == "Manager"? YES
  Output: "MANAGER"

User 3: department = "Engineering"
  Check: "Engineering" == "Executive"? NO
  Check: "Engineering" == "Manager"? NO
  Output: "STAFF"
```

### What You Should Learn

- Nested conditionals handle multiple options
- Order matters (check most specific first)
- Each level gets its own conditional object

---

## EXERCISE 4: Validate Email Format (Conditional with Methods)

### Requirement

Create a transform that:
- Checks if email contains "@"
- If YES → return "VALID"
- If NO → return "INVALID"
- Handle NULL (treat as invalid)

### Your Task

Write the JSON for this transform.

**Hint:** Use `.contains()` method in expression, check for NULL first.

### My Solution

```json
{
  "name": "Email_Format_Validate",
  "type": "conditional",
  "attributes": {
    "expression": "$email != null && $email.contains('@')",
    "positiveCondition": "VALID",
    "negativeCondition": "INVALID"
  }
}
```

### Execution Examples

```
User 1: email = "john@company.com"
  Check: email != null? YES, contains '@'? YES
  Output: "VALID"

User 2: email = "johncompany.com"
  Check: email != null? YES, contains '@'? NO
  Output: "INVALID"

User 3: email = NULL
  Check: email != null? NO
  Output: "INVALID"
```

### What You Should Learn

- NULL checks must come BEFORE .contains() or other methods
- && operator (AND) means both conditions must be true
- Use .contains(), .startsWith(), .endsWith() for string validation

---

## EXERCISE 5: Calculate Probation End Date (Date Operations)

### Requirement

Create a transform that:
- Gets hireDate from identity
- Adds 90 days to it
- Converts result to MM/dd/yyyy format
- Returns the probation end date

### Your Task

Write the JSON for this transform.

**Hint:** You'll use `dateMath`, then `dateFormat`.

### My Solution

```json
{
  "name": "ProbationEndDate_90Days",
  "type": "dateFormat",
  "attributes": {
    "input": {
      "type": "dateMath",
      "attributes": {
        "input": {
          "type": "identityAttribute",
          "attributes": { "name": "hireDate" }
        },
        "expression": "$hireDate+90d"
      }
    },
    "inputFormat": "yyyy-MM-dd",
    "outputFormat": "MM/dd/yyyy"
  }
}
```

### Execution Examples

```
User 1: hireDate = "2024-01-01"
  Math: 2024-01-01 + 90 days = 2024-04-01
  Format: "04/01/2024"

User 2: hireDate = "2023-11-15"
  Math: 2023-11-15 + 90 days = 2024-02-13
  Format: "02/13/2024"
```

### What You Should Learn

- dateMath must come BEFORE dateFormat (inside the input)
- inputFormat and outputFormat describe the format before and after
- dateFormat is the outer operation (applied last)

---

## EXERCISE 6: Identify Onboarding Status (Date Comparison)

### Requirement

Create a transform that:
- Checks if hire date is within last 30 days
- If YES → "ONBOARDING"
- If NO → "ACTIVE"
- Handle NULL hireDate as "ACTIVE"

### Your Task

Write the JSON for this transform.

**Hint:** You'll use `dateCompare` with "lt" operator.

### My Solution

```json
{
  "name": "OnboardingStatus_Check",
  "type": "conditional",
  "attributes": {
    "expression": "$hireDate != null",
    "positiveCondition": {
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
    },
    "negativeCondition": "ACTIVE"
  }
}
```

### How It Works

```
Step 1: Is hireDate not NULL?
  If NO → return "ACTIVE"
  If YES → continue to Step 2

Step 2: Is (today - 30 days) < hireDate?
  If YES → hireDate is recent → "ONBOARDING"
  If NO → hireDate is old → "ACTIVE"
```

### Execution Examples (Today = 2024-02-22)

```
User 1: hireDate = "2024-02-15" (7 days ago)
  Step 1: hireDate != null? YES
  Step 2: Is 2024-01-23 < 2024-02-15? YES
  Output: "ONBOARDING"

User 2: hireDate = "2023-11-15" (100 days ago)
  Step 1: hireDate != null? YES
  Step 2: Is 2024-01-23 < 2023-11-15? NO
  Output: "ACTIVE"

User 3: hireDate = NULL
  Step 1: hireDate != null? NO
  Output: "ACTIVE"
```

### What You Should Learn

- dateCompare needs dateMath inside it (to calculate the comparison date)
- NULL checks go BEFORE dateCompare (to prevent errors)
- "lt" operator checks if first date is LESS than second (in the past)

---

## EXERCISE 7: Multi-Stage Lifecycle Status (Complex Nesting)

### Requirement

Create a transform that classifies based on lifecycle:
- If terminated (terminationDate exists AND > 30 days ago) → "TERMINATED_DEPROVISION"
- Else if hired < 30 days ago → "ONBOARDING"
- Else if hired 30-90 days ago → "NEW_EMPLOYEE"
- Else → "ACTIVE"

### Your Task

Write the JSON for this transform.

**Hint:** This requires nested conditionals with dateCompare operations inside.

### My Solution

```json
{
  "name": "EmployeeLifecycleStatus_Full",
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
        "positiveCondition": "TERMINATED_DEPROVISION",
        "negativeCondition": "TERMINATED_ACTIVE"
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
  }
}
```

### Decision Tree

```
Start
  ├─ Has terminationDate?
  │   ├─ YES: Terminated > 30 days ago?
  │   │   ├─ YES → "TERMINATED_DEPROVISION"
  │   │   └─ NO → "TERMINATED_ACTIVE"
  │   │
  │   └─ NO: Hired < 30 days ago?
  │       ├─ YES → "ONBOARDING"
  │       └─ NO: Hired < 90 days ago?
  │           ├─ YES → "NEW_EMPLOYEE"
  │           └─ NO → "ACTIVE"
```

### Execution Examples (Today = 2024-02-22)

```
User 1: terminationDate = "2023-12-01", hireDate = "2023-01-01"
  Check: terminationDate exists? YES
  Check: 2023-12-01 < 2024-01-23? YES
  Output: "TERMINATED_DEPROVISION" ✓

User 2: terminationDate = NULL, hireDate = "2024-02-15"
  Check: terminationDate exists? NO
  Check: Is 2024-01-23 < 2024-02-15? YES (hired recently)
  Output: "ONBOARDING" ✓

User 3: terminationDate = NULL, hireDate = "2024-01-05"
  Check: terminationDate exists? NO
  Check: Is 2024-01-23 < 2024-01-05? NO
  Check: Is 2023-12-23 < 2024-01-05? YES (hired 30-90 days ago)
  Output: "NEW_EMPLOYEE" ✓

User 4: terminationDate = NULL, hireDate = "2023-01-01"
  Check: terminationDate exists? NO
  Check: Is 2024-01-23 < 2023-01-01? NO
  Check: Is 2023-12-23 < 2023-01-01? NO
  Output: "ACTIVE" ✓
```

### What You Should Learn

- Complex transforms use multiple nested operations
- Each dateCompare needs its own dateMath for the comparison date
- Read the decision tree, then build the JSON level by level
- This is 30+ lines of JSON but follows a clear structure

---

## EXERCISE 8: Generate Standardized Email (Integration of 1-5)

### Requirement

Create a transform that:
- Gets email from multiple sources (AD first, then Workday, then generate)
- If generated, use: firstName.lastName@company.com
- Convert entire result to lowercase
- Remove spaces
- Limit to 64 characters
- Handle all NULL cases

### Your Task

Write the JSON for this transform.

**Hint:** This combines firstValid (multi-source), concatenation (build), replace (clean), lower (format), and substring (limit).

### My Solution

```json
{
  "name": "Email_Production_Standard",
  "type": "substring",
  "attributes": {
    "input": {
      "type": "lower",
      "attributes": {
        "input": {
          "type": "replace",
          "attributes": {
            "input": {
              "type": "firstValid",
              "attributes": {
                "values": [
                  {
                    "type": "accountAttribute",
                    "attributes": {
                      "sourceName": "Active Directory",
                      "attributeName": "mail"
                    }
                  },
                  {
                    "type": "accountAttribute",
                    "attributes": {
                      "sourceName": "Workday",
                      "attributeName": "email"
                    }
                  },
                  {
                    "type": "concatenation",
                    "attributes": {
                      "values": [
                        {
                          "type": "identityAttribute",
                          "attributes": { "name": "firstName" }
                        },
                        { "type": "static", "attributes": { "value": "." } },
                        {
                          "type": "identityAttribute",
                          "attributes": { "name": "lastName" }
                        },
                        { "type": "static", "attributes": { "value": "@company.com" } }
                      ]
                    }
                  }
                ]
              }
            },
            "old": " ",
            "new": ""
          }
        }
      }
    },
    "begin": 0,
    "end": 64
  }
}
```

### Execution Flow

```
Step 1: firstValid tries:
  → AD email (john.smith@company.com)
  → If NULL, Workday email (JOHN.SMITH@workday.com)
  → If NULL, generate (John.Smith@company.com)

Step 2: replace removes spaces
  → "john smith@company.com" → "johnsmith@company.com"

Step 3: lower converts to lowercase
  → "JOHN.SMITH@COMPANY.COM" → "john.smith@company.com"

Step 4: substring limits to 64 characters
  → "john.smith@company.com" (42 chars, well under limit)
```

### Real-World Scenarios

```
User 1: AD has "john.smith@company.com"
  Result: "john.smith@company.com" ✓

User 2: AD NULL, Workday has "JOHN.SMITH@workday.com"
  Result: "john.smith@workday.com" ✓

User 3: Both NULL, firstName="John", lastName="Smith"
  Generated: "John.Smith@company.com"
  Result: "john.smith@company.com" ✓

User 4: Email has spaces "john smith @company.com"
  Remove spaces: "johnsmith@company.com"
  Result: "johnsmith@company.com" ✓
```

### What You Should Learn

- This is a REAL production transform
- It uses 5 different operation types
- Each operation solves one problem
- Together they create a robust solution
- This is the level of complexity you'll encounter regularly

---

## EXERCISE 9: Conditional with AND Logic (Employment Classification)

### Requirement

Create a transform that:
- If department = "IT" AND salary > 50000 → "SENIOR_IT"
- Else if department = "IT" → "IT_STAFF"
- Else → "OTHER"

### Your Task

Write the JSON for this transform.

**Hint:** Use `&&` operator in the expression.

### My Solution

```json
{
  "name": "EmploymentClassification_Complex",
  "type": "conditional",
  "attributes": {
    "expression": "$department == 'IT' && $salary > 50000",
    "positiveCondition": "SENIOR_IT",
    "negativeCondition": {
      "type": "conditional",
      "attributes": {
        "expression": "$department == 'IT'",
        "positiveCondition": "IT_STAFF",
        "negativeCondition": "OTHER"
      }
    }
  }
}
```

### Execution Examples

```
User 1: department = "IT", salary = 60000
  Check: IT && salary > 50000? YES
  Output: "SENIOR_IT"

User 2: department = "IT", salary = 40000
  Check: IT && salary > 50000? NO
  Check: IT? YES
  Output: "IT_STAFF"

User 3: department = "Finance", salary = 60000
  Check: IT && salary > 50000? NO (not IT)
  Check: IT? NO
  Output: "OTHER"
```

### What You Should Learn

- AND (&&) means BOTH conditions must be true
- If either is false, the whole expression is false
- First conditional checks the complex condition
- Second conditional handles the remaining cases

---

## EXERCISE 10: Expert Challenge - Full Employee Provisioning Logic

### Requirement

Create a single transform that:
1. Classifies employee status (Active/Onboarding/Terminated)
2. Generates username (firstName.lastName, safe, unique-ready)
3. Creates email (from sources or generate)
4. Returns all three as concatenated string: "STATUS:USERNAME:EMAIL"

### Your Task

This is HARD. Think step by step:
- What operations do you need?
- What order should they go in?
- How do you concatenate the final result?

**Hint:** You'll need conditionals (status), string ops (username), firstValid (email), then concatenation to join them all.

### My Solution

```json
{
  "name": "EmployeeProvisioning_Full",
  "type": "concatenation",
  "attributes": {
    "values": [
      {
        "type": "conditional",
        "attributes": {
          "expression": "$hireDate != null",
          "positiveCondition": {
            "type": "dateCompare",
            "attributes": {
              "firstDate": {
                "type": "dateMath",
                "attributes": {
                  "input": { "type": "static", "attributes": { "value": "now" } },
                  "expression": "now-30d"
                }
              },
              "secondDate": { "type": "identityAttribute", "attributes": { "name": "hireDate" } },
              "operator": "lt",
              "positiveCondition": "ONBOARDING",
              "negativeCondition": "ACTIVE"
            }
          },
          "negativeCondition": "ACTIVE"
        }
      },
      { "type": "static", "attributes": { "value": ":" } },
      {
        "type": "lower",
        "attributes": {
          "input": {
            "type": "concatenation",
            "attributes": {
              "values": [
                {
                  "type": "substring",
                  "attributes": {
                    "input": { "type": "identityAttribute", "attributes": { "name": "firstName" } },
                    "begin": 0,
                    "end": 1
                  }
                },
                { "type": "identityAttribute", "attributes": { "name": "lastName" } }
              ]
            }
          }
        }
      },
      { "type": "static", "attributes": { "value": ":" } },
      {
        "type": "lower",
        "attributes": {
          "input": {
            "type": "firstValid",
            "attributes": {
              "values": [
                { "type": "accountAttribute", "attributes": { "sourceName": "AD", "attributeName": "mail" } },
                { "type": "accountAttribute", "attributes": { "sourceName": "Workday", "attributeName": "email" } },
                {
                  "type": "concatenation",
                  "attributes": {
                    "values": [
                      { "type": "identityAttribute", "attributes": { "name": "firstName" } },
                      { "type": "static", "attributes": { "value": "." } },
                      { "type": "identityAttribute", "attributes": { "name": "lastName" } },
                      { "type": "static", "attributes": { "value": "@company.com" } }
                    ]
                  }
                }
              ]
            }
          }
        }
      }
    ]
  }
}
```

### Output Examples

```
User 1: John Smith, hired 2024-02-15, AD email exists
  Status: ONBOARDING (hired < 30 days)
  Username: jsmith
  Email: john.smith@company.com
  Output: "ONBOARDING:jsmith:john.smith@company.com"

User 2: Jane Doe, hired 2023-01-01, no AD email, Workday email exists
  Status: ACTIVE (hired > 30 days)
  Username: jdoe
  Email: jane.doe@workday.com
  Output: "ACTIVE:jdoe:jane.doe@workday.com"

User 3: Robert Johnson, hired 2024-02-01, no emails in sources
  Status: ONBOARDING
  Username: rjohnson
  Email: robert.johnson@company.com (generated)
  Output: "ONBOARDING:rjohnson:robert.johnson@company.com"
```

### What You Should Learn

- This combines EVERYTHING from Lessons 1-5
- Real transforms can be complex but still readable
- When stuck, break it into parts:
  1. Build status part
  2. Build username part
  3. Build email part
  4. Concatenate all together
- This is approximately 4+ years experience level for first-time building

---

## Summary: What You've Built

| Exercise | What It Teaches | Difficulty |
|----------|-----------------|------------|
| 1 | Safe data handling with firstValid | Easy |
| 2 | Nesting and concatenation | Easy |
| 3 | Nested conditionals | Easy |
| 4 | String validation with methods | Medium |
| 5 | Date math and formatting | Medium |
| 6 | Date comparisons | Medium |
| 7 | Complex multi-level nesting | Hard |
| 8 | Integration (5 operations) | Hard |
| 9 | AND logic in conditions | Medium |
| 10 | Full expert scenario | Expert |

---

## Self-Assessment

After completing these exercises, you should be able to:

- ✅ Read any transform and understand it
- ✅ Write simple transforms from scratch
- ✅ Combine multiple operations effectively
- ✅ Handle NULL values safely
- ✅ Build production-grade transforms
- ✅ Debug when something doesn't work
- ✅ Explain your transform choices

---

## Next Steps

After completing all 10 exercises:

1. **Modify each one** - Change the requirements, rebuild
2. **Combine exercises** - Exercise 1 + Exercise 6 together
3. **Test mentally** - What if this attribute is NULL?
4. **Optimize** - Can you make it shorter or clearer?

Once you're comfortable, you're ready for **Lesson 6: Reference and Generator Operations**.

---

## Tips for Success

### When You Get Stuck

1. **Break it down** - Don't try to write the whole thing at once
2. **Start innermost** - Build the data source first, then wrap operations around it
3. **Test each piece** - Does substring work? Then wrap in lower.
4. **Check the hints** - They tell you which operations to use

### Common Mistakes

- **Forgetting NULL checks** - Always check if data exists before comparing
- **Wrong operation order** - Build data FIRST, then format it
- **Too much nesting** - If it's 50+ lines, break it into smaller pieces
- **Case sensitivity** - Remember to lowercase before comparing

### Learning Strategy

- Do Exercise 1-5 completely before moving to 6-10
- After each exercise, write down what you learned
- Don't just copy answers—understand WHY each operation is there
- Modify and experiment with parameters

---

**You're ready. Start with Exercise 1.**
