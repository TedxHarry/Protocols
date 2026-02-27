# Seven Core Functions That Solve These Problems

Governance is the *what*. These seven functions are the *how*.

Think of identity governance as a problem statement — organizations need to know who has access, what they can do, and why they have it. That's the theory. The seven core functions of identity management are what actually make that theory operational. Each one solves a specific part of the problem. Together, they form a complete system.

The most important thing to understand about these seven functions: they're not a list. They're a lifecycle. Each function flows into the next, tracking an identity from the moment someone joins an organization to the moment they leave — and every change in between.

## Following One Employee Through the Lifecycle

Let's follow David Park, a new Financial Analyst joining Meridian Financial Services on Monday morning. His journey through Meridian's identity lifecycle will touch every one of the seven functions.

---

## Function 1: Provisioning — Creating Access

David's offer is accepted. HR enters him into Workday on Friday. By the time David walks in Monday morning, he has:
- An Active Directory account (`d.park@meridian.com`)
- A Salesforce account with the "Standard User" profile
- Jira access for the Finance project boards
- A Slack account added to `#finance-team` and `#all-company`
- Access to the company's financial reporting portal

Nobody opened a ticket. Nobody manually created these accounts. The moment Workday registered David as an active employee in the Finance department at Analyst level, ISC evaluated the access policies for that role and provisioned everything automatically.

**Provisioning** is the function of creating and configuring access — accounts, group memberships, entitlements — based on who someone is and what their role requires. It's the engine that turns "this person needs access" into actual access, across multiple systems, consistently.

> 🎯 **KEY POINT:** Provisioning isn't just account creation. It's the full configuration of an account — which groups, which roles, which entitlements — based on policy. The difference between a correctly provisioned account and one that's been hastily set up manually is the difference between least privilege and access sprawl.

---

## Function 2: Access Request — Getting More When Needed

Two weeks in, David discovers he needs access to a financial modeling tool that wasn't part of his standard provisioning. His manager uses it, and David needs it for a specific project.

In a mature identity governance setup, David doesn't email IT. He opens the ISC **Access Request** portal, searches for "Financial Modeling Suite," and submits a request with a business justification. The request is automatically routed to his manager for approval, then to the Security team (because the tool handles sensitive financial data), then provisioned automatically once both approvals are recorded.

**Access request** is the governed mechanism for obtaining access that isn't automatically provisioned. It replaces the ad-hoc email inbox with a structured workflow: request → justify → approve → provision → audit trail.

The key word is "governed." Every request is documented. Every approval is recorded. Every provisioning event is logged. When the auditor asks how David got access to the financial modeling tool, there's a complete, timestamped answer.

> 💡 **REMEMBER:** The access request function isn't just about convenience — it's about creating the audit trail that answers the "why" governance question. Every manually-emailed access request that bypasses the governance system is an audit finding waiting to happen.

---

## Function 3: Access Certification — Proving Access Is Still Appropriate

Four months later, David's manager — Jessica — receives a certification campaign in her ISC dashboard. It lists every access entitlement her direct reports currently hold and asks a simple question for each one: *Is this still appropriate?*

David's entries show his standard Finance Analyst provisioning plus the Financial Modeling Suite he requested. Jessica reviews each one. She certifies that all of David's standard access is appropriate. She also notices that a temporary project access entitlement — granted for a three-month initiative that wrapped up last month — is still active. She revokes it during the certification.

**Access certification** (also called access review or recertification) is the periodic process of having accountable parties — managers, application owners, risk teams — review and confirm that existing access is still appropriate. It's governance's answer to access sprawl: systematically reviewing and pruning what exists.

In ISC, certifications are campaign-driven: the platform identifies who needs to review what, delivers the reviews to the right people, tracks completions, sends reminders for outstanding items, and automatically revokes access when a reviewer says it's no longer needed.

> ⚠️ **COMMON MISTAKE:** Scheduling certifications too infrequently. Many organizations do annual certifications. But if an employee gets a temporary entitlement in January and the certification runs in December, that temporary access persisted for 11 months after it was needed. High-risk systems should certify quarterly or continuously.

---

## Function 4: Role Management — Defining What a Job Requires

As Meridian's identity governance matures, the governance team notices that they've provisioned 240 Financial Analysts over the past three years. And they've done it slightly differently every time — different approvers, slightly different entitlement sets, some with extra access that was never cleaned up.

Role management is the function that fixes this by defining, in advance, what access a "Financial Analyst" role requires — and making that definition the standard. When a new Financial Analyst is hired, they get exactly the role definition, no more, no less. When the role definition changes (new system onboarded, entitlement restructured), every person with that role is updated automatically.

