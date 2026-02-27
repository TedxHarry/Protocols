# What is an Identity in ISC?

Before you can govern access, you need to know who you're governing it *for*. That sounds obvious — but in enterprise IT environments, the answer is more complicated than it first appears.

A "person" in your organization exists in a dozen different systems. In Active Directory, they're a samAccountName. In Salesforce, they're a Username and a UserId. In Workday, they're an Employee_ID. In GitHub, they're a login handle they chose themselves five years ago. In the physical badge system, they're an employee number on a card.

Are these all the same person? Yes. Does any single system know they're the same person? Almost certainly no.

This is the core problem that the ISC **identity** model solves.

## What an Identity Actually Is in ISC

An **identity** in ISC is not an account. It's not a username. It's the person — represented as a unified, authoritative record that aggregates information from multiple source systems into a single, coherent view.

Think of it as the canonical answer to the question: "who is this person in our organization?" The identity record holds:

- **Who they are**: Legal name, preferred name, employee ID, personal and work contact information
- **Where they fit**: Department, job title, job code, cost center, location, business unit, manager
- **Their status**: Employment type (full-time, contractor, intern), lifecycle state (active, on leave, terminated)
- **Their history**: When they joined, what roles they've held, what access changes have occurred
- **Their risk profile**: ISC-calculated risk score based on the access they hold and behavioral signals

This unified record is what SailPoint calls the **Identity Cube** — a term for the complete, multidimensional view of a person's identity across the organization.

> 🎯 **KEY POINT:** In ISC, the identity is the anchor point for everything. Accounts get linked to identities. Entitlements get associated with identities through their accounts. Policies evaluate against identity attributes. Certifications are organized around identities. If the identity record is inaccurate, the entire governance framework built on top of it is inaccurate too.

## Sarah Chen: A Real Identity Record

Let's make this concrete. Sarah Chen is a Software Engineer at GlobalTech, a 12,000-person manufacturing and software company. She's been with GlobalTech for three years, started in the Operations department, and moved to the Engineering team 18 months ago.

Here is Sarah's ISC identity record:

