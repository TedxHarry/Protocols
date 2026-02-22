# STEP 02: ISC Navigation & Execution Context
## Where Transforms Live and When They Execute

---

## What You'll Master
- ✅ Where to find and create transforms in ISC
- ✅ Aggregation context (source → ISC) vs Provisioning context (ISC → target)
- ✅ What data is available in each context
- ✅ How to navigate ISC UI for transforms
- ✅ How to see transform results and errors

---

## WHERE TRANSFORMS LIVE IN ISC

### Two Main Places

**1. Identity Profile Mapping (Aggregation)**
- **Path:** Admin → Identity Management → Identity Profiles
- **What it does:** Transforms data FROM source systems INTO ISC
- **When it runs:** When data is aggregated from sources
- **Example:** Workday sends firstName, you transform it to proper case

**2. Create Profiles (Provisioning)**
- **Path:** Admin → Identity Management → Create Profiles
- **What it does:** Transforms data FROM ISC TO target systems
- **When it runs:** When access is provisioned to target systems
- **Example:** ISC has email, you transform it to match AD format

---

## THE TWO EXECUTION CONTEXTS

### Context #1: AGGREGATION (Source → ISC)

**When it happens:**
- Connector pulls data from source system
- Data flows into ISC
- Transforms modify the data BEFORE storing in identity profile

**Data Flow:**
```
Workday
  ↓ (firstName = "john")
Connector reads data
  ↓
Identity Profile Mapping applies Transform
  ↓ (Transform: titleCase)
Transform outputs: "John"
  ↓
Stored in Identity: firstName = "John"
```

**What data is available:**
- ✅ accountAttribute - Data from the source system
- ✅ static - Fixed values
- ✅ firstValid - Multiple sources with fallback
- ❌ identityAttribute - NOT available (identity not loaded yet)

**Example Transform (Aggregation):**
```json
{
  "name": "FirstName_TitleCase_FromWorkday",
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
- User gets access to a target system
- ISC sends data to target
- Transforms modify the data BEFORE sending to target

**Data Flow:**
```
ISC Identity has: email = "john.smith@company.com"
  ↓
Create Profile for Active Directory applies Transform
  ↓ (Transform: format for AD)
Transform outputs: "jsmith"
  ↓