**Roles** in ISC are collections of access entitlements grouped by job function. Role management is the ongoing discipline of defining, maintaining, and governing those collections. It's what makes provisioning deterministic instead of ad-hoc.

> 🎯 **KEY POINT:** Roles are where business context and technical access meet. "Financial Analyst" is a business concept. The entitlements it requires — specific AD groups, Salesforce profiles, application roles — are technical realities. Role management is the function that translates business identity into technical access, consistently and at scale.

---

## Function 5: Policy Management — Enforcing the Rules

Six months into his tenure, David gets promoted to Senior Financial Analyst. His new role comes with additional responsibilities — including the ability to initiate financial journal entries. ISC provisions the "Journal Entry" entitlement.

But there's a problem. David also still has the "Journal Approval" entitlement from a temporary assignment last quarter that was never properly cleaned up. These two entitlements together constitute a **Segregation of Duties (SoD) violation** — a single person who can both initiate and approve financial transactions. This is exactly the control failure that SOX exists to prevent.

Policy management is the function that catches this. ISC evaluates entitlement combinations against defined SoD policies. When David's provisioning would create a policy violation, ISC flags it — either blocking the provisioning until the conflict is resolved or generating a violation report for remediation.

**Policy management** encompasses all the rules that govern what access combinations are allowed, what requires special approval, what must be reviewed more frequently, and what's prohibited outright. It's the enforcement mechanism that makes the "why" governance question answerable.

> 💡 **REMEMBER:** SoD policies are only as good as the entitlement data ISC has. If a system isn't connected to ISC, ISC can't evaluate its entitlements against policies. Every connected system increases governance coverage. Every unconnected system is a governance blind spot.

---

## Function 6: Password Management — Maintaining Secure Access

This is often the most user-visible identity function. David forgets his Active Directory password on a Monday morning. Without self-service password reset, he calls the help desk, waits on hold, gets verified, gets a temporary password, then has to change it.

With identity governance, David goes to the ISC portal, verifies his identity through multi-factor authentication, and resets his own password — which propagates automatically to all connected systems that use the same credential.

**Password management** in the identity governance context isn't just reset workflows. It's policy enforcement (complexity requirements, expiration), synchronization across systems, and increasingly, integration with SSO and MFA solutions. ISC can enforce password policies and coordinate with identity providers to maintain consistent authentication standards.

---

## Function 7: Reporting and Analytics — Proving It All Works

A year into David's tenure, Meridian's external auditors request evidence that the organization has effective access controls over its financial systems. The compliance team needs to demonstrate:
- Who has access to financial systems (point in time and historically)
- That access was granted through an approved process
- That access has been periodically reviewed and certified
- That policy violations were identified and remediated

In a manual identity management environment, this takes weeks of data gathering from multiple systems.

In ISC, these reports are generated in minutes. Every provisioning event, every access request, every certification decision, every policy violation and remediation — all logged, all queryable, all reportable.

**Reporting and analytics** is the function that makes governance *visible* and *auditable*. It transforms the operational data generated by the other six functions into evidence for compliance, insight for risk management, and metrics for program improvement.

> 🎯 **KEY POINT:** Reporting isn't a nice-to-have. It's the function that proves every other function is working. If you can't demonstrate that provisioning was policy-based, that certifications happened, and that violations were remediated, governance didn't happen — regardless of what the system did.

---

## The Lifecycle View

| David's Milestone | Function Triggered |
|---|---|
| Hired as Financial Analyst | Provisioning (automatic) |
| Needs modeling tool | Access Request (governed workflow) |
| Q2 review | Access Certification (manager-driven) |
| Excess access found | Policy Management + Remediation |
| Promotion (role change) | Role Management + Reprovisioning |
| SoD violation detected | Policy Management (enforcement) |
| Annual audit | Reporting & Analytics |
| Resignation | Deprovisioning (immediate) |

David's departure — whenever it comes — triggers the final function: **deprovisioning**. The moment HR marks him as terminated in Workday, ISC automatically disables every account across every connected system. Not after 12 days. Not after someone remembers to send an email. Immediately.

That's the lifecycle. That's the seven functions. That's identity governance in practice.

## Key Takeaways

- The seven functions are a lifecycle, not a list — they follow an identity from hire to departure and every change in between
- Provisioning and deprovisioning are the bookends; everything in between maintains appropriate access over time
- Access certification is governance's primary tool against access sprawl — the gradual accumulation of unnecessary access
- Policy management (SoD enforcement) is where business rules become technical controls
- Reporting transforms all governance activity into auditable evidence — critical for compliance

> 🔗 **CONNECTS TO:** Now that you understand the seven functions identity governance requires, the next document introduces SailPoint Identity Security Cloud — the platform that automates all seven.
