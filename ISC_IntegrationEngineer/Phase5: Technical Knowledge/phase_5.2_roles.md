# Roles — Building the Access Model That Reflects the Business

An access profile governs one system. A role governs a job function.

When a new software engineer joins GlobalTech, they need: an AD account in the correct OUs, GitHub Contributor access, Jira Software access, Confluence edit rights, Slack workspace membership, and read-only AWS access for their team's environment. That's six access profiles across six systems. A manager shouldn't have to request each one individually. An access certification shouldn't review each one as an independent item when they all come with the job.

Roles bundle access profiles into a coherent unit that mirrors how the business actually thinks about jobs, functions, and access tiers.

---

## What a Role Contains

A role in ISC has:

**Access profiles**: One or more access profiles, each from potentially different sources. The "Software Engineer" role includes access profiles from AD, GitHub, Jira, Confluence, Slack, and AWS — six sources, bundled into one role.

**Membership criteria**: The conditions under which ISC automatically assigns this role to an identity. `department = "Engineering" AND jobTitle CONTAINS "Engineer" AND employmentType = "Full-Time"`. When these conditions match, the role assignment is automatic (birthright provisioning).

**Type**: IT Role or Business Role — each serves a different purpose (covered below).

**Owner**: The manager or team accountable for this role definition and its contents.

**Requestability**: Whether identities can request this role in addition to (or instead of) automatic assignment.

---

## Role Types: IT Roles vs Business Roles

ISC distinguishes two role types, and confusing them is a common design mistake.

### IT Roles

IT Roles are the technical building blocks — groupings of access profiles that IT defines based on technical function. They're the "how" of access.

**Examples**:
- `Standard Employee Access` — AD account in standard OUs, basic network access, standard endpoint policies
- `VPN Access` — VPN credentials, required firewall group membership
- `Privileged Admin Access` — jump host group, PAM enrollment, audit logging enabled

IT roles are typically assigned automatically based on employment status (Active → gets Standard Employee Access) or by IT-defined criteria. They're not usually something end users browse and request.

### Business Roles

Business Roles reflect job functions as the business understands them — they're the "what" from a people-management perspective. Business roles map to org chart positions, job families, or functional teams.

**Examples**:
- `Software Engineer` — everything an engineer needs across all systems
- `Sales Representative` — CRM access, call recording tools, commission tracking system
- `Finance Analyst` — ERP read access, reporting tools, finance data warehouse
- `IT Administrator` — elevated access across IT systems, PAM, monitoring dashboards

Business roles are named in business language. They're the roles managers and HR recognize. They're what new hire onboarding is based on.

**How IT and Business roles work together**: A "Software Engineer" Business Role might include both the `GitHub Contributor` access profile directly AND the `Standard Employee Access` IT Role (which itself contains multiple access profiles). Business Roles compose IT Roles — the hierarchy lets you maintain common baseline access in one place (IT Roles) while building job-specific access on top (Business Roles).

> 💡 **REMEMBER:** Business Roles talk to business stakeholders. IT Roles talk to IT. When a manager asks "what does a software engineer get?", you point to the Business Role. When the security team asks "what does standard employee access include?", you point to the IT Role. Different audiences, different objects, both valid.

---

## Birthright Roles vs Requestable Roles

### Birthright Roles

A birthright role is assigned automatically when an identity's attributes match the membership criteria — no request, no approval, no human action required. The identity enters ISC (via Workday aggregation, for example), ISC evaluates role membership criteria, and the role assignment and resulting provisioning happen automatically.

Birthright access is appropriate for: access that every person in a job function unambiguously needs from day one. A software engineer unambiguously needs a GitHub account. A finance analyst unambiguously needs read access to the financial reporting system. There's no discretion involved — it comes with the job.

Birthright role membership criteria:
```
Department = "Engineering"
  AND JobTitle CONTAINS "Engineer"
  AND EmploymentType IN ("Full-Time", "Contractor")
  AND LifecycleState = "Active"
```

When these conditions are true, ISC assigns the role. When any condition becomes false (termination → LifecycleState changes to "Inactive"), ISC revokes the role and triggers deprovisioning.

### Requestable Roles

Some access isn't for everyone in a job function — it requires additional justification, manager approval, or elevated security review. Requestable roles appear in the ISC access catalog for employees to request.

Examples at GlobalTech:
- `GitHub Maintainer` — requestable by engineers; requires manager approval
- `AWS Production Access` — requestable by senior engineers; requires manager + security approval
- `Salesforce Admin` — requestable by Salesforce power users; requires IT and business owner approval

Requestable roles go through the approval workflow defined in the role configuration. On approval, provisioning happens automatically — the requester doesn't follow up with each system individually.

> ⚠️ **COMMON MISTAKE:** Making everything birthright. The principle of least privilege says identities should have the minimum access needed for their function — not everything that might ever be useful. If a junior engineer automatically gets AWS production access because "engineers might need it," you've over-provisioned. Birthright should be the baseline that's unambiguously required. Everything else is requestable.

---

## Role Mining: Bottom-Up Role Discovery

