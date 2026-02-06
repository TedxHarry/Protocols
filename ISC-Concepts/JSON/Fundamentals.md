# JSON MASTERY - DAY 1: FUNDAMENTALS

## Welcome to JSON!

By the end of today, you'll be able to read and understand ANY JSON file, including complex SailPoint ISC configurations.

**Today's Goal:** Transform from "What is JSON?" to "I can read this!"

---

## Table of Contents
- [Morning Session: Understanding JSON](#morning-session-understanding-json)
  - [Part 1: What is JSON?](#part-1-what-is-json-30-minutes)
  - [Part 2: Basic Syntax Rules](#part-2-basic-syntax-rules-45-minutes)
  - [Part 3: Reading Simple JSON](#part-3-reading-simple-json-45-minutes)
- [Afternoon Session: Data Types & Nesting](#afternoon-session-data-types--nesting)
  - [Part 4: JSON Data Types](#part-4-json-data-types-60-minutes)
  - [Part 5: Nested Structures](#part-5-nested-structures-60-minutes)
- [Evening Session: Real SailPoint JSON](#evening-session-real-sailpoint-json)
  - [Part 6: Reading SailPoint Transforms](#part-6-reading-sailpoint-transforms-60-minutes)
- [Day 1 Assessment](#day-1-assessment)

---

## Morning Session: Understanding JSON

### Part 1: What is JSON? (30 minutes)

#### The Simple Answer

**JSON stands for:** JavaScript Object Notation

**But what does that MEAN?**

JSON is a way to write down information in a structured format that both humans and computers can understand.

---

#### Real-World Analogy: Recipe Card

Think of JSON like a recipe card:
```
Recipe Card (Human Format):
─────────────────────────
Recipe: Chocolate Chip Cookies
Servings: 24 cookies
Prep Time: 15 minutes

Ingredients:
- Flour: 2 cups
- Sugar: 1 cup
- Butter: 1 cup
- Eggs: 2
- Chocolate chips: 2 cups
```

**Same information in JSON:**
```json
{
  "recipe": "Chocolate Chip Cookies",
  "servings": 24,
  "prepTime": "15 minutes",
  "ingredients": {
    "flour": "2 cups",
    "sugar": "1 cup",
    "butter": "1 cup",
    "eggs": 2,
    "chocolateChips": "2 cups"
  }
}
```

**See? Same information, just organized with specific rules!**

---

#### Why JSON Exists

**Problem:** How do computers exchange information?
```
Computer A: "Hey Computer B, here's a person named John who is 30 years old"
Computer B: "What format is that in? I don't understand!"
```

**Solution:** JSON - A universal format both understand
```
Computer A sends:
{
  "name": "John",
  "age": 30
}

Computer B: "Perfect! I understand JSON. Got it: name is John, age is 30!"
```

---

#### Where JSON is Used

**Everywhere!**
```
✓ Web APIs (SailPoint, Google, Facebook, etc.)
✓ Configuration files
✓ Data storage
✓ Mobile apps
✓ IoT devices
✓ Cloud services

If data needs to move between systems → JSON
```

**In SailPoint ISC specifically:**
```
✓ Transform configurations
✓ Identity Profile settings
✓ Source configurations
✓ Provisioning policies
✓ Role definitions
✓ API requests/responses

Literally EVERYTHING is JSON!
```

---

#### How JSON Looks

**The most basic JSON:**
```json
{
  "message": "Hello, World!"
}
```

Let's break this down piece by piece:
```
{           ← Opening curly brace (start of object)
  "message" ← Key (always in quotes)
  :         ← Colon (separates key from value)
  "Hello, World!" ← Value (in quotes because it's text)
}           ← Closing curly brace (end of object)
```

**That's it! You just read your first JSON!**

---

#### Key Concepts to Remember
```
1. JSON is TEXT (not binary, not compiled, just plain text)
2. JSON has RULES (specific syntax that must be followed)
3. JSON is READABLE (you can read it with your eyes)
4. JSON is STRUCTURED (organized in a specific way)
```

---

#### Quick Check: Can You Spot JSON?

**Which of these is JSON?**

**Option A:**
```
name: John
age: 30
```

**Option B:**
```json
{
  "name": "John",
  "age": 30
}
```

**Option C:**
```
<person>
  <name>John</name>
  <age>30</age>
</person>
```

<details>
<summary>Click to reveal answer</summary>

**Answer: B**

- Option A: YAML format (no quotes, no braces)
- Option B: JSON ✓ (has curly braces, quotes, colons)
- Option C: XML format (angle brackets)

</details>

---

### Part 2: Basic Syntax Rules (45 minutes)

JSON has **6 fundamental syntax rules**. Master these and you can read ANY JSON!

---

#### Rule 1: Curly Braces { } Create Objects

**An object is a collection of key-value pairs**
```json
{
  "firstName": "John",
  "lastName": "Smith"
}
```

**Visual breakdown:**
```
{ ← Start of object
  
  "firstName": "John"  ← First key-value pair
  ,                    ← Comma separates pairs
  "lastName": "Smith"  ← Second key-value pair
  
} ← End of object
```

**Think of it like a container that holds related information**

---

#### Rule 2: Square Brackets [ ] Create Arrays

**An array is a list of items**
```json
[
  "Apple",
  "Banana",
  "Orange"
]
```

**Visual breakdown:**
```
[ ← Start of array
  
  "Apple"   ← First item
  ,         ← Comma separates items
  "Banana"  ← Second item
  ,
  "Orange"  ← Third item
  
] ← End of array
```

**Think of it like a shopping list**

---

#### Rule 3: Quotes " " for Strings and Keys

**ALL keys must have double quotes:**
```json
{
  "name": "John"
}
```

**✓ CORRECT:** Double quotes around key and string value

**✗ WRONG:**
```json
{
  name: "John"        // Missing quotes on key
  'name': 'John'      // Single quotes not allowed
  "name": John        // Missing quotes on string value
}
```

**REMEMBER:** 
- Keys → Always double quotes
- String values → Always double quotes
- Number values → No quotes
- Boolean values → No quotes

---

#### Rule 4: Colon : Separates Keys from Values

**The colon is like an equals sign**
```json
{
  "key": "value"
}
```

**Think of it as:** key = value
```
"name": "John"     means     name = John
"age": 30          means     age = 30
"active": true     means     active = true
```

---

#### Rule 5: Comma , Separates Items

**Commas go BETWEEN items (not after the last one!)**
```json
{
  "item1": "value1",  ← Comma here
  "item2": "value2",  ← Comma here
  "item3": "value3"   ← NO comma (last item)
}
```

**Common mistake - trailing comma:**
```json
{
  "name": "John",
  "age": 30,  ← This comma is WRONG (it's the last item)
}
```

**✓ CORRECT:**
```json
{
  "name": "John",
  "age": 30
}
```

---

#### Rule 6: No Comments Allowed

**JSON does NOT support comments!**
```json
{
  // This is a comment ← NOT ALLOWED in JSON!
  "name": "John"
}
```

**If you need notes, put them OUTSIDE the JSON or in a separate documentation file**

---

#### The Complete Syntax Rules Summary
```
RULE 1: { } for objects (containers)
RULE 2: [ ] for arrays (lists)
RULE 3: " " for keys and strings (always double quotes)
RULE 4: : separates key from value
RULE 5: , between items (not after last)
RULE 6: No comments allowed
```

**Master these 6 rules and you can read ANY JSON in the world!**

---

#### Practice Exercise 1: Spot the Errors

**Which of these JSON snippets are VALID?**

**Example A:**
```json
{
  "name": "John",
  "age": 30
}
```

**Example B:**
```json
{
  'name': 'John',
  'age': 30
}
```

**Example C:**
```json
{
  "name": "John",
  "age": 30,
}
```

**Example D:**
```json
{
  name: "John",
  age: 30
}
```

<details>
<summary>Click to reveal answers</summary>

**Example A: ✓ VALID**
- Curly braces ✓
- Double quotes ✓
- Colon separators ✓
- Comma between items ✓
- No trailing comma ✓

**Example B: ✗ INVALID**
- Single quotes not allowed
- Should be double quotes

**Example C: ✗ INVALID**
- Trailing comma after "age": 30
- Remove the comma after last item

**Example D: ✗ INVALID**
- Keys missing quotes
- Should be "name" and "age"

</details>

---

### Part 3: Reading Simple JSON (45 minutes)

Now let's practice READING JSON structures!

---

#### Example 1: Simple Person Object
```json
{
  "firstName": "John",
  "lastName": "Smith",
  "age": 30,
  "email": "john.smith@company.com"
}
```

**How to read this:**
```
This is an OBJECT (curly braces)

It has 4 properties:
1. firstName = "John"
2. lastName = "Smith"
3. age = 30 (number, no quotes)
4. email = "john.smith@company.com"
```

**Questions to answer:**
- What is the person's last name?
- How old are they?
- What's their email?

<details>
<summary>Click to reveal answers</summary>

- Last name: "Smith"
- Age: 30
- Email: "john.smith@company.com"

</details>

---

#### Example 2: Object with Array
```json
{
  "name": "Engineering Team",
  "members": [
    "John Smith",
    "Jane Doe",
    "Bob Johnson"
  ],
  "size": 3
}
```

**How to read this:**
```
This is an OBJECT

It has 3 properties:
1. name = "Engineering Team"
2. members = ARRAY of 3 names
   - "John Smith"
   - "Jane Doe"
   - "Bob Johnson"
3. size = 3
```

**Questions:**
- What is the team name?
- How many members?
- Who is the second member?

<details>
<summary>Click to reveal answers</summary>

- Team name: "Engineering Team"
- Number of members: 3
- Second member: "Jane Doe" (arrays start at 0, so position 1)

</details>

---

#### Example 3: Nested Object
```json
{
  "person": {
    "name": "John Smith",
    "age": 30
  },
  "company": {
    "name": "Acme Corp",
    "department": "Engineering"
  }
}
```

**How to read this:**
```
This is an OBJECT

It has 2 properties, each is also an object:

1. person = OBJECT
   - name = "John Smith"
   - age = 30

2. company = OBJECT
   - name = "Acme Corp"
   - department = "Engineering"
```

**Visual structure:**
```
Main Object
├─ person
│  ├─ name: "John Smith"
│  └─ age: 30
└─ company
   ├─ name: "Acme Corp"
   └─ department: "Engineering"
```

---

#### Reading Strategy: Top to Bottom, Left to Right

**Step-by-step process:**
```
Step 1: Identify the outer structure
   { } = Object
   [ ] = Array

Step 2: Read each key-value pair
   Find the key (before :)
   Find the value (after :)

Step 3: If value is complex (object/array), go deeper
   Repeat process for nested structure

Step 4: Build mental model
   Visualize as tree or outline
```

---

#### Practice Exercise 2: Read This JSON
```json
{
  "employee": {
    "id": "E12345",
    "name": "John Smith",
    "department": "Engineering",
    "skills": [
      "Python",
      "JavaScript",
      "SQL"
    ],
    "active": true
  }
}
```

**Answer these questions:**

1. What is the employee's ID?
2. What department do they work in?
3. How many skills do they have?
4. What is their second skill?
5. Are they active?

<details>
<summary>Click to reveal answers</summary>

1. Employee ID: "E12345"
2. Department: "Engineering"
3. Number of skills: 3
4. Second skill: "JavaScript" (position 1 in array)
5. Active: true (boolean value)

</details>

---

## Afternoon Session: Data Types & Nesting

### Part 4: JSON Data Types (60 minutes)

JSON supports **6 data types**. Let's master each one!

---

#### Data Type 1: String (Text)

**Definition:** Text wrapped in double quotes

**Examples:**
```json
{
  "firstName": "John",
  "lastName": "Smith",
  "email": "john@company.com",
  "description": "This is a longer piece of text",
  "emptyString": ""
}
```

**Key points:**
- Always double quotes
- Can be empty: ""
- Can contain spaces
- Can contain special characters (with escaping)

**Special characters that need escaping:**
```json
{
  "quote": "He said \"Hello\"",
  "backslash": "C:\\Users\\John",
  "newline": "Line 1\nLine 2"
}
```

---

#### Data Type 2: Number

**Definition:** Numeric value (no quotes)

**Examples:**
```json
{
  "age": 30,
  "price": 19.99,
  "negative": -5,
  "large": 1000000,
  "decimal": 0.5
}
```

**Key points:**
- No quotes around numbers
- Can be positive or negative
- Can be integers or decimals
- No commas in large numbers (use 1000000, not 1,000,000)

**Common mistake:**
```json
{
  "age": "30"  ← This is a STRING, not a number!
}
```

**✓ Correct:**
```json
{
  "age": 30  ← This is a NUMBER
}
```

---

#### Data Type 3: Boolean

**Definition:** True or false value

**Examples:**
```json
{
  "isActive": true,
  "isDeleted": false,
  "hasAccess": true
}
```

**Key points:**
- Only two values: true or false
- All lowercase (not True or FALSE)
- No quotes

**Common mistakes:**
```json
{
  "isActive": "true",  ← WRONG (this is a string)
  "isActive": True,    ← WRONG (must be lowercase)
  "isActive": TRUE     ← WRONG (must be lowercase)
}
```

**✓ Correct:**
```json
{
  "isActive": true
}
```

---

#### Data Type 4: Null

**Definition:** Represents "no value" or "empty"

**Examples:**
```json
{
  "middleName": null,
  "terminationDate": null,
  "manager": null
}
```

**Key points:**
- Lowercase: null (not NULL or Null)
- No quotes
- Different from empty string ""

**Null vs Empty String:**
```json
{
  "middleName": null,      ← No value at all
  "middleName": ""         ← Empty string (has a value, but it's empty)
}
```

---

#### Data Type 5: Object

**Definition:** Collection of key-value pairs in curly braces { }

**Examples:**
```json
{
  "person": {
    "name": "John",
    "age": 30
  },
  "address": {
    "street": "123 Main St",
    "city": "Austin",
    "state": "TX"
  }
}
```

**Visual structure:**
```
Main Object
├─ person (Object)
│  ├─ name: "John"
│  └─ age: 30
└─ address (Object)
   ├─ street: "123 Main St"
   ├─ city: "Austin"
   └─ state: "TX"
```

---

#### Data Type 6: Array

**Definition:** Ordered list of values in square brackets [ ]

**Examples:**
```json
{
  "fruits": ["Apple", "Banana", "Orange"],
  "numbers": [1, 2, 3, 4, 5],
  "mixed": ["Text", 123, true, null],
  "empty": []
}
```

**Key points:**
- Items in order (first, second, third, etc.)
- Can contain any data type
- Can contain mixed types
- Can be empty: []

**Array positions (indexes):**
```json
["Apple", "Banana", "Orange"]
   0        1         2

Position 0 = "Apple"
Position 1 = "Banana"
Position 2 = "Orange"
```

**Arrays start at 0, not 1!**

---

#### Data Types Summary Table

| Type | Example | Has Quotes? | Notes |
|------|---------|-------------|-------|
| String | `"John"` | Yes | Text in double quotes |
| Number | `30` | No | Integer or decimal |
| Boolean | `true` | No | true or false only |
| Null | `null` | No | Represents no value |
| Object | `{"key": "value"}` | No (but keys have quotes) | Collection of pairs |
| Array | `[1, 2, 3]` | No (but strings inside have quotes) | Ordered list |

---

#### Practice Exercise 3: Identify Data Types

**For each value, identify its data type:**
```json
{
  "name": "John Smith",
  "age": 30,
  "isActive": true,
  "salary": 75000.50,
  "middleName": null,
  "skills": ["Python", "Java"],
  "address": {
    "city": "Austin"
  },
  "retired": false
}
```

<details>
<summary>Click to reveal answers</summary>
```
"name": "John Smith"     → String
"age": 30                → Number
"isActive": true         → Boolean
"salary": 75000.50       → Number
"middleName": null       → Null
"skills": ["Python"...]  → Array
"address": {...}         → Object
"retired": false         → Boolean
```

</details>

---

### Part 5: Nested Structures (60 minutes)

Real-world JSON is often **nested** (objects within objects, arrays within objects, etc.)

Let's master reading nested structures!

---

#### Example 1: Object Inside Object
```json
{
  "employee": {
    "personalInfo": {
      "firstName": "John",
      "lastName": "Smith",
      "age": 30
    },
    "workInfo": {
      "department": "Engineering",
      "title": "Software Engineer",
      "startDate": "2020-01-15"
    }
  }
}
```

**Visual tree structure:**
```
Root Object
└─ employee (Object)
   ├─ personalInfo (Object)
   │  ├─ firstName: "John"
   │  ├─ lastName: "Smith"
   │  └─ age: 30
   └─ workInfo (Object)
      ├─ department: "Engineering"
      ├─ title: "Software Engineer"
      └─ startDate: "2020-01-15"
```

**How to navigate:**
```
To get firstName:
  1. Start at root
  2. Go into "employee" object
  3. Go into "personalInfo" object
  4. Read "firstName" value

Path: employee → personalInfo → firstName = "John"
```

---

#### Example 2: Array of Objects
```json
{
  "employees": [
    {
      "name": "John Smith",
      "department": "Engineering"
    },
    {
      "name": "Jane Doe",
      "department": "Finance"
    },
    {
      "name": "Bob Johnson",
      "department": "HR"
    }
  ]
}
```

**Visual structure:**
```
Root Object
└─ employees (Array of 3 Objects)
   ├─ [0] (Object)
   │  ├─ name: "John Smith"
   │  └─ department: "Engineering"
   ├─ [1] (Object)
   │  ├─ name: "Jane Doe"
   │  └─ department: "Finance"
   └─ [2] (Object)
      ├─ name: "Bob Johnson"
      └─ department: "HR"
```

**How to navigate:**
```
To get second employee's name:
  1. Start at root
  2. Go into "employees" array
  3. Access position 1 (second item)
  4. Read "name" value

Path: employees[1] → name = "Jane Doe"
```

---

#### Example 3: Deeply Nested Structure
```json
{
  "company": {
    "name": "Acme Corp",
    "departments": [
      {
        "name": "Engineering",
        "employees": [
          {
            "name": "John Smith",
            "skills": ["Python", "Java"]
          },
          {
            "name": "Jane Doe",
            "skills": ["JavaScript", "React"]
          }
        ]
      }
    ]
  }
}
```

**Visual structure:**
```
Root
└─ company (Object)
   ├─ name: "Acme Corp"
   └─ departments (Array)
      └─ [0] (Object)
         ├─ name: "Engineering"
         └─ employees (Array)
            ├─ [0] (Object)
            │  ├─ name: "John Smith"
            │  └─ skills (Array): ["Python", "Java"]
            └─ [1] (Object)
               ├─ name: "Jane Doe"
               └─ skills (Array): ["JavaScript", "React"]
```

**How to read Jane's second skill:**
```
Path:
company → departments[0] → employees[1] → skills[1] = "React"

Step by step:
1. Start at root
2. Go into "company" object
3. Go into "departments" array, position 0
4. Go into "employees" array, position 1
5. Go into "skills" array, position 1
6. Read value: "React"
```

---

#### Reading Nested JSON: Strategy

**Follow this process:**
```
Step 1: Start at the outermost level
   Identify if it's { } or [ ]

Step 2: Read each key (for objects) or position (for arrays)

Step 3: For each value, determine its type
   String? Number? Object? Array?

Step 4: If Object or Array, go DEEPER
   Repeat the process

Step 5: Build a mental tree
   Visualize parent-child relationships
```

---

#### Practice Exercise 4: Navigate Nested JSON
```json
{
  "user": {
    "id": "U123",
    "profile": {
      "name": "John Smith",
      "contact": {
        "email": "john@company.com",
        "phones": [
          "555-1234",
          "555-5678"
        ]
      }
    },
    "roles": ["admin", "user"]
  }
}
```

**Answer these questions:**

1. What is the user's ID?
2. What is the user's email?
3. What is the first phone number?
4. How many roles does the user have?
5. What is the second role?

<details>
<summary>Click to reveal answers</summary>

1. User ID: "U123"
   - Path: user → id

2. Email: "john@company.com"
   - Path: user → profile → contact → email

3. First phone: "555-1234"
   - Path: user → profile → contact → phones[0]

4. Number of roles: 2
   - Path: user → roles (array length)

5. Second role: "user"
   - Path: user → roles[1]

</details>

---

## Evening Session: Real SailPoint JSON

### Part 6: Reading SailPoint Transforms (60 minutes)

Now let's apply everything you learned to REAL SailPoint ISC JSON!

---

#### SailPoint Transform Structure

Every transform in SailPoint follows this pattern:
```json
{
  "name": "attributeName",
  "type": "transformType",
  "attributes": {
    "configuration": "goes here"
  }
}
```

**Let's break this down:**
```
"name"       → The attribute being created
"type"       → What kind of transform (static, concat, etc.)
"attributes" → Configuration specific to this transform type
```

---

#### Example 1: Static Transform (Simple)

**Remember this from your transforms course?**
```json
{
  "name": "country",
  "type": "static",
  "attributes": {
    "value": "United States"
  }
}
```

**Reading this JSON:**
```
Object with 3 properties:

1. "name": "country"
   → This transform creates the "country" attribute

2. "type": "static"
   → It's a static transform (returns fixed value)

3. "attributes": Object
   └─ "value": "United States"
      → The fixed value to return
```

**What this does:** Sets country attribute to "United States" for everyone

---

#### Example 2: Concatenation Transform

**From your Day 1 transforms course:**
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

**Let's read this step by step:**
```
Main Object:
├─ "name": "displayName"
│  → Creating displayName attribute
│
├─ "type": "concat"
│  → Using concatenation transform
│
└─ "attributes": Object
   └─ "values": Array of 3 Objects
      │
      ├─ [0]: Object
      │  ├─ "type": "identityAttribute"
      │  └─ "attributes": Object
      │     └─ "name": "firstname"
      │        → First value is firstname attribute
      │
      ├─ [1]: Object
      │  ├─ "type": "static"
      │  └─ "attributes": Object
      │     └─ "value": " "
      │        → Second value is a space
      │
      └─ [2]: Object
         ├─ "type": "identityAttribute"
         └─ "attributes": Object
            └─ "name": "lastname"
               → Third value is lastname attribute
```

**What this does:** Creates displayName as "firstname lastname"

---

#### Example 3: Transform Chain (Complex)

**From your Day 5 capstone project:**
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

**Reading this complex chain:**
```
Main Transform:
├─ Creates "email" attribute
├─ First transform: "trim"
│  └─ Input: firstname attribute
│
└─ Next transform: "concat"
   ├─ Values to concatenate:
   │  ├─ [0] Reference (result from trim)
   │  └─ [1] "@company.com"
   │
   └─ Next transform: "lower"
      └─ Converts to lowercase
```

**What this does:**
```
1. Trim firstname
2. Concatenate result + "@company.com"
3. Convert to lowercase

Example: "John" → "John" (trimmed) → "John@company.com" → "john@company.com"
```

---

#### Example 4: Conditional Transform

**From your Day 2 transforms:**
```json
{
  "name": "employeeType",
  "type": "conditional",
  "attributes": {
    "expression": "status == 'Active'",
    "positiveCondition": "Employee",
    "negativeCondition": "Not Active"
  }
}
```

**Reading this:**
```
Conditional Transform:
├─ Creates "employeeType" attribute
├─ Condition: status == 'Active'
├─ If TRUE: return "Employee"
└─ If FALSE: return "Not Active"
```

**What this does:** If-then-else logic based on status

---

#### Pattern Recognition in SailPoint JSON

**Common patterns you'll see:**

**Pattern 1: Simple value**
```json
{
  "type": "static",
  "attributes": {
    "value": "something"
  }
}
```

**Pattern 2: Reference to attribute**
```json
{
  "type": "identityAttribute",
  "attributes": {
    "name": "attributeName"
  }
}
```

**Pattern 3: Transform chaining**
```json
{
  "type": "transformType",
  "attributes": {
    "configuration": "...",
    "next": {
      "type": "nextTransform"
    }
  }
}
```

**Pattern 4: Array of items**
```json
{
  "type": "concat",
  "attributes": {
    "values": [
      { },
      { },
      { }
    ]
  }
}
```

---

#### Practice Exercise 5: Read Real Transform

**Here's a real transform from your capstone. What does it do?**
```json
{
  "name": "lifecycleState",
  "type": "conditional",
  "attributes": {
    "expression": "status == 'On Leave'",
    "positiveCondition": "Leave of Absence",
    "negativeCondition": {
      "type": "conditional",
      "attributes": {
        "expression": "isPreHire == 'true'",
        "positiveCondition": "Pre-hire",
        "negativeCondition": "Active"
      }
    }
  }
}
```

**Questions:**

1. What attribute does this create?
2. What happens if status is "On Leave"?
3. What happens if status is NOT "On Leave" AND isPreHire is true?
4. What's the final fallback value?

<details>
<summary>Click to reveal answers</summary>

1. Creates: "lifecycleState" attribute

2. If status = "On Leave":
   - Returns "Leave of Absence"

3. If status ≠ "On Leave" AND isPreHire = true:
   - Goes to nested conditional
   - Returns "Pre-hire"

4. Final fallback:
   - If all conditions false
   - Returns "Active"

**Full logic:**
```
IF status == "On Leave"
  THEN "Leave of Absence"
ELSE IF isPreHire == true
  THEN "Pre-hire"
ELSE
  "Active"
```

</details>

---

#### Reading Strategy for SailPoint JSON

**Follow this process:**
```
Step 1: Find the "type" field
   This tells you what kind of transform it is

Step 2: Look at "attributes"
   This contains the configuration

Step 3: Identify the pattern
   Static value? Concatenation? Conditional? Chain?

Step 4: For chains, follow the "next" property
   Each "next" is another transform in the chain

Step 5: Build mental model
   What is this trying to accomplish?
```

---

## Day 1 Assessment

### Knowledge Check Quiz

**Answer these 10 questions to verify your understanding:**

#### Question 1: Basic Syntax
What is WRONG with this JSON?
```json
{
  name: "John",
  "age": 30
}
```

<details>
<summary>Click to reveal answer</summary>

**Error:** Missing quotes around key "name"

**Correct:**
```json
{
  "name": "John",
  "age": 30
}
```

</details>

---

#### Question 2: Data Types
Which data type is "age" in this JSON?
```json
{
  "age": 30
}
```

<details>
<summary>Click to reveal answer</summary>

**Answer:** Number

(No quotes around 30, so it's a number, not a string)

</details>

---

#### Question 3: Arrays
What is the second item in this array?
```json
{
  "colors": ["Red", "Blue", "Green"]
}
```

<details>
<summary>Click to reveal answer</summary>

**Answer:** "Blue"

Position 0 = "Red"
Position 1 = "Blue"  ← Second item
Position 2 = "Green"

</details>

---

#### Question 4: Nested Objects
What is the value of "city"?
```json
{
  "person": {
    "name": "John",
    "address": {
      "city": "Austin",
      "state": "TX"
    }
  }
}
```

<details>
<summary>Click to reveal answer</summary>

**Answer:** "Austin"

Path: person → address → city = "Austin"

</details>

---

#### Question 5: Syntax Error
How many errors are in this JSON?
```json
{
  'name': "John",
  "age": 30,
  "active": True,
}
```

<details>
<summary>Click to reveal answer</summary>

**3 errors:**

1. Single quotes on key 'name' (should be "name")
2. Capital T in True (should be lowercase: true)
3. Trailing comma after "active" line

**Correct:**
```json
{
  "name": "John",
  "age": 30,
  "active": true
}
```

</details>

---

#### Question 6: Null vs Empty
What's the difference between these two?
```json
{
  "middleName": null
}
```
```json
{
  "middleName": ""
}
```

<details>
<summary>Click to reveal answer</summary>

**First:** null = no value at all
**Second:** "" = empty string (has a value, but it's empty)

null means "this doesn't exist"
"" means "this exists but is empty"

</details>

---

#### Question 7: SailPoint Transform
What does this transform do?
```json
{
  "type": "static",
  "attributes": {
    "value": "USA"
  }
}
```

<details>
<summary>Click to reveal answer</summary>

**Answer:** Returns the static value "USA" for all identities

This is a static transform that always returns the same value.

</details>

---

#### Question 8: Reading Concatenation
What will this produce if firstname="John" and lastname="Smith"?
```json
{
  "type": "concat",
  "attributes": {
    "values": [
      {
        "type": "identityAttribute",
        "attributes": {"name": "firstname"}
      },
      {
        "type": "static",
        "attributes": {"value": "."}
      },
      {
        "type": "identityAttribute",
        "attributes": {"name": "lastname"}
      }
    ]
  }
}
```

<details>
<summary>Click to reveal answer</summary>

**Answer:** "John.Smith"

Concatenates:
1. firstname ("John")
2. "."
3. lastname ("Smith")

Result: "John.Smith"

</details>

---

#### Question 9: Transform Chain
How many transforms are in this chain?
```json
{
  "type": "trim",
  "attributes": {
    "next": {
      "type": "lower",
      "attributes": {
        "next": {
          "type": "concat"
        }
      }
    }
  }
}
```

<details>
<summary>Click to reveal answer</summary>

**Answer:** 3 transforms

1. Trim
2. Lower (chained via "next")
3. Concat (chained via "next")

</details>

---

#### Question 10: Complex Navigation
What is the value at this path: 
`company → departments[0] → head → email`
```json
{
  "company": {
    "name": "Acme",
    "departments": [
      {
        "name": "Engineering",
        "head": {
          "name": "Jane Doe",
          "email": "jane@company.com"
        }
      }
    ]
  }
}
```

<details>
<summary>Click to reveal answer</summary>

**Answer:** "jane@company.com"

Path breakdown:
1. Start at root
2. Go to "company" object
3. Go to "departments" array
4. Access position 0 (first department)
5. Go to "head" object
6. Read "email" value

Result: "jane@company.com"

</details>

---

### Scoring
```
9-10 correct: Expert! Ready for Day 2!
7-8 correct: Great! Review mistakes and move on
5-6 correct: Good start! Review the session again
0-4 correct: Re-read sections you missed
```

---

## Day 1 Summary

### What You Learned Today

**Morning:**
- ✅ What JSON is and why it exists
- ✅ The 6 fundamental syntax rules
- ✅ How to read simple JSON structures

**Afternoon:**
- ✅ All 6 JSON data types (String, Number, Boolean, Null, Object, Array)
- ✅ How to read nested structures
- ✅ Navigation strategies for complex JSON

**Evening:**
- ✅ Reading real SailPoint ISC transform JSON
- ✅ Understanding transform patterns
- ✅ Following transform chains

---

### Skills Acquired

You can now:
```
✓ Read any JSON file
✓ Identify JSON syntax errors
✓ Understand all data types
✓ Navigate nested structures
✓ Read SailPoint transform configurations
✓ Understand transform chains
✓ Spot common patterns
```

---

### Key Concepts to Remember
```
1. JSON is just structured text
2. { } creates objects (key-value pairs)
3. [ ] creates arrays (lists)
4. " " for all keys and string values
5. : separates keys from values
6. , separates items (not after last!)
7. 6 data types: String, Number, Boolean, Null, Object, Array
8. SailPoint transforms are JSON objects
9. "next" property chains transforms
10. Reading JSON is a skill - practice makes perfect!
```

---

## Homework (Optional but Recommended)

### Exercise 1: Find JSON in the Wild

Go to your SailPoint ISC sandbox:
```
1. Navigate to Admin > Identity Profiles
2. Select any Identity Profile
3. Click on any transform
4. Look at the configuration
5. Try to identify:
   - The "type"
   - The "attributes"
   - Any nested objects
   - Any arrays
```

---

### Exercise 2: Read This Complex JSON
```json
{
  "transform": {
    "name": "email",
    "type": "firstValid",
    "attributes": {
      "values": [
        {
          "type": "identityAttribute",
          "attributes": {
            "name": "personalEmail"
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
                  "value": "@company.com"
                }
              }
            ]
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
}
```

**Questions:**
1. What attribute does this create?
2. What transform type is it?
3. How many fallback values are there?
4. What is the first priority?
5. What is the final fallback?

<details>
<summary>Click to reveal answers</summary>

1. Creates "email" attribute
2. Type: "firstValid"
3. Three fallback values in the array
4. First priority: personalEmail attribute
5. Final fallback: "noemail@company.com"

**Logic:**
```
Priority 1: Use personalEmail if exists
Priority 2: Generate firstname@company.com
Priority 3: Use "noemail@company.com"
```

</details>

---

## Tomorrow: Day 2 Preview

**What You'll Learn:**
```
Writing JSON from scratch
├─ Creating valid JSON
├─ Formatting properly
├─ Using validation tools
├─ Common errors and fixes
└─ Building SailPoint transforms in JSON
```

**You'll be able to:**
- Write valid JSON without tools
- Create SailPoint transforms in a text editor
- Fix JSON syntax errors instantly
- Format JSON for readability
- Use JSON validation tools

---

## Quick Reference Card

**Save this for quick lookup!**

### JSON Syntax Rules
```
1. { } = Object
2. [ ] = Array
3. " " = Keys and Strings
4. : = Separator
5. , = Between items
6. No comments
```

### Data Types
```
String:  "text"
Number:  42
Boolean: true/false
Null:    null
Object:  {"key": "value"}
Array:   [1, 2, 3]
```

### Common Errors
```
'key': value     → Missing double quotes
"key": 'value'   → Single quotes not allowed
"key": True      → Capital T (use true)
"key": value,    → Trailing comma
{key: value}     → Missing quotes on key
```

### SailPoint Patterns
```
Static:
{
  "type": "static",
  "attributes": {
    "value": "something"
  }
}

Attribute Reference:
{
  "type": "identityAttribute",
  "attributes": {
    "name": "attributeName"
  }
}

Chain:
{
  "type": "transform1",
  "attributes": {
    "config": "...",
    "next": {
      "type": "transform2"
    }
  }
}
```

---

**Congratulations on completing Day 1! You can now READ JSON like a pro!**

**Rest up and get ready for Day 2 where you'll learn to WRITE JSON!**

---

*End of JSON Mastery - Day 1*
