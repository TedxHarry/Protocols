# 🎓 FOUNDATION PHASE - Lesson 2
## Transform JSON Structure and Syntax

---

## What You'll Master
- ✅ Understand how to read transform JSON
- ✅ Know what each part means and why it matters
- ✅ Learn data source types and how to reference them
- ✅ Build your first real transforms

---

## The Universal Transform Pattern

Every single transform follows this pattern:

```json
{
  "name": "descriptive_name_here",
  "type": "operation_type_here",
  "attributes": {
    "parameterName": "value",
    "inputValue": "source_of_data",
    "otherSettings": "as_needed"
  }
}
```

That's it. **Every transform, no matter how complex, follows this structure.**

### The Three Required Parts

| Part | Purpose | Example |
|------|---------|---------|
| **name** | Unique identifier (used to reference this transform elsewhere) | `"Email_Lowercase"` |
| **type** | The operation being performed | `"lower"`, `"concatenation"`, `"conditional"` |
| **attributes** | Configuration details for that operation type | Varies by operation |

---

## Real Example: Breaking Down a Transform

Let's take a real transform and understand every piece:

```json
{
  "name": "Email_Standardize_Remove_Spaces",
  "type": "replace",
  "attributes": {
    "input": {
      "type": "identityAttribute",
      "attributes": {
        "name": "email"
      }
    },
    "old": " ",
    "new": ""
  }
}
```

### Line-by-Line Explanation

```json
{
  "name": "Email_Standardize_Remove_Spaces",
```
**What it is:** The transform's unique name  
**Why it matters:** You'll reference this when using the transform elsewhere  
**Naming convention:** Use descriptive, snake_case names  
**Good names:**
- ✅ `"Email_Lowercase"`
- ✅ `"Username_Generate_Unique"`
- ✅ `"FirstName_Decompose_Diacritical"`

**Bad names:**
- ❌ `"T1"` (too vague)
- ❌ `"Transform_1"` (not descriptive)
- ❌ `"fix_email"` (unclear what it does)

---

```json
  "type": "replace",
```
**What it is:** The operation this transform performs  
**What it does:** Replace all instances of a substring with another substring  
**Why this one:** We want to remove spaces, which is a "replace" operation  

**Note:** The `type` determines what parameters go in `attributes`. Different types have different requirements.

---

```json
  "attributes": {
```
**What it is:** Container for all the configuration details  
**What goes here:** Parameters specific to this operation type  
**Important:** Structure of attributes depends on the type

---

```json
    "input": {
      "type": "identityAttribute",
      "attributes": {
        "name": "email"
      }
    },
```
**What it is:** Where the data comes from (the input)  
**What it says:** "Get the email attribute from this identity"

Let's break this down further:

| Part | Meaning |
|------|---------|
| `"input"` | Parameter name (this is where the "replace" operation gets its text to search) |
| `"type": "identityAttribute"` | Get data from identity attributes (data already stored in ISC) |
| `"attributes": { "name": "email" }` | Specifically, get the attribute named "email" |

**Result:** This returns the user's email value. If the email is `"john.smith@company.com"`, that's what "input" becomes.

---

```json
    "old": " ",
    "new": ""
```
**What it is:** Parameters specific to the "replace" operation  
**What they mean:**
- `"old": " "` → Find this substring (a space character)
- `"new": ""` → Replace it with this (empty string)

**In plain English:** Replace all space characters with nothing (effectively removing spaces)

---

### Putting It All Together

