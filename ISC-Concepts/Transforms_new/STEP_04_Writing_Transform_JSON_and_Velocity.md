# STEP 04: Writing Valid Transform JSON & Velocity Basics
## How to Write Correct Transform Syntax

---

## What You'll Master
- ✅ Transform JSON syntax rules
- ✅ How to format valid JSON
- ✅ Velocity expressions ($variables, #conditionals)
- ✅ String interpolation and expressions
- ✅ Common syntax errors and how to fix them

---

## JSON SYNTAX BASICS FOR TRANSFORMS

### The Golden Rules

**Rule 1: Every transform has this structure**
```json
{
  "name": "unique_transform_name",
  "type": "operation_type",
  "attributes": {
    // operation-specific attributes
  }
}
```

**Rule 2: Curly braces { } hold objects**
```json
{
  "name": "MyTransform"        ← This is an object
}
```

**Rule 3: Square brackets [ ] hold lists**
```json
{
  "values": [
    "item1",
    "item2"
  ]
}
```

**Rule 4: Colons : separate keys from values**
```json
{
  "name": "MyTransform"    ← name : value
}
```

**Rule 5: Commas , separate items (but NOT after the last item)**
```json
{
  "name": "Transform1",
  "type": "lower",           ← Last item has NO comma
}
```

---

## COMMON JSON ERRORS

### ❌ Error #1: Missing Comma Between Items
```json
{
  "name": "MyTransform"     ← Missing comma here!
  "type": "lower"
}
```
**Fix:** Add comma after "MyTransform"
```json
{
  "name": "MyTransform",    ← Comma added
  "type": "lower"
}
```

---

### ❌ Error #2: Comma After Last Item
```json
{
  "name": "MyTransform",
  "type": "lower",          ← This comma is wrong!
}
```
**Fix:** Remove comma after last item
```json
{
  "name": "MyTransform",
  "type": "lower"           ← No comma
}
```

---

### ❌ Error #3: Single Quotes Instead of Double Quotes
```json
{
  'name': 'MyTransform'     ← Single quotes are wrong
}
```
**Fix:** Use double quotes
```json
{
  "name": "MyTransform"     ← Double quotes
}
```

---

### ❌ Error #4: Unescaped Special Characters
```json
{
  "value": "John said "hello""     ← Unescaped quotes
}
```
**Fix:** Escape special characters with backslash
```json
{
  "value": "John said \"hello\""   ← Escaped quotes
}
```

---

### ❌ Error #5: Missing Closing Brace
```json
{
  "name": "MyTransform",
  "type": "lower"
← Missing closing brace!
```
**Fix:** Add closing brace
```json
{
  "name": "MyTransform",
  "type": "lower"
}
```

---

## VELOCITY EXPRESSIONS BASICS

Velocity is used in `static` values and conditions. It lets you write expressions.

### Understanding Variables

A variable is data you're referencing. In ISC transforms, variables start with `$`:

**$firstName** = The firstName value from aggregation/provisioning  
**$lastName** = The lastName value  
**$email** = The email value  

### Example: Using Variables in Static Values

```json
{
  "type": "static",
  "attributes": {
    "value": "Hello $firstName"
  }
}
```

**Result:**
- If firstName = "John" → Output: "Hello John"
- If firstName = "Jane" → Output: "Hello Jane"

---

### Velocity Syntax: String Interpolation

#### Simple Variable Replacement
```velocity
$firstName
```
If firstName = "John" → "John"

---

#### Variables in Text
```velocity
My name is $firstName $lastName
```
If firstName = "John" and lastName = "Smith" → "My name is John Smith"

---

#### Velocity Method Calls
```velocity
$firstName.toLowerCase()
```
If firstName = "John" → "john"

---

#### Property Access (Less common in transforms)
```velocity
$user.email
```
Gets the email property of user object

---

### Velocity Syntax: Conditionals (#if)

#### Simple If/Else
```velocity
#if ($status == "Active")
  ACTIVE_USER
#else
  INACTIVE_USER
#end
```

**Execution:**
- If status = "Active" → Output: "ACTIVE_USER"
- If status = "Inactive" → Output: "INACTIVE_USER"

---

#### Nested If/Else
```velocity
#if ($department == "IT")
  IT_EMPLOYEE
#elseif ($department == "Finance")
  FINANCE_EMPLOYEE
#else
  OTHER_EMPLOYEE
#end
```

---

### Velocity Syntax: Common Operations

#### Comparison Operators
```velocity
$status == "Active"         ← Equals
$status != "Inactive"       ← Not equals
$salary > 50000            ← Greater than
$salary < 30000            ← Less than
$salary >= 50000           ← Greater or equal
$salary <= 30000           ← Less or equal
```

---

#### Logical Operators
```velocity
#if ($status == "Active" && $department == "IT")   ← AND
  ACTIVE_IT_USER
#end

#if ($role == "Admin" || $role == "Manager")       ← OR
  LEADERSHIP
#end

#if (!$isContractor)                               ← NOT
  EMPLOYEE
#end
```

---

#### String Methods
```velocity
$email.contains("@")                ← Does it contain?
$email.startsWith("john")           ← Does it start with?
$email.endsWith(".com")             ← Does it end with?
$firstName.length()                 ← What's the length?
$status.equals("Active")            ← Does it equal?
```

---

## PUTTING IT TOGETHER: REAL EXAMPLES

### Example 1: Static Value with Variable Interpolation

```json
{
  "type": "static",
  "attributes": {
    "value": "Employee ID: $employeeId"
  }
}
```

**If employeeId = "12345"**
→ Output: "Employee ID: 12345"

---

### Example 2: Static Value with Conditional

```json
{
  "type": "static",
  "attributes": {
    "value": "#if ($status == 'Active') ACTIVE #else INACTIVE #end"
  }
}
```

**If status = "Active"**
→ Output: "ACTIVE"

**If status = "Inactive"**
→ Output: "INACTIVE"

---

### Example 3: Nested Operations with Velocity

```json
{
  "name": "UserClassification",
  "type": "static",
  "attributes": {
    "value": "#if ($department == 'IT' && $salary > 50000) SENIOR_IT #elseif ($department == 'IT') IT_STAFF #else OTHER #end"
  }
}
```

**If department = "IT" AND salary = 60000**
→ Output: "SENIOR_IT"

**If department = "IT" AND salary = 40000**
→ Output: "IT_STAFF"

**If department = "Finance"**
→ Output: "OTHER"

---

## NESTING OPERATIONS IN JSON

When you use an operation as input to another operation:

```json
{
  "name": "ComplexTransform",
  "type": "lower",
  "attributes": {
    "input": {
      "type": "concatenation",
      "attributes": {
        "values": [
          { "type": "static", "attributes": { "value": "prefix_" } },
          { "type": "identityAttribute", "attributes": { "name": "firstName" } }
        ]
      }
    }
  }
}
```

**How it works:**
1. Inner operation (concatenation): "prefix_" + firstName = "prefix_john"
2. Outer operation (lower): "prefix_john" → "prefix_john" (already lowercase)

---

## VALIDATING YOUR JSON

### Online JSON Validators

Before deploying transforms, validate the JSON:

1. Go to **https://jsonlint.com/**
2. Paste your transform JSON
3. Click **Validate**
4. If it shows "Valid JSON" → Good!
5. If it shows errors → Fix them first

---

### Common Validation Errors

**Error: "Expected ',' or '}'"**
- Reason: Missing comma between items
- Fix: Add comma

**Error: "Unexpected token"**
- Reason: Syntax error somewhere
- Fix: Check for unmatched braces, quotes, etc.

**Error: "String is not properly closed"**
- Reason: Missing closing quote
- Fix: Add closing double quote

---

## FORMATTING BEST PRACTICES

### ✅ GOOD: Properly Formatted
```json
{
  "name": "Email_Standardize",
  "type": "lower",
  "attributes": {
    "input": {
      "type": "identityAttribute",
      "attributes": { "name": "email" }
    }
  }
}
```

---

### ❌ BAD: Hard to Read
```json
{"name":"Email_Standardize","type":"lower","attributes":{"input":{"type":"identityAttribute","attributes":{"name":"email"}}}}
```

**Both are valid JSON, but the first is readable. Use proper indentation!**

---

## VELOCITY QUICK REFERENCE

| What | Velocity Syntax | Example |
|------|-----------------|---------|
| Variable | $variableName | $firstName |
| String in text | Text with $var | "Hello $firstName" |
| Method call | $var.method() | $firstName.toLowerCase() |
| Equals | == | $status == "Active" |
| Not equals | != | $status != "Inactive" |
| Greater than | > | $salary > 50000 |
| Less than | < | $salary < 30000 |
| AND | && | $a == "x" && $b == "y" |
| OR | \|\| | $a == "x" \|\| $a == "y" |
| NOT | ! | !$isContractor |
| If/Else | #if() ... #else ... #end | See examples above |
| String contains | .contains() | $email.contains("@") |
| String starts with | .startsWith() | $email.startsWith("john") |
| String ends with | .endsWith() | $email.endsWith(".com") |
| String length | .length() | $firstName.length() |

---

## KEY TAKEAWAYS

1. **JSON Syntax Matters:**
   - Curly braces { } for objects
   - Square brackets [ ] for lists
   - Commas between items
   - Double quotes for strings
   - No comma after last item

2. **Velocity is Used in:**
   - static values (fixed text with variables)
   - conditional expressions (if/else logic)

3. **Variables Start with $:**
   - $firstName
   - $email
   - $status

4. **Common Velocity:**
   - String interpolation: "Hello $firstName"
   - Conditionals: #if ($x == "y") ... #end
   - Methods: $firstName.toLowerCase()

5. **Always Validate:**
   - Use JSONLint to check syntax
   - Test in ISC Preview before deploying

---

## WHAT'S NEXT

Now that you understand:
- ✅ What transforms are (STEP 01)
- ✅ Where transforms execute (STEP 02)
- ✅ How to read transform JSON (STEP 03)
- ✅ How to write valid JSON with Velocity (STEP 04)

Next you'll learn the actual operations:
- **STEP 05:** String Operations (the tools for text)

Ready? Move to STEP 05.
