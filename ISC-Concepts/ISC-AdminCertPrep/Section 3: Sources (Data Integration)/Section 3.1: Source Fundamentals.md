# Section 3.1: Source Fundamentals

**Understand what sources are, their types, and connector categories in SailPoint ISC**

---

## Table of Contents

- [3.1.1 Purpose of Sources in ISC](#311-purpose-of-sources-in-isc)
- [3.1.2 Source Types and Connectors](#312-source-types-and-connectors)
- [3.1.3 Generic Connector Categories](#313-generic-connector-categories)

---

# 3.1.1 Purpose of Sources in ISC

**Learning Objective:** Understand what sources are and why they're central to ISC functionality.

---

## What is a Source?

**Source** = Any system that provides identity or access data to ISC

**Think of a source as:**
- A data feed into ISC
- Where ISC gets identity information
- System ISC manages (reads from and/or writes to)

**Simple concept:**
```
Sources → Provide Data → ISC → Creates/Updates Identities
```

**Example:**
```
Workday (Source) → Employee data → ISC → Creates identity for John Smith
Active Directory (Source) → Account data → ISC → Links AD account to John Smith
```

**Exam Tip:** Sources are how data gets INTO ISC. Without sources, ISC has no data.

---

## Why Sources Matter

### **Core Purpose of Sources**

**1. Identity Data Provider**
```
Source provides:
  - Employee information (name, email, department, manager)
  - Contractor information
  - Customer/partner information
  
ISC uses this to:
  - Create identities
  - Update identity attributes
  - Determine lifecycle state (active/inactive)
```

**Example:**
```
Workday Source →
  First Name: John
  Last Name: Smith
  Email: john.smith@company.com
  Department: IT
  Manager: Jane Doe
  Status: Active
  
→ ISC creates/updates John Smith identity with this data
```

**2. Account Information Provider**
```
Source provides:
  - What accounts exist in the system
  - Which identity owns each account
  - Account attributes (enabled, groups, permissions)
  
ISC uses this to:
  - Correlate accounts to identities
  - Track what access users have
  - Detect orphaned accounts
```

**Example:**
```
Active Directory Source →
  Account: jsmith
  Groups: IT-All, VPN-Users, Email-Users
  Status: Enabled
  Last Login: 2024-02-10
  
→ ISC links this account to John Smith identity
→ ISC knows John has VPN and Email access
```

**3. Provisioning Target**
```
Source receives:
  - Create account commands
  - Modify account commands
  - Delete/disable commands
  
ISC sends these when:
  - New hire needs accounts
  - User requests access
  - Employee terminated
```

**Example:**
```
New Hire: John Smith

ISC → Active Directory Source: "Create account jsmith"
ISC → Email Source: "Create mailbox john.smith@company.com"
ISC → VPN Source: "Grant VPN access to jsmith"
```

---

## Sources in ISC Architecture

### **Data Flow Overview**
```
┌─────────────────────────────────────────────┐
│         Identity Sources                    │
│  (Authoritative - who people are)           │
│                                             │
│  Workday, HR Database, Azure AD             │
└──────────────┬──────────────────────────────┘
               │ Aggregation
               ↓
┌─────────────────────────────────────────────┐
│         SailPoint ISC                       │
│                                             │
│  ┌─────────────────────────────────────┐   │
│  │      Identity Records               │   │
│  │  - John Smith                       │   │
│  │  - Jane Doe                         │   │
│  │  - ...                              │   │
│  └─────────────────────────────────────┘   │
└──────────┬──────────────────┬───────────────┘
           │                  │
           │ Aggregation      │ Provisioning
           ↓                  ↓
┌──────────────────┐  ┌──────────────────┐
│  Account Sources │  │  Access Sources  │
│ (What users have)│  │ (What to grant)  │
│                  │  │                  │
│  Active Directory│  │  Salesforce      │
│  Email System    │  │  VPN             │
│  Database        │  │  Applications    │
└──────────────────┘  └──────────────────┘
```

**Key Concept:**
- **Identity Sources**: Where ISC gets people data (who exists)
- **Account Sources**: Where ISC manages accounts (what people have)
- Same system can be both (e.g., Workday provides identities AND has accounts)

---

## Source Operations

### **Aggregation (Reading Data)**

**What happens:**
```
1. ISC tells source: "Give me all your data"
2. Source returns: Users, accounts, groups, permissions
3. ISC processes: Updates identity records
4. ISC correlates: Links accounts to identities
```

**Example - AD Aggregation:**
```
ISC → Active Directory: "Get all users"
AD → ISC: Returns 10,000 user accounts

ISC processes each account:
  - jsmith → Correlates to John Smith identity
  - jdoe → Correlates to Jane Doe identity
  - orphan123 → No match, flagged as uncorrelated
```

**Frequency:**
- Scheduled (daily, hourly)
- Manual (on-demand)
- Real-time (some cloud sources)

### **Provisioning (Writing Data)**

**What happens:**
```
1. ISC determines: User needs account/access
2. ISC sends command: "Create account for John Smith"
3. Source executes: Creates account
4. Source confirms: "Account created: jsmith"
5. ISC updates: Links account to identity
```

**Example - New Hire:**
```
New Employee: John Smith hired

ISC → AD Source: "Create account jsmith"
AD → ISC: "Created: CN=jsmith,OU=Users,DC=company,DC=com"

ISC → Email Source: "Create mailbox john.smith@company.com"
Email → ISC: "Created mailbox"

ISC → VPN Source: "Grant VPN access to jsmith"
VPN → ISC: "Access granted"
```

**Exam Tip:** Aggregation = READ data from source. Provisioning = WRITE data to source.

---

## Authoritative vs Non-Authoritative Sources

### **Authoritative Source**

**Definition:** The "source of truth" for specific attributes

**Characteristics:**
- ISC trusts this source's data
- Updates from this source overwrite ISC data
- Usually HR system for employee data

**Example:**
```
Workday is authoritative for:
  - First Name
  - Last Name
  - Employee ID
  - Department
  - Manager
  - Hire Date
  
When Workday says: "John Smith's department = HR"
ISC updates John's department to HR
Even if ISC previously had department = IT
```

### **Non-Authoritative Source**

**Definition:** Provides data but isn't the source of truth

**Characteristics:**
- ISC reads accounts/access information
- Doesn't update core identity attributes
- Usually application sources

**Example:**
```
Active Directory is non-authoritative for identity attributes:
  - Doesn't change person's name in ISC
  - Doesn't change department in ISC
  - Just provides account information
  
AD tells ISC:
  - Account jsmith exists
  - Account has these groups
  - Account enabled/disabled
```

**Typical Setup:**
```
Authoritative:
  - Workday (employee data)
  - HR Database (employee data)
  - Azure AD (for cloud-first companies)

Non-Authoritative:
  - Active Directory (accounts)
  - Applications (access)
  - Databases (accounts)
```

**Exam Tip:** HR systems are typically authoritative. Application sources are typically non-authoritative.

---

## Source Lifecycle

### **Source States**

**1. Creation**
```
Admin creates source in ISC:
  - Defines connection info
  - Sets up credentials
  - Configures schema mapping
  
Status: DRAFT or PENDING
```

**2. Testing**
```
Admin tests source:
  - Test connection
  - Test aggregation
  - Verify data looks correct
  
Status: TESTING
```

**3. Production**
```
Source live and operational:
  - Scheduled aggregations running
  - Provisioning enabled
  - Integrated with workflows
  
Status: ACTIVE
```

**4. Maintenance**
```
Periodic updates:
  - Credentials rotated
  - Schema updated
  - Configuration changes
  
Status: ACTIVE (with updates)
```

**5. Decommission**
```
Source no longer needed:
  - Disabled
  - Archived
  - Eventually deleted
  
Status: DISABLED → ARCHIVED → DELETED
```

---

## Practice Exercise

**Question:** Your company uses Workday for HR data and Active Directory for accounts. Which should be authoritative and why?

<details>
<summary>Answer</summary>

**Workday should be authoritative for identity attributes.**

**Reasoning:**

**Workday (Authoritative):**
- ✅ Source of truth for employee data
- ✅ HR updates employee info here first
- ✅ Contains: name, department, manager, hire date, employee ID
- ✅ Most accurate and current employee information

**Why Workday is authoritative:**
```
When employee changes department:
  1. HR updates Workday: Department = Marketing
  2. Workday aggregates to ISC
  3. ISC updates identity: Department = Marketing
  4. This triggers workflows (move workflows, access changes)
```

**Active Directory (Non-Authoritative):**
- ❌ NOT authoritative for identity attributes
- ✅ Provides account information only
- ✅ Contains: account status, group memberships, last login

**Why AD is non-authoritative:**
```
AD might have stale department info:
  - AD shows: Department = IT
  - Workday shows: Department = Marketing (current)
  - ISC trusts Workday, not AD
  
If AD updated identity attributes:
  - Would override correct Workday data
  - Would cause data inconsistency
  - Wrong source of truth
```

**Correct Configuration:**
```
Identity Profile Configuration:

Authoritative Source: Workday
  Maps to identity attributes:
    - firstName
    - lastName
    - email
    - department
    - manager
    - employeeId

Non-Authoritative Sources: Active Directory, Salesforce, etc.
  Provide accounts only:
    - Account names
    - Account status
    - Group memberships
    - Access rights
```

**Exam Tip:** HR systems = authoritative for WHO people are. Applications = non-authoritative, provide WHAT people have.
</details>

---

# 3.1.2 Source Types and Connectors

**Learning Objective:** Understand different source types and their connector mechanisms.

---

## Source Categories

### **By Location**

| Category | Description | Connection | Examples |
|----------|-------------|------------|----------|
| **On-Premises** | Behind firewall | Via Virtual Appliance | Active Directory, Oracle DB, LDAP |
| **Cloud/SaaS** | Internet-accessible | Direct from ISC | Workday, Salesforce, ServiceNow |
| **Hybrid** | Partially on-prem | Mixed connectivity | Azure AD, Office 365 |

### **By Purpose**

| Category | Purpose | Common Types | Example |
|----------|---------|--------------|---------|
| **HR Systems** | Identity data (authoritative) | HRIS, HCM | Workday, SuccessFactors, Oracle HCM |
| **Directories** | User accounts | LDAP, AD | Active Directory, OpenLDAP |
| **Applications** | App-specific access | SaaS, On-prem | Salesforce, SAP, ServiceNow |
| **Databases** | Custom data | JDBC | Oracle, SQL Server, MySQL |
| **Files** | File-based feeds | CSV, XML | HR exports, custom feeds |

**Exam Tip:** Know that on-premises sources need VA, cloud sources connect directly.

---

## Connector Types

### **Direct Cloud Connectors (SaaS)**

**How they work:**
```
ISC (Cloud) →→→ Direct HTTPS →→→ SaaS Application

No VA needed!
```

**Characteristics:**
- Pre-built by SailPoint
- Maintained by SailPoint
- No VA required
- OAuth or API key authentication
- Automatic updates

**Common SaaS Connectors:**

| Application | Authentication | Features |
|-------------|----------------|----------|
| **Workday** | OAuth 2.0 | Identity aggregation, provisioning |
| **Salesforce** | OAuth 2.0 | User provisioning, license management |
| **ServiceNow** | OAuth/Basic | User and group management |
| **Azure AD** | OAuth 2.0 | User/group sync, provisioning |
| **Google Workspace** | OAuth 2.0 | User lifecycle, group management |
| **Okta** | API Token | User sync, app assignments |
| **Box** | OAuth 2.0 | User provisioning, collaboration |
| **Slack** | OAuth 2.0 | User and channel management |

**Configuration Example - Salesforce:**
```
1. Create Connected App in Salesforce
2. Get OAuth credentials (Client ID, Secret)
3. In ISC: Create Source → Salesforce
4. Enter credentials
5. Authorize connection
6. Configure schema mapping
7. Test aggregation
```

### **Virtual Appliance Connectors (On-Premises)**

**How they work:**
```
ISC (Cloud) ←→ VA (Your Network) →→→ On-Premises Source

VA required!
```

**Characteristics:**
- Connector runs on VA
- Requires VA installation
- Local network access to source
- Service account credentials

**Common VA Connectors:**

| Source Type | Protocol | Port | Authentication |
|-------------|----------|------|----------------|
| **Active Directory** | LDAP/LDAPS | 389/636 | Service account |
| **LDAP** | LDAP/LDAPS | 389/636 | Bind DN + password |
| **Oracle Database** | JDBC | 1521 | DB user + password |
| **SQL Server** | JDBC | 1433 | SQL auth or Windows auth |
| **MySQL** | JDBC | 3306 | DB user + password |
| **PostgreSQL** | JDBC | 5432 | DB user + password |
| **SAP** | RFC | 3300 | SAP user credentials |
| **RACF (Mainframe)** | Custom | Varies | Mainframe credentials |

**Configuration Example - Active Directory:**
```
1. Deploy Virtual Appliance
2. In ISC: Create Source → Active Directory
3. Assign to VA
4. Configure:
   - Domain controller hostname/IP
   - Port (389 or 636)
   - Service account (svc_sailpoint@domain.com)
   - Base DN (DC=company,DC=com)
   - Search filter
5. Test connection
6. Test aggregation
```

### **Web Services Connectors**

**How they work:**
```
ISC ←→ REST/SOAP API ←→ Application
```

**Characteristics:**
- Application exposes API
- ISC calls API endpoints
- Can be direct or via VA (depends on API location)

**Common Use Cases:**
- Custom applications with APIs
- Modern cloud apps without pre-built connectors
- Microservices architectures

**Example - Generic REST Connector:**
```
GET https://api.company.com/users
Authorization: Bearer <token>

Response:
[
  {
    "userId": "jsmith",
    "firstName": "John",
    "lastName": "Smith",
    "email": "john.smith@company.com",
    "department": "IT"
  },
  ...
]

ISC processes this JSON to create/update identities
```

### **File-Based Connectors**

**How they work:**
```
Source System → Generates file (CSV, XML) → ISC reads file
```

**Characteristics:**
- Source exports data to file
- ISC reads file periodically
- Simple but not real-time
- Good for legacy systems without APIs

**Common Scenarios:**
- Legacy HR systems
- Mainframe exports
- Custom data feeds
- One-way data imports

**Example - CSV Connector:**
```
File: employees.csv
Located: /mnt/hr-exports/employees.csv (on VA)

CSV Content:
EmployeeID,FirstName,LastName,Email,Department
12345,John,Smith,john.smith@company.com,IT
12346,Jane,Doe,jane.doe@company.com,HR

ISC reads this file:
  - Creates identity for John Smith
  - Creates identity for Jane Doe
  - Updates existing identities if EmployeeID matches
```

**Exam Tip:** Know the difference - SaaS connectors don't need VA, on-premises connectors do.

---

## Connector Selection Decision Tree
```
Does the source have internet-accessible API?
├─ Yes → Is it a supported SaaS connector?
│   ├─ Yes (Workday, Salesforce, etc.)
│   │   → Use Pre-built SaaS Connector
│   │   → No VA needed
│   │
│   └─ No (Custom API)
│       → Use Web Services Connector
│       → Configure API endpoints
│
└─ No → Is the source on-premises?
    ├─ Yes → What type?
    │   ├─ Active Directory
    │   │   → Use AD Connector via VA
    │   │
    │   ├─ Database
    │   │   → Use JDBC Connector via VA
    │   │
    │   ├─ LDAP
    │   │   → Use LDAP Connector via VA
    │   │
    │   └─ Legacy/No API
    │       → Use File-based Connector via VA
    │
    └─ Hybrid
        → Check specific requirements
        → May need VA or direct connection
```

---

## Connector Features Comparison

| Feature | SaaS Connector | VA Connector | File Connector |
|---------|----------------|--------------|----------------|
| **Real-time** | Yes (often) | Scheduled | Scheduled |
| **Provisioning** | Yes | Yes | Limited/No |
| **Requires VA** | No | Yes | Yes (usually) |
| **Complexity** | Low | Medium | Low |
| **Maintenance** | SailPoint | Customer | Customer |
| **Updates** | Automatic | Manual | Manual |
| **Best For** | Cloud apps | On-prem systems | Legacy systems |

---

## Practice Exercise

**Question:** You need to integrate these systems: Workday (cloud HR), Active Directory (on-prem), and a custom application with REST API (cloud). What connectors do you need?

<details>
<summary>Answer</summary>

**Analysis by System:**

**1. Workday (Cloud HR)**
```
Type: SaaS application
Connection: Direct from ISC
Connector: Pre-built Workday SaaS Connector

Requirements:
  - OAuth 2.0 credentials from Workday
  - No VA needed
  - Configure in ISC directly

Setup:
  Admin → Sources → Create Source → Workday
  Enter OAuth credentials
  Test connection
```

**2. Active Directory (On-Premises)**
```
Type: On-premises directory
Connection: Via Virtual Appliance
Connector: Active Directory Connector (on VA)

Requirements:
  - Virtual Appliance deployed
  - Network connectivity VA → Domain Controller
  - AD service account (svc_sailpoint@domain.com)

Setup:
  Deploy VA
  Admin → Sources → Create Source → Active Directory
  Assign to VA
  Configure DC, port, service account
  Test connection via VA
```

**3. Custom Application with REST API (Cloud)**
```
Type: Cloud application with API
Connection: Direct from ISC (if API is public) OR via VA (if API is internal)
Connector: Web Services Connector

Assuming public API:
  - No VA needed
  - Use Web Services connector
  - Configure API endpoints

Setup:
  Admin → Sources → Create Source → Web Services
  Configure:
    - Base URL: https://api.customapp.com
    - Authentication: API key or OAuth
    - GET /users endpoint for aggregation
    - POST /users endpoint for provisioning
  Test API calls
```

**Summary:**
```
Infrastructure Needed:
  ✓ 1 Virtual Appliance (for Active Directory)
  ✓ No VA needed for Workday or custom API

Connectors:
  1. Workday SaaS Connector (direct)
  2. Active Directory Connector (via VA)
  3. Web Services Connector (direct or via VA depending on API location)

Configuration Steps:
  1. Deploy VA first
  2. Configure Workday (easiest, no dependencies)
  3. Configure Active Directory (needs VA)
  4. Configure Custom API (may need VA depending on location)
  5. Test each source independently
  6. Configure identity profiles to use these sources
```
</details>

---

# 3.1.3 Generic Connector Categories

**Learning Objective:** Understand connector categories and when to use each type.

---

## Main Connector Categories

### **1. Standard Connectors**

**Definition:** Pre-built connectors for common systems maintained by SailPoint

**Characteristics:**
- Fully supported by SailPoint
- Regular updates and bug fixes
- Tested and certified
- Documented in SailPoint documentation
- Included with ISC subscription

**Examples:**
```
Directory Services:
  - Active Directory
  - Azure AD
  - LDAP
  - OpenLDAP

Applications:
  - Salesforce
  - ServiceNow
  - Workday
  - SAP

Databases:
  - Oracle
  - SQL Server
  - MySQL
  - PostgreSQL
```

**When to use:**
- Supported system available
- Need reliability and support
- Want automatic updates

**Exam Tip:** Standard connectors are pre-built, supported, and should be used when available.

---

### **2. Generic Connectors**

**Definition:** Flexible connectors for custom or unsupported systems

**Types:**

#### **JDBC Connector (Database)**

**Purpose:** Connect to any JDBC-compatible database

**Use cases:**
- Custom HR databases
- Legacy systems
- Home-grown applications
- Any SQL database

**Configuration:**
```
Connection:
  Driver: Oracle, SQL Server, MySQL, PostgreSQL, etc.
  Host: database.company.com
  Port: 1521 (Oracle), 1433 (SQL), 3306 (MySQL)
  Database: hr_prod
  Username: sailpoint_user
  Password: <encrypted>

Account Query (SQL):
  SELECT 
    employee_id,
    first_name,
    last_name,
    email,
    department,
    manager_id
  FROM employees
  WHERE active = 1

Provisioning (if supported):
  Create: INSERT INTO employees ...
  Update: UPDATE employees SET ...
  Delete: UPDATE employees SET active = 0 WHERE ...
```

**Advantages:**
- Works with any JDBC database
- Flexible SQL queries
- Can handle custom schemas

**Limitations:**
- Requires SQL knowledge
- Performance depends on queries
- Must write own SQL for operations

#### **LDAP Connector**

**Purpose:** Connect to any LDAP-compliant directory

**Use cases:**
- Non-Microsoft LDAP directories
- Custom LDAP implementations
- Legacy directories
- Unix/Linux user directories

**Configuration:**
```
Connection:
  Host: ldap.company.com
  Port: 389 (LDAP) or 636 (LDAPS)
  Bind DN: cn=sailpoint,ou=service-accounts,dc=company,dc=com
  Password: <encrypted>
  Base DN: dc=company,dc=com

Search Filter:
  (&(objectClass=person)(!(objectClass=computer)))

Attributes:
  - uid (username)
  - cn (common name)
  - mail (email)
  - ou (organizational unit)
  - memberOf (group membership)
```

#### **Web Services Connector**

**Purpose:** Connect to REST or SOAP APIs

**Use cases:**
- Modern cloud apps without standard connector
- Microservices
- Custom applications with APIs
- Third-party SaaS apps

**Configuration - REST Example:**
```
Base URL: https://api.company.com/v1

Aggregation:
  Endpoint: GET /users
  Authentication: Bearer token
  Response: JSON array of users

Provisioning:
  Create: POST /users
  Read: GET /users/{id}
  Update: PATCH /users/{id}
  Delete: DELETE /users/{id}

Sample Response:
{
  "users": [
    {
      "id": "12345",
      "username": "jsmith",
      "email": "john.smith@company.com",
      "status": "active"
    }
  ]
}
```

#### **Delimited File Connector**

**Purpose:** Read CSV, TSV, or other delimited files

**Use cases:**
- HR system exports
- Legacy system feeds
- One-way data imports
- Batch data loads

**Configuration:**
```
File Location: /mnt/hr-exports/employees.csv
Delimiter: Comma (,)
Header Row: Yes
Encoding: UTF-8

CSV Format:
EmployeeID,FirstName,LastName,Email,Department,Status
12345,John,Smith,john.smith@company.com,IT,Active
12346,Jane,Doe,jane.doe@company.com,HR,Active

Mapping:
  EmployeeID → identity.employeeId
  FirstName → identity.firstName
  LastName → identity.lastName
  Email → identity.email
  Department → identity.department
```

**Advantages:**
- Simple to set up
- Good for legacy systems
- Doesn't require API

**Limitations:**
- No real-time updates
- File must be generated by source
- Usually read-only (no provisioning)
- Requires file transfer mechanism

---

### **3. Custom Connectors**

**Definition:** Customer-built connectors for unique requirements

**When needed:**
- No standard connector exists
- Generic connector insufficient
- Complex custom logic required
- Proprietary protocols

**Types:**
- **Custom Java Connector:** Built using SailPoint SDK
- **PowerShell Connector:** Scripts for Windows systems
- **Custom Web Services:** Wrapper around complex APIs

**Note:** Custom connector development beyond exam scope, but know they exist.

**Exam Tip:** Use standard connectors when available, generic connectors for flexibility, custom connectors as last resort.

---

## Choosing the Right Connector

### **Decision Matrix**

| Scenario | Connector Choice |
|----------|------------------|
| **Connecting to Salesforce** | Standard Salesforce SaaS Connector |
| **Connecting to Active Directory** | Standard Active Directory Connector (via VA) |
| **Connecting to custom Oracle HR database** | Generic JDBC Connector |
| **Connecting to OpenLDAP directory** | Generic LDAP Connector |
| **Connecting to REST API app (no standard connector)** | Generic Web Services Connector |
| **Importing CSV from legacy system** | Generic Delimited File Connector |
| **Connecting to proprietary system** | Custom Connector (if needed) |

### **Best Practices**

**1. Prefer Standard Connectors**
```
✅ Use Workday connector for Workday
✅ Use Salesforce connector for Salesforce
✅ Use AD connector for Active Directory

❌ Don't use generic LDAP for Active Directory
   (AD connector has better features)

❌ Don't use Web Services for Workday
   (Standard connector already exists)
```

**2. Generic for Unsupported Systems**
```
✅ Use JDBC for custom HR database
✅ Use LDAP for non-AD LDAP directories
✅ Use Web Services for custom APIs

❌ Don't build custom connector if generic works
```

**3. Custom as Last Resort**
```
Only build custom connector when:
  - No standard connector exists
  - Generic connector can't handle requirements
  - Complex business logic needed
  - Proprietary protocol

Cost considerations:
  - Development time
  - Maintenance burden
  - Testing requirements
  - Documentation needs
```

---

## Connector Configuration Commonalities

### **All Connectors Need:**

**1. Connection Information**
```
Network:
  - Hostname or IP
  - Port number
  - Protocol (HTTP, LDAP, JDBC, etc.)

Authentication:
  - Username/password
  - API key
  - OAuth credentials
  - Certificate
```

**2. Schema Mapping**
```
Map source attributes to ISC schema:
  Source Attribute → ISC Attribute
  
  employeeId → identity.employeeId
  firstName → identity.firstName
  email → identity.email
  department → identity.department
```

**3. Account/Identity Correlation**
```
How to match:
  - Unique identifier (employee ID, email)
  - Correlation rule
  - Matching logic
```

**4. Aggregation Schedule**
```
When to run:
  - Daily at 2 AM
  - Every 6 hours
  - Manual only
```

**5. Provisioning Configuration (if applicable)**
```
Operations to support:
  - Create accounts
  - Modify accounts
  - Delete/disable accounts
  - Manage group membership
```

---

## Practice Exercise

**Question:** You need to connect to a custom employee database (Oracle) that your company built. Which connector type should you use and why?

<details>
<summary>Answer</summary>

**Answer: Generic JDBC Connector**

**Reasoning:**

**Why JDBC Connector:**
```
✅ Database is Oracle (JDBC-compatible)
✅ Custom database (no standard connector exists)
✅ Generic connector handles SQL databases
✅ Flexible - can write custom SQL queries
✅ Supports both aggregation and provisioning
```

**Why NOT other connectors:**
```
❌ Standard Oracle Connector - Doesn't exist for custom databases
   (Standard connectors are for specific apps like Workday, not generic databases)

❌ Web Services Connector - Database doesn't have REST API
   (Could work if you build API wrapper, but unnecessary)

❌ File Connector - Database is live, not file-based
   (Could export to CSV, but loses real-time capability)

❌ Custom Connector - Unnecessary when JDBC works
   (More complex, harder to maintain)
```

**Configuration Approach:**
```
1. Deploy Virtual Appliance (database is on-premises)

2. Create JDBC Source in ISC:
   Type: Generic JDBC Connector
   Assign to: VA-PROD-01
   
3. Configure Connection:
   Driver: Oracle
   Host: oracle-hr.company.com
   Port: 1521
   SID/Service: HRPROD
   Username: sailpoint_svc
   Password: <encrypted>
   
4. Write SQL for Aggregation:
   SELECT 
     employee_id,
     first_name,
     last_name,
     email,
     department,
     manager_id,
     hire_date,
     status
   FROM hr.employees
   WHERE status = 'ACTIVE'
   
5. Map Attributes:
   employee_id → identity.employeeId
   first_name → identity.firstName
   last_name → identity.lastName
   email → identity.email
   department → identity.department
   
6. Test:
   - Test connection
   - Run test aggregation
   - Verify data looks correct
   - Check correlation works
```

**Provisioning (Optional):**
```
If database supports account creation:

Create SQL:
  INSERT INTO hr.employees (
    employee_id, first_name, last_name, email
  ) VALUES (
    :employeeId, :firstName, :lastName, :email
  )

Update SQL:
  UPDATE hr.employees 
  SET email = :email, department = :department
  WHERE employee_id = :employeeId

Delete SQL:
  UPDATE hr.employees
  SET status = 'TERMINATED'
  WHERE employee_id = :employeeId
```

**Exam Tip:** JDBC connector is the go-to for ANY database - Oracle, SQL Server, MySQL, PostgreSQL, custom databases, etc.
</details>

---

## Section Summary

**Core Concepts for Certification:**

✅ **Source Purpose:**
- Sources provide identity and account data to ISC
- Aggregation = reading data from sources
- Provisioning = writing data to sources
- Authoritative sources = source of truth (usually HR)
- Non-authoritative sources = account information only

✅ **Source Types:**
- **On-Premises**: Need Virtual Appliance (AD, databases, LDAP)
- **Cloud/SaaS**: Direct connection from ISC (Workday, Salesforce, ServiceNow)
- **Hybrid**: Mix of both (Azure AD, O365)

✅ **Connector Categories:**
- **Standard**: Pre-built, supported (Workday, Salesforce, AD)
- **Generic**: Flexible for custom systems (JDBC, LDAP, Web Services, File)
- **Custom**: Built by customer (last resort)

✅ **Generic Connectors:**
- **JDBC**: Any SQL database
- **LDAP**: Any LDAP directory
- **Web Services**: REST/SOAP APIs
- **Delimited File**: CSV/TSV files

✅ **Connector Selection:**
1. Use standard connector if available
2. Use generic connector for unsupported systems
3. Build custom only when necessary

**Exam Focus Areas:**
- Know when VA is required (on-prem sources)
- Understand authoritative vs non-authoritative
- Recognize which connector for which source type
- Know the four main generic connectors (JDBC, LDAP, Web Services, File)
- Understand aggregation vs provisioning

---

**End of Section 3.1: Source Fundamentals**

**Next:** Section 3.2: Aggregation Process

---

**Study Tips:**
- Memorize: On-prem = VA, Cloud = Direct
- Know the generic connectors: JDBC, LDAP, Web Services, File
- Understand authoritative (HR) vs non-authoritative (apps)
- Remember: Use standard connectors when available
