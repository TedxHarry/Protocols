# What is Identity Governance?

Before there were identity governance platforms, there were binders. Thick three-ring binders on IT managers' shelves, filled with printouts of who had access to what, signed off by department heads during annual audits. Those binders were the governance system. They answered, imperfectly and always a little out of date, the question that regulators and security teams have always needed answered: *who has access to what, and should they?*

Identity governance is the systematic answer to that question — formalized into policies, enforced through processes, and (in modern organizations) automated through platforms like SailPoint ISC.

## The Three Questions That Define Everything

Every identity governance decision, every policy, every audit finding, and every ISC configuration ultimately comes back to three questions. Understand these and you understand the entire discipline.

### Question 1: Who Has Access?

This sounds trivially simple. Of course you know who has access — you gave it to them.

Except at Meridian Financial (from the previous document), 340 orphaned accounts proved that you don't always know. And across their 47 systems, with accounts stored in separate databases, managed by separate teams, with separate naming conventions, the answer to "who has access to the trading platform right now?" required a manual effort spanning three departments and four days.

**Identity governance** starts by creating a unified answer to this question — a single, authoritative, continuously-updated inventory of every identity in the organization and every access point that identity holds. Not a quarterly spreadsheet. A live system.

This requires aggregating account data from every connected system and linking it back to the person behind the account. In ISC, this is the identity record: a single view of a person that shows all their accounts, across all systems, updated continuously.

> 🎯 **KEY POINT:** "Who has access" sounds simple until you multiply 5,000 employees by 47 systems and account for role changes, department transfers, acquisitions, contractor relationships, and service accounts. The answer is only knowable at scale through automation.

### Question 2: What Can They Access?

Knowing *that* someone has an account on a system isn't enough. The critical question is *what that account can do*.

In Active Directory, having an account is table stakes — almost everyone has one. What matters is which **groups** that account belongs to. `Domain Users` is benign. `Finance-Admin` means the person can read salary data. `IT-Domain-Admins` means they can reset anyone's password and access every server.

This is the entitlement layer. An entitlement is the specific permission, group membership, or role that grants actual capabilities. And entitlements are where the real governance work happens.

Consider the difference between two members of Meridian's Engineering team:
- **Rafael** is a junior developer. His Active Directory account has: `Domain Users`, `VPN-Access`, `GitHub-ReadOnly`, `Dev-Environment-Access`
- **Priya** is a senior infrastructure engineer. Her AD account has: `Domain Users`, `VPN-Access`, `GitHub-Admin`, `Production-Server-Access`, `Database-Admin`, `Security-Groups-Modify`

Both have "Engineering" accounts. But Priya can deploy to production, administer databases, and modify security group memberships. Rafael cannot. An access review that only checks "does this person have an account?" misses everything important. You need to know *which entitlements* that account carries.

> 💡 **REMEMBER:** Governance operates at the entitlement level, not just the account level. Two people can have accounts on the same system with wildly different actual access. ISC tracks and governs entitlements — the specific permissions that grant real capabilities.

### Question 3: Why Do They Have It?

This is the governance question that most manual approaches never actually answer — and it's the one regulators care most about.

Why does Priya have `Database-Admin` access? Was it properly requested and approved? Does she still need it for her current role? Was it granted following a documented process with an authorized approver?

These questions matter for several reasons:

**Compliance**: SOX, SOC 2, HIPAA, PCI-DSS, and most major regulatory frameworks require that access be granted through documented, authorized processes. "Because IT did it" is not a defensible answer to an auditor.

**Security**: Access that exists for unclear reasons is access that might not need to exist. If Priya had `Database-Admin` because she was working on a project two years ago and nobody removed it, that's a security risk — not because Priya is malicious, but because every unnecessary privilege is unnecessary attack surface.

**Segregation of duties**: Some combinations of access are inherently risky. If one person can both enter and approve financial transactions, that's a control violation — regardless of whether they've ever abused it. The "why" question catches these combinations.

