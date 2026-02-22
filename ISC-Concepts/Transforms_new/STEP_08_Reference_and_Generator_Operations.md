# 🎓 FOUNDATION PHASE - Lesson 6
## Reference and Generator Operations

---

## What You'll Master
- ✅ Reference operation (reuse transforms, avoid duplication)
- ✅ Generator operations (create usernames, passwords, unique IDs)
- ✅ Lookup operation (translate codes to values)
- ✅ Build practical, maintainable transforms
- ✅ Know when NOT to nest (when to use Reference instead)

---

## Why This Lesson Matters

By now you can build complex transforms with heavy nesting. But there's a problem:

**What if you have the same logic in 5 different transforms?**

```
Transform A: Build email (complex nesting)
Transform B: Build email (same nesting, copy-pasted)
Transform C: Build email (same nesting again)

Problem: Change the logic once? You have to update 3 transforms.
```

**Solution: Reference operation**

Also, some problems are so common they need specialized operations:
- **Generate unique usernames** (UsernameGenerator)
- **Generate random passwords** (GenerateRandomString)
- **Create unique IDs** (UUIDGenerator)
- **Translate codes** (Lookup)

These are today's lesson.

---

## Operation #1: Reference

### What It Does
Reuses a transform you've already created. Lets you reference it by ID instead of rewriting the logic.

### JSON Structure
```json
{
  "type": "reference",
  "attributes": {
    "id": "transform-id-here"
  }
}
```

### Real Example: Reuse Complex Email Logic

**Scenario:** You built `Email_Standardize` in Lesson 5. Now you need that same logic in 3 different places.

**Without Reference (Bad):**
```
Transform A: Full email standardization logic (50 lines)
Transform B: Same logic (50 lines copy-pasted)
Transform C: Same logic (50 lines copy-pasted)
Total: 150 lines of duplicate code
```

**With Reference (Good):**
```
Transform "Email_Standardize": Full logic (50 lines)
Transform A: reference → Email_Standardize
Transform B: reference → Email_Standardize
Transform C: reference → Email_Standardize
Total: 50 lines of logic + 3 simple references
```

### Building a Reusable Transform

**Step 1: Create the base transform**

```json
{
  "name": "Email_Standardize_Base",
  "type": "lower",
  "attributes": {
    "input": {
      "type": "replace",
      "attributes": {
        "input": {
          "type": "firstValid",
          "attributes": {
            "values": [
              { "type": "identityAttribute", "attributes": { "name": "email" } },
              { "type": "static", "attributes": { "value": "noemail@company.com" } }
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

**Step 2: Get the transform ID**

When you create this in ISC, the system returns an ID (something like `"transform-12345abc"`).

**Step 3: Reference it elsewhere**

```json
{
  "name": "AD_Email_Provisioning",
  "type": "reference",
  "attributes": {
    "id": "transform-12345abc"
  }
}
```

This transform does EXACTLY what the base transform does, but you didn't write the logic twice.

### When to Use Reference

**Use Reference when:**
- ✅ Same logic is needed in multiple places
- ✅ You want to avoid copy-paste duplication
- ✅ You want to update logic in one place

**Don't use Reference when:**
- ❌ Logic is only used once
- ❌ You need slight variations of the logic
- ❌ You're nesting (just nest the logic directly)

### Common Pattern: Build Once, Reference Many Times

```
Organization has multiple systems:
  - Active Directory
  - ServiceNow
  - Salesforce
  - Workday

All need standardized email in the same format.

Solution:
1. Create: Email_Standard_Format (the base)
2. Reference it in AD Create Profile
3. Reference it in ServiceNow Create Profile
4. Reference it in Salesforce Create Profile
5. Reference it in Workday Create Profile

Now if email format changes, update ONE transform.
```

---

## Operation #2: UsernameGenerator

### What It Does
Automatically generates unique usernames based on a pattern, checking against a target system for duplicates.

### JSON Structure
```json
{
  "type": "usernameGenerator",
  "attributes": {
    "sourceCheck": {
      "sourceName": "TargetSystemName",
      "attributeName": "usernameField"
    },
    "pattern": "$firstName$lastName",
    "lookup": "email"
  }
}
```

### Pattern Variables

| Variable | Meaning | Example |
|----------|---------|---------|
| `$firstName` | User's first name | "John" |
| `$lastName` | User's last name | "Smith" |
| `$firstInitial` | First letter of first name | "J" |
| `$lastInitial` | First letter of last name | "S" |
| `$email` | User's email | "john.smith@company.com" |

### Real Example #1: Simple Pattern

**Requirement:** Generate AD username as firstName.lastName

```json
{
  "name": "AD_Username_Simple",
  "type": "usernameGenerator",
  "attributes": {
    "sourceCheck": {
      "sourceName": "Active Directory",
      "attributeName": "sAMAccountName"
    },
    "pattern": "$firstName.$lastName",
    "lookup": "email"
  }
}
```

**Execution:**

```
User: firstName="John", lastName="Smith"
Pattern: $firstName.$lastName
Generated: "john.smith"
Check AD: Is "john.smith" already used?
  NO → Use "john.smith"
  YES → Use "john.smith2", "john.smith3", etc.
