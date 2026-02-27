# Access Certifications — Designing Reviews That Mean Something

Access certifications are periodic reviews where designated reviewers — managers, application owners, security teams — confirm that every identity's access is still appropriate. They're the mechanism by which ISC moves from "we provisioned access at hire time" to "we continuously verify that access remains justified."

Certifications are required by virtually every compliance framework: SOX, HIPAA, SOC 2, ISO 27001, PCI DSS. They're also where ISC's value is most visible to auditors — you can show not just what access exists, but that qualified humans have reviewed and confirmed it on a schedule.

Done well, certifications catch access that should have been removed, surface roles that have drifted from their intended scope, and create a defensible audit trail. Done poorly, they produce 95% rubber-stamp approvals that tick a compliance checkbox while catching nothing.

This document is about designing certifications that catch things.

---

## What Happens in a Certification Campaign

1. An ISC administrator creates a certification campaign with defined scope and reviewers
2. ISC generates certification items — one per identity-entitlement or identity-role pair in scope
3. Reviewers receive notifications with a deadline
4. Reviewers open the campaign, see their assigned items, and for each one: **Approve** (access is still appropriate) or **Revoke** (access should be removed)
5. Revocation decisions trigger remediation — ISC automatically removes the access from the target system
6. Campaign closes at the deadline; uncompleted items escalate or are auto-decided per policy
7. ISC generates a campaign completion report — who reviewed what, decisions made, access revoked

---

## Campaign Types

ISC provides four campaign types, each suited to different governance questions:

### Manager Campaign

**Scope**: A manager reviews all access held by their direct reports.

**Governance question answered**: Is the access my team members have still appropriate for their current roles?

**When to use**: Regular enterprise-wide access review, typically quarterly or annually. Most organizations run at least one manager campaign per year for compliance.

**Who reviews**: Each employee's manager as defined in ISC (manager attribute from HR source).

**Practical size**: Manager campaigns can generate thousands of review items. A manager with 15 direct reports each holding 20 access profiles = 300 items to review. That's too many for meaningful review in one sitting. Scope controls help (see below).

### Entitlement Owner Campaign

**Scope**: An entitlement owner reviews all identities who have a specific entitlement (or set of entitlements).

**Governance question answered**: Is every person who has access to this resource still authorized to have it?

**When to use**: High-sensitivity systems (production databases, financial systems, privileged access), regulatory review requirements for specific applications, post-security incident review.

**Who reviews**: The designated owner of the access profile or entitlement — typically the application owner or system administrator.

**Practical advantage**: Entitlement owners often know their application users well. A Salesforce admin reviewing who has Salesforce Admin access is more likely to catch inappropriate access than a manager reviewing the same item in a long list.

### Role Membership Campaign

**Scope**: A role owner reviews all identities assigned to a specific role.

**Governance question answered**: Is everyone assigned to this role still appropriate for it?

**When to use**: After role changes, during access model rationalization, for high-privilege roles with broad access.

**Who reviews**: The role owner.

### Advanced Campaign (Custom Scope)

**Scope**: Custom filter — any combination of identity attributes, access profile membership, entitlement type, or source system.

**Governance question answered**: Any governance question that the standard campaign types don't address.

**Common uses**: All contractors' access (regardless of manager), all identities with access to a specific system (regardless of what access), all identities added to a high-privilege group in the last 90 days.

---

## Designing Campaigns That Catch Real Problems

The rubber-stamp problem is the most common failure mode for enterprise certification programs. Managers receive 400 items to review in 2 weeks, with deadlines that conflict with budget cycle and no guidance on what they're actually reviewing. They click Approve on everything to close the queue.

To design certifications that produce meaningful decisions:

### 1. Scope to What's Governable in the Review Window

Don't ask a manager to review 400 items if they have 10 business days. A meaningful review takes at least 30–60 seconds per item. 400 items × 45 seconds = 5 hours of active review time for one manager, on top of their regular job.

**Scope controls that help**:
- Limit to high-sensitivity access profiles (exclude low-risk read-only access from manager campaigns)
- Run entitlement owner campaigns for critical applications instead of including them in the manager campaign
- Break large campaigns into smaller, focused campaigns on a rolling schedule rather than one massive annual review

### 2. Make Descriptions Reviewable

The most common reason for rubber-stamp approvals: reviewers don't understand what they're approving.

**Before**: Campaign item shows "AD Group: `GRP-FIN-GL-PROD-RW`"
**After**: Campaign item shows "Finance - General Ledger Production Write Access: Allows creating and modifying journal entries in the production financial system"

ISC surfaces the access profile description during certification. If descriptions are technical codes or jargon, reviewers have no meaningful basis for a decision. Write descriptions for non-technical audiences. This is the highest-ROI change you can make to an existing certification program.

### 3. Set Meaningful Deadlines and Escalation

A 30-day deadline with no reminder emails and no escalation produces low completion rates. Reviewers deprioritize optional-feeling tasks.

**ISC certification settings**:
- **Reminders**: Configure automatic reminder emails at day 7, day 14, day 25 of a 30-day campaign
- **Escalation**: After deadline, uncompleted items escalate to the reviewer's manager (or to a defined backup reviewer)
- **Auto-revoke**: For security-critical campaigns, configure auto-revoke: uncompleted items past the deadline are treated as revoke decisions. This creates real pressure to complete reviews.

