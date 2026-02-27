# Provisioning — How ISC Sends Data TO Systems

Aggregation is ISC reading the world as it is. Provisioning is ISC changing the world to match what it should be.

When HR hires a new employee, ISC doesn't just make a note that this person exists. It takes action — reaching out to Active Directory, Salesforce, GitHub, AWS, and every other connected target system to create accounts, assign group memberships, and set entitlements. That's provisioning.

When someone is terminated, ISC reverses those actions across every system simultaneously. When a role changes, ISC adds the new access and removes the old. Provisioning is how ISC's governance decisions become real changes in real systems.

## The Four Provisioning Operations

ISC executes four distinct operations when provisioning:

**CREATE**: Create a new account on a target system. Happens during onboarding (new employee), when a previously unmanaged system is brought under ISC governance, or when an approved access request includes a system the person doesn't yet have an account on.

**UPDATE**: Modify an existing account's attributes or entitlement assignments. Happens during role changes (add new groups, remove old ones), attribute updates (job title change synced to target system), or when a certification revokes a specific entitlement.

**ENABLE/DISABLE**: Toggle the account's active status without changing its entitlement configuration. Enable happens when an employee returns from leave. Disable happens during offboarding (preserving entitlements for transition/handoff), or when an account is temporarily suspended.

**DELETE**: Remove the account from the target system entirely. Typically the final step in a complete offboarding, after any data retention or handoff requirements are satisfied. Some organizations prefer to disable and archive rather than delete, for compliance reasons.

> 🎯 **KEY POINT:** Disable and delete are not the same operation, and confusing them has real consequences. Disable blocks authentication but preserves all data and entitlements — the account can be re-enabled. Delete removes the account and typically its data from the target system. For regulated data, deletion may be irreversible and have compliance implications. Design deprovisioning workflows with this distinction explicitly in mind.

## The Provisioning Flow: What Happens Step by Step

When ISC needs to provision something, it doesn't just send a command. It creates a **provisioning plan** — a structured representation of exactly what changes need to happen — and executes that plan through the connector.

Here's the complete flow for a new Software Engineer joining GlobalTech:

**Day -3 (Thursday before start date)**:
HR Manager completes Sarah Chen's record in Workday. Sets start date to next Monday, department=Engineering, title=Software Engineer, manager=James Wilson.

**Day -3 (Thursday evening, 8 PM)**:
ISC's nightly Workday aggregation runs. Sarah's new record is detected. ISC creates a new identity record for her.

ISC evaluates access policies against Sarah's attributes:
- `department = Engineering` + `jobTitle = Software Engineer` → match "Software Engineer" role
- "Software Engineer" role = access profile: GitHub Dev, Jira Dev, Slack Engineering, AWS Dev, AD Engineering
- ISC generates a provisioning plan

**The Provisioning Plan**:
```
Identity: Sarah Chen (GT-48291)
Action: Create accounts and assign entitlements

Target: Active Directory
  Operation: CREATE
  Attributes:
    samAccountName: s.chen
    displayName: Sarah Chen
    mail: sarah.chen@globaltech.com
    userPrincipalName: s.chen@globaltech.com
    employeeID: GT-48291
    department: Engineering
    title: Software Engineer
    manager: CN=James Wilson,OU=Engineering,DC=globaltech,DC=com
  Groups to assign: [Domain Users, Developers, VPN-Access, Dev-Environment-Access, GitHub-SSO-Users, Engineering-AllStaff, Austin-Office-Users]

Target: GitHub Enterprise
  Operation: CREATE
  Attributes: login mapping to s.chen identity
  Teams to assign: [backend-engineers, platform-team]

Target: Jira
  Operation: CREATE
  Project roles: Developer on Engineering-Backend, Engineering-Platform

Target: Slack
  Operation: CREATE
  Channel groups: engineering-channels

Target: AWS
  Operation: CREATE
  IAM username: sarah_chen_dev
  Groups: [developers, s3-dev-access]

Target: Salesforce
  Operation: CREATE (REQUIRES APPROVAL — Salesforce licensing requires IT approval)
  Profile: Standard User
  Permission Sets: [Dev_Sandbox_Access]
```

**Day -3 (Thursday evening, 8:30 PM)**:
ISC begins executing the provisioning plan. AD account created first (other systems depend on AD SSO). GitHub, Jira, Slack, AWS provisioned in parallel. Salesforce provisioning request sent to IT approval queue.

**Day -3 (Friday morning)**:
IT approves the Salesforce request. Salesforce account provisioned.

