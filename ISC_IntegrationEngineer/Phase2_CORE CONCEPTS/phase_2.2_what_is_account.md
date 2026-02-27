# What is an Account?

Sarah Chen is one person. But to the seven systems she uses every day at GlobalTech, she's seven different entities — each described differently, each authenticated differently, each managing her access differently.

This fragmentation is by design: each system owns its own user database. Active Directory doesn't know what Salesforce thinks about Sarah. GitHub doesn't know what AWS grants her. They're independent silos with independent account records.

An **account** is Sarah's presence on one specific system. It's the credential and profile that system uses to recognize her, authenticate her, and determine what she can do. Understanding accounts — what they contain, how they differ across systems, and what states they can be in — is foundational to everything in ISC integration work.

## One Identity, Seven Accounts

Sarah has accounts on seven systems at GlobalTech. Here's what each one actually looks like:

| System | Account Identifier | Auth Method | Account Type |
|---|---|---|---|
| Active Directory | `s.chen` (samAccountName) | Password + MFA | Managed (ISC) |
| Salesforce | `sarah.chen@globaltech.com` | SSO via Okta | Managed (ISC) |
| GitHub | `schen-dev` | SSH key + PAT | Managed (ISC) |
| AWS | `sarah_chen_dev` (IAM username) | AWS SSO | Managed (ISC) |
| Slack | `sarah.chen` | SSO via Okta | Managed (ISC) |
| Jira | `SChen` | SSO via Okta | Managed (ISC) |
| SAP Finance | `SCHEN001` | SAP credentials | Managed (ISC) |

Same person. Seven different identifiers. Seven different authentication methods. Seven different attribute schemas for describing what she can do.

This is the practical reality that makes identity governance hard — and why ISC's account correlation capability (more on this in a later document) is so critical.

## Account Attributes: What Systems Actually Store

Each system stores different attributes about Sarah's account. The attribute schema reflects what that system cares about for its own purposes:

**Active Directory Account (`s.chen`)**:
```
samAccountName: s.chen
displayName: Sarah Chen
mail: sarah.chen@globaltech.com
userPrincipalName: s.chen@globaltech.com
memberOf: [CN=Domain Users,CN=Developers,CN=VPN-Access,CN=Dev-Environment]
userAccountControl: 512 (normal account, enabled)
passwordLastSet: 2024-10-12
department: Engineering
title: Software Engineer
manager: CN=James Wilson,OU=Engineering,DC=globaltech,DC=com
employeeID: GT-48291
```

**Salesforce Account (`sarah.chen@globaltech.com`)**:
```
Username: sarah.chen@globaltech.com
UserId: 0053X00000AbCdE
ProfileId: Standard User
IsActive: true
FederationIdentifier: sarah.chen@globaltech.com (SSO link)
PermissionSets: [Finance_Reports_ReadOnly, Dev_Sandbox_Access]
UserRole: Software_Engineer_Role
Department: Engineering
Title: Software Engineer
```

**GitHub Account (`schen-dev`)**:
```
login: schen-dev
id: 4829103
email: sarah.chen@globaltech.com
name: Sarah Chen
site_admin: false
Organizations: globaltech-corp
Teams: [backend-engineers, platform-team]
Repos: [globaltech-corp/core-api (write), globaltech-corp/docs (write)]
```

**AWS IAM Account (`sarah_chen_dev`)**:
```
UserName: sarah_chen_dev
UserId: AIDAEXAMPLEID
Arn: arn:aws:iam::123456789:user/sarah_chen_dev
Groups: [developers, s3-dev-access]
AttachedPolicies: [Developer-IAM-Policy]
Tags: {Department: Engineering, CostCenter: ENG-001}
PasswordLastUsed: 2024-11-28
AccessKeys: [AKIA... (active, created 2023-03-15)]
```

Notice what these schemas have in common: almost nothing, structurally. Each system has its own concept of what matters. ISC's job — and part of your job as an integration engineer — is mapping these different schemas onto a consistent view within ISC's data model.

> 🎯 **KEY POINT:** Account attribute schemas vary completely between systems. ISC doesn't assume any standard — it reads whatever the system exposes through its connector and maps it to ISC's internal schema. This mapping (what source attribute maps to what ISC attribute) is one of the core configuration tasks in integration engineering.

## Account States: More Than Active or Disabled

Accounts aren't just "on" or "off." ISC tracks several distinct account states, and each has different governance implications:

**Active**: The account is enabled and the user can authenticate with it. Normal operational state.