This transform:
```json
{
  "name": "Email_Standardize_Remove_Spaces",
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

**Does this:**
1. Get the identity's email attribute
2. Find all space characters
3. Replace them with nothing
4. Return the result

**Example:**
```
Input:  "john smith @ company.com"
Output: "johnsmith@company.com"
```

---

## The Four Data Source Types

In transforms, data comes from somewhere. You need to know how to reference each source.

### Source Type #1: Identity Attribute

**What it is:** Data from the user's ISC identity profile

**When to use:** 
- ✅ In provisioning (ISC → target system)
- ✅ In aggregation when reading already-stored identity data

**JSON Structure:**
```json
{
  "type": "identityAttribute",
  "attributes": {
    "name": "attributeName"
  }
}
```

**Real Example:**
```json
{
  "type": "identityAttribute",
  "attributes": {
    "name": "email"
  }
}
```
Returns the user's email from their ISC identity.

**Common identity attributes:**
- `firstName` - User's first name
- `lastName` - User's last name
- `email` - User's email address
- `manager` - User's manager (relationship)
- `department` - User's department
- `jobCode` - User's job classification code

---

### Source Type #2: Account Attribute

**What it is:** Data from a user's account on a specific source system

**When to use:**
- ✅ In aggregation (pulling from source into ISC)
- ✅ When the data hasn't been stored as an identity attribute yet
- ✅ When you need to reference data from a different source

**JSON Structure:**
```json
{
  "type": "accountAttribute",
  "attributes": {
    "sourceName": "SourceSystemName",
    "attributeName": "accountAttributeName"
  }
}
```

**Real Example #1: Get email from Workday during aggregation**
```json
{
  "type": "accountAttribute",
  "attributes": {
    "sourceName": "Workday",
    "attributeName": "email"
  }
}
```
This gets the email attribute from the user's Workday account.

**Real Example #2: Get AD username during AD aggregation**
```json
{
  "type": "accountAttribute",
  "attributes": {
    "sourceName": "Active Directory",
    "attributeName": "sAMAccountName"
  }
}
```
This gets the AD username from the user's AD account.

**Common source names (match your ISC configuration):**
- `"Active Directory"`
- `"Workday"`
- `"ServiceNow"`
- `"SAP"`
- `"HR_JDBC"` (example JDBC connection)

**Key difference from Identity Attribute:**
```
Identity Attribute: References data already stored in ISC
Account Attribute: References data from a source system
```

---

### Source Type #3: Static Value

**What it is:** A fixed, unchanging value

**When to use:**
- ✅ When you need a constant value (like a domain)
- ✅ When combining with other transforms (static + dynamic = result)
- ✅ For Velocity template logic (covered later)

**JSON Structure:**
```json
{
  "type": "static",
  "attributes": {
    "value": "literal_value_here"
  }
}
```

**Real Example #1: Add email domain**
```json
{
  "type": "static",
  "attributes": {
    "value": "@company.com"
  }
}
```
This contributes `"@company.com"` to whatever operation uses it.

**Real Example #2: Set a constant status**
```json
{
  "type": "static",
  "attributes": {
    "value": "ACTIVE"
  }
}
```
This contributes `"ACTIVE"` - used to set everyone's status to active by default.

**Real Example #3: Fallback value if data is missing**
```json
{
  "type": "static",
  "attributes": {
    "value": "unknown@company.com"
  }
}
```
Used as a fallback when real email is NULL.

---

### Source Type #4: First Valid

**What it is:** Try multiple sources in order; use the first one that's not NULL

**When to use:**
- ✅ When you have multiple possible sources (fallback logic)
- ✅ When you need to handle missing data gracefully
- ✅ Almost always for production safety

**JSON Structure:**
```json
{
  "type": "firstValid",
  "attributes": {
    "values": [
      { source_option_1 },
      { source_option_2 },
      { source_option_3 }
    ]
  }
}
```

**Real Example: Get email from best available source**
```json
{
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
        "type": "static",
        "attributes": {
          "value": "noemail@company.com"
        }
      }
    ]
  }
}
```

**How it works:**
1. Try AD's mail attribute → if found and not NULL, use it ✓
2. If NULL, try Workday's email → if found and not NULL, use it ✓
3. If still NULL, use default value `"noemail@company.com"` ✓
4. Never returns NULL ✓

**Execution example:**
```
User 1: AD email exists → "john@company.com" ✓
User 2: AD email NULL, Workday email exists → "john.smith@workday.com" ✓
User 3: Both NULL → "noemail@company.com" ✓
```

---

## Building Blocks: Attributes Parameter

The `attributes` section is where the operation's parameters go. **Different operation types need different attributes.**

### Attributes Are Operation-Specific

```json
// Replace operation needs: input, old, new
{
  "type": "replace",
  "attributes": {
    "input": { ... },
    "old": "find_this",
    "new": "replace_with_this"
  }
}

// Substring operation needs: input, begin, end
{
  "type": "substring",
  "attributes": {
    "input": { ... },
    "begin": 0,
    "end": 5
  }
}

// Conditional operation needs: expression, positiveCondition, negativeCondition
{
  "type": "conditional",
  "attributes": {
    "expression": "$email.contains('@')",
    "positiveCondition": "valid",
    "negativeCondition": "invalid"
  }
}
```

**Key insight:** You learn what attributes each operation needs as you learn the operations themselves (Lesson 3).

---

## Nesting: The Foundation of Complex Transforms

Transforms can contain other transforms. This is how you build sophisticated logic.

### Simple Nesting Example

**Goal:** Create email from firstName.lastName, make it lowercase

```json
{
  "name": "Email_From_Name_Lowercase",
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
            "type": "identityAttribute",
            "attributes": { "name": "lastName" }
          }
        ]
      }
    }
  }
}
```

### Breaking Down the Nesting

```
Outer operation: "lower" (convert to lowercase)
  ↓
  Takes as input: a "concatenation" operation
    ↓
    Which concatenates three things:
      1. firstName attribute
      2. Static "."
      3. lastName attribute
