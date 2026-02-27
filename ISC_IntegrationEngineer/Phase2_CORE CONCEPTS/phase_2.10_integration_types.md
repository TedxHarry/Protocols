# Different Types of Integrations You'll Build

Not every integration is the same. A connection to an HR system works completely differently from a connection to Active Directory, which works differently from a connection to a legacy payroll system that doesn't have an API.

As an integration engineer, you need to be able to look at any system and immediately understand what type of integration it requires — because the type determines the architecture, the data flow, the connector approach, and the governance strategy.

There are four integration patterns in ISC. Real enterprise implementations use all four simultaneously, each for the right use case.

## Pattern 1: Authoritative Source (Read-Only INTO ISC)

An **authoritative source** is a system that ISC reads from but never writes to. This system is the official source of truth for identity attributes.

The direction is always: Source System → ISC. Never the reverse.

**Canonical example**: Workday as an HR source.

Workday holds the ground truth about GlobalTech's employees: who they are, what department they're in, what their job title is, who their manager is, and when they're terminated. ISC reads this data through aggregation and uses it to populate identity records.

ISC does NOT write back to Workday. If someone changes their department in ISC, ISC doesn't update Workday — because Workday is authoritative. HR makes changes in Workday; ISC reads them.

**Key characteristics**:
- ISC connector: READ operations only (no CREATE, UPDATE, DELETE on accounts)
- Aggregation: scheduled (nightly is common), sometimes event-driven for high-priority changes
- Identity data: this system's records become the identity records in ISC
- Data quality: whatever is in this system becomes the governance foundation — data quality problems here cascade everywhere

**Common authoritative sources**:
- Workday (most common HR system)
- SAP SuccessFactors
- ADP Workforce Now
- UKG (Ultimate Kronos Group)
- PeopleSoft HCM
- Custom HR databases

> 🎯 **KEY POINT:** Every ISC implementation needs at least one authoritative source — the system that tells ISC who exists and what their fundamental attributes are. This is the governance foundation. Without an authoritative source, ISC doesn't know who to govern access for. Choosing and connecting the right authoritative source is often the most important first integration in any ISC deployment.

## Pattern 2: Target (Managed) System (ISC Writes TO System)

A **target system** is one that ISC provisions into. ISC reads account and entitlement data from it (via aggregation), but also writes to it — creating accounts, assigning entitlements, disabling accounts, removing access.

The primary direction is: ISC → Target System. Read aggregation also flows: Target System → ISC.

**Canonical example**: Salesforce as a target.

ISC reads Salesforce accounts and profiles during aggregation. When someone needs Salesforce access, ISC provisions a Salesforce account with the appropriate profile. When they leave, ISC disables the account. ISC controls the account lifecycle.

**Key characteristics**:
- ISC connector: FULL operations (CREATE, UPDATE, DISABLE, DELETE on accounts; ASSIGN/REMOVE on entitlements)
- Provisioning: driven by ISC policies and access profiles
- Account lifecycle: fully managed by ISC (ISC is the controlling system for this target)
- Connector type: usually a pre-built SaaS connector (200+ available in ISC)

**Common target systems**:
- Salesforce (CRM)
- ServiceNow (ITSM)
- GitHub Enterprise
- Slack
- Jira / Confluence
- Box, Dropbox
- Most SaaS applications

> 💡 **REMEMBER:** Target systems are sometimes called "managed systems" in ISC documentation. The key distinction is that ISC has write authority over these systems. When you add a new SaaS application to your environment, your first question should be: "Is there an ISC connector for this?" If yes, it likely becomes a target system. If no, you're building a custom connector or falling back to flat file.

## Pattern 3: Authoritative + Managed (Bidirectional)

Some systems are both — ISC reads identity attributes FROM them (making them an authority on some data) AND provisions accounts TO them (making them a managed target).

**Canonical example**: Active Directory.

In most enterprise environments, AD is both:
- **Authoritative** for: email addresses, display names, organizational unit structure (in some orgs, AD drives org chart even when HR doesn't)
- **Managed target** for: account creation, group membership, attribute synchronization from ISC, account disabling

ISC reads AD during aggregation to get current account and group membership data. ISC also provisions TO AD when someone is hired (creates account, assigns groups) or when access changes (adds/removes group memberships).

The data flow is genuinely bidirectional, but with clear rules about which attributes are authoritative in which direction. Workday is authoritative for department and job title — those flow Workday → ISC → AD. AD is authoritative for email format — that flows AD → ISC (informational) but ISC → AD (writes the actual email attribute during provisioning).

**Key characteristics**:
- ISC connector: Full read AND write
- Aggregation: frequent (detects changes made directly in AD)
- Provisioning: ISC creates and manages AD accounts
- Attribute authority: must be clearly defined — which system "wins" for each attribute

**Other common bidirectional systems**:
- Azure Active Directory / Entra ID
- LDAP directories
- Some ITSM systems (ServiceNow can be both source and target)

> ⚠️ **COMMON MISTAKE:** Not clearly defining attribute authority for bidirectional systems. If both Workday and AD have a `department` attribute, and changes can happen in either system, ISC needs explicit configuration: which one wins? If it's undefined, you get conflicts where Workday pushes "Engineering" but AD shows "IT" and they keep overwriting each other. This is an attribute authority design problem — solve it in design, not in debugging.

## Pattern 4: Flat File / Delimited Integration

Some systems can't be integrated via API. They're too old, too closed, or behind network boundaries that make direct connector access impractical. For these, ISC uses file-based integration.

**How it works**: The source system generates a file (CSV, delimited text, fixed-width) on a schedule. The file is placed on an agreed file share or SFTP server. ISC's file connector reads the file and processes it as an aggregation source.

**Canonical example**: GlobalTech's legacy payroll system (a 2009-era on-premises application with no REST API, no LDAP interface, only JDBC connectivity to an Oracle database that the vendor doesn't allow direct access to).