**Disabled/Inactive**: The account exists but authentication is blocked. The account and its entitlements are preserved, but the user cannot log in. Common for employees on extended leave, or as the first step in an offboarding process before full deletion.

**Locked**: The account is temporarily blocked, usually due to too many failed authentication attempts. Different from disabled — typically a security mechanism that resets automatically or requires an admin action to unlock. Not a governance state; a security state.

**Orphaned**: The account exists in the source system but ISC cannot correlate it to any identity record. This is a significant governance risk — it means there's an active account on a system with no known owner. Orphaned accounts appear in ISC as uncorrelated accounts.

**Dormant**: The account exists and is active, but has not been used for an extended period (often defined as 90+ days). ISC surfaces these during certifications as candidates for review or disable.

**Managed vs. Unmanaged**: This ISC-specific concept refers to whether ISC is actively governing the account. A managed account was either provisioned by ISC or has been brought under ISC governance through aggregation and correlation. An unmanaged account was created outside ISC's purview.

> ⚠️ **COMMON MISTAKE:** Treating disabled accounts as fully deprovisioned. A disabled AD account still holds all its group memberships and entitlements. When that account gets re-enabled (perhaps because an employee was incorrectly terminated, or returns from leave), all that access comes back instantly. True deprovisioning removes entitlements before or alongside disabling the account. ISC's deprovisioning workflows handle this sequence correctly — manual processes often don't.

## Why Account Data Differs From Identity Data

This distinction matters practically. Sarah's identity in ISC says her job title is "Software Engineer" — sourced from Workday. Her AD account also has a `title` attribute: "Software Engineer." These match right now.

But what if Workday updates Sarah's title to "Senior Software Engineer" tonight, and the AD account doesn't update until the next provisioning run tomorrow? Or what if IT manually updated her title in AD three months ago but Workday still shows "Software Engineer" because HR hasn't processed the change yet?

This is the **identity/account synchronization problem** — and it's pervasive in real implementations. ISC helps by serving as the integration layer: Workday is authoritative, ISC reads Workday during aggregation, ISC updates AD during provisioning. But there are always latency windows and edge cases.

As an integration engineer, you'll frequently deal with "account says X but identity says Y" situations. Knowing which system is authoritative — and why — is essential to diagnosing these correctly.

> 💡 **REMEMBER:** ISC's identity record reflects the authoritative source data. Account records reflect what ISC read from that system during the last aggregation. If an account attribute changed directly in the target system (bypassing ISC), the account record in ISC will be stale until the next aggregation run. This is why monitoring aggregation frequency and recency matters for governance accuracy.

## Service Accounts: The Special Case

Not every account belongs to a human. Systems use **service accounts** — credentials used by applications, scripts, and automated processes to authenticate with other systems. These are accounts without a human owner, and they represent a distinct governance challenge.

Examples at GlobalTech:
- `svc-isc-ad`: The account ISC uses to authenticate with Active Directory for aggregation
- `svc-jenkins-aws`: The service account Jenkins uses to deploy to AWS
- `svc-monitoring-jira`: The account the monitoring system uses to create Jira tickets
- `svc-backup-sql`: The account the backup system uses to access the SQL database

When ISC aggregates Active Directory, it reads all accounts — including service accounts. If the correlation rules aren't designed to handle service accounts (exclude them or assign them to a special "service account" identity), they'll appear as uncorrelated accounts in ISC, creating noise and potentially false governance concerns.

Every mature ISC implementation has an explicit strategy for service accounts: either exclude them from governance scope (documented exception), assign them to a "System" identity with a designated owner, or govern them through a separate service account governance program.

> ⚠️ **COMMON MISTAKE:** Forgetting to design service account handling before the first aggregation run. When you run your first aggregation of Active Directory and 40 of the 8,400 accounts are service accounts that can't correlate to any human, you'll generate 40 uncorrelated account alerts. Design the exclusion rules first. Your security team will thank you.

## Key Takeaways

- An account is a person's presence on a specific system — one identity can have accounts on dozens of systems
- Account schemas differ completely between systems: each system stores the attributes relevant to its own purpose
- Account states matter for governance: active, disabled, orphaned, dormant, and managed/unmanaged each have different governance implications
- Account data can drift out of sync with identity data; ISC serves as the reconciliation layer between what HR says and what target systems hold
- Service accounts need an explicit governance strategy from day one — they will appear during aggregation and need to be handled

> 🔗 **CONNECTS TO:** Accounts give Sarah presence on systems. But presence isn't access. The next document explains entitlements — the specific permissions that determine what Sarah can actually *do* on each system.
