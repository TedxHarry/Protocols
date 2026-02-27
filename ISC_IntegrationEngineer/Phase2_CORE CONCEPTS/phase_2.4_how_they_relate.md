# How Identity, Account, and Entitlement Connect

You've learned identity, account, and entitlement as three separate concepts. Now comes the part that matters for integration engineering: understanding how they connect into a single, coherent governance model — and what breaks when the connections fail.

The relationship is a hierarchy. It always follows this direction:

**Identity → (has) Accounts → (which have) Entitlements**

A person (identity) has presence on systems (accounts), and those accounts carry specific permissions (entitlements). Governance operates by understanding the full chain from person to permission — not just one layer in isolation.

## Sarah Chen's Complete Picture

Let's look at the entire model for Sarah Chen, all in one place. This is what her identity record represents in ISC — the complete answer to "what can this person access?":

| System | Account | Key Entitlements | Risk Level |
|---|---|---|---|
| Active Directory | `s.chen` | Domain Users, Developers, VPN-Access, Dev-Environment-Access, GitHub-SSO-Users | Low–Medium |
| Salesforce | `sarah.chen@globaltech.com` | Standard User (profile), Finance_Reports_ReadOnly (perm set), Dev_Sandbox_Access (perm set) | Low |
| GitHub | `schen-dev` | org: globaltech-corp, team: backend-engineers, team: platform-team | Medium |
| AWS | `sarah_chen_dev` | group: developers, group: s3-dev-access, policy: Developer-IAM-Policy | Medium |
| Slack | `sarah.chen` | workspace: globaltech, channel group: engineering-channels | Low |
| Jira | `SChen` | Project role: Developer (Engineering), Project role: Viewer (Finance) | Low |
| SAP Finance | `SCHEN001` | FI-AP-CLERK, FI-REPORTS-READ | Medium |

One identity. Seven accounts. Twenty-three entitlements. This is Sarah's access picture, and ISC maintains it continuously — updating as accounts change, entitlements are granted or revoked, and system data is aggregated.

> 🎯 **KEY POINT:** The three-layer model (identity → accounts → entitlements) is ISC's fundamental data model. Every governance action — certification, policy evaluation, provisioning, reporting — operates on this model. When you build an integration, you're building the pipeline that populates this model with accurate data. The quality of the governance you enable is directly proportional to the accuracy of the data you feed in.

## Answering the Auditor's Question

An auditor walks into GlobalTech's compliance office and asks: "Can you show me exactly what Sarah Chen can access, and verify that it's all appropriate?"

Without ISC: someone spends three days querying seven different systems, exporting data into spreadsheets, trying to map across different identifier schemes, and assembling a coherent picture — which is already out of date by the time it's assembled.

With ISC: the ISC admin pulls Sarah's identity record. In one view, they see all seven accounts, all 23 entitlements, the last certification date for each, who certified it, and whether any policy violations exist. The complete answer takes about four minutes.

Then the harder question: "Is it all appropriate?"

Walking through the model:
- AD entitlements: appropriate for Software Engineer in Engineering department ✓
- Salesforce Standard User + Dev Sandbox: appropriate ✓
- Salesforce Finance_Reports_ReadOnly: Sarah doesn't handle financial analysis — this is a question. Was it requested? Was it approved? The audit log shows: yes, requested 14 months ago for a cross-functional project, approved by Sarah's manager at the time.
- GitHub backend-engineers + platform-team: appropriate ✓
- AWS developer groups: appropriate ✓
- Jira Developer (Engineering): appropriate ✓
- Jira Viewer (Finance): legacy from Operations role, not re-certified since she moved departments — flagged for review ⚠️
- SAP FI-AP-CLERK: Sarah is not in Finance. This entitlement's presence in the model is immediately suspicious. The audit log shows it was provisioned 22 months ago during an emergency situation when Finance was understaffed — and never removed ⚠️

The model surfaces the problems. The audit trail explains them. The certification history shows what was and wasn't reviewed. This is the governance picture ISC provides — because the full identity → account → entitlement chain is populated and maintained.

> ⚠️ **COMMON MISTAKE:** Thinking that a certified entitlement is a correct entitlement. The Jira Finance Viewer role was never reviewed because Sarah's certifications only covered her Engineering-domain entitlements. The certifier (her Engineering manager) may not have visibility into Finance project roles. Certification campaign design matters — who certifies what, how it's scoped, and whether cross-functional entitlements have appropriate reviewers.

