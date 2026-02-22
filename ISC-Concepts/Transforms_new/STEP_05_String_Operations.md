# 🎓 FOUNDATION PHASE - Lesson 3
## String Operations Mastery

---

## What You'll Master
- ✅ Master 8 essential string operations
- ✅ Know when to use each operation
- ✅ Build practical string manipulation transforms
- ✅ Understand common use cases and pitfalls

---

## Why String Operations Matter

95% of your transform work will involve strings (emails, usernames, names, codes).

If you can manipulate strings, you can solve most real-world problems.

---

## Operation #1: Lower

### What It Does
Converts all uppercase letters to lowercase.

### JSON Structure
```json
{
  "type": "lower",
  "attributes": {
    "input": "source_of_data"
  }
}
```

### Real Example: Standardize Email
```json
{
  "name": "Email_Lowercase",
  "type": "lower",
  "attributes": {
    "input": {
      "type": "identityAttribute",
      "attributes": { "name": "email" }
    }
  }
}
```

### Execution
```
Input:  "JOHN.SMITH@COMPANY.COM"
Output: "john.smith@company.com"
```

### When to Use
- ✅ Standardizing emails (different systems provide different cases)
- ✅ Creating usernames (usually lowercase)
- ✅ Preventing correlation issues (JOHN ≠ john)

### Important: Upper Operation
Lower has a counterpart: `upper`

```json
{
  "type": "upper",
  "attributes": {
    "input": { ... }
  }
}
```

Does the opposite: `"john"` → `"JOHN"`

---

## Operation #2: Trim

### What It Does
Removes leading and trailing whitespace.

### JSON Structure
```json
{
  "type": "trim",
  "attributes": {
    "input": "source_of_data"
  }
}
```

### Real Example: Clean Source Data
```json
{
  "name": "FirstName_Trimmed",
  "type": "trim",
  "attributes": {
    "input": {
      "type": "accountAttribute",
      "attributes": {
        "sourceName": "Workday",
        "attributeName": "firstName"
      }
    }
  }
}
```

### Execution
```
Input:  "  John  "
Output: "John"
```

### When to Use
- ✅ Cleaning source data (sources often add spaces)
- ✅ Before doing comparisons (spaces cause "John " ≠ "John")
- ✅ Preparing for length checks

### Common Mistake
```
Input: "John " (space at end)
Without trim: email becomes "john @company.com" ❌
With trim: email becomes "john@company.com" ✅
```

---

## Operation #3: Substring

### What It Does
Extracts a portion of a string using start and end positions.

### JSON Structure
```json
{
  "type": "substring",
  "attributes": {
    "input": "source_of_data",
    "begin": 0,
    "end": 5
  }
}
```

### Important: How Positions Work
- **begin**: Starting position (0-based, inclusive)
- **end**: Ending position (0-based, exclusive)

```
String: "J O H N S M I T H"
Index:   0 1 2 3 4 5 6 7 8

substring(0, 4) returns "JOHN" (positions 0,1,2,3)
substring(4, 8) returns "SMIT" (positions 4,5,6,7)
substring(0, 1) returns "J" (position 0 only)
```

### Real Example #1: Extract Username from Email
```json
{
  "name": "Email_Extract_Username",
  "type": "substring",
  "attributes": {
    "input": {
      "type": "identityAttribute",
      "attributes": { "name": "email" }
    },
    "begin": 0,
    "end": 5
  }
}
```

### Execution
```
Input:  "john.smith@company.com"
Output: "john." (positions 0-5)
```

### Real Example #2: Get First Character for Initial
```json
{
  "name": "FirstName_Initial",
  "type": "substring",
  "attributes": {
    "input": {
      "type": "identityAttribute",
      "attributes": { "name": "firstName" }
    },
    "begin": 0,
    "end": 1
  }
}
```

### Execution
```
Input:  "John"
Output: "J"
```

### When to Use
- ✅ Extract initials from names
- ✅ Get part of an ID
- ✅ Limit string length for systems
- ✅ Extract username from email (combined with indexOf)

### Real-World: Limit to Max Length
Many systems require usernames ≤ 20 characters:

```json
{
  "name": "Username_Max_20_Characters",
  "type": "substring",
  "attributes": {
    "input": {
      "type": "concatenation",
      "attributes": { ... }
    },
    "begin": 0,
    "end": 20
  }
}
```

---

## Operation #4: Concatenation

### What It Does
Joins multiple values together into one string.

