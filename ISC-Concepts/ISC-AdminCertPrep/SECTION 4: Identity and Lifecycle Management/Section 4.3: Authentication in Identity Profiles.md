# Section 4.3: Authentication in Identity Profiles

**Master authentication configuration for identity profiles in ISC**

---

## Table of Contents

- [4.3.1 Authentication Options Available](#431-authentication-options-available)
- [4.3.2 Configuring Identity Profile Authentication](#432-configuring-identity-profile-authentication)

---

# 4.3.1 Authentication Options Available

**Learning Objective:** Understand authentication methods available for identity profiles and when to use each.

---

## What is Identity Profile Authentication?

**Definition:** Configuration that determines how users authenticate to ISC and related systems

**Purpose:**
```
Identity Profile Authentication controls:
  - How users log into ISC
  - What authentication method they use
  - Password management (if applicable)
  - SSO integration
```

**Key concept:**
```
Identity Profile → Defines authentication method
Users in that profile → Use that authentication method
```

**Example:**
```
Employee Identity Profile:
  Authentication: SAML SSO (Azure AD)
  
Result:
  - Employees log in via Azure AD
  - Single sign-on experience
  - No separate ISC password
```

**Exam Tip:** Identity profile authentication determines HOW users in that profile authenticate to ISC.

---

## Authentication Methods Overview

### **Available Options**

| Method | Description | Use Case | Complexity |
|--------|-------------|----------|------------|
| **Internal** | ISC-managed passwords | Dev/test only | Simple |
| **SAML** | SSO via external IdP | Production standard | Medium |
| **OAuth/OIDC** | Modern SSO protocol | Cloud-native apps | Medium |
| **None** | No direct login | Service identities | N/A |

---

## Authentication Method 1: Internal Authentication

### **What is Internal Authentication?**

**Definition:** ISC manages and stores user passwords directly

**How it works:**
```
1. User created in ISC
2. ISC generates/sets password
3. User logs in with ISC username/password
4. ISC validates credentials
5. User authenticated
```

**Characteristics:**
```
✓ Simple setup (no external dependencies)
✓ ISC manages passwords
✓ Self-service password reset available
✓ Password policies enforced by ISC

✗ Separate password from corporate credentials
✗ No SSO experience
✗ Higher support burden
✗ Security concerns (password sprawl)
```

### **When to Use Internal Authentication**

**Acceptable:**
```
✓ Development/test environments
✓ Proof of concept
✓ Sandbox tenants
✓ Very small deployments (<10 users)
```

**NOT recommended:**
```
✗ Production environments
✗ Enterprise deployments
✗ When SSO is available
✗ Compliance-sensitive environments
```

**Why avoid in production:**
```
Issues:
  - Users have another password to remember
  - Password resets increase help desk calls
  - No integration with corporate identity
  - Doesn't leverage existing SSO
  - Security risk (password sprawl)
  - Poor user experience
```

### **Internal Authentication Configuration**

**Settings available:**
```
Password Policy:
  - Minimum length (default: 12 characters)
  - Complexity requirements
  - Password expiration (default: 90 days)
  - Password history (prevent reuse)
  
Account Lockout:
  - Failed attempts threshold (default: 5)
  - Lockout duration (default: 30 minutes)
  - Automatic unlock
  
Self-Service:
  - Password reset via email
  - Security questions
  - Email verification
```

**Example configuration:**
```
Password Requirements:
  Minimum Length: 14 characters
  Must Contain:
    ✓ Uppercase letter
    ✓ Lowercase letter
    ✓ Number
    ✓ Special character
  Expires: 90 days
  History: Cannot reuse last 10 passwords
  
Account Lockout:
  Failed Attempts: 5
  Lockout Duration: 30 minutes
  Auto-unlock: Yes
```

**Exam Tip:** Internal authentication = ISC manages passwords. Avoid in production; use SAML instead.

---

## Authentication Method 2: SAML SSO

### **What is SAML SSO?**

**Definition:** Security Assertion Markup Language - industry standard for single sign-on

**How it works:**
```
1. User accesses ISC URL
2. ISC redirects to external IdP (Azure AD, Okta, etc.)
3. User authenticates with corporate credentials
4. IdP sends SAML assertion to ISC
5. ISC validates assertion
6. User authenticated and logged in
```

**Visual flow:**
```
User → ISC → Redirect to IdP → Authenticate → SAML Assertion → ISC → Access Granted
```

**Characteristics:**
```
✓ Single sign-on (use corporate credentials)
✓ No separate ISC password
✓ Centralized authentication
✓ MFA enforced at IdP
✓ Better security
✓ Better user experience
✓ Industry standard

✗ Requires external IdP
✗ More complex setup
✗ Dependencies on IdP availability
```

### **SAML Components**

**Service Provider (SP):** ISC
```
ISC acts as Service Provider:
  - Receives SAML assertions
  - Validates assertions
  - Grants access based on assertion
```

**Identity Provider (IdP):** Azure AD, Okta, Ping, etc.
```
IdP authenticates users:
  - Validates credentials
  - Enforces MFA
  - Sends SAML assertion to ISC
```

**SAML Assertion:**
```
XML document containing:
  - User identity (email, username)
  - Authentication status
  - Attributes (name, department, etc.)
  - Digital signature (validation)
  - Expiration timestamp
```

**Example SAML assertion (simplified):**
```xml
<saml:Assertion>
  <saml:Subject>
    <saml:NameID>john.smith@company.com</saml:NameID>
  </saml:Subject>
  <saml:AttributeStatement>
    <saml:Attribute Name="email">
      <saml:AttributeValue>john.smith@company.com</saml:AttributeValue>
    </saml:Attribute>
    <saml:Attribute Name="firstName">
      <saml:AttributeValue>John</saml:AttributeValue>
    </saml:Attribute>
    <saml:Attribute Name="lastName">
      <saml:AttributeValue>Smith</saml:AttributeValue>
    </saml:Attribute>
  </saml:AttributeStatement>
</saml:Assertion>
```

### **Common SAML Identity Providers**

**Enterprise IdPs:**
```
Azure Active Directory (Microsoft)
  - Most common for Microsoft 365 customers
  - Integrated with Azure AD
  - Native MFA support

Okta
  - Dedicated identity platform
  - Strong SSO capabilities
  - Wide app integration

Ping Identity (PingFederate, PingOne)
  - Enterprise-grade
  - Advanced features
  - Common in large enterprises

Google Workspace
  - For Google-centric organizations
  - Integrated with Google accounts

OneLogin
  - Cloud-based IdP
  - SMB and enterprise
```

### **SAML Attributes**

**Attributes passed from IdP to ISC:**
```
Required:
  - NameID or email (unique identifier)
  
Common:
  - email
  - firstName (givenName)
  - lastName (sn or surname)
  - displayName
  
Optional:
  - department
  - title
  - manager
  - phone
```

**Attribute mapping:**
```
IdP Attribute → ISC Attribute

email → email (correlation attribute)
givenName → firstName
sn → lastName
displayName → displayName

ISC uses these to:
  - Correlate user to identity
  - Update identity attributes (optional)
```

### **When to Use SAML**

**Use SAML when:**
```
✓ Production environment
✓ Enterprise deployment
✓ SSO already in place
✓ Want centralized authentication
✓ Need MFA (enforced at IdP)
✓ Compliance requirements
```

**SAML is production standard:**
```
Recommended setup:
  - All production ISC tenants
  - Use corporate IdP (Azure AD, Okta)
  - Enforce MFA at IdP
  - No internal authentication
```

**Exam Tip:** SAML is the recommended authentication method for production. Uses external IdP for SSO.

---

## Authentication Method 3: OAuth/OIDC

### **What is OAuth/OIDC?**

**OAuth 2.0:** Authorization framework
**OIDC (OpenID Connect):** Authentication layer on top of OAuth

**Primary use in ISC:**
```
Not typically for end-user login to ISC
Mainly for:
  - API authentication
  - Application integration
  - Service accounts
```

**How it differs from SAML:**
```
SAML:
  - XML-based
  - Primarily for web SSO
  - SAML assertions

OAuth/OIDC:
  - JSON-based
  - RESTful
  - Access tokens / ID tokens
  - More modern protocol
```

**When OAuth/OIDC used in ISC:**
```
API Access:
  - OAuth clients for API calls
  - Personal Access Tokens (PAT)
  - Service account authentication

Application Integration:
  - Modern cloud apps
  - SaaS connectors
  - API-based integrations
```

**Example - API Authentication:**
```
1. Application requests token from ISC
2. ISC validates OAuth client credentials
3. ISC issues access token
4. Application uses token to call ISC API
5. ISC validates token on each request
```

**Exam Tip:** OAuth/OIDC in ISC primarily for API authentication, not end-user SSO to ISC UI. SAML is for user SSO.

---

## Authentication Method 4: No Authentication (Service Identities)

### **What are Service Identities?**

**Definition:** Identities that don't log into ISC directly

**Characteristics:**
```
- No password
- No login capability
- Used for tracking only
- Example: Service accounts, shared accounts
```

**Use cases:**
```
Service Accounts:
  - svc_backup
  - app_webserver
  - Service doesn't need to log into ISC
  
Shared Accounts:
  - admin_shared
  - reception_desk
  - Multiple people use, not individual login
  
Terminated Employees:
  - Historical tracking
  - No login needed
  - Preserve for audit
```

**Configuration:**
```
Authentication: None
Result: Identity exists but cannot authenticate to ISC
```

---

## Choosing Authentication Method

### **Decision Matrix**

| Environment | Recommended Method | Reasoning |
|-------------|-------------------|-----------|
| **Production** | SAML SSO | Best security, user experience |
| **Development** | Internal or SAML | Internal acceptable for dev |
| **Sandbox** | Internal or SAML | Internal acceptable for testing |
| **POC/Demo** | Internal | Quick setup, no dependencies |

### **Best Practices**

**For Production:**
```
✓ Use SAML SSO
✓ Integrate with corporate IdP (Azure AD, Okta)
✓ Enforce MFA at IdP level
✓ Disable internal authentication
✓ Regular certificate renewal
```

**For Development/Test:**
```
✓ SAML preferred (mirrors production)
✓ Internal acceptable if SAML not available
✓ Document differences from production
```

**Security recommendations:**
```
Production:
  ✓ SAML SSO mandatory
  ✓ MFA required for all users
  ✓ Short session timeout (30 min idle)
  ✓ IP restrictions (if needed)
  
Development:
  ✓ SAML preferred
  ✓ Internal acceptable
  ✓ Different IdP/tenant than production
```

---

## Practice Exercise

**Question:** Your company uses Azure AD for authentication. You're setting up ISC for 500 employees. What authentication method should you configure and why?

<details>
<summary>Answer</summary>

**Recommended: SAML SSO with Azure AD**

**Configuration:**
```
Identity Profile: Employee Profile
Authentication Method: SAML
Identity Provider: Azure Active Directory
```

**Why SAML with Azure AD:**

**Reason 1: Single Sign-On**
```
With SAML:
  ✓ Employees use existing Azure AD credentials
  ✓ No new password to remember
  ✓ Same login as Office 365, Teams, etc.
  ✓ Seamless user experience

Without SAML (Internal):
  ✗ Separate ISC password
  ✗ More passwords to manage
  ✗ Higher help desk calls
  ✗ Poor user experience
```

**Reason 2: Centralized Authentication**
```
With SAML:
  ✓ All authentication through Azure AD
  ✓ Centralized user management
  ✓ Consistent policies
  ✓ Easier to manage 500 users

Without SAML:
  ✗ Manage users in two places (Azure AD + ISC)
  ✗ Duplicate effort
  ✗ Potential for inconsistencies
```

**Reason 3: Multi-Factor Authentication**
```
With SAML:
  ✓ MFA enforced at Azure AD
  ✓ Users already enrolled in Azure MFA
  ✓ Consistent MFA experience
  ✓ Better security

Without SAML:
  ✗ Would need separate MFA for ISC
  ✗ Users enroll in two MFA systems
  ✗ More complex
```

**Reason 4: Security**
```
With SAML:
  ✓ No passwords stored in ISC
  ✓ Leverages Azure AD security
  ✓ Azure AD handles password policies
  ✓ Centralized security logging
  ✓ Conditional access policies apply

Without SAML:
  ✗ Passwords stored in ISC
  ✗ Separate password policies
  ✗ Potential security gaps
```

**Reason 5: Compliance**
```
With SAML:
  ✓ Audit trail in Azure AD
  ✓ Centralized authentication logging
  ✓ Easier compliance reporting
  ✓ Consistent with corporate standards

Without SAML:
  ✗ Split audit trails
  ✗ More complex compliance
```

**Reason 6: User Lifecycle**
```
With SAML:
  ✓ User disabled in Azure AD → Cannot access ISC
  ✓ Automatic termination
  ✓ No orphaned access

Without SAML:
  ✗ Would need to disable in both systems
  ✗ Risk of orphaned ISC accounts
```

**Implementation Overview:**

**Step 1: Configure Azure AD**
```
1. Create Enterprise Application in Azure AD
2. Configure SAML SSO
3. Set ISC URLs:
   - Entity ID: https://yourcompany.identitynow.com
   - ACS URL: https://yourcompany.identitynow.com/saml/SSO
4. Configure attribute mappings:
   - email → user.mail
   - firstName → user.givenname
   - lastName → user.surname
5. Assign users/groups
```

**Step 2: Configure ISC**
```
1. Upload Azure AD metadata
2. Configure SAML settings
3. Map SAML attributes
4. Test with pilot user
5. Enable for all users
```

**Step 3: Deployment**
```
1. Pilot with 10-20 users
2. Verify SSO working
3. Gather feedback
4. Roll out to all 500 employees
5. Disable internal authentication (optional)
6. Monitor and support
```

**Expected User Experience:**
```
With SAML configured:

User navigates to: https://yourcompany.identitynow.com

1. ISC detects SAML configured
2. Redirects to Azure AD login
3. User enters Azure AD credentials (john.smith@company.com)
4. Azure AD prompts for MFA (if configured)
5. User completes MFA
6. Azure AD sends SAML assertion to ISC
7. ISC validates assertion
8. User logged into ISC

Total time: 10-15 seconds (if already logged into Azure AD, instant)
```

**Alternative Scenario - Why NOT Internal:**
```
If using Internal Authentication for 500 users:

Problems:
  ✗ 500 passwords to manage in ISC
  ✗ Password resets (estimated 50-100/month)
  ✗ Users forget separate ISC password
  ✗ No SSO (poor experience)
  ✗ Security risk (password sprawl)
  ✗ Higher support costs
  ✗ Not scalable

Cost impact:
  - Help desk time: 5-10 hrs/month (password resets)
  - User frustration
  - Security risk
  
SAML eliminates all these issues
```

**Summary:**
```
For 500 employees with Azure AD:

Recommended: SAML SSO
  ✓ Single sign-on experience
  ✓ No separate passwords
  ✓ MFA from Azure AD
  ✓ Better security
  ✓ Lower support costs
  ✓ Easier management
  ✓ Production best practice

NOT Recommended: Internal Authentication
  ✗ Separate passwords
  ✗ Higher support burden
  ✗ Poor user experience
  ✗ Not scalable
  ✗ Security concerns
```

**Exam Tip:** For production with existing IdP (Azure AD, Okta), always use SAML SSO, not internal authentication.
</details>

---

# 4.3.2 Configuring Identity Profile Authentication

**Learning Objective:** Learn how to configure authentication for identity profiles in ISC.

---

## SAML Configuration Overview

### **Configuration Components**

**What you need:**
```
From ISC:
  - Entity ID (ISC identifier)
  - ACS URL (Assertion Consumer Service URL)
  - Metadata URL (optional)

From IdP (Azure AD, Okta, etc.):
  - SSO URL (IdP login endpoint)
  - Issuer/Entity ID (IdP identifier)
  - Certificate (for signature validation)
  - Metadata XML (contains all above)

Configuration Steps:
  1. Configure ISC as app in IdP
  2. Get IdP metadata
  3. Configure SAML in ISC
  4. Map attributes
  5. Test
  6. Enable for users
```

---

## Step-by-Step: SAML with Azure AD

### **Phase 1: Gather ISC Information**

**Step 1: Get ISC URLs**
```
In ISC:
  Admin → Settings → Authentication

ISC provides:
  Entity ID: https://yourcompany.identitynow.com
  ACS URL: https://yourcompany.identitynow.com/saml/SSO
  Metadata URL: https://yourcompany.identitynow.com/saml/metadata
```

**These URLs needed for Azure AD configuration**

---

### **Phase 2: Configure Azure AD**

**Step 2: Create Enterprise Application**
```
Azure Portal:
  1. Navigate to Azure Active Directory
  2. Enterprise Applications
  3. New Application
  4. Create your own application
  5. Name: "SailPoint IdentityNow"
  6. Select: Integrate any other application you don't find in the gallery
  7. Create
```

**Step 3: Configure SAML in Azure AD**
```
1. Click "Set up single sign on"
2. Select "SAML"
3. Edit "Basic SAML Configuration":

   Identifier (Entity ID):
     https://yourcompany.identitynow.com
     
   Reply URL (ACS URL):
     https://yourcompany.identitynow.com/saml/SSO
     
   Sign on URL (optional):
     https://yourcompany.identitynow.com
     
4. Save
```

**Step 4: Configure Attributes in Azure AD**
```
Edit "Attributes & Claims":

Required claim:
  Name ID:
    Source attribute: user.mail
    Name ID format: Email address
    
Additional claims:
  email:
    Source attribute: user.mail
    
  firstName:
    Source attribute: user.givenname
    
  lastName:
    Source attribute: user.surname
    
  displayName:
    Source attribute: user.displayname

Save
```

**Step 5: Download Certificate and URLs**
```
In "SAML Signing Certificate" section:
  Download: Certificate (Base64)
  
In "Set up [Application]" section:
  Copy:
    - Login URL (SSO URL)
    - Azure AD Identifier (Issuer)
    - Logout URL (optional)
    
Or:
  Download: Federation Metadata XML
    (Contains all information in one file)
```

**Step 6: Assign Users in Azure AD**
```
1. Go to "Users and groups"
2. Add user/group
3. Select users or groups to grant access
4. Assign

Example:
  Assign group: "ISC-Users"
  Result: All members can access ISC via SAML
```

---

### **Phase 3: Configure ISC**

**Step 7: Configure SAML in ISC**
```
ISC:
  Admin → Settings → Authentication

Option A: Upload Metadata File
  1. Click "Configure SAML"
  2. Upload Federation Metadata XML (from Azure AD)
  3. ISC auto-populates all fields
  
Option B: Manual Configuration
  1. Click "Configure SAML"
  2. Enter manually:
     - SSO URL: (Login URL from Azure AD)
     - Issuer: (Azure AD Identifier)
     - Certificate: (Paste Base64 certificate)
  3. Save
```

**Step 8: Configure Attribute Mapping in ISC**
```
Map SAML attributes to ISC identity attributes:

SAML Attribute → ISC Attribute

NameID or email → email (for correlation)
firstName → firstName
lastName → lastName
displayName → displayName

Configuration:
  - Primary correlation: email
  - Used to match SAML user to ISC identity
```

**Step 9: Configure Identity Profile**
```
Admin → Identities → Identity Profiles → Employee Profile

Authentication Section:
  Method: SAML
  Enable SAML Authentication: Yes
  Correlation Attribute: email
  
Save
```

---

### **Phase 4: Testing**

**Step 10: Test with Pilot User**
```
1. Create test user in Azure AD
2. Assign to ISC application
3. Ensure test user has identity in ISC
4. Test login:
   
   Test Process:
   a. Navigate to: https://yourcompany.identitynow.com
   b. Should redirect to Azure AD login
   c. Enter test user credentials
   d. Complete MFA (if enabled)
   e. Should redirect back to ISC
   f. Verify logged in successfully
   
5. Verify identity correlation:
   - Check which ISC identity logged in
   - Should match based on email
   
6. Test logout
```

**Common test scenarios:**
```
Test Case 1: Successful login
  - User exists in both Azure AD and ISC
  - Email matches
  - Expected: Successful login

Test Case 2: User in Azure AD but not ISC
  - User assigned to app in Azure AD
  - No identity in ISC
  - Expected: Login fails (no identity to correlate)
  - Solution: Ensure identity created via identity profile

Test Case 3: Email mismatch
  - Azure AD email: john.smith@company.com
  - ISC email: jsmith@company.com
  - Expected: Correlation fails
  - Solution: Fix email in ISC or Azure AD

Test Case 4: Certificate validation
  - Verify SAML assertion signature validated
  - Check logs for certificate errors
  - Expected: No certificate errors
```

---

### **Phase 5: Deployment**

**Step 11: Pilot Rollout**
```
1. Select 10-20 pilot users
2. Assign to ISC app in Azure AD
3. Communicate change:
   - New login URL
   - SSO via Azure AD
   - MFA requirements
   
4. Monitor for issues:
   - Login failures
   - Correlation problems
   - Attribute mapping issues
   
5. Gather feedback
6. Resolve issues
```

**Step 12: Full Rollout**
```
1. Assign all users/groups in Azure AD
   Example: Assign "All-Employees" group
   
2. Communicate to all users:
   - Email announcement
   - Training/documentation
   - Support contact
   
3. Monitor:
   - Login success rate
   - Help desk tickets
   - Error logs
   
4. Support users through transition
```

**Step 13: Disable Internal Authentication (Optional)**
```
After successful SAML rollout:

1. Verify all users using SAML
2. Disable internal authentication:
   Admin → Settings → Authentication
   Disable: Internal Authentication
   
3. Result:
   - Only SAML login available
   - No password-based login
   - Forces SSO
   
Recommendation:
  - Wait 30 days after SAML rollout
  - Ensure no issues
  - Then disable internal
```

---

## SAML Troubleshooting

### **Issue 1: Redirect Loop**

**Symptom:**
```
User accesses ISC → Redirects to Azure AD → Redirects back to ISC → Redirects to Azure AD (loop)
```

**Common causes:**
```
1. Incorrect ACS URL in Azure AD
2. Entity ID mismatch
3. Certificate issue
```

**Solution:**
```
1. Verify URLs match exactly:
   Azure AD ACS URL = ISC ACS URL
   
2. Check Entity ID:
   Azure AD Identifier = ISC Entity ID
   
3. Check certificate:
   - Valid (not expired)
   - Uploaded correctly to ISC
```

---

### **Issue 2: Login Successful but No Access**

**Symptom:**
```
User logs in via Azure AD
SAML assertion received
User sees "Access Denied" or similar
```

**Common causes:**
```
1. User not correlated to ISC identity
2. Email mismatch
3. Identity disabled in ISC
```

**Solution:**
```
1. Check correlation:
   - SAML NameID/email attribute
   - ISC identity email
   - Do they match?
   
2. Verify identity exists:
   - Search for user in ISC
   - Check identity status (active?)
   
3. Check SAML attribute mapping:
   - Is email attribute mapped correctly?
   - Check SAML assertion in browser tools
```

---

### **Issue 3: Attribute Not Updating**

**Symptom:**
```
User's name changed in Azure AD
SAML login successful
Name not updated in ISC
```

**Common causes:**
```
1. Attribute not sent in SAML assertion
2. Attribute not mapped in ISC
3. Identity profile manages attribute (higher priority)
```

**Solution:**
```
1. Check SAML assertion:
   - Use browser developer tools
   - Verify attribute present in assertion
   
2. Check ISC attribute mapping:
   - Is SAML attribute mapped?
   - Correct ISC attribute name?
   
3. Check identity profile:
   - Does profile set this attribute?
   - Profile may override SAML data
```

---

### **Issue 4: Certificate Expired**

**Symptom:**
```
SAML login fails
Error: Certificate validation failed
```

**Solution:**
```
1. Check certificate expiration:
   - In Azure AD
   - In ISC
   
2. Generate new certificate in Azure AD
3. Download new certificate
4. Upload to ISC
5. Test login
6. Remove old certificate after validation

Prevention:
  - Set calendar reminder 30 days before expiration
  - Monitor certificate expiration
  - Renew proactively
```

---

## SAML Best Practices

### **Security**
```
✓ Use HTTPS only (no HTTP)
✓ Validate SAML signatures (certificate)
✓ Short session timeout (30 min idle)
✓ Enforce MFA at IdP
✓ Monitor for suspicious logins
✓ Regular certificate renewal
✓ Use latest TLS version
```

### **Configuration**
```
✓ Document all URLs and settings
✓ Test in sandbox before production
✓ Use metadata file (easier to manage)
✓ Map all necessary attributes
✓ Plan for certificate expiration
✓ Have rollback plan
```

### **User Management**
```
✓ Assign users via groups (not individually)
✓ Automate user assignment (if possible)
✓ Regular access reviews
✓ Remove access for termed users
✓ Communicate changes to users
```

---

## Internal Authentication Configuration

### **When to Configure Internal Auth**

**Acceptable scenarios:**
```
- Development/test environments
- Proof of concept
- Very small deployments (<10 users)
- Temporary/emergency access
```

### **Configuration Steps**

**Step 1: Enable Internal Authentication**
```
Admin → Settings → Authentication

Enable: Internal Authentication
```

**Step 2: Configure Password Policy**
```
Password Requirements:
  Minimum Length: 14 characters
  Complexity:
    ✓ Uppercase letter required
    ✓ Lowercase letter required
    ✓ Number required
    ✓ Special character required
  Expiration: 90 days
  History: 10 previous passwords
  
Account Lockout:
  Failed Attempts: 5
  Lockout Duration: 30 minutes
  Auto-unlock: Yes
```

**Step 3: Configure Identity Profile**
```
Admin → Identities → Identity Profiles → [Profile]

Authentication:
  Method: Internal
  Enable: Yes
  
Password Management:
  Allow self-service reset: Yes
  Security questions: Optional
  Email verification: Required
```

**Step 4: User Setup**
```
Options for initial password:

Option A: Auto-generate
  - ISC generates random password
  - Sends to user via email
  - User must change on first login
  
Option B: Admin sets
  - Admin sets initial password
  - Communicates securely to user
  - Force change on first login
  
Option C: User self-registration
  - If enabled, user can register
  - Sets own initial password
```

---

## Practice Exercise

**Question:** You're configuring SAML for ISC with Azure AD. During testing, user john.smith@company.com logs in via Azure AD successfully, but gets "Access Denied" in ISC. What would you check?

<details>
<summary>Answer</summary>

**Systematic Troubleshooting:**

**Issue:** User authenticates via SAML but cannot access ISC

**Step 1: Verify Identity Exists in ISC**
```
Check:
  1. Navigate to: Admin → Identities
  2. Search for: john.smith@company.com
  3. Verify identity exists
  
Possible outcomes:

Outcome A: Identity NOT found
  Problem: No identity in ISC to correlate SAML user
  Solution:
    - Identity should exist from identity profile
    - Check if identity profile created identity
    - Verify Workday/HR has this user
    - Trigger identity profile refresh if needed
    - Or manually create identity (temporary)
    
Outcome B: Identity found
  Proceed to Step 2
```

**Step 2: Check Identity Status**
```
If identity found:
  
Check identity details:
  1. Click on identity: John Smith
  2. Verify:
     - Status: Active (not disabled)
     - Lifecycle: active (not inactive)
     - Email: john.smith@company.com
  
Possible issues:

Issue A: Identity disabled
  Status shows: Disabled
  Solution: Enable the identity
  
Issue B: Lifecycle = inactive
  Lifecycle: inactive (terminated employee?)
  Solution:
    - Verify employment status
    - Update lifecycle to active if should have access
    - Check identity profile lifecycle mapping
```

**Step 3: Verify Email Correlation**
```
Check if emails match exactly:

SAML provides:
  NameID/email: john.smith@company.com
  
ISC identity has:
  Email: ??? (check this)
  
Possible mismatches:

Mismatch Example 1:
  SAML: john.smith@company.com
  ISC: jsmith@company.com
  Problem: Emails don't match - correlation fails
  
Mismatch Example 2:
  SAML: john.smith@company.com
  ISC: John.Smith@company.com (different case)
  Problem: Case sensitivity (usually OK but check)
  
Mismatch Example 3:
  SAML: john.smith@company.com
  ISC: (email field is empty/null)
  Problem: No email to correlate
  
Solution for mismatches:
  Option A: Fix email in ISC identity
    - Update to match SAML email
    - Retry login
    
  Option B: Fix email in Azure AD
    - Update user.mail attribute
    - Retry login
    
  Option C: Change correlation attribute
    - Use different attribute (employeeId)
    - Requires both SAML and ISC to have it
```

**Step 4: Check SAML Attribute Mapping**
```
Verify SAML assertion contains email:

1. Open browser developer tools (F12)
2. Network tab
3. Attempt login
4. Find SAML POST to ISC
5. View SAML assertion (base64 decode)
6. Check for email/NameID attribute

Expected in SAML assertion:
  <saml:NameID>john.smith@company.com</saml:NameID>
  OR
  <saml:Attribute Name="email">
    <saml:AttributeValue>john.smith@company.com</saml:AttributeValue>
  </saml:Attribute>

If email NOT in assertion:
  Problem: Azure AD not sending email attribute
  Solution:
    - Go to Azure AD → Enterprise App → SAML config
    - Edit "Attributes & Claims"
    - Verify email claim exists
    - Verify mapped to user.mail
    - Save and test again
```

**Step 5: Check ISC SAML Configuration**
```
Verify ISC correlation settings:

Admin → Identities → Identity Profiles → Employee Profile

Authentication:
  Method: SAML ✓
  Correlation Attribute: email
  
Verify:
  - SAML authentication enabled
  - Correlation attribute set correctly (email)
  - Matches SAML attribute name
  
If correlation attribute wrong:
  - Update to correct attribute
  - Save
  - Test login again
```

**Step 6: Check SAML Assertion Signature**
```
Verify ISC can validate SAML assertion:

Check for certificate errors in ISC logs:

Potential issues:
  - Certificate expired
  - Wrong certificate uploaded
  - Certificate not trusted
  
Solution:
  - Download current cert from Azure AD
  - Upload to ISC
  - Verify certificate valid and not expired
```

**Step 7: Check User Permissions in ISC**
```
Even if user correlates, check access:

Identity found and correlated, but...

Check:
  - Does identity have any ISC access level?
  - Is identity in correct governance group?
  - Does identity have necessary permissions?
  
Possible issue:
  Identity exists but has no ISC permissions
  
Solution:
  - Assign appropriate access level
  - Add to governance group
  - Grant necessary permissions
```

**Most Likely Causes (In Order):**
```
1. Identity doesn't exist in ISC (50%)
   Solution: Create via identity profile or manually
   
2. Email mismatch (30%)
   Solution: Fix email in ISC or Azure AD
   
3. Identity disabled/inactive (10%)
   Solution: Enable identity, check lifecycle
   
4. SAML email attribute not sent (5%)
   Solution: Configure attribute in Azure AD
   
5. Other (certificate, permissions) (5%)
   Solution: Check logs, verify configuration
```

**Quick Diagnostic Checklist:**
```
☐ Identity exists in ISC? (Search by email)
☐ Identity status = Active?
☐ Identity lifecycle = active?
☐ Identity email = john.smith@company.com?
☐ SAML sends email attribute?
☐ ISC correlation attribute = email?
☐ SAML certificate valid?
☐ User assigned in Azure AD?
☐ Identity has ISC permissions?
```

**Resolution Process:**
```
1. Identify which check failed
2. Fix the issue
3. Test login again
4. Verify successful access
5. Document resolution for future reference
```

**Exam Tip:** "SAML login successful but access denied" usually means identity doesn't exist or email correlation failing. Check identity exists and emails match exactly.
</details>

---

## Section Summary

**Core Concepts for Certification:**

✅ **Authentication Methods:**
- **Internal**: ISC manages passwords (dev/test only)
- **SAML**: SSO via external IdP (production standard)
- **OAuth/OIDC**: API authentication (not user SSO)
- **None**: Service identities (no login)

✅ **SAML Components:**
- **Service Provider (SP)**: ISC
- **Identity Provider (IdP)**: Azure AD, Okta, etc.
- **SAML Assertion**: XML with user identity and attributes
- **Required**: Entity ID, ACS URL, Certificate

✅ **SAML Configuration:**
1. Configure ISC as app in IdP
2. Get IdP metadata/certificate
3. Configure SAML in ISC
4. Map attributes (email for correlation)
5. Test with pilot users
6. Deploy to all users

✅ **Best Practices:**
- Use SAML for production (not internal)
- Enforce MFA at IdP level
- Correlate on email attribute
- Test before full deployment
- Monitor certificate expiration
- Document all configuration

✅ **Common Issues:**
- Redirect loop: Check URLs and Entity ID
- Access denied: Verify identity exists and email matches
- Attributes not updating: Check SAML assertion and mapping
- Certificate errors: Renew certificate

**Exam Focus Areas:**
- Know SAML is production standard
- Understand SAML components (SP, IdP, assertion)
- Know correlation based on email attribute
- Understand configuration steps at high level
- Recognize common SAML troubleshooting scenarios
- Remember: Internal auth for dev/test, SAML for production

---

**End of Section 4.3: Authentication in Identity Profiles**

**Section 4 Summary - All Completed:**
- ✅ 4.1: Identity Profiles (purpose, configuration, priority)
- ✅ 4.2: Identity Attributes (mappings, transforms, functions)
- ✅ 4.3: Authentication (SAML, internal, configuration)

**Next:** Section 4.4: Lifecycle States

---

**Study Tips:**
- Memorize: SAML = production, Internal = dev/test only
- Know SAML flow: User → ISC → IdP → Authenticate → Assertion → ISC
- Remember correlation: Usually on email attribute
- Understand common issues: Identity doesn't exist, email mismatch
- Know components: SP (ISC), IdP (Azure AD), Certificate, URLs
