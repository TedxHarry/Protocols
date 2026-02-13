# Section 1.2: API and Integration

**Master API authentication and understand how to integrate with SailPoint Identity Security Cloud programmatically**

---

## Table of Contents

- [1.2.1 REST API Authentication Methods](#121-rest-api-authentication-methods)
- [1.2.2 Understanding API Endpoints and Use Cases](#122-understanding-api-endpoints-and-use-cases)

---

# 1.2.1 REST API Authentication Methods

**Learning Objective:** Understand how to authenticate to the SailPoint ISC API using Personal Access Tokens and OAuth.

---

## What is the SailPoint API?

The SailPoint Identity Security Cloud API is a RESTful API that allows you to interact with ISC programmatically.

**What is an API?**
- **API** = Application Programming Interface
- A way for programs to talk to each other
- Allows automation of tasks
- Enables custom integrations

**Why use the API?**
- Automate repetitive tasks
- Build custom applications
- Integrate with other systems
- Extract data for reporting
- Bulk operations

**What is REST?**
- **REST** = Representational State Transfer
- A standard way APIs work on the web
- Uses HTTP (like web browsers)
- Returns data in JSON format

---

## Understanding Authentication

Before you can use the API, you must prove who you are.

**What is Authentication?**
- Proving your identity to ISC
- Like logging in, but for programs
- Required for every API call

**Why is Authentication Important?**
- Security - only authorized users can access
- Permissions - controls what you can do
- Audit trail - tracks who did what

---

## Two Authentication Methods in ISC

SailPoint ISC supports two main authentication methods:

| Method | Best For | Common Use Case |
|--------|----------|-----------------|
| **Personal Access Token (PAT)** | Testing, personal scripts | Developer testing, one-off scripts |
| **OAuth Client Credentials** | Production integrations | Automated systems, service accounts |

---

## Method 1: Personal Access Token (PAT)

### **What is a PAT?**

A Personal Access Token is a credential tied to YOUR user account.

**Think of it like:**
- A special password for programs
- Works like your user login
- Has YOUR permissions
- Tied to YOUR account

### **When to Use PAT**

✅ **Good for:**
- Learning and testing
- Personal automation scripts
- Development work
- Short-term projects

❌ **Not good for:**
- Production systems
- Shared applications
- Long-term integrations
- Service accounts

### **Creating a Personal Access Token**

**Step 1: Navigate to PAT Management**

1. Log into SailPoint ISC
2. Click your profile (top right)
3. Select **Preferences**
4. Click **Personal Access Tokens**

**Step 2: Create New Token**

1. Click **New Token** or **Create Token**
2. Fill in token details

**Step 3: Configure Token**
```
Token Name: My Development Token
Description: Token for learning API basics
Expiration: 90 days (or custom date)
Scope: Full access (inherits your permissions)
```

**Important notes:**
- Name should describe purpose
- Set appropriate expiration date
- Can't be edited after creation

**Step 4: Save and Copy Credentials**

After clicking **Create**, you'll see:
```
Client ID: a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6
Client Secret: z9y8x7w6v5u4t3s2r1q0p9o8n7m6l5k4j3i2h1g0f9e8d7c6b5a4
```

### **⚠️ CRITICAL WARNING**

**The Client Secret is shown ONLY ONCE!**

- Copy it immediately
- Save it securely
- You cannot view it again
- If lost, create a new token

**Where to save:**
- Password manager
- Secure notes
- Environment variables
- NOT in code or emails

### **Understanding PAT Components**

**Client ID:**
- Public identifier
- Safe to share in logs
- Used to identify your token

**Client Secret:**
- Private password
- NEVER share or expose
- Required for authentication
- Treat like a password

---

## Using Your Personal Access Token

### **Step 1: Get an Access Token**

Before calling ISC APIs, exchange your PAT for a temporary access token.

**Why this extra step?**
- Access tokens expire after 1 hour
- More secure than using PAT directly
- Standard OAuth practice

**The Request:**
```bash
curl --location --request POST 'https://yourcompany.api.identitynow.com/oauth/token' \
  --header 'Content-Type: application/x-www-form-urlencoded' \
  --data-urlencode 'grant_type=client_credentials' \
  --data-urlencode 'client_id=YOUR_CLIENT_ID' \
  --data-urlencode 'client_secret=YOUR_CLIENT_SECRET'
```

**Replace:**
- `yourcompany` with your tenant name
- `YOUR_CLIENT_ID` with your actual client ID
- `YOUR_CLIENT_SECRET` with your actual client secret

**The Response:**
```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "Bearer",
  "expires_in": 3600
}
```

**Understanding the response:**
- `access_token` - Use this for API calls
- `token_type` - Always "Bearer"
- `expires_in` - 3600 seconds = 1 hour

### **Step 2: Use Access Token in API Calls**

Now use the access token to call ISC APIs.

**Example: List Identities**
```bash
curl --location --request GET 'https://yourcompany.api.identitynow.com/v3/identities?limit=5' \
  --header 'Authorization: Bearer YOUR_ACCESS_TOKEN'
```

**Replace:**
- `YOUR_ACCESS_TOKEN` with the token from Step 1

**The Response:**
```json
[
  {
    "id": "2c9180887f4e4b52017f504f30d80234",
    "name": "john.smith",
    "displayName": "John Smith",
    "email": "john.smith@company.com",
    "cloudLifecycleState": "active"
  }
]
```

---

## Understanding API URLs

### **ISC API URL Structure**
```
https://{tenant-name}.api.identitynow.com/{version}/{endpoint}
```

**Components:**

| Part | Description | Example |
|------|-------------|---------|
| `{tenant-name}` | Your organization | `acme-corp` |
| `api.identitynow.com` | ISC API domain | Fixed |
| `{version}` | API version | `v3`, `beta` |
| `{endpoint}` | What you're accessing | `identities`, `accounts` |

**Complete example:**
```
https://acme-corp.api.identitynow.com/v3/identities
```

### **Finding Your Tenant Name**

Your tenant name is in your ISC login URL:
```
Login URL: https://acme-corp.identitynow.com
Tenant Name: acme-corp

API URL: https://acme-corp.api.identitynow.com
```

---

## Method 2: OAuth Client Credentials

### **What are OAuth Client Credentials?**

OAuth clients are application-level credentials NOT tied to a specific user.

**Think of it like:**
- A service account
- System-to-system authentication
- Independent of individual users
- Production-ready authentication

### **When to Use OAuth Clients**

✅ **Good for:**
- Production integrations
- Automated systems
- Service accounts
- Long-term applications
- Third-party integrations

❌ **Not good for:**
- Quick testing (use PAT instead)
- Learning (use PAT instead)
- Personal scripts

### **Creating an OAuth Client**

**Step 1: Navigate to OAuth Clients**

1. Log into ISC as admin
2. Click **Admin** menu
3. Go to **Security**
4. Select **API Management**
5. Click **OAuth Clients**

**Step 2: Create New Client**

1. Click **New Client** or **Create Client**
2. Fill in client details

**Step 3: Configure OAuth Client**
```
Client Name: Production HR Integration
Description: OAuth client for HR system integration
Grant Type: Client Credentials
Enabled: Yes
```

**Step 4: Assign Scopes**

Scopes control what the client can do.

**Important Principle: Least Privilege**
- Give ONLY the permissions needed
- Don't use "full access" in production
- Be specific with scopes

**Example - Limited Scopes:**
```
Selected Scopes:
☑ sp:identity:read
☑ sp:account:read
☐ sp:identity:write (not needed)
☐ sp:account:write (not needed)
☐ sp:scopes:all (too broad!)
```

**Step 5: Save Client Credentials**

After creation, you'll see:
```
Client ID: xyz789abc123def456ghi789jkl012mno345
Client Secret: s3cr3tk3yth4ty0umu5ts4v3s3cur31yf0rpr0duct10n
```

**⚠️ Same warning as PAT:**
- Client Secret shown ONLY ONCE
- Save it immediately
- Cannot retrieve later

---

## Understanding API Scopes

Scopes define what permissions the API client has.

### **Common Scopes**

| Scope | What It Allows | Use Case |
|-------|----------------|----------|
| `sp:scopes:all` | Everything | Admin tools (use carefully!) |
| `sp:identity:read` | Read identities | Reporting |
| `sp:identity:write` | Create/modify identities | Provisioning |
| `sp:account:read` | Read accounts | Auditing |
| `sp:account:write` | Create/modify accounts | Account management |
| `sp:entitlement:read` | Read entitlements | Access reviews |
| `sp:access-request:create` | Submit access requests | Self-service portals |

### **Choosing the Right Scopes**

**Example 1: Read-Only Reporting Tool**

**Needs:**
- Read identities
- Read accounts
- Read access

**Scopes to assign:**
```
☑ sp:identity:read
☑ sp:account:read
☑ sp:entitlement:read
☐ sp:identity:write (not needed)
```

**Example 2: Provisioning Integration**

**Needs:**
- Read and write identities
- Read and write accounts
- Create access requests

**Scopes to assign:**
```
☑ sp:identity:read
☑ sp:identity:write
☑ sp:account:read
☑ sp:account:write
☑ sp:access-request:create
```

---

## Token Lifecycle and Expiration

### **Understanding Token Expiration**

**Access tokens expire after 1 hour (3600 seconds)**

**What this means:**
- Must request new token when expired
- Can't use same token forever
- Normal and expected behavior

### **Token Renewal Pattern**

**Bad approach (manual):**
```
1. Get access token
2. Use for API calls
3. Token expires
4. API calls fail
5. Manually get new token
```

**Good approach (automatic):**
```
1. Get access token
2. Track expiration time
3. Use for API calls
4. Before expiration, get new token
5. Continue seamlessly
```

### **Simple Token Renewal Logic**

**Pseudocode:**
```
if (current_time >= token_expiry_time - 5_minutes):
    get_new_access_token()
    update_expiry_time()

make_api_call_with_current_token()
```

**Why subtract 5 minutes?**
- Safety buffer
- Prevents using expired token
- Accounts for clock differences

---

## Security Best Practices

### **Storing Credentials Securely**

**❌ NEVER DO THIS:**
```python
# Hardcoded in script - VERY BAD!
client_id = "abc123"
client_secret = "supersecret"
```
```javascript
// In source code - VERY BAD!
const CLIENT_SECRET = "supersecret";
```

**✅ DO THIS INSTEAD:**

**Option 1: Environment Variables**
```python
import os

client_id = os.getenv('SAILPOINT_CLIENT_ID')
client_secret = os.getenv('SAILPOINT_CLIENT_SECRET')
```

**Option 2: Configuration File (encrypted)**
```python
import json

with open('config.json') as f:
    config = json.load(f)
    client_id = config['client_id']
    client_secret = config['client_secret']
```

**Option 3: Secret Management System**
```python
# AWS Secrets Manager, Azure Key Vault, etc.
from azure.keyvault.secrets import SecretClient

secret = vault_client.get_secret("sailpoint-client-secret").value
```

### **Security Rules**

✅ **Always:**
- Store credentials in environment variables or secret managers
- Use HTTPS (never HTTP)
- Rotate credentials regularly (every 90 days)
- Use least privilege scopes
- Monitor API usage
- Delete unused tokens

❌ **Never:**
- Hardcode credentials in code
- Commit credentials to Git/GitHub
- Share credentials via email
- Use production credentials for testing
- Give full access when not needed
- Leave credentials in log files

---

## Testing Your Authentication

### **Quick Test: List Identities**

**Step 1: Get Access Token**
```bash
curl -X POST 'https://yourcompany.api.identitynow.com/oauth/token' \
  -H 'Content-Type: application/x-www-form-urlencoded' \
  -d 'grant_type=client_credentials' \
  -d 'client_id=YOUR_CLIENT_ID' \
  -d 'client_secret=YOUR_CLIENT_SECRET'
```

**Step 2: Save Access Token from Response**

Look for `access_token` in the response

**Step 3: Test API Call**
```bash
curl -X GET 'https://yourcompany.api.identitynow.com/v3/identities?limit=5' \
  -H 'Authorization: Bearer YOUR_ACCESS_TOKEN'
```

**Expected Result:**

If successful, you'll see a list of identities in JSON format.

If failed, check:
- Client ID is correct
- Client Secret is correct
- Tenant name is correct
- Token hasn't expired

---

## Common Authentication Errors

### **Error 1: 401 Unauthorized**

**Error message:**
```json
{
  "error": "invalid_client",
  "error_description": "Client authentication failed"
}
```

**Possible causes:**
- Wrong Client ID
- Wrong Client Secret
- Credentials expired/deleted

**Solution:**
- Verify Client ID and Secret
- Check if token still exists in ISC
- Create new token if needed

### **Error 2: 403 Forbidden**

**Error message:**
```json
{
  "error": "insufficient_scope",
  "error_description": "Insufficient permissions"
}
```

**Possible causes:**
- OAuth client lacks required scope
- PAT user lacks permission
- Trying to access restricted resource

**Solution:**
- Check OAuth client scopes
- Verify user permissions
- Request additional scopes if needed

### **Error 3: Token Expired**

**Error message:**
```json
{
  "error": "invalid_token",
  "error_description": "Access token expired"
}
```

**Cause:**
- Access token is older than 1 hour

**Solution:**
- Get new access token
- Implement automatic token renewal

### **Error 4: Invalid Request**

**Error message:**
```json
{
  "error": "invalid_request",
  "error_description": "Missing required parameter"
}
```

**Possible causes:**
- Wrong grant type
- Missing client_id or client_secret
- Wrong Content-Type header

**Solution:**
- Verify grant_type is "client_credentials"
- Check all required parameters present
- Use "application/x-www-form-urlencoded" Content-Type

---

## PAT vs OAuth Client Comparison

| Feature | Personal Access Token | OAuth Client |
|---------|----------------------|--------------|
| **Tied to user** | Yes | No |
| **Permissions** | User's permissions | Configured scopes |
| **Best for** | Testing, learning | Production |
| **Expiration** | Set at creation | No expiration |
| **Revocation** | User can delete | Admin can delete |
| **Audit trail** | Shows user's name | Shows client name |
| **When user leaves** | Stops working | Keeps working |

---

## Managing Access Tokens

### **Viewing Your PATs**

1. Go to **Preferences**
2. Select **Personal Access Tokens**
3. See list of your tokens

**What you can see:**
- Token name
- Created date
- Expiration date
- Last used date

**What you CANNOT see:**
- Client Secret (shown only at creation)

### **Revoking a PAT**

**When to revoke:**
- Secret was exposed
- Token no longer needed
- Security concern
- Testing completed

**How to revoke:**
1. Go to Personal Access Tokens
2. Find the token
3. Click **Delete** or **Revoke**
4. Confirm deletion

**Result:** Token immediately stops working

### **Managing OAuth Clients (Admin)**

**Location:** Admin → Security → API Management → OAuth Clients

**Actions:**
- View all clients
- Enable/disable clients
- Update scopes
- Delete clients
- View usage

---

## Rate Limits

ISC APIs have rate limits to ensure fair usage.

### **Understanding Rate Limits**

**Default limits:**
- 100 requests per minute per client
- May vary by tenant tier

**What happens when exceeded:**
- API returns HTTP 429 (Too Many Requests)
- Must wait before making more requests

### **Avoiding Rate Limits**

✅ **Best practices:**
- Batch requests when possible
- Implement retry logic with backoff
- Cache frequently-accessed data
- Don't poll continuously
- Use pagination efficiently

❌ **Don't:**
- Make requests in tight loops
- Retry immediately after failure
- Request same data repeatedly
- Ignore rate limit errors

---

## Practical Examples

### **Example 1: Python Script with PAT**
```python
import requests
import os

# Get credentials from environment
CLIENT_ID = os.getenv('SAILPOINT_CLIENT_ID')
CLIENT_SECRET = os.getenv('SAILPOINT_CLIENT_SECRET')
TENANT = 'yourcompany'

# Get access token
def get_access_token():
    url = f'https://{TENANT}.api.identitynow.com/oauth/token'
    data = {
        'grant_type': 'client_credentials',
        'client_id': CLIENT_ID,
        'client_secret': CLIENT_SECRET
    }
    response = requests.post(url, data=data)
    return response.json()['access_token']

# List identities
def list_identities():
    token = get_access_token()
    url = f'https://{TENANT}.api.identitynow.com/v3/identities'
    headers = {'Authorization': f'Bearer {token}'}
    
    response = requests.get(url, headers=headers, params={'limit': 10})
    return response.json()

# Run
identities = list_identities()
for identity in identities:
    print(f"{identity['name']} - {identity['email']}")
```

### **Example 2: Checking Token Expiration**
```python
import time

class SailPointAPI:
    def __init__(self, client_id, client_secret, tenant):
        self.client_id = client_id
        self.client_secret = client_secret
        self.tenant = tenant
        self.access_token = None
        self.token_expires_at = 0
    
    def get_access_token(self):
        # Check if current token is still valid
        if time.time() < self.token_expires_at - 300:  # 5 min buffer
            return self.access_token
        
        # Get new token
        url = f'https://{self.tenant}.api.identitynow.com/oauth/token'
        data = {
            'grant_type': 'client_credentials',
            'client_id': self.client_id,
            'client_secret': self.client_secret
        }
        response = requests.post(url, data=data)
        token_data = response.json()
        
        self.access_token = token_data['access_token']
        self.token_expires_at = time.time() + token_data['expires_in']
        
        return self.access_token
```

---

## Quick Reference

### **Authentication Flow**
```
1. Create PAT or OAuth Client
2. Save Client ID and Secret
3. Request access token using credentials
4. Use access token in API calls
5. Renew token after 1 hour
```

### **Getting Access Token**
```bash
POST https://{tenant}.api.identitynow.com/oauth/token

Headers:
  Content-Type: application/x-www-form-urlencoded

Body:
  grant_type=client_credentials
  client_id={your_client_id}
  client_secret={your_client_secret}
```

### **Using Access Token**
```bash
GET https://{tenant}.api.identitynow.com/v3/identities

Headers:
  Authorization: Bearer {access_token}
```

---

## Practice Exercises

**Exercise 1:** What are the two main authentication methods in ISC?

<details>
<summary>Answer</summary>

1. Personal Access Token (PAT)
2. OAuth Client Credentials
</details>

---

**Exercise 2:** How long is an access token valid?

<details>
<summary>Answer</summary>

1 hour (3600 seconds)
</details>

---

**Exercise 3:** You're building a production integration. Should you use PAT or OAuth Client?

<details>
<summary>Answer</summary>

OAuth Client - It's not tied to a specific user and is designed for production use
</details>

---

**Exercise 4:** What happens to a PAT when the user who created it leaves the company?

<details>
<summary>Answer</summary>

The PAT stops working because it's tied to that user's account. This is why OAuth Clients are better for production.
</details>

---

**Exercise 5:** Where should you NEVER store your Client Secret?

<details>
<summary>Answer</summary>

- Hardcoded in source code
- Committed to Git/GitHub
- In email
- In log files
- In unencrypted files
</details>

---

# 1.2.2 Understanding API Endpoints and Use Cases

**Learning Objective:** Understand common ISC API endpoints and when to use them.

---

## What are API Endpoints?

An API endpoint is a specific URL that performs a specific function.

**Think of it like:**
- A specific page on a website
- Each endpoint does one thing
- Different endpoints for different actions

**Example endpoints:**
```
/v3/identities          → Work with identities
/v3/accounts            → Work with accounts
/v3/access-profiles     → Work with access profiles
/v3/sources             → Work with sources
```

---

## API Versioning

SailPoint maintains different API versions.

### **Available Versions**

| Version | Status | When to Use |
|---------|--------|-------------|
| **v3** | Current, Stable | All new development |
| **Beta** | Preview features | Testing new features |
| **v2** | Deprecated | Legacy only (migrate to v3) |

**Always use v3 for new work**

### **Version in URL**
```
https://{tenant}.api.identitynow.com/v3/identities
                                      ^^
                                    version
```

---

## Core Endpoint Categories

### **1. Identity Endpoints**

Work with identity (user) data.

**Common endpoints:**
```
GET  /v3/identities           → List identities
GET  /v3/identities/{id}      → Get single identity
PATCH /v3/identities/{id}     → Update identity
```

### **2. Account Endpoints**

Work with accounts in source systems.

**Common endpoints:**
```
GET  /v3/accounts             → List accounts
GET  /v3/accounts/{id}        → Get single account
POST /v3/accounts/{id}/enable → Enable account
POST /v3/accounts/{id}/disable → Disable account
```

### **3. Source Endpoints**

Work with connected source systems.

**Common endpoints:**
```
GET  /v3/sources              → List sources
GET  /v3/sources/{id}         → Get single source
POST /v3/sources/{id}/load-accounts → Trigger aggregation
```

### **4. Access Request Endpoints**

Work with access requests.

**Common endpoints:**
```
POST /v3/access-requests      → Submit access request
GET  /v3/access-requests/{id} → Get request status
```

### **5. Role and Entitlement Endpoints**

Work with access items.

**Common endpoints:**
```
GET  /v3/roles                → List roles
GET  /v3/entitlements         → List entitlements
GET  /v3/access-profiles      → List access profiles
```

---

## Understanding HTTP Methods

APIs use different HTTP methods for different actions.

### **Common HTTP Methods**

| Method | Purpose | Example |
|--------|---------|---------|
| **GET** | Read/retrieve data | Get list of identities |
| **POST** | Create new item | Submit access request |
| **PATCH** | Update existing item | Update identity attribute |
| **PUT** | Replace entire item | Replace configuration |
| **DELETE** | Remove item | Delete access profile |

**Think of it like:**
- GET = Read
- POST = Create
- PATCH = Update
- DELETE = Delete

---

## Working with Identities

### **List All Identities**

**Endpoint:**
```
GET /v3/identities
```

**Query parameters:**
```
?limit=50              → Return 50 results
&offset=0              → Start at first result
&count=true            → Include total count
&filters=department eq "IT"  → Filter results
```

**Complete example:**
```
GET /v3/identities?limit=50&filters=department eq "IT"
```

**Response:**
```json
[
  {
    "id": "2c9180887f4e4b52017f504f30d80234",
    "name": "john.smith",
    "displayName": "John Smith",
    "email": "john.smith@company.com",
    "cloudLifecycleState": "active",
    "attributes": {
      "department": "IT",
      "location": "NYC"
    }
  }
]
```

### **Get Single Identity**

**Endpoint:**
```
GET /v3/identities/{id}
```

**Example:**
```
GET /v3/identities/2c9180887f4e4b52017f504f30d80234
```

**When to use:**
- Get complete details for one identity
- Check current attribute values
- View all accounts for an identity

### **Update Identity Attribute**

**Endpoint:**
```
PATCH /v3/identities/{id}
```

**Request body:**
```json
{
  "op": "replace",
  "path": "/attributes/department",
  "value": "Engineering"
}
```

**What this does:**
Changes the department attribute to "Engineering"

---

## Working with Accounts

### **List Accounts**

**Endpoint:**
```
GET /v3/accounts
```

**Common filters:**
```
?filters=uncorrelated eq true
?filters=sourceId eq "2c91808..."
?filters=disabled eq true
```

**Example: Find uncorrelated accounts**
```
GET /v3/accounts?filters=uncorrelated eq true
```

### **Get Account Details**

**Endpoint:**
```
GET /v3/accounts/{id}
```

**Response includes:**
- Account name
- Source system
- Linked identity
- Entitlements
- Status

### **Enable/Disable Account**

**Enable:**
```
POST /v3/accounts/{id}/enable
```

**Disable:**
```
POST /v3/accounts/{id}/disable
```

---

## Working with Sources

### **List All Sources**

**Endpoint:**
```
GET /v3/sources
```

**Response:**
```json
[
  {
    "id": "2c91808...",
    "name": "Active Directory",
    "type": "Active Directory - Direct",
    "connector": "active-directory",
    "status": "HEALTHY"
  }
]
```

### **Trigger Account Aggregation**

**Endpoint:**
```
POST /v3/sources/{id}/load-accounts
```

**Request body:**
```json
{
  "disableOptimization": false
}
```

**What this does:**
Starts an account aggregation for the source

**When to use:**
- Refresh account data
- After source changes
- Troubleshooting sync issues

---

## Submitting Access Requests

### **Request Access for User**

**Endpoint:**
```
POST /v3/access-requests
```

**Request body:**
```json
{
  "requestedFor": "2c9180887f4e4b52017f504f30d80234",
  "requestType": "GRANT_ACCESS",
  "requestedItems": [
    {
      "type": "ACCESS_PROFILE",
      "id": "2c9180857f4e4b52017f504f30d80567",
      "comment": "Needed for project work"
    }
  ]
}
```

**Explanation:**
- `requestedFor` - Identity ID who gets access
- `requestType` - GRANT_ACCESS or REVOKE_ACCESS
- `requestedItems` - What access to grant
- `type` - ACCESS_PROFILE, ROLE, or ENTITLEMENT
- `comment` - Business justification

**Response:**
```json
{
  "id": "2c91808a7f...",
  "requestId": "REQ-2024-001234",
  "status": "PENDING"
}
```

### **Check Request Status**

**Endpoint:**
```
GET /v3/access-requests/{id}
```

**Response shows:**
- Current status (PENDING, APPROVED, REJECTED, COMPLETED)
- Approval chain
- Completion percentage

---

## Using Search API

The Search API is very powerful for finding data.

### **Universal Search**

**Endpoint:**
```
POST /v3/search
```

**Request body:**
```json
{
  "indices": ["identities"],
  "query": {
    "query": "cloudLifecycleState:active AND department:IT"
  },
  "sort": ["name"]
}
```

**What this does:**
Searches identities using the same query syntax you learned in Section 1.1

**Response:**
```json
[
  {
    "id": "2c9180887...",
    "name": "john.smith",
    "department": "IT",
    "cloudLifecycleState": "active"
  }
]
```

### **Search Across Multiple Types**
```json
{
  "indices": ["identities", "accounts", "entitlements"],
  "query": {
    "query": "name:john*"
  }
}
```

**What this does:**
Searches identities, accounts, AND entitlements for anything matching "john*"

---

## Pagination

Most list endpoints return results in pages.

### **Understanding Pagination Parameters**

| Parameter | Purpose | Default |
|-----------|---------|---------|
| `limit` | Results per page | 250 |
| `offset` | Starting position | 0 |
| `count` | Include total count | false |

### **Getting First Page**
```
GET /v3/identities?limit=100&offset=0
```

Returns: First 100 identities

### **Getting Second Page**
```
GET /v3/identities?limit=100&offset=100
```

Returns: Next 100 identities (101-200)

### **Getting Third Page**
```
GET /v3/identities?limit=100&offset=200
```

Returns: Next 100 identities (201-300)

### **Pattern for All Pages**
```
Page 1: offset=0
Page 2: offset=100
Page 3: offset=200
Page 4: offset=300
...
```

**Formula:** offset = (page_number - 1) × limit

---

## Common Use Cases

### **Use Case 1: Find Uncorrelated Accounts**

**Goal:** Get all orphaned accounts for cleanup

**API Call:**
```
GET /v3/accounts?filters=uncorrelated eq true&limit=250
```

**What you get:**
- List of accounts not linked to identities
- Can export for review
- Can process for correlation

### **Use Case 2: Onboard New Hire**

**Goal:** Request standard access for new employee

**Steps:**

1. Find identity ID:
```
GET /v3/identities?filters=email eq "newhire@company.com"
```

2. Submit access requests:
```
POST /v3/access-requests
{
  "requestedFor": "{identity_id}",
  "requestType": "GRANT_ACCESS",
  "requestedItems": [
    {"type": "ACCESS_PROFILE", "id": "{email_access_id}"},
    {"type": "ACCESS_PROFILE", "id": "{vpn_access_id}"}
  ]
}
```

### **Use Case 3: Generate Department Report**

**Goal:** List all active IT employees

**API Call:**
```
POST /v3/search
{
  "indices": ["identities"],
  "query": {
    "query": "cloudLifecycleState:active AND department:IT"
  },
  "sort": ["name"]
}
```

**What you get:**
- All active IT identities
- Sorted by name
- Can export to CSV via code

### **Use Case 4: Trigger Source Refresh**

**Goal:** Refresh Active Directory accounts

**Steps:**

1. Find source ID:
```
GET /v3/sources?filters=name eq "Active Directory"
```

2. Trigger aggregation:
```
POST /v3/sources/{source_id}/load-accounts
{
  "disableOptimization": false
}
```

---

## Working with JSON

All API responses use JSON format.

### **What is JSON?**

**JSON** = JavaScript Object Notation
- Standard format for API data
- Human-readable
- Easy for programs to parse

### **JSON Structure**

**Object (key-value pairs):**
```json
{
  "name": "John Smith",
  "email": "john@company.com",
  "department": "IT"
}
```

**Array (list of items):**
```json
[
  {"name": "John", "dept": "IT"},
  {"name": "Jane", "dept": "HR"}
]
```

**Nested objects:**
```json
{
  "name": "John Smith",
  "attributes": {
    "department": "IT",
    "location": "NYC"
  }
}
```

### **Reading JSON Responses**

**Example response:**
```json
{
  "id": "2c9180887...",
  "name": "john.smith",
  "email": "john@company.com",
  "attributes": {
    "department": "IT"
  }
}
```

**How to read:**
- Top-level: `id`, `name`, `email`
- Nested: `attributes.department` = "IT"

---

## Error Handling

APIs return error codes when something goes wrong.

### **Common HTTP Status Codes**

| Code | Meaning | Cause |
|------|---------|-------|
| **200** | OK | Success |
| **201** | Created | Successfully created |
| **400** | Bad Request | Invalid input |
| **401** | Unauthorized | Bad credentials |
| **403** | Forbidden | No permission |
| **404** | Not Found | Resource doesn't exist |
| **429** | Too Many Requests | Rate limit exceeded |
| **500** | Server Error | ISC internal error |

### **Reading Error Responses**

**Example error:**
```json
{
  "error": "invalid_request",
  "error_description": "Missing required field: requestedFor"
}
```

**What to check:**
- `error` - Error type
- `error_description` - What went wrong

---

## Best Practices

### **API Usage Best Practices**

✅ **Do:**
- Use v3 endpoints (not v2)
- Implement pagination for large datasets
- Cache frequently-accessed data
- Handle errors gracefully
- Use appropriate HTTP methods
- Include meaningful comments in requests
- Log API calls for debugging
- Respect rate limits

❌ **Don't:**
- Make unnecessary API calls
- Poll continuously (use webhooks instead)
- Ignore error responses
- Request all data when you need specific items
- Use deprecated endpoints
- Exceed rate limits

### **Performance Tips**

**Use filters to reduce data:**
```
❌ Bad: GET /v3/identities
(Returns thousands of identities)

✅ Good: GET /v3/identities?filters=department eq "IT"
(Returns only what you need)
```

**Use pagination:**
```
❌ Bad: GET /v3/identities?limit=10000
(May timeout or fail)

✅ Good: GET /v3/identities?limit=250&offset=0
(Manageable chunks)
```

**Cache static data:**
```
✅ Cache source lists (don't request repeatedly)
✅ Cache access profile IDs
✅ Store tenant configuration
```

---

## API Documentation Resources

### **Official Documentation**

**Main API Docs:**
- https://developer.sailpoint.com/docs/api/

**API Reference:**
- https://developer.sailpoint.com/idn/api/

**What you'll find:**
- Complete endpoint list
- Request/response examples
- Authentication guides
- Best practices

### **Interactive API Explorer**

SailPoint provides an interactive API explorer where you can:
- Try API calls directly
- See real responses
- Test authentication
- Learn by doing

---

## Quick Reference

### **Common Endpoints Quick Reference**
```
Identities:
  GET  /v3/identities
  GET  /v3/identities/{id}
  PATCH /v3/identities/{id}

Accounts:
  GET  /v3/accounts
  GET  /v3/accounts/{id}
  POST /v3/accounts/{id}/enable

Sources:
  GET  /v3/sources
  POST /v3/sources/{id}/load-accounts

Access Requests:
  POST /v3/access-requests
  GET  /v3/access-requests/{id}

Search:
  POST /v3/search
```

### **HTTP Method Summary**
```
GET    = Read/Retrieve
POST   = Create/Execute
PATCH  = Update partially
PUT    = Replace completely
DELETE = Remove
```

### **Response Codes**
```
200 = Success
201 = Created
400 = Bad request
401 = Not authenticated
403 = No permission
404 = Not found
429 = Rate limited
500 = Server error
```

---

## Practice Exercises

**Exercise 6:** What API endpoint would you use to list all sources?

<details>
<summary>Answer</summary>
```
GET /v3/sources
```
</details>

---

**Exercise 7:** How do you trigger an account aggregation for a source?

<details>
<summary>Answer</summary>
```
POST /v3/sources/{source-id}/load-accounts

Request body:
{
  "disableOptimization": false
}
```
</details>

---

**Exercise 8:** What HTTP method do you use to update an identity's attributes?

<details>
<summary>Answer</summary>

PATCH

Example:
```
PATCH /v3/identities/{id}
```
</details>

---

**Exercise 9:** You want to get identities 101-200 from your tenant. What parameters do you use?

<details>
<summary>Answer</summary>
```
GET /v3/identities?limit=100&offset=100
```

- limit=100 (get 100 results)
- offset=100 (skip first 100, start at 101)
</details>

---

**Exercise 10:** What does HTTP 429 error mean?

<details>
<summary>Answer</summary>

Too Many Requests - You've exceeded the rate limit.

Solution: Wait before making more requests, implement retry logic with backoff.
</details>

---

## Section Summary

You've learned:

✅ **Authentication** - PAT and OAuth Client methods  
✅ **Access Tokens** - How to get and use them  
✅ **API Endpoints** - Common endpoints and purposes  
✅ **HTTP Methods** - GET, POST, PATCH, DELETE  
✅ **Pagination** - Working with large datasets  
✅ **Error Handling** - Understanding response codes  
✅ **Best Practices** - Security and performance  

**Next Steps:**
- Create a test PAT
- Make your first API call
- Explore the API documentation
- Build a simple integration

---

## Additional Resources

📚 **Official Resources:**
- [SailPoint API Documentation](https://developer.sailpoint.com/docs/api/)
- [Authentication Guide](https://developer.sailpoint.com/idn/docs/authentication)
- [API Reference](https://developer.sailpoint.com/idn/api/)

💻 **Developer Tools:**
- Postman Collections for SailPoint
- API Explorer
- Code samples on GitHub

🎯 **Practice:**
- Use Postman to test API calls
- Write simple scripts
- Try different endpoints
- Experiment with filters

---

**End of Section 1.2: API and Integration**

**Next Section:** [1.3 Monitoring and Activity Tracking →](#13-monitoring-and-activity-tracking)