The payroll system generates a CSV file every night at 2 AM:
```
EmployeeID,FirstName,LastName,Department,CostCenter,Status,HireDate,TermDate
GT-48291,Sarah,Chen,Engineering,ENG-001,Active,2021-08-15,
GT-39102,David,Park,Finance,FIN-002,Active,2019-03-22,
GT-11847,Jennifer,Rodriguez,Marketing,MKT-001,Terminated,2018-06-10,2024-11-30
```

The file lands on an SFTP server at 2:15 AM. ISC's flat file connector picks it up at 3:00 AM, processes it, and updates identity records accordingly.

**Key characteristics**:
- No real-time updates — latency is at least one file generation cycle (often 24 hours)
- File format changes can break the integration silently if not monitored
- Error handling is limited: if a row is malformed, it might be skipped without clear alerting
- No provisioning possible (can't write accounts back to the system via file)
- Audit trail for the source data is limited to file contents

**Common flat file sources**:
- Legacy ERP systems (pre-2010 vintage)
- Physical badge/access systems
- Contractor management systems with limited API support
- Government/regulatory systems
- Acquired company systems during transition periods

> 🎯 **KEY POINT:** Flat file integrations are not second-class citizens in ISC implementations. They're the practical solution for systems that can't support API connectivity. Many regulated industries have legacy systems that will never get REST APIs, and flat file is the governance bridge. Designing robust flat file integrations — with checksum validation, row count verification, and error alerting — is a real engineering skill.

## Decision Table: Choosing Your Integration Pattern

| Question | If Yes → | Pattern |
|---|---|---|
| Does this system define who employees ARE? | Source → ISC | Authoritative Source |
| Does ISC create/manage accounts on this system? | ISC → System | Target (Managed) |
| Both of the above? | Bidirectional | Authoritative + Managed |
| No API available, only file exports? | File → ISC | Flat File |
| Does this system have an ISC pre-built connector? | Likely Target or Bidirectional | Check connector capabilities |

## Three Questions to Ask About Any New System

When you're brought into a project to connect a new system to ISC, ask these three questions before touching a keyboard:

**1. Does this system tell ISC who exists, or does ISC tell it what to do?**
If the system is a source of identity truth (HR, contractor management, acquisition systems) → Authoritative Source pattern. If ISC controls who has access on this system → Target pattern. If both → Bidirectional.

**2. Does this system have an ISC connector?**
Check ISC's connector list (in the ISC admin console under Source → New Source). If there's a pre-built connector, start there. If not: evaluate the system's API (REST, SOAP, SCIM, LDAP, JDBC), determine if the ISC Connector SDK can build a custom connector, or fall back to flat file.

**3. What's the latency requirement?**
If terminations need to propagate within minutes → scheduled aggregation isn't enough; consider event-driven integration or at minimum very frequent delta aggregation. If 24-hour latency is acceptable → nightly aggregation is fine. If even longer latency is OK → weekly flat file might suffice.

## The Reality: All Four Patterns Coexist

GlobalTech's ISC implementation uses all four patterns simultaneously:

- **Workday**: Authoritative source (HR data drives identity records, no provisioning back to Workday)
- **Active Directory**: Authoritative + Managed (ISC reads and writes, AD is partially authoritative)
- **Salesforce, GitHub, Jira, Slack, AWS**: Target systems (ISC fully manages account lifecycle)
- **Legacy payroll**: Flat file (nightly CSV, supplementary identity data for cost center and pay grade attributes)
- **Physical badge system**: Flat file (weekly file, used for building access governance audit)

This is normal. Enterprise environments are heterogeneous. There's no single integration pattern that works for all systems. The integration engineer's job is to pick the right pattern for each system based on its capabilities, the governance requirements, and the latency needs.

## Key Takeaways

- Four integration patterns: Authoritative Source (read-only into ISC), Target/Managed (ISC writes to system), Authoritative+Managed (bidirectional), Flat File (file-based, no API)
- Pattern choice depends on three questions: who is the authority (system or ISC), does the system have a connector, and what's the latency requirement
- Bidirectional integrations (Active Directory being the prime example) require explicit attribute authority definitions to prevent overwrite conflicts
- Flat file integrations are a legitimate, widely-used pattern for legacy systems — not a shortcut but a real integration approach
- Real implementations use all four patterns — enterprise environments are heterogeneous by nature

> 🔗 **CONNECTS TO:** You now understand what integrations are. Phase 3 shifts from concepts to practice — starting with what an integration engineer actually does day-to-day, the decisions they make, and the mistakes to avoid.
