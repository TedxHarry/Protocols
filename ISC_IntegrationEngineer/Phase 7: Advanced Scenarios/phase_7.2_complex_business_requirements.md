# Real Organizational Complexity — Modeling What Organizations Actually Look Like

The clean organizational model — employees with one manager, one department, one location, full-time permanent status — exists in textbooks and rarely in actual companies. Real organizations have matrix reporting structures, contractors on time-limited agreements, employees split across multiple legal entities, part-time workers with proportional access needs, and business units with strict data isolation requirements.

Each of these deviations from the simple model requires explicit handling in ISC. If you design for the simple model and discover the complexity during implementation, you rebuild. This document covers the six organizational complexity patterns that appear most often — and how to model each in ISC.

---

## Pattern 1: Matrix Organizations — Multiple Reporting Structures

**What it looks like**: An engineer reports to an Engineering manager for HR purposes (primary reporting line) but to a Product manager for project work (functional reporting). HR data has one manager. The engineer's day-to-day work is directed by a different manager.

**Why it matters for ISC**: Manager attributes drive two critical governance processes:
1. **Access request approvals**: "Manager approval required" — which manager?
2. **Certification campaigns**: Manager certifications route to the direct manager — does the HR manager have visibility into the access the engineer uses for product work?

**ISC modeling approach**:

Workday stores both reporting relationships if configured correctly: `primaryManager` (HR line) and `functionalManager` (project/product). ISC aggregates both.

For access certifications, configure separate campaign types:
- HR manager certifications: route to `primaryManager` for HR-owned access (baseline, role-based)
- Functional manager certifications: route to `functionalManager` for project-specific access

For access request approvals, the approval workflow can branch based on the type of access:
- Standard role-based access → route to `primaryManager`
- Project-specific tool access → route to `functionalManager` (or both, if required by policy)

**GlobalTech's implementation**: Product Operations team members have dual reporting. A `functionalManagerId` attribute is aggregated from Workday and stored on the ISC identity. Approval workflows for project tool access route to the functional manager; everything else routes to the primary HR manager.

> ⚠️ **COMMON MISTAKE:** Assuming one manager field covers all approval needs. When approval routes to the HR manager for access the engineer doesn't control in their HR role, approval quality degrades — the HR manager doesn't know whether the product-specific access is appropriate. Route to the person who actually supervises the work, not just the person in the hierarchy.

---

## Pattern 2: Contractors and Temporary Workers — Time-Limited Access

**What it looks like**: A consulting firm places a contractor at GlobalTech for 6 months. The contractor needs GitHub, Jira, and Confluence access for the project. They should not have access to employee HR systems, benefits portals, or permanent system resources. Their access should automatically expire on the contract end date.

**ISC modeling approach**:

**Lifecycle state**: Contractors use a `Contractor-Active` lifecycle state (introduced in Phase 5.3) distinct from `Active`. This enables different provisioning rules and different certification frequency.

**Time-limited access**: ISC supports access expiry dates on role and access profile assignments. When assigning access to a contractor identity, set an expiry date equal to the contract end date. ISC automatically revokes access on that date without requiring manual action.

**Role separation**: Create separate contractor roles that exclude employee-only resources:
- `Contractor-Software` role: GitHub Contributor, Jira Software, Confluence — no AD full-employee groups, no benefits portal, no HR systems
- `Employee-Software Engineer` role: same technical access plus employee-specific resources

**Attribute source for contractors**: If your primary HR system (Workday) doesn't manage contractors, you may have a separate contractor management system (Beeline, SAP Fieldglass, Workday Contingent Worker module). This becomes a second authoritative source with its own aggregation and correlation configuration.

**Critical design decision — what happens at contract end**: Three options:
1. Deactivate access immediately on `contractEndDate` (lifecycle state → Terminated equivalent)
2. Send notification to manager 2 weeks before end date, deactivate on end date
3. Put in a `PendingOffboard` state for 5 business days to allow access transfer/documentation, then terminate

Option 2 (notification + auto-deactivate) is the most common and balances governance with operational smoothness.

> 💡 **REMEMBER:** Contractors who are rehired (second engagement) must be handled as a re-hire, not a new hire. If their previous accounts were archived (not deleted), ISC can re-activate them. If they were deleted, they re-provision from scratch. Decide the retention policy for contractor accounts before the first contractor is offboarded.

---

## Pattern 3: Multiple Geographies — Different Rules by Location

**What it looks like**: GlobalTech has employees in the US, UK, Germany, and Japan. The US employees use Office 365 US tenant. UK and EU employees use Office 365 EU tenant (data residency requirement). German employees have work council agreements that require manager approval for any access change. Japanese employees access SAP via a regional instance (SAP-APAC) rather than the global instance.

**Why it matters**: A single global role definition doesn't work when different regions use different systems, different tenants of the same system, or have different approval requirements.

**ISC modeling approach**:

**Region-specific roles**: Create region-aware Business Roles using membership criteria that include location:

```
US Software Engineer:
  department = "Engineering" AND location.country = "US" AND …

EU Software Engineer:
  department = "Engineering" AND location.country IN ("UK", "DE", "FR") AND …
```

Each regional role points to the correct regional system instances (US O365 vs EU O365, global SAP vs SAP-APAC).

**Regional approval workflows**: Germany's work council requirement is a workflow modification. For identities with `location.country = "DE"`, add a step to access approval workflows requiring works council notification (or notification to a designated HR representative). ISC workflow conditions support attribute-based branching.

