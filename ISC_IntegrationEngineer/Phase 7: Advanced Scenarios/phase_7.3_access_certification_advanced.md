# Access Certifications at Scale — Compliance, Automation, and Making Reviews Stick

Phase 5.5 covered how to design individual certification campaigns. This document addresses the harder problem: running a certification program across an enterprise — dozens of campaigns, hundreds of reviewers, quarterly compliance cycles — in a way that actually reduces risk rather than generating paperwork.

The difference between a certification program that works and one that doesn't is almost never the technology. It's campaign design, reviewer experience, remediation follow-through, and the governance culture that leadership reinforces. ISC provides the tools; integration engineers and governance teams determine whether those tools produce real security outcomes.

---

## The Certification Lifecycle: Planning Through Evidence

Enterprise certification programs operate in cycles — typically quarterly for financial and privileged access, annually for general corporate access. Each cycle follows a lifecycle:

**Planning** (2–4 weeks before campaign launch):
- Confirm campaign scope (which systems, which identities, which access types)
- Confirm reviewer list is current (manager changes since last cycle affect routing)
- Verify remediation queue from the previous cycle is fully resolved
- Update access profile descriptions (any new entitlements need descriptions before reviewers see them)
- Schedule campaign to avoid conflicts (quarter-end, major product launches, holiday periods)

**Launch**:
- ISC generates all certification items based on current access data
- Reviewer notifications sent with clear instructions and deadline
- Security / compliance team monitors campaign progress daily

