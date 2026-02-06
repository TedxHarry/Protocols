# DAY 7: CAPSTONE PROJECT - COMPLETE EMPLOYEE ONBOARDING SUITE

## Table of Contents
- [Project Overview](#project-overview)
- [Project Requirements](#project-requirements)
- [Phase 1: Planning & Design](#phase-1-planning--design-60-minutes)
- [Phase 2: Build Core Transforms](#phase-2-build-core-transforms-120-minutes)
- [Phase 3: Build Advanced Transforms](#phase-3-build-advanced-transforms-120-minutes)
- [Phase 4: Testing & Validation](#phase-4-testing--validation-90-minutes)
- [Phase 5: Documentation](#phase-5-documentation-60-minutes)
- [Phase 6: Final Review](#phase-6-final-review-30-minutes)
- [Congratulations](#congratulations)

---

## Project Overview

**Session Duration:** Full Day (7 hours)

**Goal:** Build a complete, production-ready employee onboarding transform suite that handles all edge cases

**What You're Building:**

A comprehensive identity attribute transformation system for new employee onboarding that:
- Handles messy HR data
- Generates all required identity attributes
- Includes proper error handling
- Works for all employee types
- Is fully documented and tested

---

## Project Requirements

### Business Context

**Scenario:** Your company is implementing SailPoint ISC for identity management. The HR system (Workday) provides employee data in various formats with inconsistent quality. You need to transform this raw data into clean, standardized identity attributes.

---

### Source Data (From Workday)
```
Available Attributes:
- employee_id: "E12345"
- firstname: "John" (may have trailing spaces, hyphens, apostrophes)
- lastname: "Smith" (may have trailing spaces, hyphens, apostrophes)
- middlename: "Michael" (optional, may be null)
- personal_email: "john.smith@gmail.com" (optional)
- work_phone: "(214) 555-1234" (various formats)
- mobile_phone: "214-555-9876" (various formats)
- hire_date: "01/15/2024" (MM/dd/yyyy format)
- term_date: "12/31/2024" (MM/dd/yyyy format, may be null)
- employee_type: "FTE" or "Contractor" or "Intern"
- status: "Active" or "On Leave" or "Inactive"
- dept_code: "ENG", "FIN", "HR", etc. (3-letter codes)
- job_title: "Senior Software Engineer" (free text, many variations)
- manager_id: "E54321" (employee_id of manager)
- office_location: "Austin" or "Austin Office - Building 2"
- country_code: "US", "UK", "JP", etc.
```

---

### Required Output Attributes

You must create these 9 attributes:
```
1. displayName
   Format: "Lastname, Firstname"
   Example: "Smith, John"

2. email
   Format: firstname.lastname@company.com (lowercase)
   Max length: 50 characters
   Fallback strategy with 5 levels

3. username
   Format: firstname.lastname (lowercase)
   Max length: 20 characters
   No special characters

4. department
   Full department name from dept_code
   Example: "ENG" → "Engineering"

5. title
   Standardized job title
   Example: "Sr. Software Engineer" → "Senior Engineer"

6. phone
   E.164 international format
   Example: "+12145551234"

7. timezone
   From office_location
   Example: "Austin" → "America/Chicago"

8. lifecycleState
   One of: Pre-hire, Active, Leave of Absence, Terminated, Archived
   Based on dates and status

9. managerEmail
   Manager's email with 5-level fallback
```

---

### Success Criteria

Your solution must:
```
✓ Handle all 10 edge case scenarios (defined below)
✓ Have zero errors with valid data
✓ Gracefully handle missing data (nulls)
✓ Be performant (benchmarked)
✓ Be fully documented
✓ Include test results for all scenarios
✓ Have rollback plan documented
```

---

## Phase 1: Planning & Design (60 minutes)

### Step 1: Analyze Dependencies

#### Transform Dependency Map
```
Create a diagram showing which transforms depend on which:

Helper Attributes (Built First):
├─ hireDateISO (needed by lifecycleState)
├─ termDateISO (needed by lifecycleState)
├─ cleanFirstname (needed by displayName, email, username)
├─ cleanLastname (needed by displayName, email, username)
└─ deptCodeUpper (needed by department lookup)

Core Attributes (Built Second):
├─ displayName (uses cleanFirstname, cleanLastname)
├─ email (uses cleanFirstname, cleanLastname)
├─ username (uses cleanFirstname, cleanLastname)
├─ department (uses deptCodeUpper)
├─ title (uses job_title)
├─ phone (uses work_phone or mobile_phone)
└─ timezone (uses office_location)

Advanced Attributes (Built Third):
├─ lifecycleState (uses hireDateISO, termDateISO, status)
└─ managerEmail (uses manager reference + multiple fallbacks)
```

---

### Step 2: Design Lookup Tables

#### Required Lookup Tables

**Table 1: Department Code Mapping**

| Input Code | Output Name |
|------------|-------------|
| ENG | Engineering |
| FIN | Finance |
| HR | Human Resources |
| IT | Information Technology |
| SALES | Sales |
| MKT | Marketing |
| OPS | Operations |
| LEGAL | Legal |
| CS | Customer Success |
| PROD | Product Management |

**Table 2: Title Standardization**

| Input Title | Output Standard Title |
|-------------|----------------------|
| Sr. Software Engineer | Senior Engineer |
| Senior Software Engineer | Senior Engineer |
| Staff Software Engineer | Senior Engineer |
| Software Engineer | Engineer |
| Jr. Software Engineer | Junior Engineer |
| Engineering Manager | Manager |
| Sr. Engineering Manager | Senior Manager |
| Director of Engineering | Director |
| VP Engineering | Vice President |
| Analyst | Analyst |
| Sr. Analyst | Senior Analyst |
| ... (add 20+ more variations) | ... |

**Table 3: Location to Timezone**

| Input Location | Output Timezone |
|----------------|----------------|
| Austin | America/Chicago |
| Dallas | America/Chicago |
| New York | America/New_York |
| London | Europe/London |
| Tokyo | Asia/Tokyo |
| Sydney | Australia/Sydney |
| ... (add more) | ... |

**Table 4: Department Head Emails** (for manager fallback)

| Department | Head Email |
|------------|------------|
| Engineering | eng-director@company.com |
| Finance | finance-director@company.com |
| HR | hr-director@company.com |
| ... | ... |

---

### Step 3: Design Test Scenarios

#### 10 Required Edge Case Test Scenarios
```
Test Scenario 1: Perfect Data
  All fields populated correctly
  No special characters
  Normal name length
  Expected: All attributes generate correctly

Test Scenario 2: Missing Personal Email
  personal_email = null
  Expected: email generates from name

Test Scenario 3: Very Long Names
  firstname = "Christopher"
  lastname = "Vanderbilt-Johnson"
  Expected: username truncated to 20 chars

Test Scenario 4: Special Characters in Name
  firstname = "Mary-Ann"
  lastname = "O'Brien"
  Expected: Special chars removed in username/email

Test Scenario 5: Missing Lastname
  firstname = "Madonna"
  lastname = null
  Expected: Handle gracefully (use firstname only)

Test Scenario 6: Pre-hire Employee
  hire_date = future date
  Expected: lifecycleState = "Pre-hire"

Test Scenario 7: Terminated Employee (Recent)
  term_date = 30 days ago
  Expected: lifecycleState = "Terminated"

Test Scenario 8: Terminated Employee (Old)
  term_date = 120 days ago
  Expected: lifecycleState = "Archived"

Test Scenario 9: Employee on Leave
  status = "On Leave"
  Expected: lifecycleState = "Leave of Absence"

Test Scenario 10: Missing Manager
  manager_id = null
  Expected: managerEmail falls back to department head
```

---

### Step 4: Create Project Plan Document

#### UI Planning Document Template
```markdown
# Employee Onboarding Transform Suite - Project Plan

## Phase 1: Lookup Tables
- [ ] Create DepartmentCodeMapping table
- [ ] Create TitleStandardization table
- [ ] Create LocationToTimezone table
- [ ] Create DepartmentHeadEmails table

## Phase 2: Helper Attributes
- [ ] Build cleanFirstname
- [ ] Build cleanLastname
- [ ] Build hireDateISO
- [ ] Build termDateISO
- [ ] Build deptCodeUpper

## Phase 3: Core Attributes
- [ ] Build displayName
- [ ] Build email (with fallbacks)
- [ ] Build username
- [ ] Build department
- [ ] Build title
- [ ] Build phone
- [ ] Build timezone

## Phase 4: Advanced Attributes
- [ ] Build lifecycleState
- [ ] Build managerEmail

## Phase 5: Testing
- [ ] Test all 10 edge case scenarios
- [ ] Document test results
- [ ] Fix any issues found

## Phase 6: Documentation
- [ ] Document each transform
- [ ] Create troubleshooting guide
- [ ] Create deployment plan
```

---

## Phase 2: Build Core Transforms (120 minutes)

### Transform 1: cleanFirstname (Helper)

#### UI Configuration
```
Attribute: cleanFirstname

Purpose: Clean and normalize firstname for use in other transforms

Transform Chain:
  Step 1: Conditional
    Condition: firstname is not null
    If True: Continue
    If False: Return "" (empty string)

  Step 2: Trim
    Input: firstname
    Output: Remove leading/trailing spaces

  Step 3: Replace "-" with ""
    Input: Previous output
    Output: Remove hyphens

  Step 4: Replace "'" with ""
    Input: Previous output
    Output: Remove apostrophes

  Step 5: Replace " " with ""
    Input: Previous output
    Output: Remove spaces (for middle names)

  Step 6: Lower
    Input: Previous output
    Output: All lowercase

Expected Output Examples:
  "John" → "john"
  "Mary-Ann" → "maryann"
  "O'Brien" → "obrien"
  null → ""
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

#### Test This Transform
```
Test Data 1:
  firstname = "John"
  Expected: "john"
  Actual: [Use Preview]
  Status: PASS/FAIL

Test Data 2:
  firstname = "Mary-Ann"
  Expected: "maryann"
  Actual: [Use Preview]
  Status: PASS/FAIL

Test Data 3:
  firstname = null
  Expected: ""
  Actual: [Use Preview]
  Status: PASS/FAIL
```

---

### Transform 2: cleanLastname (Helper)

#### UI Configuration
```
Attribute: cleanLastname

Purpose: Clean and normalize lastname for use in other transforms

Transform Chain:
  (Same as cleanFirstname, but using lastname as input)

  Step 1: Conditional (lastname != null)
  Step 2: Trim
  Step 3: Replace "-"
  Step 4: Replace "'"
  Step 5: Replace " "
  Step 6: Lower
```

---

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

### Transform 3: displayName

#### UI Configuration
```
Attribute: displayName

Purpose: Create formatted display name as "Lastname, Firstname"

Transform Chain:
  Step 1: Conditional
    Condition: lastname is not null AND firstname is not null
    If True: Continue
    If False: Use fallback logic

  Step 2: Concatenation (if both exist)
    Values:
      - lastname (original, not cleaned - keep proper case)
      - ", " (comma and space)
      - firstname (original, not cleaned - keep proper case)
    Output: "Smith, John"

  Fallback (if Step 1 = False):
    Step 3: Conditional
      Condition: firstname is not null
      If True: firstname only
      If False: lastname only (or "Unknown" if both null)

Expected Output Examples:
  firstname="John", lastname="Smith" → "Smith, John"
  firstname="Madonna", lastname=null → "Madonna"
  firstname=null, lastname="Prince" → "Prince"
  firstname=null, lastname=null → "Unknown"
```

---

#### JSON Structure
```json
{
  "name": "displayName",
  "type": "conditional",
  "attributes": {
    "expression": "lastname != null && firstname != null",
    "positiveCondition": {
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
    },
    "negativeCondition": {
      "type": "conditional",
      "attributes": {
        "expression": "firstname != null",
        "positiveCondition": {
          "type": "identityAttribute",
          "attributes": {
            "name": "firstname"
          }
        },
        "negativeCondition": {
          "type": "conditional",
          "attributes": {
            "expression": "lastname != null",
            "positiveCondition": {
              "type": "identityAttribute",
              "attributes": {
                "name": "lastname"
              }
            },
            "negativeCondition": {
              "type": "static",
              "attributes": {
                "value": "Unknown"
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

### Transform 4: username

#### UI Configuration
```
Attribute: username

Purpose: Generate username as firstname.lastname (lowercase, max 20 chars)

Transform Chain:
  Step 1: Conditional
    Condition: cleanFirstname != "" AND cleanLastname != ""
    If True: Concatenate both
    If False: Use fallback

  Step 2: Concatenation (if both exist)
    Values:
      - cleanFirstname
      - "."
      - cleanLastname
    Output: "john.smith"

  Step 3: Substring (truncate to 20)
    Begin: 0
    Length: 20
    Output: First 20 characters

  Fallback logic:
    If only firstname: use cleanFirstname
    If only lastname: use cleanLastname
    If neither: "user" + employee_id

Expected Output Examples:
  "john.smith" → "john.smith" (10 chars, no truncation)
  "christopher.vanderbiltjohnson" → "christopher.vanderbi" (truncated)
  firstname only → "john"
  neither → "userE12345"
```

---

#### JSON Structure
```json
{
  "name": "username",
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
        ],
        "next": {
          "type": "substring",
          "attributes": {
            "begin": 0,
            "end": 20
          }
        }
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
              "type": "concat",
              "attributes": {
                "values": [
                  {
                    "type": "static",
                    "attributes": {
                      "value": "user"
                    }
                  },
                  {
                    "type": "identityAttribute",
                    "attributes": {
                      "name": "employee_id"
                    }
                  }
                ]
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

### Transform 5: email (Complex with 5 Fallback Levels)

#### Helper Attribute: emailFromName

##### UI Configuration
```
Attribute: emailFromName

Purpose: Generate email from name (firstname.lastname@company.com)

Transform Chain:
  Step 1: Conditional
    Condition: cleanFirstname != "" AND cleanLastname != ""
    If True: Continue
    If False: Return null

  Step 2: Concatenation
    Values:
      - cleanFirstname
      - "."
      - cleanLastname
      - "@company.com"
    Output: "john.smith@company.com"

  Step 3: Substring (ensure max 50 chars)
    Begin: 0
    Length: 50
```

---

##### JSON Structure
```json
{
  "name": "emailFromName",
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
          },
          {
            "type": "static",
            "attributes": {
              "value": "@company.com"
            }
          }
        ],
        "next": {
          "type": "substring",
          "attributes": {
            "begin": 0,
            "end": 50
          }
        }
      }
    },
    "negativeCondition": null
  }
}
```

---

#### Helper Attribute: emailFromID

##### UI Configuration
```
Attribute: emailFromID

Purpose: Generate email from employee_id (e12345@company.com)

Transform Chain:
  Step 1: Conditional
    Condition: employee_id is not null
    If True: Continue
    If False: Return null

  Step 2: Lower
    Input: employee_id
    Output: "e12345"

  Step 3: Concatenation
    Values:
      - employee_id (lowercased)
      - "@company.com"
    Output: "e12345@company.com"
```

---

##### JSON Structure
```json
{
  "name": "emailFromID",
  "type": "conditional",
  "attributes": {
    "expression": "employee_id != null",
    "positiveCondition": {
      "type": "lower",
      "attributes": {
        "input": {
          "type": "identityAttribute",
          "attributes": {
            "name": "employee_id"
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

#### Final Attribute: email

##### UI Configuration
```
Attribute: email

Purpose: Get email with 5-level fallback strategy

Transform: FirstValid
  Priority 1: personal_email (from source, if provided)
  Priority 2: emailFromName (generated from firstname.lastname)
  Priority 3: emailFromID (generated from employee_id)
  Priority 4: username + "@company.com" (as last resort)
  Priority 5: Static "noemail@company.com" (absolute fallback)
```

---

##### JSON Structure
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
          "name": "emailFromName"
        }
      },
      {
        "type": "identityAttribute",
        "attributes": {
          "name": "emailFromID"
        }
      },
      {
        "type": "concat",
        "attributes": {
          "values": [
            {
              "type": "identityAttribute",
              "attributes": {
                "name": "username"
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

---

### Transform 6: department

#### Helper Attribute: deptCodeUpper

##### UI Configuration
```
Attribute: deptCodeUpper

Purpose: Normalize department code to uppercase for lookup

Transform Chain:
  Step 1: Trim
    Input: dept_code

  Step 2: Upper
    Input: Previous output
    Output: "ENG" (from "eng" or " eng ")
```

---

##### JSON Structure
```json
{
  "name": "deptCodeUpper",
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

---

#### Final Attribute: department

##### UI Configuration
```
Attribute: department

Purpose: Translate department code to full name

Transform: Lookup
  Input: deptCodeUpper
  Table: DepartmentCodeMapping
  Default: "Other"

Expected Output Examples:
  "ENG" → "Engineering"
  "FIN" → "Finance"
  "UNKNOWN" → "Other" (default)
```

---

##### JSON Structure
```json
{
  "name": "department",
  "type": "lookup",
  "attributes": {
    "input": {
      "type": "identityAttribute",
      "attributes": {
        "name": "deptCodeUpper"
      }
    },
    "table": {
      "id": "department-code-mapping-table-id"
    },
    "default": "Other"
  }
}
```

---

### Transform 7: title

#### UI Configuration
```
Attribute: title

Purpose: Standardize job title variations

Transform: Lookup
  Input: job_title
  Table: TitleStandardization
  Default: job_title (keep as-is if not in table)

Expected Output Examples:
  "Sr. Software Engineer" → "Senior Engineer"
  "Engineering Manager" → "Manager"
  "Some Weird Title" → "Some Weird Title" (not in table, keep original)
```

---

#### JSON Structure
```json
{
  "name": "title",
  "type": "lookup",
  "attributes": {
    "input": {
      "type": "identityAttribute",
      "attributes": {
        "name": "job_title"
      }
    },
    "table": {
      "id": "title-standardization-table-id"
    },
    "default": {
      "type": "identityAttribute",
      "attributes": {
        "name": "job_title"
      }
    }
  }
}
```

---

## Phase 3: Build Advanced Transforms (120 minutes)

### Transform 8: phone (E.164 Format)

#### Helper Attribute: phoneDigitsOnly

##### UI Configuration
```
Attribute: phoneDigitsOnly

Purpose: Extract only digits from phone number

Transform Chain:
  Step 1: FirstValid (try work_phone, then mobile_phone)
    Values:
      - work_phone
      - mobile_phone

  Step 2: Trim

  Step 3: Replace "(" with ""
  Step 4: Replace ")" with ""
  Step 5: Replace " " with ""
  Step 6: Replace "-" with ""
  Step 7: Replace "." with ""
  Step 8: Replace "+" with ""

  Output: "2145551234" (digits only)
```

---

##### JSON Structure
```json
{
  "name": "phoneDigitsOnly",
  "type": "firstValid",
  "attributes": {
    "values": [
      {
        "type": "identityAttribute",
        "attributes": {
          "name": "work_phone"
        }
      },
      {
        "type": "identityAttribute",
        "attributes": {
          "name": "mobile_phone"
        }
      }
    ],
    "next": {
      "type": "trim",
      "attributes": {
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
    }
  }
}
```

---

#### Final Attribute: phone

##### UI Configuration
```
Attribute: phone

Purpose: Format phone in E.164 (+12145551234)

Transform Chain:
  Step 1: Conditional
    Condition: phoneDigitsOnly length >= 11
    If True: Already has country code
    If False: Add country code

  Step 2: Conditional (if length = 10, add "1")
    Condition: phoneDigitsOnly length = 10
    If True: Concatenate "1" + phoneDigitsOnly
    If False: phoneDigitsOnly

  Step 3: Concatenate "+" + result

  Step 4: Validate length (11-15 digits)
    If valid: Keep result
    If invalid: Return null

Expected Output Examples:
  "(214) 555-1234" → "+12145551234"
  "+1 214-555-1234" → "+12145551234"
  "+44 20 7123 4567" → "+442071234567"
```

---

##### JSON Structure
```json
{
  "name": "phone",
  "type": "conditional",
  "attributes": {
    "expression": "phoneDigitsOnly.length() >= 11",
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
              "name": "phoneDigitsOnly"
            }
          }
        ]
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
                  "value": "+1"
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
        "negativeCondition": null
      }
    }
  }
}
```

---

### Transform 9: timezone

#### Helper Attribute: cityFromLocation

##### UI Configuration
```
Attribute: cityFromLocation

Purpose: Extract city from office_location string

Transform Chain:
  Step 1: Split by " "
    Input: "Austin Office - Building 2"
    Output: ["Austin", "Office", "-", "Building", "2"]

  Step 2: Index (position 0)
    Output: "Austin"
```

---

##### JSON Structure
```json
{
  "name": "cityFromLocation",
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

---

#### Final Attribute: timezone

##### UI Configuration
```
Attribute: timezone

Purpose: Map city to timezone

Transform: Lookup
  Input: cityFromLocation
  Table: LocationToTimezone
  Default: "UTC"

Expected Output Examples:
  "Austin" → "America/Chicago"
  "London" → "Europe/London"
  "Unknown" → "UTC" (default)
```

---

##### JSON Structure
```json
{
  "name": "timezone",
  "type": "lookup",
  "attributes": {
    "input": {
      "type": "identityAttribute",
      "attributes": {
        "name": "cityFromLocation"
      }
    },
    "table": {
      "id": "location-to-timezone-table-id"
    },
    "default": "UTC"
  }
}
```

---

### Transform 10: lifecycleState (Most Complex)

#### Helper Attributes for Dates

##### hireDateISO

###### UI Configuration
```
Attribute: hireDateISO

Purpose: Convert hire_date to ISO8601 format

Transform: Date Format
  Input: hire_date
  Input Format: MM/dd/yyyy
  Output Format: ISO8601

Expected Output:
  "01/15/2024" → "2024-01-15T00:00:00Z"
```

---

###### JSON Structure
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

---

##### termDateISO

###### UI Configuration
```
Attribute: termDateISO

Purpose: Convert term_date to ISO8601 format

Transform: Date Format
  Input: term_date
  Input Format: MM/dd/yyyy
  Output Format: ISO8601

Expected Output:
  "12/31/2024" → "2024-12-31T00:00:00Z"
  null → null
```

---

###### JSON Structure
```json
{
  "name": "termDateISO",
  "type": "dateFormat",
  "attributes": {
    "input": {
      "type": "identityAttribute",
      "attributes": {
        "name": "term_date"
      }
    },
    "inputFormat": "MM/dd/yyyy",
    "outputFormat": "ISO8601"
  }
}
```

---

#### Helper Attributes for Date Comparisons

##### isPreHire

###### UI Configuration
```
Attribute: isPreHire

Purpose: Check if hire_date is in future

Transform: Date Compare
  First Date: hireDateISO
  Comparison: Greater Than
  Second Date: [Today]
  Output: true or false
```

---

###### JSON Structure
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

##### isTerminated

###### UI Configuration
```
Attribute: isTerminated

Purpose: Check if terminated (term_date <= today)

Transform Chain:
  Step 1: Conditional (check term_date exists)
    Condition: termDateISO is not null
    If True: Continue to date compare
    If False: Return false

  Step 2: Date Compare
    First Date: termDateISO
    Comparison: Less Than or Equal
    Second Date: [Today]
    Output: true or false
```

---

###### JSON Structure
```json
{
  "name": "isTerminated",
  "type": "conditional",
  "attributes": {
    "expression": "termDateISO != null",
    "positiveCondition": {
      "type": "dateCompare",
      "attributes": {
        "firstDate": {
          "type": "identityAttribute",
          "attributes": {
            "name": "termDateISO"
          }
        },
        "operator": "lte",
        "secondDate": {
          "type": "now"
        }
      }
    },
    "negativeCondition": "false"
  }
}
```

---

##### isArchived

###### UI Configuration
```
Attribute: isArchived

Purpose: Check if terminated more than 90 days ago

Transform Chain:
  Step 1: Create "ninetyDaysAgo" helper
    Date Math: Today - 90 days

  Step 2: Conditional (check term_date exists)
    Condition: termDateISO is not null
    If True: Continue
    If False: Return false

  Step 3: Date Compare
    First Date: termDateISO
    Comparison: Less Than or Equal
    Second Date: ninetyDaysAgo
    Output: true or false
```

---

###### JSON Structure
```json
{
  "name": "ninetyDaysAgo",
  "type": "dateMath",
  "attributes": {
    "input": {
      "type": "now"
    },
    "operation": "subtract",
    "value": 90,
    "unit": "days"
  }
}
```
```json
{
  "name": "isArchived",
  "type": "conditional",
  "attributes": {
    "expression": "termDateISO != null",
    "positiveCondition": {
      "type": "dateCompare",
      "attributes": {
        "firstDate": {
          "type": "identityAttribute",
          "attributes": {
            "name": "termDateISO"
          }
        },
        "operator": "lte",
        "secondDate": {
          "type": "identityAttribute",
          "attributes": {
            "name": "ninetyDaysAgo"
          }
        }
      }
    },
    "negativeCondition": "false"
  }
}
```

---

#### Final Attribute: lifecycleState

##### UI Configuration
```
Attribute: lifecycleState

Purpose: Calculate lifecycle state based on dates and status

Transform: Conditional Chain

  Level 1: Check Leave of Absence
    Condition: status equals "On Leave"
    If True: "Leave of Absence"
    If False: Continue

  Level 2: Check Pre-hire
    Condition: isPreHire equals true
    If True: "Pre-hire"
    If False: Continue

  Level 3: Check Archived
    Condition: isArchived equals true
    If True: "Archived"
    If False: Continue

  Level 4: Check Terminated
    Condition: isTerminated equals true
    If True: "Terminated"
    If False: "Active"

Expected Output Examples:
  status="On Leave" → "Leave of Absence"
  hire_date in future → "Pre-hire"
  term_date 120 days ago → "Archived"
  term_date 30 days ago → "Terminated"
  normal employee → "Active"
```

---

##### JSON Structure
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
        "negativeCondition": {
          "type": "conditional",
          "attributes": {
            "expression": "isArchived == 'true'",
            "positiveCondition": "Archived",
            "negativeCondition": {
              "type": "conditional",
              "attributes": {
                "expression": "isTerminated == 'true'",
                "positiveCondition": "Terminated",
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

### Transform 11: managerEmail (Complex with Manager Resolution)

#### Helper Attributes

##### managerEmailFromAD (if available in your environment)

###### UI Configuration
```
Attribute: managerEmailFromAD

Purpose: Get manager's email from their AD account

Transform: Account Attribute
  Source: Active Directory
  Identity: manager
  Attribute: mail

Note: This requires manager correlation to be configured
```

---

###### JSON Structure
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

##### managerEmailGenerated

###### UI Configuration
```
Attribute: managerEmailGenerated

Purpose: Generate manager email from their name

Transform Chain:
  Step 1: Conditional (check manager exists)
    Condition: manager.firstname is not null AND manager.lastname is not null
    If True: Continue
    If False: Return null

  Step 2: Concatenation
    Values:
      - Lower(manager.firstname)
      - "."
      - Lower(manager.lastname)
      - "@company.com"
    Output: "jane.doe@company.com"
```

---

###### JSON Structure
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

##### deptHeadEmail

###### UI Configuration
```
Attribute: deptHeadEmail

Purpose: Get department head email as fallback

Transform: Lookup
  Input: department
  Table: DepartmentHeadEmails
  Default: null
```

---

###### JSON Structure
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
    },
    "default": null
  }
}
```

---

#### Final Attribute: managerEmail

##### UI Configuration
```
Attribute: managerEmail

Purpose: Get manager email with 5-level fallback

Transform: FirstValid
  Priority 1: managerEmailFromAD
  Priority 2: managerEmailGenerated
  Priority 3: manager.email (from HR)
  Priority 4: deptHeadEmail
  Priority 5: Static "no-manager@company.com"
```

---

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
          "name": "manager.email"
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

## Phase 4: Testing & Validation (90 minutes)

### Test Scenario 1: Perfect Data

#### Test Data
```
employee_id: "E12345"
firstname: "John"
lastname: "Smith"
personal_email: "john.personal@gmail.com"
work_phone: "(214) 555-1234"
hire_date: "01/15/2024"
term_date: null
employee_type: "FTE"
status: "Active"
dept_code: "ENG"
job_title: "Senior Software Engineer"
office_location: "Austin"
```

---

#### Expected Results
```
displayName: "Smith, John"
email: "john.personal@gmail.com" (Priority 1 - personal_email)
username: "john.smith"
department: "Engineering"
title: "Senior Engineer"
phone: "+12145551234"
timezone: "America/Chicago"
lifecycleState: "Active"
managerEmail: [depends on manager setup]
```

---

#### Testing Steps
```
1. Create test identity with data above
2. Apply transforms
3. Use Identity Preview to check each attribute
4. Document actual results
5. Compare to expected
6. Mark PASS or FAIL
```

---

### Test Scenario 2: Missing Personal Email

#### Test Data
```
Same as Scenario 1, but:
personal_email: null
```

---

#### Expected Results
```
email: "john.smith@company.com" (Priority 2 - emailFromName)
All other attributes same as Scenario 1
```

---

### Test Scenario 3: Very Long Names

#### Test Data
```
firstname: "Christopher"
lastname: "Vanderbilt-Johnson"
All other fields same as Scenario 1
```

---

#### Expected Results
```
displayName: "Vanderbilt-Johnson, Christopher"
email: "christopher.vanderbiltjohnson@company.com" (truncated to 50)
username: "christopher.vanderbi" (truncated to 20)
cleanFirstname: "christopher"
cleanLastname: "vanderbiltjohnson" (hyphen removed)
```

---

### Test Scenario 4: Special Characters

#### Test Data
```
firstname: "Mary-Ann"
lastname: "O'Brien"
All other fields same as Scenario 1
```

---

#### Expected Results
```
displayName: "O'Brien, Mary-Ann" (keeps original for display)
email: "maryann.obrien@company.com" (special chars removed)
username: "maryann.obrien"
cleanFirstname: "maryann"
cleanLastname: "obrien"
```

---

### Test Scenario 5: Missing Lastname

#### Test Data
```
firstname: "Madonna"
lastname: null
All other fields same as Scenario 1
```

---

#### Expected Results
```
displayName: "Madonna"
email: null (can't generate from name)
username: "madonna"
cleanFirstname: "madonna"
cleanLastname: ""
```

---

### Test Scenario 6: Pre-hire Employee

#### Test Data
```
Same as Scenario 1, but:
hire_date: "06/01/2024" (future date)
```

---

#### Expected Results
```
lifecycleState: "Pre-hire"
All other attributes same
```

---

### Test Scenario 7: Recently Terminated

#### Test Data
```
Same as Scenario 1, but:
term_date: "01/07/2024" (30 days ago from test date)
```

---

#### Expected Results
```
lifecycleState: "Terminated"
All other attributes same
```

---

### Test Scenario 8: Archived (Old Termination)

#### Test Data
```
Same as Scenario 1, but:
term_date: "10/01/2023" (120 days ago from test date)
```

---

#### Expected Results
```
lifecycleState: "Archived"
All other attributes same
```

---

### Test Scenario 9: Employee on Leave

#### Test Data
```
Same as Scenario 1, but:
status: "On Leave"
```

---

#### Expected Results
```
lifecycleState: "Leave of Absence"
All other attributes same
```

---

### Test Scenario 10: Missing Manager

#### Test Data
```
Same as Scenario 1, but:
manager_id: null
```

---

#### Expected Results
```
managerEmail: "eng-director@company.com" (dept head fallback)
All other attributes same
```

---

### Test Results Documentation Template
```markdown
# Test Results - Employee Onboarding Transform Suite

## Test Execution Summary
- Date Tested: _____________
- Tested By: _____________
- Environment: Sandbox / Dev / Production
- Total Scenarios: 10
- Passed: ___ / 10
- Failed: ___ / 10

## Scenario Results

### Scenario 1: Perfect Data
- Status: PASS / FAIL
- Issues Found: [List any issues]
- Notes: [Any observations]

### Scenario 2: Missing Personal Email
- Status: PASS / FAIL
- Issues Found: 
- Notes: 

### Scenario 3: Very Long Names
- Status: PASS / FAIL
- Issues Found: 
- Notes: 

[Continue for all 10 scenarios]

## Issues Found

### Issue 1
- Scenario: 
- Attribute: 
- Expected: 
- Actual: 
- Root Cause: 
- Fix Applied: 
- Re-test Result: 

## Performance Metrics
- Small volume (100 identities): ___ minutes
- Medium volume (1,000 identities): ___ minutes
- Performance Rating: Good / Acceptable / Needs Optimization

## Sign-Off
- All critical scenarios pass: YES / NO
- Ready for production: YES / NO
- Tester Signature: _____________
- Date: _____________
```

---

## Phase 5: Documentation (60 minutes)

### Transform Documentation Template
```markdown
# Employee Onboarding Transform Suite - Technical Documentation

## Overview
This transform suite converts raw Workday employee data into clean, standardized identity attributes for use in SailPoint ISC.

## Architecture

### Helper Attributes (Built First)
1. **cleanFirstname** - Normalized firstname
2. **cleanLastname** - Normalized lastname
3. **hireDateISO** - Hire date in ISO8601 format
4. **termDateISO** - Term date in ISO8601 format
5. **deptCodeUpper** - Uppercase department code
6. **cityFromLocation** - Extracted city from location
7. **phoneDigitsOnly** - Digits-only phone number
8. **emailFromName** - Generated email from name
9. **emailFromID** - Generated email from ID
10. **isPreHire** - Boolean: hire date in future
11. **isTerminated** - Boolean: terminated
12. **isArchived** - Boolean: terminated 90+ days ago
13. **ninetyDaysAgo** - Date 90 days ago
14. **managerEmailFromAD** - Manager email from AD
15. **managerEmailGenerated** - Generated manager email
16. **deptHeadEmail** - Department head email

### Core Attributes (Final Outputs)
1. **displayName** - "Lastname, Firstname" format
2. **email** - Primary email with 5-level fallback
3. **username** - firstname.lastname (max 20 chars)
4. **department** - Full department name
5. **title** - Standardized job title
6. **phone** - E.164 international format
7. **timezone** - Timezone from location
8. **lifecycleState** - Calculated lifecycle state
9. **managerEmail** - Manager email with fallbacks

## Lookup Tables Required

### DepartmentCodeMapping
Maps 3-letter codes to full names (10 entries minimum)

### TitleStandardization
Maps job title variations to standard titles (20+ entries)

### LocationToTimezone
Maps cities to timezones (10+ entries)

### DepartmentHeadEmails
Maps departments to head emails (10 entries)

## Transform Details

### displayName
- **Purpose:** Create formatted name for display
- **Format:** "Lastname, Firstname"
- **Dependencies:** firstname, lastname
- **Fallback:** Single name if one missing, "Unknown" if both missing
- **Edge Cases:** Handles null names gracefully

### email
- **Purpose:** Determine primary email address
- **Priority:**
  1. personal_email (source)
  2. emailFromName (generated)
  3. emailFromID (generated)
  4. username@company.com
  5. noemail@company.com (default)
- **Dependencies:** cleanFirstname, cleanLastname, employee_id, username
- **Format:** lowercase, max 50 characters
- **Edge Cases:** Handles missing name parts

### username
- **Purpose:** Generate system username
- **Format:** firstname.lastname (lowercase)
- **Max Length:** 20 characters
- **Dependencies:** cleanFirstname, cleanLastname
- **Fallback:** firstname only, lastname only, userEMPLOYEEID
- **Edge Cases:** Truncates long names, removes special characters

### department
- **Purpose:** Standardize department name
- **Method:** Lookup table
- **Dependencies:** deptCodeUpper
- **Default:** "Other" if code not found
- **Edge Cases:** Normalizes case before lookup

### title
- **Purpose:** Standardize job title
- **Method:** Lookup table
- **Dependencies:** job_title
- **Default:** Original title if not in table
- **Edge Cases:** Keeps original if no standard found

### phone
- **Purpose:** Format phone in E.164 international format
- **Format:** +[country code][number]
- **Dependencies:** work_phone or mobile_phone
- **Logic:** Adds +1 for 10-digit US numbers
- **Edge Cases:** Validates length, returns null if invalid

### timezone
- **Purpose:** Determine user's timezone
- **Method:** Extract city, lookup timezone
- **Dependencies:** office_location
- **Default:** UTC if location unknown
- **Edge Cases:** Handles complex location strings

### lifecycleState
- **Purpose:** Calculate current lifecycle state
- **States:** Pre-hire, Active, Leave of Absence, Terminated, Archived
- **Dependencies:** hireDateISO, termDateISO, status, isPreHire, isTerminated, isArchived
- **Logic:**
  1. On Leave status → "Leave of Absence"
  2. Hire date > today → "Pre-hire"
  3. Term date <= today - 90 days → "Archived"
  4. Term date <= today → "Terminated"
  5. Default → "Active"
- **Edge Cases:** Handles missing dates, validates format

### managerEmail
- **Purpose:** Resolve manager's email
- **Priority:**
  1. managerEmailFromAD (AD account)
  2. managerEmailGenerated (from name)
  3. manager.email (HR system)
  4. deptHeadEmail (department head)
  5. no-manager@company.com (default)
- **Dependencies:** manager correlation, department
- **Edge Cases:** Handles missing manager gracefully

## Error Handling

### Common Issues

**Issue:** Email returns null
- **Cause:** All name fields are null
- **Resolution:** Falls back to employee_id email

**Issue:** lifecycleState stuck on "Active"
- **Cause:** term_date not in ISO8601 format
- **Resolution:** Check hireDateISO and termDateISO helpers

**Issue:** Lookup returns default
- **Cause:** Case mismatch or value not in table
- **Resolution:** Check case normalization, verify table entries

## Performance

**Benchmarks:**
- 100 identities: ~2 minutes
- 1,000 identities: ~20 minutes
- 10,000 identities: ~120 minutes

**Optimization:**
- Helper attributes reused across multiple final attributes
- Lookup tables preferred over nested conditionals
- Efficient transform chaining

## Maintenance

**Monthly Tasks:**
- Review lookup tables for missing entries
- Add new department codes as needed
- Add new job title variations
- Verify timezone mappings

**Quarterly Tasks:**
- Performance benchmarking
- Edge case review
- Documentation updates

## Support

**Troubleshooting:**
See Day 6 Troubleshooting Guide for detailed debugging steps

**Contact:**
- Transform Owner: [Name]
- SailPoint Admin: [Name]
- Support Slack: #sailpoint-support
```

---

### Deployment Plan Document
```markdown
# Deployment Plan - Employee Onboarding Transform Suite

## Pre-Deployment Checklist

### Technical Validation
- [ ] All transforms tested in sandbox
- [ ] 10 edge case scenarios passed
- [ ] Performance benchmarked (acceptable)
- [ ] Identity Preview validated
- [ ] No errors in test logs
- [ ] Lookup tables created and populated
- [ ] Helper attributes built and tested
- [ ] JSON configuration backed up

### Business Validation
- [ ] Business requirements met
- [ ] Approved by HR stakeholder
- [ ] Approved by IT stakeholder
- [ ] Communication plan in place
- [ ] Training materials prepared (if needed)
- [ ] Support team notified

### Risk Assessment
- [ ] Rollback plan documented
- [ ] Impact analysis complete
- [ ] Maintenance window scheduled
- [ ] Backup of current configuration
- [ ] Test identities identified

## Deployment Steps

### Step 1: Backup Current State
**Estimated Time:** 15 minutes

1. Export current identity profile configuration
2. Document current attribute values for 20 sample identities
3. Save screenshots of current mappings
4. Create restore point

### Step 2: Create Lookup Tables
**Estimated Time:** 30 minutes

1. Navigate to Admin > Lookup Tables
2. Create DepartmentCodeMapping table
3. Create TitleStandardization table
4. Create LocationToTimezone table
5. Create DepartmentHeadEmails table
6. Verify all entries
7. Test each lookup with sample data

### Step 3: Deploy Helper Attributes
**Estimated Time:** 45 minutes

1. Deploy in this order:
   - cleanFirstname
   - cleanLastname
   - hireDateISO
   - termDateISO
   - deptCodeUpper
   - Other helpers...

2. After each helper:
   - Test with Identity Preview
   - Verify 5 sample identities
   - Check for errors

3. Do NOT click "Apply Changes" yet

### Step 4: Deploy Core Attributes
**Estimated Time:** 60 minutes

1. Deploy in this order:
   - displayName
   - username
   - department
   - title
   - phone
   - timezone
   - email (complex, test thoroughly)

2. After each attribute:
   - Test with Identity Preview
   - Verify with all 10 test scenarios
   - Check dependencies work

3. Still do NOT click "Apply Changes"

### Step 5: Deploy Advanced Attributes
**Estimated Time:** 45 minutes

1. Deploy:
   - lifecycleState
   - managerEmail

2. Test extensively:
   - All lifecycle state scenarios
   - All manager email fallback levels
   - Edge cases

### Step 6: Final Validation Before Apply
**Estimated Time:** 30 minutes

1. Use Identity Preview with 20 diverse identities
2. Verify every attribute looks correct
3. Check for any unexpected nulls
4. Review configuration one more time
5. Get final approval from stakeholder

### Step 7: Apply Changes
**Estimated Time:** Variable (depends on identity count)

1. Click "Apply Changes" button
2. Identity processing begins
3. Monitor progress in Admin > System > Task Results
4. Wait for completion

### Step 8: Immediate Post-Deployment Validation
**Estimated Time:** 60 minutes

1. Sample 20 random identities immediately
2. Check all 9 attributes for each
3. Verify against expected values
4. Check error logs
5. Look for anomalies

### Step 9: Extended Monitoring
**Day 1:** Sample 50 identities, review hourly
**Day 3:** Sample 100 identities, review twice daily
**Week 1:** Sample 200 identities, review daily
**Month 1:** Full audit, optimize as needed

## Rollback Procedures

### When to Rollback

Rollback immediately if:
- More than 5% error rate
- Critical business process broken
- Data corruption detected
- Unexpected widespread nulls
- Security issue identified

### How to Rollback

**Quick Rollback (Emergency):**
1. Go to each attribute mapping
2. Change Source to "Direct" mapping
3. Select source attribute (temporary)
4. Click "Apply Changes"
5. Monitor processing
6. Verify rollback successful

**Full Rollback:**
1. Restore from backup configuration
2. Delete new helper attributes
3. Delete new lookup tables
4. Click "Apply Changes"
5. Verify identities return to previous state

### Post-Rollback
1. Document what went wrong
2. Root cause analysis
3. Fix in sandbox
4. Re-test thoroughly
5. Schedule new deployment

## Communication Plan

### Pre-Deployment
**To:** All stakeholders
**When:** 1 week before
**Message:** "We're deploying new identity attribute transforms on [DATE]. You may notice changes to email, username, and display name formats. Contact [SUPPORT] with questions."

### During Deployment
**To:** Support team
**When:** Start of deployment
**Message:** "Identity transform deployment in progress. Monitor #sailpoint-alerts channel. Escalate issues immediately."

### Post-Deployment
**To:** All stakeholders
**When:** After successful validation
**Message:** "Identity transform deployment complete. All validations passed. Contact [SUPPORT] if you notice any issues."

## Success Criteria

Deployment is successful when:
- ✓ All 10 test scenarios pass
- ✓ Error rate < 1%
- ✓ No critical incidents
- ✓ Sample validation 95%+ accurate
- ✓ Performance within benchmarks
- ✓ No rollback needed
- ✓ Positive feedback from users

## Sign-Off

- Technical Lead: _____________ Date: _______
- Business Owner: _____________ Date: _______
- SailPoint Admin: _____________ Date: _______

**APPROVED FOR DEPLOYMENT:** YES / NO
```

---

## Phase 6: Final Review (30 minutes)

### Self-Assessment Checklist
```
Technical Completion:
□ All 9 core attributes built
□ All 16 helper attributes built
□ All 4 lookup tables created
□ All transforms tested individually
□ All 10 edge cases tested
□ Performance benchmarked
□ No errors in logs

Quality Assurance:
□ All attributes return expected values
□ Null handling works correctly
□ Special characters handled
□ Long names truncated properly
□ Date comparisons work
□ Fallback logic functions
□ Lookup tables accurate

Documentation:
□ Technical documentation complete
□ Deployment plan written
□ Test results documented
□ Rollback procedures defined
□ Troubleshooting guide created
□ Communication plan ready

Production Readiness:
□ Stakeholder approval obtained
□ Backup configuration saved
□ Maintenance window scheduled
□ Support team notified
□ Monitoring plan in place
```

---

### Project Completion Review
```markdown
# Employee Onboarding Transform Suite - Project Review

## What You Built

### Transforms Created: 25 Total
- 9 Core Attributes (final outputs)
- 16 Helper Attributes (intermediate processing)
- 4 Lookup Tables (reference data)

### Lines of Configuration: ~500+
- JSON configuration
- Conditional logic
- Transform chains
- Lookup mappings

### Edge Cases Handled: 10+
- Perfect data
- Missing fields
- Long names
- Special characters
- Date scenarios
- Manager fallbacks

## Skills Demonstrated

### Day 1-2 Skills
✓ String transforms (Concat, Trim, Replace, Upper, Lower, Substring)
✓ Conditional logic (if-then-else, nested conditions)
✓ FirstValid fallback chains

### Day 3 Skills
✓ Lookup tables
✓ Account Attribute
✓ Identity Attribute
✓ Cross-reference data

### Day 4 Skills
✓ Date Format conversion
✓ Date Math calculations
✓ Date Compare logic
✓ Split, Index, Join operations

### Day 5 Skills
✓ Helper attribute pattern
✓ Complex chaining
✓ Performance optimization
✓ Production-ready patterns

### Day 6 Skills
✓ Testing strategies
✓ Debugging techniques
✓ Identity Preview usage
✓ Error handling

## Complexity Metrics

**Most Complex Transform:** lifecycleState
- Dependencies: 5 helper attributes
- Conditional levels: 4 nested
- Date operations: 3
- Boolean logic: 3 conditions

**Longest Chain:** email
- Steps: 6 (including FirstValid with 5 priorities)
- Dependencies: 2 helper attributes
- Fallback levels: 5

**Most Helpers:** cleanFirstname/cleanLastname
- Used by: 3 core attributes (displayName, email, username)
- Transform steps: 6 each
- Critical for data quality

## Production Readiness Score

Rate yourself on each:

Technical Implementation: ___ / 10
- All transforms working correctly
- No errors in testing
- Performance acceptable

Edge Case Handling: ___ / 10
- All 10 scenarios pass
- Null handling complete
- Fallbacks functional

Documentation Quality: ___ / 10
- Technical docs complete
- Deployment plan thorough
- Test results documented

Overall Readiness: ___ / 30

**25-30:** Production ready, deploy with confidence
**20-24:** Minor improvements needed
**15-19:** Significant work required
**<15:** Not ready for production

## What Went Well

1. 
2. 
3. 

## Challenges Faced

1. 
2. 
3. 

## What You Learned

1. 
2. 
3. 

## Next Steps

Immediate:
1. Final review with stakeholder
2. Schedule deployment
3. Prepare monitoring

Future:
1. Monitor post-deployment
2. Gather feedback
3. Optimize based on real data
4. Document lessons learned
```

---

## Congratulations!

### You've Completed the 7-Day SailPoint ISC Transforms Mastery Program!

---

### What You've Accomplished

**Days 1-2: Foundation**
- ✅ Mastered 7 core string transforms
- ✅ Built conditional logic
- ✅ Created fallback chains
- ✅ Learned transform chaining

**Days 3-4: Advanced Techniques**
- ✅ Implemented lookup tables
- ✅ Used Account and Identity Attributes
- ✅ Mastered date operations
- ✅ Handled list/array operations

**Day 5: Production Patterns**
- ✅ Built 5 enterprise-grade patterns
- ✅ Optimized for performance
- ✅ Learned transforms vs rules
- ✅ Understood UI and JSON

**Day 6: Professional Skills**
- ✅ Mastered debugging techniques
- ✅ Learned testing strategies
- ✅ Created validation procedures
- ✅ Built troubleshooting skills

**Day 7: Capstone Achievement**
- ✅ Built complete onboarding suite
- ✅ Created production documentation
- ✅ Developed deployment plan
- ✅ Demonstrated mastery

---

### Your Transform Skillset

You can now:

**Technical Skills:**
- Build transforms from simple to complex
- Chain 10+ steps efficiently
- Handle all edge cases gracefully
- Optimize for performance
- Debug issues systematically
- Test comprehensively
- Deploy to production safely

**Business Skills:**
- Understand requirements
- Design solutions
- Document thoroughly
- Communicate with stakeholders
- Plan deployments
- Support production systems

---

### Certification of Completion
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
    SAILPOINT ISC TRANSFORMS MASTERY
           7-DAY INTENSIVE PROGRAM
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

This certifies that

    [YOUR NAME]

has successfully completed the comprehensive
SailPoint IdentityNow (ISC) Transforms Mastery
training program and has demonstrated expertise in:

✓ Transform Configuration (UI & JSON)
✓ Complex Logic Implementation
✓ Production Deployment
✓ Testing & Validation
✓ Troubleshooting & Debugging
✓ Performance Optimization
✓ Professional Documentation

Completed: [DATE]
Project: Employee Onboarding Transform Suite
Complexity: Enterprise-Grade
Readiness: Production-Ready

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

### Where to Go From Here

**Immediate Next Steps:**
1. Deploy your capstone project to production
2. Monitor and optimize based on real data
3. Share your knowledge with your team
4. Document lessons learned

**Continued Learning:**
- Explore custom transforms (advanced)
- Learn SailPoint Rules development
- Study identity lifecycle management
- Master access request workflows
- Investigate MCP server integration

**Career Growth:**
- You're now qualified for SailPoint ISC roles
- Consider SailPoint certification exams
- Contribute to SailPoint community
- Mentor others learning transforms

---

### Final Words

You started this journey 7 days ago as a transform beginner. Today, you've built a production-ready employee onboarding suite with:

- **25 transforms** working together
- **10+ edge cases** handled gracefully
- **5 fallback levels** ensuring reliability
- **500+ lines** of configuration
- **Complete documentation** for support
- **Deployment plan** for production

This is not just learning - this is **real-world expertise**.

You're ready to build, deploy, and support SailPoint ISC transforms in production environments.

**Well done!**

---

*End of SailPoint ISC Transforms Mastery 7-Day Study Plan*