```

**Execution flow:**
```
Step 1: Get firstName → "John"
Step 2: Get static → "."
Step 3: Get lastName → "Smith"
Step 4: Concatenate all → "John.Smith"
Step 5: Lower case it → "john.smith"
Final output: "john.smith"
```

### Visual Representation

```
┌─────────────────────────────────┐
│  Lower (convert to lowercase)   │ ← Outer operation
└────────────┬────────────────────┘
             │ operates on
             ↓
┌─────────────────────────────────┐
│  Concatenation (join strings)   │ ← Inner operation
├─────────────────────────────────┤
│  · firstName = "John"           │
│  · Static = "."                 │
│  · lastName = "Smith"           │
└─────────────────────────────────┘
             │
             ↓
Result: "john.smith"
```

---

## Pattern Recognition: Common Patterns You'll See

### Pattern #1: Data + Formatting

```json
{
  "type": "lower",
  "attributes": {
    "input": {
      "type": "identityAttribute",
      "attributes": { "name": "email" }
    }
  }
}
```
**What it does:** Get email, make it lowercase  
**When you use it:** Standardizing email format

### Pattern #2: Data + Fallback

```json
{
  "type": "firstValid",
  "attributes": {
    "values": [
      { "type": "accountAttribute", ... },
      { "type": "accountAttribute", ... },
      { "type": "static", ... }
    ]
  }
}
```
**What it does:** Try primary source, then secondary, then use default  
**When you use it:** Handling missing data safely

### Pattern #3: Data + Conditional

```json
{
  "type": "conditional",
  "attributes": {
    "expression": "condition_here",
    "positiveCondition": "value_if_true",
    "negativeCondition": "value_if_false"
  }
}
```
**What it does:** Apply logic: if X then Y else Z  
**When you use it:** Making decisions based on data

### Pattern #4: Data + Modification + Formatting

```json
{
  "type": "lower",
  "attributes": {
    "input": {
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
  }
}
```
**What it does:** Get email → remove spaces → make lowercase  
**When you use it:** Complex standardization (multiple steps)

---

## Real Transform Examples from Production

### Example 1: Generate Username from Name Components

```json
{
  "name": "Username_FirstInitial_LastName",
  "type": "concatenation",
  "attributes": {
    "values": [
      {
        "type": "substring",
        "attributes": {
          "input": {
            "type": "identityAttribute",
            "attributes": { "name": "firstName" }
          },
          "begin": 0,
          "end": 1
        }
      },
      {
        "type": "lower",
        "attributes": {
          "input": {
            "type": "identityAttribute",
            "attributes": { "name": "lastName" }
          }
        }
      }
    ]
  }
}
```

**What it does:**
1. Get first character of firstName (J from John)
2. Get entire lastName in lowercase (smith from Smith)
3. Concatenate them together
4. Result: "jsmith"

---

### Example 2: Email with Fallback from Multiple Sources

```json
{
  "name": "Email_MultiSource_Fallback",
  "type": "lower",
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
    }
  }
}
```

**What it does:**
1. Try AD email first
2. If NULL, try Workday email
3. If still NULL, generate from firstName.lastName@company.com
4. Convert final result to lowercase
5. Never returns NULL

---

### Example 3: Classify Employee Type

```json
{
  "name": "EmploymentType_Classification",
  "type": "conditional",
  "attributes": {
    "expression": "$department == 'Contractor'",
    "positiveCondition": "CONTRACTOR",
    "negativeCondition": {
      "type": "conditional",
      "attributes": {
        "expression": "$jobCode.startsWith('M')",
        "positiveCondition": "MANAGER",
        "negativeCondition": "EMPLOYEE"
      }
    }
  }
}
```

**What it does:**
1. Check if department = "Contractor"
   - YES → return "CONTRACTOR"
   - NO → check next condition
2. Check if jobCode starts with "M"
   - YES → return "MANAGER"
   - NO → return "EMPLOYEE"

---

## Key Concepts to Internalize

### Concept #1: Every Transform Has These Parts
```
Name (what to call it)
  ↓
Type (what operation to perform)
  ↓
