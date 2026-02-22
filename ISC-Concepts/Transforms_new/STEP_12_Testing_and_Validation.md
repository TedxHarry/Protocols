# STEP 12: Testing & Validation Guide
## How to Verify Transforms Work Before Production

---

## What You'll Master
- ✅ How to test transforms before deploying
- ✅ Test data preparation
- ✅ Validation techniques
- ✅ Common validation mistakes
- ✅ Sign-off checklist before production

---

## THE THREE TESTING STAGES

### Stage 1: Syntax Validation (Before Anything Else)

**What it is:** Verify the JSON is valid before even trying to use it.

**Why it matters:** Bad JSON won't load in ISC at all.

**How to do it:**

1. **Use an online JSON validator**
   - Go to: https://jsonlint.com/
   - Paste your transform JSON
   - Click "Validate"
   - Look for "Valid JSON" message

2. **What validation checks**
   - Matching braces { }
   - Correct commas
   - Proper quotes " "
   - Proper structure

**Example - Valid JSON:**
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

**Status:** ✅ Valid JSON

---

**Example - Invalid JSON:**
```json
{
  "name": "Email_Standardize",
  "type": "lower"    ← Missing comma!
  "attributes": { }
}
```

**Status:** ❌ Invalid JSON - Error: Expected ',' or '}'

---

### Stage 2: Preview Testing (In ISC Interface)

**What it is:** Test the transform in ISC using real data from a test user.

**Why it matters:** 
- Verify it loads
- Verify it produces correct output
- Verify no runtime errors

**How to do it:**

#### For Aggregation (Identity Profile Mapping)

1. **Navigate:**
   ```
   Admin → Identity Management → Identity Profiles
   ```

2. **Select your profile:**
   - Choose the source (e.g., Workday)

3. **Go to Mapping tab:**
   - You see attribute mappings
   - Finds the attribute with your transform

4. **Click Preview:**
   ```
   [Preview] button at top right
   ```

5. **Select a test user:**
   - Click "Select User"
   - Pick a user from source system
   - Click "Preview"

6. **Check the output:**
   ```
   Source Data    →    Transform Applied    →    Result
   firstName: "JOHN" → lower()            → "john"
   email: "john smith@company.com" → trim+lower → "johnsmith@company.com"
   ```

7. **Look for issues:**
   - ❌ Red error message?
   - ❌ Unexpected output?
   - ✅ Output looks correct?

---

#### For Provisioning (Create Profile)

1. **Navigate:**
   ```
   Admin → Identity Management → Create Profiles
   ```

2. **Select your create profile:**
   - Choose the target system (e.g., Active Directory)

3. **Click Preview:**
   ```
   [Preview] button at top right
   ```

4. **Select a test user:**
   - Click "Select User"
   - Pick a user from ISC
   - Click "Preview"

5. **Check the output:**
   ```
   Identity Data    →    Transform Applied    →    Sent to Target
   email: "john@company.com" → lower() → "john@company.com"
   username: "john.smith"  → substring(0,15) → "john.smith"
   ```

6. **Verify correctness:**
   - Does output match target requirements?
   - Are special characters handled?
   - Is length within limits?

---

### Stage 3: Production Validation (After Deployment)

**What it is:** Monitor actual production runs for issues.

**Why it matters:** Real data might behave differently than test data.

**How to do it:**

1. **Monitor logs:**
   ```
   Admin → System → Logs
   Search for: Transform name
   Look for: ERROR or WARNING
   ```

2. **Check task reports:**
   ```
   Admin → System → Task Report
   Find: Relevant provisioning tasks
   Look for: Failures related to transforms
   ```

3. **Spot check results:**
   - Pick random users
   - Verify transforms produced correct output
   - Check target system has correct data

---

## TEST DATA PREPARATION

### Creating Good Test Data

**For Aggregation Testing:**

You need test users in your source system with:

| Scenario | Test User Data | Why Test It |
|----------|---|---|
| Normal case | firstName: "John", lastName: "Smith" | Verify basic operation |
| NULL values | firstName: NULL | Verify null handling |
| Special chars | firstName: "José" | Verify unicode handling |
| Edge case 1 | firstName: "A" | Minimum length |
| Edge case 2 | firstName: "This is a very long name with many characters" | Maximum length |
| Mixed case | firstName: "JoHn" | Case conversion |
| Spaces | firstName: "  john  " | Whitespace handling |
| International | firstName: "François", lastName: "Müller" | Accent handling |

