# Five Critical Questions ISC Answers

Every identity governance program, regardless of the platform, exists to answer five specific questions. These aren't theoretical constructs — they're the questions your CISO asks during a security review, your auditor asks during a compliance assessment, and your security operations team asks during an incident.

SailPoint ISC is built around answering these five questions at enterprise scale, continuously, and with an audit trail that proves the answers. Let's work through each one using Marcus Rivera, a Senior Financial Analyst at TechCorp (8,000 employees, publicly traded, SOX-regulated).

## Question 1: Who Has Access?

The most fundamental question. Sounds simple. At scale, it's one of the hardest problems in enterprise IT.

Marcus has been at TechCorp for four years. In that time, he's moved through two departments (from Operations to Finance), participated in three project teams, requested access to various tools, and had some of that access removed — but not all. Right now, nobody at TechCorp can answer the question "what can Marcus access?" without manually querying seven different systems.

**How ISC answers it**: ISC aggregates account and entitlement data from every connected system on a scheduled basis. Every account Marcus has — Active Directory, Salesforce, the company's financial reporting portal, GitHub (from his Operations days), AWS (from a project), Jira, and SAP — is pulled into ISC, correlated to Marcus's identity record, and displayed as his complete access picture.

The answer is always current. If Marcus's entitlements changed last night, this morning's aggregation reflects it. If a new account was discovered that nobody knew Marcus had, aggregation surfaces it.

In ISC's interface, a single identity record for Marcus shows: 7 accounts, 23 active entitlements, 4 policy violations (which we'll get to), and a full history of every access event for the past four years.

> 🎯 **KEY POINT:** The "who has access" question can only be answered continuously if data aggregation is continuous. Organizations that rely on quarterly manual audits are answering the question four times a year, at best. ISC answers it every day — for every identity, across every connected system.

## Question 2: Should They Have It?

Having access established, the governance question becomes: is this access appropriate? Does Marcus *need* everything he has? Does his current role justify his current entitlements?

Marcus currently has:
- `Finance-Analyst` and `Finance-SeniorAnalyst` groups in Active Directory — appropriate for his role
- Salesforce "Standard User" with Finance Reports permission set — appropriate
- GitHub `org-member` with `team: operations-engineering` — he's no longer in Operations. Left over from two years ago.
- AWS `S3-ReadOnly` on production buckets — from a project that ended 14 months ago. No longer needed.
- SAP `Journal-Entry` AND `Journal-Approval` entitlements — **this is a SOX violation**. One person shouldn't be able to both initiate and approve financial transactions.

**How ISC answers it**: Through two mechanisms:

**Access Certifications**: Quarterly, ISC generates a certification campaign. Marcus's manager receives a list of every entitlement Marcus holds and answers: "Does this person still need this?" The GitHub and AWS access, which look obviously stale to Marcus's manager, get revoked during the certification. Automatically, based on the manager's "revoke" decision.

**Policy Enforcement (SoD)**: ISC's policy engine evaluates entitlement combinations against defined rules. The `Journal-Entry + Journal-Approval` combination is a configured SoD policy violation. ISC flags it as a violation, routes it to the compliance team for remediation, and tracks it until it's resolved. Marcus's manager may not have noticed this during the certification — but the policy engine never misses it.

> ⚠️ **COMMON MISTAKE:** Treating certifications and policy enforcement as the same thing. Certifications catch inappropriate access through human review — they depend on managers paying attention. Policy enforcement catches prohibited access combinations automatically, regardless of whether anyone reviews it. Both are necessary; they catch different problems.

## Question 3: How Did They Get It?

When an auditor finds that Marcus has `Journal-Approval` access and wants to know how it was granted, there are two possible situations:

**Without ISC**: Someone emails IT, IT sysadmin adds the entitlement, maybe logs a Jira ticket, maybe doesn't. The auditor gets a Jira ticket from 2022 with minimal context, no approval chain documented, and no way to verify the approval was from an authorized person.

**With ISC**: Every access grant in ISC is a logged event with a full chain of evidence: who requested it, what justification was provided, who approved it (with timestamp), and when it was provisioned. If `Journal-Approval` was granted through a proper access request workflow, there's a complete audit trail. If it was provisioned outside ISC (someone added Marcus to the SAP group directly, bypassing governance), ISC surfaces this as a **rogue entitlement** during the next aggregation.

**How ISC answers it**: The audit log. Every provisioning event, every access request, every approval, every certification decision is logged with user, timestamp, system, entitlement, and business justification. The compliance team can generate a complete access history for Marcus — or any identity — covering the full period of ISC's operation.

For Marcus's `Journal-Approval` entitlement: ISC's audit log shows it was provisioned 18 months ago through an access request workflow, approved by Marcus's previous manager (who approved it for a specific project), but never time-bounded or reviewed since. That's a process failure — but it's a documented, traceable one. The organization knows exactly what happened. That's the difference between a finding that results in remediation and a finding that results in a control deficiency.

