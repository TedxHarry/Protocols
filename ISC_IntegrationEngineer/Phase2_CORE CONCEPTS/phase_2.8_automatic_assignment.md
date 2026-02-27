# How ISC Automatically Assigns Access

If every access decision required a human, identity governance at scale would be impossible. You can't manually evaluate what 12,000 employees should be able to access across 47 systems — not daily, not even weekly.

ISC's answer is **automatic access assignment**: a policy-driven system that evaluates who someone is (their identity attributes) and automatically determines what access they should have (their entitlements). When the attributes match a policy, the access provisions automatically. When attributes change, the access updates automatically. When someone leaves, the access revokes automatically.

This isn't magic — it's a configuration challenge. And understanding the mechanism is central to integration engineering work.

## The Building Blocks: Access Profiles and Roles

ISC has two key constructs for organizing and assigning access:

**Access Profile** — a collection of entitlements from a SINGLE source system, bundled together because they're logically related and typically granted together.

Think of an access profile as a pre-packaged access bundle for one system. "GitHub Developer Access" might include:
- Org membership: `globaltech-corp`
- Team: `backend-engineers`
- Team: `platform-team`

When someone needs GitHub Developer Access, ISC provisions all three entitlements as a unit. You don't grant them individually — you grant the profile.

Access profiles can also have a **provisioning source**: if someone is assigned this access profile, ISC knows it needs to provision it on GitHub (and which GitHub connector to use).

**Role** — a collection of access profiles spanning MULTIPLE source systems, representing the complete access package for a business function.

A role maps to a business concept: "Software Engineer," "Finance Analyst," "IT Administrator." It bundles all the access profiles from all the systems that person needs.

GlobalTech's "Software Engineer" role includes:

| Access Profile | Source System | Entitlements Included |
|---|---|---|
| GitHub Developer Access | GitHub Enterprise | org-member, team:backend-engineers, team:platform-team |
| Jira Developer Access | Jira | Project role: Developer (Engineering projects) |
| Slack Engineering Access | Slack | workspace:globaltech, channel-group:engineering-channels |
| AWS Developer Access | AWS | groups: developers + s3-dev-access |
| AD Engineering Base | Active Directory | Domain Users, Developers, VPN-Access, Dev-Environment-Access |

When ISC assigns the "Software Engineer" role to Sarah, all five access profiles provision — meaning all entitlements across all five systems get created or assigned automatically.

> 🎯 **KEY POINT:** The role/access profile hierarchy is how ISC translates business identity (job title) into technical access (specific entitlements). Designing this correctly is one of the most important integration engineering tasks. Poorly designed roles lead to over-provisioning, under-provisioning, and governance complexity. Well-designed roles make everything downstream cleaner.

## Lifecycle States: The Trigger Mechanism

ISC uses **lifecycle states** to drive automatic access assignment and removal. Every identity in ISC is in exactly one lifecycle state at any time, and transitions between states trigger provisioning actions.

**Core ISC lifecycle states**:

- **Active**: The person is an active employee. They have all access their role and policies entitle them to.
- **Inactive**: Temporarily non-working status (extended leave, sabbatical). Accounts are suspended or disabled, but not deleted — access is preserved for when they return.
- **Terminated**: Employment has ended. Full deprovisioning triggered.

Beyond these core states, ISC supports custom lifecycle states for organizational needs. GlobalTech has added:
- **Onboarding**: First 30 days. Limited access until onboarding compliance training completed.
- **Leave**: Parental leave or medical leave. AD disabled, most access suspended, but certain communication tools (email) remain active per HR policy.
- **Pre-termination**: Notice period. Access begins wind-down, handoff processes triggered.

Each lifecycle state has an associated **provisioning policy** that defines what access someone in that state should have. Transition between states triggers ISC to evaluate the new state's policy and provision/deprovision accordingly.

