# STEP 02: ISC Navigation & Execution Context
## Where Transforms Live and When They Execute

**Version: CORRECTED** (Based on Official SailPoint Documentation)  
**Last Updated: February 2026**

---

## What You'll Master
- ✅ Where to find and configure aggregation transforms in ISC (UI-based)
- ✅ Where to configure provisioning transforms in ISC (API-only, no UI)
- ✅ Aggregation context (source → ISC) vs Provisioning context (ISC → target)
- ✅ What data is available in each context
- ✅ How to navigate ISC admin console for aggregation setup
- ✅ How to deploy provisioning policies via REST API

---

## CRITICAL DISTINCTION: UI vs API

### The Most Important Thing to Understand

In ISC, transforms work in TWO different execution contexts with DIFFERENT deployment methods:

| Context | Location | Method | Data Available |
|---------|----------|--------|-----------------|
| **Aggregation** | Identity Profile Mapping | ISC Admin UI | accountAttribute |
| **Provisioning** | Provisioning Policy | REST API Only | identityAttribute |

**This is NOT like IIQ.** ISC does NOT have a web UI for provisioning policy configuration. All provisioning policies must be deployed via REST API.

---

## PART 1: AGGREGATION TRANSFORMS (UI-Based)

### Where Aggregation Happens

Aggregation is configured in the ISC Admin console by navigating to Connections → Sources, selecting the source, and configuring the mapping in the Identity Profile section.

### Step-by-Step: Finding Aggregation Transforms in UI

1. **Login to ISC Admin Console**
   - URL: `https://<your-tenant>.identitynow.com/admin`

2. **Navigate to Identity Profiles**
   - Click **Admin** (top navigation bar)
   - Click **Identities** (left sidebar)
   - Click **Identity Profiles**

3. **Select Your Source's Identity Profile**
   - Choose the Identity Profile tied to your source (e.g., "Workday Employees")
   - Click the profile to open it

4. **Find the Mapping Section**
   - Look for **Attribute Mappings** or **Mapping** tab
   - You'll see a table with three columns:
     - **Source Attribute** (left) - Data coming FROM the source system
     - **Transform** (middle) - Where you can add transformations
     - **Identity Attribute** (right) - Where data goes INTO ISC

5. **Add/Edit a Transform**
   - Click on an attribute row
   - Click the **Transform** button or field
   - You can create an inline transform or reference an existing transform

### What Data is Available During Aggregation

During aggregation, the following data sources are available:

✅ **accountAttribute** - Data from the source system account
```json
{
  "type": "accountAttribute",
  "attributes": {
    "sourceName": "Workday",
    "attributeName": "firstName"
  }
}
```

✅ **static** - Fixed, unchanging values
```json
{
  "type": "static",
  "attributes": {
    "value": "Employee"
  }
}
```

✅ **firstValid** - Fallback logic (try multiple sources, use first with data)
```json
{
  "type": "firstValid",
  "attributes": {
    "values": [
      { "type": "accountAttribute", "attributes": { "sourceName": "Workday", "attributeName": "firstName" } },
      { "type": "accountAttribute", "attributes": { "sourceName": "SuccessFactors", "attributeName": "firstName" } }
    ]
  }
}
```

❌ **identityAttribute** - NOT available during aggregation
- The identity hasn't been created yet, so there's no identity to reference

### Example Aggregation Transform (UI)

**Scenario:** Workday sends firstName in all caps ("JOHN"), but you want it as proper case ("John")

1. Go to Identity Profile Mapping
2. Find the **firstName** row
3. Click **Add Transform**
4. Select transform type: **Lower** or **Case** (depending on your ISC version)
5. Configure to apply to the firstName attribute
6. Save

**Result:** During aggregation, "JOHN" becomes "John" before storing in ISC identity profile.

### Testing Aggregation Transforms in UI

1. Go to **Admin → Identities → Identity Profiles**
2. Select your Identity Profile
3. Click **Preview** button
4. Select a test identity/account from the source
5. Click **Run Preview**
6. You'll see what the transforms OUTPUT for that user

