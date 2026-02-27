# Connector Types — Choosing the Right Connection Method

ISC connects to systems through connectors. There are more than 200 pre-built connectors in SailPoint's connector catalog, covering nearly every enterprise application that has meaningful governance implications. There are also four connector frameworks for systems that don't have a pre-built connector.

Picking the wrong connector type for a system is one of the most expensive early mistakes in an integration project — wrong architecture decisions create rework measured in weeks, not hours. This document gives you the map.

---

## The Connector Catalog: Pre-Built Connectors

Pre-built connectors are ready-to-use integrations that SailPoint maintains. You configure them (credentials, endpoint URLs, attribute mappings) rather than building them. The catalog includes:

- **HRMS**: Workday, SAP SuccessFactors, Oracle HCM, ADP, UKG, Ceridian
- **Identity infrastructure**: Active Directory, Azure AD/Entra ID, LDAP, Okta, PingIdentity
- **SaaS productivity**: Microsoft 365, Google Workspace, Slack, Zoom, Box, Dropbox
- **ITSM**: ServiceNow, Jira, Atlassian Cloud
- **CRM/business apps**: Salesforce, Dynamics 365, HubSpot
- **Dev tools**: GitHub, GitLab, Bitbucket, Azure DevOps
- **Cloud infrastructure**: AWS, Azure, GCP (IAM roles and policies)
- **ERP**: SAP ERP, Oracle EBS, Workday Financials
- **Database**: Oracle, SQL Server, MySQL, PostgreSQL (via JDBC connector)
- **Security tools**: CyberArk, BeyondTrust, HashiCorp Vault (PAM integration)

If a system in your project scope is in this list, use the pre-built connector. Don't build a custom integration for a system that already has one — you'd be recreating work SailPoint's connector engineering team has already done, with better error handling and attribute schema support than a custom build would typically have.

> 🎯 **KEY POINT:** Check the connector catalog before any technical design conversation. What looks like a complex custom integration requirement frequently has a pre-built connector that covers 90% of the use case. Spending 15 minutes in the catalog can save weeks of custom development.

---

## The Two Deployment Models: Direct vs VA-Required

Pre-built connectors fall into two deployment categories based on whether they need a VA:

### Direct Connectors (No VA Required)

Cloud SaaS systems that ISC can reach directly over the internet don't need a VA. ISC cloud talks directly to the system's API.

**Systems using direct connectors**: Workday, Salesforce, ServiceNow, Google Workspace, Microsoft 365, GitHub, Okta, Slack, Box, AWS (using IAM role), Azure/Entra ID, GitLab, Jira Cloud.

The shared characteristic: these systems expose a public API endpoint that ISC cloud can call from the internet. No corporate network involvement required.

**Configuration**: Credentials only — typically an OAuth client ID/secret, API token, or service account credentials. No VA configuration. You configure the connector in the ISC admin console, enter credentials, run a test aggregation, and you're done.

### VA-Required Connectors

Systems that run inside your network — or that require corporate network access — need a VA as the intermediary.

