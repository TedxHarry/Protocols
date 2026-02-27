# Access Profiles — The Unit of Access Management

Phase 4 covered how ISC reads data from connected systems. Phase 5 covers what ISC does with that data — how it makes governed access decisions, provisions accounts, and ensures ongoing compliance.

Everything in ISC's governance layer builds on one foundational object: the access profile. If you understand access profiles — what they are, what they contain, and how to design them well — you understand the structural unit from which everything else is assembled.

---

## What an Access Profile Is

An access profile is a named, governed bundle of entitlements from a single source system, packaged with the provisioning instructions needed to grant those entitlements to an identity.

Break that down:

**Named**: Access profiles have human-readable names that mean something to business users — "Salesforce Sales Rep", "GitHub Contributor", "ServiceNow ITIL User". Not "SF-PROF-002" or "GH-ENT-WRITE". The name is what managers and employees see when they request access or review certifications.

**Governed**: Access profiles have owners (the person accountable for the access they represent), descriptions (what this access allows someone to do), and request policy (whether it can be self-requested, manager-requested, or is birthright).

**Bundle of entitlements from a single source**: An access profile is always scoped to one source system. It can contain multiple entitlements from that system (multiple AD groups, multiple Salesforce profiles, multiple GitHub teams) — but only from that one system. Cross-system bundles are called roles, not access profiles.

**With provisioning instructions**: The access profile knows how to grant access — which entitlements to add, which account attributes to set, whether to create a new account or modify an existing one.

---

## What Access Profiles Contain

### Source

Every access profile is tied to exactly one source (one connected system). This is a hard constraint — you can't have an access profile that spans AD and Salesforce. If you need cross-system bundling, that's a role (which bundles access profiles from multiple sources).

### Entitlements

The specific entitlements from the source system that this access profile grants:
- AD: one or more security groups
- Salesforce: one or more Profiles or Permission Sets
- GitHub: one or more Teams or Repository access levels
- ServiceNow: one or more Roles

A single access profile can contain multiple entitlements from the same source. "GitHub Contributor" might include membership in both the `engineering-team` and `ci-readers` groups — both needed for the job function, so bundled together.

### Provisioning Policy

The instructions for granting this access. When ISC decides to provision "GitHub Contributor" for a new identity, the provisioning policy specifies:
- Does the identity already have a GitHub account? If yes, add entitlements to existing account. If no, create one.
- What attribute values should the new account have? (Username format, display name, email)
- Which entitlement operations to execute? (Add to `engineering-team`, add to `ci-readers`)

### Owner

The person or group accountable for this access profile. Used in:
- Certification campaigns (entitlement owner certifications route to the access profile owner)
- Access request approvals (owner can be part of the approval chain)
- Governance reporting (who is accountable for this access?)

### Description

A plain-language description of what this access profile allows. This is what a manager sees when a direct report requests access, and what they review during a certification. If the description is technical jargon, expect rubber-stamp approvals — managers approve what they don't understand. Write descriptions for a non-technical audience.

---

## Designing Access Profiles Well

### The Granularity Decision

The most consequential design decision for access profiles is how granular to make them. Two failure modes:

**Too fine-grained**: One access profile per entitlement. Result: 847 access profiles in your catalog, most with names that mean nothing to non-technical users. Access requests become overwhelming — users see a wall of options. Certifications become tedious — reviewers approve individual AD groups without understanding what they collectively allow.

**Too coarse-grained**: One "Engineering Access" profile that grants 40 AD groups, 3 GitHub teams, write access to all repos, and JIRA admin. Result: over-provisioning. Everyone in engineering gets everything whether they need it or not. Certifications rubber-stamp access that's never individually reviewed.

**The right level**: An access profile should represent a coherent job function or access tier — what someone in a specific role actually needs to do their work in that system.

GlobalTech's GitHub access profiles:
- `GitHub Read-Only` — can clone any repo; cannot push. For contractors, auditors, compliance reviewers.
- `GitHub Contributor` — push to feature branches, create PRs, can't merge to main. For all developers.
- `GitHub Maintainer` — merge PRs, manage issues, configure branch protection. For senior engineers.
- `GitHub Admin` — full repo admin, manage access, configure webhooks. For DevOps team.