Attributes (how to configure it)
```

### Concept #2: Attributes Depend on Type
Different operation types need different parameters. You look up what attributes each type needs.

### Concept #3: Data Sources
You reference data from:
- **identityAttribute** - ISC identity attributes
- **accountAttribute** - Source system attributes
- **static** - Fixed values
- **firstValid** - Multiple sources with fallback

### Concept #4: Nesting Enables Complexity
Simple transforms handle simple cases.  
Nested transforms handle complex cases.  
Deep nesting becomes hard to read—avoid it.

### Concept #5: Null Handling
Always think: "What if this data is missing?"
Use FirstValid to provide fallback values.

---

## Practice: Read This Transform

Without looking at the explanation, read this transform and explain what it does:

```json
{
  "name": "Department_Code_to_Name",
  "type": "lookup",
  "attributes": {
    "input": {
      "type": "identityAttribute",
      "attributes": { "name": "departmentCode" }
    },
    "map": {
      "IT": "Information Technology",
      "HR": "Human Resources",
      "FIN": "Finance"
    },
    "defaultValue": "Other"
  }
}
```

<details>
<summary>Click to reveal explanation</summary>

**This transform:**
1. Gets the identity's `departmentCode` attribute
2. Looks it up in a map:
   - "IT" → becomes "Information Technology"
   - "HR" → becomes "Human Resources"
   - "FIN" → becomes "Finance"
   - Anything else → becomes "Other"
3. Returns the mapped value

**Example:**
- Input: departmentCode = "IT"
- Output: "Information Technology"

**Use case:** Convert department codes to human-readable names.

</details>

---

## Mastery Checkpoint #2

### Question 1: Structure
What are the three required parts of a transform JSON?

<details>
<summary>Answer</summary>

1. **name** - Unique identifier
2. **type** - Operation type
3. **attributes** - Configuration parameters

</details>

---

### Question 2: Data Sources
You're in aggregation from Workday. You need the user's Workday employee ID. Should you use `identityAttribute` or `accountAttribute`?

<details>
<summary>Answer</summary>

**accountAttribute** - Because the data is coming FROM Workday (the source), not from ISC (the identity).

If you were in provisioning and needed to reference the already-stored employee ID in ISC, you'd use **identityAttribute**.

</details>

---

### Question 3: FirstValid Logic
In this transform, which value will be returned if the user has no AD email and no Workday email?

```json
{
  "type": "firstValid",
  "attributes": {
    "values": [
      { "type": "accountAttribute", "attributes": { "sourceName": "AD", "attributeName": "mail" } },
      { "type": "accountAttribute", "attributes": { "sourceName": "Workday", "attributeName": "email" } },
      { "type": "static", "attributes": { "value": "noemail@company.com" } }
    ]
  }
}
```

<details>
<summary>Answer</summary>

**"noemail@company.com"** - The static value at the end.

FirstValid tries each source in order until it finds a non-NULL value. The static value is the fallback.

</details>

---

### Question 4: Nesting
In this transform, which operation runs first?

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
          { "type": "identityAttribute", "attributes": { "name": "lastName" } }
        ]
      }
    }
  }
}
```

<details>
<summary>Answer</summary>

**Concatenation runs first** (the inner operation), then **lower** (the outer operation) converts the result to lowercase.

Data flows from inside-out: inner operations complete, then outer operations work on their results.

</details>

---

## Summary: Lesson 2 Essentials

| Topic | Key Takeaway |
|-------|--------------|
| **Structure** | All transforms: name + type + attributes |
| **Data Sources** | identityAttribute, accountAttribute, static, firstValid |
| **Attributes** | Different types need different parameters |
| **Nesting** | Inner operations complete, outer operations use their results |
| **Null Handling** | Use firstValid with fallback values |
| **Naming** | Use descriptive, snake_case names |

---

## What's Next

In **Lesson 3**, you'll:
- ✅ Master all major operation types
- ✅ Learn when to use each operation
- ✅ See operation families and patterns
- ✅ Build increasingly complex transforms

---

## Quick Reference: Transform Anatomy

```json
{
  "name": "Unique_Identifier_For_This_Transform",
  "type": "the_operation_type",
  "attributes": {
    "input": {
      "type": "where_data_comes_from",
      "attributes": {
        "specificDetails": "depends_on_source_type"
      }
    },
    "otherParam1": "operation_specific_parameters",
    "otherParam2": "operation_specific_parameters"
  }
}
```

---

**Homework before Lesson 3:**
1. Look at the 3 real examples provided
2. For each, trace the data flow step-by-step
3. Think about: "What if firstName was NULL in example 2?"
4. Come back with any questions!