---

**For Provisioning Testing:**

You need test users in ISC with:

| Scenario | User Data | Why Test It |
|----------|---|---|
| Complete | All attributes populated | Normal operation |
| Missing | Some attributes NULL | NULL handling |
| Special format | Email has + sign | Special character handling |
| International | Name with accents | Unicode handling |
| Boundary | Extremely long values | Length limits |
| Edge case | Single character names | Minimum values |

---

## VALIDATION TECHNIQUES

### Technique 1: Output Verification

**Check that output matches expectations:**

```
Transform: lower()
Input:     "JohnSmith"
Expected:  "johnsmith"
Actual:    "johnsmith"
Status:    ✅ PASS
```

```
Transform: email extraction
Input:     "john.smith@company.com"
Expected:  "john.smith"
Actual:    "john.smith"
Status:    ✅ PASS
```

---

### Technique 2: Type Verification

**Check that output is correct data type:**

```
Transform: concatenation
Input:     firstName="John", lastName="Smith"
Expected:  String "JohnSmith"
Actual:    String "JohnSmith"
Status:    ✅ Correct type
```

---

### Technique 3: Length Verification

**Check that output doesn't exceed limits:**

```
Transform: username generation
Input:     firstName="John", lastName="VeryLongLastName"
Expected:  Max 20 chars
Actual:    "JohnVeryLongLastName" (20 chars)
Status:    ✅ Within limit
```

---

### Technique 4: Format Verification

**Check that output matches required format:**

```
Transform: email standardization
Input:     "JOHN.SMITH@COMPANY.COM"
Expected:  Lowercase, valid email format
Actual:    "john.smith@company.com"
Status:    ✅ Correct format
```

---

### Technique 5: Null Handling Verification

**Check that transform handles NULL gracefully:**

```
Transform: firstName (with FirstValid + default)
Input:     firstName=NULL
Expected:  "Unknown" (default value)
Actual:    "Unknown"
Status:    ✅ Null handled correctly
```

---

## COMMON VALIDATION MISTAKES

### ❌ Mistake 1: Testing Only Happy Path

**Wrong approach:**
- Only test normal cases
- Never test NULL values
- Never test edge cases
- Never test special characters

**Correct approach:**
- Test normal case ✅
- Test NULL values ✅
- Test edge cases ✅
- Test special characters ✅
- Test boundary conditions ✅

---

### ❌ Mistake 2: Not Testing Across Contexts

**Wrong approach:**
- Test transform in one source
- Assume it works everywhere
- Don't test with different data types

**Correct approach:**
- Test with multiple users ✅
- Test with different data sources ✅
- Test with different data types ✅
- Test with real production data patterns ✅

---

### ❌ Mistake 3: Testing Only in Preview

**Wrong approach:**
- Test in preview
- Deploy immediately
- Assume production will work same as preview

**Correct approach:**
- Test in preview ✅
- Deploy to test environment ✅
- Run actual aggregation/provisioning ✅
- Check logs for errors ✅
- Then deploy to production ✅

---

### ❌ Mistake 4: Skipping Target Validation

**Wrong approach:**
- Test transform output
- Don't verify target system accepts it

**Correct approach:**
- Test transform output ✅
- Check target system requirements ✅
- Verify target accepts the format ✅
- Verify special characters are handled ✅
- Verify length limits respected ✅

---

### ❌ Mistake 5: Not Documenting Tests

**Wrong approach:**
- Test mentally, don't write it down
- "I tested it and it works"
- Can't reproduce what was tested

**Correct approach:**
- Document test cases ✅
- Document expected results ✅
- Document actual results ✅
- Keep proof of testing ✅
- Sign off on testing ✅

---

## TESTING CHECKLIST

### Before Deploying Any Transform

- [ ] **JSON Validation**
  - [ ] JSON is valid (use JSONLint)
  - [ ] All braces matched
  - [ ] All commas correct
  - [ ] All quotes proper

- [ ] **Syntax & Logic**
  - [ ] Operation type is correct
  - [ ] All attributes are spelled correctly
  - [ ] Nesting is correct
  - [ ] Context is correct (aggregation vs provisioning)

- [ ] **Data Testing**
  - [ ] Tested with normal data
  - [ ] Tested with NULL values
  - [ ] Tested with special characters
  - [ ] Tested with international characters
  - [ ] Tested with boundary values (very short, very long)

