# DAY 2: CONDITIONAL LOGIC & DECISION MAKING - AFTERNOON SESSION

## Table of Contents
- [Overview](#overview)
- [What You'll Build Today](#what-youll-build-today)
- [Pattern 1: Smart Email Generator](#pattern-1-smart-email-generator-45-minutes)
- [Pattern 2: Complete Lifecycle State Calculator](#pattern-2-complete-lifecycle-state-calculator-60-minutes)
- [Pattern 3: Manager Email Resolution](#pattern-3-manager-email-resolution-35-minutes)
- [Pattern 4: Country Assignment with Complex Logic](#pattern-4-country-assignment-with-complex-logic-30-minutes)
- [Day 2 Final Assessment](#day-2-final-assessment)
- [Homework Challenges](#homework-challenges)

---

## Overview

**Session Duration:** 2.5 hours

**Prerequisites:** 
- Completed Day 2 Morning Session
- Understand Conditional transform
- Understand FirstValid transform
- Comfortable with transform chaining

**Goal:** Build production-ready transforms that handle real-world complexity

---

## What You'll Build Today

By the end of this afternoon session, you'll have built:

- ✅ Smart email generator with 5-level fallback
- ✅ Complete lifecycle state calculator with date logic
- ✅ Manager email resolver with multiple strategies
- ✅ Country assignment with location parsing
- ✅ All edge cases handled (nulls, special chars, formatting)

These are **production-ready patterns** you can use immediately!

---

## Pattern 1: Smart Email Generator (45 minutes)

### Business Requirements

Create an email address with the following priority:
```
Priority 1: Use preferred_email if exists and is valid
Priority 2: Use work_email if exists
Priority 3: Generate firstname.lastname@company.com if both names exist
Priority 4: Generate employeeID@company.com if ID exists
Priority 5: Use "noemail@company.com" as absolute fallback
```

**Additional Requirements:**
- All emails must be lowercase
- Remove any spaces
- Handle special characters in names
- Max length 50 characters

---

### Step 1: Create Generated Email from Name (20 min)

This will be our Priority 3 option.

#### UI Style: Create Helper Attribute: generatedEmailFromName
```
1. Go to Identity Profile > Mappings
2. Create new attribute "generatedEmailFromName"
3. Build transform chain:

   Transform 1: Conditional (Check if names exist)
     Condition: firstname ne null
     If True: Continue to next transform
     If False: Return null (stops here)
   
   Transform 2: Conditional (Check lastname)
     Condition: lastname ne null
     If True: Continue
     If False: Return null
   
   Transform 3: Trim
     Input: firstname
     Output: Cleaned firstname
   
   Transform 4: Trim
     Input: lastname
     Output: Cleaned lastname
   
   Transform 5: Concatenation
     Values: firstname + "." + lastname + "@company.com"
     Output: "John.Smith@company.com"
   
   Transform 6: Replace (remove spaces)
     Find: " "
     Replace: ""
     Output: "John.Smith@company.com"
   
   Transform 7: Replace (remove hyphens)
     Find: "-"
     Replace: ""
     Output: "JohnSmith@company.com" (if name had hyphen)
   
   Transform 8: Replace (remove apostrophes)
     Find: "'"
     Replace: ""
     Output: Clean name
   
   Transform 9: Lower
     Output: "john.smith@company.com"
   
   Transform 10: Substring (ensure max length)
     Begin: 0
     Length: 50
     Output: Truncated if needed

4. Preview with test identities
5. Save
```

#### JSON Style: Transform Definition
```json
{
  "name": "generatedEmailFromName",
  "type": "conditional",
  "attributes": {
    "expression": "firstname ne null",
    "positiveCondition": {
      "type": "conditional",
      "attributes": {
        "expression": "lastname ne null",
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
                  "type": "replace",
                  "attributes": {
                    "regex": " ",
                    "replacement": "",
                    "next": {
                      "type": "replace",
                      "attributes": {
                        "regex": "-",
                        "replacement": "",
                        "next": {
                          "type": "replace",
                          "attributes": {
                            "regex": "'",
                            "replacement": "",
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
                    }
                  }
                }
              }
            }
          }
        },
        "negativeCondition": null
      }
    },
    "negativeCondition": null
  }
}
```

#### Test Cases
```
Test 1:
  firstname = "John"
  lastname = "Smith"
  Output: "john.smith@company.com"

Test 2:
  firstname = "Mary-Ann"
  lastname = "O'Brien"
  Output: "maryann.obrien@company.com"

Test 3:
  firstname = null
  lastname = "Smith"
  Output: null (failed conditional, becomes null)

Test 4:
  firstname = "Christopher"
  lastname = "Van Der Berg"
  Output: "christopher.vanderbberg@company.com" (spaces removed)
```

---

### Step 2: Create Generated Email from ID (10 min)

This will be our Priority 4 option.

#### UI Style: Create Helper Attribute: generatedEmailFromID
```
1. Create new attribute "generatedEmailFromID"
2. Build transform chain:

   Transform 1: Conditional
     Condition: employeeID ne null
     If True: Continue
     If False: Return null
   
   Transform 2: Trim
     Input: employeeID
   
   Transform 3: Concatenation
     Values: employeeID + "@company.com"
     Output: "E12345@company.com"
   
   Transform 4: Lower
     Output: "e12345@company.com"

3. Preview
4. Save
```

#### JSON Style: Transform Definition
```json
{
  "name": "generatedEmailFromID",
  "type": "conditional",
  "attributes": {
    "expression": "employeeID ne null",
    "positiveCondition": {
      "type": "trim",
      "attributes": {
        "input": {
          "type": "identityAttribute",
          "attributes": {
            "name": "employeeID"
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
    },
    "negativeCondition": null
  }
}
```

#### Test Cases
```
Test 1:
  employeeID = "E12345"
  Output: "e12345@company.com"

Test 2:
  employeeID = null
  Output: null
```

---

### Step 3: Build Final Email with FirstValid (15 min)

Now combine everything!

#### UI Style: Create Attribute: primaryEmail
```
1. Create new attribute "primaryEmail"
2. Transform: FirstValid
3. Configuration:
   
   Values to check (in order):
   1. preferred_email (attribute from source)
   2. work_email (attribute from source)
   3. generatedEmailFromName (attribute we created)
   4. generatedEmailFromID (attribute we created)
   5. Static "noemail@company.com"

4. Preview with multiple identity scenarios
5. Save
```

#### JSON Style: Transform Definition
```json
{
  "name": "primaryEmail",
  "type": "firstValid",
  "attributes": {
    "values": [
      {
        "type": "identityAttribute",
        "attributes": {
          "name": "preferred_email"
        }
      },
      {
        "type": "identityAttribute",
        "attributes": {
          "name": "work_email"
        }
      },
      {
        "type": "identityAttribute",
        "attributes": {
          "name": "generatedEmailFromName"
        }
      },
      {
        "type": "identityAttribute",
        "attributes": {
          "name": "generatedEmailFromID"
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

#### Comprehensive Test Cases
```
Test Case 1: User has preferred email
  preferred_email = "john.preferred@company.com"
  work_email = "john.work@company.com"
  firstname = "John"
  lastname = "Smith"
  employeeID = "E12345"
  Output: "john.preferred@company.com" (Priority 1)

Test Case 2: No preferred, has work
  preferred_email = null
  work_email = "john.work@company.com"
  Output: "john.work@company.com" (Priority 2)

Test Case 3: No emails, has name
  preferred_email = null
  work_email = null
  firstname = "John"
  lastname = "Smith"
  employeeID = "E12345"
  Output: "john.smith@company.com" (Priority 3 - generated)

Test Case 4: No emails, no lastname
  preferred_email = null
  work_email = null
  firstname = "John"
  lastname = null
  employeeID = "E12345"
  Output: "e12345@company.com" (Priority 4 - ID fallback)

Test Case 5: Nothing available
  All attributes = null
  Output: "noemail@company.com" (Priority 5 - absolute fallback)

Test Case 6: Special characters in name
  firstname = "François"
  lastname = "Müller-Schmidt"
  Output: "françois.mullerschmidt@company.com" (cleaned)
```

---

### Key Learning Points

- **Layered fallback strategy** handles all scenarios
- **Helper attributes** make complex logic manageable
- **Each priority level** has its own validation
- **Always have absolute fallback** (static value)
- **Test with null scenarios** at every level

---

## Pattern 2: Complete Lifecycle State Calculator (60 minutes)

### Business Requirements

Calculate lifecycle state based on:
```
States: Pre-hire, Active, Leave of Absence, Terminated, Archived

Logic:
- Pre-hire: hire_date is in the future
- Active: hire_date <= today AND (term_date is null OR term_date > today) AND leave_flag = false
- Leave of Absence: leave_flag = true
- Terminated: term_date <= today AND term_date > (today - 90 days)
- Archived: term_date <= (today - 90 days)
```

**Note:** This requires understanding dates, which we'll handle step by step.

---

### Understanding Date Handling in ISC

#### ISO 8601 Format Requirement

SailPoint expects dates in this format:
```
ISO 8601: "2024-01-15T00:00:00Z"

Not: "01/15/2024"
Not: "15-Jan-2024"
Not: "2024-01-15" (missing time component)
```

#### Date Comparison Concepts

For today, we'll use a **simplified approach** with Conditional operators:
```
Instead of: hire_date > today (requires Date Compare transform)
We'll use: Check if hire_date contains future year "2026" (simplified)
```

**In Day 4, we'll learn proper Date Compare and Date Math transforms.**

For now, let's build the logical structure.

---

### Step 1: Build Simplified Lifecycle State (30 min)

#### UI Style: Create Attribute: calculatedLifecycleState
```
1. Create new attribute "calculatedLifecycleState"
2. Build nested conditional chain:

LEVEL 1: Check for Leave of Absence

Transform 1: Conditional
  Condition: leave_flag eq true (or leave_flag eq "true" if it's a string)
  If True: "Leave of Absence"
  If False: Continue to Level 2

LEVEL 2: Check for Terminated

Transform 2: Conditional
  Condition: term_date ne null
  If True: "Terminated" (simplified - we'll improve with dates later)
  If False: Continue to Level 3

LEVEL 3: Check for Pre-hire

Transform 3: Conditional
  Condition: hire_date contains "2026" (simplified future check)
  If True: "Pre-hire"
  If False: "Active"

3. Preview with different scenarios
4. Save
```

#### JSON Style: Transform Definition
```json
{
  "name": "calculatedLifecycleState",
  "type": "conditional",
  "attributes": {
    "expression": "leave_flag eq true",
    "positiveCondition": "Leave of Absence",
    "negativeCondition": {
      "type": "conditional",
      "attributes": {
        "expression": "term_date ne null",
        "positiveCondition": "Terminated",
        "negativeCondition": {
          "type": "conditional",
          "attributes": {
            "expression": "hire_date contains \"2026\"",
            "positiveCondition": "Pre-hire",
            "negativeCondition": "Active"
          }
        }
      }
    }
  }
}
```

#### Test Cases (Simplified Version)
```
Test 1: Leave of Absence
  leave_flag = true
  hire_date = "2024-01-15"
  term_date = null
  Output: "Leave of Absence"

Test 2: Terminated
  leave_flag = false
  hire_date = "2024-01-15"
  term_date = "2024-12-31"
  Output: "Terminated"

Test 3: Pre-hire
  leave_flag = false
  hire_date = "2026-06-01"
  term_date = null
  Output: "Pre-hire"

Test 4: Active
  leave_flag = false
  hire_date = "2024-01-15"
  term_date = null
  Output: "Active"
```

---

### Step 2: Add Status Field Logic (15 min)

**Enhancement:** Also check a status field for explicit state marking.

#### Updated Logic
```
Priority 1: If status = "On Leave", return "Leave of Absence"
Priority 2: If leave_flag = true, return "Leave of Absence"
Priority 3: If term_date exists, check if archived or terminated
Priority 4: If hire_date in future, return "Pre-hire"
Priority 5: Default to "Active"
```

#### UI Style: Updated Transform Chain
```
LEVEL 1: Check status field

Transform 1: Conditional
  Condition: status eq "On Leave"
  If True: "Leave of Absence"
  If False: Continue

LEVEL 2: Check leave_flag

Transform 2: Conditional
  Condition: leave_flag eq true
  If True: "Leave of Absence"
  If False: Continue

LEVEL 3: Check termination

Transform 3: Conditional
  Condition: term_date ne null
  If True: Continue to Terminated vs Archived logic
  If False: Continue to Pre-hire check

LEVEL 3a: Terminated vs Archived (simplified)

Transform 4: Conditional
  Condition: term_date contains "2024" (recent)
  If True: "Terminated"
  If False: "Archived" (older dates)

LEVEL 4: Pre-hire check

Transform 5: Conditional
  Condition: hire_date contains "2026"
  If True: "Pre-hire"
  If False: "Active"
```

#### JSON Style: Transform Definition
```json
{
  "name": "calculatedLifecycleState",
  "type": "conditional",
  "attributes": {
    "expression": "status eq \"On Leave\"",
    "positiveCondition": "Leave of Absence",
    "negativeCondition": {
      "type": "conditional",
      "attributes": {
        "expression": "leave_flag eq true",
        "positiveCondition": "Leave of Absence",
        "negativeCondition": {
          "type": "conditional",
          "attributes": {
            "expression": "term_date ne null",
            "positiveCondition": {
              "type": "conditional",
              "attributes": {
                "expression": "term_date contains \"2024\"",
                "positiveCondition": "Terminated",
                "negativeCondition": "Archived"
              }
            },
            "negativeCondition": {
              "type": "conditional",
              "attributes": {
                "expression": "hire_date contains \"2026\"",
                "positiveCondition": "Pre-hire",
                "negativeCondition": "Active"
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

### Step 3: Handle Edge Cases (15 min)

#### Add Null Handling

What if critical fields are null?

#### UI Style: Enhanced Logic
```
LEVEL 0: Validate required fields

Transform 1: Conditional
  Condition: hire_date eq null
  If True: "Unknown - Missing Hire Date"
  If False: Continue to normal logic

Then continue with previous levels...
```

#### JSON Style: Transform Definition with Null Handling
```json
{
  "name": "calculatedLifecycleState",
  "type": "conditional",
  "attributes": {
    "expression": "hire_date eq null",
    "positiveCondition": "Unknown - Missing Hire Date",
    "negativeCondition": {
      "type": "conditional",
      "attributes": {
        "expression": "status eq \"On Leave\"",
        "positiveCondition": "Leave of Absence",
        "negativeCondition": {
          "type": "conditional",
          "attributes": {
            "expression": "leave_flag eq true",
            "positiveCondition": "Leave of Absence",
            "negativeCondition": {
              "type": "conditional",
              "attributes": {
                "expression": "term_date ne null",
                "positiveCondition": {
                  "type": "conditional",
                  "attributes": {
                    "expression": "term_date contains \"2024\"",
                    "positiveCondition": "Terminated",
                    "negativeCondition": "Archived"
                  }
                },
                "negativeCondition": {
                  "type": "conditional",
                  "attributes": {
                    "expression": "hire_date contains \"2026\"",
                    "positiveCondition": "Pre-hire",
                    "negativeCondition": "Active"
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

#### Add Error States
```
Additional states:
- "Unknown - Missing Hire Date"
- "Unknown - Invalid Data"
- "Pending - Awaiting Data"
```

#### Complete Test Matrix
```
| leave_flag | hire_date  | term_date  | status      | Expected Output        |
|------------|------------|------------|-------------|------------------------|
| true       | 2024-01-15 | null       | Active      | Leave of Absence       |
| false      | 2024-01-15 | 2024-12-31 | Active      | Terminated             |
| false      | 2026-06-01 | null       | Active      | Pre-hire               |
| false      | 2024-01-15 | null       | Active      | Active                 |
| false      | 2024-01-15 | 2020-01-01 | Active      | Archived               |
| false      | null       | null       | Active      | Unknown - Missing Data |
| false      | 2024-01-15 | null       | On Leave    | Leave of Absence       |
```

---

### Key Learning Points

- **Order of checks matters** - most specific first
- **Multiple ways to reach same state** (status field OR flag)
- **Edge case handling** prevents errors
- **Unknown states** better than failures
- **Document your logic** - complex conditionals need comments

---

## Pattern 3: Manager Email Resolution (35 minutes)

### Business Requirements

Get manager's email with multiple fallback strategies:
```
Priority 1: Get manager's email from their Active Directory account
Priority 2: Generate from manager's firstname.lastname
Priority 3: Look up department head email by department
Priority 4: Use "no-manager@company.com"
```

---

### Step 1: Create Manager Generated Email (15 min)

#### UI Style: Create Helper Attribute: managerGeneratedEmail
```
1. Create attribute "managerGeneratedEmail"
2. Build transform chain:

   Transform 1: Conditional (check manager name exists)
     Condition: manager_firstname ne null
     If True: Continue
     If False: Return null
   
   Transform 2: Conditional (check lastname)
     Condition: manager_lastname ne null
     If True: Continue
     If False: Return null
   
   Transform 3: Trim
     Input: manager_firstname
   
   Transform 4: Trim
     Input: manager_lastname
   
   Transform 5: Concatenation
     Values: manager_firstname + "." + manager_lastname + "@company.com"
   
   Transform 6: Lower
     Output: "jane.doe@company.com"

3. Save
```

#### JSON Style: Transform Definition
```json
{
  "name": "managerGeneratedEmail",
  "type": "conditional",
  "attributes": {
    "expression": "manager_firstname ne null",
    "positiveCondition": {
      "type": "conditional",
      "attributes": {
        "expression": "manager_lastname ne null",
        "positiveCondition": {
          "type": "trim",
          "attributes": {
            "input": {
              "type": "identityAttribute",
              "attributes": {
                "name": "manager_firstname"
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
                          "name": "manager_lastname"
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
        },
        "negativeCondition": null
      }
    },
    "negativeCondition": null
  }
}
```

---

### Step 2: Create Department Head Lookup (10 min)

#### UI Style: Create Lookup Table
```
1. Go to Admin > Identity Management > Lookup Tables (or configure via transform)
2. Create lookup table: "DepartmentHeads"
3. Add entries:

   Engineering --> eng-head@company.com
   Finance --> finance-head@company.com
   HR --> hr-head@company.com
   IT --> it-head@company.com
   Sales --> sales-head@company.com
   Marketing --> marketing-head@company.com
   Operations --> ops-head@company.com
   ... (add all departments)

4. Save lookup table
```

#### UI Style: Create Helper Attribute: deptHeadEmail
```
1. Create attribute "deptHeadEmail"
2. Transform: Lookup
   Input: department
   Lookup Table: DepartmentHeads
   Default Value: null (or "unknown-dept@company.com")
3. Save
```

#### JSON Style: Transform Definition
```json
{
  "name": "deptHeadEmail",
  "type": "lookup",
  "attributes": {
    "input": {
      "type": "identityAttribute",
      "attributes": {
        "name": "department"
      }
    },
    "table": {
      "id": "department-heads"
    },
    "default": null
  }
}
```

---

### Step 3: Combine with FirstValid (10 min)

#### UI Style: Create Attribute: managerEmail
```
1. Create attribute "managerEmail"
2. Transform: FirstValid
3. Values (in order):
   
   1. Account Attribute
      Source: Active Directory (or manager's authoritative source)
      Attribute: mail (or email attribute name)
      Note: This gets the manager's actual AD email
   
   2. managerGeneratedEmail (attribute we created)
   
   3. deptHeadEmail (attribute we created)
   
   4. Static "no-manager@company.com"

4. Preview with different scenarios
5. Save
```

#### JSON Style: Transform Definition
```json
{
  "name": "managerEmail",
  "type": "firstValid",
  "attributes": {
    "values": [
      {
        "type": "accountAttribute",
        "attributes": {
          "sourceName": "Active Directory",
          "attributeName": "mail"
        }
      },
      {
        "type": "identityAttribute",
        "attributes": {
          "name": "managerGeneratedEmail"
        }
      },
      {
        "type": "identityAttribute",
        "attributes": {
          "name": "deptHeadEmail"
        }
      },
      {
        "type": "static",
        "attributes": {
          "value": "no-manager@company.com"
        }
      }
    ],
    "ignoreErrors": true
  }
}
```

---

### Test Cases
```
Test Case 1: Manager has AD account with email
  Manager exists in AD with mail = "jane.doe@company.com"
  Output: "jane.doe@company.com" (from AD)

Test Case 2: No AD email, but has name
  Manager not in AD (or no email in AD)
  manager_firstname = "Jane"
  manager_lastname = "Doe"
  Output: "jane.doe@company.com" (generated)

Test Case 3: No manager name, but has department
  manager_firstname = null
  department = "Engineering"
  Lookup finds: Engineering --> "eng-head@company.com"
  Output: "eng-head@company.com"

Test Case 4: No manager info at all
  All manager fields null
  department = null or not in lookup
  Output: "no-manager@company.com" (fallback)
```

---

### Important Note on Account Attribute

**Account Attribute transform** requires:
- Manager must have an identity in ISC
- Manager must have account on the specified source (AD)
- Account must be correlated to manager's identity
- Attribute must exist on that account

If any of these fail, it returns null and moves to next FirstValid option.

---

### Key Learning Points

- **Account Attribute** powerful but has prerequisites
- **Generated values** good backup when account data missing
- **Lookup tables** efficient for department-based defaults
- **Combination approach** handles all scenarios
- **Test with missing manager** scenarios

---

## Pattern 4: Country Assignment with Complex Logic (30 minutes)

### Business Requirements

Determine country based on office location with complex parsing:
```
Office locations can be:
- Simple: "Austin"
- Complex: "Austin Office - Building 2"
- Multiple cities: "New York/London"
- Null or unknown

Rules:
- US cities (Austin, Dallas, New York, etc.) --> "USA"
- UK cities (London, Manchester) --> "United Kingdom"
- Japan cities (Tokyo, Osaka) --> "Japan"
- Parse multi-location strings
- Default to "Unknown"
```

---

### Step 1: Simple Location Matching (15 min)

#### UI Style: Create Attribute: country
```
1. Create attribute "country"
2. Build conditional chain:

LEVEL 1: Check for US cities

Transform 1: Conditional
  Condition: office_location contains "Austin"
  If True: "USA"
  If False: Continue

Transform 2: Conditional
  Condition: office_location contains "Dallas"
  If True: "USA"
  If False: Continue

Transform 3: Conditional
  Condition: office_location contains "New York"
  If True: "USA"
  If False: Continue

Transform 4: Conditional
  Condition: office_location contains "Chicago"
  If True: "USA"
  If False: Continue

LEVEL 2: Check for UK cities

Transform 5: Conditional
  Condition: office_location contains "London"
  If True: "United Kingdom"
  If False: Continue

Transform 6: Conditional
  Condition: office_location contains "Manchester"
  If True: "United Kingdom"
  If False: Continue

LEVEL 3: Check for Japan cities

Transform 7: Conditional
  Condition: office_location contains "Tokyo"
  If True: "Japan"
  If False: Continue

Transform 8: Conditional
  Condition: office_location contains "Osaka"
  If True: "Japan"
  If False: "Unknown"

3. Preview
4. Save
```

#### JSON Style: Transform Definition (Nested Conditionals)
```json
{
  "name": "country",
  "type": "conditional",
  "attributes": {
    "expression": "office_location contains \"Austin\"",
    "positiveCondition": "USA",
    "negativeCondition": {
      "type": "conditional",
      "attributes": {
        "expression": "office_location contains \"Dallas\"",
        "positiveCondition": "USA",
        "negativeCondition": {
          "type": "conditional",
          "attributes": {
            "expression": "office_location contains \"New York\"",
            "positiveCondition": "USA",
            "negativeCondition": {
              "type": "conditional",
              "attributes": {
                "expression": "office_location contains \"Chicago\"",
                "positiveCondition": "USA",
                "negativeCondition": {
                  "type": "conditional",
                  "attributes": {
                    "expression": "office_location contains \"London\"",
                    "positiveCondition": "United Kingdom",
                    "negativeCondition": {
                      "type": "conditional",
                      "attributes": {
                        "expression": "office_location contains \"Manchester\"",
                        "positiveCondition": "United Kingdom",
                        "negativeCondition": {
                          "type": "conditional",
                          "attributes": {
                            "expression": "office_location contains \"Tokyo\"",
                            "positiveCondition": "Japan",
                            "negativeCondition": {
                              "type": "conditional",
                              "attributes": {
                                "expression": "office_location contains \"Osaka\"",
                                "positiveCondition": "Japan",
                                "negativeCondition": "Unknown"
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
      }
    }
  }
}
```

---

### Step 2: Better Approach - Using Lookup Table (15 min)

**Problem:** Too many conditionals is hard to maintain!

**Better Solution:** Use Lookup table with location parsing.

#### UI Style: Create City to Country Lookup
```
1. Create lookup table: "CityToCountry"
2. Add entries:

   Austin --> USA
   Dallas --> USA
   Houston --> USA
   New York --> USA
   Chicago --> USA
   San Francisco --> USA
   London --> United Kingdom
   Manchester --> United Kingdom
   Birmingham --> United Kingdom
   Tokyo --> Japan
   Osaka --> Japan
   Sydney --> Australia
   Toronto --> Canada
   ... (add all cities)

3. Set default value: "Unknown"
4. Save
```

#### Extract City from Location String

Sometimes office_location is complex: "Austin Office - Building 2"

We need to extract just "Austin"

##### UI Style: Create Helper Attribute: extractedCity
```
1. Create attribute "extractedCity"
2. Transform chain:

   Transform 1: Split
     Input: office_location
     Delimiter: " "
     Output: ["Austin", "Office", "-", "Building", "2"]
   
   Transform 2: Index
     Position: 0
     Output: "Austin" (first word)

3. Save
```

##### JSON Style: Transform Definition
```json
{
  "name": "extractedCity",
  "type": "split",
  "attributes": {
    "input": {
      "type": "identityAttribute",
      "attributes": {
        "name": "office_location"
      }
    },
    "delimiter": " ",
    "next": {
      "type": "index",
      "attributes": {
        "position": 0
      }
    }
  }
}
```

#### UI Style: Use Lookup with Extracted City
```
1. Create/edit attribute "country"
2. Transform: Lookup
   Input: extractedCity (attribute we created)
   Lookup Table: CityToCountry
   Default: "Unknown"
3. Save
```

#### JSON Style: Transform Definition
```json
{
  "name": "country",
  "type": "lookup",
  "attributes": {
    "input": {
      "type": "identityAttribute",
      "attributes": {
        "name": "extractedCity"
      }
    },
    "table": {
      "id": "city-to-country"
    },
    "default": "Unknown"
  }
}
```

---

### Test Cases
```
Test 1: Simple location
  office_location = "Austin"
  extractedCity = "Austin"
  country = "USA"

Test 2: Complex location
  office_location = "Austin Office - Building 2"
  extractedCity = "Austin"
  country = "USA"

Test 3: UK location
  office_location = "London"
  country = "United Kingdom"

Test 4: Unknown location
  office_location = "RemoteLocation"
  extractedCity = "RemoteLocation"
  country = "Unknown" (default from lookup)

Test 5: Null location
  office_location = null
  extractedCity = null
  country = "Unknown" (default)
```

---

### Key Learning Points

- **Lookup tables** more maintainable than many conditionals
- **String parsing** (Split + Index) extracts key data
- **Default values** in lookups handle unknowns
- **Helper attributes** break complex logic into steps
- **When you have 10+ options, use Lookup instead of Conditional**

---

## Day 2 Final Assessment

### What You Built Today

**Morning Session:**
- [ ] Simple conditionals
- [ ] Nested conditionals
- [ ] FirstValid fallback chains
- [ ] Combined conditional logic

**Afternoon Session:**
- [ ] Smart email generator (5-level fallback)
- [ ] Lifecycle state calculator (nested logic)
- [ ] Manager email resolver (Account Attribute + fallbacks)
- [ ] Country assignment (Lookup + parsing)

**Total: 10+ production-ready transform patterns!**

---

### Concepts Mastered

- ✅ If-then-else logic with Conditional
- ✅ All comparison operators
- ✅ Nested conditionals (3-5 levels deep)
- ✅ FirstValid for fallback chains
- ✅ Account Attribute for cross-identity data
- ✅ Lookup tables for value mapping
- ✅ String parsing with Split + Index
- ✅ Helper attributes for complex logic
- ✅ Edge case handling (nulls, unknowns)
- ✅ Production-ready patterns
- ✅ Both UI and JSON configuration

---

### Final Challenge Quiz

#### Challenge 1: Build Complete Access Level

Requirements:
```
If title contains "VP" OR title contains "C-level"
  Return "Executive"
Else if title contains "Director"
  Return "Director"
Else if title contains "Manager"
  Return "Manager"
Else if title contains "Senior" OR years_of_service > 5
  Return "Senior"
Else
  Return "Staff"
```

<details>
<summary>Click to reveal solution</summary>

**JSON Style:**
```json
{
  "name": "accessLevel",
  "type": "conditional",
  "attributes": {
    "expression": "title contains \"VP\"",
    "positiveCondition": "Executive",
    "negativeCondition": {
      "type": "conditional",
      "attributes": {
        "expression": "title contains \"C-level\"",
        "positiveCondition": "Executive",
        "negativeCondition": {
          "type": "conditional",
          "attributes": {
            "expression": "title contains \"Director\"",
            "positiveCondition": "Director",
            "negativeCondition": {
              "type": "conditional",
              "attributes": {
                "expression": "title contains \"Manager\"",
                "positiveCondition": "Manager",
                "negativeCondition": {
                  "type": "conditional",
                  "attributes": {
                    "expression": "title contains \"Senior\"",
                    "positiveCondition": "Senior",
                    "negativeCondition": {
                      "type": "conditional",
                      "attributes": {
                        "expression": "years_of_service gt 5",
                        "positiveCondition": "Senior",
                        "negativeCondition": "Staff"
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

**Note:** For years_of_service, you'd need to ensure it's a number. May need additional validation.

</details>

---

#### Challenge 2: Smart Phone Number

Build a phone number resolver:
```
Priority 1: Use mobile_phone if exists and starts with "+"
Priority 2: Use work_phone if exists
Priority 3: Use cleaned home_phone (remove formatting)
Priority 4: Use "555-0000"
```

<details>
<summary>Click to reveal solution</summary>

**Step 1:** Create cleaned home phone

**JSON Style:**
```json
{
  "name": "cleanedHomePhone",
  "type": "replace",
  "attributes": {
    "input": {
      "type": "identityAttribute",
      "attributes": {
        "name": "home_phone"
      }
    },
    "regex": "\\(",
    "replacement": "",
    "next": {
      "type": "replace",
      "attributes": {
        "regex": "\\)",
        "replacement": "",
        "next": {
          "type": "replace",
          "attributes": {
            "regex": " ",
            "replacement": "",
            "next": {
              "type": "replace",
              "attributes": {
                "regex": "-",
                "replacement": ""
              }
            }
          }
        }
      }
    }
  }
}
```

**Step 2:** Create validation for mobile
```json
{
  "name": "validatedMobilePhone",
  "type": "conditional",
  "attributes": {
    "expression": "mobile_phone startsWith \"+\"",
    "positiveCondition": {
      "type": "identityAttribute",
      "attributes": {
        "name": "mobile_phone"
      }
    },
    "negativeCondition": null
  }
}
```

**Step 3:** Combine with FirstValid
```json
{
  "name": "primaryPhone",
  "type": "firstValid",
  "attributes": {
    "values": [
      {
        "type": "identityAttribute",
        "attributes": {
          "name": "validatedMobilePhone"
        }
      },
      {
        "type": "identityAttribute",
        "attributes": {
          "name": "work_phone"
        }
      },
      {
        "type": "identityAttribute",
        "attributes": {
          "name": "cleanedHomePhone"
        }
      },
      {
        "type": "static",
        "attributes": {
          "value": "555-0000"
        }
      }
    ],
    "ignoreErrors": true
  }
}
```

</details>

---

#### Challenge 3: Department Normalization

Build department standardization:
```
Map 50+ department variations to 5 standard departments:
- All engineering variations --> "Engineering"
- All finance variations --> "Finance"
- All HR variations --> "Human Resources"
- All IT variations --> "Information Technology"
- All sales/marketing --> "Revenue"
- Everything else --> "Other"
```

What transform approach would you use?

<details>
<summary>Click to reveal solution</summary>

**Best Approach: Lookup Table**

**UI Style:**
```
1. Create Lookup Table: DepartmentNormalization

Entries (50+ variations):
Engineering --> Engineering
ENG --> Engineering
Software Development --> Engineering
Dev --> Engineering
R&D --> Engineering
Engineering-Software --> Engineering

Finance --> Finance
FIN --> Finance
Accounting --> Finance
Financial Services --> Finance

HR --> Human Resources
Human Resources --> Human Resources
People Ops --> Human Resources
Talent --> Human Resources

IT --> Information Technology
Information Technology --> Information Technology
Technology --> Information Technology
Tech --> Information Technology

Sales --> Revenue
Marketing --> Revenue
Business Development --> Revenue

... (continue for all variations)

Default: Other

2. Create Transform:
Attribute: standardDepartment
Transform: Lookup
  Input: department
  Table: DepartmentNormalization
  Default: "Other"
```

**JSON Style:**
```json
{
  "name": "standardDepartment",
  "type": "lookup",
  "attributes": {
    "input": {
      "type": "identityAttribute",
      "attributes": {
        "name": "department"
      }
    },
    "table": {
      "id": "department-normalization"
    },
    "default": "Other"
  }
}
```

**Why Lookup instead of Conditional:**
- 50+ variations would require 50+ nested conditionals
- Lookup is more maintainable (add entries without changing transform)
- Better performance
- Easier to document
- Can be updated by non-technical users

</details>

---

## Homework Challenges

### Challenge 1: Risk Score Calculator

Build a risk score based on multiple factors:
```
Calculate risk level:
- If access_level = "Executive" AND data_classification = "Confidential" --> "Critical"
- If failed_logins > 3 OR suspicious_activity = true --> "High"
- If external_user = true --> "Medium"
- Otherwise --> "Low"
```

**Hint:** You'll need nested conditionals checking multiple attributes.

---

### Challenge 2: Smart Display Name with Titles

Build displayName with business logic:
```
If title contains "Dr." or "PhD"
  Format: "Dr. Lastname, Firstname"
If title contains "VP" or "C-level"
  Format: "Lastname, Firstname (Executive)"
Otherwise
  Format: "Lastname, Firstname"
```

**Hint:** Combine Conditional + Concatenation

---

### Challenge 3: Multi-Source Email Validator

Build email validator that checks multiple sources:
```
Priority 1: AD email (if account exists)
Priority 2: HR email (if account exists)
Priority 3: Generated email (if names exist)
Priority 4: Previous email (from lastKnownEmail attribute)
Priority 5: Generate from username
Priority 6: Default

All emails must:
- Be lowercase
- Not contain spaces
- End with @company.com
```

**Hint:** You'll need Account Attribute + multiple helper attributes + FirstValid

---

## Day 2 Complete!

### You've Mastered

- ✅ Conditional logic (simple and nested)
- ✅ FirstValid fallback strategies
- ✅ Account Attribute transform
- ✅ Lookup tables for value mapping
- ✅ String parsing and extraction
- ✅ Production-ready patterns
- ✅ Edge case handling
- ✅ Complex decision trees
- ✅ Both UI and JSON configuration methods

---

## What's Next: Day 3 Preview

Tomorrow you'll learn:

### Lookup & Reference Transforms

- **Lookup** - Advanced table mapping
- **Account Attribute** - Deep dive into cross-identity data
- **Identity Attribute** - Reference other identity attributes
- **Reference** - External data lookups

### Real-World Applications

- Department hierarchy resolution
- Cross-source data correlation
- Reference data integration
- Multi-source attribute aggregation

**You'll learn to pull data from anywhere and combine it intelligently!**

---

## Study Tips for Tomorrow

### Review These Concepts

- How Account Attribute works
- When to use Lookup vs Conditional
- Identity correlation basics
- Account aggregation flow

### Prepare Questions

Think about:
- How do you get data from a user's manager?
- How do you reference another identity's attributes?
- How do lookup tables handle updates?
- What happens if referenced account doesn't exist?

---

**Excellent work on Day 2! You've built serious conditional logic skills!**

---

*Part of the SailPoint ISC Transforms Mastery 7-Day Study Plan*