---

## PART 2: PROVISIONING TRANSFORMS (API-Only)

### Critical Fact: NO UI in ISC

Provisioning policies in ISC are managed entirely through REST APIs. You must call the Get Provisioning Policy API for the source you want to add your transform to, then upload your complete CREATE provisioning policy using the Create Provisioning Policy API, or use the Update Provisioning Policy API to update an existing provisioning policy.

**There is no web interface to configure provisioning transforms.** All configuration is JSON-based, deployed via API.

### Where Provisioning Policies Live

Provisioning policies are stored per source and per operation type:
- Stored in SailPoint's backend database
- Accessed ONLY via REST API endpoints
- No UI "Create Profile" section exists in ISC

### The Provisioning Policy REST APIs

**Endpoint:** `PUT /sources/{sourceId}/provisioning-policy`

**Required Authority:** API, ORG_ADMIN, SOURCE_ADMIN, or SOURCE_SUBADMIN token

**Step 1: Get Existing Provisioning Policy**

```bash
curl -X GET 'https://<tenant>.identitynow.com/v3/sources/{sourceId}/provisioning-policy?usageType=CREATE' \
  -H 'Authorization: Bearer {token}' \
  -H 'Content-Type: application/json'
```

**Step 2: Modify the Policy JSON**

Add transforms to the `fields` array for any attribute that needs transformation.

**Step 3: Update/Create the Provisioning Policy**

```bash
curl -X PUT 'https://<tenant>.identitynow.com/v3/sources/{sourceId}/provisioning-policy' \
  -H 'Authorization: Bearer {token}' \
  -H 'Content-Type: application/json' \
  -d @provisioning-policy.json
```

### What Data is Available During Provisioning

During provisioning, different data is available than aggregation:

✅ **identityAttribute** - Data from the ISC identity profile (use ONLY this for provisioning)
```json
{
  "type": "identityAttribute",
  "attributes": {
    "name": "firstName"
  }
}
```

✅ **accountAttribute** - Data from another source the user already has an account on
```json
{
  "type": "accountAttribute",
  "attributes": {
    "sourceName": "Active Directory",
    "attributeName": "sAMAccountName"
  }
}
```

✅ **getReferenceIdentityAttribute** - Data from a related identity (like a manager)
```json
{
  "type": "getReferenceIdentityAttribute",
  "attributes": {
    "referenceName": "manager",
    "attributeName": "email"
  }
}
```

✅ **static** - Fixed values
```json
{
  "type": "static",
  "attributes": {
    "value": "contractor"
  }
}
```

✅ **reference** - Reference other transforms within the same policy
```json
{
  "type": "reference",
  "attributes": {
    "id": "userName"
  }
}
```

❌ **accountAttribute from authoritative source** - NOT available
- Aggregation is completed; you only work with ISC identity data

### Example Provisioning Policy with Transforms

**Scenario:** You're provisioning to Active Directory. ISC has email "john.smith@company.com" but AD expects a 12-character sAMAccountName.

**Complete Provisioning Policy JSON:**

```json
{
  "name": "Active Directory Provisioning Policy",
  "description": "Policy for creating and updating AD accounts",
  "usageType": "CREATE",
  "fields": [
    {
      "name": "sAMAccountName",
      "transform": {
        "type": "lower",
        "attributes": {
          "input": {
            "type": "identityAttribute",
            "attributes": {
              "name": "email"
            }
          }
        }
      },
      "attributes": {
        "template": "${firstname}.${lastname}",
        "cloudMaxSize": "12",
        "cloudRequired": "true"
      },
      "isRequired": true,
      "type": "string",
      "isMultiValued": false
    },
    {
      "name": "givenName",
      "transform": null,
      "attributes": {},
      "isRequired": false,
      "type": "string",
      "isMultiValued": false
    },
    {
      "name": "mail",
      "transform": {
        "type": "identity",
        "attributes": {
          "input": {
            "type": "identityAttribute",
            "attributes": {
              "name": "email"
            }
          }
        }
      },
      "attributes": {},
      "isRequired": true,
      "type": "string",
      "isMultiValued": false
    }
  ]
}
```