> ⚠️ **COMMON MISTAKE:** Not designing lifecycle state transitions carefully. The transition from Active to Leave should suspend certain access but preserve others (email, some collaboration tools). The transition from Leave to Active should restore all suspended access. If the Leave state provisioning policy is too aggressive (disables everything) or the restore logic is missing (access doesn't come back on return), you create either security risk or significant operational disruption.

## Attribute-Based Access Assignment: The Logic Layer

How does ISC decide which role to assign Sarah? Through **access assignment rules** — policy logic that evaluates identity attributes and assigns roles based on what they match.

GlobalTech's access assignment rules (simplified):

```
Rule: Software Engineer Assignment
IF identity.department = "Engineering"
AND identity.jobTitle CONTAINS "Engineer"
AND identity.employmentType IN ["Full-Time", "Contractor"]
AND identity.lifecycleState = "Active"
THEN assign role: "Software Engineer"

Rule: Finance Analyst Assignment
IF identity.department = "Finance"
AND identity.jobTitle CONTAINS "Analyst"
AND identity.employmentType = "Full-Time"
AND identity.lifecycleState = "Active"
THEN assign role: "Finance Analyst"

Rule: Manager Additional Access
IF identity.isManager = true
AND identity.department = "Engineering"
AND identity.lifecycleState = "Active"
THEN assign access profile: "Engineering Manager - Team Reports"
```

When Sarah's Workday record arrives with `department=Engineering`, `jobTitle=Software Engineer`, `employmentType=Full-Time`, ISC evaluates these rules and assigns her the "Software Engineer" role. The role assignment triggers provisioning of all five access profiles. All access provisions automatically.

The power of this approach: the rules run continuously. Every time ISC aggregates Sarah's identity attributes, it re-evaluates the rules. If Sarah gets promoted to "Senior Software Engineer," the title change triggers rule re-evaluation. If there's a "Senior Engineer" role with expanded access, ISC assigns it and provisions the difference. If the title change moves her out of scope for "Software Engineer" but into scope for "Principal Engineer," ISC revokes the old role's access and provisions the new one's.

This is dynamic access management — access stays aligned with the current state of the person's attributes, automatically.

> 💡 **REMEMBER:** Access assignment rules evaluate identity attributes, which come from authoritative sources (Workday). When Workday data is wrong — incorrect department, wrong job code, missing attributes — the wrong rules fire and the wrong access provisions. This is another way that HR data quality directly impacts governance quality. An engineer in the wrong Workday department could be assigned the wrong role and receive access they shouldn't have.

## A Complete Example: Sarah Chen Joins GlobalTech

Let's trace Sarah's complete onboarding through the automatic assignment system:

**Thursday PM**: Workday record created.
- `employeeId`: GT-48291
- `firstName`: Sarah, `lastName`: Chen
- `department`: Engineering
- `jobTitle`: Software Engineer
- `jobCode`: ENG-SW-003
- `manager`: james.wilson@globaltech.com
- `employmentType`: Full-Time
- `lifecycleState`: (will become Active on start date)
- `startDate`: Monday

**Thursday PM, ISC aggregation runs**:
- ISC creates identity record for Sarah
- `lifecycleState` evaluated: `startDate` is in the future → ISC sets state to `Pre-Start`
- Pre-Start policy: create AD account in disabled state (so email/alias reservations work), no other provisioning

**Monday AM, start date**:
- ISC detects `startDate` = today → transitions `lifecycleState` to `Active`
- Access assignment rules evaluate:
  - department=Engineering + jobTitle contains "Engineer" + Full-Time + Active → match "Software Engineer" role
  - No manager flag → no manager-specific profiles
- ISC generates provisioning plan for "Software Engineer" role
- All five access profiles provision:
  - AD account enabled + engineering groups added
  - GitHub account created + teams assigned
  - Jira access provisioned
  - Slack workspace + channels
  - AWS account + groups

**Sarah arrives for her first day**: everything is ready.

Total manual IT interventions: zero (except the Salesforce approval for licensing, which is by policy design).

## Birthright Access vs. Requested Access

The automatic assignment model provisions **birthright access** — the access every person in a role is entitled to from day one. This covers maybe 80% of what someone needs.

The remaining 20% is **requested access** — additional entitlements specific to the person's projects, temporary needs, or niche tools. This goes through the Access Request workflow (ISC's governed request system), requires business justification, and is approved by the appropriate people.

Birthright access is automatic, role-based, and revoked when the role ends. Requested access is individual, project-based, and managed through certifications and expiration dates.

Designing the boundary between birthright and requested is a governance judgment: too much birthright access and you over-provision everyone; too little and the access request queue becomes unmanageable.

## Key Takeaways

- ISC's automatic access assignment uses two key constructs: Access Profiles (entitlements from one system, bundled) and Roles (access profiles from multiple systems, bundled by job function)
- Lifecycle states trigger automatic provisioning/deprovisioning — transitions between states are governance events
- Attribute-based rules evaluate identity attributes (from authoritative sources) and assign roles automatically based on matches
- The system is dynamic: when attributes change, rules re-evaluate and access updates automatically
- Birthright access (role-based, automatic) covers the standard access package; requested access (individual, approved) covers the rest

> 🔗 **CONNECTS TO:** Automatic assignment handles steady-state. But identities change — promotions, department transfers, name changes, departures. The next document covers what happens to access throughout the identity lifecycle when things change.