```

### Real Example #2: Use Initials

**Requirement:** Generate username as first initial + lastName

```json
{
  "name": "ServiceNow_Username_Initial",
  "type": "usernameGenerator",
  "attributes": {
    "sourceCheck": {
      "sourceName": "ServiceNow",
      "attributeName": "user_id"
    },
    "pattern": "$firstInitial$lastName",
    "lookup": "email"
  }
}
```

**Execution:**

```
User: firstName="John", lastName="Smith"
Pattern: $firstInitial$lastName
Generated: "Jsmith"
Check ServiceNow: Is "Jsmith" used?
  NO → Use "Jsmith"
  YES → Use "Jsmith2", etc.
```

### Real Example #3: Complex Pattern

**Requirement:** Generate as firstName_lastName_YYYY (with year)

```json
{
  "name": "SAP_Username_Complex",
  "type": "usernameGenerator",
  "attributes": {
    "sourceCheck": {
      "sourceName": "SAP",
      "attributeName": "UNAME"
    },
    "pattern": "$firstName_$lastName",
    "lookup": "email"
  }
}
```

**Execution:**

```
User: firstName="John", lastName="Smith"
Pattern: $firstName_$lastName
Generated: "John_Smith"
Check SAP: Unique?
  NO → Use "John_Smith"
  YES → Use "John_Smith2"
```

### How UsernameGenerator Handles Duplicates

```
Generated username: "john.smith"

Check target system:
  "john.smith" exists?
    YES → Try "john.smith2"
      "john.smith2" exists?
        YES → Try "john.smith3"
        NO → Use "john.smith3"
```

It automatically appends numbers until finding a unique username.

### Important: Pattern Constraints

The pattern is checked against the target system. If the system has constraints (max 20 characters, no special chars), the pattern must respect that.

```
SAP allows max 12 characters, no dots.
Pattern: "$firstName.$lastName"
Generated: "john.smith" (10 chars - OK)
But if: firstName="Christopher", lastName="Wellington"
Generated: "christopher.wellington" (23 chars - TOO LONG!)
System will fail.

Better pattern for SAP: "$firstInitial$lastName" (shorter)
```

---

## Operation #3: GenerateRandomString and UUID

### GenerateRandomString: What It Does
Creates a random alphanumeric string of specified length.

### JSON Structure
```json
{
  "type": "generateRandomString",
  "attributes": {
    "length": 12
  }
}
```

### Real Example: Generate Temporary Password

```json
{
  "name": "TempPassword_12Chars",
  "type": "generateRandomString",
  "attributes": {
    "length": 12
  }
}
```

**Output:** "aBcDeFgHiJkL" (random, every execution different)

### UUID: What It Does
Generates a universally unique identifier.

### JSON Structure
```json
{
  "type": "uuidGenerator"
}
```

### Real Example: Generate Unique Reference ID

```json
{
  "name": "Employee_Reference_ID",
  "type": "uuidGenerator"
}
```

**Output:** "550e8400-e29b-41d4-a716-446655440000" (unique identifier)

### When to Use Each

**GenerateRandomString:**
- ✅ Temporary passwords
- ✅ Session tokens
- ✅ Random codes

**UUIDGenerator:**
- ✅ Unique identifiers
- ✅ Reference IDs
- ✅ Tracking numbers

---

## Operation #4: Lookup

### What It Does
Maps a value to another value using a predefined dictionary. Think of it as a translation table.

### JSON Structure
```json
{
  "type": "lookup",
  "attributes": {
    "input": "source_of_data",
    "map": {
      "key1": "value1",
      "key2": "value2"
    },
    "defaultValue": "fallback_if_not_found"
  }
}
```

### Real Example #1: Department Code to Name

**Scenario:** Source system has department codes ("IT", "HR", "FIN"), but ISC needs department names.

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
      "FIN": "Finance",
      "OPS": "Operations"
    },
    "defaultValue": "Other Department"
  }
}
```

**Execution:**

```
Input: departmentCode = "IT"
Lookup: Find "IT" in map
Output: "Information Technology"

Input: departmentCode = "UNKNOWN"
Lookup: "UNKNOWN" not in map
Output: "Other Department" (default)
```

### Real Example #2: Status Code to Access Level

