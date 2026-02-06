# DAY 1: FOUNDATIONS & BASIC TRANSFORMS - AFTERNOON SESSION

## Table of Contents
- [Overview](#overview)
- [The 7 Core String Transforms](#the-7-core-string-transforms)
- [Transform 1: Static](#transform-1-static-15-minutes)
- [Transform 2: Concatenation](#transform-2-concatenation-30-minutes)
- [Transform 3: Lower](#transform-3-lower-20-minutes)
- [Transform 4: Upper](#transform-4-upper-10-minutes)
- [Transform 5: Trim](#transform-5-trim-15-minutes)
- [Transform 6: Substring](#transform-6-substring-25-minutes)
- [Transform 7: Replace](#transform-7-replace-30-minutes)
- [Day 1 Final Assessment](#day-1-final-assessment)
- [Homework Challenges](#homework-challenges-optional-but-recommended)

---

## Overview

**Session Duration:** 2 hours

**Goal:** Master the 7 fundamental string transforms by building 10+ working examples

**What You'll Build:**
- Static country attribute
- Multiple displayName formats
- Email generator with transform chaining
- Username generators (multiple variations)
- Phone number cleaner
- Special character handlers

By the end of this session, you'll be comfortable with **transform chaining** up to 6 steps!

---

## The 7 Core String Transforms

Today we master these essential transforms:

| Transform | Purpose | Common Use Cases |
|-----------|---------|------------------|
| **Static** | Fixed values | Default values, constants |
| **Concatenation** | Combine strings | Names, emails, descriptions |
| **Substring** | Extract parts | Codes, truncation, parsing |
| **Upper** | UPPERCASE conversion | Codes, standardization |
| **Lower** | lowercase conversion | Emails, usernames |
| **Trim** | Remove whitespace | Data cleanup |
| **Replace** | Swap characters | Special char removal, formatting |

---

## Transform 1: Static (15 minutes)

### What It Does

Outputs a **fixed value**, ignoring all source data

### When to Use

- Set a default value for all identities
- Assign same value to everyone in this profile
- Placeholder values

### Configuration

- Just one input: the static value you want

---

### Hands-On Exercise 1A: Set Country to "United States"

**Scenario:** All employees in this Identity Profile are in the USA. Set their country attribute to "United States" for everyone.

#### Steps
```
1. Go to Identity Profile > Mappings
2. Find the "country" attribute (if it doesn't exist, create it)
3. If "country" doesn't exist:
   - Click "Add New Attribute" at top
   - Name: "Country"
   - Technical Name: "country" (auto-fills)
   - Click "Add"
4. Click on the "country" attribute row
5. In the overlay:
   - Source: Select "Transform"
   - Transform: Select "Static"
   - Value: Type "United States"
6. Click "Preview" to test
7. See that ALL identities show "United States" in preview
8. Click "Save" (don't Apply Changes yet)
```

#### Expected Result
```
Source: (ignored)
Transform: Static
Value: "United States"

Output for ANY identity: "United States"
```

#### Key Learning

Static ignores all source data. Everyone gets the same value.

---

### Hands-On Exercise 1B: Set Default Email Domain

**Scenario:** Create a "domain" attribute that's always "company.com"

#### Steps
```
1. Add new attribute "Domain"
2. Technical name: "domain"
3. Source: Transform
4. Transform: Static
5. Value: "company.com"
6. Save
```

**Result:** Now everyone has `domain = "company.com"`

We'll use this later in chaining exercises!

---

## Transform 2: Concatenation (30 minutes)

### What It Does

Combines multiple strings into one

### When to Use

- Create displayName from firstname + lastname
- Build email from parts
- Combine address fields
- Create descriptions

### Configuration

- Can combine 2+ values
- Can add literal strings (like spaces, punctuation)

---

### Hands-On Exercise 2A: Create Display Name

**Scenario:** Create displayName as "Firstname Lastname"

#### Steps
```
1. Go to Mappings
2. Find "displayName" attribute
3. Click on it
4. Source: Transform
5. Transform: Concatenation
6. Configuration:
   Values to concatenate:
   - Click "Add"
   - Type: Attribute
   - Select: firstname
   - Click "Add" again
   - Type: String
   - Value: " " (single space)
   - Click "Add" again  
   - Type: Attribute
   - Select: lastname
7. Preview with different identities
8. Should see: "John Smith", "Mary Johnson", etc.
9. Save
```

#### Expected Result
```
Source: firstname = "John", lastname = "Smith"
Transform: Concatenation (firstname + " " + lastname)
Output: "John Smith"
```

---

### Hands-On Exercise 2B: Create Formatted Display Name

**Scenario:** Create displayName as "Lastname, Firstname"

#### Steps
```
1. Edit the displayName mapping again
2. Transform: Concatenation
3. Values:
   - lastname
   - ", " (comma and space)
   - firstname
4. Preview
5. Should see: "Smith, John"
6. Save
```

#### Key Learning

Order matters! `firstname + lastname` is different from `lastname + firstname`

---

### Hands-On Exercise 2C: Build Email Address (Without Chaining)

**Scenario:** Create email as "firstname.lastname@company.com"

**Problem:** We need lowercase, but Concatenation doesn't handle that!

**For now, just build the structure:**

#### Steps
```
1. Create/edit "email" attribute
2. Transform: Concatenation
3. Values:
   - firstname
   - "."
   - lastname
   - "@company.com"
4. Preview
```

#### What You'll See
```
Input: firstname = "John", lastname = "Smith"
Output: "John.Smith@company.com"
```

**Issue:** Capital letters! Email should be lowercase.

**We'll fix this with CHAINING in the next exercise!**

---

## Transform 3: Lower (20 minutes)

### What It Does

Converts string to lowercase

### When to Use

- Email addresses (should always be lowercase)
- Usernames
- Normalization for comparisons

---

### Hands-On Exercise 3A: Convert to Lowercase

**First, let's see Lower by itself:**

#### Steps
```
1. Create test attribute "testLower"
2. Source: Transform
3. Transform: Lower
4. Input: Select "lastname" (or any text attribute)
5. Preview
```

#### Result
```
Input: lastname = "Smith"
Output: "smith"

Input: lastname = "O'BRIEN"
Output: "o'brien"
```

Simple! It just lowercases everything.

---

### Hands-On Exercise 3B: Fix Email with Transform Chaining!

Now let's combine Concatenation + Lower:

**This is your first CHAIN!**

#### Steps

> **Note:** UI varies by ISC version. Here are both methods:

**Method A (Newer UI):**
```
1. Click email mapping
2. You'll see current transform: Concatenation
3. Look for "+ Add Transform" or "Chain" button
4. Click it
5. Select "Lower"
6. Configure: Input is the output from previous transform
7. Save
```

**Method B (Alternative):**
```
1. When editing email attribute
2. After configuring Concatenation
3. Look for option to add another transform in sequence
4. Add "Lower" transform
5. It will automatically use previous output as input
```

#### Expected Chain
```
Step 1: Concatenation
  firstname + "." + lastname + "@company.com"
  Output: "John.Smith@company.com"
  ↓
Step 2: Lower
  Input: "John.Smith@company.com"
  Output: "john.smith@company.com"
```

#### Preview Different Identities
```
- "Mary" + "Johnson" --> "mary.johnson@company.com"
- "Christopher" + "Anderson" --> "christopher.anderson@company.com"
```

### Congratulations!

You just built your **first transform chain**!

---

## Transform 4: Upper (10 minutes)

### What It Does

Converts to UPPERCASE

### When to Use

- Department codes
- State abbreviations
- Standardized codes

---

### Hands-On Exercise 4: Create Uppercase Display Name

**Quick exercise:**

#### Steps
```
1. Create attribute "displayNameUpper"
2. Transform chain:
   - Concatenation (firstname + " " + lastname)
   - Upper
3. Preview
```

#### Result
```
"John Smith" --> "JOHN SMITH"
```

Easy! Same concept as Lower, just opposite direction.

---

## Transform 5: Trim (15 minutes)

### What It Does

Removes whitespace from beginning and end

### When to Use

- Source data has extra spaces
- Clean up messy imports
- Before concatenating (avoid double spaces)

### Important Note

Trim removes spaces at **START and END only**, not in the middle!

---

### Hands-On Exercise 5A: Clean Messy Data

**Scenario:** Your HR system exports data with trailing spaces

#### Steps
```
1. Create attribute "cleanFirstname"
2. Transform: Trim
3. Input: firstname
4. Preview with an identity
```

#### What It Does
```
Input: "John   " (trailing spaces)
Output: "John" (spaces removed)

Input: "  Mary" (leading spaces)
Output: "Mary"

Input: "  Bob  " (both sides)
Output: "Bob"

Input: "Van Der Berg" (middle spaces)
Output: "Van Der Berg" (middle spaces stay!)
```

---

### Best Practice Alert

**Always Trim before Concatenating** to avoid issues like:
```
BAD: "John   " + " " + "Smith" = "John     Smith" (extra spaces)
GOOD: Trim("John   ") + " " + "Smith" = "John Smith"
```

---

### Hands-On Exercise 5B: Improve Email with Trim

Let's make our email even better:

#### Updated Email Chain
```
Step 1: Trim
  Input: firstname
  Output: "John" (no spaces)
  ↓
Step 2: Trim
  Input: lastname
  Output: "Smith" (no spaces)
  ↓
Step 3: Concatenation
  firstname + "." + lastname + "@company.com"
  Output: "John.Smith@company.com"
  ↓
Step 4: Lower
  Output: "john.smith@company.com"
```

#### Why This Matters

If source has `"John "` (with space), without Trim you get:
```
"John " + "." + "Smith" = "John .Smith" (space before period!)
```

**With Trim:**
```
Trim("John ") = "John"
"John" + "." + "Smith" = "John.Smith" ✓
```

---

## Transform 6: Substring (25 minutes)

### What It Does

Extracts part of a string

### Configuration

- **Begin Index:** Where to start (0 = first character)
- **End Index (optional):** Where to end
- OR **Length:** How many characters to extract

### Important Note

String indexes start at 0!
```
"John"
 0123
 
J = position 0
o = position 1
h = position 2
n = position 3
```

---

### Hands-On Exercise 6A: Extract Department Code

**Scenario:** department field has "ENG-Engineering-USA" but you only want "ENG"

#### Steps
```
1. Create attribute "deptCode"
2. Transform: Substring
3. Input: department
4. Begin Index: 0
5. Length: 3 (or End Index: 3)
6. Preview
```

#### Result
```
Input: "ENG-Engineering-USA"
Output: "ENG"

Input: "FIN-Finance-USA"
Output: "FIN"
```

---

### Hands-On Exercise 6B: Create Username from Name

**Scenario:** Username = first letter of firstname + lastname

#### Steps
```
1. Create attribute "username"
2. Transform chain:

   Step 1: Substring
     Input: firstname
     Begin Index: 0
     Length: 1
     Output: "J" (from "John")
   
   Step 2: Concatenation
     Values: 
       - Result from Step 1 ("J")
       - lastname ("Smith")
     Output: "JSmith"
   
   Step 3: Lower
     Input: "JSmith"
     Output: "jsmith"
```

#### Result
```
"John" + "Smith" --> "jsmith"
"Mary" + "Johnson" --> "mjohnson"
"Christopher" + "Anderson" --> "canderson"
```

**This is a 3-step chain!** Well done!

---

### Hands-On Exercise 6C: Truncate Long Values

**Scenario:** Some systems have username limits (max 8 characters)

#### Steps
```
1. Edit the username mapping
2. Add one more transform to the chain:

   Step 4: Substring
     Input: output from Step 3
     Begin Index: 0
     Length: 8
```

#### Result
```
"jsmith" --> "jsmith" (6 chars, no change)
"canderson" --> "canders" (truncated to 8)
"mvanderbilt" --> "mvanderb" (truncated to 8)
```

---

### Key Learning

Substring is great for:
- Extracting parts (first 3 chars, last 4 chars)
- Truncating to max length
- Getting specific portions (area code from phone number)

---

## Transform 7: Replace (30 minutes)

### What It Does

Finds and replaces characters or strings

### Configuration

- **Find:** What to look for
- **Replace with:** What to substitute

### Use Cases

- Remove special characters
- Fix formatting
- Standardize data

---

### Hands-On Exercise 7A: Clean Phone Numbers

**Scenario:** Phone numbers come as "(214) 555-1234" but you want "2145551234"

#### Steps
```
1. Create attribute "cleanPhone"
2. Transform chain:
   
   Step 1: Replace
     Input: phoneNumber
     Find: "("
     Replace: "" (empty string)
     Output: "214) 555-1234"
   
   Step 2: Replace
     Input: output from Step 1
     Find: ")"
     Replace: ""
     Output: "214 555-1234"
   
   Step 3: Replace
     Input: output from Step 2
     Find: " " (space)
     Replace: ""
     Output: "214555-1234"
   
   Step 4: Replace
     Input: output from Step 3
     Find: "-"
     Replace: ""
     Output: "2145551234"
```

**This is a 4-STEP CHAIN!**

#### Result
```
Input: "(214) 555-1234"
Output: "2145551234"
```

#### Important Note

You need **SEPARATE Replace transforms** for each character!

Can't do multiple replacements in one transform.

---

### Hands-On Exercise 7B: Remove Special Characters from Names

**Scenario:** Names have hyphens and apostrophes: "Mary-Ann O'Brien"  
For username, you want: "maryannobrien"

#### Steps
```
1. Create attribute "cleanUsername"
2. Transform chain:
   
   Step 1: Concatenation
     firstname + "." + lastname
     Output: "Mary-Ann.O'Brien"
   
   Step 2: Replace "-" with ""
     Output: "MaryAnn.O'Brien"
   
   Step 3: Replace "'" with ""
     Output: "MaryAnn.OBrien"
   
   Step 4: Replace " " with ""
     Output: "MaryAnn.OBrien"
   
   Step 5: Replace "." with ""
     Output: "MaryAnnOBrien"
   
   Step 6: Lower
     Output: "maryannobrien"
```

**This is a 6-STEP CHAIN!**

#### Test Cases
```
"Mary-Ann" + "O'Brien" --> "maryannobrien"
"François" + "Müller" --> "françoismuller" (accents stay!)
```

---

## Day 1 Final Assessment

### What You Built Today

Count them:

- [ ] Static country attribute
- [ ] Concatenated display name (2 versions)
- [ ] Email generator with chaining (Concatenate + Lower)
- [ ] Uppercase display name
- [ ] Trimmed firstname
- [ ] Department code extractor (Substring)
- [ ] Username generator (3-step chain)
- [ ] Truncated username (4-step chain)
- [ ] Clean phone number (4-step chain)
- [ ] Clean username with special char removal (6-step chain)

**Total: 10+ working transforms!**

---

### Concepts Mastered

- ✅ What transforms are and why they exist  
- ✅ Transform vs mapping vs rule  
- ✅ When transforms execute  
- ✅ Input/output model  
- ✅ How to navigate Identity Profile mappings  
- ✅ How to use Identity Preview  
- ✅ Static transform  
- ✅ Concatenation  
- ✅ Upper/Lower  
- ✅ Trim  
- ✅ Substring  
- ✅ Replace  
- ✅ **TRANSFORM CHAINING** (up to 6 steps!)  

---

### Final Quiz - Test Yourself!

#### Question 1: Build This Transform

Create email from:
```
firstname = "John"
lastname = "Smith"

Requirements:
- Format: firstname.lastname@company.com
- Lowercase
- Remove spaces if any
- Max 50 characters

What's your transform chain?
```

<details>
<summary>Click to reveal answer</summary>
```
1. Trim (firstname)
2. Trim (lastname)
3. Concatenation (firstname + "." + lastname + "@company.com")
4. Replace (" " with "")
5. Lower
6. Substring (0, 50)

Output: "john.smith@company.com"
```

</details>

---

#### Question 2: What's Wrong With This?
```
Input: firstname = "John   " (trailing spaces)

Transform: Concatenation (firstname + "." + lastname)

Output: "John   .Smith"
```

<details>
<summary>Click to reveal answer</summary>

**Missing Trim!** Should be:
```
1. Trim (firstname)
2. Trim (lastname)
3. Concatenation

Output: "John.Smith"
```

</details>

---

#### Question 3: Build Username
```
firstname = "Christopher"
lastname = "Anderson"

Requirements:
- First letter + lastname
- Lowercase
- Max 8 characters

Transform chain?
```

<details>
<summary>Click to reveal answer</summary>
```
1. Substring (firstname, 0, 1) --> "C"
2. Concatenation ("C" + lastname) --> "CAnderson"
3. Lower --> "canderson"
4. Substring (0, 8) --> "canders"
```

</details>

---

## Homework Challenges (Optional but Recommended)

Before Day 2, try these challenges:

### Challenge 1: Custom Display Name

Build a display name in this format:
```
"LASTNAME, Firstname"
Example: "SMITH, John"
```

<details>
<summary>Hint</summary>

You'll need:
- Upper (for lastname)
- Concatenation
- No change to firstname (already proper case)

</details>

---

### Challenge 2: Short Username

Create a username that:
```
- Uses first 3 letters of firstname + first 3 of lastname
- Lowercase
- Example: "John Smith" --> "johsmi"
```

<details>
<summary>Hint</summary>

You'll need:
- Substring (firstname, 0, 3)
- Substring (lastname, 0, 3)
- Concatenation
- Lower

</details>

---

### Challenge 3: International Phone

Clean this phone number:
```
Input: "+1 (214) 555-1234"
Output: "12145551234"
(Keep the country code but remove all formatting)
```

<details>
<summary>Hint</summary>

You'll need multiple Replace transforms:
- Replace "+"
- Replace "("
- Replace ")"
- Replace " "
- Replace "-"

Or: Replace "+" with empty, then use your existing phone cleaning chain

</details>

---

## Day 1 Complete!

### You've Achieved

- ✅ Understand 7 core string transforms
- ✅ Built 10+ working examples
- ✅ Mastered transform chaining (up to 6 steps!)
- ✅ Used Identity Preview to test
- ✅ Ready for Day 2: Conditional Logic!

---

## What's Next: Day 2 Preview

Tomorrow we tackle:

### Conditional Transforms (If-Then-Else Logic)

- **Conditional** - If-then-else decision making
- **FirstValid** - Fallback chains for null handling
- **Complex nested conditions**
- **Multi-level logic**

### Real-World Scenarios

- Smart email generators with fallbacks
- Lifecycle state calculations
- Manager email resolution
- Attribute validation and defaults

**You'll build transforms that make intelligent decisions based on data!**

---

## Additional Practice Tips

### Use Identity Preview Extensively

- Test every transform immediately
- Try with 5+ different identities
- Look for edge cases (nulls, special chars, long values)

### Document Your Chains

Keep notes like:
```
Email Generator v2:
1. Trim firstname
2. Trim lastname  
3. Concatenate (firstname.lastname@company.com)
4. Replace spaces
5. Lower
6. Substring (0, 50)

Handles: trailing spaces, mixed case, very long names
```

### Break Complex Chains Into Steps

Don't try to build a 6-step chain all at once:

1. Build step 1, test it
2. Add step 2, test it
3. Add step 3, test it
4. Continue incrementally

### Common Mistakes to Avoid

| Mistake | Impact | Solution |
|---------|--------|----------|
| Forgetting Trim | Extra spaces in output | Always Trim before Concatenate |
| Wrong Substring index | Extract wrong characters | Remember: starts at 0! |
| Missing Lower on email | Uppercase in email address | Always lowercase emails |
| Not testing with nulls | Transforms fail on missing data | Test with incomplete data |
| Too many steps | Hard to debug | Keep chains under 8 steps if possible |

---

## Need Help?

### Debugging Checklist

When a transform doesn't work:

1. ✓ Check Identity Preview - what does it show?
2. ✓ Verify source attribute has data
3. ✓ Test each step in chain individually
4. ✓ Check for null values
5. ✓ Verify transform configuration (indexes, values)
6. ✓ Test with different identities

### Resources

- [SailPoint Transforms Documentation](https://documentation.sailpoint.com/saas/help/transforms/index.html)
- [SailPoint Developer Community](https://developer.sailpoint.com/)
- Your sandbox environment for practice

---

**Congratulations on completing Day 1! You're well on your way to transform mastery!**

---

*Part of the SailPoint ISC Transforms Mastery 7-Day Study Plan*
