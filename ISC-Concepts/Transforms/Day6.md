# DAY 6: TROUBLESHOOTING & DEBUGGING TRANSFORMS

## Table of Contents
- [Overview](#overview)
- [Part 1: Identity Preview Mastery](#part-1-identity-preview-mastery-45-minutes)
- [Part 2: Common Error Patterns](#part-2-common-error-patterns-90-minutes)
- [Part 3: Troubleshooting Playbook](#part-3-troubleshooting-playbook-60-minutes)
- [Part 4: Testing Strategies](#part-4-testing-strategies-60-minutes)
- [Part 5: Production Validation](#part-5-production-validation-45-minutes)
- [Day 6 Final Assessment](#day-6-final-assessment)

---

## Overview

**Session Duration:** Full Day (6 hours)

**Prerequisites:** 
- Completed Days 1-5
- Built multiple transform chains
- Understand all transform types
- Have access to ISC sandbox

**Goal:** Master the art of debugging, troubleshooting, and validating transforms in production

---

## Part 1: Identity Preview Mastery (45 minutes)

### Understanding Identity Preview

**Identity Preview** is your PRIMARY debugging tool. It shows you:
- Source attribute values (raw data from sources)
- Transform execution at each step
- Output values after each transform
- Final attribute values

---

### How to Access Identity Preview

#### UI Steps
```
1. Navigate to: Admin > Identities > Identity Profiles
2. Click on the Identity Profile you're working with
3. Click "Mappings" in left sidebar
4. At the top, you'll see identity preview dropdown
5. Select an identity from the dropdown
6. The preview loads below the mappings table
```

---

### What You See in Identity Preview

#### Example Preview Display
```
Attribute: displayName
├─ Source: Transform
├─ Source Value: 
│  firstname: "John"
│  lastname: "Smith"
├─ Transform Chain:
│  Step 1: Concatenation
│    Input: firstname="John", lastname="Smith"
│    Output: "John Smith"
│  Step 2: Upper
│    Input: "John Smith"
│    Output: "JOHN SMITH"
└─ Final Value: "JOHN SMITH"
```

---

### Using Preview to Debug: Scenario 1

**Problem:** Email not generating correctly

#### UI Investigation Steps
```
1. Select identity with problem
2. Find "email" attribute in mappings
3. Look at preview for this attribute
4. Check each step:

Step 1: Trim firstname
  Source: "John  " (has trailing spaces)
  Output: "John" ✓

Step 2: Trim lastname
  Source: "Smith"
  Output: "Smith" ✓

Step 3: Concatenation
  Input: "John", "Smith"
  Output: "John.Smith" ✓

Step 4: Lower
  Input: "John.Smith"
  Output: "john.smith" ✓

Step 5: Concatenation (add domain)
  Input: "john.smith"
  Output: "john.smith@company.com" ✓

All steps working! Email should be correct.
```

**If email is STILL wrong, the issue is elsewhere:**
- Wrong source attribute selected
- Identity not processing
- Different identity profile applies

---

### Using Preview to Debug: Scenario 2

**Problem:** Lifecycle state stuck on "Active" when should be "Terminated"

#### UI Investigation Steps
```
1. Select terminated identity
2. Find "lifecycleState" attribute
3. Check preview:

Step 1: Check leave_flag
  Source: leave_flag = false
  Condition: leave_flag = true
  Result: FALSE (continue to next)

Step 2: Check hire_date > today
  Source: hireDateISO = "2020-01-15T00:00:00Z"
  Condition: hire_date > today (2024-02-06)
  Result: FALSE (continue to next)

Step 3: Check term_date <= today - 90 days
  Source: termDateISO = null ❌ PROBLEM FOUND!
  Condition: Can't compare null
  Result: FALSE (continue to next)

Final: Returns "Active" (default)

ROOT CAUSE: term_date is null in source!
```

**Solution:**
- Check source system (Workday/HR)
- Verify term_date field is populated
- Check aggregation mapping
- Verify date format conversion

---

### Using Preview to Debug: Scenario 3

**Problem:** Username truncated incorrectly

#### UI Investigation Steps
```
1. Select identity with long name
2. Find "username" attribute
3. Check preview:

Expected: "christopher.ande"
Actual: "christoph"

Step-by-step check:

Step 1: Trim firstname
  Source: "Christopher"
  Output: "Christopher" ✓

Step 2: Trim lastname
  Source: "Anderson"
  Output: "Anderson" ✓

Step 3: Concatenation
  Input: "Christopher", "Anderson"
  Output: "Christopher.Anderson" ✓

Step 4: Lower
  Input: "Christopher.Anderson"
  Output: "christopher.anderson" ✓

Step 5: Substring (0, 20)
  Input: "christopher.anderson" (21 characters)
  Output: "christoph" ❌ PROBLEM!

ROOT CAUSE: Substring end position is 9, not 20!
```

**Solution:**
- Check substring configuration
- Should be: Begin=0, Length=20 (or End=20)
- Currently: Begin=0, End=9
- Fix the substring parameters

---

### Preview Best Practices
```
✅ DO:
- Test with multiple identities (not just one)
- Test with edge cases (nulls, long values, special chars)
- Check EVERY step in chain
- Verify source values first
- Compare expected vs actual at each step

❌ DON'T:
- Assume one identity represents all
- Skip intermediate steps
- Ignore warning messages
- Test only "happy path" data
- Make changes without testing first
```

---

### Preview Limitations

**Identity Preview does NOT show:**
- Real-time source data (uses cached/aggregated data)
- Rules execution details (rules are black boxes)
- Performance metrics
- Historical values

**For these, you need:**
- Source aggregation logs
- Task execution logs
- API queries
- Audit logs

---

## Part 2: Common Error Patterns (90 minutes)

### Error Pattern 1: Null Input to Transform

#### The Problem
```
Transform expects value but receives null
```

#### UI Symptom
```
Attribute: email
Preview shows:
  Step 1: Trim
    Input: firstname = null
    Output: null
  
  Step 2: Concatenation
    Input: null + "." + "Smith"
    Output: ".Smith" ❌ WRONG!
```

#### JSON Structure (Broken)
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
      "type": "concat",
      "attributes": {
        "values": [
          {"type": "reference"},
          {"type": "static", "attributes": {"value": "."}},
          {"type": "identityAttribute", "attributes": {"name": "lastname"}}
        ]
      }
    }
  }
}
```

---

#### Solution: Add Null Check

##### UI Fix
```
Add Conditional BEFORE Trim:

Step 1: Conditional
  Condition: firstname is not null AND lastname is not null
  If True: Continue to Trim
  If False: Return null (or default value)

Step 2: Trim firstname
Step 3: Concatenation
...
```

##### JSON Structure (Fixed)
```json
{
  "type": "conditional",
  "attributes": {
    "expression": "firstname != null && lastname != null",
    "positiveCondition": {
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
              {"type": "reference"},
              {"type": "static", "attributes": {"value": "."}},
              {"type": "identityAttribute", "attributes": {"name": "lastname"}}
            ]
          }
        }
      }
    },
    "negativeCondition": null
  }
}
```

---

### Error Pattern 2: Wrong Data Type (String vs Array)

#### The Problem
```
Transform expects array but receives string (or vice versa)
```

#### UI Symptom
```
Attribute: primaryGroup
Preview shows:
  Step 1: Account Attribute (adGroups)
    Output: ["CN=Admins,OU=Groups", "CN=Developers,OU=Groups"]
  
  Step 2: Lower ❌ ERROR!
    Lower expects STRING, got ARRAY
    Output: ERROR or unexpected result