### JSON Structure
```json
{
  "type": "concatenation",
  "attributes": {
    "values": [
      "item1",
      "item2",
      "item3"
    ]
  }
}
```

### Real Example: Build Email from Components
```json
{
  "name": "Email_Build_From_Name",
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
        "type": "identityAttribute",
        "attributes": { "name": "lastName" }
      },
      {
        "type": "static",
        "attributes": { "value": "@company.com" }
      }
    ]
  }
}
```

### Execution
```
firstName: "John"
static: "."
lastName: "Smith"
static: "@company.com"

Output: "john.smith@company.com"
```

### When to Use
- ✅ Build emails from components
- ✅ Create usernames from multiple fields
- ✅ Build display names
- ✅ Create full addresses

### Important: Order Matters
The order in the `values` array determines the output order:

```
values: ["J", ".", "smith"]    → Output: "J.smith"
values: ["smith", ".", "J"]    → Output: "smith.J"
```

### Null Handling
If any value in the concatenation is NULL, you need to handle it:

```json
// RISKY: If lastName is NULL, output becomes "John.NULL"
{
  "type": "concatenation",
  "attributes": {
    "values": [
      { firstName },
      { "." },
      { lastName }  // If NULL, problem!
    ]
  }
}

// SAFER: Use firstValid
{
  "type": "concatenation",
  "attributes": {
    "values": [
      { firstName },
      { "." },
      {
        "type": "firstValid",
        "attributes": {
          "values": [
            { lastName },
            { "type": "static", "value": "User" }
          ]
        }
      }
    ]
  }
}
```

---

## Operation #5: Replace

### What It Does
Finds all instances of a substring and replaces them with another.

### JSON Structure
```json
{
  "type": "replace",
  "attributes": {
    "input": "source_of_data",
    "old": "find_this",
    "new": "replace_with_this"
  }
}
```

### Real Example #1: Remove Spaces
```json
{
  "name": "Remove_All_Spaces",
  "type": "replace",
  "attributes": {
    "input": {
      "type": "identityAttribute",
      "attributes": { "name": "email" }
    },
    "old": " ",
    "new": ""
  }
}
```

### Execution
```
Input:  "john smith @ company.com"
Output: "johnsmith@company.com"
```

### Real Example #2: Standardize Hyphen to Underscore
```json
{
  "name": "Username_Hyphen_to_Underscore",
  "type": "replace",
  "attributes": {
    "input": {
      "type": "identityAttribute",
      "attributes": { "name": "username" }
    },
    "old": "-",
    "new": "_"
  }
}
```

### Execution
```
Input:  "john-smith"
Output: "john_smith"
```

### Real Example #3: Remove Special Characters
```json
{
  "name": "Remove_Apostrophes",
  "type": "replace",
  "attributes": {
    "input": {
      "type": "identityAttribute",
      "attributes": { "name": "lastName" }
    },
    "old": "'",
    "new": ""
  }
}
```

### Execution
```
Input:  "O'Brien"
Output: "OBrien"
```

### When to Use
- ✅ Remove unwanted characters
- ✅ Replace problematic characters with safe ones
- ✅ Standardize data from different sources
- ✅ Make data system-compatible

### Important: Replace ALL Instances
Replace replaces EVERY instance it finds:

```
Input: "John-Smith-Jr"
Replace "-" with "_"
Output: "John_Smith_Jr" (all hyphens replaced)
```

---

## Operation #6: indexOf and lastIndexOf

### What It Does
Finds the position of a substring within a string.

### JSON Structure
```json
{
  "type": "indexOf",
  "attributes": {
    "input": "source_of_data",
    "searchFor": "substring_to_find"
  }
}
```

### Real Example: Find Email Domain
```json
{
  "name": "Email_Find_At_Sign_Position",
  "type": "indexOf",
  "attributes": {
    "input": {
      "type": "identityAttribute",
      "attributes": { "name": "email" }
    },
    "searchFor": "@"
  }
}
```

### Execution
```
Input:  "john.smith@company.com"
Output: 10 (@ is at position 10)

String positions:
j o h n . s m i t h @ c o m p a n y . c o m
0 1 2 3 4 5 6 7 8 9 10
```

### When to Use
- ✅ Validate format (does it contain "@"?)
- ✅ Find position of special characters
- ✅ Decide how to split strings

### Important: lastIndexOf
For finding the LAST occurrence:

```
Input: "john.smith.jr@company.com"
indexOf ".":   Output: 4 (first dot)
lastIndexOf ".": Output: 13 (last dot before @)
```