- [ ] **Output Validation**
  - [ ] Output type is correct (string, number, etc.)
  - [ ] Output format is correct
  - [ ] Output length is within limits
  - [ ] Output matches target requirements

- [ ] **Error Handling**
  - [ ] NULL values handled gracefully
  - [ ] No uncaught exceptions
  - [ ] Error messages are clear (if any)

- [ ] **Performance**
  - [ ] Transform completes in reasonable time
  - [ ] No noticeable delays
  - [ ] Doesn't cause timeouts

- [ ] **Logging**
  - [ ] Checked logs for errors
  - [ ] No warning messages
  - [ ] No unexpected behavior

- [ ] **Documentation**
  - [ ] Documented what was tested
  - [ ] Documented expected results
  - [ ] Documented actual results
  - [ ] Documented any issues found and fixed

---

## SIGN-OFF BEFORE PRODUCTION

**Template for sign-off:**

```
Transform Name: Email_Standardize
Created By: [Your Name]
Date: [Date]

Testing Summary:
✅ Syntax validated (JSONLint)
✅ Preview tested with 5 test users
✅ Tested with NULL values
✅ Tested with special characters
✅ Output format verified
✅ Logs checked for errors
✅ No issues found

Ready for Production: YES
Approved By: [Manager Name]
Deployment Date: [Date]
```

---

## MONITORING IN PRODUCTION

### What to Monitor

**Daily checks:**
- [ ] Check logs for transform errors
- [ ] Check task reports for failures
- [ ] Spot-check target system data quality

**Weekly checks:**
- [ ] Review error trends
- [ ] Check for performance issues
- [ ] Verify data completeness

**Monthly checks:**
- [ ] Full data quality audit
- [ ] Compare expected vs actual counts
- [ ] Review any schema changes needed

---

## IF ERRORS FOUND IN PRODUCTION

**Immediate Actions:**
1. Disable the transform if safe to do
2. Note what users/data are affected
3. Check logs for exact error
4. Document the issue

**Fix Process:**
1. Reproduce in test environment
2. Fix the transform
3. Test thoroughly
4. Deploy fix to production

**Verification:**
1. Check logs show no more errors
2. Spot-check affected data
3. Document fix and results

---

## REAL EXAMPLE: COMPLETE TEST PLAN

### Transform: Email_Standardize

**Test Case 1: Normal Case**
```
Input:  email = "John.Smith@Company.Com"
Expected: "john.smith@company.com"
Actual: "john.smith@company.com"
Status: ✅ PASS
```

**Test Case 2: Already Lowercase**
```
Input:  email = "john.smith@company.com"
Expected: "john.smith@company.com"
Actual: "john.smith@company.com"
Status: ✅ PASS
```

**Test Case 3: With Spaces**
```
Input:  email = " john.smith@company.com "
Expected: "john.smith@company.com"
Actual: "john.smith@company.com"
Status: ✅ PASS
```

**Test Case 4: NULL Value**
```
Input:  email = NULL
Expected: "noemail@company.com" (default)
Actual: "noemail@company.com"
Status: ✅ PASS
```

**Test Case 5: Special Characters**
```
Input:  email = "josé.garcía@company.com"
Expected: "josé.garcía@company.com" (or "jose.garcia")
Actual: "josé.garcía@company.com"
Status: ✅ PASS
```

**Overall Status:** ✅ ALL TESTS PASSED - READY FOR PRODUCTION

---

## KEY TAKEAWAYS

1. **Three Stages of Testing:**
   - Syntax validation (JSONLint)
   - Preview testing (ISC interface)
   - Production monitoring (logs)

2. **Test Data Preparation:**
   - Normal cases
   - NULL values
   - Special characters
   - Edge cases
   - International data

3. **Validation Techniques:**
   - Output verification
   - Type verification
   - Length verification
   - Format verification
   - Null handling verification

4. **Common Mistakes to Avoid:**
   - Only testing happy path
   - Not testing edge cases
   - Skipping production validation
   - Not documenting tests
   - Deploying without approval

5. **Always Sign Off:**
   - Document what was tested
   - Verify all tests passed
   - Get approval before production
   - Monitor after deployment

---

## WHAT'S NEXT

You've now learned:
- ✅ STEP 01-11: Everything from basics to error prevention
- ✅ STEP 12: Testing and validation

Next:
- **STEP 13:** Practice Module - 10 hands-on exercises
- Then: Build real transforms in your ISC environment

Ready? Move to STEP 13.
