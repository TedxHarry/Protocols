
# Section 1.6: Authentication and Configuration

**Master tenant authentication options, configuration methods, and backup/restore procedures in SailPoint Identity Security Cloud**

---

## Table of Contents

- [1.6.1 Tenant Authentication Options](#161-tenant-authentication-options)
- [1.6.2 Configuring Authentication Methods](#162-configuring-authentication-methods)
- [1.6.3 Backup and Restore Configurations](#163-backup-and-restore-configurations)

---

# 1.6.1 Tenant Authentication Options

**Learning Objective:** Understand the different ways users can authenticate to your ISC tenant and when to use each method.

---

## What is Tenant Authentication?

Tenant authentication determines how users log into your ISC environment.

**Authentication controls:**
- How users prove their identity
- Password requirements
- Multi-factor authentication (MFA)
- Single sign-on (SSO) options
- Session management

**Why authentication matters:**
- Security foundation
- User experience
- Compliance requirements
- Integration with corporate identity

---

## Available Authentication Methods

### **Overview of Options**

| Method | Description | Best For |
|--------|-------------|----------|
| **Internal** | ISC manages passwords | Non-production, testing |
| **SAML 2.0** | SSO with external IdP | Production environments |
| **OAuth/OIDC** | Modern authentication | API access, integrations |
| **Multi-Factor (MFA)** | Additional verification layer | All production access |

---

## Internal Authentication

### **What is Internal Authentication?**

ISC stores and manages user passwords directly.

**How it works:**
```
1. User enters username/password
2. ISC validates credentials
3. ISC manages password lifecycle
4. ISC enforces password policies
```

**Features:**
- Password policies (complexity, expiration)
- Self-service password reset
- Account lockout protection
- Password history

### **When to Use Internal Authentication**

✅ **Appropriate for:**
- Development/sandbox tenants
- Testing environments
- Proof of concept
- Small deployments (<50 users)
- Environments without SSO infrastructure

❌ **NOT appropriate for:**
- Production enterprise environments
- Compliance-regulated environments
- Large user populations
- Environments with SSO capability

### **Internal Authentication Configuration**

**Basic settings:**
```
Authentication Type: Internal

Password Policy:
  Minimum Length: 12 characters
  Complexity Required: Yes
    - Uppercase letters
    - Lowercase letters
    - Numbers
    - Special characters
  
  Password Expiration: 90 days
  Password History: 12 passwords
  
Account Lockout:
  Failed Attempts: 5
  Lockout Duration: 30 minutes
  Reset After: 15 minutes of inactivity

Self-Service Reset:
  Enabled: Yes
  Method: Email verification
  Security Questions: 3 required
```

### **Limitations of Internal Authentication**

**Drawbacks:**
- Users manage separate ISC password
- No integration with corporate SSO
- Higher support burden (password resets)
- Less secure than SSO + MFA
- No centralized authentication policy
- More complex user lifecycle management

---

## SAML 2.0 Authentication (SSO)

### **What is SAML Authentication?**

SAML (Security Assertion Markup Language) enables Single Sign-On with external identity providers.

**How SAML works:**
```
1. User attempts to access ISC
2. ISC redirects to identity provider (IdP)
3. User authenticates at IdP (e.g., Azure AD)
4. IdP sends SAML assertion to ISC
5. ISC validates assertion
6. User logged into ISC (no password in ISC)
```

**Key components:**
- **Identity Provider (IdP):** External system (Azure AD, Okta, Ping)
- **Service Provider (SP):** SailPoint ISC
- **SAML Assertion:** Secure token proving identity
- **Metadata:** Configuration exchange between IdP and SP

### **Benefits of SAML SSO**

✅ **Advantages:**
- Single password for users (IdP password)
- Centralized authentication
- MFA enforced at IdP level
- Reduced help desk calls
- Better security posture
- Corporate policy enforcement
- Simplified user lifecycle
- Audit trail at IdP

**User experience:**
```
Without SSO:
  User has 15 different passwords
  Must remember ISC password separately
  Password resets for each system

With SSO:
  User has 1 corporate password
  Logs in once, accesses all apps
  Password managed centrally
```

### **Common SAML Identity Providers**

**Enterprise IdPs:**
- Microsoft Azure Active Directory (Azure AD / Entra ID)
- Okta
- Ping Identity (PingFederate, PingOne)
- ForgeRock
- Auth0
- OneLogin
- Google Workspace
- ADFS (Active Directory Federation Services)

**ISC acts as Service Provider (SP) to these IdPs**

---

## OAuth 2.0 / OIDC Authentication

### **What is OAuth/OIDC?**

Modern authentication protocols for API access and application integration.

**OAuth 2.0:**
- Authorization framework
- Used for API access
- Token-based authentication

**OIDC (OpenID Connect):**
- Built on OAuth 2.0
- Adds identity layer
- Used for user authentication

### **OAuth in ISC Context**

**Primary use cases:**
1. **API Authentication**
   - Personal Access Tokens (PAT)
   - OAuth client credentials
   - See Section 1.2.1 for details

2. **Application Integration**
   - Third-party apps accessing ISC
   - Custom integrations
   - Service-to-service authentication

**Not typically used for:**
- End user login to ISC web interface
- Regular administrative access
- Manual user authentication

---

## Multi-Factor Authentication (MFA)

### **What is MFA?**

Additional verification beyond password.

**Authentication factors:**
1. **Something you know:** Password, PIN
2. **Something you have:** Phone, hardware token
3. **Something you are:** Fingerprint, face recognition

**MFA requires 2+ factors**

### **MFA Methods in ISC**

**Available MFA options:**

| Method | Description | Security Level |
|--------|-------------|----------------|
| **Authenticator App** | Google/Microsoft Authenticator | High |
| **SMS Code** | Text message with code | Medium |
| **Email Code** | Email with verification code | Low |
| **Hardware Token** | Physical security key | Very High |
| **Push Notification** | Approve on mobile app | High |

**Recommended: Authenticator App or Hardware Token**

### **MFA Configuration Levels**

**Level 1: Optional MFA**
```
MFA: Enabled
Required: No
Users Can Opt-in: Yes

Use Case: Transition period, user education
```

**Level 2: Required for Admins**
```
MFA: Enabled
Required: Administrators only
Regular Users: Optional

Use Case: Most common production configuration
```

**Level 3: Required for All Users**
```
MFA: Enabled
Required: All users
No Exceptions: True

Use Case: High security environments, compliance
```

### **MFA Best Practices**

✅ **Recommendations:**
- Require MFA for all administrators (minimum)
- Support multiple MFA methods
- Allow users to register multiple devices
- Provide clear enrollment instructions
- Have fallback/recovery process
- Test MFA before enforcing
- Consider requiring MFA for all users

❌ **Avoid:**
- SMS as only MFA option (less secure)
- Email-based MFA in production
- No MFA for administrators
- No recovery process
- Forcing single MFA method only

---

## Authentication Architecture Patterns

### **Pattern 1: Internal Auth (Non-Production)**
```
User → ISC
       ↓
   Password stored in ISC
       ↓
   ISC validates
       ↓
   Access granted

Use: Dev/Test environments only
```

### **Pattern 2: SAML SSO (Production Standard)**
```
User → ISC
       ↓
   Redirect to Azure AD
       ↓
   User logs into Azure AD
   (with corporate password + MFA)
       ↓
   Azure AD sends SAML assertion
       ↓
   ISC validates assertion
       ↓
   Access granted (no password in ISC)

Use: Standard production deployment
```

### **Pattern 3: SAML + Conditional MFA (Advanced)**
```
User → ISC
       ↓
   Redirect to Okta
       ↓
   Okta checks:
     - User location
     - Device trust
     - Risk score
       ↓
   If High Risk: Require MFA
   If Low Risk: Password only
       ↓
   Okta sends assertion
       ↓
   Access granted

Use: Risk-based authentication
```

---

## Session Management

### **Session Settings**

Control how long users stay logged in.

**Key settings:**
```
Session Timeout:
  Idle Timeout: 30 minutes
  Maximum Session: 12 hours
  
Remember Me:
  Enabled: Yes
  Duration: 7-30 days
  
Session Security:
  Session Binding: IP address + User-Agent
  Concurrent Sessions: Allow 3 per user
```

**Session timeout behavior:**
```
Idle Timeout (30 min):
  - User inactive for 30 minutes
  - Warning at 28 minutes: "Session expiring"
  - User can extend or logout
  - Auto-logout at 30 minutes

Maximum Session (12 hours):
  - Hard limit regardless of activity
  - User must re-authenticate
  - Prevents indefinite sessions
```

### **Session Best Practices**

**Production recommendations:**
```
Standard Users:
  Idle Timeout: 30 minutes
  Max Session: 12 hours
  Remember Me: 30 days

Administrators:
  Idle Timeout: 15 minutes
  Max Session: 8 hours
  Remember Me: 7 days or disabled

High Security:
  Idle Timeout: 10 minutes
  Max Session: 4 hours
  Remember Me: Disabled
```

---

## Choosing Authentication Method

### **Decision Matrix**

**For Non-Production Tenants:**
```
Environment: Development, Sandbox, POC
Users: <50
Duration: Temporary

Recommendation: Internal Authentication
Rationale: Simple, sufficient for testing
```

**For Production - Small Deployment:**
```
Environment: Production
Users: <100
SSO Available: No

Recommendation: Internal + MFA
Rationale: Secure, no SSO dependency
```

**For Production - Enterprise:**
```
Environment: Production
Users: >100
SSO Available: Yes

Recommendation: SAML SSO + MFA at IdP
Rationale: Best security, user experience, manageability
```

**For High Security:**
```
Environment: Production
Compliance: SOC2, HIPAA, etc.
Risk Level: High

Recommendation: SAML SSO + Hardware MFA
Rationale: Maximum security, audit compliance
```

### **Migration Path**

**Typical evolution:**
```
Phase 1: POC
  Internal authentication
  Test users only

Phase 2: Pilot
  SAML SSO configured
  Pilot user group
  Internal auth still available

Phase 3: Production
  SAML SSO for all users
  MFA required for admins
  Internal auth disabled

Phase 4: Hardened
  SAML SSO mandatory
  MFA for all users
  Advanced security policies
```

---

## Authentication Troubleshooting

### **Common Issues**

**Issue 1: Users can't login**

**Possible causes:**
- Account locked (too many failed attempts)
- Password expired
- MFA device unavailable
- SAML configuration error
- IdP unavailable

**Troubleshooting steps:**
1. Check account status
2. Verify authentication method
3. Test SAML configuration
4. Check IdP availability
5. Review audit logs
6. Check user's MFA enrollment

**Issue 2: SAML login fails**

**Error: "SAML assertion invalid"**

**Check:**
- SAML certificate expired?
- Clock skew between ISC and IdP?
- SAML assertion signing correct?
- Attribute mapping correct?

**Issue 3: MFA enrollment fails**

**Possible causes:**
- QR code not scanning
- Wrong time on authenticator device
- Email not received
- SMS not delivered

**Solutions:**
- Provide manual entry code
- Check device time sync
- Verify email address
- Try alternative MFA method

---

## Quick Reference

### **Authentication Methods Summary**

| Method | Security | Complexity | Best For |
|--------|----------|------------|----------|
| **Internal** | Low-Medium | Low | Dev/Test |
| **SAML SSO** | High | Medium | Production |
| **SAML + MFA** | Very High | Medium | Enterprise |
| **SAML + Hardware MFA** | Highest | High | High Security |

### **Configuration Checklist**
```
Production Authentication Setup:

☐ Choose authentication method (SAML recommended)
☐ Configure IdP integration
☐ Map user attributes
☐ Set password policies (if using internal)
☐ Enable MFA
☐ Configure session timeouts
☐ Test authentication flow
☐ Test MFA enrollment
☐ Document configuration
☐ Train users
☐ Plan fallback/recovery
```

---

## Practice Exercises

**Exercise 1:** You're setting up a production ISC tenant for 500 users. Your company uses Azure AD. What authentication method should you use?

<details>
<summary>Answer</summary>

**SAML 2.0 SSO with Azure AD**

**Why:**
- ✅ Production environment requires SSO
- ✅ 500 users = enterprise scale
- ✅ Azure AD already available
- ✅ Centralized authentication
- ✅ Better security and user experience

**Configuration:**
1. Configure SAML integration with Azure AD
2. Enable MFA at Azure AD level
3. Map user attributes (email, name)
4. Test with pilot group
5. Roll out to all users
6. Disable internal authentication

**Do NOT use:**
- ❌ Internal authentication (not appropriate for production)
- ❌ Separate ISC passwords (poor user experience)
</details>

---

**Exercise 2:** What's the minimum MFA requirement for a production ISC tenant?

<details>
<summary>Answer</summary>

**Minimum: MFA required for all administrators**

**Reasoning:**
- Administrators have elevated privileges
- Compromise would be severe
- Industry best practice
- Most compliance frameworks require it

**Recommended configuration:**
```
MFA Settings:
  Enabled: Yes
  Required For: Administrators (minimum)
  Methods Allowed:
    - Authenticator app
    - Hardware token
  
  Optional: Regular users can opt-in
```

**Better: MFA for all users**
- Even higher security
- Many organizations require this
- Compliance requirements

**Never acceptable:**
- ❌ No MFA for any users
- ❌ SMS-only MFA for admins
</details>

---

**Exercise 3:** A user's session keeps timing out after 10 minutes of activity. What setting controls this?

<details>
<summary>Answer</summary>

**This is controlled by: Idle Timeout setting**

**The user is likely hitting:**
```
Session Settings:
  Idle Timeout: 10 minutes ← This one
  Maximum Session: 12 hours
```

**Difference:**
- **Idle Timeout:** Logs out after inactivity
- **Maximum Session:** Hard limit regardless of activity

**In this case:**
- User is active but still logging out
- This suggests Maximum Session is set too low OR
- Session binding is causing re-authentication OR
- There's a configuration error

**Should check:**
1. Idle timeout value
2. Maximum session value
3. Session binding settings
4. Browser cookie settings
5. Any proxy/network issues
</details>

---

# 1.6.2 Configuring Authentication Methods

**Learning Objective:** Learn step-by-step how to configure SAML SSO and MFA in ISC.

---

## Before You Begin

### **Prerequisites**

**For SAML configuration:**
- Admin access to ISC tenant
- Admin access to identity provider (Azure AD, Okta, etc.)
- SSL certificate from IdP
- Understanding of SAML basics
- Test user accounts

**For MFA configuration:**
- Admin access to ISC
- Test mobile device or authenticator app
- User email addresses configured

---

## Configuring SAML 2.0 SSO

### **High-Level SAML Setup Process**
```
1. Gather information from ISC
2. Configure ISC as app in IdP
3. Upload IdP metadata to ISC
4. Map user attributes
5. Test with pilot user
6. Enable for all users
```

---

## Step-by-Step: SAML with Azure AD

### **Part 1: Gather ISC Information**

**Step 1: Access ISC SAML Settings**

1. Log into ISC as administrator
2. Navigate to: **Admin** → **Security Settings** → **Authentication**
3. Click **External Identity Provider**
4. Select **SAML 2.0**

**Step 2: Copy ISC Service Provider Details**
```
ISC provides:
  Entity ID: https://yourcompany.identitynow.com
  ACS URL: https://yourcompany.login.sailpoint.com/saml/SSO/alias/yourcompany-sp
  
Copy these - you'll need them for Azure AD
```

### **Part 2: Configure Azure AD**

**Step 1: Create Enterprise Application**

1. Log into Azure AD portal
2. Go to: **Enterprise Applications**
3. Click **New application**
4. Click **Create your own application**
5. Name: "SailPoint IdentityNow"
6. Select: **Integrate any other application you don't find in the gallery**
7. Click **Create**

**Step 2: Configure Single Sign-On**

1. In the new app, click **Single sign-on**
2. Select **SAML**
3. Click **Edit** on Basic SAML Configuration

**Step 3: Enter ISC Details**
```
Identifier (Entity ID):
  https://yourcompany.identitynow.com

Reply URL (ACS URL):
  https://yourcompany.login.sailpoint.com/saml/SSO/alias/yourcompany-sp

Sign on URL (optional):
  https://yourcompany.identitynow.com
```

Click **Save**

**Step 4: Configure Attributes**

Under **Attributes & Claims**, verify:
```
Required Claim:
  Name: email
  Source Attribute: user.mail
  
Optional Claims:
  Name: firstname
  Source Attribute: user.givenname
  
  Name: lastname
  Source Attribute: user.surname
  
  Name: displayName
  Source Attribute: user.displayname
```

**Step 5: Download Certificate**

1. Under **SAML Signing Certificate**
2. Download **Certificate (Base64)**
3. Save file (you'll upload to ISC)

**Step 6: Copy Azure AD URLs**
```
Copy these from Azure AD:
  Login URL: https://login.microsoftonline.com/...
  Azure AD Identifier: https://sts.windows.net/...
```

### **Part 3: Configure ISC with Azure AD Details**

**Step 1: Upload IdP Certificate**

1. Back in ISC: **Security Settings** → **Authentication** → **SAML**
2. Under **Identity Provider Certificate**
3. Click **Import** or **Upload**
4. Select the certificate file from Azure AD
5. Click **Save**

**Step 2: Enter IdP Details**
```
Identity Provider Settings:

Entity ID (from Azure AD):
  https://sts.windows.net/your-tenant-id/

SSO URL (Login URL from Azure AD):
  https://login.microsoftonline.com/.../saml2

SLO URL (Logout URL - optional):
  https://login.microsoftonline.com/.../saml2
```

**Step 3: Configure Attribute Mapping**
```
ISC Attribute    →    IdP Attribute
--------------         --------------
email            →    email (or http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress)
firstname        →    firstname
lastname         →    lastname
displayName      →    displayName
```

**Step 4: Save Configuration**

Click **Save** or **Update**

### **Part 4: Assign Users in Azure AD**

**Step 1: Assign Users/Groups**

1. In Azure AD app
2. Go to **Users and groups**
3. Click **Add user/group**
4. Select test user first
5. Click **Assign**

**For pilot testing:**
- Assign 2-3 test users only
- Test thoroughly
- Then assign broader groups

### **Part 5: Test SAML Authentication**

**Step 1: Test with Test User**

1. Open incognito/private browser window
2. Go to: `https://yourcompany.identitynow.com`
3. Should redirect to Azure AD login
4. Enter test user credentials
5. Should redirect back to ISC
6. Verify successful login

**Step 2: Verify User Details**

1. Check user profile in ISC
2. Verify email, name populated correctly
3. Check attributes mapped properly

**Step 3: Test Logout**

1. Log out of ISC
2. Verify proper logout
3. Test login again

**If issues occur:**
- Check SAML assertion in browser dev tools
- Review Azure AD sign-in logs
- Check ISC audit logs
- Verify certificate is valid
- Check attribute mapping

### **Part 6: Enable for All Users**

**Once testing succeeds:**

1. In Azure AD, assign all user groups
2. In ISC, enable SAML for all users
3. Communicate change to users
4. Monitor for issues
5. Provide support documentation

**Optional: Disable Internal Authentication**
```
Once SAML proven working:
1. Verify all users can login via SAML
2. Keep internal auth for emergency admin
3. Or disable completely for full SSO enforcement
```

---

## Step-by-Step: SAML with Okta

### **Quick Configuration Guide**

**Part 1: Add ISC to Okta**

1. Log into Okta admin console
2. Go to **Applications** → **Applications**
3. Click **Browse App Catalog**
4. Search for "SailPoint IdentityNow"
5. Click **Add Integration**
6. Name the integration
7. Click **Done**

**Part 2: Configure SAML Settings**

Okta pre-configures most settings for SailPoint.

**Update:**
```
Subdomain: yourcompany
(This is your ISC tenant name)
```

**Part 3: Assign Users**

1. Go to **Assignments** tab
2. Click **Assign** → **Assign to People** or **Assign to Groups**
3. Select users/groups
4. Click **Assign** → **Done**

**Part 4: Get Okta Metadata**

1. Go to **Sign On** tab
2. Under **SAML Setup**
3. Click **View SAML setup instructions**
4. Copy:
   - Identity Provider Single Sign-On URL
   - Identity Provider Issuer
   - Download X.509 Certificate

**Part 5: Configure in ISC**

1. In ISC: **Security Settings** → **Authentication** → **SAML**
2. Upload Okta certificate
3. Enter Okta SSO URL and Entity ID
4. Save configuration

**Part 6: Test**

1. Test login with pilot user
2. Verify successful authentication
3. Enable for all users

---

## Configuring Multi-Factor Authentication (MFA)

### **Step-by-Step: Enable MFA**

**Step 1: Access MFA Settings**

1. Log into ISC as admin
2. Go to: **Admin** → **Security Settings** → **Multi-Factor Authentication**
3. Or: **Global** → **Security Settings** → **Authentication** → **MFA**

**Step 2: Enable MFA**
```
MFA Configuration:
  Enable MFA: ☑ Yes
  
Enforcement:
  ○ Optional (users can opt-in)
  ○ Required for administrators only
  ● Required for all users
  
Allowed Methods:
  ☑ Authenticator App (Google/Microsoft)
  ☑ Hardware Token
  ☐ SMS (less secure - optional)
  ☐ Email (not recommended)
```

**Step 3: Configure Grace Period**
```
Enrollment Grace Period: 7 days

During this period:
  - Users see MFA enrollment prompt
  - Can skip X times
  - Must enroll by deadline
  - After deadline: Cannot login without MFA
```

**Step 4: Configure Remember Device**
```
Remember Device Settings:
  Enable: Yes
  Duration: 30 days
  
Trusted Networks:
  - Corporate office: Don't require MFA
  - All other locations: Require MFA
```

**Step 5: Save Configuration**

Click **Save** or **Apply**

### **Step 6: User MFA Enrollment Process**

**What users experience:**

**First login after MFA enabled:**

1. User logs in with username/password
2. Sees: "Multi-Factor Authentication Required"
3. Clicks "Set up authenticator"

**Enrollment steps:**
```
1. Download authenticator app
   - Google Authenticator
   - Microsoft Authenticator
   - Authy
   
2. Scan QR code
   Or enter manual code:
   ABCD EFGH IJKL MNOP
   
3. Authenticator generates 6-digit code
   
4. Enter code in ISC
   
5. Verify successful
   
6. Save backup codes (optional)
   
7. MFA enrollment complete
```

**Subsequent logins:**
```
1. Enter username/password
2. Open authenticator app
3. Enter 6-digit code
4. Click "Verify"
5. Logged in
```

---

## MFA Administration

### **Helping Users with MFA**

**Common user issues:**

**Issue: Can't scan QR code**
```
Solution: Provide manual entry code
  Code: ABCD EFGH IJKL MNOP
  User enters manually in authenticator
```

**Issue: Code not working**
```
Possible causes:
  - Device time not synced
  - Using old code (30-second expiration)
  - Wrong authenticator app

Solutions:
  - Check device time sync
  - Wait for new code
  - Verify correct app
```

**Issue: Lost phone with authenticator**
```
Solution: Admin MFA reset
  1. Admin: Find user in ISC
  2. Go to user's MFA settings
  3. Click "Reset MFA"
  4. User can re-enroll on next login
```

### **Admin MFA Management**

**Reset user's MFA:**

1. Navigate to: **Admin** → **Users**
2. Search for user
3. Click user profile
4. Go to **Authentication** or **MFA** section
5. Click **Reset MFA** or **Clear MFA**
6. Confirm reset
7. User must re-enroll on next login

**Bypass MFA temporarily:**
```
For emergency admin access:

Option 1: Use break-glass account
  - Dedicated account with MFA bypass
  - Highly monitored
  - Use only in emergency

Option 2: Temporary exemption
  - Grant user temporary MFA bypass
  - Set expiration (24 hours)
  - Monitor usage
  - Require re-enrollment after
```

---

## Authentication Migration Strategies

### **Strategy 1: Pilot Approach**

**Safest for production**
```
Week 1: Planning
  - Document current state
  - Design target state
  - Create pilot group (10-20 users)
  
Week 2: Configure
  - Set up SAML in test environment
  - Test thoroughly
  - Document issues
  
Week 3: Pilot
  - Enable SAML for pilot group
  - Keep internal auth available
  - Monitor closely
  - Gather feedback
  
Week 4: Evaluate
  - Review pilot results
  - Fix any issues
  - Document learnings
  
Week 5: Rollout Phase 1
  - Enable for 25% of users
  - Monitor for issues
  
Week 6: Rollout Phase 2
  - Enable for 50% of users
  
Week 7: Rollout Phase 3
  - Enable for 100% of users
  
Week 8: Cleanup
  - Disable internal authentication
  - Final verification
```

### **Strategy 2: Big Bang (Higher Risk)**

**Only for small deployments**
```
Preparation:
  - Thorough testing in non-prod
  - All users trained
  - Support team ready
  - Communication sent
  
Cutover:
  - Weekend maintenance window
  - Enable SAML for all users
  - Disable internal auth
  - Monitor closely
  - Support team on call
```

---

## Troubleshooting Authentication

### **SAML Troubleshooting**

**Tools to use:**
- Browser developer tools (Network tab)
- SAML tracer browser extension
- ISC audit logs
- IdP sign-in logs

**Common SAML errors:**

**Error: "SAML assertion not valid"**
```
Check:
  ☐ Certificate not expired
  ☐ Clock sync between ISC and IdP
  ☐ Assertion signed correctly
  ☐ Audience/Entity ID matches
  ☐ ACS URL correct
```

**Error: "Attribute missing"**
```
Check:
  ☐ IdP sending required attributes (email)
  ☐ Attribute mapping in ISC correct
  ☐ Attribute names match
  ☐ User has attributes in IdP
```

**Error: "User not found"**
```
Possible causes:
  - User doesn't exist in ISC
  - Email attribute doesn't match
  - User disabled in ISC

Solutions:
  - Verify user exists
  - Check email match
  - Verify user active
```

---

## Quick Reference

### **SAML Configuration Checklist**
```
☐ ISC Service Provider info copied
☐ IdP application created
☐ ISC details entered in IdP
☐ Attributes configured in IdP
☐ Certificate downloaded from IdP
☐ Certificate uploaded to ISC
☐ IdP URLs entered in ISC
☐ Attribute mapping configured
☐ Test user assigned
☐ SAML login tested successfully
☐ Pilot group assigned
☐ Pilot monitoring period complete
☐ All users assigned
☐ Communication sent to users
```

### **MFA Configuration Checklist**
```
☐ MFA enabled in ISC
☐ Enforcement level selected
☐ Allowed methods chosen
☐ Grace period configured
☐ Remember device settings configured
☐ Admin team enrolled
☐ Pilot users enrolled
☐ User documentation created
☐ Support team trained
☐ Rollout communicated
☐ All users enrolled
```

---

## Practice Exercises

**Exercise 4:** You're configuring SAML with Azure AD. What information do you need from ISC to configure in Azure?

<details>
<summary>Answer</summary>

**Information needed from ISC:**

1. **Entity ID** (Identifier)
```
   https://yourcompany.identitynow.com
```

2. **ACS URL** (Reply URL)
```
   https://yourcompany.login.sailpoint.com/saml/SSO/alias/yourcompany-sp
```

3. **Sign-on URL** (optional)
```
   https://yourcompany.identitynow.com
```

**Where to enter in Azure AD:**
- Go to Enterprise Application for ISC
- Single Sign-On → SAML
- Basic SAML Configuration
- Enter the above URLs

**Also need to know:**
- Required attributes to map (email, name, etc.)
- Whether to assign users or groups first
</details>

---

**Exercise 5:** A user reports their MFA code isn't working. What should you check?

<details>
<summary>Answer</summary>

**Troubleshooting steps:**

1. **Check time sync**
```
   Device time must be accurate
   MFA codes expire after 30 seconds
   Time sync issues = codes won't work
```

2. **Verify correct code**
```
   User might be using old code
   Wait for new code to generate
   Codes change every 30 seconds
```

3. **Check authenticator app**
```
   Verify using correct app entry
   User might have multiple ISC entries
   Ensure using right organization
```

4. **Check enrollment**
```
   Verify user's MFA is enrolled
   Check in ISC user profile
```

**If still not working:**
- **Admin reset MFA** for user
- User re-enrolls on next login
- Test with new enrollment

**Prevention:**
- Provide clear enrollment instructions
- Test enrollment immediately
- Save backup codes during enrollment
</details>

---

# 1.6.3 Backup and Restore Configurations

**Learning Objective:** Understand how to backup and restore ISC configurations for disaster recovery and change management.

---

## Why Backup Configurations?

**Backup protects against:**
- Accidental configuration changes
- Misconfiguration during updates
- Need to rollback changes
- Disaster recovery
- Compliance requirements
- Documentation needs

**What can be backed up:**
- Identity profiles
- Access profiles and roles
- Sources and connectors
- Provisioning policies
- Workflows
- Transforms
- Governance groups
- Certifications
- Rules

**What is NOT backed up:**
- Actual identity data (managed by SailPoint)
- Historical events
- Audit logs
- Temporary runtime data

---

## Configuration Backup Methods

### **Method 1: Configuration Export (UI)**

**Manual export through ISC interface**

**Step 1: Navigate to Export**

1. Log into ISC as admin
2. Go to: **Admin** → **Import/Export** or **Configuration**
3. Select **Export**

**Step 2: Select Objects to Export**
```
Select Configuration Objects:
  ☑ Identity Profiles
  ☑ Access Profiles
  ☑ Roles
  ☑ Sources
  ☑ Transforms
  ☑ Workflows
  ☑ Provisioning Policies
  ☐ Actual identity data (not recommended)
```

**Step 3: Export**

1. Click **Export**
2. Download JSON file
3. Save securely with version/date

**File format:**
```
sailpoint-config-export-2024-02-12.json
```

**File contains:**
```json
{
  "version": "1.0",
  "tenant": "yourcompany",
  "exportDate": "2024-02-12T14:30:00Z",
  "objects": {
    "identityProfiles": [...],
    "accessProfiles": [...],
    "roles": [...],
    "sources": [...],
    "transforms": [...]
  }
}
```

### **Method 2: API-Based Backup**

**Automated backup using ISC API**

**Why use API:**
- Automation
- Scheduled backups
- Version control integration
- Selective object backup

**Example: Backup Script**
```python
import requests
import json
from datetime import datetime

# Configuration
TENANT = "yourcompany"
CLIENT_ID = "your-client-id"
CLIENT_SECRET = "your-client-secret"

# Get access token
def get_token():
    url = f"https://{TENANT}.api.identitynow.com/oauth/token"
    data = {
        'grant_type': 'client_credentials',
        'client_id': CLIENT_ID,
        'client_secret': CLIENT_SECRET
    }
    response = requests.post(url, data=data)
    return response.json()['access_token']

# Backup sources
def backup_sources(token):
    url = f"https://{TENANT}.api.identitynow.com/v3/sources"
    headers = {'Authorization': f'Bearer {token}'}
    response = requests.get(url, headers=headers)
    return response.json()

# Backup access profiles
def backup_access_profiles(token):
    url = f"https://{TENANT}.api.identitynow.com/v3/access-profiles"
    headers = {'Authorization': f'Bearer {token}'}
    response = requests.get(url, headers=headers)
    return response.json()

# Main backup function
def perform_backup():
    token = get_token()
    
    backup_data = {
        'timestamp': datetime.now().isoformat(),
        'sources': backup_sources(token),
        'access_profiles': backup_access_profiles(token)
    }
    
    # Save to file
    filename = f"isc-backup-{datetime.now().strftime('%Y%m%d-%H%M%S')}.json"
    with open(filename, 'w') as f:
        json.dump(backup_data, f, indent=2)
    
    print(f"Backup saved to {filename}")

# Run backup
perform_backup()
```

### **Method 3: Version Control (Git)**

**Store configurations in Git repository**

**Workflow:**
```
1. Export configurations via API or UI
2. Commit to Git repository
3. Tag with version/date
4. Push to remote (GitHub, GitLab, etc.)

Benefits:
  - Version history
  - Change tracking
  - Rollback capability
  - Collaboration
  - Audit trail
```

**Example structure:**
```
isc-configurations/
├── identity-profiles/
│   ├── employees.json
│   └── contractors.json
├── access-profiles/
│   ├── email-access.json
│   └── vpn-access.json
├── sources/
│   ├── active-directory.json
│   └── salesforce.json
├── transforms/
│   ├── generate-email.json
│   └── calculate-lifecycle.json
└── README.md
```

---

## Backup Best Practices

### **Backup Schedule**

**Recommended frequency:**
```
Daily (Automated):
  - Sources
  - Access profiles
  - Roles
  - Critical workflows

Weekly (Automated):
  - Full configuration export
  - All transforms
  - All provisioning policies

Before Major Changes:
  - Manual full backup
  - Document what's changing
  - Test restore process

After Major Changes:
  - Backup new configuration
  - Compare with previous
  - Document differences
```

### **Backup Storage**

**Where to store backups:**

✅ **Recommended:**
- Version control system (Git)
- Secure file storage (encrypted)
- Cloud storage (AWS S3, Azure Blob)
- Network share with access controls
- Document management system

❌ **Not recommended:**
- Personal laptops only
- Unencrypted storage
- Public repositories
- Email attachments
- Temporary storage

**Storage retention:**
```
Daily backups: Keep 7 days
Weekly backups: Keep 4 weeks
Monthly backups: Keep 12 months
Before major changes: Keep 2 years
```

---

## Configuration Restore

### **When to Restore**

**Common scenarios:**
- Rollback failed change
- Restore after misconfiguration
- Disaster recovery
- Clone to new tenant
- Migrate configurations

### **Restore Process (UI)**

**Step 1: Prepare**
```
Before restore:
  ☐ Backup current state first
  ☐ Review what will be restored
  ☐ Notify team
  ☐ Plan maintenance window
  ☐ Test in non-prod if possible
```

**Step 2: Navigate to Import**

1. Log into ISC as admin
2. Go to: **Admin** → **Import/Export**
3. Select **Import**

**Step 3: Upload Backup File**

1. Click **Choose File**
2. Select backup JSON file
3. Review import summary

**Step 4: Select Import Options**
```
Import Options:
  ○ Replace existing (dangerous!)
  ● Create new + update existing
  ○ Create new only
  
Conflict Resolution:
  ○ Skip conflicts
  ● Overwrite with imported
  ○ Prompt for each
```

**Step 5: Execute Import**

1. Click **Import**
2. Monitor progress
3. Review results
4. Check for errors

**Step 6: Verify Restoration**

1. Check restored objects exist
2. Test functionality
3. Verify configurations correct
4. Test integrations

### **Restore Process (API)**

**API-based restore for automation:**
```python
def restore_source(token, source_config):
    """Restore a source configuration"""
    url = f"https://{TENANT}.api.identitynow.com/v3/sources"
    headers = {
        'Authorization': f'Bearer {token}',
        'Content-Type': 'application/json'
    }
    response = requests.post(url, headers=headers, json=source_config)
    return response.json()

# Load backup
with open('backup.json', 'r') as f:
    backup = json.load(f)

# Restore each source
token = get_token()
for source in backup['sources']:
    try:
        result = restore_source(token, source)
        print(f"Restored source: {source['name']}")
    except Exception as e:
        print(f"Error restoring {source['name']}: {e}")
```

---

## Disaster Recovery Planning

### **DR Preparation**

**Essential components:**
```
1. Documentation
   - Architecture diagrams
   - Configuration guides
   - Contact lists
   - Recovery procedures
   
2. Backups
   - Daily automated backups
   - Offsite storage
   - Multiple versions
   - Tested restores
   
3. Access
   - Emergency credentials
   - Break-glass accounts
   - Alternative authentication
   
4. Communication
   - Incident response plan
   - Stakeholder notifications
   - Status updates
```

### **DR Testing**

**Test your DR plan:**
```
Quarterly DR Test:

1. Simulate disaster
   - Configuration lost
   - Need to restore
   
2. Execute recovery
   - Access backup
   - Restore configuration
   - Verify functionality
   
3. Measure
   - Time to restore
   - Completeness
   - Issues encountered
   
4. Document
   - What worked
   - What failed
   - Improvements needed
   
5. Update plan
   - Fix issues
   - Update procedures
   - Train team
```

---

## Change Management Integration

### **Using Backups for Change Control**

**Before change:**
```
1. Create backup
2. Document what's changing
3. Get approval
4. Schedule change window
```

**During change:**
```
1. Make changes
2. Document actual changes
3. Test functionality
4. Create post-change backup
```

**After change:**
```
1. Compare before/after
2. Document differences
3. Update documentation
4. Archive backups
```

### **Rollback Procedure**

**If change fails:**
```
1. Stop further changes
2. Notify stakeholders
3. Retrieve pre-change backup
4. Restore configuration
5. Verify rollback successful
6. Document incident
7. Post-mortem analysis
```

---

## Configuration Documentation

### **What to Document**

**Essential documentation:**
```
1. Configuration Inventory
   - All sources
   - All access profiles
   - All roles
   - All workflows
   - All transforms
   
2. Change Log
   - Who changed what
   - When changed
   - Why changed
   - Approval records
   
3. Dependencies
   - What depends on what
   - Integration points
   - Critical paths
   
4. Procedures
   - How to backup
   - How to restore
   - Emergency contacts
   - Escalation paths
```

### **Documentation Format**

**Example configuration document:**
```markdown
# ISC Configuration Documentation

## Last Updated
2024-02-12

## Configuration Overview
- Tenant: yourcompany
- Environment: Production
- Last Backup: 2024-02-12 02:00 AM
- Next Scheduled Backup: 2024-02-13 02:00 AM

## Sources (5 total)
1. Active Directory
   - Type: Active Directory - Direct
   - VA: ISC-VA-01
   - Owner: IT Team
   - Last Aggregation: 2024-02-12 01:00 AM

2. Salesforce
   - Type: Salesforce
   - Connection: SaaS
   - Owner: Sales Ops
   - Last Aggregation: 2024-02-12 00:30 AM

## Access Profiles (23 total)
[List of access profiles with owners and purposes]

## Recent Changes
| Date | Type | Object | Change | By | Ticket |
|------|------|--------|--------|----|----- --|
| 2024-02-10 | Modified | VPN Access Profile | Added firewall rule | alice | CHG-1234 |
| 2024-02-08 | Created | Contractor Role | New contractor role | bob | CHG-1220 |

## Backup Locations
- Daily: s3://backups/isc/daily/
- Weekly: s3://backups/isc/weekly/
- Git: https://github.com/company/isc-configs
```

---

## Security Considerations

### **Protecting Backups**

**Security requirements:**
```
1. Encryption
   - Encrypt at rest
   - Encrypt in transit
   - Strong encryption (AES-256)
   
2. Access Control
   - Limited access (admins only)
   - MFA required
   - Audit access
   
3. Sensitive Data
   - Passwords not in backups
   - API keys separate
   - Secrets management
   
4. Compliance
   - Meet regulatory requirements
   - Retention policies
   - Geographic restrictions
```

**What NOT to include in backups:**

❌ **Never backup:**
- User passwords
- API credentials
- Client secrets
- OAuth tokens
- Personal Access Tokens
- Encryption keys

✅ **Safe to backup:**
- Configuration structures
- Access profile definitions
- Role memberships (IDs not passwords)
- Workflow definitions
- Transform logic
- Source configurations (without credentials)

---

## Quick Reference

### **Backup Checklist**
```
☐ Automated daily backups configured
☐ Backup storage secured
☐ Backup encryption enabled
☐ Multiple backup locations
☐ Backup tested (restore works)
☐ Documentation up to date
☐ Team trained on restore process
☐ DR plan documented
☐ Change management integrated
☐ Retention policy defined
```

### **Restore Checklist**
```
☐ Current state backed up first
☐ Correct backup file selected
☐ Import options reviewed
☐ Conflicts plan determined
☐ Team notified
☐ Maintenance window scheduled
☐ Restore executed
☐ Functionality verified
☐ Testing completed
☐ Documentation updated
```

---

## Practice Exercises

**Exercise 6:** You're about to make a major change to all access profiles. What should you do first?

<details>
<summary>Answer</summary>

**Before making major changes:**

1. **Create full backup**
```
   - Export all access profiles
   - Export related configurations
   - Save with clear naming
   - Store in multiple locations
```

2. **Document the change**
```
   - What's changing and why
   - Expected impact
   - Rollback plan
   - Approval evidence
```

3. **Test in non-production**
```
   - If possible, test in sandbox
   - Verify change works
   - Document any issues
```

4. **Prepare rollback**
```
   - Keep backup accessible
   - Test restore process
   - Have rollback procedure ready
```

5. **Schedule appropriately**
```
   - Low-usage time window
   - Team available for support
   - Stakeholders notified
```

**Do NOT:**
- ❌ Make changes without backup
- ❌ Skip documentation
- ❌ Change during peak hours
- ❌ Change without approval
</details>

---

**Exercise 7:** Your automated daily backup script failed last night. What should you do?

<details>
<summary>Answer</summary>

**Immediate actions:**

1. **Investigate failure**
```
   - Check error logs
   - Identify root cause
   - Document the issue
```

2. **Manual backup NOW**
```
   - Create backup immediately
   - Don't wait for tonight
   - Verify backup successful
```

3. **Fix automation**
```
   - Resolve root cause
   - Test fix
   - Monitor next run
```

4. **Verify backup integrity**
```
   - Check backup file valid
   - Test restore
   - Confirm completeness
```

5. **Prevent recurrence**
```
   - Add monitoring/alerting
   - Improve error handling
   - Document solution
```

6. **Update documentation**
```
   - Record incident
   - Update procedures
   - Share with team
```

**Remember:**
- Backups are useless if they don't work
- Test restores regularly
- Monitor backup jobs
- Have redundant backup methods
</details>

---

**Exercise 8:** What should NOT be included in configuration backups?

<details>
<summary>Answer</summary>

**Never include in backups:**

❌ **Credentials and Secrets:**
- User passwords
- API keys
- Client secrets
- OAuth tokens
- Personal Access Tokens
- Encryption keys
- Service account passwords

❌ **Sensitive PII:**
- Social Security Numbers
- Credit card numbers
- Health information
- Personal financial data

❌ **Runtime Data:**
- Actual identity records (unless specifically needed)
- Audit logs (backed up separately)
- Temporary session data
- Cache files

**Safe to include:**

✅ **Configuration:**
- Identity profile definitions
- Access profile structures
- Role definitions (membership logic)
- Source configurations (metadata only)
- Workflow definitions
- Transform logic
- Provisioning policies

**Why?**
- Security: Credentials in backups = major risk
- Compliance: PII has separate requirements
- Practical: Configuration ≠ data
</details>

---

## Section Summary

You've learned:

✅ **Authentication Options** - Internal, SAML, MFA methods  
✅ **SAML Configuration** - Step-by-step SSO setup  
✅ **MFA Implementation** - Multi-factor authentication  
✅ **Backup Strategies** - Protect configurations  
✅ **Restore Procedures** - Disaster recovery  
✅ **Best Practices** - Security, testing, documentation  

**Key Takeaways:**
- Use SAML SSO in production
- Require MFA for administrators
- Automate configuration backups
- Test restore procedures regularly
- Document everything
- Integrate with change management

**Congratulations!** You've completed **Section 1: Platform Fundamentals**

---

## Additional Resources

📚 **Official Documentation:**
- [Authentication Guide](https://documentation.sailpoint.com/saas/help/common/admin_console.html)
- [SAML Configuration](https://documentation.sailpoint.com/saas/help/common/saml.html)
- [MFA Setup](https://documentation.sailpoint.com/saas/help/common/mfa.html)
- [Configuration Management](https://documentation.sailpoint.com/saas/help/common/config_mgmt.html)

💡 **Pro Tips:**
- Always backup before major changes
- Test authentication in non-prod first
- Keep multiple backup copies
- Document your DR plan
- Train team on procedures

---

**End of Section 1.6: Authentication and Configuration**

**End of Section 1: Platform Fundamentals**

**You have completed ALL of Section 1!** 🎉

Ready to move to **Section 2: Virtual Appliances**?
