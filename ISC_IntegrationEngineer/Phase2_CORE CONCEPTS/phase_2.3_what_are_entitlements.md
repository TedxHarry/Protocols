# What are Entitlements?

Having an account on a system tells you almost nothing about what that person can do.

Sarah Chen has an Active Directory account. So does the receptionist, the CFO, and the network administrator. They all have accounts. But they have completely different capabilities in every system they touch — because they have different **entitlements**.

An entitlement is the specific permission, group membership, role assignment, or policy attachment that grants a user the ability to do something. If an account is Sarah's presence on a system, entitlements are what she's allowed to do once she's there.

This distinction matters enormously for governance. Identity governance operates at the entitlement level, not the account level. Knowing Sarah "has Salesforce" tells you she exists in Salesforce. Knowing Sarah has the "Finance Reports" permission set and the "Sales Cloud Standard" profile tells you what she can actually do — and whether it's appropriate.

## Three Types of Entitlements

Entitlements appear differently depending on the system, but they fall into three general categories:

**Group Memberships** — The most common form in directory-based systems. A user belongs to a group, and the group grants a collection of permissions. The user inherits whatever the group can do.

Example: Active Directory group `CN=Finance-ReadWrite` grants read and write access to the Finance network share, access to the financial reporting portal, and membership in distribution list `finance-team@globaltech.com`. Any user in this group gets all three capabilities.

**Profiles and Roles** — Common in SaaS applications. Instead of groups, the system uses predefined profiles or roles that bundle specific application capabilities.

Example: Salesforce's "Standard User" profile grants access to standard CRM objects (Accounts, Contacts, Opportunities) with read/edit/create/delete permissions. The "System Administrator" profile grants full platform access. These aren't group memberships — they're application-layer permission bundles.

**Permissions and Policy Attachments** — Common in cloud infrastructure platforms. Access is granted by directly attaching policies to a user, role, or group.

Example: AWS IAM policy `S3-ReadWrite-Dev` allows `s3:GetObject`, `s3:PutObject`, `s3:DeleteObject` on the development S3 bucket ARN. Attached to Sarah's IAM user, this policy grants those specific API operations on that specific resource. No other S3 buckets, no other operations.

> 🎯 **KEY POINT:** Entitlements live ON the source systems, not in ISC. ISC reads and tracks them through aggregation, but the actual enforcement happens in the system itself. When ISC "grants" Sarah an entitlement, it's really sending a provisioning instruction to the target system to make the change there. ISC governs — the systems enforce.

## Sarah's Entitlements Across All Seven Systems

Here's the complete picture of Sarah's active entitlements at GlobalTech:

**Active Directory (7 entitlements)**:
- `Domain Users` — baseline membership, all AD accounts have this
- `Developers` — grants access to dev file shares, dev build systems
- `VPN-Access` — grants VPN connectivity for remote access
- `Dev-Environment-Access` — grants access to development environment tools
- `GitHub-SSO-Users` — grants GitHub Enterprise SSO access
- `Engineering-AllStaff` — engineering distribution list + shared calendar
- `Austin-Office-Users` — site-specific resources (printer, local shares)

**Salesforce (3 entitlements)**:
- Profile: `Standard User` — standard CRM access (Accounts, Contacts, Opportunities, Reports)
- Permission Set: `Finance_Reports_ReadOnly` — read-only access to finance dashboards
- Permission Set: `Dev_Sandbox_Access` — access to Salesforce development sandboxes

**GitHub (4 entitlements)**:
- Org membership: `globaltech-corp` — base organization access
- Team: `backend-engineers` — access to backend repositories (read/write)
- Team: `platform-team` — access to infrastructure repositories (read/write)
- Repository role: `globaltech-corp/core-api` — direct repository role (write)

**AWS (3 entitlements)**:
- Group: `developers` — inherits `Developer-Base-Policy` (EC2 describe, CloudWatch read, basic CodeCommit)
- Group: `s3-dev-access` — inherits `S3-Dev-ReadWrite` (dev bucket access)
- Attached Policy: `Developer-IAM-Policy` — specific developer API permissions

**Slack (2 entitlements)**:
- Workspace membership: `globaltech` — base workspace access
- Channel group: `engineering-channels` — auto-added to Engineering-specific channels

**Jira (2 entitlements)**:
- Project role: `Developer` on GlobalTech Engineering projects
- Project role: `Viewer` on Finance projects (legacy from Operations days)