> ⚠️ **COMMON MISTAKE:** Auto-revoke without communication. If reviewers don't know that uncompleted items will be auto-revoked, they'll be surprised when employees lose access they should have kept. Communicate the auto-revoke policy explicitly in the campaign announcement. "If you don't review by [date], the access will be automatically revoked" produces faster completion than any reminder email.

### 4. Use AI-Assisted Recommendations

ISC's AI engine analyzes peer group access patterns and historical certification decisions to generate recommendations for each item: likely appropriate, possibly inappropriate, needs review.

**How to present recommendations**: Surface AI recommendations as a starting point, not a decision. Reviewers who blindly approve AI-recommended items are still rubber-stamping — just with an algorithm as cover. Use recommendations to:
- Pre-sort review queues (suspicious items surfaced first)
- Flag items for closer attention
- Provide a "this person's peers typically have this access" context for decisions

**Recommendation quality improves over time**: The AI trains on certification decision history. Early campaigns with low-quality decisions (rubber stamps) produce poor recommendations. As certifications mature and reviewers make more genuine decisions, recommendation quality improves.

### 5. Granularity of Revocation Decisions

When a reviewer revokes access, ISC needs to know: revoke the specific entitlement? The entire access profile? The role?

**Item-level revocation**: The reviewer sees and decides on each access profile or entitlement independently. Most flexible; can surgically remove specific overprovisioned access while retaining appropriate access.

**Profile-level revocation**: Revoking an access profile removes all entitlements it contains. Appropriate when the whole job function is no longer applicable.

**Role-level revocation**: Revoking a role removes all access profiles the role contained. Use when the identity's entire job function has changed.

Configure the granularity based on what reviewers can meaningfully evaluate. Item-level is most precise but requires the most effort. Role-level is fastest but blunt.

---

## Remediation: What Happens After Revoke

A revoke decision is only valuable if it results in access actually being removed. ISC's remediation options:

**Automatic remediation**: ISC immediately executes the deprovisioning operation in the target system. Access is removed within minutes of the revoke decision. Works for all systems with direct provisioning connectivity.

**Manual remediation**: For systems without direct provisioning (service desk ticketing systems), ISC creates a work item in the remediation queue. An IT administrator sees the request and manually removes the access. ISC tracks whether the manual remediation was completed.

**Remediation tracking**: Every revoke decision creates a remediation record. The campaign completion report shows: which access was revoked, whether remediation was completed, how long remediation took. If manual remediations are left open for weeks, that's a process problem that needs addressing — revoke decisions that don't result in access removal within a reasonable window defeat the purpose of the review.

---

## Campaign Scheduling: Frequency and Timing

**Standard frequency guidelines** (adapt to your compliance requirements):

| Access Type | Review Frequency | Campaign Type |
|---|---|---|
| General access (all employees) | Annually | Manager |
| Elevated / admin access | Quarterly | Entitlement Owner |
| Privileged access (PAM, production) | Monthly | Entitlement Owner |
| New employees (30/60/90 days) | At 30, 60, and 90 days | Manager (scoped to new joiners) |
| Role membership | After role changes, annually | Role Membership |

**Timing considerations**:
- Don't schedule large campaigns during month-end, quarter-end, or budget season — managers are unavailable and completion rates drop
- Stagger campaigns — don't run 5 campaigns simultaneously or reviewers get alert fatigue
- For compliance-driven campaigns (SOX quarterly, PCI annual), schedule to complete before the compliance reporting deadline, not on it

---

## Reading Campaign Results

After a campaign closes, ISC generates a completion report. Key metrics:

**Completion rate**: What percentage of assigned items were reviewed by humans? Below 80% is a red flag — many items were either escalated or auto-decided, not genuinely reviewed.

**Revocation rate**: What percentage of reviewed items were revoked? Consistently 0% means reviewers are rubber-stamping. Consistently 50%+ means scope is wrong — you're including access that shouldn't be in scope, or access assignments are badly managed.

**Median review time per item**: ISC can track how long reviewers spent on each item. Average under 10 seconds per item is rubber-stamping — no one reads the description in 10 seconds. Track this and raise it with reviewers if needed.

**Open remediations**: How many revoke decisions haven't been remediated? Any open remediation is access that should have been removed but hasn't been.

---

## Key Takeaways

- Certifications produce meaningful governance outcomes only when scope, descriptions, deadlines, and escalation are designed thoughtfully — rubber-stamp campaigns tick compliance checkboxes while catching nothing
- Four campaign types: Manager (team access), Entitlement Owner (who has this access), Role Membership (who's in this role), Advanced (custom scope)
- Write access profile descriptions for non-technical reviewers — if they can't understand what they're approving, they'll approve everything
- AI recommendations are a starting point for reviewers, not a substitute for judgment; quality improves as genuine decisions accumulate
- Auto-revoke (uncompleted items automatically revoked at deadline) drives completion — communicate the policy explicitly before enabling it
- Monitor completion rate, revocation rate, review time per item, and open remediations — these metrics reveal whether certifications are working

> 🔗 **CONNECTS TO:** Phase 5 is complete. You understand the full governance layer: access profiles (what access looks like), roles (how access is organized by job function), lifecycle states (when access changes), provisioning policies (how accounts are created), and certifications (how access is reviewed over time). Phase 6 moves into operational concerns — troubleshooting live implementations, SoD policy enforcement, and integrating ISC with the broader security ecosystem.