---

## Operation #7: Left Pad and Right Pad

### What It Does
Adds padding characters to reach a specific length.

### JSON Structure
```json
{
  "type": "leftPad",
  "attributes": {
    "input": "source_of_data",
    "length": 8,
    "padding": "0"
  }
}
```

### Real Example: Format Employee ID
```json
{
  "name": "EmployeeID_Pad_with_Zeros",
  "type": "leftPad",
  "attributes": {
    "input": {
      "type": "identityAttribute",
      "attributes": { "name": "employeeId" }
    },
    "length": 8,
    "padding": "0"
  }
}
```

### Execution
```
Input:  "12345"
Output: "00012345" (padded to 8 characters with leading zeros)

Input:  "999999"
Output: "00999999"
```

### Real Example: Right Pad
```json
{
  "name": "Code_Pad_Right",
  "type": "rightPad",
  "attributes": {
    "input": {
      "type": "identityAttribute",
      "attributes": { "name": "code" }
    },
    "length": 10,
    "padding": "-"
  }
}
```

### Execution
```
Input:  "CODE"
Output: "CODE------" (padded to 10 characters)
```

### When to Use
- ✅ Format IDs to required lengths
- ✅ Comply with system requirements
- ✅ Standardize output formatting

### Common Use Case
SAP usernames must be exactly 12 characters:

```json
{
  "type": "leftPad",
  "attributes": {
    "input": { "type": "identityAttribute", "attributes": { "name": "username" } },
    "length": 12,
    "padding": " "
  }
}
```

---

## Operation #8: Get End of String

### What It Does
Gets the last N characters of a string.

### JSON Structure
```json
{
  "type": "getEndOfString",
  "attributes": {
    "input": "source_of_data",
    "length": 4
  }
}
```

### Real Example: Get Extension from Phone
```json
{
  "name": "Phone_Get_Extension",
  "type": "getEndOfString",
  "attributes": {
    "input": {
      "type": "identityAttribute",
      "attributes": { "name": "phone" }
    },
    "length": 4
  }
}
```

### Execution
```
Input:  "555-123-4567"
Output: "4567" (last 4 characters)
```

### Real Example: Get Year from Date
```json
{
  "name": "Date_Extract_Year",
  "type": "getEndOfString",
  "attributes": {
    "input": {
      "type": "identityAttribute",
      "attributes": { "name": "hireDate" }
    },
    "length": 4
  }
}
```

### Execution
```
Input:  "2020-01-15"
Output: "2020" (last 4 characters)
```

### When to Use
- ✅ Extract extensions or parts
- ✅ Get year from dates
- ✅ Extract file extensions

---

## String Operation Patterns

### Pattern #1: Standardize Case + Remove Spaces
```json
{
  "type": "lower",
  "attributes": {
    "input": {
      "type": "replace",
      "attributes": {
        "input": { "type": "identityAttribute", "attributes": { "name": "email" } },
        "old": " ",
        "new": ""
      }
    }
  }
}
```
**Use case:** Clean email data

---

### Pattern #2: Trim + Lowercase
```json
{
  "type": "lower",
  "attributes": {
    "input": {
      "type": "trim",
      "attributes": {
        "input": { "type": "accountAttribute", "attributes": { "sourceName": "Workday", "attributeName": "firstName" } }
      }
    }
  }
}
```
**Use case:** Clean source data that might have spaces and mixed case

---

### Pattern #3: Extract + Limit
```json
{
  "type": "substring",
  "attributes": {
    "input": { "type": "identityAttribute", "attributes": { "name": "username" } },
    "begin": 0,
    "end": 20
  }
}
```
**Use case:** Enforce max length requirement

---

### Pattern #4: Build + Format
```json
{
  "type": "lower",
  "attributes": {
    "input": {
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
  }
}
```
**Use case:** Build and format email

---

## String Operations: Quick Reference

| Operation | Input | Output | Use Case |
|-----------|-------|--------|----------|
| **lower** | "JOHN" | "john" | Standardize case |
| **upper** | "john" | "JOHN" | Uppercase |
| **trim** | " John " | "John" | Remove spaces |
| **substring** | "John"(0,1) | "J" | Extract portion |
| **concatenation** | "J"+"."+"Smith" | "J.Smith" | Join strings |
| **replace** | "J-Smith"→"-"→"_" | "J_Smith" | Replace chars |
| **indexOf** | "john@company"→"@" | 4 | Find position |
| **leftPad** | "123", len 5, "0" | "00123" | Add padding |
| **getEndOfString** | "2020-01-15", len 4 | "2020" | Get last chars |