Four profiles cover the meaningful access tiers. Not one profile per team. Not one profile per permission. Four — and each one has a name that a non-technical manager can understand when certifying.

> 🎯 **KEY POINT:** The test for access profile granularity: can a manager explain in one sentence what this access allows, and does approving it mean something substantive? If yes, it's well-sized. If a manager has to read five screens of technical details to understand what they're approving, it's too granular.

### Access Profile Naming Conventions

Establish a naming convention before creating any access profiles. Once created at scale, renaming is painful.

**Pattern**: `[System] [Access Tier]` or `[System] [Function]`

Examples:
- `Salesforce Sales Rep`
- `Salesforce Sales Manager`
- `GitHub Contributor`
- `ServiceNow ITIL User`
- `ServiceNow IT Admin`
- `AWS Developer`
- `AWS Cloud Admin`

Avoid:
- Technical codes: `SF-PROF-002`, `AD-GRP-2849`
- Ambiguous terms: `Salesforce Standard`, `GitHub Access`
- System abbreviations non-users won't recognize: `SNOW-ITIL-U`

### Ownership Assignment

Every access profile needs an owner before it goes into production. An orphaned access profile (no owner) generates open items in governance reporting and means no one is accountable for certifying it.

Ownership rules of thumb:
- Application/system access profiles → owned by the application owner or IT team
- Role-based access profiles → owned by the business manager of that function
- Privileged access profiles → owned by the security or IT operations team

For GlobalTech's 150 access profiles, IT set up an ownership review as part of the initial rollout — every profile required a designated owner before it could be activated for governance operations.

---

## Access Profiles vs Direct Entitlement Assignment

ISC supports both access profiles and direct entitlement assignment. Understanding when to use each prevents a common design mistake.

**Direct entitlement assignment**: Assign a specific group or permission directly to an identity, without wrapping it in an access profile.

**When direct assignment makes sense**: Highly specific, one-off entitlements that don't represent a coherent access pattern and won't need regular certification as a unit. Emergency access grants. Unique individual exceptions.

**When access profiles are required**: Any access that you want to appear in the access request catalog, participate in certifications as a discrete item, be assigned via roles, or have an accountable owner. This is the vast majority of enterprise access.

> ⚠️ **COMMON MISTAKE:** Assigning entitlements directly instead of through access profiles because it's faster to set up. The result: certifications can't review the access as a unit, the access request catalog doesn't expose it, role assignments can't include it, and there's no ownership trail. Direct assignment works, but it bypasses the governance layer. Use access profiles for any access that needs governed lifecycle management.

---

## Access Profiles in the Request Workflow

When an employee requests access in ISC, they browse the access catalog — which surfaces access profiles (not raw entitlements). The request goes through an approval workflow (defined on the access profile), and on approval, the provisioning policy executes.

This flow works well when:
- Access profile names and descriptions are clear enough for employees to find what they need
- Approval workflows match the organization's actual authority model (not every access profile should require the CISO's approval)
- Provisioning policies are tested and handle edge cases (account already exists, account in a different state)

---

## Key Takeaways

- An access profile is the atomic unit of ISC governance — a named bundle of entitlements from one source with provisioning instructions and an owner
- Granularity: match access profiles to coherent job functions or access tiers, not individual entitlements; 4 well-named profiles beat 40 cryptic ones
- Every access profile needs a plain-language description that a non-technical manager can use when approving or certifying
- Assign owners before production — orphaned access profiles are a governance gap
- Direct entitlement assignment bypasses governance — use access profiles for any access that needs lifecycle management
- Naming conventions set early save significant pain later — `[System] [Function]` is clear and consistent

> 🔗 **CONNECTS TO:** Access profiles are the atoms. Roles are the molecules — bundles of access profiles across multiple systems that represent complete job functions. The next document covers how roles work, when to use them, and how to build a role model that doesn't collapse under its own complexity.