**Systems requiring a VA**:
- **Active Directory / On-Prem LDAP**: Domain controllers are not internet-accessible (and shouldn't be). VA connects to them via LDAP/LDAPS from inside your network.
- **On-premises databases**: Oracle, SQL Server, MySQL running in your datacenter. VA has JDBC connectivity to them.
- **On-premises applications**: Legacy ERP, homegrown applications, any system without a public API.
- **Systems behind corporate network restrictions**: Some organizations route all system API traffic through internal proxies or restrict external API access for compliance reasons. VA handles those.
- **Active Directory**: Every AD connector runs through a VA. This is the most common VA-connected system in enterprise ISC deployments.

> ⚠️ **COMMON MISTAKE:** Assuming all AD connectors need a VA. Azure Active Directory (now Entra ID) does NOT need a VA — it's a cloud service with a public API. On-premises Windows Server Active Directory DOES need a VA. They're different connectors with different deployment models. Many implementations get this wrong in design and have to reconfigure when they discover the VA requirement.

---

## Connector Frameworks: When No Pre-Built Connector Exists

If a system doesn't have a pre-built connector, you have four framework options:

### 1. SCIM 2.0 Connector

SCIM (System for Cross-domain Identity Management) is an industry standard protocol for identity provisioning. If a system supports SCIM 2.0, ISC can connect to it with the SCIM connector — no custom development needed.

Many modern SaaS applications have added SCIM 2.0 support because it's the industry standard integration method. Before building a custom connector for any SaaS system, check if it supports SCIM — the answer is frequently yes.

**When to use**: The target system supports SCIM 2.0 and your governance requirements don't exceed what SCIM can model (user and group management).

**Limitations**: SCIM is designed for provisioning — it handles account creation, attribute updates, deactivation, and group membership well. It's less suited for complex entitlement models that don't map to the SCIM schema.

### 2. Web Services Connector

For systems with a REST or SOAP API but no ISC pre-built connector and no SCIM support, the Web Services connector lets you configure API calls without writing code.

The Web Services connector uses a rule-based configuration where you define:
- API endpoints (URLs, HTTP methods)
- Request payloads (JSON/XML templates)
- Response parsing (how to extract account attributes from the API response)
- Authentication method (OAuth, API key, Basic Auth)
- Operation mapping (which API calls map to Create, Read, Update, Delete)

This is configuration-intensive but avoids custom code. For a system with a relatively simple REST API, a Web Services connector configuration can be done in a day.

**When to use**: System has a documented REST or SOAP API, no pre-built connector exists, no SCIM support, and the API follows reasonably standard patterns.

**Limitations**: Complex APIs with non-standard authentication flows, multi-step operations, or complex data structures may exceed what the Web Services connector can model with configuration alone. Those cases escalate to a custom connector.

### 3. JDBC Database Connector

ISC includes a pre-built JDBC connector for direct database connectivity. You configure the JDBC connection string, credentials, and SQL queries — ISC runs the queries and maps results to identity attributes or account records.

**When to use**: Identity and access data lives in a database that ISC should read directly. Common cases:
- Legacy applications with no API but a readable database
- Systems where the authoritative data is in a database table (provisioning status, access flags)
- Audit data exports from systems that write access logs to a database

**JDBC in read vs write scenarios**: JDBC is commonly used as a source (ISC reads from the database) and less commonly as a target (ISC writes provisioning changes to the database). Writing directly to an application's database — bypassing the application's own business logic — is risky. Only write via JDBC if the application vendor confirms this is a supported integration pattern.

### 4. Custom Connector (Connector SDK)

For systems that don't fit any of the above — proprietary APIs, complex authentication flows, custom provisioning models, or systems with governance requirements that existing frameworks can't model — you build a custom connector using the SailPoint Connector SDK.

Custom connectors are Java-based and implement a defined interface (read, create, update, delete, aggregation operations). You build the connector, package it, deploy it to ISC, and it appears in the connector catalog for your tenant.

**When to use**: Last resort. Custom connectors work but they create maintenance obligations — your team owns the connector code, the upgrade compatibility, and the bug fixes. Every ISC platform upgrade needs to be tested against your custom connector. If a pre-built connector, SCIM, Web Services, or JDBC can do the job, use that instead.

**Justified uses**:
- Proprietary mainframe systems with EBCDIC formats and non-standard protocols
- Hardware-based systems (badge readers, physical access control) with vendor-specific APIs
- Highly custom applications where the vendor provides a support engagement with connector development

---

## The Flat File Connector

Not really a framework — more of a fallback for systems that can't support any real-time integration method.

The flat file connector reads structured data files (CSV, delimited, fixed-width) that source systems place on an agreed location (SFTP, file share, local path). ISC processes the file as a batch aggregation.

**When to use**:
- Systems with no API at all — they can only produce file exports
- Legacy systems in highly controlled environments where direct connectivity is prohibited
- Transition periods: during migration from an old system to a new one, the old system may only support file exports
- Physical systems (badge access, facilities management) that export activity logs as files

**Flat file as source only**: The flat file connector reads from files — it can't write back. It's useful for importing identity attributes or account data into ISC. It cannot provision accounts in the source system.

---

## Decision Matrix

| System Type | Connector Choice |
|---|---|
| Cloud SaaS with pre-built connector | Pre-built connector (direct, no VA) |
| On-prem/datacenter with pre-built connector | Pre-built connector (VA required) |
| Any system with SCIM 2.0 support | SCIM connector |
| System with REST/SOAP API, no SCIM | Web Services connector |
| Identity data in a database | JDBC connector |
| File export only | Flat file connector |
| None of the above | Custom connector (SDK) |

**Escalation rule**: Work down the list in order. Start at "pre-built connector" and only escalate to the next option when the one above it genuinely can't work. Each step down adds development complexity and maintenance cost.

---

## GlobalTech's Connector Map

GlobalTech's 47 connected systems use:

| Connector Type | Systems | Count |
|---|---|---|
| Pre-built (direct) | Workday, Salesforce, GitHub, ServiceNow, Jira Cloud, Slack, AWS, Azure, Google Workspace, Office 365, Okta | 11 |
| Pre-built (VA-required) | Active Directory (3 domains), Oracle DB (2 instances), SAP ERP | 6 |
| SCIM 2.0 | 4 modern SaaS vendors with SCIM support | 4 |
| Web Services | 12 systems with REST APIs but no pre-built connector | 12 |
| JDBC | Legacy ERP database (read-only), payroll database | 2 |
| Flat file | Legacy payroll file, badge system, benefits admin | 3 |
| Custom SDK | 1 proprietary manufacturing floor system | 1 |
| No connector (service desk) | 8 legacy systems — manual provisioning via tickets | 8 |

The 8 systems on service desk tickets represent technical debt. Each one is a system that ISC can request access to (governance), but where the actual provisioning step is a human following a ticket. The access request lifecycle works; real-time provisioning doesn't.

> 💡 **REMEMBER:** Not every system needs direct provisioning connectivity. For systems with low provisioning volume, high complexity, or vendor restrictions, the right answer is sometimes governance coverage (access requests, certifications, visibility) with manual provisioning execution. ISC's governance value doesn't require direct provisioning capability — it requires knowing who has access and having a controlled request process.

---

## Key Takeaways

- Check the pre-built connector catalog first — 200+ connectors cover the vast majority of enterprise systems
- Direct connectors (cloud SaaS) need credentials only; VA-required connectors (on-premises) need a configured VA cluster
- Azure AD/Entra ID is direct (no VA); on-premises Windows Active Directory requires a VA — don't confuse them
- SCIM 2.0 connector: try before building anything custom for modern SaaS systems
- Web Services connector covers REST/SOAP APIs without code; JDBC covers database-as-source integrations
- Custom SDK connectors are last resort — they create maintenance obligations your team will own
- Governance coverage (access requests, certifications) doesn't require direct provisioning — service desk ticketing is a legitimate integration pattern for some systems

> 🔗 **CONNECTS TO:** You know what connector to use. Now you need to understand what aggregation actually does — how it runs, what it produces, and what can go wrong at every stage of the process.