---

## Real Production Example: Complex Email Standardization

Requirement: Create a standard email format from various sources:
- Remove leading/trailing spaces
- Remove internal spaces
- Convert to lowercase
- Maximum 64 characters
- Default to firstName.lastName@company.com if missing

```json
{
  "name": "Email_Standard_Production",
  "type": "substring",
  "attributes": {
    "input": {
      "type": "lower",
      "attributes": {
        "input": {
          "type": "replace",
          "attributes": {
            "input": {
              "type": "trim",
              "attributes": {
                "input": {
                  "type": "firstValid",
                  "attributes": {
                    "values": [
                      {
                        "type": "accountAttribute",
                        "attributes": { "sourceName": "AD", "attributeName": "mail" }
                      },
                      {
                        "type": "accountAttribute",
                        "attributes": { "sourceName": "Workday", "attributeName": "email" }
                      },
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

**Execution Flow:**
1. Try AD email → if NULL, try Workday → if NULL, generate from firstName.lastName
2. Trim whitespace from result
3. Remove all internal spaces
4. Convert to lowercase
5. Limit to 64 characters
6. Never returns NULL

---

## Mastery Checkpoint #3

### Question 1: Which Operation?
You need to extract the first 3 characters of a username. Which operation?

<details>
<summary>Answer</summary>

**substring** with `begin: 0, end: 3`

```json
{
  "type": "substring",
  "attributes": {
    "input": { ... },
    "begin": 0,
    "end": 3
  }
}
```

</details>

---

### Question 2: Null Handling
You're concatenating firstName + "." + lastName. What happens if lastName is NULL?

<details>
<summary>Answer</summary>

The output becomes "John.null" (bad).

**Fix:** Wrap lastName in firstValid with a fallback:

```json
{
  "type": "concatenation",
  "attributes": {
    "values": [
      { firstName },
      { "." },
      {
        "type": "firstValid",
        "attributes": {
          "values": [
            { lastName },
            { "type": "static", "attributes": { "value": "User" } }
          ]
        }
      }
    ]
  }
}
```

</details>

---

### Question 3: Execution Order
In this transform, what is the final result?

```json
{
  "type": "lower",
  "attributes": {
    "input": {
      "type": "substring",
      "attributes": {
        "input": { "type": "static", "attributes": { "value": "JohnSmith123" } },
        "begin": 0,
        "end": 8
      }
    }
  }
}
```

<details>
<summary>Answer</summary>

**"johnsmit"** (8 characters, lowercase)

1. Substring extracts positions 0-8: "JohnSmit"
2. Lower converts it: "johnsmit"

</details>

---

### Question 4: Real Scenario
Your AD system requires usernames to be exactly 20 characters, padded with underscores on the right. Build the transform.

<details>
<summary>Answer</summary>

```json
{
  "name": "AD_Username_Padded_20",
  "type": "rightPad",
  "attributes": {
    "input": {
      "type": "identityAttribute",
      "attributes": { "name": "username" }
    },
    "length": 20,
    "padding": "_"
  }
}
```

Input: "john.smith" (10 chars)
Output: "john.smith__________" (20 chars)

</details>

---

## Summary: String Operations Essentials

String operations are your foundation. Master these 8, and you handle 60% of real transforms.

| Operation | Speed to Learn | Frequency of Use | Difficulty |
|-----------|----------------|------------------|------------|
| lower/upper | 2 min | Very High | Easy |
| trim | 2 min | High | Easy |
| substring | 5 min | Very High | Medium |
| concatenation | 5 min | Very High | Medium |
| replace | 3 min | High | Easy |
| indexOf | 3 min | Medium | Medium |
| leftPad/rightPad | 3 min | Medium | Easy |
| getEndOfString | 2 min | Low | Easy |

---

## What's Next

In **Lesson 4**, you'll learn:
- ✅ Conditional operations (if/then logic)
- ✅ FirstValid (fallback handling)
- ✅ Building decision trees

These add logic to your string manipulation.

---

## Homework Before Lesson 4

1. For each of the 8 operations, think of ONE real use case in your environment
2. Write down which you'd use most frequently
3. Think about: "What happens if the input is NULL for each operation?"

**Next lesson: Conditional Logic** — where your transforms become smart.
