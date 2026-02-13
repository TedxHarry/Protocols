# Section 1.4: Security Administration

**Master security settings, user access levels, and administrative role management in SailPoint Identity Security Cloud**

---

## Table of Contents

- [1.4.1 Security Settings Overview](#141-security-settings-overview)
- [1.4.2 User Access Levels and Permissions](#142-user-access-levels-and-permissions)
- [1.4.3 Admin Role Management](#143-admin-role-management)

---

# 1.4.1 Security Settings Overview

**Learning Objective:** Understand and configure security settings in SailPoint ISC to protect your tenant and data.

---

## What are Security Settings?

Security settings control how users access ISC and what security measures protect your tenant.

**Security settings include:**
- Authentication methods
- Password policies
- Session management
- API security
- Network access controls
- Security notifications

**Why security settings matter:**
- Protect identity data
- Prevent unauthorized access
- Meet compliance requirements
- Enforce security policies
- Audit user activities

---

## Accessing Security Settings

### **Navigation**

**Method 1: Admin Menu**
1. Log into ISC as administrator
2. Click **Admin** (top navigation)
3. Select **Security Settings** or **Global Settings**
4. Choose **Security**

**Method 2: Global Menu**
1. Click **Global** (if available in your tenant)
2. Select **Security Settings**
3. View security configuration options

### **What You'll See**

Security settings organized by category:
- Authentication
- Session Management
- Password Policies
- API Management
- Network Security
- Audit Settings

---

## Authentication Settings

### **Understanding Authentication Options**

Authentication controls how users prove their identity to ISC.

**Available authentication methods:**

| Method | Description | Use Case |
|--------|-------------|----------|
| **Internal** | ISC manages passwords | Development, testing |
| **SAML** | Single Sign-On with identity provider | Enterprise production |
| **Multi-Factor (MFA)** | Additional verification required | High security environments |
| **OAuth/OIDC** | Modern authentication protocols | Cloud integrations |

### **Internal Authentication**

**What it is:**
- ISC stores and validates passwords
- Users log in with username/password
- ISC manages password resets

**Configuration:**
```
Authentication Type: Internal
Password Policy: (See Password Policies section)
Self-Service: Enabled/Disabled
Password Reset: Email/Security Questions
```

**When to use:**
- Non-production environments
- Testing/development tenants
- Small deployments without SSO

**When NOT to use:**
- Production environments (use SAML/SSO)
- High-security requirements
- Enterprise deployments

### **SAML Authentication (SSO)**

**What it is:**
- Users authenticate with external identity provider
- Examples: Azure AD, Okta, Ping Identity
- Single Sign-On across applications

**Benefits:**
- Centralized authentication
- No passwords in ISC
- MFA enforcement at IdP
- Better user experience
- Simplified user management

**Basic SAML configuration:**
```
Service Provider (ISC):
  Entity ID: https://yourcompany.identitynow.com
  ACS URL: https://yourcompany.login.sailpoint.com/saml/SSO/alias/yourcompany-sp

Identity Provider:
  IdP Entity ID: (from your IdP)
  SSO URL: (from your IdP)
  Certificate: (from your IdP)
```

**Configuration steps:**
1. Navigate to **Security Settings** → **Authentication**
2. Select **External Identity Provider**
3. Choose **SAML 2.0**
4. Configure IdP details
5. Upload IdP certificate
6. Map attributes (email, name)
7. Test configuration
8. Enable for users

### **Multi-Factor Authentication (MFA)**

**What it is:**
- Second verification factor required
- Something you know (password) + something you have (phone)

**MFA options:**
- SMS codes
- Authenticator apps (Google Authenticator, Microsoft Authenticator)
- Hardware tokens
- Email verification

**Configuration:**
```
MFA Settings:
  Enabled: Yes
  Required For: All users / Admins only / Specific groups
  Methods Allowed:
    ☑ Authenticator App
    ☑ SMS
    ☐ Email (less secure)
  
  Remember Device: 30 days
  Bypass for Trusted Networks: Yes/No
```

**Best practices:**
- Require MFA for all administrators
- Consider MFA for all users in production
- Allow multiple MFA methods
- Don't rely on SMS alone (less secure)

---

## Password Policies

### **Password Requirements**

Control password strength and lifecycle.

**Common password policy settings:**
```
Minimum Length: 12 characters
Character Requirements:
  ☑ Uppercase letters (A-Z)
  ☑ Lowercase letters (a-z)
  ☑ Numbers (0-9)
  ☑ Special characters (!@#$%^&*)

Password History: Remember last 12 passwords
Password Age:
  Maximum: 90 days (force change)
  Minimum: 1 day (prevent rapid changes)

Account Lockout:
  Failed Attempts: 5
  Lockout Duration: 30 minutes
  Reset Counter After: 15 minutes
```

### **Strong Password Policy Example**

**Production environment:**
```
Minimum Length: 14 characters
Required Characters:
  - At least 1 uppercase
  - At least 1 lowercase
  - At least 1 number
  - At least 1 special character

History: Last 24 passwords
Max Age: 90 days
Min Age: 1 day

Lockout: 5 attempts, 1 hour lockout
```

### **Password Reset Options**

**Self-service password reset:**
```
Enabled: Yes
Methods:
  - Email verification
  - Security questions (3 required)
  - Manager approval
  
Security Questions:
  Minimum Required: 3
  Must Answer Correctly: 2
```

**Administrator password reset:**
```
Admins Can Reset: Yes
Require Reason: Yes
Send Notification: Yes
Log Action: Yes
```

---

## Session Management

### **Session Settings**

Control how long users stay logged in.

**Session configuration:**
```
Session Timeout:
  Idle Timeout: 30 minutes
  Maximum Session: 12 hours
  
Remember Me:
  Enabled: Yes
  Duration: 30 days
  
Concurrent Sessions:
  Allow Multiple: Yes
  Maximum: 3 per user
```

### **Session Timeout Behavior**

**Idle timeout:**
- User inactive for X minutes
- Warning appears: "Session expiring in 5 minutes"
- User can extend or will be logged out

**Maximum session:**
- Hard limit regardless of activity
- Prevents indefinite sessions
- Forces re-authentication

**Best practices:**
```
Production Environment:
  Idle Timeout: 15-30 minutes
  Maximum Session: 8-12 hours
  Remember Me: 7-30 days

High Security:
  Idle Timeout: 10 minutes
  Maximum Session: 4 hours
  Remember Me: Disabled
```

---

## API Security Settings

### **API Access Control**

**OAuth Client Management:**
```
Location: Admin → Security → API Management → OAuth Clients

Settings:
  Allow OAuth Clients: Yes
  Client Approval: Admin approval required
  Client Expiration: Indefinite / Set date
  
Rate Limits:
  Requests per Minute: 100
  Burst Allowance: 200
```

**Personal Access Tokens (PAT):**
```
Location: Preferences → Personal Access Tokens

Settings:
  Allow PATs: Yes
  Maximum per User: 10
  Default Expiration: 90 days
  Maximum Expiration: 365 days
```

### **API Rate Limiting**

**Rate limit configuration:**
```
Standard Tier:
  Requests per Minute: 100
  Requests per Hour: 5,000
  Requests per Day: 100,000

Actions when exceeded:
  HTTP 429: Too Many Requests
  Retry After: Wait time in seconds
```

**Monitoring API usage:**
```
Location: Admin → Security → API Usage

View:
  - Requests by client
  - Requests by endpoint
  - Rate limit hits
  - Throttling events
```

---

## Network Security

### **IP Allowlisting (Whitelisting)**

Restrict ISC access to specific IP addresses.

**Configuration:**
```
Location: Admin → Security → Network Access

IP Allowlist:
  Enabled: Yes
  
Allowed IPs:
  - 203.0.113.0/24 (Office network)
  - 198.51.100.50 (VPN gateway)
  - 192.0.2.100 (Admin home IP)

Action for Blocked IPs:
  - Deny access
  - Show error message
  - Log attempt
```

**When to use IP allowlisting:**
- High security environments
- Compliance requirements
- Restrict admin access
- Limit API access

**Considerations:**
- Remote workers need VPN
- Dynamic IPs require ranges
- Can lock out legitimate users
- Emergency access procedures needed

### **Trusted Network Configuration**

**Trusted networks bypass certain security:**
```
Trusted Networks:
  - Corporate Office: 203.0.113.0/24
    MFA: Not required
    Remember Device: Extended
    
  - Untrusted/Internet: All others
    MFA: Required
    Remember Device: Limited
```

---

## Security Notifications

### **Security Event Alerts**

Configure notifications for security events.

**Alert types:**
```
Failed Login Attempts:
  Alert After: 5 failed attempts
  Recipients: security@company.com
  
New Admin Created:
  Alert: Immediately
  Recipients: admin-team@company.com, ciso@company.com
  
API Client Created:
  Alert: Immediately
  Recipients: security@company.com
  
Unusual Activity:
  Alert: High number of access requests
  Alert: Bulk account changes
  Recipients: iam-team@company.com
```

**Notification delivery:**
- Email
- Webhook to SIEM
- Integration with monitoring tools

---

## Audit and Compliance Settings

### **Audit Logging**

**What gets audited:**
- User logins
- Admin actions
- Configuration changes
- Access requests
- Provisioning operations
- API calls

**Audit log retention:**
```
Retention Period: 90 days to 7 years (varies by tenant)
Access: Admin → Audit Logs
Export: CSV, JSON formats
```

**Audit log example:**
```
Timestamp: 2024-02-12 14:30:15
User: admin@company.com
Action: Created OAuth Client
Target: "Production API Client"
IP Address: 203.0.113.45
Details: Client ID: abc123...
```

### **Compliance Reports**

**Available reports:**
- User activity reports
- Admin action reports
- Configuration change history
- Access review results

**Generating compliance report:**
1. Navigate to **Audit** → **Reports**
2. Select report type
3. Choose date range
4. Apply filters
5. Generate and download

---

## Data Protection Settings

### **Data Encryption**

**Encryption in ISC:**
```
Data at Rest:
  - Database: AES-256 encryption
  - File storage: Encrypted
  - Backups: Encrypted

Data in Transit:
  - HTTPS/TLS 1.2+ required
  - API calls: Encrypted
  - All connections: SSL/TLS
```

**Customer-controlled encryption:**
- Some ISC features support customer-managed keys
- "Zero knowledge" encryption for sensitive data
- Check with SailPoint for availability

### **Data Residency**

**Tenant location:**
```
Data Centers:
  - US East (Virginia)
  - US West (Oregon)
  - EU (Frankfurt)
  - UK (London)
  - Australia
  - Canada

Setting: Configured at tenant creation
Change: Requires SailPoint support
```

**Compliance considerations:**
- GDPR requirements for EU data
- Data sovereignty requirements
- Choose location at setup

---

## Security Best Practices

### **Essential Security Hardening**

**For Production Tenants:**

✅ **Do:**
1. Enable SAML/SSO authentication
2. Require MFA for all administrators
3. Implement strong password policies
4. Set appropriate session timeouts
5. Enable IP allowlisting for admin access
6. Monitor security audit logs
7. Regularly review admin accounts
8. Use least privilege for API clients
9. Rotate API credentials regularly
10. Enable security notifications

❌ **Don't:**
1. Use internal authentication in production
2. Allow weak passwords
3. Skip MFA for admins
4. Leave sessions indefinite
5. Grant unnecessary admin rights
6. Ignore audit logs
7. Use shared admin accounts
8. Leave unused OAuth clients enabled
9. Share API credentials
10. Disable security notifications

### **Regular Security Reviews**

**Weekly:**
- Review failed login attempts
- Check new admin accounts
- Monitor API usage anomalies

**Monthly:**
- Audit admin account list
- Review OAuth clients
- Check security settings unchanged
- Review access patterns

**Quarterly:**
- Full security audit
- Password policy review
- Session settings review
- Compliance report generation

---

## Common Security Scenarios

### **Scenario 1: Enabling SSO**

**Goal:** Replace internal auth with SAML SSO

**Steps:**
1. Configure identity provider (Azure AD, Okta)
2. Obtain IdP metadata
3. Configure SAML in ISC Security Settings
4. Map user attributes (email, name)
5. Test with pilot group
6. Enable for all users
7. Disable internal authentication

### **Scenario 2: Implementing MFA**

**Goal:** Add MFA for all administrators

**Steps:**
1. Navigate to Security Settings → MFA
2. Enable MFA
3. Select "Administrators Only"
4. Allow multiple methods (app + SMS)
5. Set grace period (7 days to enroll)
6. Communicate to admin team
7. Monitor enrollment
8. Enforce after grace period

### **Scenario 3: IP Allowlisting for Admins**

**Goal:** Restrict admin access to office network

**Steps:**
1. Identify admin access IPs
2. Navigate to Network Security
3. Enable IP allowlist
4. Add office network range
5. Add VPN gateway IP
6. Add emergency access IP
7. Test from allowed/blocked locations
8. Document emergency bypass process

---

## Troubleshooting Security Issues

### **Issue 1: Users Cannot Login**

**Possible causes:**
- SAML misconfiguration
- IdP unavailable
- IP blocked by allowlist
- Account locked
- Session expired

**Troubleshooting:**
1. Check SAML configuration
2. Verify IdP status
3. Check user's IP address
4. Review audit logs
5. Check account lock status
6. Test with different user

### **Issue 2: MFA Not Prompting**

**Possible causes:**
- MFA not enabled for user group
- Device remembered
- Trusted network bypass
- Configuration error

**Troubleshooting:**
1. Verify MFA settings
2. Check user is in enforced group
3. Clear browser cache/cookies
4. Try from untrusted network
5. Check device remember settings

### **Issue 3: API Calls Failing**

**Possible causes:**
- OAuth client disabled
- Insufficient scopes
- Rate limit exceeded
- IP blocked

**Troubleshooting:**
1. Verify OAuth client enabled
2. Check scopes assigned
3. Review rate limit usage
4. Check IP allowlist
5. Review API logs

---

## Quick Reference

### **Security Settings Locations**

| Setting | Navigation Path |
|---------|----------------|
| **Authentication** | Admin → Security → Authentication |
| **Password Policy** | Admin → Security → Password Policy |
| **Session Settings** | Admin → Security → Session Management |
| **MFA** | Admin → Security → Multi-Factor Auth |
| **API Management** | Admin → Security → API Management |
| **Network Security** | Admin → Security → Network Access |
| **Audit Logs** | Admin → Audit Logs |

### **Recommended Security Baseline**
```
Production Environment:

Authentication: SAML/SSO
MFA: Required for admins
Password Policy:
  - 14+ characters
  - Complexity required
  - 90-day expiration
Session:
  - 30-minute idle timeout
  - 12-hour max session
API:
  - Least privilege scopes
  - Rate limiting enabled
Network:
  - IP allowlist for admins (optional)
  - Trusted network defined
Monitoring:
  - Audit logging enabled
  - Security alerts configured
```

---

## Practice Exercises

**Exercise 1:** What authentication method is recommended for production ISC tenants?

<details>
<summary>Answer</summary>

**SAML/SSO** with an external Identity Provider (Azure AD, Okta, etc.)

**Reasons:**
- Centralized authentication
- No passwords stored in ISC
- Better security controls
- MFA enforcement at IdP
- Single Sign-On experience
- Enterprise standard
</details>

---

**Exercise 2:** Should MFA be required for all ISC administrators?

<details>
<summary>Answer</summary>

**Yes, absolutely.**

MFA should be required for all administrator accounts because:
- Admins have elevated privileges
- Compromise would be severe
- Protects against password theft
- Industry best practice
- Compliance requirement
- Minimal user impact
</details>

---

**Exercise 3:** A user reports being automatically logged out after 30 minutes of work. What setting controls this?

<details>
<summary>Answer</summary>

**Session Timeout** settings control this behavior.

Specifically:
- **Idle Timeout** - logs out after inactivity
- **Maximum Session** - hard limit regardless of activity

The user is likely hitting the **Idle Timeout** of 30 minutes.

Solution: Either the user needs to stay active, or the timeout setting needs adjustment.
</details>

---

# 1.4.2 User Access Levels and Permissions

**Learning Objective:** Understand ISC user access levels and how they control what users can do.

---

## What are User Access Levels?

User access levels determine what a user can view and do in ISC.

**Think of access levels as:**
- Predefined permission sets
- Like "roles" for using ISC itself
- Different from access profiles (which are for target systems)

**Why access levels matter:**
- Control who can administer ISC
- Limit access to sensitive data
- Enforce least privilege
- Support role separation

---

## Standard User Access Levels

### **Overview of Access Levels**

SailPoint ISC has several standard access levels:

| Access Level | Description | Typical Users |
|-------------|-------------|---------------|
| **Admin** | Full administrative access | IAM administrators |
| **ORG_ADMIN** | Organization-wide admin | Senior IAM team |
| **HELPDESK** | Limited support functions | IT help desk |
| **CERT_ADMIN** | Certification management | Governance team |
| **REPORT_ADMIN** | Reporting capabilities | Compliance team |
| **ROLE_ADMIN** | Role management | IAM analysts |
| **ROLE_SUBADMIN** | Limited role management | Department admins |
| **SOURCE_ADMIN** | Source management | Integration team |
| **DASHBOARD** | View-only dashboards | Managers, viewers |

**Note:** Available levels may vary by tenant configuration

---

## Admin Access Level

### **What Admins Can Do**

**Full administrative capabilities:**
- Manage all identities
- Configure sources
- Manage access profiles and roles
- Configure workflows
- Manage certifications
- View all reports
- Configure security settings
- Manage other admins
- Access all administrative functions

**Typical responsibilities:**
- Daily ISC administration
- Configuration changes
- Troubleshooting
- User support
- System maintenance

**Who should have Admin access:**
- IAM team members
- Dedicated ISC administrators
- Senior technical staff

**Who should NOT:**
- Regular end users
- Managers (unless required)
- External contractors (usually)
- Temporary staff

---

## ORG_ADMIN Access Level

### **Organization Admin Capabilities**

**Highest level admin:**
- Everything Admin can do, plus:
- Tenant-wide configuration
- Security policy settings
- API management
- Global settings
- Organization structure

**Differences from Admin:**
- Can modify organization-level settings
- Can create/manage admins
- Access to billing/licensing (sometimes)
- Highest privilege level

**Who should have ORG_ADMIN:**
- IAM leadership
- Very limited number (2-3 people)
- Senior administrators only

---

## HELPDESK Access Level

### **Help Desk Capabilities**

**What help desk can do:**
```
✅ Can:
  - Search for users
  - View identity details
  - View account status
  - Unlock accounts
  - Reset passwords
  - View access requests
  - View provisioning status
  - Run pre-defined reports

❌ Cannot:
  - Modify identities
  - Change configurations
  - Approve access requests
  - Launch certifications
  - Manage sources
  - Access admin settings
```

**Typical help desk tasks:**
1. User calls: "I can't access Salesforce"
   - Help desk searches for user
   - Views their Salesforce account
   - Checks provisioning status
   - Unlocks account if needed

2. User calls: "I need my password reset"
   - Help desk finds user
   - Initiates password reset
   - Confirms reset email sent

**Who should have HELPDESK access:**
- IT support staff
- Service desk personnel
- First-level support teams

---

## CERT_ADMIN Access Level

### **Certification Admin Capabilities**

**What cert admins can do:**
```
✅ Can:
  - Create certifications
  - Launch campaigns
  - Monitor campaign progress
  - Configure certification settings
  - View certification results
  - Export certification data
  - Manage reminders

❌ Cannot:
  - Make certification decisions (that's reviewers)
  - Configure sources
  - Manage identities directly
  - Access full admin functions
```

**Typical cert admin tasks:**
- Launch quarterly access reviews
- Monitor completion rates
- Send reminders to reviewers
- Export results for compliance
- Close completed campaigns

**Who should have CERT_ADMIN:**
- Governance team members
- Compliance analysts
- Access review coordinators

---

## REPORT_ADMIN Access Level

### **Report Admin Capabilities**

**What report admins can do:**
```
✅ Can:
  - Create custom reports
  - Schedule reports
  - Export data
  - View all reports
  - Share reports
  - Access analytics

❌ Cannot:
  - Modify identities
  - Configure sources
  - Launch certifications
  - Access admin settings
```

**Typical reporting tasks:**
- Generate monthly compliance reports
- Create executive dashboards
- Export user lists
- Analyze access patterns

**Who should have REPORT_ADMIN:**
- Compliance team
- Audit staff
- Business analysts
- Governance team

---

## ROLE_ADMIN and ROLE_SUBADMIN

### **Role Admin Capabilities**

**ROLE_ADMIN can:**
```
✅ Full role management:
  - Create roles
  - Modify roles
  - Delete roles
  - Assign roles
  - Configure role criteria
  - View role analytics
```

**ROLE_SUBADMIN can:**
```
✅ Limited role management:
  - Manage specific roles only
  - View role assignments
  - Request role changes
  
❌ Cannot:
  - Create new roles
  - Delete roles
  - Modify all roles
```

**Use case for ROLE_SUBADMIN:**
- Department managers managing department-specific roles
- Application owners managing app-specific roles
- Delegated role administration

**Who should have these:**
- ROLE_ADMIN: IAM team, role designers
- ROLE_SUBADMIN: Department admins, app owners

---

## SOURCE_ADMIN Access Level

### **Source Admin Capabilities**

**What source admins can do:**
```
✅ Can:
  - Manage sources
  - Configure source settings
  - Trigger aggregations
  - View source logs
  - Test connections
  - Configure correlation rules
  - Manage provisioning policies

❌ Cannot:
  - Configure global settings
  - Manage identities directly
  - Access security settings
```

**Typical source admin tasks:**
- Add new source connectors
- Configure Active Directory connection
- Troubleshoot aggregation failures
- Update source schemas
- Test provisioning

**Who should have SOURCE_ADMIN:**
- Integration specialists
- Connector administrators
- Technical IAM staff

---

## DASHBOARD Access Level

### **Dashboard/Viewer Capabilities**

**What dashboard users can do:**
```
✅ Can:
  - View dashboards
  - See reports
  - View read-only data
  - Export visible data

❌ Cannot:
  - Make any changes
  - Access admin functions
  - Modify configurations
  - Approve requests
```

**Who should have DASHBOARD:**
- Executives
- Managers
- Stakeholders
- Auditors (view-only)
- Business users needing visibility

---

## Assigning User Access Levels

### **How to Assign Access Levels**

**Method 1: Individual Assignment**

**Steps:**
1. Navigate to **Admin** → **Users** or **Identities**
2. Search for the user
3. Click on user profile
4. Go to **Access** or **Admin Roles** tab
5. Click **Add Access Level**
6. Select access level (e.g., HELPDESK)
7. Click **Save**

**Method 2: Governance Groups**

**Steps:**
1. Create governance group
2. Add users to group
3. Assign access level to group
4. All group members inherit access

**Example governance groups:**
```
Group: IAM Administrators
Access Level: Admin
Members: alice@company.com, bob@company.com

Group: IT Help Desk
Access Level: HELPDESK
Members: helpdesk team members

Group: Compliance Team
Access Level: CERT_ADMIN, REPORT_ADMIN
Members: compliance team members
```

---

## Access Level Best Practices

### **Principle of Least Privilege**

**Golden rule:** Give users the MINIMUM access needed to do their job.

**Examples:**

✅ **Good:**
```
Service desk agent needs to unlock accounts:
  → Assign HELPDESK level
  
Compliance analyst needs to run reports:
  → Assign REPORT_ADMIN level
  
Manager needs to view team's access:
  → Assign DASHBOARD level
```

❌ **Bad:**
```
Service desk agent needs to unlock accounts:
  → DON'T assign Admin level
  
Compliance analyst needs to run reports:
  → DON'T assign full Admin
  
Manager curious about ISC:
  → DON'T assign any admin access
```

### **Regular Access Reviews**

**Review admin access:**

**Monthly:**
- List all users with admin access
- Verify each still needs access
- Check for terminated employees
- Remove unnecessary access

**Example review:**
```
User: john.smith@company.com
Access Level: Admin
Last Login: 2024-01-15 (30 days ago)
Status: Still employed
Job Role: IAM Administrator
Keep Access: ✅ Yes

User: jane.doe@company.com
Access Level: HELPDESK
Last Login: Never
Status: Changed departments
Keep Access: ❌ No - Remove access
```

### **Separation of Duties**

**Don't give all access to one person:**

❌ **Bad:**
```
Bob has:
  - Admin access
  - Approver for all access requests
  - Certification reviewer
  - Sole person with all access
```

✅ **Good:**
```
Alice: Admin access
Bob: Certification admin
Carol: Access request approver
Derek: Backup admin

No single person controls everything
```

---

## Viewing User Permissions

### **Checking What Access a User Has**

**Steps:**
1. Navigate to **Admin** → **Users**
2. Search for user
3. Click user profile
4. View **Access Levels** section

**You'll see:**
```
User: alice@company.com

Access Levels:
  - Admin
  - CERT_ADMIN

Capabilities:
  - Full administrative access
  - Certification management
  
Last Login: 2024-02-12 09:15:23
```

### **Checking Your Own Access**

**Steps:**
1. Click your profile (top right)
2. Select **Preferences** or **My Access**
3. View your access levels

**What you'll see:**
```
Your Access Levels:
  - HELPDESK

You Can:
  ✅ Search identities
  ✅ View account status
  ✅ Reset passwords
  ✅ Unlock accounts
  ✅ View reports

You Cannot:
  ❌ Modify configurations
  ❌ Approve access requests
  ❌ Launch certifications
```

---

## Delegated Administration

### **What is Delegated Administration?**

Delegated administration lets you give limited admin rights to specific people for specific areas.

**Examples:**
- HR admin manages HR department identities
- App owner manages their application's roles
- Department manager reviews their team's access

### **Setting Up Delegated Admin**

**Scenario:** Let HR manager manage HR employees only

**Configuration:**
```
1. Create governance group: "HR Identities"
   Add all HR employees to group

2. Create admin group: "HR Administrators"
   Add HR managers

3. Grant HR Administrators:
   Access Level: ROLE_SUBADMIN
   Scope: HR Identities group only
   
4. HR managers can now:
   ✅ Manage HR employees
   ❌ Cannot manage other departments
```

---

## Access Level Combinations

### **Multiple Access Levels**

Users can have multiple access levels for different functions.

**Example combinations:**

**Combination 1: Governance Lead**
```
Access Levels:
  - CERT_ADMIN (manage certifications)
  - REPORT_ADMIN (generate compliance reports)
  - ROLE_ADMIN (manage roles)

Purpose: Complete governance management
```

**Combination 2: Technical Admin**
```
Access Levels:
  - Admin (general administration)
  - SOURCE_ADMIN (manage integrations)

Purpose: Full technical administration
```

**Combination 3: Support + Reporting**
```
Access Levels:
  - HELPDESK (user support)
  - REPORT_ADMIN (generate tickets reports)

Purpose: Support with analytics
```

---

## Removing Access Levels

### **When to Remove Access**

**Immediate removal:**
- Employee terminated
- Changed roles/departments
- Security incident
- Access no longer needed

**How to remove:**
1. Navigate to user profile
2. Go to Access Levels
3. Select access level
4. Click **Remove** or **Revoke**
5. Confirm removal
6. Document reason

**Effects of removal:**
- Immediate loss of permissions
- User logged out if currently logged in
- Cannot perform admin functions
- May retain regular user access

---

## Quick Reference

### **Access Levels Summary**

| Level | Primary Use | Can Modify Configs | Can View Data | Can Approve |
|-------|-------------|-------------------|---------------|-------------|
| **Admin** | Full administration | ✅ Yes | ✅ Yes | ✅ Yes |
| **ORG_ADMIN** | Org-wide admin | ✅ Yes | ✅ Yes | ✅ Yes |
| **HELPDESK** | User support | ❌ No | ✅ Limited | ❌ No |
| **CERT_ADMIN** | Certifications | ✅ Certs only | ✅ Yes | ❌ No |
| **REPORT_ADMIN** | Reporting | ❌ No | ✅ Yes | ❌ No |
| **ROLE_ADMIN** | Role management | ✅ Roles only | ✅ Yes | ❌ No |
| **SOURCE_ADMIN** | Sources | ✅ Sources only | ✅ Limited | ❌ No |
| **DASHBOARD** | View only | ❌ No | ✅ Limited | ❌ No |

### **Common Access Level Assignments**
```
IAM Administrator → Admin
IAM Manager → ORG_ADMIN
Help Desk → HELPDESK
Compliance Analyst → CERT_ADMIN, REPORT_ADMIN
Integration Specialist → SOURCE_ADMIN
Department Manager → DASHBOARD
Application Owner → ROLE_SUBADMIN
Executive → DASHBOARD
```

---

## Practice Exercises

**Exercise 4:** An IT help desk agent needs to unlock user accounts. What access level should they have?

<details>
<summary>Answer</summary>

**HELPDESK**

This access level provides:
- Search for users
- View account status
- Unlock accounts
- Reset passwords
- View provisioning status

**NOT Admin** - that would give too much access (violates least privilege)
</details>

---

**Exercise 5:** Your compliance team needs to create and manage access certifications. What access level do they need?

<details>
<summary>Answer</summary>

**CERT_ADMIN**

This access level provides:
- Create certifications
- Launch campaigns
- Monitor progress
- Configure settings
- View results
- Export data

They do NOT need full Admin access for this function.
</details>

---

**Exercise 6:** Can a user with HELPDESK access approve access requests?

<details>
<summary>Answer</summary>

**No**

HELPDESK access is read-only for most functions. It allows:
- ✅ Viewing access requests
- ❌ Approving access requests

Approving access requests requires:
- Being designated as an approver in the approval workflow
- OR having Admin level access
</details>

---

# 1.4.3 Admin Role Management

**Learning Objective:** Understand how to manage administrative roles and responsibilities in ISC.

---

## What is Admin Role Management?

Admin role management controls who can perform administrative functions in ISC.

**Key concepts:**
- Governance groups for admins
- Role-based administration
- Workgroup assignments
- Admin role lifecycle

**Why it matters:**
- Ensure proper oversight
- Enable team collaboration
- Support role separation
- Maintain security

---

## Governance Groups

### **What are Governance Groups?**

Governance groups are collections of users who share administrative responsibilities.

**Think of governance groups as:**
- Teams within your IAM organization
- Groups of admins with similar functions
- Way to organize administrative access

**Common governance groups:**
```
IAM Administrators
  - Full admin team
  - Daily operations
  
IAM Leadership
  - ORG_ADMIN level
  - Strategic decisions
  
Help Desk Team
  - HELPDESK level
  - User support
  
Compliance Team
  - CERT_ADMIN, REPORT_ADMIN
  - Governance functions
  
Integration Team
  - SOURCE_ADMIN
  - Technical integrations
```

### **Creating Governance Groups**

**Steps:**
1. Navigate to **Admin** → **Governance Groups**
2. Click **Create Group** or **New Group**
3. Enter group details

**Configuration:**
```
Group Name: IAM Administrators
Description: Full administrators for daily ISC operations

Members:
  - alice@company.com
  - bob@company.com
  - carol@company.com

Access Levels:
  - Admin

Owner: IAM Manager (manager@company.com)
```

**Step 4:** Save group

**Step 5:** All members now have Admin access

---

## Workgroups

### **What are Workgroups?**

Workgroups define ownership and responsibility for specific ISC objects.

**Workgroups control:**
- Who manages specific sources
- Who owns access profiles
- Who handles certifications
- Approval routing

**Example workgroups:**
```
Active Directory Team
  - Owns: AD source
  - Manages: AD provisioning policies
  - Members: AD admins

Salesforce Team
  - Owns: Salesforce source
  - Manages: SF access profiles
  - Members: SF admins

HR Systems Team
  - Owns: Workday source
  - Manages: HR-related roles
  - Members: HR system admins
```

### **Creating Workgroups**

**Steps:**
1. Navigate to **Admin** → **Workgroups**
2. Click **Create Workgroup**
3. Configure workgroup

**Configuration:**
```
Workgroup Name: Active Directory Administration
Description: Team responsible for AD integration

Members:
  - ad-admin1@company.com
  - ad-admin2@company.com

Responsibilities:
  - Manage AD source
  - Handle AD provisioning issues
  - Approve AD access requests

Email Alias: ad-team@company.com
```

### **Assigning Workgroups to Objects**

**Objects that can have workgroups:**
- Sources
- Access profiles
- Roles
- Certifications

**Example assignment:**
```
Object: Active Directory Source
Workgroup: Active Directory Administration

Result:
  - AD team gets notifications
  - AD team manages source
  - Requests route to AD team
```

---

## Admin Ownership and Responsibilities

### **Defining Admin Roles**

**Typical admin roles in an IAM team:**

**1. IAM Administrator**
```
Responsibilities:
  - Daily ISC operations
  - User support
  - Troubleshoot provisioning
  - Manage access requests
  - Monitor system health

Access Level: Admin
Workgroups: All systems
```

**2. IAM Architect**
```
Responsibilities:
  - Design identity solutions
  - Configure complex workflows
  - Define role models
  - Integration architecture
  - Strategic planning

Access Level: Admin, ORG_ADMIN
Workgroups: All systems
```

**3. Integration Specialist**
```
Responsibilities:
  - Configure source connectors
  - Troubleshoot integrations
  - Manage Virtual Appliances
  - API integrations
  - Performance tuning

Access Level: SOURCE_ADMIN, Admin
Workgroups: Technical systems
```

**4. Governance Analyst**
```
Responsibilities:
  - Manage certifications
  - Generate compliance reports
  - Analyze access patterns
  - Role mining
  - Audit support

Access Level: CERT_ADMIN, REPORT_ADMIN
Workgroups: Governance
```

**5. Help Desk Agent**
```
Responsibilities:
  - Unlock accounts
  - Reset passwords
  - Answer user questions
  - Ticket resolution
  - Basic troubleshooting

Access Level: HELPDESK
Workgroups: User Support
```

---

## Admin Assignment Methods

### **Method 1: Direct Assignment**

Assign access level directly to individual user.

**When to use:**
- Small team
- Unique requirements
- Temporary access
- Testing

**Steps:**
1. Find user profile
2. Add access level
3. Save

**Example:**
```
User: temp-admin@company.com
Access: Admin
Duration: 30 days (temporary)
Reason: Covering for vacation
```

### **Method 2: Governance Group Assignment**

Assign access via group membership.

**When to use:**
- Standard team structure
- Consistent permissions
- Easy management
- Production environments

**Steps:**
1. Create/update governance group
2. Add users to group
3. Group has access level
4. Users inherit access

**Example:**
```
Group: IAM Administrators
Access: Admin
Members: alice, bob, carol (all get Admin)

Adding derek:
  - Add derek to group
  - Automatically gets Admin
```

### **Method 3: Role-Based Assignment**

Assign based on job role/title.

**When to use:**
- Large organizations
- Automated onboarding
- HR integration
- Role-based access model

**Configuration:**
```
Rule: If job title = "IAM Administrator"
Then: Add to "IAM Administrators" group
Then: Grants Admin access

Rule: If department = "Help Desk"
Then: Add to "Help Desk Team" group
Then: Grants HELPDESK access
```

---

## Admin Lifecycle Management

### **Onboarding New Admin**

**Steps:**
1. **Verify need**
   - Manager approval
   - Business justification
   - Access level determination

2. **Create account** (if new to ISC)
   - Standard user identity
   - Email verified

3. **Assign access**
   - Add to governance group OR
   - Direct access level assignment

4. **Training**
   - ISC basics training
   - Admin function training
   - Security awareness
   - Documentation provided

5. **Verify access**
   - Admin logs in
   - Tests functions
   - Confirms access working

**Example onboarding checklist:**
```
☐ Manager approval received
☐ User account exists
☐ Added to "IAM Administrators" group
☐ Admin access confirmed
☐ Training completed
☐ Documentation provided
☐ Emergency contact info recorded
☐ Onboarding documented in ticket
```

### **Admin Role Changes**

**When roles change:**

**Scenario 1: Promotion**
```
Before: Help Desk (HELPDESK access)
After: IAM Administrator (Admin access)

Actions:
  1. Remove from "Help Desk Team" group
  2. Add to "IAM Administrators" group
  3. Verify new access working
  4. Update training records
```

**Scenario 2: Department Transfer**
```
Before: IAM Administrator
After: Security Analyst (different team)

Actions:
  1. Remove all admin access levels
  2. Remove from governance groups
  3. Grant appropriate security analyst access
  4. Update documentation
```

### **Offboarding Admin**

**When admin leaves:**

**Immediate actions:**
1. Remove all admin access
2. Remove from governance groups
3. Revoke API tokens/OAuth clients
4. Document removal
5. Notify team

**Follow-up actions:**
1. Review recent changes made
2. Update documentation
3. Reassign responsibilities
4. Archive account (don't delete)

**Example offboarding:**
```
User: departed-admin@company.com
Action: Terminated employment
Date: 2024-02-12

Steps Taken:
  ☑ Removed from "IAM Administrators" group
  ☑ Revoked all access levels
  ☑ Deleted 2 OAuth clients
  ☑ Deleted 1 Personal Access Token
  ☑ Documented in ticket #12345
  ☑ Team notified via email
  ☑ Responsibilities reassigned to bob@company.com
```

---

## Admin Access Auditing

### **Regular Admin Audits**

**What to audit:**
- Who has admin access
- When last accessed
- What they've done recently
- Whether access still needed

**Monthly audit process:**

**Step 1: Export admin list**
```
Navigate to: Admin → Users
Filter: Access Level = Admin
Export: CSV

Result: List of all admins with details
```

**Step 2: Review each admin**
```
For each admin:
  ✓ Still employed?
  ✓ Still in IAM role?
  ✓ Last login recent?
  ✓ Access still needed?
  ✓ Appropriate access level?
```

**Step 3: Document findings**
```
Audit Date: 2024-02-12
Admins Reviewed: 12
Access Removed: 2 (terminated, transferred)
Access Modified: 1 (reduced from Admin to HELPDESK)
Access Verified: 9 (all appropriate)

Next Audit: 2024-03-12
```

**Step 4: Take action**
- Remove unnecessary access
- Update access levels
- Document changes

### **Audit Report Example**
```
ISC Admin Access Audit
Date: February 12, 2024

Total Admins: 12

By Access Level:
  ORG_ADMIN: 2
  Admin: 6
  CERT_ADMIN: 2
  HELPDESK: 4
  SOURCE_ADMIN: 3
  (some users have multiple levels)

Activity Summary:
  Active (logged in last 30 days): 11
  Inactive: 1

Actions Taken:
  ✓ Removed access: jane.doe (terminated)
  ✓ Modified access: john.smith (transferred)
  ✓ Verified appropriate: 10 remaining

Issues Found:
  ⚠ 1 admin not logged in for 60 days
  → Following up with manager

Next Audit: March 12, 2024
Auditor: alice@company.com
```

---

## Emergency Admin Access

### **Break-Glass Accounts**

**What are break-glass accounts?**
- Emergency admin accounts
- Used only in emergencies
- Highly monitored
- Strong security controls

**When to use:**
- Primary admins unavailable
- Critical system issue
- All other admins locked out
- Emergency changes needed

**Break-glass configuration:**
```
Account: emergency-admin@company.com
Access Level: ORG_ADMIN
Password: Complex, in sealed envelope
MFA: Required
Monitoring: All actions logged and alerted

Usage Process:
  1. Confirm emergency exists
  2. Get approval from leadership
  3. Use account for emergency only
  4. Document all actions
  5. Report usage immediately
  6. Reset password after use
```

**Security controls:**
```
✓ Account monitored 24/7
✓ Any login triggers alert
✓ All actions audited
✓ Password changed after each use
✓ Usage must be justified
✓ Regular testing (quarterly)
```

---

## Admin Collaboration

### **Team Coordination**

**For effective admin team:**

**Communication:**
- Regular team meetings
- Shared documentation
- Incident escalation process
- On-call rotation

**Documentation:**
- Runbooks for common tasks
- Configuration standards
- Troubleshooting guides
- Contact lists

**Change management:**
- Document all changes
- Peer review for major changes
- Testing in non-prod first
- Communication before production changes

### **Admin Workload Distribution**

**Balancing responsibilities:**
```
Primary Admins:
  Alice: AD/LDAP sources, provisioning
  Bob: Cloud apps (Salesforce, O365), API integrations
  Carol: Certifications, compliance reporting

Backup Coverage:
  Alice covers Bob (on vacation/sick)
  Bob covers Carol
  Carol covers Alice

Specializations:
  Derek: Workflows, complex transforms
  Emily: Reporting, analytics
```

---

## Quick Reference

### **Admin Management Checklist**

**New Admin:**
```
☐ Business justification documented
☐ Manager approval obtained
☐ Appropriate access level determined
☐ Added to governance group
☐ Access verified
☐ Training completed
☐ Emergency contacts recorded
```

**Admin Audit:**
```
☐ Export current admin list
☐ Verify each admin still needs access
☐ Check last login dates
☐ Remove/modify inappropriate access
☐ Document audit results
☐ Schedule next audit
```

**Admin Offboarding:**
```
☐ Remove from governance groups
☐ Revoke all access levels
☐ Delete OAuth clients
☐ Delete Personal Access Tokens
☐ Document removal
☐ Notify team
☐ Reassign responsibilities
```

### **Common Admin Scenarios**

| Scenario | Access Level | Method |
|----------|-------------|---------|
| **Daily IAM ops** | Admin | Governance group |
| **User support** | HELPDESK | Governance group |
| **Compliance work** | CERT_ADMIN, REPORT_ADMIN | Governance group |
| **Integration work** | SOURCE_ADMIN | Governance group |
| **Temporary coverage** | Admin | Direct assignment |
| **Emergency access** | ORG_ADMIN | Break-glass account |

---

## Practice Exercises

**Exercise 7:** Your IAM team has 5 administrators. What's the best way to give them all Admin access?

<details>
<summary>Answer</summary>

**Create a Governance Group**

Steps:
1. Create group: "IAM Administrators"
2. Add all 5 team members to group
3. Assign Admin access to group
4. All members inherit Admin access

**Benefits:**
- Easy to manage (one group vs 5 individual assignments)
- Easy to add/remove members
- Consistent access for team
- Clear documentation of who has access
- Follows best practice
</details>

---

**Exercise 8:** An admin is leaving the company today. What should you do immediately?

<details>
<summary>Answer</summary>

**Immediate Actions:**

1. **Remove admin access**
   - Remove from all governance groups
   - Revoke all access levels

2. **Revoke credentials**
   - Delete any OAuth clients they created
   - Delete any Personal Access Tokens

3. **Document**
   - Record removal in ticket
   - Note what access was removed

4. **Notify**
   - Alert admin team
   - Reassign their responsibilities

5. **Review**
   - Check recent changes they made
   - Verify no suspicious activity

**Do NOT:**
- Delete the account (disable instead for audit trail)
- Wait until "later" to remove access
- Forget to document
</details>

---

**Exercise 9:** You need to audit who has admin access. Where do you look and what do you check?

<details>
<summary>Answer</summary>

**Where to Look:**
1. **Admin** → **Users** or **Identities**
2. Filter by access level (Admin, ORG_ADMIN, etc.)
3. Export list to CSV

**What to Check:**
- ✓ Is user still employed?
- ✓ Is user still in IAM role?
- ✓ When did they last log in?
- ✓ Do they still need this access level?
- ✓ Is the access level appropriate for their role?

**Also Check:**
- **Admin** → **Governance Groups**
- Review group memberships
- Verify group purposes still valid

**Document:**
- Date of audit
- Number of admins reviewed
- Actions taken (access removed/modified)
- Issues found
- Next audit date
</details>

---

**Exercise 10:** Should you give a new help desk agent full Admin access so they can "learn the system"?

<details>
<summary>Answer</summary>

**No, absolutely not.**

**Instead:**

Give them **HELPDESK** access because:
- ✅ Follows least privilege principle
- ✅ They can learn with appropriate access
- ✅ Can perform their job duties
- ✅ Can't accidentally break things
- ✅ Lower security risk

**Admin access should only be given to:**
- Dedicated IAM administrators
- Staff with specific admin responsibilities
- After proper training
- With business justification

**For learning:**
- Use HELPDESK in production
- Use Admin in non-prod/sandbox tenant
- Provide training documentation
- Shadow experienced admins
</details>

---

## Section Summary

You've learned:

✅ **Security Settings** - Authentication, MFA, passwords, API security  
✅ **User Access Levels** - Admin, HELPDESK, CERT_ADMIN, and others  
✅ **Admin Management** - Governance groups, workgroups, lifecycle  
✅ **Best Practices** - Least privilege, auditing, separation of duties  

**Key Takeaways:**
- Use SAML/SSO in production
- Require MFA for administrators
- Apply least privilege principle
- Use governance groups for team management
- Audit admin access regularly
- Document everything

**Next Steps:**
- Review your tenant's security settings
- Audit current admin access
- Create governance groups
- Document admin roles and responsibilities

---

## Additional Resources

📚 **Official Documentation:**
- [Security Settings Guide](https://documentation.sailpoint.com/saas/help/common/admin_console.html)
- [User Access Levels](https://documentation.sailpoint.com/saas/help/common/users/user_level.html)
- [Governance Groups](https://documentation.sailpoint.com/saas/help/common/governance_groups.html)

💡 **Pro Tips:**
- Start with least privilege - add access as needed
- Use governance groups for team-based access
- Audit admin access monthly
- Document why each person has admin access
- Test security settings in non-prod first

---

**End of Section 1.4: Security Administration**

**Next Section:** [1.5 Automation and Workflows →](#15-automation-and-workflows)