## Toxic Combinations: When the Model Reveals Risk

The relationship model doesn't just answer individual access questions — it enables detection of dangerous access combinations that span across accounts.

Consider this scenario: After Sarah's previous manager approved the SAP `FI-AP-CLERK` role for emergency purposes, Sarah also got access to a separate SAP role: `FI-GL-POST-APPROVE` (General Ledger Journal Approval). This might have been an oversight during a system reconfiguration.

Individually:
- `FI-AP-CLERK`: accounts payable clerk functions — reasonable for a temporary Finance support context
- `FI-GL-POST-APPROVE`: approve journal entries — high-risk, SOX-relevant

Together: a person who can both initiate AP transactions and approve GL journal entries. This is a textbook SOD violation.

Neither entitlement looks dangerous in isolation. The risk only appears when you see them together, connected through Sarah's identity. ISC's SoD policy engine evaluates against the full entitlement picture — every entitlement across every account — and flags the combination.

This is why the connected model matters. If ISC only tracked entitlements per-system without linking them through the identity, this cross-system SOD violation would be invisible. The governance value is in seeing the complete picture.

> 🎯 **KEY POINT:** SOD violations often span systems. A person might have "initiate" access on System A and "approve" access on System B. Neither system knows about the other. Only ISC — which aggregates both and links them through the identity — can detect the combination. This is one of the most compelling arguments for broad system coverage in ISC implementations.

## What Breaks When the Connections Fail

The three-layer model requires that each layer accurately maps to the next. When connections break, governance breaks.

**Identity → Account connection breaks (correlation failure)**:
ISC aggregates 8,400 AD accounts. 12 of them don't match any identity in ISC's database. These 12 are "orphaned" or "uncorrelated" accounts. They have entitlements. ISC can see those entitlements. But without the identity connection, ISC doesn't know who those accounts belong to. They can't be included in certifications (no identity = no certifier). They can't be evaluated against person-level SoD policies. They're governance blind spots.

Real consequence: two of those 12 orphaned accounts belong to former employees whose identities were cleaned up but whose AD accounts were missed in deprovisioning. Active accounts, active entitlements, no human owner — exactly the security risk identity governance is supposed to prevent.

**Account → Entitlement connection breaks (aggregation failure)**:
The Jira connector is configured to aggregate accounts but not project roles. Sarah's account exists in ISC. But ISC doesn't know about her project roles — the entitlements aren't aggregated. Her Jira access appears in ISC as "account exists, no known entitlements." Certifications for her Jira access show nothing to certify. The Finance Viewer role is invisible. It never gets reviewed. It never gets revoked.

Real consequence: The entitlement that shouldn't exist — the stale Finance Viewer role from her Operations days — persists indefinitely because it's not in scope for governance.

> 💡 **REMEMBER:** The governance model is only as complete as the data that populates it. Gaps in aggregation create gaps in governance. Every system you connect and every entitlement type you aggregate extends the visibility — and the protection — that ISC provides.

## The Mental Model for Integration Engineers

When you design or debug an ISC integration, always trace through the three layers:

1. **Does the identity exist?** Is there a properly populated identity record? Are the source attributes accurate and current? Is the authoritative source feeding correctly?

2. **Are accounts correlated?** Do all accounts for this identity link back to it? Are there uncorrelated accounts that need correlation rule fixes? Are service accounts excluded correctly?

3. **Are entitlements visible?** Is the connector configured to aggregate the right entitlement types? Are entitlement descriptions populated for governance readability? Are risk levels assigned?

If governance isn't working for a particular person or system, the problem is almost always in one of these three layers. Systematic diagnosis starts here.

## Key Takeaways

- The identity → account → entitlement hierarchy is ISC's fundamental data model — all governance operates on this structure
- The complete picture for any identity spans all their accounts across all connected systems and all entitlements on each
- SOD violations often cross system boundaries — only the connected model that links everything through the identity can detect them
- Correlation failures (uncorrelated accounts) and aggregation gaps (missing entitlements) create governance blind spots
- When governance breaks, trace through all three layers systematically: identity quality, correlation accuracy, entitlement aggregation scope

> 🔗 **CONNECTS TO:** Now that you understand the model, the next document explains how data actually gets into ISC — the aggregation process that reads account and entitlement data from source systems.