> ⚠️ **COMMON MISTAKE:** Treating access certification as a rubber-stamp exercise. If managers are reviewing 200 access items in a quarterly certification and clicking "approve" on everything because it's faster than actually investigating, you're checking a compliance box without actually governing. Real governance requires that "why" be answerable with specifics.

## Real Governance in Practice: Finance vs. Engineering

The governance model looks completely different depending on the business context. Identity governance isn't one-size-fits-all — it's implemented based on the organization's risk profile, regulatory requirements, and operational needs.

**Finance team governance at Meridian:**
- Access to financial systems requires documented approval from both the direct manager AND the CFO's office
- No single person can have both "journal entry" and "journal approval" entitlements (SOX segregation of duties)
- Quarterly certifications: every entitlement reviewed and re-approved every 90 days
- Any change to finance system access triggers an audit log entry and notification to the compliance team
- Contractors are prohibited from certain financial entitlements regardless of role

**Engineering team governance at Meridian:**
- Access to development environments is auto-provisioned based on job title (friction would slow development)
- Production system access requires manager + security team approval
- Annual certifications (lower risk profile than finance)
- Service accounts tracked separately with documented business justification
- Privileged access (admin rights) triggers quarterly review regardless of team

The same three questions — who, what, why — get answered differently based on the sensitivity of the systems involved and the regulatory context. A governance platform like ISC allows you to encode these different rules into policies that apply automatically, consistently, and continuously.

## From Questions to Framework

Identity governance takes these three questions and operationalizes them through a framework of policies, processes, and controls:

**Policies** define the rules: what access is allowed for which roles, what combinations are prohibited, how often access must be reviewed, who can approve what.

**Processes** implement the policies: how access is requested, how it's approved, how it's provisioned, how it's reviewed, how it's revoked.

**Controls** verify that policies are being followed: automated checks that prevent policy violations, audit logs that record every change, certifications that confirm ongoing appropriateness.

ISC is the technology layer that automates all three. It holds the policies, executes the processes, and enforces the controls — continuously, across every connected system, for every identity in the organization.

> 🎯 **KEY POINT:** Identity governance isn't a product you buy — it's a discipline you implement. ISC is the tool that makes implementing it at scale feasible. Understanding governance as a *framework of questions and answers* (not just software features) is what separates engineers who configure systems from engineers who solve real business problems.

## Why These Three Questions Are Everything in ISC

Every feature in SailPoint ISC maps back to answering these three questions:

| ISC Capability | Question It Answers |
|---|---|
| Identity aggregation & correlation | Who has access? |
| Entitlement management | What can they access? |
| Access certification | Should they still have it (why)? |
| Provisioning & role management | How is access granted (process)? |
| Policy enforcement / SOD | What combinations are prohibited? |
| Audit logs & reporting | Can you prove the why? |

When you're configuring a source connector, you're answering "who has access" for that system. When you're designing an access profile, you're defining "what can they access" in a structured way. When you're setting up a certification campaign, you're implementing the "why" question at scale.

Everything in ISC comes back to these three questions. Keep them in mind and you'll always know *why* you're doing what you're doing — which makes you a better integration engineer and a better partner to the business stakeholders you're serving.

## Key Takeaways

- Identity governance is the systematic, policy-driven approach to answering three questions: who has access, what can they access, and why do they have it
- Governance operates at the entitlement level — specific permissions matter more than just account existence
- Governance rules differ by context: Finance teams need stricter controls than Engineering teams, and ISC allows you to encode those differences into policy
- Every ISC feature maps to answering one of the three governance questions
- The "why" question — documented authorization and ongoing justification — is what regulators actually care about most

> 🔗 **CONNECTS TO:** Now that you understand what governance *is*, the next document maps out the seven specific functions that implement it — and shows how they connect into a complete identity lifecycle.