**SAP Finance (2 entitlements)**:
- Role: `FI-AP-CLERK` — accounts payable clerk functions (view invoices, post payments)
- Role: `FI-REPORTS-READ` — read-only access to financial reports

Total: 23 entitlements across 7 systems.

> ⚠️ **COMMON MISTAKE:** Looking at entitlement counts and assuming more means more risky. Context matters. "Domain Users" is an entitlement but it grants almost nothing. "Finance-Admin" in AD could grant the ability to modify the financial system's user database. One high-risk entitlement can be more dangerous than 20 low-risk ones. ISC assigns risk scores to individual entitlements — and those scores feed into identity risk calculations.

## Where Entitlements Live (And Why This Matters)

Entitlements are defined and enforced in source systems. ISC doesn't store or enforce the actual permissions — it reads them through aggregation, tracks them against policies, and manages them through provisioning.

This has a critical implication: ISC can only govern entitlements it knows about. If a system isn't connected to ISC, or if a system's connector isn't configured to read a specific type of entitlement, those entitlements are invisible to governance.

Look at Sarah's Jira entitlements: she has a Viewer role on Finance projects. This is a leftover from when she was in Operations and needed to track Finance project status. Now she's in Engineering and it's stale access. ISC knows about it because GlobalTech configured the Jira connector to aggregate project roles. If ISC only aggregated Jira accounts (not roles), this entitlement would be invisible and ungoverned.

This is why entitlement schema design matters in integration work. When you connect a system to ISC, you decide what to aggregate — accounts, which entitlement types, which attributes. Broader aggregation = more complete governance. But more data also means more governance workload (more items in certifications, more potential violations to evaluate).

The integration engineer's judgment call: what entitlements are worth governing? For high-risk systems (financial, admin-level, HR data), govern everything. For low-risk systems (Slack), maybe just account existence is sufficient.

> 💡 **REMEMBER:** Every entitlement you choose NOT to aggregate is a governance blind spot. If something can grant meaningful access, it should be in scope. The question isn't "do we want to govern this?" — it's "what's the cost of NOT governing it if something goes wrong?"

## Entitlement Descriptions: Making Governance Human-Readable

Raw entitlement names are often cryptic. The AD group `CN=FIN-GL-POSTINGS-APPRVL` means something to the Finance system's creator. It means nothing to Sarah's manager during a certification review.

ISC allows you to enrich entitlements with human-readable descriptions during the integration setup. This is more important than it sounds. During certifications, managers review entitlements and decide whether each person should keep them. If the entitlement says `FIN-GL-POSTINGS-APPRVL`, the manager will probably click "approve" because they don't know what it is. If it says "Finance: General Ledger — Approve Journal Entries (SOD risk — cannot be combined with posting access)," they'll pause and think.

Entitlement description enrichment is often undervalued in integration projects. It takes time but directly improves certification quality — which is the primary ongoing governance mechanism.

## High-Risk Entitlements and Risk Scoring

Not all entitlements carry the same risk. ISC allows organizations to tag entitlements with risk levels (typically Low, Medium, High, Critical), which feed into several governance mechanisms:

- **Identity risk score**: Identities with high-risk entitlements get higher risk scores, surfacing them for more frequent review
- **Certification priority**: High-risk entitlement certifications can be configured to require stricter approval workflows
- **Policy enforcement**: High-risk entitlement combinations trigger SoD violation detection

For Sarah, `FI-AP-CLERK` in SAP is medium risk on its own. `FI-REPORTS-READ` is low risk. But together with `Journal-Entry` (if she somehow had it), they'd form a high-risk combination that ISC's SoD policies would flag.

## Key Takeaways

- Entitlements are the specific permissions (group memberships, profiles, policy attachments) that grant actual capabilities — they're what governance actually manages
- Three main types: group memberships (AD, LDAP), profiles/roles (Salesforce, ServiceNow), permissions/policy attachments (AWS, GCP)
- Entitlements live on source systems; ISC reads them through aggregation and manages them through provisioning, but enforcement happens at the system level
- Only entitlements that are aggregated into ISC can be governed — connector configuration determines the governance scope
- Entitlement descriptions matter for certification quality; unreadable entitlement names lead to rubber-stamp certifications

> 🔗 **CONNECTS TO:** You now understand all three core concepts individually — identity, account, and entitlement. The next document shows how they connect into a unified model: the relationship that makes ISC's governance framework possible.