```

---

#### Solution: Add Index to Extract String

##### UI Fix
```
Step 1: Account Attribute (adGroups)
  Output: Array

Step 2: Index (position 0)
  Input: Array
  Output: "CN=Admins,OU=Groups" (String)

Step 3: Lower
  Input: String
  Output: "cn=admins,ou=groups" ✓
```

##### JSON Structure (Fixed)
```json
{
  "type": "accountAttribute",
  "attributes": {
    "sourceName": "Active Directory",
    "attributeName": "memberOf",
    "next": {
      "type": "index",
      "attributes": {
        "position": 0,
        "next": {
          "type": "lower"
        }
      }
    }
  }
}
```

---

### Error Pattern 3: Date Format Mismatch

#### The Problem
```
Date Compare or Date Math receives wrong format
```

#### UI Symptom
```
Attribute: isPreHire
Preview shows:
  Step 1: Date Compare
    First Date: hire_date = "01/15/2024" ❌ NOT ISO8601!
    Second Date: today = "2024-02-06T00:00:00Z"
    Result: ERROR or wrong comparison (string compare)
```

---

#### Solution: Convert to ISO8601 First

##### UI Fix
```
Helper Attribute: hireDateISO

Step 1: Date Format
  Input: hire_date
  Input Format: MM/dd/yyyy
  Output Format: ISO8601
  Output: "2024-01-15T00:00:00Z"

Then use hireDateISO in comparisons:

Attribute: isPreHire
Step 1: Date Compare
  First Date: hireDateISO (ISO8601 format)
  Second Date: today
  Result: TRUE or FALSE (correct comparison)