### Critical Rule: Use identityAttribute in Provisioning

You must use the identityAttribute type when you're writing transforms in provisioning policies. The accountAttribute type won't work during provisioning.

❌ **WRONG:**
```json
{
  "type": "accountAttribute",
  "attributes": {
    "sourceName": "Workday",
    "attributeName": "firstName"
  }
}
```

✅ **CORRECT:**
```json
{
  "type": "identityAttribute",
  "attributes": {
    "name": "firstName"
  }
}
```

---

## PART 3: The Two Execution Contexts - Side by Side

### Context #1: AGGREGATION (Source → ISC)

**When it happens:**
- Scheduled aggregation job runs
- Connector pulls data from source system
- Data flows into ISC identity profile

**Data Flow:**
```
Workday System
  ↓ (firstName = "JOHN")
Connector reads account data
  ↓
Identity Profile Mapping applies Transform
  ↓ (Transform: lower case)
Transform outputs: "john"
  ↓
Stored in ISC Identity: firstName = "john"
```

**Where configured:** ISC Admin UI → Identity Profiles → Attribute Mappings

**Available data sources:**
- ✅ accountAttribute (from source)
- ✅ static (fixed values)
- ✅ firstValid (fallback)
- ❌ identityAttribute (not created yet)

**Example:**
```json
{
  "name": "firstName_LowerCase",
  "type": "lower",
  "attributes": {
    "input": {
      "type": "accountAttribute",
      "attributes": {
        "sourceName": "Workday",
        "attributeName": "firstName"
      }
    }
  }
}
```

---

### Context #2: PROVISIONING (ISC → Target)

**When it happens:**
- User submits access request
- ISC builds provisioning plan
- ISC sends data to target system via connector
- Transforms modify data before sending

**Data Flow:**
```
ISC Identity has: email = "john.smith@company.com"
  ↓
Provisioning Policy applies Transform
  ↓ (Transform: substring to 12 chars, lowercase)
Transform outputs: "john.smith"
  ↓
Sent to Active Directory: sAMAccountName = "john.smith"
```

**Where configured:** REST API only (no UI)

**Available data sources:**
- ✅ identityAttribute (from ISC)
- ✅ accountAttribute (from other sources)
- ✅ getReferenceIdentityAttribute (manager, etc.)
- ✅ static (fixed values)
- ✅ reference (other transforms)

**Example:**
```json
{
  "name": "AD_sAMAccountName",
  "usageType": "CREATE",
  "fields": [
    {
      "name": "sAMAccountName",
      "transform": {
        "type": "lower",
        "attributes": {
          "input": {
            "type": "identityAttribute",
            "attributes": {
              "name": "email"
            }
          }
        }
      },
      "isRequired": true,
      "type": "string"
    }
  ]
}
```

---

## Key Differences: Aggregation vs Provisioning

| Aspect | Aggregation | Provisioning |
|--------|-------------|--------------|
| **Direction** | Source → ISC | ISC → Target |
| **When runs** | Scheduled aggregation job | Access request/provisioning task |
| **Configured where** | ISC Admin UI ✅ | REST API ONLY ❌ |
| **Primary data source** | accountAttribute | identityAttribute |
| **Can use identityAttribute?** | NO (identity not created yet) | YES (use exclusively) |
| **Can use accountAttribute?** | YES | Only from other sources |
| **Stored where** | Identity Profile Mapping | Provisioning Policy (database) |

---

## CRITICAL: You CAN'T Mix Contexts

### ❌ DON'T DO THIS #1: identityAttribute in Aggregation

During aggregation, the identity hasn't been created yet. This will fail:

```json
{
  "type": "lower",
  "attributes": {
    "input": {
      "type": "identityAttribute",
      "attributes": {
        "name": "firstName"
      }
    }
  }
}
```