Organizations with mature existing access patterns often start role design by asking: what roles do we actually have, based on who has what access today?

ISC's role mining capability analyzes actual entitlement data across connected systems and surfaces access patterns — groups of identities that have similar access. These patterns become role suggestions.

**How role mining works**:
1. ISC analyzes the full access matrix: all identities × all entitlements across all sources
2. Clustering algorithms identify groups of identities that share access patterns
3. ISC surfaces role candidates: "47 identities in the Engineering department all have these 12 entitlements in common"
4. Integration engineers and business stakeholders review the candidates and formalize them as roles

**When role mining is useful**:
- Greenfield ISC deployment at an organization with years of ad-hoc access accumulation
- Post-merger environment where two companies' access models need to be rationalized
- Compliance-driven role cleanup where access has drifted from what was originally intended

**What role mining can't do**: It tells you what access patterns exist. It doesn't tell you whether those patterns are correct. If everyone in Finance has access to the Payroll system because it was granted ad-hoc to 80 people over 5 years, role mining will surface "Finance → Payroll Access" as a role candidate. Whether that's appropriate access is a business decision, not a technical one.

> 🎯 **KEY POINT:** Role mining generates candidates for human review — not finished roles for direct deployment. Every role mining suggestion requires a business stakeholder to confirm: "Yes, every Finance analyst should have this access" or "No, this is access that accumulated inappropriately and should be revoked."

---

## Building the Role Model: Top-Down vs Bottom-Up

Two approaches to building a role model from scratch:

### Top-Down (Business-First)

Start with the org chart. For each job function that exists in the business, define what access that function requires. Work with department managers and application owners to fill in the access profiles for each role.

**Advantages**: Roles reflect how the business thinks about jobs. Role model makes intuitive sense to non-technical stakeholders. Access is scoped to what jobs actually require (no historical drift).

**Challenges**: Time-intensive. Requires sustained engagement from business stakeholders. May miss access patterns that exist in reality but weren't in the job description.

**When to use**: New ISC deployments, organizations with clean org structures, projects where compliance requires demonstrably principled access design.

### Bottom-Up (Mining-First)

Start with existing access data. Use role mining to discover actual access patterns. Rationalize the patterns with business stakeholders — approve patterns that are correct, revoke patterns that are historical accumulation.

**Advantages**: Faster to get initial role coverage. Immediately identifies access that exists but shouldn't. Grounded in reality rather than idealized job descriptions.

**Challenges**: Starts from accumulated access drift, not principled access design. Risk of formalizing access patterns that are actually over-provisioning.

**When to use**: Mature environments with years of access history, post-acquisition rationalization, organizations that need quick governance coverage before they can do detailed role design.

**GlobalTech's approach**: Hybrid. Top-down for the 12 primary business functions (Software Engineer, Sales Rep, Finance Analyst, etc.) — these were well-defined enough to specify from scratch. Bottom-up role mining for the 40 supporting functions where access patterns were unclear — mine, review, rationalize.

---

## Role Explosion: The Design Risk to Avoid

Role explosion is what happens when a role model tries to be too precise — a different role for every combination of department, level, location, and team.

GlobalTech's early design attempt: roles for `Senior Software Engineer - Backend`, `Senior Software Engineer - Frontend`, `Mid-Level Software Engineer - Backend`, `Mid-Level Software Engineer - Frontend`, `Junior Software Engineer - Backend`, `Junior Software Engineer - Frontend`. That's 6 roles before accounting for other departments. Multiply across 15 engineering teams and you have 90 roles for engineering alone — each requiring maintenance when access needs change.

The redesign: `Software Engineer` (covers all seniority levels and specializations) + requestable `GitHub Maintainer` and `AWS Production Access` for the senior-level access that juniors don't get automatically. Three roles instead of 90.

**Role explosion tests**:
- If you have more roles than job titles in your HR system, you've probably over-specified
- If a manager can't explain in one sentence what a role contains, it's too granular
- If updating one policy change requires updating 40 roles, the model needs consolidation

> ⚠️ **COMMON MISTAKE:** Creating a role for every permutation of job attributes. Roles should represent categories of access that the business treats as distinct — not every possible combination of attributes. When in doubt, start coarser and add exceptions than start fine-grained and try to merge.

---

## Key Takeaways

- Roles bundle access profiles across multiple sources into job-function-level governance objects
- IT Roles = technical building blocks; Business Roles = job functions the business recognizes — both are needed, they compose
- Birthright roles provision automatically when membership criteria match; requestable roles go through approval
- Birthright should be the unambiguous minimum for a job function — everything discretionary is requestable
- Role mining discovers existing access patterns; business stakeholders decide which patterns to formalize vs revoke
- Role explosion (too many granular roles) creates maintenance debt — start coarser, add specificity where the business genuinely treats access differently
- Top-down (org-first) and bottom-up (mining-first) approaches both work; hybrid is often most practical

> 🔗 **CONNECTS TO:** Roles define what access identities should have. Lifecycle states determine when they should have it — the conditions that trigger provisioning, suspension, and deprovisioning throughout an identity's time at the organization. The next document covers how ISC models the identity lifecycle.
