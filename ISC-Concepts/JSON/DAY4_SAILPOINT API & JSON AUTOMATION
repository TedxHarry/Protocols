# JSON MASTERY - DAY 4: SAILPOINT API & JSON AUTOMATION

## Welcome to Day 4!

Today you'll learn to control SailPoint ISC programmatically using its REST API with JSON!

**Today's Goal:** Master SailPoint API to automate everything with JSON

---

## Table of Contents
- [Morning Session: API Fundamentals](#morning-session-api-fundamentals)
  - [Part 1: Understanding REST APIs](#part-1-understanding-rest-apis-30-minutes)
  - [Part 2: SailPoint API Authentication](#part-2-sailpoint-api-authentication-45-minutes)
  - [Part 3: Your First API Call](#part-3-your-first-api-call-45-minutes)
- [Afternoon Session: CRUD Operations](#afternoon-session-crud-operations)
  - [Part 4: Reading Configurations (GET)](#part-4-reading-configurations-get-60-minutes)
  - [Part 5: Creating Transforms (POST)](#part-5-creating-transforms-post-60-minutes)
- [Evening Session: Advanced Operations](#evening-session-advanced-operations)
  - [Part 6: Updating & Deleting (PATCH/DELETE)](#part-6-updating--deleting-patchdelete-60-minutes)
- [Day 4 Assessment](#day-4-assessment)

---

## Morning Session: API Fundamentals

### Part 1: Understanding REST APIs (30 minutes)

Before diving into SailPoint API, let's understand what an API is!

---

#### What is an API?

**API = Application Programming Interface**

**Plain English:** A way for programs to talk to each other

**Real-World Analogy: Restaurant**
```
You (Client)
    ↓
"I want a burger" (Request)
    ↓
Waiter (API)
    ↓
Kitchen (Server)
    ↓
Makes burger
    ↓
Waiter brings burger (Response)
    ↓
You receive burger (Data)
```

---

#### REST API Basics

**REST = Representational State Transfer**

**Key Concepts:**

**1. URLs (Endpoints)**
```
https://api.example.com/v3/transforms
                          ↑
                       Endpoint
```

Think of endpoints like different restaurant menu items.

---

**2. HTTP Methods (Actions)**
```
GET     = Read (get menu)
POST    = Create (order food)
PATCH   = Update (modify order)
PUT     = Replace (change entire order)
DELETE  = Remove (cancel order)
```

---

**3. Request/Response**

**Request:**
```
Method: POST
URL: https://api.example.com/v3/transforms
Headers: {
  "Authorization": "Bearer token123",
  "Content-Type": "application/json"
}
Body: {
  "name": "email",
  "type": "static",
  "attributes": {
    "value": "test@company.com"
  }
}
```

**Response:**
```
Status: 201 Created
Body: {
  "id": "abc123",
  "name": "email",
  "type": "static",
  "attributes": {
    "value": "test@company.com"
  }
}
```

---

#### HTTP Status Codes

**Status codes tell you what happened:**
```
200 OK              = Success! Everything worked
201 Created         = Success! New thing created
204 No Content      = Success! Nothing to return
400 Bad Request     = You sent invalid data
401 Unauthorized    = You're not authenticated
403 Forbidden       = You don't have permission
404 Not Found       = That thing doesn't exist
429 Too Many Requests = Slow down! Rate limited
500 Server Error    = Server had a problem
```

**Remember:**
- 2xx = Success ✓
- 4xx = You did something wrong ✗
- 5xx = Server did something wrong ✗

---

#### JSON in APIs

**APIs use JSON for data exchange:**

**Request (JSON):**
```json
{
  "name": "John",
  "email": "john@company.com"
}
```

**Response (JSON):**
```json
{
  "id": "12345",
  "name": "John",
  "email": "john@company.com",
  "created": "2024-02-06T10:00:00Z"
}
```

**This is why you learned JSON!**

---

### Part 2: SailPoint API Authentication (45 minutes)

To use the SailPoint API, you need to authenticate (prove who you are).

---

#### Authentication Methods

**SailPoint ISC supports two methods:**

**Method 1: Personal Access Token (PAT)**
- Simple
- Good for testing
- Expires (needs renewal)
- Created in UI

**Method 2: OAuth 2.0**
- More secure
- Good for automation
- Client ID + Client Secret
- Programmatic

**We'll use PAT for learning (simpler)**

---

#### Creating a Personal Access Token

**Steps in SailPoint ISC UI:**
```
1. Login to SailPoint ISC
2. Click your profile icon (top right)
3. Go to "Preferences"
4. Click "Personal Access Tokens"
5. Click "New Token"
6. Enter name: "API Learning"
7. Select expiration: 90 days
8. Click "Create Token"
9. COPY THE TOKEN IMMEDIATELY (you can't see it again!)
10. Save it securely
```

**Your token looks like:**
```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

**⚠️ NEVER share your token! Treat it like a password!**

---

#### Understanding Your Token

**Your token contains:**
- Your identity
- Permissions
- Expiration date
- Tenant information

**It's like a temporary password that proves you are you**

---

#### Using the Token in API Calls

**Every API request needs the token in the header:**
```
Authorization: Bearer YOUR_TOKEN_HERE
```

**Full example:**
```
GET https://[tenant].api.identitynow.com/v3/transforms
Headers:
  Authorization: Bearer eyJhbGci...
  Content-Type: application/json
```

---

#### Finding Your Tenant Name

**Your SailPoint URL looks like:**
```
https://[tenant-name].identitynow.com

Example:
https://acme-corp.identitynow.com
         ↑
      tenant name
```

**API Base URL:**
```
https://[tenant-name].api.identitynow.com

Example:
https://acme-corp.api.identitynow.com
```

---

### Part 3: Your First API Call (45 minutes)

Let's make your first SailPoint API call!

---

#### Tool: Postman (Free API Client)

**Download Postman:**
```
1. Go to postman.com
2. Download for your OS
3. Install and open
4. Create free account (optional but recommended)
```

**Why Postman?**
```
✓ Visual interface
✓ Save requests
✓ Test APIs easily
✓ See formatted responses
✓ Industry standard
```

---

#### Setting Up Postman for SailPoint

**Step 1: Create New Request**
```
1. Click "New" → "HTTP Request"
2. Name it: "Get All Transforms"
3. Save to a collection: "SailPoint ISC"
```

**Step 2: Configure the Request**
```
Method: GET
URL: https://[YOUR-TENANT].api.identitynow.com/v3/transforms
```

**Step 3: Add Authorization Header**
```
1. Click "Headers" tab
2. Add header:
   Key: Authorization
   Value: Bearer [YOUR-TOKEN]
3. Add header:
   Key: Content-Type
   Value: application/json
```

**Step 4: Send Request**
```
1. Click "Send" button
2. Wait for response
3. See your transforms in JSON!
```

---

#### Your First Response

**You should see something like:**
```json
[
  {
    "id": "2cd78adghjkja",
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
  },
  {
    "id": "8f3h2kdjsk3j",
    "name": "email",
    "type": "static",
    "attributes": {
      "value": "test@company.com"
    }
  }
]
```

**🎉 Congratulations! You just called the SailPoint API!**

---

#### Understanding the Response

**This is an array of transform objects:**
```json
[
  {transform 1},
  {transform 2},
  {transform 3}
]
```

**Each transform has:**
- `id` - Unique identifier
- `name` - Attribute name
- `type` - Transform type
- `attributes` - Configuration

---

#### Alternative: Using curl (Command Line)

**If you prefer command line:**
```bash
curl -X GET \
  'https://[tenant].api.identitynow.com/v3/transforms' \
  -H 'Authorization: Bearer [YOUR-TOKEN]' \
  -H 'Content-Type: application/json'
```

**Response:** Same JSON as Postman

---

#### Common First-Call Issues

**Issue 1: 401 Unauthorized**
```
Problem: Token is invalid or expired
Solution: Generate new token in UI
```

**Issue 2: 404 Not Found**
```
Problem: Wrong URL/tenant name
Solution: Check tenant name in URL
```

**Issue 3: No response**
```
Problem: Network issue or wrong URL
Solution: Check internet, verify URL
```

---

### Practice Exercise 1: Make Your First Call

**Task: Get all transforms from your SailPoint instance**

**Steps:**
1. Create Personal Access Token
2. Open Postman
3. Create GET request
4. Add Authorization header
5. Send request
6. Save the response

**Questions:**
- How many transforms do you have?
- What is the name of the first transform?
- What type is it?

---

## Afternoon Session: CRUD Operations

### Part 4: Reading Configurations (GET) (60 minutes)

**CRUD = Create, Read, Update, Delete**

Let's master **READ** operations with GET requests.

---

#### API Endpoint Reference

**Here are the main SailPoint API endpoints:**

| Endpoint | What It Does | Method |
|----------|--------------|--------|
| `/v3/transforms` | List all transforms | GET |
| `/v3/transforms/{id}` | Get specific transform | GET |
| `/v3/identity-profiles` | List all profiles | GET |
| `/v3/identity-profiles/{id}` | Get specific profile | GET |
| `/v3/sources` | List all sources | GET |
| `/v3/sources/{id}` | Get specific source | GET |
| `/v3/identities` | List all identities | GET |
| `/v3/identities/{id}` | Get specific identity | GET |

**Full API docs:** https://developer.sailpoint.com/docs/api/v3/

---

#### GET Request 1: List All Transforms

**Request:**
```
GET https://[tenant].api.identitynow.com/v3/transforms
Headers:
  Authorization: Bearer [token]
  Content-Type: application/json
```

**Response:**
```json
[
  {
    "id": "transform-1-id",
    "name": "displayName",
    "type": "concat",
    "attributes": {...}
  },
  {
    "id": "transform-2-id",
    "name": "email",
    "type": "static",
    "attributes": {...}
  }
]
```

---

#### GET Request 2: Get Specific Transform

**Request:**
```
GET https://[tenant].api.identitynow.com/v3/transforms/[transform-id]
Headers:
  Authorization: Bearer [token]
  Content-Type: application/json
```

**Response:**
```json
{
  "id": "transform-id",
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
          "value": "@company.com"
        }
      }
    ]
  }
}
```

---

#### GET Request 3: List Identity Profiles

**Request:**
```
GET https://[tenant].api.identitynow.com/v3/identity-profiles
Headers:
  Authorization: Bearer [token]
  Content-Type: application/json
```

**Response:**
```json
[
  {
    "id": "profile-id",
    "name": "Employees",
    "description": "All employees from Workday",
    "priority": 10,
    "sourceIds": ["workday-source-id"],
    "identityAttributeConfig": {
      "attributeTransforms": [...]
    }
  }
]
```

---

#### GET Request 4: Search/Filter Results

**SailPoint supports filtering:**

**Get transforms by name:**
```
GET https://[tenant].api.identitynow.com/v3/transforms?filters=name eq "email"
```

**Get transforms by type:**
```
GET https://[tenant].api.identitynow.com/v3/transforms?filters=type eq "concat"
```

**Pagination (limit results):**
```
GET https://[tenant].api.identitynow.com/v3/transforms?limit=10&offset=0
```

---

#### Saving API Responses

**In Postman:**
```
1. Send request
2. Click response body
3. Click "Save Response" → "Save to File"
4. Choose location
5. Save as: transform-backup.json
```

**Now you have a backup!**

---

#### Processing JSON Responses

**Example: Extract all transform names**

**Using Python:**
```python
import requests
import json

# API call
url = "https://[tenant].api.identitynow.com/v3/transforms"
headers = {
    "Authorization": "Bearer [token]",
    "Content-Type": "application/json"
}

response = requests.get(url, headers=headers)
transforms = response.json()

# Extract names
names = [t["name"] for t in transforms]
print(names)
# Output: ['displayName', 'email', 'username', ...]
```

**Using JavaScript:**
```javascript
fetch('https://[tenant].api.identitynow.com/v3/transforms', {
  headers: {
    'Authorization': 'Bearer [token]',
    'Content-Type': 'application/json'
  }
})
.then(response => response.json())
.then(transforms => {
  const names = transforms.map(t => t.name);
  console.log(names);
});
```

---

### Practice Exercise 2: GET Operations

**Complete these GET requests:**

1. Get all transforms
2. Find a transform with type "concat"
3. Get that specific transform by ID
4. Save the response to a file
5. Get all identity profiles
6. Find the profile for employees

---

### Part 5: Creating Transforms (POST) (60 minutes)

Now let's **CREATE** new transforms via API!

---

#### POST Request Basics

**POST = Create new resource**

**Structure:**
```
POST [endpoint]
Headers: {authentication + content type}
Body: {JSON data for new thing}
```

---

#### Creating a Simple Transform

**Create a static transform:**

**Request:**
```
POST https://[tenant].api.identitynow.com/v3/transforms
Headers:
  Authorization: Bearer [token]
  Content-Type: application/json
Body:
{
  "name": "defaultCountry",
  "type": "static",
  "attributes": {
    "value": "United States"
  }
}
```

**Response (201 Created):**
```json
{
  "id": "newly-generated-id",
  "name": "defaultCountry",
  "type": "static",
  "attributes": {
    "value": "United States"
  }
}
```

**🎉 You just created a transform via API!**

---

#### Step-by-Step in Postman

**1. Create New Request**
```
Click "New" → "HTTP Request"
Name: "Create Transform"
```

**2. Set Method and URL**
```
Method: POST
URL: https://[tenant].api.identitynow.com/v3/transforms
```

**3. Add Headers**
```
Authorization: Bearer [token]
Content-Type: application/json
```

**4. Add Body**
```
Click "Body" tab
Select "raw"
Select "JSON" from dropdown
Paste JSON:
{
  "name": "testTransform",
  "type": "static",
  "attributes": {
    "value": "test"
  }
}
```

**5. Send Request**
```
Click "Send"
Check response status: 201 Created
See the new transform with ID
```

**6. Verify in UI**
```
Go to SailPoint ISC UI
Navigate to Identity Profiles
Check transforms list
See "testTransform" there!
```

---

#### Creating Complex Transforms

**Create concatenation transform:**
```json
{
  "name": "fullName",
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

**POST this to `/v3/transforms`**

---

#### Creating Chained Transforms

**Create email with lowercase:**
```json
{
  "name": "emailLowercase",
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
    ],
    "next": {
      "type": "lower"
    }
  }
}
```

---

#### Common POST Errors

**Error 1: 400 Bad Request - Invalid JSON**
```json
Response:
{
  "error": "Invalid JSON"
}

Problem: JSON syntax error
Solution: Validate JSON in JSONLint first
```

**Error 2: 400 Bad Request - Missing Required Field**
```json
Response:
{
  "error": "Missing required field: type"
}

Problem: Didn't include "type"
Solution: Add all required fields (name, type, attributes)
```

**Error 3: 409 Conflict - Transform Already Exists**
```json
Response:
{
  "error": "Transform with name 'email' already exists"
}

Problem: Transform name already used
Solution: Use different name or update existing one
```

---

#### Bulk Transform Creation

**Create multiple transforms at once:**

**Python Script:**
```python
import requests
import json

url = "https://[tenant].api.identitynow.com/v3/transforms"
headers = {
    "Authorization": "Bearer [token]",
    "Content-Type": "application/json"
}

transforms = [
    {
        "name": "defaultCountry",
        "type": "static",
        "attributes": {"value": "United States"}
    },
    {
        "name": "defaultTimezone",
        "type": "static",
        "attributes": {"value": "America/Chicago"}
    },
    {
        "name": "defaultCurrency",
        "type": "static",
        "attributes": {"value": "USD"}
    }
]

for transform in transforms:
    response = requests.post(url, headers=headers, json=transform)
    if response.status_code == 201:
        print(f"Created: {transform['name']}")
    else:
        print(f"Failed: {transform['name']} - {response.text}")
```

**Run this script → Creates 3 transforms instantly!**

---

### Practice Exercise 3: POST Operations

**Create these 3 transforms via API:**

1. **defaultLanguage**
   - Type: static
   - Value: "en_US"

2. **displayNameUpper**
   - Type: concat (firstname + space + lastname)
   - Chain to: upper

3. **emailFallback**
   - Type: firstValid
   - Try: personal_email → work_email → "noemail@company.com"

<details>
<summary>Click for answers</summary>

**Transform 1:**
```json
{
  "name": "defaultLanguage",
  "type": "static",
  "attributes": {
    "value": "en_US"
  }
}
```

**Transform 2:**
```json
{
  "name": "displayNameUpper",
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
    ],
    "next": {
      "type": "upper"
    }
  }
}
```

**Transform 3:**
```json
{
  "name": "emailFallback",
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
          "name": "work_email"
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

</details>

---

## Evening Session: Advanced Operations

### Part 6: Updating & Deleting (PATCH/DELETE) (60 minutes)

Complete the CRUD operations with **UPDATE** and **DELETE**!

---

#### PATCH vs PUT

**Two ways to update:**

**PATCH = Partial update**
```
Only send fields you want to change
Other fields stay the same
```

**PUT = Full replacement**
```
Send complete new object
Replaces everything
```

**SailPoint uses PATCH for transforms**

---

#### Updating a Transform (PATCH)

**Scenario: Change static value**

**Original transform:**
```json
{
  "id": "abc123",
  "name": "defaultCountry",
  "type": "static",
  "attributes": {
    "value": "United States"
  }
}
```

**Update Request:**
```
PATCH https://[tenant].api.identitynow.com/v3/transforms/abc123
Headers:
  Authorization: Bearer [token]
  Content-Type: application/json
Body:
{
  "attributes": {
    "value": "Canada"
  }
}
```

**Response (200 OK):**
```json
{
  "id": "abc123",
  "name": "defaultCountry",
  "type": "static",
  "attributes": {
    "value": "Canada"
  }
}
```

**Only the value changed!**

---

#### Updating Complex Transforms

**Scenario: Add step to chain**

**Original:**
```json
{
  "id": "def456",
  "name": "email",
  "type": "concat",
  "attributes": {
    "values": [...]
  }
}
```

**Update to add lowercase:**
```
PATCH https://[tenant].api.identitynow.com/v3/transforms/def456
Body:
{
  "attributes": {
    "values": [...],
    "next": {
      "type": "lower"
    }
  }
}
```

**Now email is lowercased!**

---

#### Full Transform Replacement

**To completely change a transform:**

**Step 1: Get current transform**
```
GET /v3/transforms/[id]
```

**Step 2: Modify entire JSON**
```json
{
  "name": "email",
  "type": "firstValid",
  "attributes": {
    "values": [...]
  }
}
```

**Step 3: PATCH with complete new definition**
```
PATCH /v3/transforms/[id]
Body: {complete new JSON}
```

---

#### Deleting a Transform (DELETE)

**⚠️ Warning: Deletion is permanent!**

**Request:**
```
DELETE https://[tenant].api.identitynow.com/v3/transforms/[transform-id]
Headers:
  Authorization: Bearer [token]
```

**Response (204 No Content):**
```
(Empty response body)
Status: 204 No Content
```

**Transform is now gone!**

---

#### Safe Deletion Workflow

**Before deleting, check dependencies:**

**Step 1: Search where transform is used**
```
1. Check Identity Profiles
2. Check other transforms (that might reference it)
3. Check provisioning policies
```

**Step 2: Export backup**
```
GET /v3/transforms/[id]
Save response to file
```

**Step 3: Delete**
```
DELETE /v3/transforms/[id]
```

**Step 4: Verify**
```
Try to GET it (should return 404)
```

---

#### Bulk Update Script

**Update multiple transforms at once:**

**Python Example:**
```python
import requests
import json

url_base = "https://[tenant].api.identitynow.com/v3/transforms"
headers = {
    "Authorization": "Bearer [token]",
    "Content-Type": "application/json"
}

# Get all static transforms
response = requests.get(
    f"{url_base}?filters=type eq 'static'",
    headers=headers
)
transforms = response.json()

# Update each one (example: add suffix to value)
for transform in transforms:
    transform_id = transform["id"]
    current_value = transform["attributes"]["value"]
    new_value = f"{current_value} - Updated"
    
    patch_data = {
        "attributes": {
            "value": new_value
        }
    }
    
    response = requests.patch(
        f"{url_base}/{transform_id}",
        headers=headers,
        json=patch_data
    )
    
    if response.status_code == 200:
        print(f"Updated: {transform['name']}")
    else:
        print(f"Failed: {transform['name']}")
```

---

#### Bulk Delete Script

**Delete all test transforms:**

**Python Example:**
```python
import requests

url_base = "https://[tenant].api.identitynow.com/v3/transforms"
headers = {
    "Authorization": "Bearer [token]"
}

# Get all transforms with "test" in name
response = requests.get(
    f"{url_base}?filters=name co 'test'",
    headers=headers
)
transforms = response.json()

# Delete each one
for transform in transforms:
    transform_id = transform["id"]
    
    response = requests.delete(
        f"{url_base}/{transform_id}",
        headers=headers
    )
    
    if response.status_code == 204:
        print(f"Deleted: {transform['name']}")
    else:
        print(f"Failed to delete: {transform['name']}")
```

---

### Practice Exercise 4: Update & Delete

**Tasks:**

1. Create a test transform
2. Update its value via PATCH
3. Verify the update via GET
4. Delete the transform
5. Verify deletion (GET should return 404)

---

#### Environment Promotion Workflow

**Problem:** How to move transforms from Dev to Prod?

**Solution: Export → Modify → Import**

**Step 1: Export from Dev**
```python
# Get all transforms from Dev
dev_url = "https://dev-tenant.api.identitynow.com/v3/transforms"
dev_headers = {"Authorization": "Bearer [dev-token]"}

response = requests.get(dev_url, headers=dev_headers)
dev_transforms = response.json()

# Save to file
with open("dev_transforms.json", "w") as f:
    json.dump(dev_transforms, f, indent=2)
```

**Step 2: Modify for Prod (if needed)**
```python
# Read transforms
with open("dev_transforms.json", "r") as f:
    transforms = json.load(f)

# Modify (example: change domain)
for transform in transforms:
    if transform["type"] == "concat":
        for value in transform["attributes"].get("values", []):
            if value.get("type") == "static":
                if "@dev.com" in value["attributes"]["value"]:
                    value["attributes"]["value"] = value["attributes"]["value"].replace("@dev.com", "@prod.com")
```

**Step 3: Import to Prod**
```python
# Create in Prod
prod_url = "https://prod-tenant.api.identitynow.com/v3/transforms"
prod_headers = {"Authorization": "Bearer [prod-token]"}

for transform in transforms:
    # Remove ID (will be generated)
    transform.pop("id", None)
    
    # Create in Prod
    response = requests.post(
        prod_url,
        headers=prod_headers,
        json=transform
    )
    
    if response.status_code == 201:
        print(f"Created in Prod: {transform['name']}")
    else:
        print(f"Failed: {transform['name']}")
```

---

### Advanced: Transaction Safety

**Problem:** What if bulk operation fails halfway?

**Solution: Rollback capability**
```python
import requests
import json

created_ids = []

try:
    for transform in transforms:
        response = requests.post(url, headers=headers, json=transform)
        response.raise_for_status()  # Raises exception if not 2xx
        
        created_id = response.json()["id"]
        created_ids.append(created_id)
        print(f"Created: {transform['name']}")

except Exception as e:
    print(f"Error occurred: {e}")
    print("Rolling back...")
    
    # Delete all created transforms
    for transform_id in created_ids:
        requests.delete(f"{url}/{transform_id}", headers=headers)
    
    print("Rollback complete")
```

---

## Day 4 Assessment

### Knowledge Check

#### Question 1: HTTP Methods

Match the HTTP method to its purpose:

1. GET
2. POST
3. PATCH
4. DELETE

A. Create new resource
B. Read/retrieve resource
C. Update existing resource
D. Remove resource

<details>
<summary>Click to reveal answer</summary>

1. GET = B (Read/retrieve)
2. POST = A (Create new)
3. PATCH = C (Update existing)
4. DELETE = D (Remove)

</details>

---

#### Question 2: Status Codes

What does each status code mean?

1. 200
2. 201
3. 400
4. 401
5. 404

<details>
<summary>Click to reveal answer</summary>

1. 200 = OK (success)
2. 201 = Created (new resource created)
3. 400 = Bad Request (invalid data sent)
4. 401 = Unauthorized (authentication failed)
5. 404 = Not Found (resource doesn't exist)

</details>

---

#### Question 3: Create Transform via API

Write the complete API request to create a static transform named "testCountry" with value "USA".

<details>
<summary>Click to reveal answer</summary>
```
Method: POST
URL: https://[tenant].api.identitynow.com/v3/transforms
Headers:
  Authorization: Bearer [token]
  Content-Type: application/json
Body:
{
  "name": "testCountry",
  "type": "static",
  "attributes": {
    "value": "USA"
  }
}
```

</details>

---

### Final Challenge: Build an Automation Script

**Goal:** Create a script that:

1. Gets all transforms
2. Finds all static transforms
3. Backs them up to a JSON file
4. Creates a new static transform
5. Updates an existing static transform
6. Deletes a test transform

**Choose your language: Python, JavaScript, or bash with curl**

<details>
<summary>Click for Python solution</summary>
```python
import requests
import json
from datetime import datetime

# Configuration
TENANT = "your-tenant"
TOKEN = "your-token"
BASE_URL = f"https://{TENANT}.api.identitynow.com/v3"
HEADERS = {
    "Authorization": f"Bearer {TOKEN}",
    "Content-Type": "application/json"
}

# 1. Get all transforms
print("1. Getting all transforms...")
response = requests.get(f"{BASE_URL}/transforms", headers=HEADERS)
all_transforms = response.json()
print(f"   Found {len(all_transforms)} transforms")

# 2. Find all static transforms
print("2. Filtering static transforms...")
static_transforms = [t for t in all_transforms if t["type"] == "static"]
print(f"   Found {len(static_transforms)} static transforms")

# 3. Backup to file
print("3. Backing up to file...")
backup_file = f"backup_{datetime.now().strftime('%Y%m%d_%H%M%S')}.json"
with open(backup_file, "w") as f:
    json.dump(static_transforms, f, indent=2)
print(f"   Saved to {backup_file}")

# 4. Create new transform
print("4. Creating new transform...")
new_transform = {
    "name": "apiTestTransform",
    "type": "static",
    "attributes": {
        "value": "Created via API"
    }
}
response = requests.post(
    f"{BASE_URL}/transforms",
    headers=HEADERS,
    json=new_transform
)
if response.status_code == 201:
    new_id = response.json()["id"]
    print(f"   Created with ID: {new_id}")
else:
    print(f"   Failed: {response.text}")

# 5. Update existing transform
if static_transforms:
    print("5. Updating first static transform...")
    first_id = static_transforms[0]["id"]
    update_data = {
        "attributes": {
            "value": "Updated via API"
        }
    }
    response = requests.patch(
        f"{BASE_URL}/transforms/{first_id}",
        headers=HEADERS,
        json=update_data
    )
    if response.status_code == 200:
        print(f"   Updated transform: {static_transforms[0]['name']}")
    else:
        print(f"   Failed: {response.text}")

# 6. Delete test transform
print("6. Deleting test transform...")
response = requests.delete(
    f"{BASE_URL}/transforms/{new_id}",
    headers=HEADERS
)
if response.status_code == 204:
    print(f"   Deleted transform")
else:
    print(f"   Failed to delete")

print("\nDone!")
```

</details>

---

## Day 4 Summary

### What You Learned Today

**Morning:**
- ✅ REST API fundamentals
- ✅ SailPoint API authentication
- ✅ Personal Access Tokens
- ✅ Making your first API call

**Afternoon:**
- ✅ GET requests (reading configurations)
- ✅ POST requests (creating transforms)
- ✅ Filtering and pagination
- ✅ Bulk operations

**Evening:**
- ✅ PATCH requests (updating transforms)
- ✅ DELETE requests (removing transforms)
- ✅ Environment promotion
- ✅ Automation scripts

---

### Skills Acquired

You can now:
```
✓ Authenticate to SailPoint API
✓ Read all configurations via GET
✓ Create transforms via POST
✓ Update transforms via PATCH
✓ Delete transforms via DELETE
✓ Filter and search results
✓ Write automation scripts
✓ Promote configs across environments
✓ Bulk create/update/delete
✓ Build rollback capabilities
```

---

### API Quick Reference

**Authentication:**
```
Header: Authorization: Bearer [token]
```

**Endpoints:**
```
GET    /v3/transforms           List all
GET    /v3/transforms/{id}      Get one
POST   /v3/transforms           Create
PATCH  /v3/transforms/{id}      Update
DELETE /v3/transforms/{id}      Delete
```

**Status Codes:**
```
200 OK
201 Created
204 No Content
400 Bad Request
401 Unauthorized
404 Not Found
```

---

## Homework

### Exercise 1: Build Complete CRUD Script

Create a script that performs full CRUD cycle:
1. Create a transform
2. Read it back
3. Update it
4. Delete it
5. Verify deletion

---

### Exercise 2: Backup System

Build a backup script that:
1. Gets all transforms
2. Saves to dated JSON file
3. Runs daily (cron/scheduled task)
4. Keeps last 7 days of backups

---

### Exercise 3: Deployment Pipeline

Create a Dev → Prod promotion script:
1. Export from Dev
2. Validate JSON
3. Import to Prod
4. Verify in Prod
5. Rollback on error

---

## Tomorrow: Day 5 Preview

**What You'll Learn:**
```
Advanced JSON & Mastery
├─ JSON Schema validation
├─ Complex nested patterns
├─ Performance optimization
├─ Error handling best practices
├─ JSON diff and merge
└─ Final Capstone Project
```

**You'll be able to:**
- Validate JSON with schemas
- Handle complex structures
- Optimize for performance
- Build production-grade systems
- Complete JSON mastery

---

**Congratulations on completing Day 4! You now control SailPoint via API!**

**One more day to JSON mastery!**

---

*End of JSON Mastery - Day 4*