Sent to Active Directory: sAMAccountName = "jsmith"
```

**What data is available:**
- ✅ identityAttribute - Data from ISC identity profile
- ✅ accountAttribute - Account on another source
- ✅ getReferenceIdentityAttribute - Another identity's data (like manager)
- ✅ static - Fixed values
- ✅ reference - Other transforms

**Example Transform (Provisioning):**
```json
{
  "name": "AD_Username_FromIdentity",
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

## KEY DIFFERENCE: Aggregation vs Provisioning

| Aspect | Aggregation | Provisioning |
|--------|-------------|--------------|
| **Direction** | Source → ISC | ISC → Target |
| **When** | Data import | Access request |
| **What's available** | accountAttribute | identityAttribute |
| **Where configured** | Identity Profile Mapping | Create Profile |
| **Output** | Stored in identity | Sent to target |

---

## CRITICAL: You CAN'T Mix Contexts

### ❌ WRONG: Using identityAttribute in Aggregation
```json
{
  "type": "lower",
  "attributes": {
    "input": {
      "type": "identityAttribute",  // ❌ WRONG - doesn't exist yet
      "attributes": { "name": "firstName" }
    }
  }
}
```
**Why it fails:** During aggregation, the identity isn't loaded yet, so identityAttribute doesn't have a value.

### ❌ WRONG: Using accountAttribute in Provisioning
```json
{
  "type": "lower",
  "attributes": {
    "input": {
      "type": "accountAttribute",  // ❌ WRONG - not available in provisioning
      "attributes": {
        "sourceName": "Workday",
        "attributeName": "firstName"
      }
    }
  }
}
```
**Why it fails:** During provisioning, source data isn't available. You only have ISC identity.

---

## HOW TO NAVIGATE ISC UI

### Finding Identity Profile Mapping (Aggregation)

1. Click **Admin** (top navigation)
2. Click **Identity Management** (left menu)
3. Click **Identity Profiles**
4. Select your source (e.g., "Workday")
5. Click **Mapping** tab
6. You see a table of attributes with Transform options

**What you see:**
```
Source Attribute   →   Transform   →   Identity Attribute
firstName          →   [optional]  →   firstName
lastName           →   [optional]  →   lastName
email              →   [optional]  →   email
```

**Click on Transform column** to add/edit transforms for that attribute.

---

### Finding Create Profiles (Provisioning)

1. Click **Admin** (top navigation)
2. Click **Identity Management** (left menu)
3. Click **Create Profiles**
4. Select your target system (e.g., "Active Directory")
5. You see a table of mappings

**What you see:**
```
Identity Attribute  →  Transform   →  Target Account Attribute
firstName           →  [optional]  →  givenName
lastName            →  [optional]  →  sn
email               →  [optional]  →  mail
```

**Click on Transform column** to add/edit transforms for that mapping.

---

## WHAT DATA IS AVAILABLE IN EACH PLACE

### In Identity Profile Mapping (Aggregation)

**Column 1 (Left) - Source Data:**
- Comes from accountAttribute
- Example: firstName from Workday

**Column 3 (Right) - Output:**
- Goes to identityAttribute
- Example: stored as firstName in ISC

**Available in Transform:**
- ✅ accountAttribute (from source)
- ✅ static (fixed values)
- ✅ firstValid (try multiple sources)
- ❌ identityAttribute (not loaded yet)

---

### In Create Profile (Provisioning)

**Column 1 (Left) - ISC Data:**
- Comes from identityAttribute
- Example: firstName from ISC identity

**Column 3 (Right) - Output:**
- Goes to target system
- Example: sent as givenName to AD

**Available in Transform:**
- ✅ identityAttribute (from ISC)
- ✅ accountAttribute (from other sources)
- ✅ getReferenceIdentityAttribute (manager, etc.)
- ✅ reference (other transforms)
- ✅ static (fixed values)

---

## SEEING TRANSFORM RESULTS

### After Running Aggregation

1. Go to **Admin** → **Identity Management** → **Identity Profiles**
2. Click **Preview**
3. Select a test user from the source
4. Click **Preview**
5. You see what transforms OUTPUT for that user

**What you see:**
```
Source Data          Transform Applied        Result
firstName: "JOHN"  → lower()                → "john"
lastName: "SMITH"  → lower()                → "smith"
email: "john smith@workday" → trim + lower → "johnsmith@workday"
```

---

### After Running Provisioning

1. Go to **Admin** → **Identity Management** → **Create Profiles**
2. Click **Preview**
3. Select a test user from ISC
4. Click **Preview**
5. You see what will be sent to target

**What you see:**
```
Identity Data              Transform Applied        What Gets Sent
firstName: "John"        → (none)                → givenName: "John"
email: "john@company.com" → lower()              → mail: "john@company.com"
username: "john.smith"   → substring(0,10)      → sAMAccountName: "john.smit"
```

---

## SEEING ERRORS

### If a Transform Fails

**During Aggregation:**
1. Check **Admin** → **System** → **Logs**
2. Search for error messages
3. Look for transform-related issues
4. Common errors:
   - `NullPointerException` - Data was NULL
   - `Invalid JSON` - Syntax error in transform
   - `Unknown attribute` - Referenced wrong attribute name

**During Provisioning:**
1. Check **Admin** → **System** → **Task Report**
2. Find the provisioning task
3. Click to see errors
4. Common errors:
   - `Transform failed` - Transform has error
   - `Invalid value for target` - Output doesn't match target requirements
   - `Null value not allowed` - Target system doesn't accept NULL

---

## IMPORTANT: Testing Transforms

### Before deploying transforms to production:

1. **Test in Preview:**
   - Go to Identity Profile Mapping or Create Profile
   - Click **Preview**
   - Select a test user
   - See what transform outputs
   - Verify it's correct

2. **Test with Multiple Users:**
   - Try with different data types
   - Try with NULL values
   - Try with special characters
   - Try with edge cases

3. **Check Logs:**
   - Deploy to test environment
   - Run against test data
   - Check Admin → System → Logs
   - Verify no errors

4. **Sign Off:**
   - Only deploy to production after testing
   - Document what was tested
   - Keep test results

---

## QUICK REFERENCE: Where Transforms Go

### Aggregation Flow (Source → ISC)
```
Source System
    ↓
Connector reads data
    ↓
Identity Profile Mapping
    ├─ Source Attribute (accountAttribute)
    ├─ [TRANSFORM HERE]
    └─ Identity Attribute (stored in ISC)
    ↓
ISC Identity Profile
```

### Provisioning Flow (ISC → Target)
```
ISC Identity Profile
    ↓
Create Profile
    ├─ Identity Attribute (identityAttribute)
    ├─ [TRANSFORM HERE]
    └─ Target Account Attribute
    ↓
Target System (AD, ServiceNow, etc.)
```

---

## KEY TAKEAWAYS

1. **Two places transforms live:**
   - Identity Profile Mapping (Aggregation)
   - Create Profile (Provisioning)

2. **Two different data sources available:**
   - Aggregation: Use accountAttribute
   - Provisioning: Use identityAttribute

3. **Test before production:**
   - Use Preview feature
   - Check logs for errors
   - Test with multiple users

4. **Can't mix contexts:**
   - Don't use identityAttribute in aggregation
   - Don't use accountAttribute in provisioning

5. **Know the flow:**
   - Where data comes from
   - What transform does
   - Where result goes

---

## WHAT'S NEXT

Now that you understand:
- ✅ What transforms are (STEP 01)
- ✅ Where transforms live and how they execute (STEP 02)

Next you'll learn:
- **STEP 03:** How to READ transform JSON
- **STEP 04:** How to WRITE valid transform JSON with Velocity

Ready to continue? Move to STEP 03.
