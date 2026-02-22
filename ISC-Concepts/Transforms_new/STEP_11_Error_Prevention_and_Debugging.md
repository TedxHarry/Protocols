# STEP 11: Error Prevention & Debugging Guide
## How to Prevent, Detect, and Fix Transform Errors

---

## What You'll Master
- ✅ Common transform errors and their causes
- ✅ How to prevent errors before they happen
- ✅ How to detect when transforms fail
- ✅ Step-by-step debugging approach
- ✅ Common error messages and solutions

---

## THE 5 CATEGORIES OF TRANSFORM ERRORS

### Category 1: NULL Errors - When Data is Missing

**What it is:**
Data you're trying to use is NULL (doesn't exist).

**Common Causes:**
- Aggregation: Source system doesn't have the data
- Provisioning: Identity doesn't have the required attribute
- Lookup: Key doesn't exist in the map
- FirstValid: All sources are NULL

**Examples:**
- Transform tries to use `$firstName` but firstName is NULL
- Transform looks up code "ZZZ" in a map, but key doesn't exist
- Aggregation from source that has NULL email

**How It Fails:**
```
❌ Transform: $firstName.length()
❌ Input: firstName is NULL
❌ Error: "Cannot call method on null"
```

---

### Category 2: Type Errors - Using Data as Wrong Type

**What it is:**
Treating data as the wrong type.

**Common Causes:**
- Trying math on a string
- Trying string operations on a number
- Comparing incompatible types

**Examples:**
- Math operation on "five" (should be 5)
- Substring on a number value
- Comparing "5" (string) to 5 (number)

**How It Fails:**
```
❌ Transform: $salary > 50000
❌ Input: salary = "not a number"
❌ Error: "Cannot compare string to number"
```

---

### Category 3: Syntax Errors - Bad JSON

**What it is:**
Transform JSON is not valid.

**Common Causes:**
- Missing comma between attributes
- Wrong quotes (single instead of double)
- Mismatched braces
- Invalid Velocity syntax

