# Why Organizations Need Identity Management

Here's a number that should make any IT leader uncomfortable: the average organization takes **197 days** to identify a breach and another 69 days to contain it. In most of those cases, the attacker didn't break in — they logged in. They used legitimate credentials attached to an account that should have been disabled, or had access that was never properly scoped in the first place.

Identity is the perimeter now. And most organizations are managing it with spreadsheets, tribal knowledge, and hope.

## The 5,000-Employee Problem

Imagine Meridian Financial Services: 5,200 employees, offices in six cities, operating under SOX and PCI-DSS compliance. They have 47 different software systems in use — from Active Directory and Salesforce to a proprietary trading platform and a legacy payroll system from 2009.

Here's how they manage access today: a shared IT inbox called `access@meridian.com`.

When someone needs access to something, they email the inbox. IT tickets it in Jira. A sysadmin manually creates the account. When someone leaves, their manager *hopefully* emails HR, who *hopefully* notifies IT, who *hopefully* disables the accounts. In each system. Individually.

This is not a made-up cautionary tale. This is the actual state of identity management at most mid-size organizations right now.

> ⚠️ **COMMON MISTAKE:** Assuming identity chaos is only a problem for large enterprises. A 500-person company with 15 SaaS apps has the same structural problem — just at smaller scale. The risk grows with every new system added.

## The Three Problems Identity Chaos Creates

### 1. Security Risk: Access That Shouldn't Exist

At Meridian, a sysadmin ran a manual audit last quarter. What they found: 340 **orphaned accounts** — active credentials attached to people who no longer work at the company. Twelve of those accounts had admin-level privileges on financial systems.

Some of those people had been gone for over a year.

Every one of those accounts is an open door. Former employees who left on bad terms. Credentials that could be phished from people who no longer check their work email. Attack surface that nobody is watching because nobody knows it exists.

**Least privilege** — the principle that users should have exactly the access they need, nothing more — is impossible to enforce manually at scale. When access is granted ad hoc through an email inbox, it accumulates. Nobody removes the old access when new access is added. Nobody audits what was granted two years ago.

> 🎯 **KEY POINT:** The average employee accumulates access over their tenure but rarely has it removed. By year three, most employees have three times more access than they actually need. This is called **access sprawl**, and it's endemic in organizations without identity governance.

### 2. Compliance Risk: The Audit You Can't Pass

Meridian is under SOX compliance. That means auditors need to verify that only authorized personnel have access to financial systems — and that access was granted through an approved process with documented authorization.

Current state: access was granted via email threads that may or may not be archived. Some approvals are Slack messages. Some are verbal. The audit trail is a patchwork of Jira tickets and email forwards.

Last year's audit cost Meridian 600 hours of IT staff time manually compiling access reports. They still had 23 findings. Two were material weaknesses.

Regulators increasingly require organizations to demonstrate **continuous control** — not just point-in-time snapshots. That means ongoing visibility into who has access, regular reviews (certifications), and documented remediation when access is inappropriate. Manual processes simply cannot provide this at scale.

**Separation of duties** is another compliance landmine. SOX requires that no single person can both initiate and approve financial transactions. If your sysadmin doesn't know that David in Finance has both the "Accounts Payable" and "Payment Approval" roles, they can't prevent that toxic combination from existing.

### 3. Operational Inefficiency: The Onboarding Tax

A new employee at Meridian waits an average of 4.3 days before they have all the access they need to do their job. During that time, they're either idle or borrowing credentials from colleagues — which creates its own security problems.

Each new hire requires manual tickets across an average of 11 systems. IT handles roughly 600 of these per year, plus role changes, plus offboarding. That's thousands of manual access operations, each one a potential mistake.

The flip side: when someone leaves, the offboarding process takes an average of 12 days. Which means for 12 days, a departing employee — whether they left voluntarily or were terminated — has active credentials on company systems.

> 💡 **REMEMBER:** The operational cost of manual identity management isn't just the IT hours. It's productivity loss (new employees waiting for access), security incidents (orphaned accounts), and compliance fines. Organizations that implement proper identity governance typically report 70% faster onboarding and 90% reduction in orphaned accounts within the first year.

## Why "Just Being More Careful" Doesn't Scale

The obvious response to all of this is: "We'll just be more disciplined about our manual process." Set up better checklists. Send reminder emails when someone leaves. Do quarterly audits.

This works until it doesn't. And it stops working at a predictable point: the moment the organization grows faster than the IT team.

Meridian added 400 employees last year through an acquisition. The IT team stayed flat. The access backlog tripled in three months. Three different sysadmins developed their own variations of the access granting process because the "official" process couldn't handle the volume.

The math is straightforward: manual identity management is O(n) work — it scales linearly with headcount. But the *risk* from poor identity management scales faster than that, because every new employee adds accounts on every system, every system adds new entitlements, and the combinations multiply.

At 100 employees with 5 systems, you can manually track who has what. At 1,000 employees with 20 systems, you cannot. At 5,000 employees with 47 systems, the problem is structurally unsolvable without automation.

> ⚠️ **COMMON MISTAKE:** Thinking a better ticketing system or a more detailed spreadsheet solves the identity management problem. Tools that make manual processes faster are still manual processes. The problem isn't speed — it's the fundamental inability of humans to continuously track, review, and enforce access policies across dozens of systems at scale.

## What Needs to Change

What Meridian actually needs isn't a better inbox system. They need:

- **Automated provisioning**: when HR says someone is hired, all their accounts are created automatically based on their role
- **Automated deprovisioning**: when HR says someone is terminated, all accounts are disabled immediately — not after a 12-day manual process
- **Continuous visibility**: a single system that shows who has access to what, across all 47 systems, right now
- **Access certification**: regular automated reviews that ask managers "does this person still need this access?" and enforce the answers
- **Policy enforcement**: rules that prevent toxic combinations like "approve + execute financial transactions"

This is **identity governance**. And this is the problem SailPoint Identity Security Cloud was built to solve.

## Key Takeaways

- Organizations managing identity manually face three compounding problems: security risk (orphaned accounts, excess access), compliance risk (no audit trail, no continuous control), and operational inefficiency (slow onboarding, manual overhead)
- Access sprawl is structural — it's a predictable outcome of granting access without systematic removal
- Manual identity management cannot scale past a threshold that most growing organizations hit within a few years
- The solution isn't better manual processes — it's automation built around a clear governance model

> 🔗 **CONNECTS TO:** Now that you understand why the problem exists, the next document defines exactly what "identity governance" means — and the three critical questions that every governance framework must answer.