**Error:** Transform fails because identityAttribute is null.

---

### ❌ DON'T DO THIS #2: accountAttribute in Provisioning

During provisioning, source data isn't available. This will fail:

```json
{
  "type": "lower",
  "attributes": {
    "input": {
      "type": "accountAttribute",
      "attributes": {
        "sourceName": "Workday",
        "attributeName": "firstName"
      }
    }
  }
}
```

**Error:** Transform fails because accountAttribute isn't in the provisioning context.

---

## How to Navigate ISC for Aggregation Setup

### Navigate to Identity Profiles (Aggregation)

1. Click **Admin** (top navigation)
2. Navigate to **Identities** → **Identity Profile**
3. Click on the Identity Profile tied to your source
4. Look for the **Mapping** or **Attribute Mapping** section
5. You'll see a table with source attributes, transforms, and identity attributes

### Configure an Aggregation Transform

1. Find the attribute you want to transform
2. Click in the **Transform** column for that row
3. Choose to create a new transform or reference an existing one
4. Build your transform logic
5. Save
6. Run aggregation preview to test

### Find Import Data Settings

1. Go to **Admin** → **Connections** → **Sources**
2. Click on your source
3. Navigate **Import Data** tab
4. You'll find:
   - **Correlation** tab - Account correlation rules
   - **Password Settings** - Password policies
   - **Account Schema** - Additional attributes

---

## How to Deploy Provisioning Transforms via API

### Prerequisites

1. **Personal Access Token or Client Credentials**
   - OAuth 2.0 Client with API scope
   - Service account with ORG_ADMIN or SOURCE_ADMIN role

2. **Source ID**
   - Get from: Admin → Connections → Sources → Click source → Copy ID from URL

3. **REST API Client**
   - Postman, curl, Python requests, or your IDE

### Step-by-Step: Deploy a Provisioning Policy Transform

**Step 1: Retrieve Existing Provisioning Policy**

```bash
curl -X GET "https://tenant.identitynow.com/v3/sources/{sourceId}/provisioning-policy?usageType=CREATE" \
  -H "Authorization: Bearer {YOUR_TOKEN}" \
  -H "Content-Type: application/json"
```

**Step 2: Modify the JSON**

Add a transform to one of the fields. Example to add a transform to the `mail` attribute:

```json
{
  "name": "email_to_mail_transform",
  "description": "Map ISC email to AD mail",
  "usageType": "CREATE",
  "fields": [
    {
      "name": "mail",
      "transform": {
        "type": "identity",
        "attributes": {
          "input": {
            "type": "identityAttribute",
            "attributes": {
              "name": "email"
            }
          }
        }
      },
      "attributes": {},
      "isRequired": true,
      "type": "string",
      "isMultiValued": false
    },
    {
      "name": "sAMAccountName",
      "transform": {
        "type": "lower",
        "attributes": {
          "input": {
            "type": "identityAttribute",
            "attributes": {
              "name": "email"
            }
          }
        }
      },
      "attributes": {
        "cloudMaxSize": "20",
        "cloudRequired": "true"
      },
      "isRequired": true,
      "type": "string",
      "isMultiValued": false
    }
  ]
}
```

**Step 3: Update the Provisioning Policy**

```bash
curl -X PUT "https://tenant.identitynow.com/v3/sources/{sourceId}/provisioning-policy" \
  -H "Authorization: Bearer {YOUR_TOKEN}" \
  -H "Content-Type: application/json" \
  -d @provisioning-policy.json
```

**Response:** HTTP 200 OK with the updated policy

### Using Postman for Provisioning Policy API

1. Create new PUT request
2. URL: `https://tenant.identitynow.com/v3/sources/{sourceId}/provisioning-policy`
3. Headers tab:
   - Key: `Authorization` | Value: `Bearer {token}`
   - Key: `Content-Type` | Value: `application/json`
4. Body tab: Select **raw** → **JSON**
5. Paste your provisioning policy JSON
6. Click **Send**

---

## Testing & Validation

