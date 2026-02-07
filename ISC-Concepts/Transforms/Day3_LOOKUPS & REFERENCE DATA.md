# DAY 3: LOOKUPS & REFERENCE DATA - MORNING SESSION

## Table of Contents
- [Overview](#overview)
- [What You'll Learn Today](#what-youll-learn-today)
- [Understanding Lookup Transforms](#understanding-lookup-transforms)
- [Transform 1: Lookup - Basic Usage](#transform-1-lookup---basic-usage-30-minutes)
- [Transform 2: Lookup - Advanced Patterns](#transform-2-lookup---advanced-patterns-30-minutes)
- [Transform 3: Account Attribute](#transform-3-account-attribute-45-minutes)
- [Transform 4: Identity Attribute](#transform-4-identity-attribute-15-minutes)
- [Morning Session Review](#morning-session-review)
- [Checkpoint Quiz](#checkpoint-quiz)

---

## Overview

**Session Duration:** 2 hours

**Prerequisites:** 
- Completed Day 1 (String transforms)
- Completed Day 2 (Conditional logic)
- Understand transform chaining
- Comfortable with FirstValid

**Goal:** Master data lookup and cross-referencing transforms

---

## What You'll Learn Today

By the end of this morning session, you'll be able to:

- ✅ Create and use lookup tables for value mapping
- ✅ Build hierarchical lookups for complex mappings
- ✅ Use Account Attribute to get data from user's accounts
- ✅ Use Identity Attribute to reference other identity attributes
- ✅ Combine lookup transforms with conditionals
- ✅ Handle lookup misses and defaults gracefully

---

## Understanding Lookup Transforms

### What Are Lookup Transforms?

**Lookup transforms** map input values to output values using a predefined table.

Think of it as a **dictionary** or **translation table**:
```
Input Value --> Lookup Table --> Output Value

"ENG" --> [Lookup Table] --> "Engineering"
"FIN" --> [Lookup Table] --> "Finance"
"HR" --> [Lookup Table] --> "Human Resources"
```

---

### When to Use Lookup vs Conditional

| Scenario | Use Lookup | Use Conditional |
|----------|-----------|-----------------|
| 2-5 simple mappings | Maybe | Yes |
| 10+ mappings | Yes | No (too many nested conditionals) |
| Static value mappings | Yes | No |
| Logic-based decisions | No | Yes |
| Frequently updated mappings | Yes (easier to update) | No |
| Need default for unknowns | Yes (has default field) | Need else clause |

**Example:**
```
Good for Lookup:
  Map 50 department codes to department names
  ENG --> Engineering
  FIN --> Finance
  ... (50 more)

Good for Conditional:
  If hire_date > today then "Pre-hire" else "Active"
  (This is logic, not static mapping)
```

---

### Lookup Table Structure

A lookup table has three components:
```
1. INPUT COLUMN: The value to search for
2. OUTPUT COLUMN: The value to return
3. DEFAULT VALUE: What to return if input not found (optional)
```

**Example Table: Department Codes**

| Input (Code) | Output (Full Name) |
|--------------|-------------------|
| ENG | Engineering |
| FIN | Finance |
| HR | Human Resources |
| IT | Information Technology |
| OPS | Operations |
| **Default** | **Other** |

---

## Transform 1: Lookup - Basic Usage (30 minutes)

### Exercise 1A: Department Code to Full Name

**Scenario:** Your HR system uses 3-letter codes, but you need full department names in ISC.

#### Step 1: Create the Lookup Table (10 min)

#### UI Style: Steps
```
1. In SailPoint ISC, navigate to:
   Admin > Identities > Lookup Tables
   
   (If you don't see this, lookup tables may be created inline with transforms)

2. Click "Create Lookup Table" (or this may be done within the transform)

3. Configure:
   Name: DepartmentCodeMapping
   Description: Maps department codes to full names

4. Add entries:

   Input | Output
   ------|--------
   ENG   | Engineering
   FIN   | Finance
   HR    | Human Resources
   IT    | Information Technology
   OPS   | Operations
   MKT   | Marketing
   SALES | Sales
   CS    | Customer Success
   PROD  | Product Management
   LEGAL | Legal

5. Set Default Value: "Other"

6. Save the lookup table
```

> **Note:** UI may vary. Some ISC versions create lookup tables directly in the transform configuration rather than as separate objects.

---

#### Step 2: Use the Lookup in a Transform (10 min)

#### UI Style: Steps
```
1. Go to Identity Profile > Mappings
2. Create new attribute "departmentFullName"
3. Source: Transform
4. Transform: Lookup
5. Configuration:

   Input Attribute: department (or dept_code, whatever your source field is)
   Lookup Table: DepartmentCodeMapping (select from dropdown)
   Default Value: "Other" (if not set in table)

6. Preview with different identities
7. Save
```

#### JSON Style: Transform Definition
```json
{
  "name": "departmentFullName",
  "type": "lookup",
  "attributes": {
    "input": {
      "type": "identityAttribute",
      "attributes": {
        "name": "department"
      }
    },
    "table": {
      "id": "department-code-mapping"
    },
    "default": "Other"
  }
}
```

---

#### Test Cases (10 min)
```
Test Case 1: Exact match
  Input: department = "ENG"
  Output: "Engineering"

Test Case 2: Exact match (different code)
  Input: department = "FIN"
  Output: "Finance"

Test Case 3: Not in table
  Input: department = "UNKNOWN"
  Output: "Other" (default value)

Test Case 4: Null input
  Input: department = null
  Output: "Other" (default value)

Test Case 5: Case sensitivity
  Input: department = "eng" (lowercase)
  Output: "Other" (lookup is case-sensitive!)
```

---

#### Important Note: Case Sensitivity

**Lookups are case-sensitive!**
```
"ENG" matches "ENG" ✓
"eng" does NOT match "ENG" ✗
"Eng" does NOT match "ENG" ✗
```

**Solution:** Normalize the input before lookup:
```
Transform chain:
1. Upper (convert input to uppercase)
2. Lookup (now "eng" becomes "ENG" and matches)
```

**Let's fix this:**

#### UI Style: Steps
```
1. Edit the departmentFullName mapping
2. Add transform BEFORE lookup:

   Transform 1: Upper
     Input: department
     Output: "ENG" (from "eng")
   
   Transform 2: Lookup
     Input: Output from Transform 1
     Table: DepartmentCodeMapping
     Output: "Engineering"

3. Save
```

#### JSON Style: Transform Definition (with normalization)
```json
{
  "name": "departmentFullName",
  "type": "upper",
  "attributes": {
    "input": {
      "type": "identityAttribute",
      "attributes": {
        "name": "department"
      }
    },
    "next": {
      "type": "lookup",
      "attributes": {
        "input": {
          "type": "reference"
        },
        "table": {
          "id": "department-code-mapping"
        },
        "default": "Other"
      }
    }
  }
}
```

Now test again:
```
Input: department = "eng"
After Upper: "ENG"
After Lookup: "Engineering" ✓
```

---

### Exercise 1B: Title Normalization

**Scenario:** Your organization has 50+ job title variations that need to be standardized to 10 standard titles.

#### UI Style: Create Comprehensive Lookup Table
```
1. Create lookup table: TitleStandardization
2. Add entries (sample - you'd have 50+):

Input Title                    | Output Standard Title
-------------------------------|----------------------
Software Engineer              | Engineer
Sr. Software Engineer          | Senior Engineer
Senior Software Engineer       | Senior Engineer
Staff Software Engineer        | Senior Engineer
Principal Engineer             | Principal Engineer
Lead Engineer                  | Principal Engineer
Engineering Manager            | Manager
Manager, Engineering           | Manager
Director of Engineering        | Director
VP Engineering                 | Vice President
Vice President Engineering     | Vice President
Developer                      | Engineer
Sr. Developer                  | Senior Engineer
Analyst                        | Analyst
Senior Analyst                 | Senior Analyst
Consultant                     | Consultant
Senior Consultant              | Senior Consultant

... (continue for all variations)

Default: Individual Contributor

3. Save
```

---

#### UI Style: Use in Transform
```
1. Create attribute "standardTitle"
2. Transform: Lookup
   Input: title
   Table: TitleStandardization
   Default: "Individual Contributor"
3. Save
```

#### JSON Style: Transform Definition
```json
{
  "name": "standardTitle",
  "type": "lookup",
  "attributes": {
    "input": {
      "type": "identityAttribute",
      "attributes": {
        "name": "title"
      }
    },
    "table": {
      "id": "title-standardization"
    },
    "default": "Individual Contributor"
  }
}
```

---

#### Test Cases
```
Test 1:
  Input: "Software Engineer"
  Output: "Engineer"

Test 2:
  Input: "Sr. Software Engineer"
  Output: "Senior Engineer"

Test 3:
  Input: "Some Weird Title Not In Table"
  Output: "Individual Contributor" (default)
```

---

### Key Learning Points

- ✅ Lookup tables map input to output
- ✅ Case-sensitive by default - use Upper/Lower to normalize
- ✅ Default value handles unknowns
- ✅ Great for standardization (many variations to few standards)
- ✅ More maintainable than 50+ nested conditionals
- ✅ Can be updated without changing transform logic

---

## Transform 2: Lookup - Advanced Patterns (30 minutes)

### Exercise 2A: Multi-Column Mapping with Helpers

**Scenario:** You need multiple attributes from a location code.
```
Location Code --> City, State, Country, Timezone

Example:
  "ATX" --> "Austin", "TX", "USA", "America/Chicago"
  "LON" --> "London", null, "UK", "Europe/London"
```

**Problem:** Lookup returns ONE value, but we need FOUR!

**Solution:** Create FOUR separate lookups!

---

#### Create Four Lookup Tables

**Table 1: LocationToCity**
```
Input | Output
------|--------
ATX   | Austin
DFW   | Dallas
NYC   | New York
LON   | London
TYO   | Tokyo
```

**Table 2: LocationToState**
```
Input | Output
------|--------
ATX   | TX
DFW   | TX
NYC   | NY
LON   | null (or leave empty - UK doesn't have states)
TYO   | null
```

**Table 3: LocationToCountry**
```
Input | Output
------|--------
ATX   | USA
DFW   | USA
NYC   | USA
LON   | United Kingdom
TYO   | Japan
```

**Table 4: LocationToTimezone**
```
Input | Output
------|--------
ATX   | America/Chicago
DFW   | America/Chicago
NYC   | America/New_York
LON   | Europe/London
TYO   | Asia/Tokyo
```

---

#### Create Four Attributes with Lookups

#### UI Style: Steps
```
1. Attribute: city
   Transform: Lookup
   Input: location_code
   Table: LocationToCity
   Default: "Unknown"

2. Attribute: state
   Transform: Lookup
   Input: location_code
   Table: LocationToState
   Default: null

3. Attribute: country
   Transform: Lookup
   Input: location_code
   Table: LocationToCountry
   Default: "Unknown"

4. Attribute: timezone
   Transform: Lookup
   Input: location_code
   Table: LocationToTimezone
   Default: "UTC"
```

#### JSON Style: Transform Definitions

**Attribute 1: city**
```json
{
  "name": "city",
  "type": "lookup",
  "attributes": {
    "input": {
      "type": "identityAttribute",
      "attributes": {
        "name": "location_code"
      }
    },
    "table": {
      "id": "location-to-city"
    },
    "default": "Unknown"
  }
}
```

**Attribute 2: state**
```json
{
  "name": "state",
  "type": "lookup",
  "attributes": {
    "input": {
      "type": "identityAttribute",
      "attributes": {
        "name": "location_code"
      }
    },
    "table": {
      "id": "location-to-state"
    },
    "default": null
  }
}
```

**Attribute 3: country**
```json
{
  "name": "country",
  "type": "lookup",
  "attributes": {
    "input": {
      "type": "identityAttribute",
      "attributes": {
        "name": "location_code"
      }
    },
    "table": {
      "id": "location-to-country"
    },
    "default": "Unknown"
  }
}
```

**Attribute 4: timezone**
```json
{
  "name": "timezone",
  "type": "lookup",
  "attributes": {
    "input": {
      "type": "identityAttribute",
      "attributes": {
        "name": "location_code"
      }
    },
    "table": {
      "id": "location-to-timezone"
    },
    "default": "UTC"
  }
}
```

---

#### Test Results
```
Input: location_code = "ATX"

Outputs:
  city = "Austin"
  state = "TX"
  country = "USA"
  timezone = "America/Chicago"
```

---

### Exercise 2B: Hierarchical Lookup (Chained Lookups)

**Scenario:** Map department to division to executive.
```
Department --> Division --> Executive

Engineering --> Technology --> CTO
IT --> Technology --> CTO
Finance --> Corporate --> CFO
HR --> Corporate --> CFO
Sales --> Revenue --> CRO
Marketing --> Revenue --> CRO
```

This requires **chaining two lookups**!

---

#### Create Two Lookup Tables

**Table 1: DepartmentToDivision**
```
Input          | Output
---------------|----------
Engineering    | Technology
IT             | Technology
Security       | Technology
Finance        | Corporate
HR             | Corporate
Legal          | Corporate
Sales          | Revenue
Marketing      | Revenue
Default        | Other
```

**Table 2: DivisionToExecutive**
```
Input      | Output
-----------|-------
Technology | CTO
Corporate  | CFO
Revenue    | CRO
Other      | CEO
```

---

#### Chain the Lookups

#### UI Style: Steps
```
1. Create intermediate attribute: division
   Transform: Lookup
   Input: department
   Table: DepartmentToDivision
   Default: "Other"

2. Create final attribute: executiveSponsor
   Transform: Lookup
   Input: division (the attribute we just created!)
   Table: DivisionToExecutive
   Default: "CEO"

3. Save both
```

#### JSON Style: Transform Definitions

**Attribute 1: division**
```json
{
  "name": "division",
  "type": "lookup",
  "attributes": {
    "input": {
      "type": "identityAttribute",
      "attributes": {
        "name": "department"
      }
    },
    "table": {
      "id": "department-to-division"
    },
    "default": "Other"
  }
}
```

**Attribute 2: executiveSponsor**
```json
{
  "name": "executiveSponsor",
  "type": "lookup",
  "attributes": {
    "input": {
      "type": "identityAttribute",
      "attributes": {
        "name": "division"
      }
    },
    "table": {
      "id": "division-to-executive"
    },
    "default": "CEO"
  }
}
```

---

#### How It Works
```
Input: department = "Engineering"

Step 1: Lookup DepartmentToDivision
  "Engineering" --> "Technology"
  (division attribute = "Technology")

Step 2: Lookup DivisionToExecutive
  "Technology" --> "CTO"
  (executiveSponsor = "CTO")

Final Result: CTO
```

---

#### Test Cases
```
Test 1:
  department = "Engineering"
  division = "Technology"
  executiveSponsor = "CTO"

Test 2:
  department = "Finance"
  division = "Corporate"
  executiveSponsor = "CFO"

Test 3:
  department = "UnknownDept"
  division = "Other" (default from first lookup)
  executiveSponsor = "CEO" (from second lookup)
```

---

### Key Learning Points

- ✅ Multiple lookups can extract different attributes from same input
- ✅ Lookups can be chained (output of one feeds into next)
- ✅ Create intermediate attributes for clarity
- ✅ Hierarchical mappings possible (dept → division → exec)
- ✅ Default values cascade through chains

---

## Transform 3: Account Attribute (45 minutes)

### Understanding Account Attribute Transform

**Account Attribute** lets you get data from a **user's account** on a specific source.

#### Mental Model
```
Identity: John Smith
├─ Authoritative Account: Workday
│  └─ Attributes: emp_id, hire_date, department
├─ Account: Active Directory
│  └─ Attributes: samAccountName, mail, memberOf
└─ Account: Salesforce
   └─ Attributes: username, role, territory

Account Attribute transform can:
- Get John's email from his AD account
- Get John's territory from his Salesforce account
- Get John's manager ID from his Workday account
```

---

### How Account Attribute Works

#### Prerequisites

For Account Attribute to work, the identity must have:

1. **An account on the specified source**
2. **Account must be correlated to the identity**
3. **Attribute must exist on that account**

If any of these fail, the transform returns **null**.

---

### Exercise 3A: Get Email from Active Directory

**Scenario:** Get user's email address from their AD account.

#### UI Style: Steps
```
1. Go to Identity Profile > Mappings
2. Create/edit attribute "emailFromAD"
3. Source: Transform
4. Transform: Account Attribute
5. Configuration:

   Source: Active Directory (select from dropdown)
   Attribute Name: mail (or email, depending on your AD schema)

6. Preview with identities that have AD accounts
7. Save
```

#### JSON Style: Transform Definition
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

---

#### What Happens Under the Hood
```
For identity "John Smith":

1. SailPoint finds John's identity
2. Looks for accounts correlated to John
3. Finds account on source "Active Directory"
4. Reads the "mail" attribute from that account
5. Returns the value: "john.smith@company.com"

If John doesn't have AD account:
  Returns: null
```

---

#### Test Cases
```
Test Case 1: User has AD account with email
  Identity: John Smith
  AD Account exists: YES
  AD Account mail attribute: "john.smith@company.com"
  Output: "john.smith@company.com"

Test Case 2: User has AD account, but no email
  Identity: Jane Doe
  AD Account exists: YES
  AD Account mail attribute: null
  Output: null

Test Case 3: User has no AD account
  Identity: New Hire
  AD Account exists: NO
  Output: null
```

---

### Exercise 3B: Get Manager Email from AD

**Scenario:** Get the user's manager's email from Active Directory.

**Challenge:** You need the manager's email, not the user's!

#### The Approach
```
1. Get manager ID from user's account
2. Find manager's identity
3. Get manager's AD account
4. Extract email from manager's AD account
```

This is **complex** - Account Attribute alone won't do it!

---

#### Solution: Use Manager Correlation

SailPoint has **manager correlation** built-in. Once configured, each identity has a `manager` attribute that references their manager's identity.

Then you can use Account Attribute on the **manager's** account:

#### UI Style: Steps
```
1. Create attribute "managerEmailFromAD"
2. Transform: Account Attribute
3. Configuration:

   Source: Active Directory
   Attribute: mail
   Identity Context: Manager (if available in UI)

4. Save
```

> **Note:** The exact UI for getting attributes from a manager's account varies by ISC version. Some versions require using the `manager` identity attribute first, then getting their account attribute.

#### JSON Style: Transform Definition
```json
{
  "name": "managerEmailFromAD",
  "type": "accountAttribute",
  "attributes": {
    "sourceName": "Active Directory",
    "attributeName": "mail",
    "accountSortAttribute": "created",
    "accountSortDescending": false,
    "accountReturnFirstLink": false,
    "accountFilter": null
  }
}
```

> **Note:** Getting manager's account attribute may require additional configuration or a different approach depending on ISC version.

---

#### Alternative: Two-Step Approach

If direct manager account attribute isn't available:

**Step 1:** Get manager's identity attribute
```
Attribute: managerIdentity
Transform: Identity Attribute
Input: manager (references the manager identity object)
```

**Step 2:** Get manager's AD email

This may require a more complex approach or using the Reference transform.

For now, understand the **concept**: Account Attribute can traverse to related identities' accounts.

---

### Exercise 3C: Get Groups from AD (Multi-Value)

**Scenario:** Get user's AD group memberships.

#### Challenge: Multi-Value Attributes

AD `memberOf` attribute is **multi-value** (array of groups):
```
memberOf: [
  "CN=Admins,OU=Groups,DC=company,DC=com",
  "CN=Developers,OU=Groups,DC=company,DC=com",
  "CN=Engineering,OU=Groups,DC=company,DC=com"
]
```

Account Attribute returns the **entire array**.

---

#### Get First Group

#### UI Style: Steps
```
1. Create attribute "adGroups"
2. Transform chain:

   Transform 1: Account Attribute
     Source: Active Directory
     Attribute: memberOf
     Output: ["CN=Admins...", "CN=Developers...", ...]
   
   Transform 2: Index (get first group)
     Position: 0
     Output: "CN=Admins,OU=Groups,DC=company,DC=com"

3. Save
```

#### JSON Style: Transform Definition
```json
{
  "name": "adGroups",
  "type": "accountAttribute",
  "attributes": {
    "sourceName": "Active Directory",
    "attributeName": "memberOf",
    "next": {
      "type": "index",
      "attributes": {
        "index": 0
      }
    }
  }
}
```

---

#### Extract Just Group Name

Continue the chain:

#### UI Style: Steps
```
   Transform 3: Split
     Delimiter: ","
     Output: ["CN=Admins", "OU=Groups", "DC=company", ...]
   
   Transform 4: Index
     Position: 0
     Output: "CN=Admins"
   
   Transform 5: Split
     Delimiter: "="
     Output: ["CN", "Admins"]
   
   Transform 6: Index
     Position: 1
     Output: "Admins"
```

Full chain gets first AD group name: "Admins"

#### JSON Style: Complete Transform Chain
```json
{
  "name": "primaryGroupName",
  "type": "accountAttribute",
  "attributes": {
    "sourceName": "Active Directory",
    "attributeName": "memberOf",
    "next": {
      "type": "index",
      "attributes": {
        "index": 0,
        "next": {
          "type": "split",
          "attributes": {
            "delimiter": ",",
            "next": {
              "type": "index",
              "attributes": {
                "index": 0,
                "next": {
                  "type": "split",
                  "attributes": {
                    "delimiter": "=",
                    "next": {
                      "type": "index",
                      "attributes": {
                        "index": 1
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

---

#### Get All Groups (Joined)

#### UI Style: Steps
```
Transform chain:

1. Account Attribute
   Source: AD
   Attribute: memberOf
   Output: ["CN=Admins...", "CN=Developers...", ...]

2. Join
   Delimiter: "; "
   Output: "CN=Admins...; CN=Developers...; ..."
```

This creates a single string with all groups.

#### JSON Style: Transform Definition
```json
{
  "name": "adGroupsJoined",
  "type": "accountAttribute",
  "attributes": {
    "sourceName": "Active Directory",
    "attributeName": "memberOf",
    "next": {
      "type": "join",
      "attributes": {
        "delimiter": "; "
      }
    }
  }
}
```

---

### Key Learning Points

- ✅ Account Attribute gets data from user's accounts on specific sources
- ✅ Requires account to exist and be correlated
- ✅ Returns null if account or attribute missing
- ✅ Can get manager's account attributes (with manager correlation)
- ✅ Handles multi-value attributes (returns array)
- ✅ Combine with Index, Split, Join for parsing

---

## Transform 4: Identity Attribute (15 minutes)

### Understanding Identity Attribute Transform

**Identity Attribute** references **other attributes on the same identity**.

#### When to Use
```
You need to:
- Reference a calculated attribute in another transform
- Copy one attribute to another
- Use an identity attribute in a complex calculation
```

---

### Exercise 4A: Copy Attribute

**Scenario:** Copy displayName to a backup attribute.

#### UI Style: Steps
```
1. Create attribute "displayNameBackup"
2. Transform: Identity Attribute
3. Configuration:

   Attribute: displayName

4. Save
```

#### JSON Style: Transform Definition
```json
{
  "name": "displayNameBackup",
  "type": "identityAttribute",
  "attributes": {
    "name": "displayName"
  }
}
```

Result: `displayNameBackup` = whatever `displayName` is.

---

### Exercise 4B: Reference Calculated Attribute

**Scenario:** Use a previously calculated attribute in a new calculation.

#### Example

#### UI Style: Steps
```
You have:
  - Attribute "standardDepartment" (calculated via Lookup)
  - Want to create "departmentCode" from it

Transform chain:
1. Identity Attribute
   Attribute: standardDepartment
   Output: "Engineering"

2. Lookup
   Input: Output from step 1
   Table: DepartmentNameToCode
   Output: "ENG"
```

#### JSON Style: Transform Definition
```json
{
  "name": "departmentCode",
  "type": "identityAttribute",
  "attributes": {
    "name": "standardDepartment",
    "next": {
      "type": "lookup",
      "attributes": {
        "input": {
          "type": "reference"
        },
        "table": {
          "id": "department-name-to-code"
        },
        "default": "UNK"
      }
    }
  }
}
```

---

### Exercise 4C: Conditional Based on Identity Attribute

**Scenario:** Make decisions based on other calculated attributes.

#### UI Style: Steps
```
Attribute: accessLevel
Transform: Conditional

Condition: Identity Attribute "standardTitle" equals "Manager"
If True: "Manager Access"
If False: "Standard Access"
```

#### JSON Style: Transform Definition
```json
{
  "name": "accessLevel",
  "type": "conditional",
  "attributes": {
    "expression": "standardTitle eq \"Manager\"",
    "positiveCondition": "Manager Access",
    "negativeCondition": "Standard Access"
  }
}
```

---

### Key Learning Points

- ✅ Identity Attribute references other attributes on same identity
- ✅ Useful for multi-step calculations
- ✅ Can reference any attribute (source or calculated)
- ✅ Allows complex transform chains to build on each other
- ✅ Creates dependencies (attribute A uses attribute B)

---

## Morning Session Review

### Transforms Learned

| Transform | Purpose | Key Use Cases |
|-----------|---------|---------------|
| **Lookup** | Map input values to output values | Code translation, standardization |
| **Account Attribute** | Get data from user's accounts | Email from AD, groups, cross-source data |
| **Identity Attribute** | Reference other identity attributes | Multi-step calculations, copying |

---

### What You Built

- [ ] Department code to full name lookup
- [ ] Title standardization lookup (50+ variations)
- [ ] Multi-column location mapping (4 lookups)
- [ ] Hierarchical lookup (dept → division → exec)
- [ ] Email from AD account
- [ ] Manager email from AD (concept)
- [ ] AD groups extraction and parsing
- [ ] Identity attribute references

**Total: 8+ lookup and reference patterns!**

---

### Key Concepts Mastered

- ✅ Creating and using lookup tables
- ✅ Case sensitivity in lookups (and how to fix)
- ✅ Default values in lookups
- ✅ Multiple lookups for multi-column data
- ✅ Chaining lookups (hierarchical mapping)
- ✅ Account Attribute prerequisites
- ✅ Getting data from user's accounts
- ✅ Handling multi-value account attributes
- ✅ Identity Attribute for cross-referencing
- ✅ Both UI and JSON configuration

---

## Checkpoint Quiz

### Question 1: When to Use Lookup?

You need to map 30 department codes to full names. Should you use:
A) 30 nested Conditional transforms
B) Lookup table
C) Static transform with conditionals
D) Replace transforms

<details>
<summary>Click to reveal answer</summary>

**Answer: B) Lookup table**

**Why:**
- 30+ nested conditionals is unmaintainable
- Lookup table is clean, simple, and easy to update
- Can add/change mappings without modifying transform
- Better performance than 30 conditionals

**JSON Example:**
```json
{
  "name": "departmentFullName",
  "type": "lookup",
  "attributes": {
    "input": {
      "type": "identityAttribute",
      "attributes": {
        "name": "department"
      }
    },
    "table": {
      "id": "department-code-mapping"
    },
    "default": "Other"
  }
}
```

</details>

---

### Question 2: Lookup Case Sensitivity

Your lookup table has:
```
ENG --> Engineering
FIN --> Finance
```

Input: `department = "eng"` (lowercase)

What is the output?

<details>
<summary>Click to reveal answer</summary>

**Answer:** Default value (or null if no default)

**Why:** Lookups are case-sensitive. "eng" does NOT match "ENG".

**Solution:** Add Upper transform before Lookup:

**JSON Style:**
```json
{
  "name": "departmentFullName",
  "type": "upper",
  "attributes": {
    "input": {
      "type": "identityAttribute",
      "attributes": {
        "name": "department"
      }
    },
    "next": {
      "type": "lookup",
      "attributes": {
        "input": {
          "type": "reference"
        },
        "table": {
          "id": "department-code-mapping"
        },
        "default": "Other"
      }
    }
  }
}
```

</details>

---

### Question 3: Account Attribute Returns Null

You're using Account Attribute to get email from AD, but it returns null. What could be wrong? List 3 possible reasons.

<details>
<summary>Click to reveal answer</summary>

**Possible reasons:**

1. **User has no AD account**
   - Account doesn't exist in AD
   - Account hasn't been aggregated yet

2. **Account not correlated**
   - AD account exists but isn't linked to the identity
   - Correlation rules not configured correctly

3. **Attribute doesn't exist or is null**
   - AD account has no `mail` attribute
   - `mail` attribute is null/empty in AD

4. **Wrong attribute name**
   - You specified "email" but AD uses "mail"
   - Attribute name is case-sensitive

5. **Wrong source selected**
   - Selected wrong source in transform config
   - Multiple AD sources and picked wrong one

**How to debug:**
- Check identity's Accounts tab - does AD account show?
- Check AD account attributes - does mail attribute exist?
- Verify attribute name in source schema

</details>

---

### Question 4: Chained Lookups

Build a two-step lookup:
```
Office Code --> City --> Timezone

Office codes:
  ATX --> Austin
  NYC --> New York

Cities:
  Austin --> America/Chicago
  New York --> America/New_York
```

<details>
<summary>Click to reveal answer</summary>

**Solution:**

**Lookup Table 1: OfficeToCity**
```
ATX --> Austin
NYC --> New York
```

**Lookup Table 2: CityToTimezone**
```
Austin --> America/Chicago
New York --> America/New_York
```

**JSON Style - Step 1:**
```json
{
  "name": "city",
  "type": "lookup",
  "attributes": {
    "input": {
      "type": "identityAttribute",
      "attributes": {
        "name": "office_code"
      }
    },
    "table": {
      "id": "office-to-city"
    }
  }
}
```

**JSON Style - Step 2:**
```json
{
  "name": "timezone",
  "type": "lookup",
  "attributes": {
    "input": {
      "type": "identityAttribute",
      "attributes": {
        "name": "city"
      }
    },
    "table": {
      "id": "city-to-timezone"
    }
  }
}
```

**Result:**
```
Input: office_code = "ATX"
Step 1: ATX --> Austin (city = "Austin")
Step 2: Austin --> America/Chicago (timezone = "America/Chicago")
```

</details>

---

### Question 5: Multi-Value Account Attribute

You get AD groups with Account Attribute, and it returns:
```
["CN=Admins,OU=Groups", "CN=Developers,OU=Groups", "CN=Engineering,OU=Groups"]
```

How do you:
A) Get just the first group?
B) Get just the group names (without CN= and OU=)?
C) Create a comma-separated string of all groups?

<details>
<summary>Click to reveal answer</summary>

**A) Get first group:**

**JSON Style:**
```json
{
  "name": "firstGroup",
  "type": "accountAttribute",
  "attributes": {
    "sourceName": "Active Directory",
    "attributeName": "memberOf",
    "next": {
      "type": "index",
      "attributes": {
        "index": 0
      }
    }
  }
}
```
Output: "CN=Admins,OU=Groups"

**B) Get group name from first group:**

**JSON Style:**
```json
{
  "name": "firstGroupName",
  "type": "accountAttribute",
  "attributes": {
    "sourceName": "Active Directory",
    "attributeName": "memberOf",
    "next": {
      "type": "index",
      "attributes": {
        "index": 0,
        "next": {
          "type": "split",
          "attributes": {
            "delimiter": ",",
            "next": {
              "type": "index",
              "attributes": {
                "index": 0,
                "next": {
                  "type": "split",
                  "attributes": {
                    "delimiter": "=",
                    "next": {
                      "type": "index",
                      "attributes": {
                        "index": 1
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
Output: "Admins"

**C) Join all groups:**

**JSON Style:**
```json
{
  "name": "allGroups",
  "type": "accountAttribute",
  "attributes": {
    "sourceName": "Active Directory",
    "attributeName": "memberOf",
    "next": {
      "type": "join",
      "attributes": {
        "delimiter": ", "
      }
    }
  }
}
```
Output: "CN=Admins,OU=Groups, CN=Developers,OU=Groups, ..."

</details>

---

## Morning Session Complete!

### You've Mastered

- ✅ Lookup tables for value mapping
- ✅ Case handling in lookups
- ✅ Multi-column lookups (multiple tables)
- ✅ Hierarchical lookups (chaining)
- ✅ Account Attribute transform
- ✅ Getting data from user's accounts
- ✅ Multi-value attribute handling
- ✅ Identity Attribute references
- ✅ Both UI and JSON configuration

---

## What's Next: Afternoon Session Preview

In the afternoon, you'll learn:

### Advanced Lookup Patterns

- **Complex reference data** scenarios
- **Combining Account Attribute with conditionals**
- **Cross-source data aggregation**
- **Building lookup tables from multiple sources**

### Real-World Complete Solutions

- Department hierarchy with multiple lookups
- Cross-source identity enrichment
- Manager chain resolution
- Access level calculation from multiple sources

**You'll build sophisticated data integration patterns!**

---

## Practice Before Afternoon

Try building these:

### Challenge 1: Cost Center Lookup

Create lookup that maps:
```
Department --> Cost Center --> Budget Owner

Engineering --> CC-1000 --> John Smith
Finance --> CC-2000 --> Jane Doe
```

Build two chained lookups to get from Department to Budget Owner.

---

### Challenge 2: Multi-Source Email

Get email with this priority:
```
1. Email from AD account (Account Attribute)
2. Email from Workday account (Account Attribute)
3. Generated email (from name)
4. Default
```

Use FirstValid + Account Attribute

---

### Challenge 3: Group-Based Access Level

Get user's highest AD group:
```
AD Groups --> Access Level

If memberOf contains "Admins" --> "Admin"
If memberOf contains "Managers" --> "Manager"
Otherwise --> "User"
```

Use Account Attribute + Conditional logic

---

**Great work! Take a break, then continue to the Afternoon Session!**

---

*Part of the SailPoint ISC Transforms Mastery 7-Day Study Plan*