```json
{
  "name": "AccessLevel_By_Status",
  "type": "lookup",
  "attributes": {
    "input": {
      "type": "identityAttribute",
      "attributes": { "name": "employmentStatus" }
    },
    "map": {
      "E": "EMPLOYEE",
      "C": "CONTRACTOR",
      "T": "TEMP",
      "V": "VENDOR"
    },
    "defaultValue": "RESTRICTED"
  }
}
```

**Execution:**

```
Input: employmentStatus = "E"
Output: "EMPLOYEE"

Input: employmentStatus = "C"
Output: "CONTRACTOR"

Input: employmentStatus = "X"
Output: "RESTRICTED" (not in map)
```

### Important: Lookup is Case-Sensitive

```
Map: { "IT": "Information Technology" }

Input: "it" (lowercase)
Result: NOT FOUND → "defaultValue"

Input: "IT" (matches case)
Result: "Information Technology"
```

If your data might have case issues, standardize it first:

```json
{
  "type": "lookup",
  "attributes": {
    "input": {
      "type": "lower",
      "attributes": {
        "input": { "type": "identityAttribute", "attributes": { "name": "departmentCode" } }
      }
    },
    "map": {
      "it": "Information Technology",
      "hr": "Human Resources",
      "fin": "Finance"
    },
    "defaultValue": "Other Department"
  }
}
```

---

## Lesson 6 Quick Reference

| Operation | Use Case | Example |
|-----------|----------|---------|
| **Reference** | Reuse complex logic | Reference Email_Standardize |
| **UsernameGenerator** | Create unique usernames | Pattern: $firstName.$lastName |
| **GenerateRandomString** | Create random values | Generate 12-char password |
| **UUIDGenerator** | Create unique IDs | Employee reference ID |
| **Lookup** | Translate codes | Map "IT" → "Information Technology" |

---

## Building Block Exercise: Provision to Multiple Systems

**Scenario:** You're provisioning an employee to AD and ServiceNow. Both need:
1. Username (generated, unique per system)
2. Email (standardized)
3. Department (translated from code)

**Solution Using Today's Operations:**

**Create Profile for Active Directory:**

```json
{
  "name": "AD_Username_Unique",
  "type": "usernameGenerator",
  "attributes": {
    "sourceCheck": {
      "sourceName": "Active Directory",
      "attributeName": "sAMAccountName"
    },
    "pattern": "$firstName.$lastName",
    "lookup": "email"
  }
}
```

```json
{
  "name": "Email_Standardized",
  "type": "reference",
  "attributes": {
    "id": "email-standardize-base-id"
  }
}
```

```json
{
  "name": "Department_Lookup",
  "type": "lookup",
  "attributes": {
    "input": { "type": "identityAttribute", "attributes": { "name": "deptCode" } },
    "map": {
      "IT": "Information Technology",
      "HR": "Human Resources"
    },
    "defaultValue": "Other"
  }
}
```

**Create Profile for ServiceNow:**

```json
{
  "name": "SNOW_Username_Unique",
  "type": "usernameGenerator",
  "attributes": {
    "sourceCheck": {
      "sourceName": "ServiceNow",
      "attributeName": "user_name"
    },
    "pattern": "$firstInitial$lastName",
    "lookup": "email"
  }
}
```

```json
{
  "name": "Email_Standardized",
  "type": "reference",
  "attributes": {
    "id": "email-standardize-base-id"
  }
}
```

```json
{
  "name": "Department_Lookup",
  "type": "reference",
  "attributes": {
    "id": "department-lookup-id"
  }
}
```

**Result:**
- Both systems get standardized email (via Reference)
- Both systems get translated department (via Reference)
- Each system gets unique username (different patterns)
- Zero duplicated logic

---

## When NOT to Use These Operations

### Don't Use Reference When:
- ❌ Logic is only used once (just use the operations directly)
- ❌ You need variations (build separate transforms)
- ❌ The reference ID is unclear (causes confusion)

### Don't Use UsernameGenerator When:
- ❌ You need complex logic (use string operations instead)
- ❌ You need conditional patterns (use conditional + string ops)
- ❌ Pattern doesn't match system requirements

### Don't Use Lookup When:
- ❌ You have complex logic (use conditional instead)
- ❌ You have thousands of mappings (will be huge JSON)
- ❌ Values change frequently (hardcoding isn't ideal)

---

## Mastery Checkpoint #6

### Question 1: When to Use Reference?

You've created a complex email standardization transform. Now you need the same logic in 5 different places. Should you:
A) Copy the JSON to all 5 places
B) Use Reference to point to the same transform
C) Create 5 different transforms with different names

<details>
<summary>Answer</summary>

**B) Use Reference**

Copy-pasting (A) creates maintenance nightmares. Reference lets you update logic in one place.

</details>

---

### Question 2: UsernameGenerator Pattern