> 💡 **REMEMBER:** The audit trail is only complete for access managed through ISC. If someone was granted access directly in a system that bypasses ISC, ISC will see the entitlement during aggregation — but won't have the history of how it was granted. This is why rogue entitlement detection matters: it flags access that exists outside governance.

## Question 4: What Are They Doing With It?

This is the question that shifts identity governance from reactive to proactive. Not just "does Marcus have access?" but "is Marcus using it appropriately?"

Marcus's `S3-ReadOnly` access on production AWS buckets has been noted as stale — he hasn't been on the relevant project for 14 months. But has he *used* it during that time?

**How ISC answers it**: ISC's Activity Insights capability integrates with system activity logs to show whether entitlements are being actively used. If Marcus's AWS role hasn't been invoked in 14 months, ISC surfaces it as "unused" during certification campaigns — giving reviewers concrete evidence for revocation decisions rather than just guesswork.

Beyond individual usage, ISC's AI capabilities identify **behavioral anomalies**: patterns that deviate from expected activity for a role or peer group. If Marcus suddenly starts accessing financial records he's never touched in four years, ISC's peer analysis compares his behavior to other Senior Financial Analysts. If it's an outlier, it generates a risk alert — not necessarily a policy violation, but a signal worth investigating.

This is one of the clearest differentiators between ISC and older governance platforms: the AI-native risk scoring layer doesn't just track what access exists — it monitors how that access is being used and flags patterns that look wrong.

> 🎯 **KEY POINT:** Activity data transforms access certifications from binary yes/no decisions ("does Marcus need this access?") to evidence-based decisions ("Marcus hasn't used this access in 14 months — should we really keep it?"). Unused access is risk. ISC makes unused access visible.

## Question 5: What Happens When They Leave?

Marcus resigns. Effective Friday.

Without identity governance at TechCorp, here's what typically happens: HR notifies Marcus's manager, who forwards to IT, who opens a ticket, who works through a checklist across multiple systems over several days. The AD account gets disabled Monday. Salesforce gets disabled Wednesday. SAP never gets updated because it's managed by a different team that wasn't on the notification chain. Eight months later, Marcus's SAP credentials are still valid.

**How ISC answers it**: ISC integrates with Workday (TechCorp's HR system) as an authoritative source. The moment Marcus's employment status changes to "Terminated" in Workday, ISC triggers a lifecycle event. The termination workflow executes automatically:

1. All Active Directory accounts disabled — immediately
2. All Salesforce access revoked — immediately
3. All GitHub access removed — immediately
4. All AWS access revoked — immediately
5. All Jira access disabled — immediately
6. SAP account disabled — immediately (assuming SAP is a connected system)
7. A termination report generated and sent to the security team
8. A certification campaign triggered for Marcus's manager to formally verify all access is removed

Total time from Workday status change to all accounts disabled: under 15 minutes. No manual steps. No forgotten systems. No 8-month window of active credentials for a former employee.

The lifecycle event also handles data retention considerations: flagging accounts for archival rather than deletion where required, preserving audit logs, and documenting the deprovisioning chain for compliance purposes.

> ⚠️ **COMMON MISTAKE:** Assuming deprovisioning is complete when the AD account is disabled. AD is often the most visible system — but it's rarely the only one. A user with a disabled AD account but active SaaS credentials (Salesforce, GitHub, Slack) is still a security risk. ISC's value in deprovisioning is specifically that it handles ALL connected systems simultaneously, not just the obvious ones.

## How the Five Questions Connect

These five questions aren't independent. They form a continuous governance loop:

1. **Who has access?** → Aggregation establishes baseline
2. **Should they have it?** → Certifications and policies audit the baseline
3. **How did they get it?** → Audit logs explain the history
4. **What are they doing with it?** → Activity monitoring identifies risk
5. **What happens when they leave?** → Lifecycle management closes the loop

Each question builds on the previous. You can't certify access you haven't aggregated. You can't audit history you haven't logged. You can't monitor activity on access you haven't governed. Identity governance is the practice of answering all five questions, continuously and completely.

## Key Takeaways

- The five governance questions are the lens through which to evaluate any ISC capability or integration design
- "Who has access?" requires continuous aggregation from all connected systems — not quarterly snapshots
- "Should they have it?" is answered through both human review (certifications) and automated enforcement (SoD policies)
- "How did they get it?" depends on access being managed *through* ISC — bypasses create gaps in the audit trail
- "What happens when they leave?" is the highest-stakes question — ISC's automated termination workflow closes this risk in minutes, not days

> 🔗 **CONNECTS TO:** You now understand the problem, the governance framework, the seven functions, and the platform. The next document explains the specific advantages of ISC's cloud architecture — what actually changes operationally when identity governance moves to SaaS.
