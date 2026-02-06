# JSON MASTERY - DAY 3: JSON IN SAILPOINT ISC

## Welcome to Day 3!

Today you'll learn the REAL power of JSON - controlling SailPoint ISC programmatically!

**Today's Goal:** Master SailPoint ISC JSON structures and configurations

---

## Table of Contents
- [Morning Session: SailPoint Transform JSON](#morning-session-sailpoint-transform-json)
  - [Part 1: Complete Transform Reference](#part-1-complete-transform-reference-45-minutes)
  - [Part 2: Exporting Transforms](#part-2-exporting-transforms-30-minutes)
  - [Part 3: Transform Patterns Library](#part-3-transform-patterns-library-45-minutes)
- [Afternoon Session: Identity Profile JSON](#afternoon-session-identity-profile-json)
  - [Part 4: Identity Profile Structure](#part-4-identity-profile-structure-60 minutes)
  - [Part 5: Source Configuration JSON](#part-5-source-configuration-json-60-minutes)
- [Evening Session: Advanced JSON Operations](#evening-session-advanced-json-operations)
  - [Part 6: Bulk Operations with JSON](#part-6-bulk-operations-with-json-60-minutes)
- [Day 3 Assessment](#day-3-assessment)

---

## Morning Session: SailPoint Transform JSON

### Part 1: Complete Transform Reference (45 minutes)

Let's document EVERY transform type you learned in the 7-day transforms course!

---

#### Transform Type 1: Static

**Purpose:** Return a fixed value for all identities

**JSON Structure:**
```json
{
  "name": "attributeName",
  "type": "static",
  "attributes": {
    "value": "fixed value"
  }
}
```

**Real Example:**
```json
{
  "name": "country",
  "type": "static",
  "attributes": {
    "value": "United States"
  }
}
```

**What it does:** Sets country = "United States" for every identity

**When to use:**
- Default values for all users
- Constants that don't change
- Placeholder values

---

#### Transform Type 2: identityAttribute

**Purpose:** Reference another attribute on the same identity

**JSON Structure:**
```json
{
  "type": "identityAttribute",
  "attributes": {
    "name": "sourceAttributeName"
  }
}
```

**Real Example:**
```json
{
  "name": "displayEmail",
  "type": "identityAttribute",
  "attributes": {
    "name": "email"
  }
}
```

**What it does:** Copies value from email attribute to displayEmail

**When to use:**
- Copy one attribute to another
- Reference calculated values
- Use in concatenation or conditions

---

#### Transform Type 3: Concatenation

**Purpose:** Join multiple values together

**JSON Structure:**
```json
{
  "name": "attributeName",
  "type": "concat",
  "attributes": {
    "values": [
      {
        "type": "identityAttribute",
        "attributes": {
          "name": "attribute1"
        }
      },
      {
        "type": "static",
        "attributes": {
          "value": "delimiter"
        }
      },
      {
        "type": "identityAttribute",
        "attributes": {
          "name": "attribute2"
        }
      }
    ]
  }
}
```

**Real Example:**
```json
{
  "name": "displayName",
  "type": "concat",
  "attributes": {
    "values": [
      {
        "type": "identityAttribute",
        "attributes": {
          "name": "firstname"
        }
      },
      {
        "type": "static",
        "attributes": {
          "value": " "
        }
      },
      {
        "type": "identityAttribute",
        "attributes": {
          "name": "lastname"
        }
      }
    ]
  }
}
```

**What it does:** Creates "firstname lastname"

**When to use:**
- Build display names
- Generate emails
- Combine multiple fields

---

#### Transform Type 4: Substring

**Purpose:** Extract portion of a string

**JSON Structure:**
```json
{
  "name": "attributeName",
  "type": "substring",
  "attributes": {
    "input": {
      "type": "identityAttribute",
      "attributes": {
        "name": "sourceAttribute"
      }
    },
    "begin": 0,
    "end": 10
  }
}
```

**Real Example:**
```json
{
  "name": "shortUsername",
  "type": "substring",
  "attributes": {
    "input": {
      "type": "identityAttribute",
      "attributes": {
        "name": "username"
      }
    },
    "begin": 0,
    "end": 8
  }
}
```

**What it does:** Takes first 8 characters of username

**When to use:**
- Truncate long values
- Extract portions of strings
- Enforce length limits

**Position Rules:**
```
String: "ABCDEFGH"
Index:   01234567

Substring(0, 3) = "ABC"
Substring(2, 5) = "CDE"
Substring(0, -1) = entire string (some versions)
```

---

#### Transform Type 5: Upper

**Purpose:** Convert string to uppercase

**JSON Structure:**
```json
{
  "name": "attributeName",
  "type": "upper",
  "attributes": {
    "input": {
      "type": "identityAttribute",
      "attributes": {
        "name": "sourceAttribute"
      }
    }
  }
}
```

**Real Example:**
```json
{
  "name": "deptCodeUpper",
  "type": "upper",
  "attributes": {
    "input": {
      "type": "identityAttribute",
      "attributes": {
        "name": "dept_code"
      }
    }
  }
}
```

**What it does:** "eng" → "ENG"

**When to use:**
- Normalize case for lookups
- Format codes
- Ensure consistency

---

#### Transform Type 6: Lower

**Purpose:** Convert string to lowercase

**JSON Structure:**
```json
{
  "name": "attributeName",
  "type": "lower",
  "attributes": {
    "input": {
      "type": "identityAttribute",
      "attributes": {
        "name": "sourceAttribute"
      }
    }
  }
}
```

**Real Example:**
```json
{
  "name": "emailLower",
  "type": "lower",
  "attributes": {
    "input": {
      "type": "identityAttribute",
      "attributes": {
        "name": "email"
      }
    }
  }
}
```

**What it does:** "John@Company.COM" → "john@company.com"

**When to use:**
- Email normalization
- Username generation
- Case-insensitive comparisons

---

#### Transform Type 7: Trim

**Purpose:** Remove leading and trailing whitespace

**JSON Structure:**
```json
{
  "name": "attributeName",
  "type": "trim",
  "attributes": {
    "input": {
      "type": "identityAttribute",
      "attributes": {
        "name": "sourceAttribute"
      }
    }
  }
}
```

**Real Example:**
```json
{
  "name": "firstnameTrimmed",
  "type": "trim",
  "attributes": {
    "input": {
      "type": "identityAttribute",
      "attributes": {
        "name": "firstname"
      }
    }
  }
}
```

**What it does:** "  John  " → "John"

**When to use:**
- Clean messy source data
- Before concatenation
- Data quality improvement

---

#### Transform Type 8: Replace

**Purpose:** Find and replace text

**JSON Structure:**
```json
{
  "name": "attributeName",
  "type": "replace",
  "attributes": {
    "input": {
      "type": "identityAttribute",
      "attributes": {
        "name": "sourceAttribute"
      }
    },
    "regex": "pattern",
    "replacement": "replacement"
  }
}
```

**Real Example:**
```json
{
  "name": "phoneClean",
  "type": "replace",
  "attributes": {
    "input": {
      "type": "identityAttribute",
      "attributes": {
        "name": "phone"
      }
    },
    "regex": "-",
    "replacement": ""
  }
}
```

**What it does:** "214-555-1234" → "2145551234"

**Common Patterns:**
```json
Remove hyphens:     "regex": "-",       "replacement": ""
Remove spaces:      "regex": " ",       "replacement": ""
Remove parens:      "regex": "\\(",     "replacement": ""
Replace dots:       "regex": "\\.",     "replacement": "_"
```

**When to use:**
- Remove special characters
- Clean phone numbers
- Sanitize input

---

#### Transform Type 9: Conditional

**Purpose:** If-then-else logic

**JSON Structure:**
```json
{
  "name": "attributeName",
  "type": "conditional",
  "attributes": {
    "expression": "condition",
    "positiveCondition": "value if true",
    "negativeCondition": "value if false"
  }
}
```

**Real Example:**
```json
{
  "name": "employeeType",
  "type": "conditional",
  "attributes": {
    "expression": "employee_type == 'FTE'",
    "positiveCondition": "Full-Time",
    "negativeCondition": "Contractor"
  }
}
```

**Operators:**
```
Equals:              ==
Not equals:          !=
Greater than:        >
Less than:           
Greater or equal:    >=
Less or equal:       <=
Contains:            contains
Starts with:         startsWith
Ends with:           endsWith
AND:                 &&
OR:                  ||
```

**Nested Conditionals:**
```json
{
  "type": "conditional",
  "attributes": {
    "expression": "status == 'Active'",
    "positiveCondition": "Active",
    "negativeCondition": {
      "type": "conditional",
      "attributes": {
        "expression": "status == 'Terminated'",
        "positiveCondition": "Terminated",
        "negativeCondition": "Unknown"
      }
    }
  }
}
```

**When to use:**
- Decision logic
- Multi-value selection
- State calculation

---

#### Transform Type 10: FirstValid

**Purpose:** Return first non-null value from a list

**JSON Structure:**
```json
{
  "name": "attributeName",
  "type": "firstValid",
  "attributes": {
    "values": [
      {
        "type": "identityAttribute",
        "attributes": {
          "name": "option1"
        }
      },
      {
        "type": "identityAttribute",
        "attributes": {
          "name": "option2"
        }
      },
      {
        "type": "static",
        "attributes": {
          "value": "default"
        }
      }
    ],
    "ignoreErrors": true
  }
}
```

**Real Example:**
```json
{
  "name": "email",
  "type": "firstValid",
  "attributes": {
    "values": [
      {
        "type": "identityAttribute",
        "attributes": {
          "name": "personal_email"
        }
      },
      {
        "type": "identityAttribute",
        "attributes": {
          "name": "work_email"
        }
      },
      {
        "type": "static",
        "attributes": {
          "value": "noemail@company.com"
        }
      }
    ],
    "ignoreErrors": true
  }
}
```

**What it does:** 
```
Try personal_email → if null, try work_email → if null, use default
```

**When to use:**
- Fallback chains
- Priority-based selection
- Null handling

---

#### Transform Type 11: Lookup

**Purpose:** Map input values to output values via table

**JSON Structure:**
```json
{
  "name": "attributeName",
  "type": "lookup",
  "attributes": {
    "input": {
      "type": "identityAttribute",
      "attributes": {
        "name": "sourceAttribute"
      }
    },
    "table": {
      "id": "lookup-table-id"
    },
    "default": "default value"
  }
}
```

**Real Example:**
```json
{
  "name": "departmentName",
  "type": "lookup",
  "attributes": {
    "input": {
      "type": "identityAttribute",
      "attributes": {
        "name": "dept_code"
      }
    },
    "table": {
      "id": "dept-code-mapping"
    },
    "default": "Unknown"
  }
}
```

**What it does:**
```
Lookup table "dept-code-mapping":
  ENG → Engineering
  FIN → Finance
  HR → Human Resources

Input: "ENG"
Output: "Engineering"

Input: "SALES"
Output: "Unknown" (not in table, use default)
```

**When to use:**
- Code to name mappings
- Standardization
- Multiple value mappings (10+)

---

#### Transform Type 12: Date Format

**Purpose:** Convert date from one format to another

**JSON Structure:**
```json
{
  "name": "attributeName",
  "type": "dateFormat",
  "attributes": {
    "input": {
      "type": "identityAttribute",
      "attributes": {
        "name": "sourceDate"
      }
    },
    "inputFormat": "MM/dd/yyyy",
    "outputFormat": "ISO8601"
  }
}
```

**Real Example:**
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

**What it does:** "01/15/2024" → "2024-01-15T00:00:00Z"

**Common Formats:**
```
MM/dd/yyyy          01/15/2024
dd-MMM-yyyy         15-Jan-2024
yyyyMMdd            20240115
ISO8601             2024-01-15T00:00:00Z
yyyy-MM-dd          2024-01-15
```

**When to use:**
- Convert source dates to ISO8601
- Date comparisons (require ISO8601)
- Date math operations

---

#### Transform Type 13: Date Math

**Purpose:** Add or subtract time from a date

**JSON Structure:**
```json
{
  "name": "attributeName",
  "type": "dateMath",
  "attributes": {
    "input": {
      "type": "identityAttribute",
      "attributes": {
        "name": "sourceDate"
      }
    },
    "operation": "add",
    "value": 90,
    "unit": "days"
  }
}
```

**Real Example:**
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
    "operation": "add",
    "value": 90,
    "unit": "days"
  }
}
```

**What it does:** Add 90 days to hire date

**Operations:**
```
"operation": "add"       Add time
"operation": "subtract"  Subtract time

Units:
"unit": "days"
"unit": "months"
"unit": "years"
"unit": "hours"
```

**When to use:**
- Calculate expiration dates
- Probation periods
- Account validity

---

#### Transform Type 14: Date Compare

**Purpose:** Compare two dates

**JSON Structure:**
```json
{
  "name": "attributeName",
  "type": "dateCompare",
  "attributes": {
    "firstDate": {
      "type": "identityAttribute",
      "attributes": {
        "name": "date1"
      }
    },
    "operator": "gt",
    "secondDate": {
      "type": "now"
    }
  }
}
```

**Real Example:**
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
    "operator": "gt",
    "secondDate": {
      "type": "now"
    }
  }
}
```

**What it does:** Returns true if hire date > today (future hire date)

**Operators:**
```
"operator": "gt"    Greater than (>)
"operator": "lt"    Less than (<)
"operator": "eq"    Equals (=)
"operator": "gte"   Greater or equal (>=)
"operator": "lte"   Less or equal (<=)
```

**Second Date Options:**
```
Current time:
{
  "type": "now"
}

Another attribute:
{
  "type": "identityAttribute",
  "attributes": {
    "name": "otherDate"
  }
}
```

**When to use:**
- Lifecycle state logic
- Pre-hire detection
- Expiration checking

---

#### Transform Type 15: Split

**Purpose:** Split string into array

**JSON Structure:**
```json
{
  "name": "attributeName",
  "type": "split",
  "attributes": {
    "input": {
      "type": "identityAttribute",
      "attributes": {
        "name": "sourceString"
      }
    },
    "delimiter": ","
  }
}
```

**Real Example:**
```json
{
  "name": "nameParts",
  "type": "split",
  "attributes": {
    "input": {
      "type": "identityAttribute",
      "attributes": {
        "name": "full_name"
      }
    },
    "delimiter": ","
  }
}
```

**What it does:** "Smith, John" → ["Smith", " John"]

**When to use:**
- Parse delimited strings
- Extract name parts
- Break down complex strings

---

#### Transform Type 16: Index

**Purpose:** Get item from array by position

**JSON Structure:**
```json
{
  "name": "attributeName",
  "type": "index",
  "attributes": {
    "input": {
      "type": "identityAttribute",
      "attributes": {
        "name": "arrayAttribute"
      }
    },
    "position": 0
  }
}
```

**Real Example:**
```json
{
  "name": "lastname",
  "type": "index",
  "attributes": {
    "input": {
      "type": "identityAttribute",
      "attributes": {
        "name": "nameParts"
      }
    },
    "position": 0
  }
}
```

**What it does:** Extract first item from array

**Positions:**
```
Array: ["A", "B", "C"]
Position 0 = "A"
Position 1 = "B"
Position 2 = "C"
```

**When to use:**
- Extract from split results
- Get first group from AD
- Parse multi-value fields

---

#### Transform Type 17: Join

**Purpose:** Combine array into string

**JSON Structure:**
```json
{
  "name": "attributeName",
  "type": "join",
  "attributes": {
    "input": {
      "type": "identityAttribute",
      "attributes": {
        "name": "arrayAttribute"
      }
    },
    "delimiter": ", "
  }
}
```

**Real Example:**
```json
{
  "name": "skillsList",
  "type": "join",
  "attributes": {
    "input": {
      "type": "identityAttribute",
      "attributes": {
        "name": "skills"
      }
    },
    "delimiter": ", "
  }
}
```

**What it does:** ["Python", "Java", "SQL"] → "Python, Java, SQL"

**When to use:**
- Display multi-value attributes
- Combine array items
- Create comma-separated lists

---

#### Transform Type 18: Account Attribute

**Purpose:** Get attribute from user's account on a source

**JSON Structure:**
```json
{
  "name": "attributeName",
  "type": "accountAttribute",
  "attributes": {
    "sourceName": "Source Name",
    "attributeName": "attributeName"
  }
}
```

**Real Example:**
```json
{
  "name": "emailFromAD",
  "type": "accountAttribute",
  "attributes": {
    "sourceName": "Active Directory",
    "attributeName": "mail"
  }
}
```

**What it does:** Gets "mail" attribute from user's AD account

**When to use:**
- Pull data from connected systems
- Get email from AD
- Extract groups, title, etc.

---

### Transform Chaining with "next"

**Any transform can chain to another using the "next" property:**
```json
{
  "type": "trim",
  "attributes": {
    "input": {
      "type": "identityAttribute",
      "attributes": {
        "name": "firstname"
      }
    },
    "next": {
      "type": "lower",
      "next": {
        "type": "concat",
        "attributes": {
          "values": [
            {
              "type": "reference"
            },
            {
              "type": "static",
              "attributes": {
                "value": "@company.com"
              }
            }
          ]
        }
      }
    }
  }
}
```

**Flow:**
```
Input: "  John  "
→ Trim → "John"
→ Lower → "john"
→ Concat → "john@company.com"
```

**The "reference" type:**
```json
{
  "type": "reference"
}
```
This references the output from the PREVIOUS transform in the chain.

---

### Part 2: Exporting Transforms (30 minutes)

Learn how to get JSON from SailPoint ISC UI.

---

#### Method 1: Export via UI (if available)

**Some ISC versions support direct export:**
```
1. Navigate to Identity Profile > Mappings
2. Click on an attribute
3. Look for "Export" or "View JSON" button
4. Copy JSON to clipboard
5. Paste into text editor
```

> **Note:** Export UI varies by ISC version. Check your version's documentation.

---

#### Method 2: Browser Developer Tools

**Every browser can capture network traffic:**

**Steps:**
```
1. Open SailPoint ISC
2. Press F12 (opens Developer Tools)
3. Go to "Network" tab
4. Navigate to Identity Profile > Mappings
5. Click on a transform
6. Look in Network tab for API calls
7. Find the response containing transform JSON
8. Copy JSON
```

**Example Network Call:**
```
Request URL: https://[tenant].api.identitynow.com/v3/transforms/[id]
Method: GET
Response: {transform JSON}
```

---

#### Method 3: API Call (Most Reliable)

**Use the SailPoint API to get transforms:**

**Using Postman or curl:**
```bash
# Get all transforms
GET https://[tenant].api.identitynow.com/v3/transforms

# Get specific transform
GET https://[tenant].api.identitynow.com/v3/transforms/[transform-id]
```

**Response Example:**
```json
{
  "id": "2cd78adghjkja",
  "name": "displayName",
  "type": "concat",
  "attributes": {
    "values": [...]
  }
}
```

**We'll cover API in detail in Day 4!**

---

#### Method 4: Export Entire Identity Profile

**Export complete profile configuration:**
```
1. Navigate to Identity Profile
2. Click "Export" (if available)
3. Download JSON file
4. Open in text editor
5. Find "attributeTransforms" section
6. All transforms are listed there
```

**Exported Profile Structure:**
```json
{
  "id": "profile-id",
  "name": "Employees",
  "sourceId": "workday-source-id",
  "identityAttributeConfig": {
    "attributeTransforms": [
      {
        "identityAttributeName": "displayName",
        "transformDefinition": {
          "type": "concat",
          "attributes": {...}
        }
      },
      {
        "identityAttributeName": "email",
        "transformDefinition": {
          "type": "firstValid",
          "attributes": {...}
        }
      }
    ]
  }
}
```

---

### Practice Exercise 1: Reading Exported JSON

**Here's a real exported transform. What does it do?**
```json
{
  "id": "abc123",
  "name": "email",
  "type": "trim",
  "attributes": {
    "input": {
      "type": "identityAttribute",
      "attributes": {
        "name": "firstname"
      }
    },
    "next": {
      "type": "concat",
      "attributes": {
        "values": [
          {
            "type": "reference"
          },
          {
            "type": "static",
            "attributes": {
              "value": "."
            }
          },
          {
            "type": "trim",
            "attributes": {
              "input": {
                "type": "identityAttribute",
                "attributes": {
                  "name": "lastname"
                }
              }
            }
          },
          {
            "type": "static",
            "attributes": {
              "value": "@company.com"
            }
          }
        ],
        "next": {
          "type": "lower"
        }
      }
    }
  }
}
```

<details>
<summary>Click to reveal answer</summary>

**What it does:**
```
Step 1: Trim firstname
  "  John  " → "John"

Step 2: Concatenate:
  - Trimmed firstname ("John")
  - "."
  - Trimmed lastname ("Smith")
  - "@company.com"
  Result: "John.Smith@company.com"

Step 3: Convert to lowercase
  "John.Smith@company.com" → "john.smith@company.com"

Final: email = "john.smith@company.com"
```

**Transform chain:**
```
Trim(firstname) → Concat(result + "." + Trim(lastname) + "@company.com") → Lower
```

</details>

---

### Part 3: Transform Patterns Library (45 minutes)

**Build your own library of reusable transform patterns!**

---

#### Pattern 1: Clean and Lowercase Email

**Use Case:** Generate email from first and last name
```json
{
  "name": "email",
  "type": "trim",
  "attributes": {
    "input": {
      "type": "identityAttribute",
      "attributes": {
        "name": "firstname"
      }
    },
    "next": {
      "type": "lower",
      "next": {
        "type": "concat",
        "attributes": {
          "values": [
            {
              "type": "reference"
            },
            {
              "type": "static",
              "attributes": {
                "value": "."
              }
            },
            {
              "type": "trim",
              "attributes": {
                "input": {
                  "type": "identityAttribute",
                  "attributes": {
                    "name": "lastname"
                  }
                },
                "next": {
                  "type": "lower"
                }
              }
            },
            {
              "type": "static",
              "attributes": {
                "value": "@company.com"
              }
            }
          ]
        }
      }
    }
  }
}
```

---

#### Pattern 2: Normalize Department Code

**Use Case:** Clean and uppercase department codes for lookup
```json
{
  "name": "deptCodeNormalized",
  "type": "trim",
  "attributes": {
    "input": {
      "type": "identityAttribute",
      "attributes": {
        "name": "dept_code"
      }
    },
    "next": {
      "type": "upper"
    }
  }
}
```

---

#### Pattern 3: Smart Email with Fallbacks

**Use Case:** Try multiple email sources
```json
{
  "name": "primaryEmail",
  "type": "firstValid",
  "attributes": {
    "values": [
      {
        "type": "identityAttribute",
        "attributes": {
          "name": "personal_email"
        }
      },
      {
        "type": "accountAttribute",
        "attributes": {
          "sourceName": "Active Directory",
          "attributeName": "mail"
        }
      },
      {
        "type": "concat",
        "attributes": {
          "values": [
            {
              "type": "identityAttribute",
              "attributes": {
                "name": "firstname"
              }
            },
            {
              "type": "static",
              "attributes": {
                "value": "."
              }
            },
            {
              "type": "identityAttribute",
              "attributes": {
                "name": "lastname"
              }
            },
            {
              "type": "static",
              "attributes": {
                "value": "@company.com"
              }
            }
          ],
          "next": {
            "type": "lower"
          }
        }
      },
      {
        "type": "static",
        "attributes": {
          "value": "noemail@company.com"
        }
      }
    ],
    "ignoreErrors": true
  }
}
```

---

#### Pattern 4: Clean Phone Number

**Use Case:** Remove formatting from phone
```json
{
  "name": "phoneClean",
  "type": "replace",
  "attributes": {
    "input": {
      "type": "identityAttribute",
      "attributes": {
        "name": "phone"
      }
    },
    "regex": "\\(",
    "replacement": "",
    "next": {
      "type": "replace",
      "attributes": {
        "regex": "\\)",
        "replacement": "",
        "next": {
          "type": "replace",
          "attributes": {
            "regex": " ",
            "replacement": "",
            "next": {
              "type": "replace",
              "attributes": {
                "regex": "-",
                "replacement": ""
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

#### Pattern 5: Lifecycle State Calculation

**Use Case:** Determine lifecycle state
```json
{
  "name": "lifecycleState",
  "type": "conditional",
  "attributes": {
    "expression": "leave_flag == true",
    "positiveCondition": "Leave of Absence",
    "negativeCondition": {
      "type": "conditional",
      "attributes": {
        "expression": "isPreHire == 'true'",
        "positiveCondition": "Pre-hire",
        "negativeCondition": {
          "type": "conditional",
          "attributes": {
            "expression": "isTerminated == 'true'",
            "positiveCondition": "Terminated",
            "negativeCondition": "Active"
          }
        }
      }
    }
  }
}
```

---

#### Pattern 6: Username with Truncation

**Use Case:** Generate 20-character max username
```json
{
  "name": "username",
  "type": "concat",
  "attributes": {
    "values": [
      {
        "type": "identityAttribute",
        "attributes": {
          "name": "firstname"
        }
      },
      {
        "type": "static",
        "attributes": {
          "value": "."
        }
      },
      {
        "type": "identityAttribute",
        "attributes": {
          "name": "lastname"
        }
      }
    ],
    "next": {
      "type": "lower",
      "next": {
        "type": "substring",
        "attributes": {
          "begin": 0,
          "end": 20
        }
      }
    }
  }
}
```

---

#### Pattern 7: Extract First AD Group

**Use Case:** Get primary group from AD
```json
{
  "name": "primaryGroup",
  "type": "accountAttribute",
  "attributes": {
    "sourceName": "Active Directory",
    "attributeName": "memberOf",
    "next": {
      "type": "index",
      "attributes": {
        "position": 0,
        "next": {
          "type": "split",
          "attributes": {
            "delimiter": ",",
            "next": {
              "type": "index",
              "attributes": {
                "position": 0,
                "next": {
                  "type": "split",
                  "attributes": {
                    "delimiter": "=",
                    "next": {
                      "type": "index",
                      "attributes": {
                        "position": 1
                      }
                    }
                  }
                }
              }
            }
          }
        }
      }
    }
  }
}
```

**What it does:**
```
Input: ["CN=Admins,OU=Groups,DC=company,DC=com", ...]
→ Index(0): "CN=Admins,OU=Groups,DC=company,DC=com"
→ Split(","): ["CN=Admins", "OU=Groups", "DC=company", "DC=com"]
→ Index(0): "CN=Admins"
→ Split("="): ["CN", "Admins"]
→ Index(1): "Admins"

Output: "Admins"
```

---

#### Pattern 8: Date Comparison for Pre-Hire

**Use Case:** Check if future hire date
```json
{
  "name": "isPreHire",
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
        "operator": "gt",
        "secondDate": {
          "type": "now"
        }
      }
    }
  }
}
```

---

## Afternoon Session: Identity Profile JSON

### Part 4: Identity Profile Structure (60 minutes)

**Understanding the complete Identity Profile JSON structure**

---

#### Complete Identity Profile JSON

**Here's what a full Identity Profile looks like in JSON:**
```json
{
  "id": "2c91808a7xxxxxxx",
  "name": "Employees",
  "description": "Profile for all employees from Workday",
  "owner": {
    "type": "IDENTITY",
    "id": "owner-identity-id",
    "name": "admin"
  },
  "priority": 10,
  "sourceIds": ["workday-source-id"],
  "identityRefreshRequired": true,
  "identityCount": 5000,
  "identityAttributeConfig": {
    "enabled": true,
    "attributeTransforms": [
      {
        "identityAttributeName": "displayName",
        "transformDefinition": {
          "type": "concat",
          "attributes": {
            "values": [
              {
                "type": "identityAttribute",
                "attributes": {
                  "name": "firstname"
                }
              },
              {
                "type": "static",
                "attributes": {
                  "value": " "
                }
              },
              {
                "type": "identityAttribute",
                "attributes": {
                  "name": "lastname"
                }
              }
            ]
          }
        }
      },
      {
        "identityAttributeName": "email",
        "transformDefinition": {
          "type": "concat",
          "attributes": {
            "values": [
              {
                "type": "identityAttribute",
                "attributes": {
                  "name": "firstname"
                }
              },
              {
                "type": "static",
                "attributes": {
                  "value": "."
                }
              },
              {
                "type": "identityAttribute",
                "attributes": {
                  "name": "lastname"
                }
              },
              {
                "type": "static",
                "attributes": {
                  "value": "@company.com"
                }
              }
            ]
          },
          "next": {
            "type": "lower"
          }
        }
      }
    ]
  },
  "identityExceptionReportReference": null,
  "hasTimeBasedAttr": true
}
```

---

#### Key Sections Explained

**Section 1: Basic Info**
```json
{
  "id": "unique-id",
  "name": "Profile Name",
  "description": "What this profile is for",
  "priority": 10
}
```

**Priority:** Lower number = higher priority (if multiple profiles match)

---

**Section 2: Source Linkage**
```json
{
  "sourceIds": ["workday-source-id"]
}
```

This links the profile to the authoritative source (usually HR system)

---

**Section 3: Transform Configuration**
```json
{
  "identityAttributeConfig": {
    "enabled": true,
    "attributeTransforms": [
      {
        "identityAttributeName": "attributeName",
        "transformDefinition": {
          "type": "transformType",
          "attributes": {...}
        }
      }
    ]
  }
}
```

**Structure:**
```
identityAttributeConfig
└─ attributeTransforms (array)
   ├─ Transform 1
   │  ├─ identityAttributeName: "displayName"
   │  └─ transformDefinition: {transform JSON}
   ├─ Transform 2
   │  ├─ identityAttributeName: "email"
   │  └─ transformDefinition: {transform JSON}
   └─ Transform 3...
```

---

#### Modifying Identity Profile JSON

**To add a new transform:**

**Step 1: Export current profile JSON**

**Step 2: Find the attributeTransforms array**
```json
"attributeTransforms": [
  {existing transforms}
]
```

**Step 3: Add your new transform**
```json
"attributeTransforms": [
  {existing transforms},
  {
    "identityAttributeName": "newAttribute",
    "transformDefinition": {
      "type": "static",
      "attributes": {
        "value": "new value"
      }
    }
  }
]
```

**Step 4: Import back via API** (covered in Day 4)

---

### Part 5: Source Configuration JSON (60 minutes)

**Understanding Source (connector) configurations**

---

#### Source JSON Structure
```json
{
  "id": "source-id",
  "name": "Workday",
  "description": "Workday HR System",
  "type": "DelimitedFile",
  "connectorAttributes": {
    "host": "files.workday.com",
    "authenticationType": "UsernamePassword",
    "delimiter": ",",
    "columnNames": [
      "employeeId",
      "firstname",
      "lastname",
      "email",
      "department"
    ]
  },
  "accountSchema": {
    "id": "schema-id",
    "name": "account",
    "attributes": [
      {
        "name": "employeeId",
        "type": "string",
        "description": "Employee ID"
      },
      {
        "name": "firstname",
        "type": "string",
        "description": "First Name"
      },
      {
        "name": "lastname",
        "type": "string",
        "description": "Last Name"
      }
    ],
    "identityAttribute": "employeeId"
  },
  "authoritative": true,
  "managementWorkgroup": null,
  "healthy": true,
  "status": "SOURCE_STATE_HEALTHY"
}
```

---

#### Key Sections

**Connector Attributes:**
```json
{
  "connectorAttributes": {
    "host": "server.com",
    "authenticationType": "OAuth2",
    "delimiter": ",",
    "columnNames": [...]
  }
}
```

**Schema Definition:**
```json
{
  "accountSchema": {
    "attributes": [
      {
        "name": "attributeName",
        "type": "string|int|boolean",
        "description": "What this is"
      }
    ],
    "identityAttribute": "unique key field"
  }
}
```

**Authoritative Flag:**
```json
{
  "authoritative": true
}
```

`true` = This source creates identities  
`false` = This source only adds accounts to existing identities

---

## Evening Session: Advanced JSON Operations

### Part 6: Bulk Operations with JSON (60 minutes)

**Using JSON to manage multiple transforms efficiently**

---

#### Scenario: Bulk Transform Creation

**Goal:** Create 10 similar transforms at once

**Manual Approach (slow):**
```
1. Go to UI
2. Create transform 1
3. Click, click, click...
4. Create transform 2
5. Click, click, click...
... repeat 10 times
```

**JSON Approach (fast):**
```
1. Write JSON for all 10 transforms
2. POST to API once
3. Done!
```

---

#### Template-Based Generation

**Create a template:**
```json
{
  "name": "REPLACE_NAME",
  "type": "static",
  "attributes": {
    "value": "REPLACE_VALUE"
  }
}
```

**Generate 10 transforms programmatically:**
```javascript
// JavaScript example (could be Python, etc.)
const template = {
  "name": "REPLACE_NAME",
  "type": "static",
  "attributes": {
    "value": "REPLACE_VALUE"
  }
};

const transforms = [];
const departments = [
  "Engineering", "Finance", "HR", "IT", "Sales",
  "Marketing", "Operations", "Legal", "Support", "Admin"
];

departments.forEach(dept => {
  const transform = JSON.parse(JSON.stringify(template)); // Deep copy
  transform.name = `default${dept}`;
  transform.attributes.value = dept;
  transforms.push(transform);
});

// Now POST each transform via API
transforms.forEach(transform => {
  // API call to create transform
});
```

**Result:** 10 transforms created in seconds!

---

#### Bulk Modification

**Scenario:** Change "@company.com" to "@newcompany.com" in 20 transforms

**Steps:**

**1. Export all transforms via API**
```bash
GET /v3/transforms
```

**2. Load JSON into text editor**

**3. Find and replace**
```
Find: "@company.com"
Replace: "@newcompany.com"
```

**4. Save modified JSON**

**5. Update via API**
```bash
PATCH /v3/transforms/{id}
```

**This beats clicking through 20 transforms in the UI!**

---

#### Version Control with JSON

**Best Practice:** Store transform JSON in Git

**Workflow:**
```
1. Export all transforms to JSON files
2. Store in Git repository
3. Make changes in JSON
4. Commit to Git (tracked history!)
5. Deploy via API
```

**File structure:**
```
sailpoint-config/
├─ transforms/
│  ├─ displayName.json
│  ├─ email.json
│  ├─ username.json
│  └─ lifecycleState.json
├─ profiles/
│  └─ employees.json
└─ README.md
```

**Benefits:**
```
✓ Track all changes (who, when, what)
✓ Rollback to previous versions
✓ Compare versions (diff)
✓ Collaborate with team
✓ Deploy across environments (dev → prod)
```

---

#### Environment Promotion

**Problem:** How to move configs from Dev to Prod?

**Solution:** JSON export/import

**Steps:**
```
Dev Environment:
1. Build and test transform
2. Export transform JSON
3. Save to file

Prod Environment:
4. Import JSON file
5. POST to API
6. Transform created!
```

**No manual recreation needed!**

---

### Practice Exercise 2: Bulk Transform Creation

**Task:** Create JSON for 5 static transforms

**Transforms needed:**
1. defaultCountry = "United States"
2. defaultTimezone = "America/Chicago"
3. defaultCurrency = "USD"
4. defaultLanguage = "en_US"
5. defaultRegion = "North America"

**Build all 5 in JSON format:**

<details>
<summary>Click to reveal answer</summary>
```json
[
  {
    "name": "defaultCountry",
    "type": "static",
    "attributes": {
      "value": "United States"
    }
  },
  {
    "name": "defaultTimezone",
    "type": "static",
    "attributes": {
      "value": "America/Chicago"
    }
  },
  {
    "name": "defaultCurrency",
    "type": "static",
    "attributes": {
      "value": "USD"
    }
  },
  {
    "name": "defaultLanguage",
    "type": "static",
    "attributes": {
      "value": "en_US"
    }
  },
  {
    "name": "defaultRegion",
    "type": "static",
    "attributes": {
      "value": "North America"
    }
  }
]
```

**This is an array of transforms - ready to bulk create via API!**

</details>

---

## Day 3 Assessment

### Knowledge Check

#### Question 1: Transform Structure

What are the 3 required top-level properties in every transform?

<details>
<summary>Click to reveal answer</summary>
```json
{
  "name": "...",      // Required
  "type": "...",      // Required
  "attributes": {...} // Required
}
```

**Three required properties:**
1. `name` - The attribute being created
2. `type` - The transform type
3. `attributes` - Configuration for that transform type

</details>

---

#### Question 2: Chaining Transforms

How do you chain transforms together in JSON?

<details>
<summary>Click to reveal answer</summary>

**Use the "next" property:**
```json
{
  "type": "trim",
  "attributes": {
    "input": {...},
    "next": {
      "type": "lower",
      "next": {
        "type": "concat",
        "attributes": {...}
      }
    }
  }
}
```

Each "next" property contains the next transform in the chain.

</details>

---

#### Question 3: FirstValid Pattern

Write JSON for an email attribute with 3 fallbacks:
1. personal_email
2. work_email
3. "noemail@company.com"

<details>
<summary>Click to reveal answer</summary>
```json
{
  "name": "email",
  "type": "firstValid",
  "attributes": {
    "values": [
      {
        "type": "identityAttribute",
        "attributes": {
          "name": "personal_email"
        }
      },
      {
        "type": "identityAttribute",
        "attributes": {
          "name": "work_email"
        }
      },
      {
        "type": "static",
        "attributes": {
          "value": "noemail@company.com"
        }
      }
    ],
    "ignoreErrors": true
  }
}
```

</details>

---

### Final Challenge: Build Complex Transform

**Create a transform for "email" that:**
1. Trims firstname
2. Trims lastname
3. Concatenates: firstname + "." + lastname + "@company.com"
4. Converts to lowercase
5. Truncates to 50 characters

<details>
<summary>Click to reveal answer</summary>
```json
{
  "name": "email",
  "type": "trim",
  "attributes": {
    "input": {
      "type": "identityAttribute",
      "attributes": {
        "name": "firstname"
      }
    },
    "next": {
      "type": "concat",
      "attributes": {
        "values": [
          {
            "type": "reference"
          },
          {
            "type": "static",
            "attributes": {
              "value": "."
            }
          },
          {
            "type": "trim",
            "attributes": {
              "input": {
                "type": "identityAttribute",
                "attributes": {
                  "name": "lastname"
                }
              }
            }
          },
          {
            "type": "static",
            "attributes": {
              "value": "@company.com"
            }
          }
        ],
        "next": {
          "type": "lower",
          "next": {
            "type": "substring",
            "attributes": {
              "begin": 0,
              "end": 50
            }
          }
        }
      }
    }
  }
}
```

**Transform chain:**
```
Trim(firstname)
→ Concat(result + "." + Trim(lastname) + "@company.com")
→ Lower
→ Substring(0, 50)
```

</details>

---

## Day 3 Summary

### What You Learned Today

**Morning:**
- ✅ Complete reference for all 18+ transform types
- ✅ How to export transforms from ISC
- ✅ Building a transform patterns library

**Afternoon:**
- ✅ Identity Profile JSON structure
- ✅ Source configuration JSON
- ✅ Modifying configurations via JSON

**Evening:**
- ✅ Bulk transform operations
- ✅ Version control with JSON
- ✅ Environment promotion strategies

---

### Skills Acquired

You can now:
```
✓ Write any SailPoint transform type in JSON
✓ Chain complex multi-step transforms
✓ Export configurations from ISC
✓ Modify configurations via JSON
✓ Create bulk transforms efficiently
✓ Use version control for configs
✓ Promote configs across environments
✓ Build reusable pattern libraries
```

---

### Transform Quick Reference

**Save this for rapid development:**

| Transform | Purpose | Key Properties |
|-----------|---------|----------------|
| static | Fixed value | value |
| concat | Join values | values[] |
| substring | Extract portion | begin, end |
| upper/lower | Change case | input |
| trim | Remove spaces | input |
| replace | Find/replace | regex, replacement |
| conditional | If-then-else | expression, positiveCondition, negativeCondition |
| firstValid | Fallback chain | values[], ignoreErrors |
| lookup | Map values | table, default |
| dateFormat | Convert dates | inputFormat, outputFormat |
| dateMath | Add/subtract | operation, value, unit |
| dateCompare | Compare dates | operator, firstDate, secondDate |
| split | String to array | delimiter |
| index | Get array item | position |
| join | Array to string | delimiter |
| accountAttribute | Get from account | sourceName, attributeName |

---

## Homework

### Exercise 1: Pattern Library

Create JSON for these 3 patterns:

1. **Smart Username:** Generate from firstname.lastname, lowercase, max 20 chars
2. **Department Normalization:** Trim + uppercase dept_code
3. **Phone Cleaner:** Remove (, ), -, and spaces

<details>
<summary>Click for hints</summary>

1. Use: Concat → Lower → Substring
2. Use: Trim → Upper
3. Use: Multiple Replace transforms chained

</details>

---

### Exercise 2: Bulk Creation

Create JSON array of 5 lookup-based transforms:
- deptToName (dept_code → department name)
- titleToLevel (job_title → employee level)
- locationToTimezone (city → timezone)
- countryToRegion (country → region)
- statusToLifecycle (status → lifecycle state)

All should use lookup tables with "Unknown" default.

---

## Tomorrow: Day 4 Preview

**What You'll Learn:**
```
SailPoint API with JSON
├─ Authentication (access tokens)
├─ GET (read configurations)
├─ POST (create transforms)
├─ PATCH (update configurations)
├─ DELETE (remove transforms)
└─ Automation with Postman
```

**You'll be able to:**
- Use SailPoint API with confidence
- Create transforms via API
- Automate configuration management
- Build deployment scripts
- Manage ISC programmatically

---

**Congratulations on completing Day 3! You now master SailPoint ISC JSON!**

**Tomorrow we unlock API automation!**

---

*End of JSON Mastery - Day 3*