**Data residency**: If EU data must stay in EU infrastructure, ensure your ISC tenant configuration, VA placement, and connected system configurations comply. SailPoint offers EU-hosted ISC tenants for organizations with EU data residency requirements. This is a configuration choice, not an ISC feature limitation.

> 🎯 **KEY POINT:** Regulatory requirements by geography (GDPR, German work council laws, Japan's APPI) must be captured in the requirements phase and encoded in workflows, not discovered after go-live. Ask during requirements: are there location-based approval or compliance requirements? The answer is almost always yes in global organizations.

---

## Pattern 4: Complex Role Hierarchies — Role Inheritance and Layering

**What it looks like**: GlobalTech has a "Cloud Infrastructure Engineer" role that inherits from "Software Engineer" (which has base developer access), with additional AWS admin permissions layered on top. Senior engineers get all of Software Engineer plus elevated GitHub Maintainer access.

**ISC role modeling for hierarchical access**:

ISC roles can contain other roles (nesting). A `Cloud Infrastructure Engineer` Business Role includes:
- `Standard Employee Access` IT Role (AD, VPN, email)
- `Developer` access profile bundle (GitHub Contributor, Jira, Confluence)
- `AWS Developer` access profile (baseline AWS access)
- `AWS Infrastructure Admin` access profile (elevated AWS access specific to infra team)

Rather than duplicating the Software Engineer access profiles in every role that needs them, the Software Engineer role is defined once and nested inside Cloud Infrastructure Engineer. When Software Engineer access changes (a new tool is added), the update propagates to all roles that include it.

**Pitfall: Role nesting depth**:
Deeply nested roles (A includes B includes C includes D) become hard to understand and audit. A manager reviewing access for a Cloud Infrastructure Engineer shouldn't have to trace 4 levels of role nesting to understand what that engineer has. Keep nesting to 2 levels maximum: IT Roles → Business Roles, with Business Roles containing access profiles directly.

**Requestable elevation**: The pattern for "senior gets more access" shouldn't be a separate Senior Engineer role for each access tier. Instead:
- `Software Engineer` role (birthright, based on HR attributes)
- `GitHub Maintainer` access profile (requestable, requires manager approval)
- `AWS Production Access` access profile (requestable, requires manager + security approval)

Seniority-based elevation is requestable add-on, not a parallel role hierarchy.

---

## Pattern 5: Part-Time Workers — Proportional Access

**What it looks like**: A part-time employee works Monday-Wednesday. They need the same tools as full-time employees in their role but only have authorized access during their working hours.

**ISC's capabilities here are limited**: ISC governs access (whether someone has an account with specific permissions) but does not natively enforce time-of-day access restrictions. Time-based access control is typically enforced at the authentication layer (Active Directory logon hours, Okta time-of-day policies, network access control).

**What ISC can manage**:
- Role assignment: part-time employees get the same roles as full-time employees in the same function
- Lifecycle-based access: if a part-time employee goes on extended leave, ISC manages the LOA lifecycle correctly
- Certification: part-time employees are included in access certifications like all other employees

**The integration point**: If your organization wants time-of-day restrictions for part-time workers, configure this in the authentication system (AD logon hours, Okta time policies) and use ISC to govern who is configured with which time restriction. ISC ensures the right policy is applied; the authentication system enforces it.

---

## Pattern 6: Business Unit Data Isolation

**What it looks like**: GlobalTech has two divisions — Software Products and Manufacturing. Each has its own GitHub Enterprise instance, its own Jira project space, and its own Confluence wiki. Manufacturing engineers should not see Software source code; Software engineers should not have access to manufacturing system specifications.

**ISC modeling approach**:

**Separate access profiles per division**: `GitHub - Software Engineering` and `GitHub - Manufacturing` are distinct access profiles, each connected to the relevant GitHub Enterprise instance (or organization within a single instance).

**Role separation**: `Software Engineer` role does not include Manufacturing GitHub access. `Manufacturing Engineer` role does not include Software GitHub access. The divisional boundary is encoded in role membership criteria.

**Cross-division access exception process**: When a manufacturing engineer needs temporary access to the software GitHub for a joint project, this becomes an access request — a time-limited, manager-approved exception. ISC governs the exception request and the expiry.

**Shared services**: Both divisions share AD, Office 365, Slack. Shared-access systems are included in both divisional roles; division-specific systems are not.

> ⚠️ **COMMON MISTAKE:** Trying to use a single role for employees who sometimes work across divisions. "Full Access" roles that span both Manufacturing and Software violate the data isolation the business intended. Model the exception process in ISC instead — the default is division-scoped, and cross-divisional access is a governed, time-limited exception.

---

## Key Takeaways

- Matrix organizations need both primary and functional manager attributes aggregated to support correct approval routing and certification assignment
- Contractor lifecycle management requires time-limited access with automatic expiry on contract end date — and a defined retention policy for contractor accounts post-engagement
- Geographic complexity (different regional systems, regulatory requirements) should be captured in requirements and encoded in regional roles and workflow conditions
- Role hierarchies use nesting (IT Roles → Business Roles) for inheritance but should stay shallow (2 levels max) for auditability
- Part-time time-of-day restrictions are enforced at the authentication layer; ISC governs who has which policy applied
- Business unit data isolation is modeled through separate access profiles and roles with clear divisional scope; cross-divisional access is an exception process with expiry

> 🔗 **CONNECTS TO:** Understanding organizational complexity tells you how to model identities. Separation of Duties tells you which combinations of access create risk — and how ISC detects and enforces conflict prevention across all the systems and organizational structures you've been modeling.