What will this generate for firstName="Christopher", lastName="Wellington"?

```json
{
  "type": "usernameGenerator",
  "attributes": {
    "sourceCheck": { "sourceName": "AD", "attributeName": "sAMAccountName" },
    "pattern": "$firstName.$lastName"
  }
}
```

<details>
<summary>Answer</summary>

**"christopher.wellington"** (if unique in AD)

If "christopher.wellington" already exists, it would try "christopher.wellington2", etc.

</details>

---

### Question 3: Lookup with Case Issue

What happens if the map has "IT" but the input is "it" (lowercase)?

```json
{
  "type": "lookup",
  "attributes": {
    "input": { "type": "identityAttribute", "attributes": { "name": "deptCode" } },
    "map": {
      "IT": "Information Technology",
      "HR": "Human Resources"
    },
    "defaultValue": "Other"
  }
}
```

<details>
<summary>Answer</summary>

**Returns "Other"** (the default value)

Lookup is case-sensitive. "it" ≠ "IT"

**Fix:** Lowercase the input first, then lookup:

```json
{
  "type": "lookup",
  "attributes": {
    "input": {
      "type": "lower",
      "attributes": { "input": { ... } }
    },
    "map": {
      "it": "Information Technology",
      "hr": "Human Resources"
    }
  }
}
```

</details>

---

### Question 4: Real Scenario

You need to generate a username for ServiceNow that:
- Follows pattern: firstName_lastName
- Checks for uniqueness in ServiceNow
- Has a fallback if pattern conflicts

Which operation should you use and why?

<details>
<summary>Answer</summary>

**UsernameGenerator**

- It generates based on pattern ✓
- It checks for uniqueness automatically ✓
- It auto-appends numbers if conflict exists ✓

```json
{
  "type": "usernameGenerator",
  "attributes": {
    "sourceCheck": {
      "sourceName": "ServiceNow",
      "attributeName": "user_name"
    },
    "pattern": "$firstName_$lastName",
    "lookup": "email"
  }
}
```

</details>

---

## PRACTICE EXERCISE: Multi-System Provisioning

**Build this yourself:**

You're setting up provisioning for a new employee to 3 systems:
1. Active Directory (username: firstname.lastname)
2. ServiceNow (username: first initial + lastname)
3. Salesforce (username: firstname_lastname)

All three systems need:
- Email (standardized)
- Department (translated from code)

Create the transforms needed for ServiceNow provisioning.

**Hint:** You'll need:
- UsernameGenerator for ServiceNow username
- Reference for email (assuming it exists)
- Reference for department lookup
- OR Lookup if department lookup doesn't exist yet

### My Solution

```json
{
  "name": "SNOW_Provision_Username",
  "type": "usernameGenerator",
  "attributes": {
    "sourceCheck": {
      "sourceName": "ServiceNow",
      "attributeName": "user_name"
    },
    "pattern": "$firstInitial$lastName",
    "lookup": "email"
  }
}
```

```json
{
  "name": "SNOW_Provision_Email",
  "type": "reference",
  "attributes": {
    "id": "your-email-standardize-id"
  }
}
```

```json
{
  "name": "SNOW_Provision_Department",
  "type": "lookup",
  "attributes": {
    "input": {
      "type": "lower",
      "attributes": {
        "input": { "type": "identityAttribute", "attributes": { "name": "deptCode" } }
      }
    },
    "map": {
      "it": "Information Technology",
      "hr": "Human Resources",
      "sales": "Sales",
      "ops": "Operations"
    },
    "defaultValue": "Other"
  }
}
```

**What This Does:**
- Username is generated unique to ServiceNow
- Email is standardized (reused from Reference)
- Department code is translated to name

Three transforms, all needed for ServiceNow provisioning.

---

## Summary: Lesson 6

| Operation | Purpose | Frequency |
|-----------|---------|-----------|
| **Reference** | Reuse transforms | Medium (when logic repeats) |
| **UsernameGenerator** | Create unique usernames | High (almost every AD/system) |
| **GenerateRandomString** | Create random values | Low (special cases) |
| **UUIDGenerator** | Create unique IDs | Low (special cases) |
| **Lookup** | Translate codes | Medium (common for data mapping) |

---

## What's Next

In **Lesson 7**, you'll learn:
- ✅ Advanced nesting patterns
- ✅ When nesting is readable vs unreadable
- ✅ Breaking down complex transforms
- ✅ Real production patterns

This is the "mastery" lesson—where everything comes together.

---

## Homework Before Lesson 7

1. Review Exercise 8 and 10 from Practice Module (they use some of today's concepts)
2. Identify 3 transforms in your environment that could use Reference operations
3. Think about: What patterns would you need for username generation in your systems?

**Next: Advanced Patterns and Mastery** — where you become a true expert.