**Examples:**
- {"name": "Transform" "type": "lower"} - missing comma
- {'name': 'Transform'} - single quotes
- {"name": "Transform" - missing closing brace

**How It Fails:**
```
❌ Error: "Invalid JSON syntax"
❌ Transform doesn't even load in ISC
```

---

### Category 4: Context Errors - Using Wrong Data Source

**What it is:**
Using wrong data source in wrong context.

**Common Causes:**
- Using `identityAttribute` in aggregation (doesn't exist yet)
- Using `accountAttribute` in provisioning (not available)
- Referencing attribute that doesn't exist in that context

**Examples:**
- Aggregation transform tries to use identityAttribute
- Provisioning transform tries to use accountAttribute from source
- Create profile tries to use data that's only in different source

**How It Fails:**
```
❌ Aggregation Transform
❌ Using: identityAttribute (hasn't been created yet)
❌ Error: "Attribute not found"
```

---

### Category 5: Logic Errors - Wrong Outcome

**What it is:**
Transform runs but produces wrong output.

**Common Causes:**
- Wrong comparison operator (> instead of <)
- Wrong conditional logic
- Missing null check before using data
- Case sensitivity mismatch

**Examples:**
- Using == instead of != (opposite condition)
- Comparing "IT" to "it" (case mismatch)
- Not checking for NULL before using string method

**How It Fails:**
```
❌ Transform runs successfully
❌ But output is wrong or unexpected
❌ Example: Should be "ACTIVE", outputs "INACTIVE"
```

---

## ERROR PREVENTION STRATEGIES

### Strategy 1: Always Check for NULL Before Using

#### ❌ WRONG: Assumes data exists
```json
{
  "type": "lower",
  "attributes": {
    "input": {
      "type": "identityAttribute",
      "attributes": { "name": "firstName" }
    }
  }
}
```

**Problem:** If firstName is NULL, transform fails.

---

#### ✅ RIGHT: Use FirstValid with Default
```json
{
  "type": "lower",
  "attributes": {
    "input": {
      "type": "firstValid",
      "attributes": {
        "values": [
          {
            "type": "identityAttribute",
            "attributes": { "name": "firstName" }
          },
          {
            "type": "static",
            "attributes": { "value": "Unknown" }
          }
        ]
      }
    }
  }
}
```

**Benefit:** If firstName is NULL, uses "Unknown" instead.

---

### Strategy 2: Check Before String Operations

#### ❌ WRONG: String method on potentially NULL value
```json
{
  "type": "conditional",
  "attributes": {
    "expression": "$email.contains('@')",
    "positiveCondition": "HAS_EMAIL",
    "negativeCondition": "NO_EMAIL"
  }
}
```

**Problem:** If email is NULL, `.contains()` fails.

---

#### ✅ RIGHT: Check for NULL first
```json
{
  "type": "conditional",
  "attributes": {
    "expression": "$email != null && $email.contains('@')",
    "positiveCondition": "VALID_EMAIL",
    "negativeCondition": "INVALID_EMAIL"
  }
}
```

**Benefit:** Only checks `.contains()` if email is not NULL.

---

### Strategy 3: Validate Data Type Before Using

#### ❌ WRONG: Assumes correct type
```json
{
  "type": "conditional",
  "attributes": {
    "expression": "$salary > 50000",
    "positiveCondition": "HIGH_SALARY",
    "negativeCondition": "LOW_SALARY"
  }
}
```

**Problem:** If salary is "not a number", comparison fails.

---

#### ✅ RIGHT: Use static value or normalize first
```json
{
  "type": "conditional",
  "attributes": {
    "expression": "$salaryCode == 'HIGH'",
    "positiveCondition": "HIGH_SALARY",
    "negativeCondition": "LOW_SALARY"
  }
}
```

**Benefit:** Comparing strings, no type confusion.

---

### Strategy 4: Handle International Characters

#### ❌ WRONG: Assumes ASCII names only
```json
{
  "type": "lower",
  "attributes": {
    "input": {
      "type": "identityAttribute",
      "attributes": { "name": "lastName" }
    }
  }
}
```

**Problem:** International names with accents may fail in some systems.

---

#### ✅ RIGHT: Remove accents first
```json
{
  "type": "lower",
  "attributes": {
    "input": {
      "type": "decomposeDiacriticalMarks",
      "attributes": {
        "input": {
          "type": "identityAttribute",
          "attributes": { "name": "lastName" }
        }
      }
    }
  }
}
```

**Benefit:** José becomes jose, works in all systems.

---

### Strategy 5: Use Defensive Nesting Pattern

#### ✅ RIGHT: Build safely with null checks
```json
{
  "type": "concatenation",
  "attributes": {
    "values": [
      {
        "type": "firstValid",
        "attributes": {
          "values": [
            { "type": "identityAttribute", "attributes": { "name": "firstName" } },
            { "type": "static", "attributes": { "value": "Unknown" } }
          ]
        }
      },
      { "type": "static", "attributes": { "value": "_" } },
      {
        "type": "firstValid",
        "attributes": {
          "values": [
            { "type": "identityAttribute", "attributes": { "name": "lastName" } },
            { "type": "static", "attributes": { "value": "User" } }
          ]
        }
      }
    ]
  }
}
```

**Benefit:** Never returns NULL, always has safe defaults.

---

## DETECTING ERRORS

### Where to Look for Errors

#### 1. ISC Preview (Best for Testing)
```
Go to: Admin → Identity Management → Identity Profiles (or Create Profiles)
Click: Preview
Select: A test user
Look for: Red error messages or unexpected output
```

---

#### 2. ISC Logs
```
Go to: Admin → System → Logs
Search for: Transform errors
Filter by: Date/time of issue
Look for: "Transform failed" or similar messages
```

---

#### 3. Task Report (For Provisioning)
```
Go to: Admin → System → Task Report
Find: Provisioning task that failed
Click: To see details
Look for: Transform-related errors in output
```

---

### Common Error Messages

| Error Message | Cause | Fix |
|---|---|---|
| "Cannot call method on null" | NULL value used | Add NULL check |
| "Invalid JSON syntax" | Bad transform JSON | Validate JSON |
| "Attribute not found" | Wrong data source for context | Use correct context |
| "Type mismatch" | Using wrong data type | Convert type first |
| "NullPointerException" | NULL in unexpected place | Add firstValid fallback |
| "Unknown attribute" | Attribute name typo | Check exact name |

---

## STEP-BY-STEP DEBUGGING

### Scenario: Transform Produces Wrong Output

**Step 1: Isolate the Problem**
- Does it fail completely or just wrong output?
- Does it work in preview but fail in production?
- Does it work for some users but not others?

**Step 2: Check Input Data**
- Is source data what you expect?
- Are there NULL values?
- Are there special characters?

**Step 3: Test Each Operation**
- Test the outer operation
- Test the inner operations
- Test operations individually if possible

**Step 4: Check Error Logs**
- Admin → System → Logs
- Search for your transform name
- Look for error details

**Step 5: Fix and Retest**
- Make changes
- Use Preview to test
- Test with multiple users
- Check logs again

---

## EXAMPLE: DEBUG A FAILING TRANSFORM

### Initial Problem
"Our transforms work in preview but fail in production"

---

### Investigation Steps

**Step 1: Check logs**
```
Admin → System → Logs
Error: "Cannot call method on null"
Location: Email_Standardize transform
```

**Step 2: Review the transform**
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

**Step 3: Identify problem**
- Some users have NULL email
- Transform tries `.lower()` on NULL
- Fails with "Cannot call method on null"

**Step 4: Fix with NULL check**
```json
{
  "type": "lower",
  "attributes": {
    "input": {
      "type": "firstValid",
      "attributes": {
        "values": [
          { "type": "identityAttribute", "attributes": { "name": "email" } },
          { "type": "static", "attributes": { "value": "noemail@company.com" } }
        ]
      }
    }
  }
}
```

**Step 5: Test and verify**
- Test in preview with user who has NULL email
- Verify it uses default: "noemail@company.com"
- Check logs - no more errors
- Deploy to production

---

## DEFENSIVE TRANSFORM CHECKLIST

Before deploying any transform, verify:

- [ ] **NULL Handling**
  - Does it check for NULL before using data?
  - Does it have fallback values?

- [ ] **Type Safety**
  - Are all operations using compatible types?
  - Are numbers used as numbers, not strings?

- [ ] **Syntax**
  - Is JSON valid? (Use JSONLint)
  - Are all braces matched?
  - Are all commas correct?

- [ ] **Context Correctness**
  - Is it in aggregation? Using only accountAttribute?
  - Is it in provisioning? Using only identityAttribute?

- [ ] **Edge Cases**
  - What if data is NULL?
  - What if data is empty string?
  - What if data has special characters?
  - What if data is international?

- [ ] **Testing**
  - Tested in preview?
  - Tested with multiple users?
  - Tested with edge cases?
  - Checked logs for errors?

---

## KEY TAKEAWAYS

1. **Prevent Errors:**
   - Always check for NULL
   - Use FirstValid with defaults
   - Check before string operations
   - Handle international characters

2. **Detect Errors:**
   - Use ISC Preview
   - Check Admin → System → Logs
   - Look for error messages

3. **Debug Systematically:**
   - Isolate the problem
   - Check input data
   - Test operations separately
   - Review logs
   - Fix and retest

4. **Common Errors:**
   - NULL values (use FirstValid)
   - Wrong context (use right data source)
   - Bad JSON (use JSONLint)
   - Type mismatches (convert first)

5. **Always Be Defensive:**
   - Assume data could be missing
   - Assume data could be wrong type
   - Assume data could have special characters
   - Plan for failures

---

## WHAT'S NEXT

Now you've learned:
- ✅ STEP 01-09: Foundation through advanced operations
- ✅ STEP 10: Advanced patterns
- ✅ STEP 11: Error prevention and debugging

Next:
- **STEP 12:** Testing & Validation
- **STEP 13:** Practice Module (10 exercises)

Ready? Move to STEP 12.