### Testing Aggregation Transforms

1. **In ISC UI:**
   - Go to Identity Profile
   - Click **Preview**
   - Select a test user
   - Click **Run Preview**
   - Verify transformed values are correct

2. **Run Manual Aggregation:**
   - Go to Admin → Connections → Sources
   - Select source
   - Go to **Import Data** tab
   - Click **Start** under Manual Aggregation
   - Wait for completion
   - Check identity profile for correct values

### Testing Provisioning Transforms

1. **Use REST API Preview (if available):**
   - Some SailPoint instances provide a transform preview endpoint
   - Contact SailPoint support for your tenant's preview endpoint

2. **Test in Development Environment:**
   - Deploy policy to dev tenant first
   - Create a test access request
   - Verify connector receives correct transformed values
   - Check target system for correct account attribute values

3. **Check Provisioning Logs:**
   - Go to Admin → System → Logs
   - Search for provisioning events
   - Look for transform execution messages
   - Verify no errors in transform application

---

## Common Errors & Solutions

### Aggregation Errors

**Error: "identityAttribute not found"**
- **Cause:** Tried to use identityAttribute in aggregation
- **Fix:** Use accountAttribute instead

**Error: "Transform returned null"**
- **Cause:** Source data was null/empty
- **Fix:** Add null-handling logic or use firstValid with fallback

**Error: "Account not correlated"**
- **Cause:** Correlation rule didn't match accounts to identities
- **Fix:** Review correlation configuration in Import Data → Correlation tab

---

### Provisioning Errors

**Error: "Invalid provisioning policy JSON"**
- **Cause:** Syntax error in JSON
- **Fix:** Validate JSON at jsonlint.com before uploading

**Error: "accountAttribute not available in provisioning context"**
- **Cause:** Tried to use accountAttribute from authoritative source
- **Fix:** Use identityAttribute instead

**Error: "Transform failed during provisioning"**
- **Cause:** Data type mismatch or invalid transform logic
- **Fix:** Check Logs for detailed error message, review transform type

**Error: "401 Unauthorized"**
- **Cause:** Invalid token or insufficient permissions
- **Fix:** Verify token is valid, user has ORG_ADMIN or SOURCE_ADMIN role

---

## Quick Reference: Where Things Are

### For Aggregation Configuration
```
Admin Menu
└── Identities
    └── Identity Profiles
        └── [Select your profile]
            └── Attribute Mappings (configure transforms here)
```

### For Source Configuration
```
Admin Menu
└── Connections
    └── Sources
        └── [Select your source]
            └── Import Data tab
                ├── Correlation tab
                ├── Password Settings
                └── Account Schema
```

### For Provisioning Configuration
```
REST API: /v3/sources/{sourceId}/provisioning-policy
GET    - Retrieve existing policy
PUT    - Update or create policy
```

---

## Key Takeaways

1. **Aggregation = UI + API**
   - Configure basic mappings in ISC UI
   - Advanced transforms via Transform REST API
   - Uses accountAttribute data source

2. **Provisioning = API ONLY**
   - No web UI for provisioning policy transforms
   - Configure entirely via REST API
   - Uses identityAttribute data source

3. **Different Data Contexts**
   - During aggregation: accountAttribute available, identityAttribute NOT
   - During provisioning: identityAttribute available, accountAttribute limited

4. **Test Everything**
   - Use Preview for aggregation
   - Check logs for provisioning
   - Validate data in target system

5. **Common Mistake**
   - Expecting a UI for provisioning policies like in IIQ
   - All provisioning configuration is API-driven in ISC

---

## What's Next

Now that you understand:
- ✅ Where aggregation transforms live (ISC UI)
- ✅ Where provisioning transforms live (REST API)
- ✅ What data is available in each context
- ✅ How to navigate ISC for configuration

Next, you'll learn:
- **STEP 03:** How to READ transform JSON syntax
- **STEP 04:** How to WRITE valid transforms with Velocity syntax

Ready to continue? Move to STEP 03.

