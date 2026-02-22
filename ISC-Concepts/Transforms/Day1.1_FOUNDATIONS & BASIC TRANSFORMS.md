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
- [Understanding Transform Chaining](#understanding-transform-chaining)
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

#### UI Style: Steps to Configure

```
1. Go to Admin > Identities > Identity Profiles
2. Select your Identity Profile
3. Click "Mappings" tab
4. Find the "country" attribute (if it doesn't exist, create it)
5. If "country" doesn't exist:
   - Click "Add New Attribute" at top
   - Display Name: "Country"
   - Technical Name: "country" (auto-fills)
   - Click "Add"
6. Click on the "country" attribute row to edit
7. In the configuration panel:
   - Source: Select "Transform" from dropdown
   - Transform: Select "Static" from transform list
   - Value: Type "United States"
8. Click "Preview" button to test with sample identities
9. Verify that ALL identities show "United States" in preview
10. Click "Save" (don't Apply Changes yet - we're still configuring)
```

#### JSON Style: Transform Definition

```json
{
  "name": "country",
  "type": "static",
  "attributes": {
    "value": "United States"
  }
}
```

#### Expected Result

```
Source: (ignored - Static ignores all input)
Transform: Static
Value: "United States"

Output for ANY identity: "United States"
```

#### Key Learning

Static ignores all source data. Every single identity gets the same value - no exceptions.

---

### Hands-On Exercise 1B: Set Default Email Domain

**Scenario:** Create a "domain" attribute that's always "company.com"

#### UI Style: Steps

```
1. In Mappings, create new attribute "Domain"
2. Technical name: "domain"
3. Click to edit
4. Source: Transform
5. Transform: Static
6. Value: "company.com"
7. Click Save
```

#### JSON Style: Transform Definition

```json
{
  "name": "domain",
  "type": "static",
  "attributes": {
    "value": "company.com"
  }
}
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

#### UI Style: Steps

```
1. Go to Mappings
2. Find or create "displayName" attribute
3. Click on it to edit
4. Source: Transform
5. Transform: Concatenation
6. In the Configuration panel:
   Values to concatenate:
   - Click "Add Value" button
   - Type: "Attribute"
   - Select: "firstname"
   - Click "Add Value" button again
   - Type: "String"
   - Value: " " (single space) - this is literal text
   - Click "Add Value" button again
   - Type: "Attribute"
   - Select: "lastname"
7. Click "Preview" to test with different identities
8. Verify output shows: "John Smith", "Mary Johnson", etc.
9. Click "Save"
```

#### JSON Style: Transform Definition

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

#### Expected Result

```
Source: firstname = "John", lastname = "Smith"
Transform: Concatenation (firstname + " " + lastname)
Output: "John Smith"
```

---

### Hands-On Exercise 2B: Create Formatted Display Name

**Scenario:** Create displayName as "Lastname, Firstname"

#### UI Style: Steps

```
1. Edit the displayName mapping again
2. Transform: Concatenation
3. Values to concatenate:
   - Add: lastname (attribute)
   - Add: ", " (string literal)
   - Add: firstname (attribute)
4. Click Preview
5. Verify output shows: "Smith, John"
6. Click Save
```

#### JSON Style: Transform Definition

```json
{
  "name": "displayName",
  "type": "concat",
  "attributes": {
    "values": [
      {
        "type": "identityAttribute",
        "attributes": {
          "name": "lastname"
        }
      },
      {
        "type": "static",
        "attributes": {
          "value": ", "
        }
      },
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

#### Key Learning

Order matters! `firstname + lastname` is different from `lastname + firstname`

---

### Hands-On Exercise 2C: Build Email Address (Without Chaining)

**Scenario:** Create email as "firstname.lastname@company.com"

**Problem:** We need lowercase, but Concatenation doesn't handle that!

**For now, just build the structure:**

#### UI Style: Steps

```
1. Create/edit "email" attribute
2. Transform: Concatenation
3. Values to concatenate:
   - Add: firstname (attribute)
   - Add: "." (string)
   - Add: lastname (attribute)
   - Add: "@company.com" (string)
4. Click Preview
```

#### JSON Style: Transform Definition

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

#### What You'll See

```
Input: firstname = "John", lastname = "Smith"
Output: "John.Smith@company.com"
```

**Issue:** Capital letters! Email should be lowercase.

**We'll fix this with CHAINING in the next section!**

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

#### UI Style: Steps

```
1. Create test attribute "testLower"
2. Source: Transform
3. Transform: Lower
4. Input: Select "lastname" (or any text attribute)
5. Click Preview to see results
```

#### JSON Style: Transform Definition

```json
{
  "name": "testLower",
  "type": "lower",
  "attributes": {
    "input": {
      "type": "identityAttribute",
      "attributes": {
        "name": "lastname"
      }
    }
  }
}
```

#### Result

```
Input: lastname = "Smith"
Output: "smith"

Input: lastname = "O'BRIEN"
Output: "o'brien"

Input: lastname = "GARCÍA"
Output: "garcía"
```

Simple! It just lowercases everything.

---

### Hands-On Exercise 3B: Fix Email with Transform Chaining!

**Scenario:** Now we fix the email from 2C to be lowercase

**New email requirement:** "john.smith@company.com" (all lowercase)

#### UI Style: Steps

```
1. Edit the "email" attribute mapping
2. Transform: Lower
3. Input: Instead of selecting an attribute, we select "Transform"
4. Select: Concatenation (the email transform we built in 2C)
5. This chains Lower AFTER Concatenation
6. Click Preview
7. Verify output: "john.smith@company.com" (all lowercase!)
8. Click Save
```

#### JSON Style: Transform Definition

```json
{
  "name": "email",
  "type": "lower",
  "attributes": {
    "input": {
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
  }
}
```

#### Result

```
Input: firstname = "John", lastname = "Smith"
Concatenation Output: "John.Smith@company.com"
Lower Output: "john.smith@company.com" ✓ Perfect!
```

#### Key Learning

**Transform Chaining**: Lower's input is NOT an identity attribute - it's the output of Concatenation!

---

## Transform 4: Upper (10 minutes)

### What It Does

Converts string to UPPERCASE

### When to Use

- Standardizing codes
- Department codes
- Status values

---

### Hands-On Exercise 4A: Uppercase Department Code

**Scenario:** Create a department code attribute that's always uppercase

#### UI Style: Steps

```
1. Create/edit "departmentCode" attribute
2. Source: Transform
3. Transform: Upper
4. Input: Select "department" (identity attribute)
5. Preview to verify uppercase output
6. Save
```

#### JSON Style: Transform Definition

```json
{
  "name": "departmentCode",
  "type": "upper",
  "attributes": {
    "input": {
      "type": "identityAttribute",
      "attributes": {
        "name": "department"
      }
    }
  }
}
```

#### Result

```
Input: department = "Engineering"
Output: "ENGINEERING"

Input: department = "it operations"
Output: "IT OPERATIONS"
```

---

### Hands-On Exercise 4B: Create Uppercase Display Name

**Scenario:** Create displayName as "JOHN SMITH"

#### UI Style: Steps

```
1. Create/edit "displayNameUpper" attribute
2. Transform: Upper
3. Input: Transform (select Concatenation)
4. Use the firstname + " " + lastname concatenation
5. Preview: Should show "JOHN SMITH"
6. Save
```

#### JSON Style: Transform Definition

```json
{
  "name": "displayNameUpper",
  "type": "upper",
  "attributes": {
    "input": {
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
  }
}
```

---

## Transform 5: Trim (15 minutes)

### What It Does

Removes whitespace from beginning AND end of string

### When to Use

- Data cleanup (source data often has trailing/leading spaces)
- Before concatenation (to prevent spaces in output)
- Before email/username generation

---

### Hands-On Exercise 5A: Clean Up Source Data

**Scenario:** Source has "  John  " (with spaces). We need "John"

#### UI Style: Steps

```
1. Create test attribute "testTrim"
2. Transform: Trim
3. Input: Select "firstname" 
4. Preview: See "John" (spaces removed)
5. Save
```

#### JSON Style: Transform Definition

```json
{
  "name": "testTrim",
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

#### Result

```
Input: firstname = "  John  " (leading and trailing spaces)
Output: "John"

Input: firstname = "\t Smith \n" (tabs and newlines too)
Output: "Smith"
```

---

### Hands-On Exercise 5B: Build Clean Email with Trim + Concatenation

**Scenario:** Create email that handles messy source data

**Real problem:** Source often has "  John  " and "  Smith  "

#### UI Style: Steps

```
1. Edit "email" attribute
2. Transform: Trim
3. Input: Attribute "firstname"
4. Next Step: Add another transform
5. Transform: Trim again
6. Input: Attribute "lastname"
7. Next Step: Add Concatenation
8. Combine: [trimmed firstname].[trimmed lastname]@company.com
9. Next Step: Add Lower
10. Preview: Clean lowercase email!
11. Save
```

#### JSON Style: Transform Definition

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
      ]
    },
    "next": {
      "type": "lower"
    }
  }
}
```

#### Result

```
Input: firstname = "  John  ", lastname = "  Smith  "
Output: "john.smith@company.com" (clean!)
```

---

## Transform 6: Substring (25 minutes)

### What It Does

Extracts a portion of a string based on start and end positions

### When to Use

- Extract area code from phone number
- Get first N characters
- Extract employee ID from longer string
- Truncate long names

---

### Hands-On Exercise 6A: Extract First Initial

**Scenario:** Get first letter of firstname for a username initial

#### UI Style: Steps

```
1. Create attribute "firstInitial"
2. Transform: Substring
3. Input: Select "firstname"
4. Begin: 0 (start position - counts from 0)
5. End: 1 (extract 1 character)
6. Preview: Shows "J" for John
7. Save
```

#### JSON Style: Transform Definition

```json
{
  "name": "firstInitial",
  "type": "substring",
  "attributes": {
    "input": {
      "type": "identityAttribute",
      "attributes": {
        "name": "firstname"
      }
    },
    "begin": 0,
    "end": 1
  }
}
```

#### Result

```
Input: firstname = "John"
Substring (0, 1): "J"

Input: firstname = "Christopher"
Substring (0, 1): "C"
```

#### Key: Zero-based indexing!

- Position 0 = first character
- Position 1 = second character
- Begin = inclusive
- End = exclusive (so end: 1 means "up to but not including position 1")

---

### Hands-On Exercise 6B: Build Username (First Letter + Last Name)

**Scenario:** Username = first letter of firstname + full lastname, lowercase, max 8 chars

Example: "Christopher Anderson" → "canders"

#### UI Style: Steps

```
1. Create attribute "username"
2. Transform: Substring
3. Input: firstname
4. Begin: 0, End: 1 (get first letter)
5. Next: Add Concatenation
6. Values: [first letter] + [lastname]
7. Next: Add Lower
8. Next: Add Substring again
9. Begin: 0, End: 8 (max 8 chars)
10. Preview: Should show "canders" or similar
11. Save
```

#### JSON Style: Transform Definition

```json
{
  "name": "username",
  "type": "substring",
  "attributes": {
    "input": {
      "type": "identityAttribute",
      "attributes": {
        "name": "firstname"
      }
    },
    "begin": 0,
    "end": 1
  },
  "next": {
    "type": "concat",
    "attributes": {
      "values": [
        {
          "type": "reference"
        },
        {
          "type": "identityAttribute",
          "attributes": {
            "name": "lastname"
          }
        }
      ]
    },
    "next": {
      "type": "lower",
      "next": {
        "type": "substring",
        "attributes": {
          "begin": 0,
          "end": 8
        }
      }
    }
  }
}
```

#### Result

```
Input: firstname = "Christopher", lastname = "Anderson"
Step 1 (Substring): "C"
Step 2 (Concat): "CAnderson"
Step 3 (Lower): "canderson"
Step 4 (Substring 0,8): "canderson" (already 9 chars, truncated to 8)
Final Output: "canderson" or "canders" if lastname has max 7 chars
```

---

## Transform 7: Replace (30 minutes)

### What It Does

Finds text and replaces it with something else

### When to Use

- Remove special characters
- Format phone numbers
- Clean data
- Standardize formats

---

### Hands-On Exercise 7A: Remove Spaces

**Scenario:** Phone number has spaces: "555 123 4567" → Remove spaces → "5551234567"

#### UI Style: Steps

```
1. Create attribute "phoneClean"
2. Transform: Replace
3. Input: Select "phone" attribute
4. Regex: " " (space character)
5. Replacement: "" (empty - removes it)
6. Preview: Shows phone without spaces
7. Save
```

#### JSON Style: Transform Definition

```json
{
  "name": "phoneClean",
  "type": "replace",
  "attributes": {
    "input": {
      "type": "identityAttribute",
      "attributes": {
        "name": "phone"
      }
    },
    "regex": " ",
    "replacement": ""
  }
}
```

#### Result

```
Input: phone = "555 123 4567"
Output: "5551234567"
```

---

### Hands-On Exercise 7B: Remove Special Characters

**Scenario:** Clean phone number completely: "(555) 123-4567" → "5551234567"

#### UI Style: Steps

```
1. Create attribute "phoneDigitsOnly"
2. Transform: Replace (need to chain multiple)
3. First Replace: "(" with ""
4. Then Replace: ")" with ""
5. Then Replace: "-" with ""
6. Then Replace: " " with ""
7. Result: Pure digits
8. Save
```

#### JSON Style: Transform Definition

```json
{
  "name": "phoneDigitsOnly",
  "type": "replace",
  "attributes": {
    "input": {
      "type": "identityAttribute",
      "attributes": {
        "name": "phone"
      }
    },
    "regex": "\\(",
    "replacement": ""
  },
  "next": {
    "type": "replace",
    "attributes": {
      "regex": "\\)",
      "replacement": ""
    },
    "next": {
      "type": "replace",
      "attributes": {
        "regex": "-",
        "replacement": ""
      },
      "next": {
        "type": "replace",
        "attributes": {
          "regex": " ",
          "replacement": ""
        }
      }
    }
  }
}
```

#### Result

```
Input: phone = "(555) 123-4567"
After Remove (: "555) 123-4567"
After Remove ): "555 123-4567"
After Remove -: "555 1234567"
After Remove space: "5551234567" ✓
```

---

## Understanding Transform Chaining

### How Chaining Works

**Transform Chaining** = Feed the output of one transform as input to the next transform

```
Input Data
    ↓
[Transform 1] → Output 1
                    ↓
                [Transform 2] → Output 2
                                  ↓
                              [Transform 3] → Final Output
```

### In ISC UI

When editing a transform, look for:
- **Input** field at the top (what data goes in)
- **Transform type** dropdown
- **Configuration** for that transform
- **Next** section (optional - chain another transform here)

### In JSON

Chaining uses the **"next"** property:

```json
{
  "name": "myAttribute",
  "type": "transform1",
  "attributes": { /* config */ },
  "next": {
    "type": "transform2",
    "attributes": { /* config */ },
    "next": {
      "type": "transform3",
      "attributes": { /* config */ }
    }
  }
}
```

### Using "reference" in Concatenation

When concatenating a chained result, use **"reference"** to mean "the output from previous step":

```json
{
  "type": "concat",
  "attributes": {
    "values": [
      {
        "type": "reference"  // This means "use the output from previous transform"
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

---

## Day 1 Final Assessment

### Checkpoint Quiz

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

**UI Style (Chaining Order):**
```
1. Trim (firstname)
2. Trim (lastname)
3. Concatenation (trim output + "." + trim output + "@company.com")
4. Replace (" " with "")
5. Lower
6. Substring (0, 50)
```

**JSON Style:**
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
      ]
    },
    "next": {
      "type": "replace",
      "attributes": {
        "regex": " ",
        "replacement": ""
      },
      "next": {
        "type": "lower",
        "next": {
          "type": "substring",
          "attributes": {
            "begin": 0,
            "end": 50
          }
        }
      }
    }
  }
}
```

**Output:** "john.smith@company.com"

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

Output: "John.Smith" ✓
```

Always trim text attributes before concatenation!

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

**JSON Style:**
```json
{
  "name": "username",
  "type": "substring",
  "attributes": {
    "input": {
      "type": "identityAttribute",
      "attributes": {
        "name": "firstname"
      }
    },
    "begin": 0,
    "end": 1
  },
  "next": {
    "type": "concat",
    "attributes": {
      "values": [
        {
          "type": "reference"
        },
        {
          "type": "identityAttribute",
          "attributes": {
            "name": "lastname"
          }
        }
      ]
    },
    "next": {
      "type": "lower",
      "next": {
        "type": "substring",
        "attributes": {
          "begin": 0,
          "end": 8
        }
      }
    }
  }
}
```

**Output:** "canders" (c + anderson = canderson, lowercased, truncated to 8 chars)

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

**JSON:**
```json
{
  "name": "displayName",
  "type": "upper",
  "attributes": {
    "input": {
      "type": "identityAttribute",
      "attributes": {
        "name": "lastname"
      }
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
            "value": ", "
          }
        },
        {
          "type": "identityAttribute",
          "attributes": {
            "name": "firstname"
          }
        }
      ]
    }
  }
}
```

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
- Concatenation (combine the two substrings)
- Lower (lowercase the result)

Think about the order!

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

You'll need multiple Replace transforms chained together:
- Replace "+" (but keep the + just remove space after)
- Replace "(" with ""
- Replace ")" with ""
- Replace " " with ""
- Replace "-" with ""

Or think: Replace all non-digits? Actually, keep it simple with 5 Replace steps.

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
| Wrong regex in Replace | Not replacing what you expect | Test regex carefully (escape special chars) |

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

---

**Congratulations on completing Day 1! You're well on your way to transform mastery!**

---

*Part of the SailPoint ISC Transforms Mastery 7-Day Study Plan*