**Active Review Period**:
- Track completion rates by reviewer — identify who is falling behind and escalate
- Answer reviewer questions (the #1 question: "What does this access actually do?")
- Handle reviewer changes (manager change mid-campaign — re-route items to new manager)

**Deadline and Escalation**:
- Uncompleted items auto-escalate to manager's manager per policy
- OR auto-revoke fires (for high-risk access with auto-revoke enabled)
- Final completion report generated

**Remediation**:
- All revoke decisions trigger deprovisioning (automatic for connected systems, manual for service desk systems)
- Track remediation completion — a revoke decision that isn't acted on is as good as an approve decision
- Remediation SLA: all revocations completed within 5 business days (typical policy)

**Evidence Collection**:
- Compliance team exports campaign completion reports for audit evidence
- Evidence package includes: campaign scope, reviewer assignments, decisions made, remediation completion dates

---

## Scaling Reviewer Experience

At scale, reviewer fatigue is the primary risk to certification quality. A manager with 12 direct reports who each have 25 access profiles faces 300 review items. Doing this quarterly means 1,200 review items per year on top of their actual job.

**Strategies to keep reviews meaningful at scale**:

**Scope stratification**: Don't put all access in every campaign. Create three tiers:
- Tier 1 (quarterly): Privileged access, financial system access, security tool access — high-risk, small scope
- Tier 2 (semi-annual): Elevated SaaS permissions, cross-functional system access
- Tier 3 (annual): Standard role-based access that everyone in a function gets

A manager reviews Tier 1 quarterly (maybe 5–10 items) and Tier 3 annually (all items, once a year). The quarterly burden is manageable; the annual review is comprehensive.

**Risk-sorting review queues**: ISC can sort certification items by risk score — items that the AI flags as potentially inappropriate appear first. Reviewers who start at the top of the queue see the items most worth their attention. Even if they don't complete everything, the high-risk items get reviewed.

**Pre-populating with AI recommendations**: ISC's access recommendations (based on peer group analysis) pre-populate each item with a suggested decision. Reviewers see: "Recommended: Approve (85% of peers in this role have this access)" or "Recommended: Review (only 12% of peers in this role have this access)". This converts blank-slate review into confirmation or investigation — significantly faster for clearly appropriate access.

**Contextual descriptions**: Write access profile descriptions that answer the question a manager will ask: "Can I approve this?" A good description: "GitHub Maintainer — Allows merging pull requests and managing branch protection rules on all engineering repositories. Appropriate for senior engineers who own code review quality." A bad description: "Git group membership Level 3."

> 🎯 **KEY POINT:** The access profile description is written once by an integration engineer and read by hundreds of reviewers over multiple certification cycles. Spending 30 minutes writing a good description saves hours of reviewer time and reviewer-asking-IT-team time. Treat descriptions as a deliverable, not an afterthought.

---

## Regulatory Certification Requirements

Different compliance frameworks have specific certification requirements. ISC's campaign configuration must satisfy these exactly:

**SOX (Sarbanes-Oxley)**:
- Annual certification of access to financial reporting systems required
- Semi-annual for privileged access and admin accounts
- Evidence must show: who reviewed, what was reviewed, decisions made, revocations completed
- ISC audit trail: campaigns → completion reports → remediation records

**SOC 2 (Trust Services Criteria)**:
- Periodic review of logical access (frequency varies — quarterly is common)
- Must demonstrate that access reviews are performed and that inappropriate access is removed
- ISC campaign completion reports + remediation records = SOC 2 audit evidence

**PCI DSS**:
- At least annual review of user access, at least semi-annual for privileged access
- Must cover all systems in the cardholder data environment (CDE)
- Campaign scope must explicitly include all CDE systems — demonstrate to auditors which systems are in scope

**HIPAA**:
- Covered entities must review access to PHI systems regularly (frequency not specified — quarterly is industry standard)
- Workforce access reviews must be documented

**Configuring ISC campaigns for regulatory evidence**:
- Campaign names should reference the regulation and period: `SOX Q3 2025 Financial Systems Access Review`
- Scope explicitly defined and documented before campaign launch
- Campaign reports retained per your organization's record retention policy
- Remediation completion timestamps captured — auditors often ask "how long after the review was inappropriate access removed?"

---

## Certification for Privileged Access — Special Requirements

Standard certifications review role-based access. Privileged access certification is more demanding — the accounts being certified have higher blast radius if compromised, and reviewers need to understand elevated privilege, not just functional access.

**What's different about privileged access certification**:

**More frequent**: Quarterly at minimum for admin accounts, IT service accounts, and domain admin access. Monthly for root/super-admin access.

**Different reviewer pool**: Privileged access should be reviewed by security team or IT management — not by the employee's functional manager, who may not understand AD domain admin or CyberArk privileged session management.

**Owner + security review**: Many organizations require two reviewers for privileged access: the account owner (or their manager) AND a security team reviewer. ISC's certification campaigns support multi-reviewer configurations with both reviewers required to approve or the minimum of one to revoke.

**Justification required on certify**: Configure privileged access campaigns to require reviewers to enter a written justification for approve decisions. "Needed for job function" is minimal; "James Rivera manages Active Directory infrastructure for GlobalTech's manufacturing division and requires Domain Admin access for daily operations" is an audit-ready justification.

**CyberArk integration**: Organizations using CyberArk PAM integrate it with ISC so privileged account certifications include CyberArk safe membership and privileged session recording access. ISC certifies who has access to privileged credentials; CyberArk enforces access to use them.

---

## Certification Campaign Metrics and Continuous Improvement

Running campaigns is not enough. Measuring their effectiveness and improving them over cycles is what separates a mature certification program from a compliance checkbox.

**Metrics to track per campaign**:

| Metric | Target | Red Flag |
|---|---|---|
| Completion rate (human decisions) | > 90% | < 75% → escalation not working |
| Revocation rate | 2–10% | 0% → rubber-stamping; > 15% → over-provisioning systemic |
| Average time per review item | > 30 seconds | < 15 seconds → not reading descriptions |
| Remediation completion rate | 100% within SLA | Anything below 95% |
| Overdue items at campaign close | < 5% | > 15% → deadline/reminder strategy needs work |

**Campaign-over-campaign trends**:
- Revocation rate dropping over time: good — access is becoming more appropriately provisioned
- Revocation rate static at 0%: bad — access is accumulating, not being reviewed
- Completion rate dropping: bad — reviewer fatigue or competing priorities

**Post-campaign improvement actions**:
After each certification cycle, review the metrics and ask:
- Which reviewers had the lowest completion rates? Do they need training? Better tooling?
- Which access profiles had the highest revocation rates? Is this over-provisioning in the role model?
- Which remediations took the longest? Is the manual remediation process inefficient?

Document the improvement actions and track whether they produced results in the next cycle. This is how a certification program matures — not by running the same process repeatedly, but by measuring outcomes and adjusting.

---

## Handling Reviewer Edge Cases

**Manager gone mid-campaign**: If a reviewer leaves the organization during an active campaign, their items don't complete. ISC allows campaign administrators to reassign reviewer items to another manager. Create a process: operations team reviews open items for departed reviewers weekly and reassigns promptly.

**Reviewer on leave**: Configure each campaign with an escalation reviewer for items not completed. When a manager is on leave, their items escalate to their manager or a designated deputy after X days.

**Identity has no manager**: Identities with null manager attribute have certification items with no assigned reviewer. Detect these before campaign launch: run a pre-campaign report showing identities with null manager and assign a default reviewer (e.g., the identity's department head or the IT governance team).

**Access changed after campaign start**: If access changes after a campaign has launched (new entitlement added, role changed), ISC can be configured to include or exclude mid-campaign changes. Most organizations exclude mid-campaign changes — the certification covers the state at campaign launch, and the new access will be reviewed in the next cycle.

---

## Key Takeaways

- Enterprise certification programs operate in cycles with explicit planning, launch, review, escalation, remediation, and evidence collection phases
- Scope stratification by risk tier (quarterly for privileged, annually for standard) keeps reviewer burden manageable
- AI recommendations and risk-sorted review queues preserve reviewer attention for items that need it most
- Different compliance frameworks (SOX, PCI DSS, SOC 2) have specific frequency and evidence requirements — map campaign configurations to each requirement explicitly
- Privileged access certification needs higher frequency, security team reviewers, and written justifications — different from standard access certification
- Track and improve metrics over certification cycles: completion rate, revocation rate, time per item, remediation rate

> 🔗 **CONNECTS TO:** Certifications review access periodically. Separation of Duties prevents specific dangerous access combinations continuously — in real time, not just at review time. The next document covers how to design and enforce SoD policies in ISC.
