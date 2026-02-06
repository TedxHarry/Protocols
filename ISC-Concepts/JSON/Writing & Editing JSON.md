# JSON MASTERY - DAY 2: WRITING & EDITING JSON

## Welcome to Day 2!

Yesterday you learned to READ JSON. Today you'll learn to WRITE it!

**Today's Goal:** Write perfect JSON from scratch and fix any errors instantly

---

## Table of Contents
- [Morning Session: JSON Syntax Mastery](#morning-session-json-syntax-mastery)
  - [Part 1: The Golden Rules](#part-1-the-golden-rules-45-minutes)
  - [Part 2: Common Errors & Fixes](#part-2-common-errors--fixes-45-minutes)
  - [Part 3: Validation Tools](#part-3-validation-tools-30-minutes)
- [Afternoon Session: Building JSON](#afternoon-session-building-json)
  - [Part 4: Writing Step-by-Step](#part-4-writing-step-by-step-60-minutes)
  - [Part 5: Hands-On Projects](#part-5-hands-on-projects-60-minutes)
- [Evening Session: SailPoint Transforms](#evening-session-sailpoint-transforms)
  - [Part 6: Creating Transforms in JSON](#part-6-creating-transforms-in-json-60-minutes)
- [Day 2 Assessment](#day-2-assessment)

---

## Morning Session: JSON Syntax Mastery

### Part 1: The Golden Rules (45 minutes)

Before you write a single character of JSON, memorize these **10 Golden Rules**:

---

#### Rule 1: Always Start with { or [

**Every JSON document starts with one of these:**
```json
{
  "this is an object"
}
```

**OR**
```json
[
  "this is an array"
]
```

**NOT this:**
```
"just a string"  ← Invalid JSON document
42               ← Invalid JSON document
true             ← Invalid JSON document
```

**The root must be an object or array!**

---

#### Rule 2: All Keys Must Have Double Quotes

**✓ CORRECT:**
```json
{
  "name": "John"
}
```

**✗ WRONG:**
```json
{
  name: "John"        // Missing quotes
  'name': "John"      // Single quotes
  "name": "John"      // Smart quotes (looks right but isn't!)
}
```

**Pro Tip:** Use a proper code editor (VS Code, Sublime, etc.) that uses straight quotes

---

#### Rule 3: All String Values Must Have Double Quotes

**✓ CORRECT:**
```json
{
  "name": "John",
  "city": "Austin",
  "status": "active"
}
```

**✗ WRONG:**
```json
{
  "name": John,           // Missing quotes
  "city": 'Austin',       // Single quotes
  "status": "active"      // Smart quotes
}
```

**Remember:** Numbers, booleans, and null don't use quotes!

---

#### Rule 4: Use Colon Between Key and Value

**✓ CORRECT:**
```json
{
  "key": "value"
}
```

**✗ WRONG:**
```json
{
  "key" = "value"     // Equal sign, not colon
  "key""value"        // Missing colon
  "key" : : "value"   // Double colon
}
```

**Mental model:** Think of `:` as `=` (equals)

---

#### Rule 5: Use Comma Between Items (But NOT After Last)

**✓ CORRECT:**
```json
{
  "first": "John",
  "last": "Smith",
  "age": 30
}
```

**✗ WRONG:**
```json
{
  "first": "John"
  "last": "Smith"     // Missing comma between items
  "age": 30
}
```

**✗ ALSO WRONG:**
```json
{
  "first": "John",
  "last": "Smith",
  "age": 30,          // Trailing comma after last item
}
```

**The Comma Rule:**
```
Item 1,  ← Comma here
Item 2,  ← Comma here
Item 3   ← NO comma (it's the last one!)
```

---

#### Rule 6: Proper Nesting - Open and Close

**Every opening brace/bracket MUST have a closing one:**
```json
{                    ← Opening brace
  "object": {        ← Opening nested brace
    "key": "value"
  }                  ← Closing nested brace
}                    ← Closing brace
```

**Visual counting method:**
```json
{                    ← Count: 1 open
  "data": {          ← Count: 2 open
    "nested": {      ← Count: 3 open
      "value": "x"
    }                ← Count: 2 open (closed one)
  }                  ← Count: 1 open (closed another)
}                    ← Count: 0 (all closed!) ✓
```

**If your count doesn't end at 0, you have unmatched braces!**

---

#### Rule 7: No Comments Allowed in JSON

**✗ WRONG:**
```json
{
  // This is a comment  ← NOT ALLOWED
  "name": "John",
  /* Multi-line comment */ ← NOT ALLOWED
  "age": 30
}
```

**If you need notes:**

**Option A:** Put them in a separate documentation file

**Option B:** Add a special key (but it will be part of the data):
```json
{
  "_comment": "This is metadata, not actually used",
  "name": "John",
  "age": 30
}
```

---

#### Rule 8: Keys Must Be Unique in Objects

**✗ WRONG:**
```json
{
  "name": "John",
  "name": "Jane"     ← Duplicate key!
}
```

**What happens:** Only the last value is kept ("Jane")

**✓ CORRECT:**
```json
{
  "firstName": "John",
  "lastName": "Jane"
}
```

---

#### Rule 9: Escape Special Characters in Strings

**Certain characters need escaping with backslash `\`:**
```json
{
  "quote": "He said \"Hello\"",
  "backslash": "C:\\Users\\John",
  "newline": "Line 1\nLine 2",
  "tab": "Column1\tColumn2"
}
```

**Common escapes:**

| Character | Escape | Example |
|-----------|--------|---------|
| Quote | `\"` | `"Say \"Hi\""` |
| Backslash | `\\` | `"C:\\path"` |
| Newline | `\n` | `"Line1\nLine2"` |
| Tab | `\t` | `"Col1\tCol2"` |
| Carriage return | `\r` | `"Line\r\n"` |

---

#### Rule 10: Case Sensitivity Matters

**These are ALL different:**
```json
{
  "name": "John",
  "Name": "Jane",
  "NAME": "Bob"
}
```

**Booleans and null are lowercase only:**
```json
{
  "active": true,      // ✓ Correct
  "deleted": false,    // ✓ Correct
  "value": null        // ✓ Correct
}
```

**✗ WRONG:**
```json
{
  "active": True,      // Capital T
  "deleted": FALSE,    // All caps
  "value": NULL        // All caps
}
```

---

### The Golden Rules Summary Card

**Print this and keep it visible while writing JSON:**
```
═══════════════════════════════════════════════════════════
               JSON GOLDEN RULES
═══════════════════════════════════════════════════════════

1.  Start with { or [
2.  All keys: "double quotes"
3.  String values: "double quotes"
4.  Use : between key and value
5.  Use , between items (not after last!)
6.  Open { [ must close } ]
7.  No comments allowed
8.  No duplicate keys
9.  Escape special chars: \" \\ \n \t
10. Case sensitive (true not True)

═══════════════════════════════════════════════════════════
```

---

### Practice Exercise 1: Spot the Violations

**Each of these violates a golden rule. Which rule and how?**

**Example A:**
```json
name: "John"
```

<details>
<summary>Click to reveal answer</summary>

**Violations:**
1. Rule 1: Not starting with { or [
2. Rule 2: Key missing quotes

**Fixed:**
```json
{
  "name": "John"
}
```

</details>

---

**Example B:**
```json
{
  'firstName': "John",
  "lastName": "Smith"
}
```

<details>
<summary>Click to reveal answer</summary>

**Violation:**
- Rule 2: Single quotes on key (should be double)

**Fixed:**
```json
{
  "firstName": "John",
  "lastName": "Smith"
}
```

</details>

---

**Example C:**
```json
{
  "name": "John",
  "age": 30,
}
```

<details>
<summary>Click to reveal answer</summary>

**Violation:**
- Rule 5: Trailing comma after last item

**Fixed:**
```json
{
  "name": "John",
  "age": 30
}
```

</details>

---

**Example D:**
```json
{
  "active": True,
  "deleted": false
}
```

<details>
<summary>Click to reveal answer</summary>

**Violation:**
- Rule 10: Capital T in True (should be lowercase)

**Fixed:**
```json
{
  "active": true,
  "deleted": false
}
```

</details>

---

**Example E:**
```json
{
  "message": "He said "Hello""
}
```

<details>
<summary>Click to reveal answer</summary>

**Violation:**
- Rule 9: Quotes inside string not escaped

**Fixed:**
```json
{
  "message": "He said \"Hello\""
}
```

</details>

---

### Part 2: Common Errors & Fixes (45 minutes)

Let's learn the **10 most common JSON errors** and how to fix them instantly!

---

#### Error 1: Missing Comma Between Items

**ERROR:**
```json
{
  "firstName": "John"
  "lastName": "Smith"
}
```

**Error Message:**
```
Unexpected token "lastName"
or
Expected comma or }
```

**How to spot:** Look for two items with no comma between them

**Fix:**
```json
{
  "firstName": "John",   ← Add comma here
  "lastName": "Smith"
}
```

---

#### Error 2: Trailing Comma

**ERROR:**
```json
{
  "firstName": "John",
  "lastName": "Smith",   ← Extra comma
}
```

**Error Message:**
```
Trailing comma
or
Unexpected token }
```

**How to spot:** Comma right before closing brace or bracket

**Fix:**
```json
{
  "firstName": "John",
  "lastName": "Smith"    ← Remove comma
}
```

---

#### Error 3: Single Quotes Instead of Double

**ERROR:**
```json
{
  'name': 'John'
}
```

**Error Message:**
```
Unexpected token '
or
Invalid character
```

**How to spot:** Look for ' instead of "

**Fix:**
```json
{
  "name": "John"         ← Change to double quotes
}
```

---

#### Error 4: Missing Quotes on Key

**ERROR:**
```json
{
  name: "John"
}
```

**Error Message:**
```
Unexpected token name
or
Property name expected
```

**How to spot:** Key without quotes before colon

**Fix:**
```json
{
  "name": "John"         ← Add quotes around key
}
```

---

#### Error 5: Missing Quotes on String Value

**ERROR:**
```json
{
  "name": John
}
```

**Error Message:**
```
Unexpected token John
or
Invalid value
```

**How to spot:** Text without quotes after colon

**Fix:**
```json
{
  "name": "John"         ← Add quotes around value
}
```

---

#### Error 6: Unmatched Braces/Brackets

**ERROR:**
```json
{
  "person": {
    "name": "John"
  }
  // Missing closing }
```

**Error Message:**
```
Unexpected end of input
or
Expected }
```

**How to spot:** Count opening and closing braces - should match!

**Fix:**
```json
{
  "person": {
    "name": "John"
  }
}                        ← Add missing closing brace
```

**Counting trick:**
```
{              Count: 1
  "person": {  Count: 2
    "name": "John"
  }            Count: 1 (one closed)
}              Count: 0 (all closed!) ✓
```

---

#### Error 7: Using = Instead of :

**ERROR:**
```json
{
  "name" = "John"
}
```

**Error Message:**
```
Unexpected token =
```

**How to spot:** Equal sign between key and value

**Fix:**
```json
{
  "name": "John"         ← Use colon, not equals
}
```

---

#### Error 8: Boolean/Null with Wrong Case

**ERROR:**
```json
{
  "active": True,
  "value": NULL
}
```

**Error Message:**
```
Unexpected token True
or
Invalid value
```

**How to spot:** Capitalized true/false/null

**Fix:**
```json
{
  "active": true,        ← Lowercase
  "value": null          ← Lowercase
}
```

---

#### Error 9: Number with Quotes (When You Want a Number)

**ERROR:**
```json
{
  "age": "30"
}
```

**This is VALID JSON but...**
- "30" is a STRING
- 30 is a NUMBER

**They're different types!**

**If you want a number:**
```json
{
  "age": 30              ← No quotes
}
```

**If you want a string:**
```json
{
  "age": "30"            ← With quotes
}
```

---

#### Error 10: Unescaped Special Characters

**ERROR:**
```json
{
  "path": "C:\Users\John"
}
```

**Error Message:**
```
Invalid escape sequence
or
Unexpected token
```

**How to spot:** Backslashes not escaped

**Fix:**
```json
{
  "path": "C:\\Users\\John"  ← Escape backslashes
}
```

---

### Error Detection Checklist

**When your JSON has an error, check these in order:**
```
□ 1. Are all keys in "double quotes"?
□ 2. Are all string values in "double quotes"?
□ 3. Is there a comma between every pair of items?
□ 4. Is there NO comma after the last item?
□ 5. Does every { have a matching }?
□ 6. Does every [ have a matching ]?
□ 7. Are true/false/null lowercase?
□ 8. Are special characters escaped (\", \\, \n)?
□ 9. Are you using : between key and value (not =)?
□ 10. Is your JSON starting with { or [?
```

---

### Practice Exercise 2: Find and Fix Errors

**Fix all errors in this JSON:**
```json
{
  'person': {
    name: "John Smith",
    "age": "30",
    active: True,
    "email": "john@company.com",
    "address": {
      "city": "Austin"
      "state": "TX",
    }
  }
  "role": "Engineer"
}
```

<details>
<summary>Click to reveal answer</summary>

**Errors found:**
1. Line 2: Single quotes on 'person' → Change to "person"
2. Line 3: Missing quotes on name → Change to "name"
3. Line 5: True capitalized → Change to true
4. Line 8: Missing comma after "city": "Austin"
5. Line 9: Trailing comma after "state": "TX"
6. Line 10: Missing closing } for person object
7. Line 12: Missing comma before "role"

**Fixed JSON:**
```json
{
  "person": {
    "name": "John Smith",
    "age": "30",
    "active": true,
    "email": "john@company.com",
    "address": {
      "city": "Austin",
      "state": "TX"
    }
  },
  "role": "Engineer"
}
```

</details>

---

### Part 3: Validation Tools (30 minutes)

Don't rely on your eyes alone! Use tools to validate JSON.

---

#### Tool 1: JSONLint (Online Validator)

**Website:** https://jsonlint.com

**How to use:**
```
1. Go to jsonlint.com
2. Paste your JSON
3. Click "Validate JSON"
4. See errors with line numbers
5. Fix and re-validate
```

**Example Error Message:**
```
Error: Parse error on line 5:
...  "age": 30,}
--------------^
Expecting 'STRING', 'NUMBER', 'NULL', 'TRUE', 'FALSE', '{', '['
```

**This tells you:**
- Line 5 has the error
- It found an unexpected }
- It was expecting a value

---

#### Tool 2: VS Code (Best JSON Editor)

**Why VS Code?**
```
✓ Real-time syntax checking
✓ Auto-formatting (makes JSON pretty)
✓ Auto-completion
✓ Brace matching
✓ Error highlighting
✓ FREE!
```

**How to set up VS Code for JSON:**
```
1. Download VS Code (free from code.visualstudio.com)
2. Create new file: File > New File
3. Save as: anything.json
4. VS Code automatically enables JSON mode
5. Start typing JSON
6. See errors highlighted in red immediately!
```

**VS Code JSON Features:**
```
Auto-Format:
  Right-click → Format Document
  or
  Shift + Alt + F (Windows)
  Shift + Option + F (Mac)

Brace Matching:
  Click on { and matching } highlights

Error Detection:
  Red squiggly lines show errors
  Hover over red line to see error message

Auto-Complete:
  Type " and it auto-adds closing "
  Type { and it auto-adds closing }
```

---

#### Tool 3: Browser Developer Tools

**Every browser has built-in JSON validation:**

**How to use (Chrome/Firefox):**
```
1. Press F12 to open Developer Tools
2. Go to Console tab
3. Type: JSON.parse('your json here')
4. Press Enter
5. See error if invalid, or parsed object if valid
```

**Example:**
```javascript
// Valid JSON
JSON.parse('{"name": "John"}')
// Returns: {name: "John"} ✓

// Invalid JSON
JSON.parse('{name: "John"}')
// Returns: Unexpected token n in JSON at position 1 ✗
```

---

#### Tool 4: Command Line (for advanced users)

**Mac/Linux:**
```bash
echo '{"name": "John"}' | python -m json.tool
```

**Windows:**
```powershell
echo '{"name": "John"}' | python -m json.tool
```

**If valid:** Pretty-printed JSON
**If invalid:** Error message

---

#### Tool 5: Online JSON Formatters

**These make JSON readable:**

**JSONFormatter.org**
- Paste messy JSON
- Click "Format"
- Get beautifully formatted JSON

**Before:**
```json
{"name":"John","age":30,"active":true}
```

**After:**
```json
{
  "name": "John",
  "age": 30,
  "active": true
}
```

---

#### Recommended Workflow
```
Step 1: Write JSON in VS Code
  - Get real-time error detection
  - Auto-complete helps prevent errors

Step 2: Format with VS Code
  - Shift + Alt/Option + F
  - Makes it readable

Step 3: Validate with JSONLint
  - Double-check before using
  - Clear error messages

Step 4: Use in SailPoint
  - Paste validated JSON
  - Confident it will work!
```

---

### Practice Exercise 3: Tool Practice

**Take this messy, broken JSON and fix it using tools:**
```json
{person:{name:"John Smith",age:"30",email:"john@company.com"active:true}}
```

**Steps:**
```
1. Paste into VS Code
2. Save as practice.json
3. See red squigglies appear
4. Add proper formatting
5. Fix all errors
6. Validate in JSONLint
7. Format in VS Code
```

<details>
<summary>Click to reveal fixed version</summary>

**Fixed and formatted:**
```json
{
  "person": {
    "name": "John Smith",
    "age": "30",
    "email": "john@company.com",
    "active": true
  }
}
```

**Errors fixed:**
1. Added quotes around "person" key
2. Added quotes around "name", "email", "active" keys
3. Added comma after email
4. Added proper indentation
5. Added line breaks for readability

</details>

---

## Afternoon Session: Building JSON

### Part 4: Writing Step-by-Step (60 minutes)

Now let's write JSON from SCRATCH using a systematic approach.

---

#### The 5-Step Writing Process
```
Step 1: Decide the structure (Object or Array?)
Step 2: Start with the outer brackets
Step 3: Add keys and placeholders
Step 4: Fill in values
Step 5: Validate and format
```

---

#### Example 1: Simple Person Object

**Requirement:** Create JSON for a person with name and age

**Step 1: Decide structure**
```
This is ONE person with properties → Object { }
```

**Step 2: Start with brackets**
```json
{

}
```

**Step 3: Add keys**
```json
{
  "name":,
  "age":
}
```

**Step 4: Fill in values**
```json
{
  "name": "John Smith",
  "age": 30
}
```

**Step 5: Validate**
```
✓ Keys have quotes
✓ String value has quotes
✓ Number value has no quotes
✓ Comma between items
✓ No trailing comma
✓ VALID!
```

---

#### Example 2: Array of Colors

**Requirement:** List of 5 colors

**Step 1: Decide structure**
```
This is a LIST of items → Array [ ]
```

**Step 2: Start with brackets**
```json
[

]
```

**Step 3: Add placeholders**
```json
[
  ,
  ,
  ,
  ,
]
```

**Step 4: Fill in values**
```json
[
  "Red",
  "Blue",
  "Green",
  "Yellow",
  "Purple"
]
```

**Step 5: Validate and fix**
```json
[
  "Red",
  "Blue",
  "Green",
  "Yellow",
  "Purple"     ← Remove trailing comma
]
```

---

#### Example 3: Person with Address (Nested)

**Requirement:** Person with name and address (city, state)

**Step 1: Decide structure**
```
Main: Object (person)
  - name: String
  - address: Object (nested)
    - city: String
    - state: String
```

**Step 2: Start with outer brackets**
```json
{

}
```

**Step 3: Add first-level keys**
```json
{
  "name":,
  "address":
}
```

**Step 4: Add nested structure**
```json
{
  "name":,
  "address": {
    "city":,
    "state":
  }
}
```

**Step 5: Fill in values**
```json
{
  "name": "John Smith",
  "address": {
    "city": "Austin",
    "state": "TX"
  }
}
```

**Step 6: Validate**
```
✓ All keys quoted
✓ All strings quoted
✓ Commas correct
✓ Braces matched
✓ VALID!
```

---

#### Example 4: Array of Objects

**Requirement:** List of 3 employees (name and department each)

**Step 1: Decide structure**
```
Main: Array [ ] (list of employees)
  Each item: Object { } (employee)
    - name: String
    - department: String
```

**Step 2: Start with array**
```json
[

]
```

**Step 3: Add object placeholders**
```json
[
  {
  },
  {
  },
  {
  }
]
```

**Step 4: Add keys to each object**
```json
[
  {
    "name":,
    "department":
  },
  {
    "name":,
    "department":
  },
  {
    "name":,
    "department":
  }
]
```

**Step 5: Fill in values**
```json
[
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
```

**Step 6: Validate**
```
✓ Array of objects
✓ All keys quoted
✓ All values quoted (strings)
✓ Commas between objects
✓ No trailing comma after last object
✓ VALID!
```

---

#### Pro Tips for Writing

**Tip 1: Start Small, Build Up**
```
Don't try to write a complex structure all at once!

Start with:
{
  "simple": "value"
}

Then add:
{
  "simple": "value",
  "another": "value"
}

Then nest:
{
  "simple": "value",
  "nested": {
    "key": "value"
  }
}
```

---

**Tip 2: Use Indentation**
```
Bad (hard to read):
{"person":{"name":"John","address":{"city":"Austin"}}}

Good (easy to read):
{
  "person": {
    "name": "John",
    "address": {
      "city": "Austin"
    }
  }
}
```

**Standard indentation: 2 or 4 spaces**

---

**Tip 3: One Property Per Line**
```
Bad:
{"name": "John", "age": 30, "city": "Austin"}

Good:
{
  "name": "John",
  "age": 30,
  "city": "Austin"
}
```

---

**Tip 4: Validate Frequently**
```
Don't write 100 lines then validate!

Write 5-10 lines → Validate → Continue
```

---

### Practice Exercise 4: Build from Scratch

**Build JSON for this requirement:**
```
A company object with:
- name: "Acme Corp"
- Founded: 1990
- Active: true
- Employees: Array of 2 people
  - Person 1: Name "John", Role "CEO"
  - Person 2: Name "Jane", Role "CTO"
```

**Try it yourself first, then check the answer!**

<details>
<summary>Click to reveal answer</summary>
```json
{
  "name": "Acme Corp",
  "founded": 1990,
  "active": true,
  "employees": [
    {
      "name": "John",
      "role": "CEO"
    },
    {
      "name": "Jane",
      "role": "CTO"
    }
  ]
}
```

**Structure breakdown:**
```
Main Object
├─ name: String
├─ founded: Number
├─ active: Boolean
└─ employees: Array
   ├─ [0]: Object
   │  ├─ name: String
   │  └─ role: String
   └─ [1]: Object
      ├─ name: String
      └─ role: String
```

</details>

---

### Part 5: Hands-On Projects (60 minutes)

Let's build 5 progressively complex JSON structures!

---

#### Project 1: Student Record

**Build JSON for a student:**
```
Requirements:
- Student ID: "S12345"
- First Name: "John"
- Last Name: "Smith"
- GPA: 3.8
- Enrolled: true
- Graduation Year: null (hasn't graduated yet)
```

**Build it yourself first!**

<details>
<summary>Click to reveal answer</summary>
```json
{
  "studentId": "S12345",
  "firstName": "John",
  "lastName": "Smith",
  "gpa": 3.8,
  "enrolled": true,
  "graduationYear": null
}
```

**Key points:**
- String values in quotes
- Number (3.8) no quotes
- Boolean (true) no quotes
- Null is lowercase, no quotes

</details>

---

#### Project 2: Product Catalog

**Build JSON for a product:**
```
Requirements:
- Product Name: "Laptop"
- SKU: "LAP-001"
- Price: 999.99
- In Stock: true
- Categories: ["Electronics", "Computers", "Business"]
- Specs:
  - RAM: "16GB"
  - Storage: "512GB SSD"
  - Screen: "15 inch"
```

**Build it yourself first!**

<details>
<summary>Click to reveal answer</summary>
```json
{
  "productName": "Laptop",
  "sku": "LAP-001",
  "price": 999.99,
  "inStock": true,
  "categories": [
    "Electronics",
    "Computers",
    "Business"
  ],
  "specs": {
    "ram": "16GB",
    "storage": "512GB SSD",
    "screen": "15 inch"
  }
}
```

**Structure:**
```
Main Object
├─ productName: String
├─ sku: String
├─ price: Number
├─ inStock: Boolean
├─ categories: Array of Strings
└─ specs: Nested Object
   ├─ ram: String
   ├─ storage: String
   └─ screen: String
```

</details>

---

#### Project 3: Restaurant Menu

**Build JSON for a menu section:**
```
Requirements:
Section: "Appetizers"
Items: 3 dishes
  1. Name: "Wings", Price: 12.99, Spicy: true
  2. Name: "Nachos", Price: 10.99, Spicy: false
  3. Name: "Salad", Price: 8.99, Spicy: false
```

**Build it yourself first!**

<details>
<summary>Click to reveal answer</summary>
```json
{
  "section": "Appetizers",
  "items": [
    {
      "name": "Wings",
      "price": 12.99,
      "spicy": true
    },
    {
      "name": "Nachos",
      "price": 10.99,
      "spicy": false
    },
    {
      "name": "Salad",
      "price": 8.99,
      "spicy": false
    }
  ]
}
```

**Structure:**
```
Main Object
├─ section: String
└─ items: Array of Objects
   ├─ [0]: Object {name, price, spicy}
   ├─ [1]: Object {name, price, spicy}
   └─ [2]: Object {name, price, spicy}
```

</details>

---

#### Project 4: Organization Chart

**Build JSON for a department:**
```
Requirements:
Department: "Engineering"
Head: "Jane Doe"
Team Members: 
  - Name: "John", Title: "Senior Engineer", Years: 5
  - Name: "Bob", Title: "Engineer", Years: 2
Projects:
  - Project 1: Name "API", Status "Active", Priority "High"
  - Project 2: Name "Mobile App", Status "Planning", Priority "Medium"
```

**Build it yourself first!**

<details>
<summary>Click to reveal answer</summary>
```json
{
  "department": "Engineering",
  "head": "Jane Doe",
  "teamMembers": [
    {
      "name": "John",
      "title": "Senior Engineer",
      "years": 5
    },
    {
      "name": "Bob",
      "title": "Engineer",
      "years": 2
    }
  ],
  "projects": [
    {
      "name": "API",
      "status": "Active",
      "priority": "High"
    },
    {
      "name": "Mobile App",
      "status": "Planning",
      "priority": "Medium"
    }
  ]
}
```

**Structure:**
```
Main Object
├─ department: String
├─ head: String
├─ teamMembers: Array of Objects
│  ├─ [0]: {name, title, years}
│  └─ [1]: {name, title, years}
└─ projects: Array of Objects
   ├─ [0]: {name, status, priority}
   └─ [1]: {name, status, priority}
```

</details>

---

#### Project 5: Complex Employee Record

**Build JSON for an employee:**
```
Requirements:
- Employee ID: "E12345"
- Name: "John Smith"
- Department: "Engineering"
- Salary: 85000
- Skills: ["Python", "JavaScript", "SQL"]
- Address:
  - Street: "123 Main St"
  - City: "Austin"
  - State: "TX"
  - Zip: "78701"
- Work History: 2 jobs
  - Job 1: Company "TechCorp", Title "Developer", Years "2018-2020"
  - Job 2: Company "StartupXYZ", Title "Engineer", Years "2020-2024"
- Active: true
- Manager: null (no manager assigned yet)
```

**Build it yourself first!**

<details>
<summary>Click to reveal answer</summary>
```json
{
  "employeeId": "E12345",
  "name": "John Smith",
  "department": "Engineering",
  "salary": 85000,
  "skills": [
    "Python",
    "JavaScript",
    "SQL"
  ],
  "address": {
    "street": "123 Main St",
    "city": "Austin",
    "state": "TX",
    "zip": "78701"
  },
  "workHistory": [
    {
      "company": "TechCorp",
      "title": "Developer",
      "years": "2018-2020"
    },
    {
      "company": "StartupXYZ",
      "title": "Engineer",
      "years": "2020-2024"
    }
  ],
  "active": true,
  "manager": null
}
```

**Structure (Complex!):**
```
Main Object
├─ employeeId: String
├─ name: String
├─ department: String
├─ salary: Number
├─ skills: Array of Strings
├─ address: Nested Object
│  ├─ street: String
│  ├─ city: String
│  ├─ state: String
│  └─ zip: String
├─ workHistory: Array of Objects
│  ├─ [0]: Object {company, title, years}
│  └─ [1]: Object {company, title, years}
├─ active: Boolean
└─ manager: Null
```

</details>

---

## Evening Session: SailPoint Transforms

### Part 6: Creating Transforms in JSON (60 minutes)

Now let's apply your JSON skills to writing **SailPoint ISC transforms!**

---

#### SailPoint Transform JSON Pattern

**Every transform follows this structure:**
```json
{
  "name": "attributeName",
  "type": "transformType",
  "attributes": {
    "configuration": "specific to transform type"
  }
}
```

---

#### Transform 1: Static Transform (Simplest)

**Create a static transform that sets country to "United States"**

**Step-by-step:**

**Step 1: Start with basic structure**
```json
{
  "name": "",
  "type": "",
  "attributes": {
  }
}
```

**Step 2: Fill in name and type**
```json
{
  "name": "country",
  "type": "static",
  "attributes": {
  }
}
```

**Step 3: Add static-specific configuration**
```json
{
  "name": "country",
  "type": "static",
  "attributes": {
    "value": "United States"
  }
}
```

**Done! Valid static transform.**

---

#### Transform 2: Concatenation Transform

**Create displayName as "firstname lastname"**

**Pattern for concatenation:**
```json
{
  "type": "concat",
  "attributes": {
    "values": [
      array of things to concatenate
    ]
  }
}
```

**Step-by-step:**

**Step 1: Basic structure**
```json
{
  "name": "displayName",
  "type": "concat",
  "attributes": {
    "values": [
    ]
  }
}
```

**Step 2: Add firstname reference**
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
      }
    ]
  }
}
```

**Step 3: Add space**
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
      }
    ]
  }
}
```

**Step 4: Add lastname**
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

**Done! Valid concatenation transform.**

---

#### Transform 3: Transform Chain (Trim + Lower)

**Create username as lowercase trimmed firstname**

**Pattern for chaining:**
```json
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

**Step-by-step:**

**Step 1: First transform (Trim)**
```json
{
  "name": "username",
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

**Step 2: Add next transform (Lower)**
```json
{
  "name": "username",
  "type": "trim",
  "attributes": {
    "input": {
      "type": "identityAttribute",
      "attributes": {
        "name": "firstname"
      }
    },
    "next": {
      "type": "lower"
    }
  }
}
```

**Done! Chained transform: Trim → Lower**

**What it does:**
```
Input: "  John  "
After Trim: "John"
After Lower: "john"
```

---

#### Transform 4: Conditional Transform

**Create employeeType: If status="Active" then "Employee" else "Inactive"**

**Pattern for conditional:**
```json
{
  "type": "conditional",
  "attributes": {
    "expression": "condition",
    "positiveCondition": "value if true",
    "negativeCondition": "value if false"
  }
}
```

**Build it:**
```json
{
  "name": "employeeType",
  "type": "conditional",
  "attributes": {
    "expression": "status == 'Active'",
    "positiveCondition": "Employee",
    "negativeCondition": "Inactive"
  }
}
```

**Done! Conditional transform.**

---

#### Transform 5: FirstValid (Multiple Fallbacks)

**Create email with fallbacks: personal_email → work_email → "noemail@company.com"**

**Pattern for FirstValid:**
```json
{
  "type": "firstValid",
  "attributes": {
    "values": [
      option1,
      option2,
      option3
    ]
  }
}
```

**Build it:**
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

**Done! FirstValid with 3 fallback levels.**

---

#### Transform 6: Complex Chain (Trim + Concat + Lower)

**Create email as "trimmed_firstname.trimmed_lastname@company.com" (lowercase)**

**Build it step by step:**
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
          "type": "lower"
        }
      }
    }
  }
}
```

**What it does:**
```
Step 1: Trim firstname ("  John  " → "John")
Step 2: Concatenate:
  - Result from step 1 ("John")
  - "."
  - Trimmed lastname ("Smith")
  - "@company.com"
  Result: "John.Smith@company.com"
Step 3: Lower → "john.smith@company.com"
```

---

### Practice Exercise 5: Build SailPoint Transforms

**Build these transforms in JSON:**

#### Exercise A: Static Department

Create a transform named "defaultDepartment" that always returns "General"

<details>
<summary>Click to reveal answer</summary>
```json
{
  "name": "defaultDepartment",
  "type": "static",
  "attributes": {
    "value": "General"
  }
}
```

</details>

---

#### Exercise B: Concatenate Email

Create "email" as "firstname.lastname@company.com"

<details>
<summary>Click to reveal answer</summary>
```json
{
  "name": "email",
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
  }
}
```

</details>

---

#### Exercise C: Chain Trim and Upper

Create "departmentCode" as uppercase trimmed dept_code

<details>
<summary>Click to reveal answer</summary>
```json
{
  "name": "departmentCode",
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

</details>

---

#### Exercise D: Conditional with Nested Fallback

Create "lifecycleState":
- If status = "Active" → "Active"
- Else if status = "Terminated" → "Inactive"
- Else → "Unknown"

<details>
<summary>Click to reveal answer</summary>
```json
{
  "name": "lifecycleState",
  "type": "conditional",
  "attributes": {
    "expression": "status == 'Active'",
    "positiveCondition": "Active",
    "negativeCondition": {
      "type": "conditional",
      "attributes": {
        "expression": "status == 'Terminated'",
        "positiveCondition": "Inactive",
        "negativeCondition": "Unknown"
      }
    }
  }
}
```

</details>

---

### SailPoint Transform Templates

**Save these as templates for future use!**

#### Static Template
```json
{
  "name": "ATTRIBUTE_NAME",
  "type": "static",
  "attributes": {
    "value": "STATIC_VALUE"
  }
}
```

#### Concatenation Template
```json
{
  "name": "ATTRIBUTE_NAME",
  "type": "concat",
  "attributes": {
    "values": [
      {
        "type": "identityAttribute",
        "attributes": {
          "name": "SOURCE_ATTRIBUTE"
        }
      },
      {
        "type": "static",
        "attributes": {
          "value": "DELIMITER"
        }
      }
    ]
  }
}
```

#### Transform Chain Template
```json
{
  "name": "ATTRIBUTE_NAME",
  "type": "FIRST_TRANSFORM",
  "attributes": {
    "input": {
      "type": "identityAttribute",
      "attributes": {
        "name": "SOURCE"
      }
    },
    "next": {
      "type": "NEXT_TRANSFORM"
    }
  }
}
```

#### Conditional Template
```json
{
  "name": "ATTRIBUTE_NAME",
  "type": "conditional",
  "attributes": {
    "expression": "CONDITION",
    "positiveCondition": "TRUE_VALUE",
    "negativeCondition": "FALSE_VALUE"
  }
}
```

#### FirstValid Template
```json
{
  "name": "ATTRIBUTE_NAME",
  "type": "firstValid",
  "attributes": {
    "values": [
      {
        "type": "identityAttribute",
        "attributes": {
          "name": "OPTION_1"
        }
      },
      {
        "type": "static",
        "attributes": {
          "value": "FALLBACK"
        }
      }
    ],
    "ignoreErrors": true
  }
}
```

---

## Day 2 Assessment

### Final Skills Check

**Build these 3 transforms completely in JSON:**

#### Challenge 1: Department Lookup

Create a transform that:
- Takes dept_code attribute
- Trims it
- Converts to uppercase
- Returns result

Name: "normalizedDeptCode"

<details>
<summary>Click to reveal answer</summary>
```json
{
  "name": "normalizedDeptCode",
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

</details>

---

#### Challenge 2: Smart Email Generator

Create an email transform with fallbacks:
1. Use personal_email if exists
2. Otherwise generate firstname.lastname@company.com
3. Otherwise use employee_id@company.com
4. Final fallback: "noemail@company.com"

Name: "primaryEmail"

<details>
<summary>Click to reveal answer</summary>
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
        }
      },
      {
        "type": "concat",
        "attributes": {
          "values": [
            {
              "type": "identityAttribute",
              "attributes": {
                "name": "employee_id"
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
    ],
    "ignoreErrors": true
  }
}
```

</details>

---

#### Challenge 3: Lifecycle State Logic

Create lifecycleState with this logic:
- If status = "On Leave" → "Leave of Absence"
- Else if status = "Active" → "Active"
- Else if status = "Terminated" → "Terminated"
- Else → "Unknown"

Name: "lifecycleState"

<details>
<summary>Click to reveal answer</summary>
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
  }
}
```

</details>

---

## Day 2 Summary

### What You Learned Today

**Morning:**
- ✅ The 10 Golden Rules of JSON syntax
- ✅ 10 most common errors and how to fix them
- ✅ Validation tools (JSONLint, VS Code, etc.)

**Afternoon:**
- ✅ 5-step process for writing JSON
- ✅ Building simple to complex structures
- ✅ 5 hands-on projects completed

**Evening:**
- ✅ SailPoint transform JSON patterns
- ✅ Writing 6 different transform types
- ✅ Creating complex chained transforms

---

### Skills Acquired

You can now:
```
✓ Write valid JSON from scratch
✓ Fix any JSON syntax error
✓ Use validation tools effectively
✓ Build nested JSON structures
✓ Create arrays of objects
✓ Write SailPoint transforms in JSON
✓ Chain multiple transforms
✓ Use templates for common patterns
```

---

### Key Concepts to Remember
```
1. Always validate your JSON before using it
2. Use VS Code for real-time error detection
3. Start simple, build up complexity
4. Follow the 5-step writing process
5. Use proper indentation (2 or 4 spaces)
6. Remember the comma rules (between items, not after last)
7. SailPoint transforms follow predictable patterns
8. Use "next" to chain transforms
9. Use templates to speed up writing
10. Practice makes perfect!
```

---

## Homework (Highly Recommended)

### Exercise 1: Build Your Own Transform

Using what you learned, create a complete transform in JSON:

**Requirements:**
- Name: "displayNameFormatted"
- Take firstname and lastname
- Trim both
- Concatenate as "LASTNAME, Firstname"
- Uppercase the lastname part
- Lowercase the firstname part

<details>
<summary>Click for hint</summary>

You'll need:
1. Trim transform for lastname
2. Upper transform
3. Trim transform for firstname
4. Lower transform
5. Concatenation to combine
6. Chaining with "next"

</details>

---

### Exercise 2: Fix Broken JSON

**Find and fix all errors in this SailPoint transform:**
```json
{
  name: "email",
  'type': "concat"
  attributes: {
    values: [
      {
        "type": "identityAttribute"
        "attributes": {
          "name": 'firstname',
        }
      },
      {
        type: "static",
        "attributes": {
          "value": "@company.com"
        }
      },
    ]
  }
}
```

<details>
<summary>Click to reveal fixed version</summary>
```json
{
  "name": "email",
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
}
```

**Errors fixed:**
1. Line 2: Added quotes around "name" key
2. Line 3: Changed single quotes to double on 'type'
3. Line 3: Added missing comma after "concat"
4. Line 4: Added missing comma after "attributes"
5. Line 7: Added missing comma after "identityAttribute"
6. Line 9: Changed single quotes to double on 'firstname'
7. Line 9: Removed trailing comma
8. Line 13: Changed name to "type" (was missing quotes)
9. Line 18: Removed trailing comma after last object
10. Line 19: Removed trailing comma after values array

</details>

---

## Tomorrow: Day 3 Preview

**What You'll Learn:**
```
JSON in SailPoint ISC (Deep Dive)
├─ Complete transform JSON reference
├─ Identity Profile JSON structure
├─ Source configuration JSON
├─ Reading exported configurations
└─ Modifying configurations via JSON
```

**You'll be able to:**
- Export any ISC configuration as JSON
- Modify configurations in a text editor
- Import modified configurations
- Understand complete ISC JSON structure
- Use JSON for version control

---

## Quick Reference: Writing Checklist

**Use this every time you write JSON:**
```
Before you start:
□ Do I need an object { } or array [ ]?
□ What properties/items do I need?

While writing:
□ Are all keys in "double quotes"?
□ Are all string values in "double quotes"?
□ Am I using : between key and value?
□ Am I using , between items?
□ Did I remove trailing commas?

After writing:
□ Does every { have a matching }?
□ Does every [ have a matching ]?
□ Did I validate in JSONLint?
□ Did I format in VS Code?
□ Is it readable (proper indentation)?
```

---

**Congratulations on completing Day 2! You can now WRITE JSON like a pro!**

**Tomorrow we dive deep into SailPoint ISC JSON structures!**

---

*End of JSON Mastery - Day 2*