**Monday (Sarah's start date)**:
Sarah receives her AD credentials. All other accounts are already active. Her laptop is configured to authenticate against AD, which SSO-federates to Salesforce, GitHub, Jira, and Slack. She logs in once and has access to everything she needs.

Total elapsed time from Workday record to full access: approximately 12 hours. Manual process (before ISC): average 4.3 days.

> 💡 **REMEMBER:** The provisioning plan is ISC's intent declaration. ISC creates the plan, validates it against policies (no SoD violations, all approvals obtained), then executes it. If a step in the plan fails (e.g., AD is unreachable at 8:30 PM), ISC retries the failed step and tracks the outstanding provisioning obligation. The plan doesn't close until all operations complete successfully.

## Provisioning Approval Workflows

Not all provisioning is automatic. Some target systems, high-risk entitlements, or organizational policies require human approval before ISC executes provisioning.

ISC's workflow engine handles approval routing. The approval configuration determines:
- What triggers an approval requirement (specific entitlement, system, risk level, requester role)
- Who must approve (direct manager, IT team, security team, application owner — or combinations)
- What happens on approval (provisioning executes)
- What happens on rejection (request denied, requester notified, audit log entry created)
- What happens when no response comes (reminder notifications, escalation, auto-reject after deadline)

For GlobalTech's Salesforce system, every new account requires IT approval because Salesforce licenses cost money — IT needs to verify the business justification before incurring the licensing cost. For AD accounts, no approval required — provisioning happens automatically based on role assignment.

This tiered approach is standard: low-risk, role-based access provisions automatically; high-risk or expensive access goes through approval workflows.

> ⚠️ **COMMON MISTAKE:** Designing approval workflows without considering the approval bottleneck. If every access request for a 5,000-person organization requires IT approval, IT becomes a bottleneck and the access request queue backs up. Smart workflow design pushes approvals to the appropriate level: manager approves standard role-based access, IT approves non-standard requests, security approves high-risk entitlements. The goal is governance without friction.

## Failed Provisioning: What ISC Does When Something Goes Wrong

Systems are unavailable. APIs fail. Service accounts get locked. Provisioning failures are inevitable.

When a provisioning operation fails, ISC:

1. **Logs the failure** with error details (error code, timestamp, target system response)
2. **Retries automatically** based on configured retry policy (typically 3 attempts with exponential backoff)
3. **Creates a pending provisioning task** if retries are exhausted — the provisioning obligation isn't abandoned, it's queued for manual resolution
4. **Notifies configured administrators** about the failed provisioning
5. **Maintains the partial state** — if 5 of 6 target systems provisioned successfully and 1 failed, the 5 successful ones remain provisioned while ISC tracks the outstanding obligation

The pending provisioning task is important: it's ISC's guarantee that nothing falls through the cracks. Even if the AD connector goes down for three hours, the obligation to create Sarah's account is tracked in ISC's queue. When connectivity restores, the pending task executes.

For systems without direct connector support, ISC can fall back to **service desk ticketing integration** — generating a ticket in ServiceNow (or similar) that tells the IT team "manually create an account on [legacy system] with these parameters, then mark this ticket resolved." Not ideal, but it keeps the provisioning event tracked and auditable even when automation isn't possible.

> 🎯 **KEY POINT:** ISC provisioning is not fire-and-forget. It tracks every provisioning obligation and maintains state until completion. This tracking is what makes the audit trail reliable — you can always determine whether provisioning completed, when it completed, and if it failed, what happened. A governance system that silently drops failed provisioning events is not a governance system.

## Deprovisioning: The Reverse Flow

Everything in provisioning applies in reverse during deprovisioning, but the stakes are higher.

When Sarah eventually leaves GlobalTech, Workday marks her as terminated. ISC detects the lifecycle change during aggregation (or via event-driven trigger if configured). ISC generates a deprovisioning plan — the inverse of the original provisioning plan.

**Deprovisioning sequence (order matters)**:
1. Disable AD account immediately (blocks SSO to all federated systems)
2. Revoke all entitlements across all accounts (remove group memberships, permission sets, team memberships)
3. Disable or delete accounts on each target system per policy
4. Generate deprovisioning completion report for security team
5. Trigger certification campaign for any access that required manager review
6. Archive identity record per retention policy

The sequence matters because SSO dependencies exist: disabling the AD account effectively blocks access to Salesforce, GitHub, Jira, and Slack (all SSO-federated through AD). But ISC still revokes entitlements explicitly on each system, because some systems have local authentication fallbacks that could bypass SSO.

Total time from Workday termination to all accounts disabled: under 15 minutes (for GlobalTech's ISC implementation). The 4.3-day deprovisioning risk window that existed before ISC is gone.

## Key Takeaways

- Provisioning is the WRITE direction — ISC sends changes to target systems based on governance decisions
- Four operations: CREATE (new account), UPDATE (change attributes/entitlements), ENABLE/DISABLE (toggle access), DELETE (remove account)
- ISC creates a provisioning plan before executing — a structured declaration of intent that is validated, tracked, and audited
- Failed provisioning creates pending tasks, not silent failures — every provisioning obligation is tracked until completion
- Deprovisioning follows the same flow in reverse, with sequence ordering that matters (SSO dependencies, entitlement removal before account deletion)

> 🔗 **CONNECTS TO:** Provisioning executes access changes. But what determines what access someone should get in the first place? That's ISC's automatic access assignment logic — roles, access profiles, and attribute-based policies, covered next.