**Core Attributes (sourced from Workday — GlobalTech's authoritative HR system)**:
- `displayName`: Sarah Chen
- `firstName`: Sarah
- `lastName`: Chen
- `employeeId`: GT-48291
- `email`: sarah.chen@globaltech.com
- `department`: Engineering
- `jobTitle`: Software Engineer
- `jobCode`: ENG-SW-003
- `manager`: james.wilson@globaltech.com (James Wilson, Engineering Manager)
- `location`: Austin, TX
- `employmentType`: Full-Time
- `startDate`: 2021-08-15
- `lifecycleState`: Active

**Calculated Attributes (derived by ISC from source data)**:
- `riskScore`: 42 (Medium — elevated due to production system access)
- `lastCertification`: 2024-11-15
- `accountCount`: 7
- `entitlementCount`: 23
- `policyViolations`: 0

**Account Links (correlated by ISC)**:
- Active Directory account: `s.chen`
- Salesforce account: `sarah.chen@globaltech.com`
- GitHub account: `schen-dev`
- AWS account: `sarah_chen_dev`
- Slack account: `sarah.chen`
- Jira account: `SChen`
- SAP Finance account: `SCHEN001`

Every one of these accounts is linked to Sarah's identity record. When you look at Sarah's identity in ISC, you see all seven accounts, all 23 entitlements across those accounts, and the full history of how each was granted.

> 💡 **REMEMBER:** The attributes on Sarah's identity record come from Workday — GlobalTech's **authoritative source**. This means Workday is the system of record for Sarah's core attributes. When Workday says Sarah's job title is "Software Engineer," ISC uses that. When Workday updates her title to "Senior Software Engineer," ISC picks it up during the next aggregation and updates her identity record — which then triggers policy re-evaluation and potential access changes.

## The Authoritative Source: Where Identity Data Comes From

ISC doesn't invent identity attributes. It reads them from authoritative source systems — systems that are designated as the official source of truth for specific attributes.

In most organizations:

- **HR system (Workday, SAP SuccessFactors, ADP)**: Authoritative for core employment attributes — name, department, title, manager, employee ID, start date, employment status
- **Active Directory / Azure AD**: Sometimes authoritative for email address and display name (if HR doesn't manage these)
- **Custom identity sources**: Some organizations maintain separate systems for contractor data or executive information

The authoritative source designation matters enormously. If Workday says Sarah is "Active" and her manager's system says she's "Terminated," ISC needs to know which one to believe. In GlobalTech's configuration, Workday wins — it's the system of record for employment status. When Workday changes, ISC follows.

This is also why HR data quality directly impacts identity governance quality. Inaccurate data in Workday — wrong department, wrong manager, wrong job code — produces an inaccurate identity record, which produces incorrect policy evaluation, which provisions wrong access or flags wrong violations. Garbage in, governance failure out.

> ⚠️ **COMMON MISTAKE:** Assuming the HR system is always clean and correct. In real implementations, you'll encounter employees with two records (name change), contractors with incorrect employment types, department codes that don't match any policy definition, and managers who left the company six months ago still showing as active in Workday. Part of your integration engineering work is surfacing these data quality issues and designing correlation strategies that handle them gracefully.

## Why Attributes Drive Everything

Sarah's identity attributes aren't just descriptive — they're the inputs that drive every automated governance decision ISC makes.

**Access Provisioning**: ISC evaluates `department = Engineering` and `jobTitle = Software Engineer` against access policies. These attributes trigger the "Software Engineer" role assignment, which provisions her accounts on GitHub, Jira, Slack (with Dev channels), and AWS (developer role). Change the attributes, change the access.

**Policy Evaluation**: ISC checks Sarah's entitlements against SoD policies. If she somehow accumulated Finance entitlements while in Engineering, a policy rule would flag the combination. The identity attributes that identify her as an Engineer are part of the policy context.

**Certification Targeting**: Access certifications are assigned based on identity attributes. All certifications for Sarah's entitlements go to `james.wilson` (her manager per her identity record). If her manager changes in Workday, ISC updates the identity record, and future certifications route to the new manager automatically.

**Lifecycle Triggers**: When Sarah's `lifecycleState` changes (from Active to Leave to Active again, or eventually to Terminated), ISC triggers the corresponding lifecycle workflow — suspending access, restoring it, or deprovisioning it entirely.

> 🎯 **KEY POINT:** Identity attributes are the control plane for automated access management. The attribute values in an identity record are the variables that ISC's policy engine evaluates to make decisions. This is why ISC integration engineers spend significant time on attribute mapping — getting the right attributes flowing accurately from source systems into identity records is foundational to everything else working correctly.

## Identity vs. Account: The Critical Distinction

This distinction will come up constantly in your work, and it's worth being crystal clear:

**Identity** = the person (Sarah Chen, employee GT-48291)
**Account** = the person's presence on a specific system (Sarah's AD account `s.chen`, Sarah's GitHub account `schen-dev`)

Sarah is one identity with seven accounts. The identity record is ISC's unified view. Each account is a system-specific credential that Sarah uses to access that system.

When ISC governs Sarah's access, it governs through her identity — but the actual governance actions (enable/disable, add/remove entitlements) execute against her accounts on each specific system.

This matters when something breaks. If Sarah can't access GitHub, the problem isn't her *identity* — it's her *account* on GitHub (possibly disabled, misconfigured, or uncorrelated). If Sarah's access policies are wrong, the problem is her *identity* attributes (wrong department triggers wrong policies). Knowing which layer has the problem tells you where to look.

## Key Takeaways

- In ISC, an identity is the unified representation of a person — not an account, not a username
- Identity attributes (department, title, manager, status) come from authoritative sources (usually the HR system) and drive all automated governance decisions
- ISC's Identity Cube aggregates account data from all connected systems and links it to the person's identity record
- The quality of identity governance depends directly on the quality of the underlying identity data — HR data problems become governance problems
- Identity ≠ Account: one person has one identity and potentially many accounts across different systems

> 🔗 **CONNECTS TO:** Now that you understand what an identity is, the next document dives into accounts — what they are, how they differ across systems, and the important distinction between having an account and having access.