```

##### JSON Structure (Fixed)
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

---

### Error Pattern 4: Lookup Miss (Case Sensitivity)

#### The Problem
```
Lookup doesn't find match due to case mismatch
```

#### UI Symptom
```
Lookup Table: DepartmentCodes
  ENG -> Engineering
  FIN -> Finance

Attribute: departmentName
Preview shows:
  Step 1: Lookup
    Input: dept_code = "eng" (lowercase)
    Table: DepartmentCodes
    Output: null (default) ❌ No match found!

Expected: "Engineering"
Got: null
```

---

#### Solution: Normalize Case Before Lookup

##### UI Fix
```
Step 1: Upper
  Input: dept_code = "eng"
  Output: "ENG"

Step 2: Lookup
  Input: "ENG" (now matches table)
  Table: DepartmentCodes
  Output: "Engineering" ✓
```

##### JSON Structure (Fixed)
```json
{
  "type": "upper",
  "attributes": {
    "input": {
      "type": "identityAttribute",
      "attributes": {
        "name": "dept_code"
      }
    },
    "next": {
      "type": "lookup",
      "attributes": {
        "table": {
          "id": "department-codes-table-id"
        },
        "default": "Unknown"
      }
    }
  }
}
```

---

### Error Pattern 5: Index Out of Bounds

#### The Problem
```
Trying to access array position that doesn't exist
```

#### UI Symptom
```
Attribute: secondGroup
Preview shows:
  Step 1: Account Attribute (adGroups)
    Output: ["CN=Admins,OU=Groups"] (only 1 item!)
  
  Step 2: Index (position 1)
    Input: Array with 1 item
    Position: 1 (doesn't exist - array is [0])
    Output: null or ERROR
```

---

#### Solution: Check Array Length or Use FirstValid

##### UI Fix (Option 1: Conditional)
```
Helper: hasMultipleGroups
  Condition: adGroups.length >= 2
  True/False

Attribute: secondGroup
  Conditional:
    If hasMultipleGroups = true
      Then: Index(adGroups, 1)
      Else: null
```

##### UI Fix (Option 2: FirstValid)
```
Attribute: secondGroup
  FirstValid:
    Try: Index(adGroups, 1)
    Fallback: null
  
  (FirstValid returns null if Index fails)
```

> **Note:** Array length checking syntax varies by ISC version. Verify available methods.

---

### Error Pattern 6: Circular Reference

#### The Problem
```
Attribute references itself in transform chain
```

#### UI Symptom
```
Attribute: displayName

Transform: Concatenation
  Values: displayName + " " + title ❌ CIRCULAR!

This creates infinite loop!
```

#### Solution: Use Different Attributes

##### UI Fix
```
Don't do this:
  displayName = displayName + title

Do this instead:
  displayName = firstname + " " + lastname
  displayNameWithTitle = displayName + " - " + title
```

---

### Error Pattern 7: Account Attribute Returns Null

#### The Problem
```
Account Attribute returns null when expected data
```

#### Possible Causes
```
1. Account doesn't exist for this identity
2. Account not correlated
3. Attribute doesn't exist on account
4. Wrong source name
5. Wrong attribute name
6. Account hasn't been aggregated
```

#### UI Debugging Steps
```
1. Check Identity's "Accounts" tab
   - Does AD account show?
   - Is it correlated?

2. Check Account details
   - Does "mail" attribute exist?
   - Does it have a value?

3. Check source configuration
   - Is source name correct? "Active Directory" vs "AD"
   - Is attribute name correct? "mail" vs "email"

4. Check aggregation
   - When was last aggregation?
   - Did it succeed?
   - Are accounts being created?
```

---

#### Solution: Add Fallbacks

##### UI Fix
```
Attribute: email

Transform: FirstValid
  Priority 1: Account Attribute (AD, mail)
  Priority 2: Account Attribute (Workday, email)
  Priority 3: Generated email
  Priority 4: Default "noemail@company.com"

This way, if AD account doesn't exist, falls back to other options
```

---

### Error Pattern 8: Split Returns Unexpected Results

#### The Problem
```
Split doesn't find delimiter, returns single-item array
```

#### UI Symptom
```
Attribute: lastname
Input: "Smith, John"

Expected:
  Split by ","
  Index(1) = " John"
  Trim = "John"

Got:
  Input: "SmithJohn" (no comma in data!)
  Split by "," -> ["SmithJohn"] (single item array)
  Index(1) -> null (position 1 doesn't exist)
```

---

#### Solution: Check for Delimiter First

##### UI Fix
```
Step 1: Conditional
  Condition: full_name contains ","
  If True: Split and extract
  If False: Use full_name as-is (or different logic)

If True path:
  Split by ","
  Index(1)
  Trim

If False path:
  full_name (use as-is)
```

---

### Error Pattern 9: Replace Not Working

#### The Problem
```
Replace transform not removing characters
```

#### UI Symptom
```
Attribute: cleanPhone

Step 1: Replace
  Find: "("
  Replace: ""
  Input: "(214) 555-1234"
  Output: "(214) 555-1234" ❌ Still has parentheses!
```

#### Possible Causes
```
1. Wrong regex syntax (need to escape special chars)
2. Wrong "find" value
3. Replace is case-sensitive
```

---

#### Solution: Escape Special Characters

##### UI Fix
```
Special characters need escaping in regex:
  ( -> \\(
  ) -> \\)
  . -> \\.
  [ -> \\[
  ] -> \\]

Step 1: Replace
  Find: "\\(" (escaped)
  Replace: ""
  Output: "214) 555-1234" ✓

Step 2: Replace
  Find: "\\)" (escaped)
  Replace: ""
  Output: "214 555-1234" ✓
```

> **Note:** Regex syntax requirements vary by ISC version. Some use literal strings, others require regex. Check documentation.

---

### Error Pattern 10: Conditional Always Returns Same Result

#### The Problem
```
Conditional seems to ignore condition
```

#### UI Symptom
```
Attribute: employeeType

Conditional:
  Condition: status = "Active"
  If True: "Employee"
  If False: "Contractor"

Result: ALWAYS returns "Employee" even for status="Inactive"
```

#### Possible Causes
```
1. Wrong operator (= vs ==, depending on syntax)
2. Case sensitivity ("Active" vs "active")
3. Extra whitespace ("Active " with trailing space)
4. Condition syntax error
```

---

#### Solution: Debug Condition

##### UI Fix
```
Step 1: Create helper to see actual value
  Helper: statusTrimmed
    Trim status
    Lower status
    Output: "active" (now we see exact value)

Step 2: Fix conditional
  Condition: statusTrimmed equals "active" (lowercase, no spaces)
  If True: "Employee"
  If False: "Contractor"
```

---

## Part 3: Troubleshooting Playbook (60 minutes)

### Systematic Debugging Process
```
When a transform doesn't work, follow these steps IN ORDER:

1. Verify source data exists
2. Check Identity Preview
3. Test each transform step individually
4. Check for null values
5. Verify data types
6. Check case sensitivity
7. Validate date formats
8. Test with multiple identities
9. Review error logs
10. Simplify and rebuild
```

---

### Step 1: Verify Source Data Exists

#### UI Steps
```
1. Go to the identity's detail page
2. Click "Accounts" tab
3. Find the source account (e.g., Workday, AD)
4. Click on the account
5. View all attributes
6. Verify the source attribute has a value

Example:
  Looking for: firstname
  
  Check Workday account:
    firstname: "John" ✓ EXISTS
  
  If not exists:
    - Check source system
    - Check aggregation mapping
    - Check attribute name spelling
```

---

### Step 2: Check Identity Preview

#### UI Steps
```
1. Go to Identity Profile > Mappings
2. Select the identity
3. Find the problematic attribute
4. Expand transform chain in preview
5. Check EACH step:
   - What's the input?
   - What's the output?
   - Does output match expected?

Look for:
  - Nulls appearing unexpectedly
  - Wrong values
  - Type mismatches (string/array)
  - Errors or warnings
```

---

### Step 3: Test Each Transform Individually

#### UI Steps
```
Isolate the problem:

If you have 10-step chain and step 7 fails:

1. Temporarily remove steps 8-10
2. Test steps 1-7 only
3. If works, issue is in 8-10
4. If still fails, issue is in 1-7

Binary search approach:
  - Test first half
  - Test second half
  - Narrow down to specific step
```

---

### Step 4: Check for Null Values

#### Common Null Scenarios
```
Scenario A: Source attribute is null
  Solution: Add Conditional to check before processing

Scenario B: Transform output is null
  Solution: Add FirstValid with fallback

Scenario C: Referenced attribute is null
  Solution: Use Conditional or provide default

Always ask: "What if this is null?"
```

---

### Step 5: Verify Data Types

#### Type Checking Checklist
```
Transform: Lower
  Expects: String
  Fails if: Array
  Check: Does previous step output string?

Transform: Index
  Expects: Array
  Fails if: String
  Check: Does previous step output array?

Transform: Date Compare
  Expects: ISO8601 date strings
  Fails if: Other date format
  Check: Use Date Format first
```

---

### Step 6: Check Case Sensitivity

#### Case-Sensitive Operations
```
✓ Lookups (table keys)
✓ String comparisons in Conditionals
✓ Attribute names
✓ Source names

✗ Transform types (case doesn't matter)

Fix: Add Upper or Lower before operation
```

---

### Step 7: Validate Date Formats

#### Date Debugging Checklist
```
□ Is input in ISO8601 format?
□ Did I use Date Format to convert?
□ Is timezone component present? (T00:00:00Z)
□ Are date strings, not objects?
□ Is "today" reference working?

Common fix: Add Date Format helper attribute
```

---

### Step 8: Test with Multiple Identities

#### Testing Strategy
```
Don't test with just ONE identity!

Test with:
✓ Perfect data identity (all fields populated)
✓ Missing data identity (nulls, empty strings)
✓ Long name identity (truncation test)
✓ Special character identity (hyphens, apostrophes)
✓ International identity (non-ASCII characters)
✓ Edge case identity (single name, no lastname)

Each identity reveals different issues!
```

---

### Step 9: Review Error Logs

#### Where to Find Logs
```
UI Navigation:
  Admin > System > Task Results

Look for:
  - Identity processing tasks
  - Aggregation tasks
  - Transform errors
  - Validation errors

Filter by:
  - Task type
  - Date range
  - Error status
```

---

### Step 10: Simplify and Rebuild

#### When All Else Fails
```
If transform is too complex to debug:

1. Delete the transform chain
2. Start with Step 1 only
3. Test
4. Add Step 2
5. Test
6. Add Step 3
7. Test
... continue until you find the problem

This is SLOW but GUARANTEED to find the issue
```

---

### Troubleshooting Decision Tree
```
Transform not working?
│
├─ Is source data present?
│  ├─ NO → Check source system, aggregation
│  └─ YES → Continue
│
├─ Does Preview show correct inputs?
│  ├─ NO → Check attribute mapping, source selection
│  └─ YES → Continue
│
├─ Does Step 1 work?
│  ├─ NO → Fix Step 1 (check inputs, nulls, types)
│  └─ YES → Continue
│
├─ Does Step 2 work?
│  ├─ NO → Fix Step 2
│  └─ YES → Continue to Step 3
│
... repeat for each step
│
└─ All steps work but final output wrong?
   → Check if correct attribute being used
   → Check if identity processing ran
   → Check if right identity profile applies
```

---

## Part 4: Testing Strategies (60 minutes)

### Testing Levels
```
Level 1: Unit Testing (individual transforms)
Level 2: Integration Testing (transform chains)
Level 3: Scenario Testing (edge cases)
Level 4: Volume Testing (performance)
Level 5: Production Validation (real data)
```

---

### Level 1: Unit Testing Individual Transforms

#### Test Each Transform Type

**Static Transform Test**
```
UI Test:
  Attribute: testStatic
  Transform: Static
    Value: "TEST"
  
  Expected: "TEST"
  Actual: [Preview shows "TEST"]
  Result: ✓ PASS
```

**Concatenation Test**
```
UI Test:
  Attribute: testConcat
  Transform: Concatenation
    Values: "Hello" + " " + "World"
  
  Expected: "Hello World"
  Actual: [Preview shows "Hello World"]
  Result: ✓ PASS
```

**Conditional Test**
```
UI Test:
  Create test identities:
    Identity 1: status = "Active"
    Identity 2: status = "Inactive"
  
  Attribute: testConditional
  Transform: Conditional
    Condition: status equals "Active"
    True: "ACTIVE"
    False: "INACTIVE"
  
  Identity 1 Expected: "ACTIVE"
  Identity 1 Actual: [Preview shows "ACTIVE"]
  Result: ✓ PASS
  
  Identity 2 Expected: "INACTIVE"
  Identity 2 Actual: [Preview shows "INACTIVE"]
  Result: ✓ PASS
```

---

### Level 2: Integration Testing Transform Chains

#### Test Complete Chains

**Email Generator Chain Test**
```
Test Data:
  firstname: "John"
  lastname: "Smith"

Transform Chain:
  1. Trim firstname
  2. Trim lastname
  3. Concatenation (firstname + "." + lastname)
  4. Lower
  5. Concatenation (result + "@company.com")

Expected Final: "john.smith@company.com"

Test Steps:
  1. Create test identity with data above
  2. View Preview
  3. Check Step 1 output: "John" ✓
  4. Check Step 2 output: "Smith" ✓
  5. Check Step 3 output: "John.Smith" ✓
  6. Check Step 4 output: "john.smith" ✓
  7. Check Step 5 output: "john.smith@company.com" ✓
  
  Result: ✓ PASS
```

---

### Level 3: Scenario Testing Edge Cases

#### Edge Case Test Matrix

**Email Generator Edge Cases**

| Scenario | firstname | lastname | Expected Output | Test Result |
|----------|-----------|----------|----------------|-------------|
| Normal | "John" | "Smith" | "john.smith@company.com" | [ ] PASS/FAIL |
| Trailing spaces | "John  " | "Smith " | "john.smith@company.com" | [ ] PASS/FAIL |
| Hyphens | "Mary-Ann" | "O'Brien" | "maryann.obrien@company.com" | [ ] PASS/FAIL |
| Long names | "Christopher" | "Vanderbilt" | "christopher.vanderbilt@company.com" | [ ] PASS/FAIL |
| Missing lastname | "Madonna" | null | null or "madonna@company.com" | [ ] PASS/FAIL |
| Missing firstname | null | "Prince" | null or "prince@company.com" | [ ] PASS/FAIL |
| Both null | null | null | null or default | [ ] PASS/FAIL |
| Special chars | "François" | "Müller" | "françois.müller@company.com" | [ ] PASS/FAIL |
| Single char | "J" | "S" | "j.s@company.com" | [ ] PASS/FAIL |

**Create test identity for EACH scenario and verify!**

---

### Level 4: Volume Testing Performance

#### Performance Test Strategy
```
Small Scale (100 identities):
  1. Apply transform to 100 identities
  2. Measure execution time
  3. Check for errors
  
  Expected: < 5 minutes
  
Medium Scale (1,000 identities):
  1. Apply to 1,000 identities
  2. Measure time
  3. Check error rate
  
  Expected: < 30 minutes
  
Large Scale (10,000+ identities):
  1. Apply to production volume
  2. Measure time
  3. Monitor system load
  
  Expected: < 2 hours
```

#### Performance Benchmarks
```
Transform Complexity vs Time:

Simple (1-3 steps):
  100 identities: ~1 minute
  1,000 identities: ~10 minutes
  10,000 identities: ~60 minutes

Medium (4-7 steps):
  100 identities: ~2 minutes
  1,000 identities: ~20 minutes
  10,000 identities: ~120 minutes

Complex (8+ steps):
  100 identities: ~5 minutes
  1,000 identities: ~45 minutes
  10,000 identities: ~240 minutes

If you exceed these, optimize!
```

---

### Level 5: Production Validation

#### Pre-Production Checklist
```
Before applying transforms in production:

□ Tested with 10+ diverse identities
□ Tested all edge cases
□ Verified null handling
□ Checked performance with volume
□ Documented expected behavior
□ Created rollback plan
□ Tested in sandbox/dev first
□ Got approval from stakeholders
□ Scheduled maintenance window
□ Communicated to users
```

---

#### Production Validation Steps
```
After applying to production:

Day 1:
  □ Sample 20 random identities
  □ Verify attributes are correct
  □ Check for unexpected nulls
  □ Review error logs

Day 3:
  □ Sample 50 identities
  □ Check edge cases
  □ Monitor helpdesk tickets
  □ Review user complaints

Week 1:
  □ Sample 100 identities
  □ Statistical validation (% correct)
  □ Performance monitoring
  □ Adjust as needed

Month 1:
  □ Full audit
  □ Document issues found
  □ Optimize based on real data
```

---

### Testing Documentation Template

#### Transform Test Plan Template
```markdown
# Transform Test Plan: [Attribute Name]

## Transform Description
- Purpose: [What does this calculate?]
- Input Attributes: [List source attributes]
- Transform Steps: [List each step]
- Expected Output: [Describe expected format/values]

## Unit Tests

### Test 1: [Test Name]
- Input: [Test data]
- Expected: [Expected output]
- Actual: [Actual result from preview]
- Status: PASS/FAIL
- Notes: [Any observations]

### Test 2: [Test Name]
...

## Edge Case Tests

### Test 1: Null Input
- Input: firstname=null, lastname=null
- Expected: null or default
- Actual: 
- Status: 

### Test 2: Special Characters
- Input: firstname="Mary-Ann", lastname="O'Brien"
- Expected: 
- Actual: 
- Status: 

## Performance Test

- Volume: [Number of identities]
- Execution Time: [Minutes]
- Error Count: [Number]
- Performance Rating: GOOD/ACCEPTABLE/POOR

## Production Validation

- Date Deployed: 
- Sample Size: 
- Accuracy Rate: 
- Issues Found: 
- Resolution: 

## Sign-Off

- Tested By: [Name]
- Date: [Date]
- Approved By: [Name]
- Date: [Date]
```

---

## Part 5: Production Validation (45 minutes)

### Pre-Deployment Validation

#### Final Checklist
```
Transform Name: ___________________________
Created By: ___________________________
Date: ___________________________

Technical Validation:
□ Works in sandbox/dev
□ Tested with 20+ identities
□ All edge cases tested
□ Performance acceptable
□ No errors in logs
□ Preview shows correct output
□ JSON/configuration backed up

Business Validation:
□ Meets requirements
□ Approved by business owner
□ Documentation complete
□ Training materials ready (if needed)
□ Communication plan in place

Risk Assessment:
□ Rollback plan documented
□ Impact analysis complete
□ Maintenance window scheduled
□ Support team notified

Ready for Production: YES / NO
```

---

### Deployment Process

#### Recommended Deployment Steps
```
Step 1: Backup Current Configuration
  1. Export current transform (if exists)
  2. Document current behavior
  3. Create restore point

Step 2: Deploy to Production
  1. Apply transform configuration
  2. Do NOT click "Apply Changes" yet!
  3. Test with Preview on 5 identities
  4. Verify looks correct

Step 3: Limited Rollout
  1. Click "Apply Changes"
  2. Wait for processing to complete
  3. Immediately check 20 sample identities
  4. Verify attributes are correct

Step 4: Monitor
  1. Check error logs every hour for 4 hours
  2. Sample 50 identities at end of day
  3. Review helpdesk tickets
  4. Check for anomalies

Step 5: Full Validation
  1. After 24 hours, sample 100 identities
  2. Statistical validation
  3. Document any issues
  4. Adjust if needed
```

---

### Rollback Procedures

#### When to Rollback
```
Rollback immediately if:
❌ More than 5% of identities have errors
❌ Critical business process breaks
❌ Data corruption detected
❌ Unexpected null values widespread
❌ Performance degradation severe
❌ Security issue identified
```

---

#### How to Rollback

##### UI Rollback Steps
```
1. Go to Identity Profile > Mappings
2. Find the problematic attribute
3. Click on attribute mapping
4. Revert to previous configuration:
   
   Option A: Use backed-up configuration
     - Paste previous transform config
     - Save
   
   Option B: Change to direct mapping temporarily
     - Change Source to "Direct"
     - Select source attribute
     - Save
   
   Option C: Set to static default
     - Change to Static transform
     - Enter safe default value
     - Save

5. Click "Apply Changes"
6. Monitor identity processing
7. Verify rollback successful
8. Communicate to stakeholders
```

---

### Post-Deployment Monitoring

#### Daily Monitoring (First Week)
```
Daily Tasks:
□ Review error logs (morning and evening)
□ Sample 20 random identities
□ Check helpdesk ticket queue
□ Monitor provisioning success rate
□ Check for null values in new attribute
□ Verify edge cases still work
□ Document any anomalies

Red Flags:
⚠️ Increase in error rate
⚠️ Null values appearing
⚠️ Helpdesk ticket spike
⚠️ User complaints
⚠️ Provisioning failures
⚠️ Performance degradation
```

---

#### Weekly Monitoring (First Month)
```
Weekly Tasks:
□ Statistical sampling (100 identities)
□ Accuracy rate calculation
□ Performance benchmarking
□ Edge case verification
□ Documentation updates
□ Lessons learned capture

Success Criteria:
✓ >95% accuracy rate
✓ <1% error rate
✓ No critical incidents
✓ Performance within benchmarks
✓ Zero rollbacks needed
✓ Positive user feedback
```

---

### Issue Resolution Process

#### Issue Triage
```
When issue is discovered:

Priority 1 - CRITICAL (fix immediately):
  - Data corruption
  - Security breach
  - System down
  - Widespread failures (>10% identities)
  Action: Rollback immediately, investigate offline

Priority 2 - HIGH (fix within 24 hours):
  - Significant error rate (5-10%)
  - Business process impacted
  - Multiple user complaints
  Action: Create hotfix, test, deploy

Priority 3 - MEDIUM (fix within 1 week):
  - Minor error rate (1-5%)
  - Edge case failures
  - Cosmetic issues
  Action: Schedule fix in next maintenance window

Priority 4 - LOW (fix in next release):
  - Documentation issues
  - Enhancement requests
  - Single-user edge case
  Action: Add to backlog
```

---

#### Root Cause Analysis Template
```markdown
# Incident Report: [Attribute Name Issue]

## Issue Summary
- Date Discovered: 
- Reported By: 
- Severity: CRITICAL / HIGH / MEDIUM / LOW
- Impact: [How many identities? What broke?]

## Root Cause
- Underlying Problem: [What actually caused the issue?]
- Why It Happened: [Configuration error? Data quality? Logic flaw?]
- Why It Wasn't Caught: [Gap in testing? Edge case?]

## Resolution
- Fix Applied: [What did you change?]
- Date Fixed: 
- Verification: [How did you verify it's fixed?]

## Prevention
- Testing Gap: [What test would have caught this?]
- Process Change: [How do we prevent this in future?]
- Documentation Update: [What do we need to document?]

## Lessons Learned
- What went well: 
- What went wrong: 
- What we'll do differently: 
```

---

## Day 6 Final Assessment

### What You Learned Today

- ✅ Identity Preview mastery (primary debugging tool)
- ✅ 10+ common error patterns and solutions
- ✅ Systematic troubleshooting process
- ✅ 5 levels of testing strategy
- ✅ Production validation procedures
- ✅ Deployment best practices
- ✅ Rollback procedures
- ✅ Post-deployment monitoring
- ✅ Issue triage and resolution

---

### Key Takeaways
```
Debugging:
- Preview is your best friend
- Check each step individually
- Test with multiple identities
- Always check for nulls first
- Verify data types match

Testing:
- Unit test individual transforms
- Integration test chains
- Scenario test edge cases
- Volume test performance
- Validate in production

Production:
- Have rollback plan
- Deploy incrementally
- Monitor closely
- Document everything
- Learn from issues
```

---

### Troubleshooting Quick Reference
```
Problem: Transform returns null
→ Check: Source data exists? Nulls handled? Type mismatch?

Problem: Wrong output value
→ Check: Each step in Preview, case sensitivity, date format

Problem: Lookup not working
→ Check: Case match? Value exists in table? Default set?

Problem: Date compare fails
→ Check: ISO8601 format? Date Format transform used?

Problem: Index out of bounds
→ Check: Array has enough items? Use FirstValid for safety

Problem: Performance slow
→ Check: Chain length? Use Lookup vs Conditionals? Helper attributes?

Problem: Conditional always same result
→ Check: Condition syntax? Case? Whitespace? Actual values?

Problem: Account Attribute null
→ Check: Account exists? Correlated? Attribute name? Aggregation?
```

---

## Practice Exercises

### Exercise 1: Debug This Transform
```
Given transform chain (BROKEN):

Attribute: email
  Step 1: Concatenation (firstname + "." + lastname)
  Step 2: Lower
  Step 3: Concatenation (result + "@company.com")

Problem: Returns ".smith@company.com" for some users

Task: Debug and fix
```

<details>
<summary>Click to reveal solution</summary>

**Root Cause:** firstname is null for some users

**Fix:** Add null check
```
Step 1: Conditional
  Condition: firstname != null AND lastname != null
  If True: Continue
  If False: Return null

Step 2: Trim firstname
Step 3: Trim lastname
Step 4: Concatenation
Step 5: Lower
Step 6: Add domain
```

</details>

---

### Exercise 2: Find the Bug
```
Given: lifecycleState always returns "Active" even for terminated users

Transform chain:
  Step 1: Check leave_flag = true → "Leave"
  Step 2: Check hire_date > today → "Pre-hire"  
  Step 3: Check term_date != null → "Terminated"
  Step 4: Else → "Active"

Task: Find the bug
```

<details>
<summary>Click to reveal solution</summary>

**Root Cause:** Step 3 only checks if term_date exists, not if it's in the past!

**Fix:** 
```
Step 3: Check term_date <= today → "Terminated"

Need to add date comparison, not just null check.

Also need to ensure term_date is in ISO8601 format before comparing.
```

</details>

---

### Exercise 3: Performance Issue
```
Transform with 50 nested conditionals runs for 3 hours on 10,000 identities.

Task: Optimize
```

<details>
<summary>Click to reveal solution</summary>

**Root Cause:** 50 nested conditionals is inefficient

**Fix:** Use Lookup table instead
```
Before: 50 conditionals checking dept_code
  Worst case: 50 comparisons per identity
  10,000 identities × 50 = 500,000 operations

After: 1 Lookup transform
  10,000 identities × 1 lookup = 10,000 operations

Performance improvement: 50x faster
```

</details>

---

## Day 6 Complete!

### You've Mastered

- ✅ Identity Preview debugging
- ✅ Common error patterns
- ✅ Systematic troubleshooting
- ✅ Comprehensive testing strategies
- ✅ Production deployment procedures
- ✅ Rollback and recovery
- ✅ Monitoring and validation
- ✅ Issue resolution processes

---

## What's Next: Day 7 Preview

Tomorrow is the CAPSTONE PROJECT:

### Complete Employee Onboarding Transform Suite

**You'll Build:**
- 9 production-ready transforms
- Complete test suite (10+ edge cases)
- Full documentation
- Deployment plan
- Support runbook

**Transforms to Build:**
1. displayName
2. email (with 5 fallbacks)
3. username (handle all edge cases)
4. department (normalized)
5. title (standardized)
6. phone (E.164 format)
7. timezone (from location)
8. lifecycleState (complete logic)
9. managerEmail (5-level fallback)

**This pulls together EVERYTHING from Days 1-6!**

---

**Excellent work on Day 6! You're now a transform debugging expert!**

---

*Part of the SailPoint ISC Transforms Mastery 7-Day Study Plan*
