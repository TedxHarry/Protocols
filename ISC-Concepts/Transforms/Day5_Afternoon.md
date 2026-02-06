# DAY 5: TRANSFORM CHAINING MASTERY - AFTERNOON SESSION

## Table of Contents
- [Overview](#overview)
- [Pattern 1: Smart Username Generator](#pattern-1-smart-username-generator-30-minutes)
- [Pattern 2: International Phone Normalizer](#pattern-2-international-phone-normalizer-30-minutes)
- [Pattern 3: Complete Employee Profile Enrichment](#pattern-3-complete-employee-profile-enrichment-40-minutes)
- [Pattern 4: Dynamic Access Level Calculator](#pattern-4-dynamic-access-level-calculator-30-minutes)
- [Pattern 5: Intelligent Manager Resolution](#pattern-5-intelligent-manager-resolution-30-minutes)
- [Day 5 Final Assessment](#day-5-final-assessment)
- [Homework Challenges](#homework-challenges)

---

## Overview

**Session Duration:** 2.5 hours

**Prerequisites:** 
- Completed Day 5 Morning Session
- Understand advanced chaining strategies
- Comfortable with all transform types

**Goal:** Build 5 complete production-ready transform patterns that handle real-world complexity

---

## Pattern 1: Smart Username Generator (30 minutes)

### Business Requirements
```
Generate username with these requirements:

Format: firstname.lastname (lowercase)
Max length: 20 characters
Handle special characters (remove hyphens, apostrophes, spaces)
Handle very long names (truncate intelligently)
Handle missing lastname (use firstname only)
Handle international characters (keep as-is or transliterate)
No spaces, no special chars except period
```

---

### Architecture Overview
```
Helper Attributes:
1. cleanFirstname - Cleaned first name
2. cleanLastname - Cleaned last name
3. combinedName - firstname.lastname
4. usernameRaw - Combined and cleaned
5. username - Final truncated version
```

---

### Step 1: Clean Firstname

#### UI Configuration
```
Attribute: cleanFirstname

Transform Chain:
  Step 1: Conditional
    Condition: firstname is not null
    If True: Continue
    If False: Return empty string ""

  Step 2: Trim
    Input: firstname
    Output: "John" (from "John  ")

  Step 3: Replace (remove hyphens)
    Find: "-"
    Replace: ""
    Output: "John" (from "John-Paul" -> "JohnPaul")

  Step 4: Replace (remove apostrophes)
    Find: "'"
    Replace: ""
    Output: "John" (from "O'John" -> "OJohn")

  Step 5: Replace (remove spaces)
    Find: " "
    Replace: ""
    Output: "John" (from "John Michael" -> "JohnMichael")

  Step 6: Lower
    Output: "john"
```

---

#### JSON Structure
```json
{
  "name": "cleanFirstname",
  "type": "conditional",
  "attributes": {
    "expression": "firstname != null",
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
                  "type": "replace",
                  "attributes": {
                    "regex": " ",
                    "replacement": "",
                    "next": {
                      "type": "lower"
                    }
                  }
                }
              }
            }
          }
        }
      }
    },
    "negativeCondition": {
      "type": "static",
      "attributes": {
        "value": ""
      }
    }
  }
}
```

---

### Step 2: Clean Lastname (Same Pattern)

#### UI Configuration
```
Attribute: cleanLastname

Transform Chain:
  (Same as cleanFirstname but for lastname)
  
  Step 1: Conditional (lastname != null)
  Step 2: Trim
  Step 3: Replace "-"
  Step 4: Replace "'"
  Step 5: Replace " "
  Step 6: Lower
```

#### JSON Structure
```json
{
  "name": "cleanLastname",
  "type": "conditional",
  "attributes": {
    "expression": "lastname != null",
    "positiveCondition": {
      "type": "trim",
      "attributes": {
        "input": {
          "type": "identityAttribute",
          "attributes": {
            "name": "lastname"
          }
        },
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
                  "type": "replace",
                  "attributes": {
                    "regex": " ",
                    "replacement": "",
                    "next": {
                      "type": "lower"
                    }
                  }
                }
              }
            }
          }
        }
      }
    },
    "negativeCondition": {
      "type": "static",
      "attributes": {
        "value": ""
      }
    }
  }
}
```

---

### Step 3: Combine Names Intelligently

#### UI Configuration
```
Attribute: usernameRaw

Transform: Conditional Chain

  Level 1: Both names exist?
    Condition: cleanFirstname != "" AND cleanLastname != ""
    If True: Concatenate (cleanFirstname + "." + cleanLastname)
    If False: Continue

  Level 2: Only firstname exists?
    Condition: cleanFirstname != ""
    If True: Use cleanFirstname only
    If False: Continue

  Level 3: Only lastname exists?
    Condition: cleanLastname != ""
    If True: Use cleanLastname only
    If False: "user" (fallback)
```

---

#### JSON Structure
```json
{
  "name": "usernameRaw",
  "type": "conditional",
  "attributes": {
    "expression": "cleanFirstname != '' && cleanLastname != ''",
    "positiveCondition": {
      "type": "concat",
      "attributes": {
        "values": [
          {
            "type": "identityAttribute",
            "attributes": {
              "name": "cleanFirstname"
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
              "name": "cleanLastname"
            }
          }
        ]
      }
    },
    "negativeCondition": {
      "type": "conditional",
      "attributes": {
        "expression": "cleanFirstname != ''",
        "positiveCondition": {
          "type": "identityAttribute",
          "attributes": {
            "name": "cleanFirstname"
          }
        },
        "negativeCondition": {
          "type": "conditional",
          "attributes": {
            "expression": "cleanLastname != ''",
            "positiveCondition": {
              "type": "identityAttribute",
              "attributes": {
                "name": "cleanLastname"
              }
            },
            "negativeCondition": {
              "type": "static",
              "attributes": {
                "value": "user"
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

### Step 4: Truncate to Max Length

#### UI Configuration
```
Attribute: username

Transform Chain:
  Step 1: Reference usernameRaw
  
  Step 2: Substring
    Begin: 0
    Length: 20
    Output: First 20 characters
```

---

#### JSON Structure
```json
{
  "name": "username",
  "type": "substring",
  "attributes": {
    "input": {
      "type": "identityAttribute",
      "attributes": {
        "name": "usernameRaw"
      }
    },
    "begin": 0,
    "end": 20
  }
}
```

---

### Complete Test Cases
```
Test Case 1: Normal name
  firstname = "John"
  lastname = "Smith"
  cleanFirstname = "john"
  cleanLastname = "smith"
  usernameRaw = "john.smith"
  username = "john.smith" (10 chars, no truncation)

Test Case 2: Special characters
  firstname = "Mary-Ann"
  lastname = "O'Brien"
  cleanFirstname = "maryann"
  cleanLastname = "obrien"
  usernameRaw = "maryann.obrien"
  username = "maryann.obrien" (14 chars)

Test Case 3: Very long names
  firstname = "Christopher"
  lastname = "Vanderbilt-Johnson"
  cleanFirstname = "christopher"
  cleanLastname = "vanderbiltjohnson"
  usernameRaw = "christopher.vanderbiltjohnson" (30 chars)
  username = "christopher.vanderbi" (truncated to 20)

Test Case 4: Spaces in names
  firstname = "John Michael"
  lastname = "Van Der Berg"
  cleanFirstname = "johnmichael"
  cleanLastname = "vanderberg"
  usernameRaw = "johnmichael.vanderberg"
  username = "johnmichael.vanderbe" (truncated to 20)

Test Case 5: Missing lastname
  firstname = "Madonna"
  lastname = null
  cleanFirstname = "madonna"
  cleanLastname = ""
  usernameRaw = "madonna" (only firstname)
  username = "madonna"

Test Case 6: Missing firstname
  firstname = null
  lastname = "Prince"
  cleanFirstname = ""
  cleanLastname = "prince"
  usernameRaw = "prince" (only lastname)
  username = "prince"

Test Case 7: Both missing
  firstname = null
  lastname = null
  cleanFirstname = ""
  cleanLastname = ""
  usernameRaw = "user" (fallback)
  username = "user"
```

---

### Key Learning Points

- ✅ Break complex logic into helper attributes
- ✅ Handle all edge cases (nulls, special chars, long names)
- ✅ Conditional chains for intelligent fallback
- ✅ Clean data before combining
- ✅ Truncate at the end (after all processing)
- ✅ Always have a fallback for impossible scenarios

---

## Pattern 2: International Phone Normalizer (30 minutes)

### Business Requirements
```
Normalize phone numbers to E.164 format:

Input formats:
  (214) 555-1234
  +1 214-555-1234
  214.555.1234
  1-214-555-1234
  +44 20 7123 4567 (UK)
  +81-3-1234-5678 (Japan)

Output: E.164 format
  +12145551234 (US)
  +442071234567 (UK)
  +81312345678 (Japan)

Requirements:
- Keep country code if present
- Add default country code if missing (assume US = +1)
- Remove all formatting (parentheses, spaces, hyphens, periods)
- Validate length (10-15 digits)
```

---

### Architecture Overview
```
Helper Attributes:
1. phoneRaw - Source phone number
2. phoneDigitsOnly - All non-digits removed
3. phoneWithCountryCode - Add +1 if missing
4. phoneNormalized - Final E.164 format
```

---

### Step 1: Extract Digits Only

#### UI Configuration
```
Attribute: phoneDigitsOnly

Transform Chain:
  Step 1: Conditional
    Condition: phone is not null
    If True: Continue
    If False: Return null

  Step 2: Trim
    Input: phone

  Step 3: Replace (remove opening paren)
    Find: "("
    Replace: ""

  Step 4: Replace (remove closing paren)
    Find: ")"
    Replace: ""

  Step 5: Replace (remove spaces)
    Find: " "
    Replace: ""

  Step 6: Replace (remove hyphens)
    Find: "-"
    Replace: ""

  Step 7: Replace (remove periods)
    Find: "."
    Replace: ""

  Step 8: Replace (remove plus - we'll add it back)
    Find: "+"
    Replace: ""

Output: "12145551234" (digits only)
```

---

#### JSON Structure
```json
{
  "name": "phoneDigitsOnly",
  "type": "conditional",
  "attributes": {
    "expression": "phone != null",
    "positiveCondition": {
      "type": "trim",
      "attributes": {
        "input": {
          "type": "identityAttribute",
          "attributes": {
            "name": "phone"
          }
        },
        "next": {
          "type": "replace",
          "attributes": {
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
                        "replacement": "",
                        "next": {
                          "type": "replace",
                          "attributes": {
                            "regex": "\\.",
                            "replacement": "",
                            "next": {
                              "type": "replace",
                              "attributes": {
                                "regex": "\\+",
                                "replacement": ""
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
    },
    "negativeCondition": null
  }
}
```

---

### Step 2: Add Country Code if Missing

#### UI Configuration
```
Attribute: phoneWithCountryCode

Transform: Conditional Chain

  Level 1: Check if starts with country code (11+ digits)
    Condition: Length of phoneDigitsOnly >= 11
    If True: phoneDigitsOnly (already has country code)
    If False: Continue

  Level 2: US number without country code (10 digits)
    Condition: Length of phoneDigitsOnly = 10
    If True: Concatenate ("1" + phoneDigitsOnly)
    If False: phoneDigitsOnly (keep as-is, might be invalid)
```

---

#### JSON Structure
```json
{
  "name": "phoneWithCountryCode",
  "type": "conditional",
  "attributes": {
    "expression": "phoneDigitsOnly.length() >= 11",
    "positiveCondition": {
      "type": "identityAttribute",
      "attributes": {
        "name": "phoneDigitsOnly"
      }
    },
    "negativeCondition": {
      "type": "conditional",
      "attributes": {
        "expression": "phoneDigitsOnly.length() == 10",
        "positiveCondition": {
          "type": "concat",
          "attributes": {
            "values": [
              {
                "type": "static",
                "attributes": {
                  "value": "1"
                }
              },
              {
                "type": "identityAttribute",
                "attributes": {
                  "name": "phoneDigitsOnly"
                }
              }
            ]
          }
        },
        "negativeCondition": {
          "type": "identityAttribute",
          "attributes": {
            "name": "phoneDigitsOnly"
          }
        }
      }
    }
  }
}
```

> **Note:** Length checking in conditionals may require specific syntax in your ISC version. Verify with documentation.

---

### Step 3: Add Plus Sign (E.164 Format)

#### UI Configuration
```
Attribute: phoneNormalized

Transform: Conditional Chain

  Level 1: Validate length (11-15 digits)
    Condition: phoneWithCountryCode length between 11 and 15
    If True: Concatenate ("+" + phoneWithCountryCode)
    If False: null (invalid phone number)
```

---

#### JSON Structure
```json
{
  "name": "phoneNormalized",
  "type": "conditional",
  "attributes": {
    "expression": "phoneWithCountryCode.length() >= 11 && phoneWithCountryCode.length() <= 15",
    "positiveCondition": {
      "type": "concat",
      "attributes": {
        "values": [
          {
            "type": "static",
            "attributes": {
              "value": "+"
            }
          },
          {
            "type": "identityAttribute",
            "attributes": {
              "name": "phoneWithCountryCode"
            }
          }
        ]
      }
    },
    "negativeCondition": null
  }
}
```

---

### Complete Test Cases
```
Test Case 1: US number with formatting
  Input: "(214) 555-1234"
  phoneDigitsOnly = "2145551234"
  phoneWithCountryCode = "12145551234" (added 1)
  phoneNormalized = "+12145551234"

Test Case 2: US number with country code
  Input: "+1 214-555-1234"
  phoneDigitsOnly = "12145551234"
  phoneWithCountryCode = "12145551234" (already 11 digits)
  phoneNormalized = "+12145551234"

Test Case 3: UK number
  Input: "+44 20 7123 4567"
  phoneDigitsOnly = "442071234567"
  phoneWithCountryCode = "442071234567" (already 12 digits)
  phoneNormalized = "+442071234567"

Test Case 4: Japan number
  Input: "+81-3-1234-5678"
  phoneDigitsOnly = "81312345678"
  phoneWithCountryCode = "81312345678" (already 11 digits)
  phoneNormalized = "+81312345678"

Test Case 5: Invalid - too short
  Input: "555-1234"
  phoneDigitsOnly = "5551234"
  phoneWithCountryCode = "5551234" (7 digits, doesn't match 10)
  phoneNormalized = null (length < 11)

Test Case 6: Invalid - too long
  Input: "1234567890123456"
  phoneDigitsOnly = "1234567890123456"
  phoneWithCountryCode = "1234567890123456" (16 digits)
  phoneNormalized = null (length > 15)

Test Case 7: Null input
  Input: null
  phoneDigitsOnly = null
  phoneWithCountryCode = null
  phoneNormalized = null
```

---

### Key Learning Points

- ✅ Remove formatting systematically (multiple Replace transforms)
- ✅ Validate before processing
- ✅ Add missing data intelligently (country code)
- ✅ Validate output format
- ✅ Return null for invalid data (don't guess)
- ✅ Handle international formats

---

## Pattern 3: Complete Employee Profile Enrichment (40 minutes)

### Business Requirements
```
Enrich identity from multiple sources:

Sources:
  - Workday (HR): title, department, hire_date, manager_id
  - Active Directory: email, phone, groups
  - Salesforce: territory, quota, region
  - Badge System: building, floor, desk

Derived Attributes:
  - Employee level (from title + tenure)
  - Access level (from groups + title)
  - Location summary (building-floor-desk)
  - Profile completion percentage
  - Risk score
```

---

### Architecture Overview
```
Phase 1: Extract from sources (Account Attributes)
Phase 2: Calculate derived values (Conditionals + Lookups)
Phase 3: Aggregate into summary attributes
```

---

### Phase 1: Extract from Sources

#### Step 1: Get AD Email

##### UI Configuration
```
Attribute: emailFromAD

Transform: Account Attribute
  Source: Active Directory
  Attribute: mail
```

##### JSON Structure
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

#### Step 2: Get AD Phone

##### UI Configuration
```
Attribute: phoneFromAD

Transform: Account Attribute
  Source: Active Directory
  Attribute: telephoneNumber
```

##### JSON Structure
```json
{
  "name": "phoneFromAD",
  "type": "accountAttribute",
  "attributes": {
    "sourceName": "Active Directory",
    "attributeName": "telephoneNumber"
  }
}
```

---

#### Step 3: Get AD Groups

##### UI Configuration
```
Attribute: adGroups

Transform: Account Attribute
  Source: Active Directory
  Attribute: memberOf
  Output: Array of group DNs
```

##### JSON Structure
```json
{
  "name": "adGroups",
  "type": "accountAttribute",
  "attributes": {
    "sourceName": "Active Directory",
    "attributeName": "memberOf"
  }
}
```

---

#### Step 4: Get Salesforce Territory

##### UI Configuration
```
Attribute: salesTerritory

Transform: Account Attribute
  Source: Salesforce
  Attribute: Territory__c
```

##### JSON Structure
```json
{
  "name": "salesTerritory",
  "type": "accountAttribute",
  "attributes": {
    "sourceName": "Salesforce",
    "attributeName": "Territory__c"
  }
}
```

---

#### Step 5: Get Badge Location

##### UI Configuration
```
Attribute: badgeBuilding

Transform: Account Attribute
  Source: Badge System
  Attribute: building_code

Attribute: badgeFloor

Transform: Account Attribute
  Source: Badge System
  Attribute: floor_number

Attribute: badgeDesk

Transform: Account Attribute
  Source: Badge System
  Attribute: desk_id
```

##### JSON Structure
```json
{
  "name": "badgeBuilding",
  "type": "accountAttribute",
  "attributes": {
    "sourceName": "Badge System",
    "attributeName": "building_code"
  }
}
```

---

### Phase 2: Calculate Derived Values

#### Step 6: Calculate Employee Level

##### UI Configuration
```
Attribute: employeeLevel

Transform: Conditional Chain

  Level 1: Check for executives
    Condition: title contains "VP" OR title contains "Chief"
    If True: "Executive"
    If False: Continue

  Level 2: Check for directors
    Condition: title contains "Director"
    If True: "Director"
    If False: Continue

  Level 3: Check for managers
    Condition: title contains "Manager"
    If True: "Manager"
    If False: Continue

  Level 4: Check for seniors (title OR tenure)
    Condition: title contains "Senior" OR yearsOfService >= 5
    If True: "Senior"
    If False: "Individual Contributor"
```

##### JSON Structure
```json
{
  "name": "employeeLevel",
  "type": "conditional",
  "attributes": {
    "expression": "title.contains('VP') || title.contains('Chief')",
    "positiveCondition": "Executive",
    "negativeCondition": {
      "type": "conditional",
      "attributes": {
        "expression": "title.contains('Director')",
        "positiveCondition": "Director",
        "negativeCondition": {
          "type": "conditional",
          "attributes": {
            "expression": "title.contains('Manager')",
            "positiveCondition": "Manager",
            "negativeCondition": {
              "type": "conditional",
              "attributes": {
                "expression": "title.contains('Senior') || yearsOfService >= 5",
                "positiveCondition": "Senior",
                "negativeCondition": "Individual Contributor"
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

#### Step 7: Calculate Access Level

##### UI Configuration
```
Attribute: accessLevel

Transform: Conditional Chain

  Level 1: Check for admin groups
    Condition: adGroups contains "Admins"
    If True: "System Administrator"
    If False: Continue

  Level 2: Check for privileged access
    Condition: adGroups contains "PrivilegedAccess" OR employeeLevel = "Executive"
    If True: "Privileged"
    If False: Continue

  Level 3: Check for manager access
    Condition: employeeLevel = "Manager" OR employeeLevel = "Director"
    If True: "Manager"
    If False: "Standard"
```

##### JSON Structure
```json
{
  "name": "accessLevel",
  "type": "conditional",
  "attributes": {
    "expression": "adGroups.contains('Admins')",
    "positiveCondition": "System Administrator",
    "negativeCondition": {
      "type": "conditional",
      "attributes": {
        "expression": "adGroups.contains('PrivilegedAccess') || employeeLevel == 'Executive'",
        "positiveCondition": "Privileged",
        "negativeCondition": {
          "type": "conditional",
          "attributes": {
            "expression": "employeeLevel == 'Manager' || employeeLevel == 'Director'",
            "positiveCondition": "Manager",
            "negativeCondition": "Standard"
          }
        }
      }
    }
  }
}
```

---

#### Step 8: Create Location Summary

##### UI Configuration
```
Attribute: locationSummary

Transform: Conditional Chain

  Check if all location fields exist:
    Condition: badgeBuilding != null AND badgeFloor != null AND badgeDesk != null
    If True: Concatenate (building + "-" + floor + "-" + desk)
    If False: "Remote" or partial info
```

##### JSON Structure
```json
{
  "name": "locationSummary",
  "type": "conditional",
  "attributes": {
    "expression": "badgeBuilding != null && badgeFloor != null && badgeDesk != null",
    "positiveCondition": {
      "type": "concat",
      "attributes": {
        "values": [
          {
            "type": "identityAttribute",
            "attributes": {
              "name": "badgeBuilding"
            }
          },
          {
            "type": "static",
            "attributes": {
              "value": "-"
            }
          },
          {
            "type": "identityAttribute",
            "attributes": {
              "name": "badgeFloor"
            }
          },
          {
            "type": "static",
            "attributes": {
              "value": "-"
            }
          },
          {
            "type": "identityAttribute",
            "attributes": {
              "name": "badgeDesk"
            }
          }
        ]
      }
    },
    "negativeCondition": {
      "type": "static",
      "attributes": {
        "value": "Remote"
      }
    }
  }
}
```

---

### Phase 3: Calculate Profile Completion

#### Step 9: Profile Completion Score

##### UI Configuration
```
Attribute: profileCompletion

Logic (requires helper conditionals):

Helper: hasEmail
  Condition: emailFromAD != null
  True: 1, False: 0

Helper: hasPhone
  Condition: phoneFromAD != null
  True: 1, False: 0

Helper: hasTitle
  Condition: title != null
  True: 1, False: 0

Helper: hasDepartment
  Condition: department != null
  True: 1, False: 0

Helper: hasManager
  Condition: manager_id != null
  True: 1, False: 0

Helper: hasLocation
  Condition: badgeBuilding != null
  True: 1, False: 0

Then sum and calculate percentage...
```

> **Note:** Calculating percentages requires arithmetic that may not be available in standard transforms. This might need a rule or custom transform.

**Simplified Approach:**
```
Attribute: profileCompletionText

Transform: Conditional
  If all 6 fields exist: "Complete"
  Else if 4-5 fields exist: "Mostly Complete"
  Else if 2-3 fields exist: "Partially Complete"
  Else: "Incomplete"
```

---

### Complete Test Case
```
Employee: John Smith

Source Data:
  Workday:
    title = "Senior Account Executive"
    department = "Sales"
    hire_date = "2020-01-15"
    manager_id = "MGR123"

  Active Directory:
    mail = "john.smith@company.com"
    telephoneNumber = "214-555-1234"
    memberOf = ["CN=Sales,OU=Groups", "CN=ManagerAccess,OU=Groups"]

  Salesforce:
    Territory__c = "West Region"
    Quota__c = "500000"

  Badge System:
    building_code = "HQ1"
    floor_number = "3"
    desk_id = "3042"

Derived Attributes:
  emailFromAD = "john.smith@company.com"
  phoneFromAD = "214-555-1234"
  adGroups = ["CN=Sales,OU=Groups", "CN=ManagerAccess,OU=Groups"]
  salesTerritory = "West Region"
  badgeBuilding = "HQ1"
  badgeFloor = "3"
  badgeDesk = "3042"
  employeeLevel = "Senior" (contains "Senior" in title)
  accessLevel = "Manager" (has ManagerAccess group)
  locationSummary = "HQ1-3-3042"
  profileCompletion = "Complete" (all fields present)
```

---

### Key Learning Points

- ✅ Account Attribute extracts from multiple sources
- ✅ Derive intelligence from raw data
- ✅ Combine cross-source data logically
- ✅ Use conditionals for classification
- ✅ Create human-readable summaries
- ✅ Handle missing data gracefully

---

## Pattern 4: Dynamic Access Level Calculator (30 minutes)

### Business Requirements
```
Calculate access level from multiple factors:

Factors:
  - Job title
  - AD group memberships
  - Department
  - Years of service
  - Security clearance
  - Training completion

Access Levels (highest wins):
  1. System Administrator
  2. Privileged User
  3. Manager
  4. Senior Employee
  5. Standard Employee
  6. Contractor
  7. Restricted
```

---

### Architecture Overview
```
Helper Attributes (Boolean Checks):
  - isInAdminGroup
  - hasPrivilegedAccess
  - isManager
  - hasSeniorTitle
  - hasLongTenure
  - hasSecurityClearance
  - hasCompletedTraining

Final Attribute:
  - accessLevel (uses all helpers)
```

---

### Step 1: Create Boolean Helpers

#### Helper: isInAdminGroup

##### UI Configuration
```
Attribute: isInAdminGroup

Transform: Conditional
  Condition: adGroups contains "Admins"
  If True: true
  If False: false
```

##### JSON Structure
```json
{
  "name": "isInAdminGroup",
  "type": "conditional",
  "attributes": {
    "expression": "adGroups.contains('Admins')",
    "positiveCondition": "true",
    "negativeCondition": "false"
  }
}
```

---

#### Helper: hasPrivilegedAccess

##### UI Configuration
```
Attribute: hasPrivilegedAccess

Transform: Conditional
  Condition: adGroups contains "PrivilegedAccess" OR department = "Security"
  If True: true
  If False: false
```

##### JSON Structure
```json
{
  "name": "hasPrivilegedAccess",
  "type": "conditional",
  "attributes": {
    "expression": "adGroups.contains('PrivilegedAccess') || department == 'Security'",
    "positiveCondition": "true",
    "negativeCondition": "false"
  }
}
```

---

#### Helper: isManager

##### UI Configuration
```
Attribute: isManager

Transform: Conditional
  Condition: title contains "Manager" OR title contains "Director" OR adGroups contains "Managers"
  If True: true
  If False: false
```

##### JSON Structure
```json
{
  "name": "isManager",
  "type": "conditional",
  "attributes": {
    "expression": "title.contains('Manager') || title.contains('Director') || adGroups.contains('Managers')",
    "positiveCondition": "true",
    "negativeCondition": "false"
  }
}
```

---

### Step 2: Build Access Level Decision Tree

#### UI Configuration
```
Attribute: accessLevel

Transform: Conditional Chain (Priority Order)

  Level 1: System Administrator
    Condition: isInAdminGroup = true
    If True: "System Administrator"
    If False: Continue

  Level 2: Privileged User
    Condition: hasPrivilegedAccess = true
    If True: "Privileged User"
    If False: Continue

  Level 3: Manager
    Condition: isManager = true
    If True: "Manager"
    If False: Continue

  Level 4: Senior Employee
    Condition: (title contains "Senior" OR yearsOfService >= 5) AND employee_type = "FTE"
    If True: "Senior Employee"
    If False: Continue

  Level 5: Standard Employee
    Condition: employee_type = "FTE"
    If True: "Standard Employee"
    If False: Continue

  Level 6: Contractor
    Condition: employee_type = "Contractor"
    If True: "Contractor"
    If False: "Restricted"
```

---

#### JSON Structure
```json
{
  "name": "accessLevel",
  "type": "conditional",
  "attributes": {
    "expression": "isInAdminGroup == 'true'",
    "positiveCondition": "System Administrator",
    "negativeCondition": {
      "type": "conditional",
      "attributes": {
        "expression": "hasPrivilegedAccess == 'true'",
        "positiveCondition": "Privileged User",
        "negativeCondition": {
          "type": "conditional",
          "attributes": {
            "expression": "isManager == 'true'",
            "positiveCondition": "Manager",
            "negativeCondition": {
              "type": "conditional",
              "attributes": {
                "expression": "(title.contains('Senior') || yearsOfService >= 5) && employee_type == 'FTE'",
                "positiveCondition": "Senior Employee",
                "negativeCondition": {
                  "type": "conditional",
                  "attributes": {
                    "expression": "employee_type == 'FTE'",
                    "positiveCondition": "Standard Employee",
                    "negativeCondition": {
                      "type": "conditional",
                      "attributes": {
                        "expression": "employee_type == 'Contractor'",
                        "positiveCondition": "Contractor",
                        "negativeCondition": "Restricted"
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

### Complete Test Cases
```
Test Case 1: System Administrator
  adGroups contains "Admins" = true
  isInAdminGroup = true
  accessLevel = "System Administrator"

Test Case 2: Privileged User
  adGroups does NOT contain "Admins"
  department = "Security"
  hasPrivilegedAccess = true
  accessLevel = "Privileged User"

Test Case 3: Manager
  title = "Engineering Manager"
  isManager = true
  accessLevel = "Manager"

Test Case 4: Senior Employee
  title = "Senior Engineer"
  employee_type = "FTE"
  yearsOfService = 6
  accessLevel = "Senior Employee"

Test Case 5: Standard Employee
  title = "Engineer"
  employee_type = "FTE"
  yearsOfService = 2
  accessLevel = "Standard Employee"

Test Case 6: Contractor
  employee_type = "Contractor"
  accessLevel = "Contractor"

Test Case 7: Restricted
  employee_type = "Intern"
  accessLevel = "Restricted" (falls through all checks)
```

---

### Key Learning Points

- ✅ Boolean helpers simplify complex conditions
- ✅ Priority order matters (check highest first)
- ✅ Multiple factors contribute to single decision
- ✅ First match wins (short-circuit evaluation)
- ✅ Always have final fallback
- ✅ Testable logic (each helper independent)

---

## Pattern 5: Intelligent Manager Resolution (30 minutes)

### Business Requirements
```
Resolve manager information with comprehensive fallback:

Priority Order:
1. Manager's email from their AD account
2. Generated email from manager's name
3. Manager's email from HR system
4. Department head email (lookup)
5. Default manager email

Also resolve:
- Manager's display name
- Manager's title
- Manager's department
- Skip-level manager (manager's manager)
- Manager phone
```

---

### Architecture Overview
```
Phase 1: Get manager's basic info
Phase 2: Get manager's email (5 fallback levels)
Phase 3: Get manager's other attributes
Phase 4: Get skip-level manager
```

---

### Phase 1: Manager Basic Info

#### Step 1: Manager Display Name

##### UI Configuration
```
Attribute: managerDisplayName

Transform: Identity Attribute
  Attribute: manager.displayName
  (References the manager identity's displayName)
```

##### JSON Structure
```json
{
  "name": "managerDisplayName",
  "type": "identityAttribute",
  "attributes": {
    "name": "manager.displayName"
  }
}
```

> **Note:** Syntax for referencing manager attributes varies by ISC version. Verify in your environment.

---

#### Step 2: Manager Title

##### UI Configuration
```
Attribute: managerTitle

Transform: Identity Attribute
  Attribute: manager.title
```

##### JSON Structure
```json
{
  "name": "managerTitle",
  "type": "identityAttribute",
  "attributes": {
    "name": "manager.title"
  }
}
```

---

### Phase 2: Manager Email with Fallbacks

#### Helper 1: Manager Email from AD

##### UI Configuration
```
Attribute: managerEmailFromAD

Transform: Account Attribute
  Source: Active Directory
  Identity: manager
  Attribute: mail
```

##### JSON Structure
```json
{
  "name": "managerEmailFromAD",
  "type": "accountAttribute",
  "attributes": {
    "sourceName": "Active Directory",
    "identityAttribute": "manager",
    "attributeName": "mail"
  }
}
```

---

#### Helper 2: Generated Manager Email

##### UI Configuration
```
Attribute: managerEmailGenerated

Transform Chain:
  Step 1: Conditional (check manager.firstname exists)
  Step 2: Conditional (check manager.lastname exists)
  Step 3: Concatenate (manager.firstname + "." + manager.lastname)
  Step 4: Lower
  Step 5: Concatenate (result + "@company.com")
```

##### JSON Structure
```json
{
  "name": "managerEmailGenerated",
  "type": "conditional",
  "attributes": {
    "expression": "manager.firstname != null && manager.lastname != null",
    "positiveCondition": {
      "type": "concat",
      "attributes": {
        "values": [
          {
            "type": "lower",
            "attributes": {
              "input": {
                "type": "identityAttribute",
                "attributes": {
                  "name": "manager.firstname"
                }
              }
            }
          },
          {
            "type": "static",
            "attributes": {
              "value": "."
            }
          },
          {
            "type": "lower",
            "attributes": {
              "input": {
                "type": "identityAttribute",
                "attributes": {
                  "name": "manager.lastname"
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
      }
    },
    "negativeCondition": null
  }
}
```

---

#### Helper 3: Manager Email from HR

##### UI Configuration
```
Attribute: managerEmailFromHR

Transform: Identity Attribute
  Attribute: manager.workEmail
  (Assuming HR system populates workEmail)
```

##### JSON Structure
```json
{
  "name": "managerEmailFromHR",
  "type": "identityAttribute",
  "attributes": {
    "name": "manager.workEmail"
  }
}
```

---

#### Helper 4: Department Head Email

##### UI Configuration
```
Attribute: deptHeadEmail

Transform: Lookup
  Input: department (employee's department)
  Table: DepartmentHeadEmails
  Default: null
```

##### JSON Structure
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
      "id": "department-head-emails-table-id"
    }
  }
}
```

---

#### Final: Manager Email

##### UI Configuration
```
Attribute: managerEmail

Transform: FirstValid
  Priority 1: managerEmailFromAD
  Priority 2: managerEmailGenerated
  Priority 3: managerEmailFromHR
  Priority 4: deptHeadEmail
  Priority 5: Static "no-manager@company.com"
```

##### JSON Structure
```json
{
  "name": "managerEmail",
  "type": "firstValid",
  "attributes": {
    "values": [
      {
        "type": "identityAttribute",
        "attributes": {
          "name": "managerEmailFromAD"
        }
      },
      {
        "type": "identityAttribute",
        "attributes": {
          "name": "managerEmailGenerated"
        }
      },
      {
        "type": "identityAttribute",
        "attributes": {
          "name": "managerEmailFromHR"
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

### Phase 3: Skip-Level Manager

#### UI Configuration
```
Attribute: skipLevelManagerName

Transform: Identity Attribute
  Attribute: manager.manager.displayName
  (Chain through manager to their manager)
```

##### JSON Structure
```json
{
  "name": "skipLevelManagerName",
  "type": "identityAttribute",
  "attributes": {
    "name": "manager.manager.displayName"
  }
}
```

---
```
Attribute: skipLevelManagerEmail

Transform: Account Attribute
  Source: Active Directory
  Identity: manager.manager
  Attribute: mail
```

##### JSON Structure
```json
{
  "name": "skipLevelManagerEmail",
  "type": "accountAttribute",
  "attributes": {
    "sourceName": "Active Directory",
    "identityAttribute": "manager.manager",
    "attributeName": "mail"
  }
}
```

---

### Complete Test Cases
```
Test Case 1: Manager with AD account
  Manager: Jane Doe
  Manager has AD account with mail = "jane.doe@company.com"
  
  Results:
    managerDisplayName = "Jane Doe"
    managerTitle = "Director of Sales"
    managerEmailFromAD = "jane.doe@company.com"
    managerEmail = "jane.doe@company.com" (Priority 1)

Test Case 2: Manager without AD, but has name
  Manager: John Manager
  No AD account
  manager.firstname = "John"
  manager.lastname = "Manager"
  
  Results:
    managerEmailFromAD = null
    managerEmailGenerated = "john.manager@company.com"
    managerEmail = "john.manager@company.com" (Priority 2)

Test Case 3: No manager, use department head
  Manager: null
  Employee department = "Engineering"
  Lookup: Engineering -> "eng-director@company.com"
  
  Results:
    managerDisplayName = null
    managerEmailFromAD = null
    managerEmailGenerated = null
    managerEmailFromHR = null
    deptHeadEmail = "eng-director@company.com"
    managerEmail = "eng-director@company.com" (Priority 4)

Test Case 4: Skip-level manager
  Manager: Jane Doe
  Jane's Manager: Bob Johnson (VP)
  
  Results:
    skipLevelManagerName = "Bob Johnson"
    skipLevelManagerEmail = "bob.johnson@company.com"
```

---

### Key Learning Points

- ✅ FirstValid perfect for priority fallback
- ✅ Multiple strategies ensure always have manager contact
- ✅ Identity Attribute chains through relationships
- ✅ Account Attribute from related identities
- ✅ Department head good backup
- ✅ Skip-level useful for escalations
- ✅ Null handling at every level

---

## Day 5 Final Assessment

### What You Built Today

**Morning Session:**
- [ ] Advanced chaining strategies
- [ ] Performance optimization techniques
- [ ] Transforms vs Rules decision framework
- [ ] Debugging methodology

**Afternoon Session:**
- [ ] Smart username generator (10-step chain)
- [ ] International phone normalizer (E.164 format)
- [ ] Complete employee enrichment (multi-source)
- [ ] Dynamic access level calculator (boolean helpers)
- [ ] Intelligent manager resolution (5-level fallback)

**Total: 5 production-ready enterprise patterns!**

---

### Concepts Mastered

- ✅ UI configuration approach
- ✅ JSON structure understanding
- ✅ Helper attribute pattern
- ✅ Incremental building strategy
- ✅ FirstValid for prioritization
- ✅ Boolean helper conditionals
- ✅ Cross-source data aggregation
- ✅ Multi-level fallback chains
- ✅ Complex string manipulation
- ✅ International data handling
- ✅ Performance best practices
- ✅ Production-ready patterns

---

## Homework Challenges

### Challenge 1: Smart Email with Domain Validation

Build email generator that:
- Generates from name
- Validates domain (lookup allowed domains)
- Handles subdomains (sales.company.com vs company.com)
- Adds country code prefix for international (uk.firstname.lastname@company.com)

---

### Challenge 2: Risk Score Calculator

Build risk score based on:
- Failed login attempts (from AD)
- Access level
- Data classification access
- External user flag
- Account age
- Training completion

Output: Critical, High, Medium, Low

---

### Challenge 3: Complete Org Chart Data

Build complete org chart attributes:
- Manager (level 1)
- Skip-level (level 2)
- VP (level 3)
- Division head (level 4)
- CEO (level 5)

With emails and names for each level.

---

## Day 5 Complete!

### You've Mastered

- ✅ Advanced transform chaining
- ✅ UI and JSON understanding
- ✅ Production-ready patterns
- ✅ Performance optimization
- ✅ Enterprise-level complexity
- ✅ Real-world scenarios
- ✅ Helper attribute strategies
- ✅ Multi-source aggregation
- ✅ Comprehensive error handling
- ✅ International data support

---

## What's Next: Day 6 Preview

Tomorrow you'll learn:

### Troubleshooting & Debugging Deep Dive

**All Day Session:**
- Identity Preview mastery
- Common error patterns (20+ scenarios)
- Troubleshooting playbook
- Fixing broken transforms
- Performance debugging
- Testing strategies
- Edge case handling
- Production validation
- Rollback procedures

**You'll become an expert at diagnosing and fixing transform issues!**

---

**Excellent work on Day 5! You've built enterprise-grade transform solutions!**

---

*Part of the SailPoint ISC Transforms Mastery 7-Day Study Plan*
