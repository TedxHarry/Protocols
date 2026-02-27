# Why Cloud Architecture Matters for Identity

Let's be direct: "cloud is better than on-prem" is marketing. The real question is *what specifically gets better* when identity governance moves to SaaS — and what the honest trade-offs are.

For identity governance specifically, the cloud shift solves a set of problems that were genuinely painful in on-premises deployments. Understanding these problems (and their solutions) gives you the context to explain ISC's strategic value to clients and stakeholders who are evaluating whether to move from IdentityIQ to ISC, or adopting SailPoint for the first time.

## The On-Premises Identity Problem

Before ISC, organizations running IdentityIQ owned the full operational stack: servers, databases (Oracle or SQL Server), application infrastructure, connector management, patching cycles, upgrade projects, and the specialized engineers to run all of it.

Here's what that looked like in practice at Hartwell Manufacturing, a 3,200-person company running IdentityIQ on-prem:

**Quarterly patch cycles**: IdentityIQ releases security patches and minor updates quarterly. Each patch requires a change window, testing in a non-production environment, coordination with the database team, and a deployment window on a Saturday morning. The typical patch cycle takes three people about 40 hours across preparation, testing, and deployment. That's 160 hours per year just on patches.

**Major upgrades**: IdentityIQ major version upgrades (e.g., 8.1 → 8.3) are multi-month projects. Hartwell's last upgrade took four months: two months of planning and environment preparation, one month of testing, one month of parallel operation before cutover. Cost: roughly $180,000 in professional services plus internal engineer time. Result: access to features that ISC customers got automatically on release day.

**Connector management**: When Salesforce changes its API (which happens frequently), the IdentityIQ Salesforce connector needs to be updated. On-prem customers download the new connector, test it in non-prod, and deploy it — each connector update is its own mini-project. With 15 connected systems, connector management became a perpetual background task.

**Infrastructure**: Two application servers (primary + failover), Oracle RAC database, load balancer, storage array. Hartwell's identity team spent roughly 20% of their time on infrastructure operations rather than identity governance work.

> 🎯 **KEY POINT:** On-premises identity governance creates an ongoing operational tax. The team that should be focused on governing access spends a significant fraction of their time governing the platform itself. This isn't bad IT — it's the structural reality of self-managed software at enterprise scale.

## What ISC Actually Changes

### Operational Overhead: Eliminated at the Infrastructure Layer

With ISC, SailPoint owns and operates the infrastructure. Hartwell's identity team doesn't patch servers, tune databases, or manage failover configurations. When SailPoint updates ISC (which happens continuously, with major releases quarterly), it happens automatically — in most cases, Hartwell's team notices new features in the interface without any action on their part.

Connector updates work the same way. When Salesforce updates their API and breaks the integration, SailPoint updates the ISC Salesforce connector on their end. The connection continues working. Hartwell's team gets a changelog notification, not a weekend deployment project.

**Quantified for Hartwell**: Moving from IdentityIQ to ISC freed approximately 0.7 FTE of engineering time previously consumed by infrastructure operations. That engineer is now doing governance work: designing better certifications, refining SoD policies, onboarding new systems.

> 💡 **REMEMBER:** "No infrastructure management" doesn't mean "no engineering work." ISC still requires engineers to configure sources, build roles, design workflows, and connect new systems. The eliminated work is specifically the operational overhead of running the platform — not the configuration work of implementing governance.

### Scaling: From 100 to 10,000 Employees

This is where the cloud architecture difference becomes most concrete. Consider Cascade Innovations, starting as a 100-person startup and growing to 10,000 employees over six years.

**Year 1 (100 employees, on-prem scenario)**:
If Cascade had chosen IdentityIQ, they'd have bought servers sized for current load. Identity governance works fine. Quarterly patch cycles are manageable. Two people handle it alongside other duties.

**Year 3 (1,200 employees)**:
The servers are approaching capacity. Database performance is degrading as the identity store grows. The team needs to expand infrastructure — purchase new hardware, migrate the database, scale up. This is a six-month project that happens while the current system is straining. The identity team, now three people, is spending half their time on infrastructure rather than governance.

**Year 5 (5,000 employees, post-acquisition)**:
Cascade acquires a 2,000-person company. They need to integrate the acquisition's identity systems in 90 days to meet deal commitments. On-prem: provision new hardware, expand database, load-test at new scale, integrate acquired company's systems — while running the current deployment. This is a crisis-level infrastructure project.

**With ISC**:
Year 1: Configure ISC tenant, connect HR system and 8 applications. Works.
Year 3: Add 15 more sources as the company grows. ISC scales automatically — SailPoint's infrastructure handles the additional load. No hardware projects.
Year 5: Acquisition integration. The acquired company gets a new ISC tenant or is onboarded to the existing one. SailPoint's infrastructure scales to accommodate. Cascade's identity team focuses on integration configuration — correlating acquired identities, mapping roles, building access profiles — not on infrastructure expansion.

**Quantified difference**: For Cascade's year 5 scenario, the ISC approach saved approximately four months and $240,000 compared to the equivalent on-prem expansion. More importantly, it meant the identity team's bandwidth went to governance work rather than infrastructure management.

> ⚠️ **COMMON MISTAKE:** Assuming cloud infrastructure means no performance tuning work. Large ISC deployments (50,000+ identities, 100+ sources) still require careful configuration: aggregation scheduling, workflow efficiency, connector parallelism. The infrastructure scales automatically — but *how* you configure aggregation and provisioning affects performance significantly at scale.

### Connector Currency: Always Up to Date

In on-premises identity governance, every connected system is a potential maintenance burden. When the system updates its API or changes authentication requirements, you need to update the connector — which means testing, deploying, and potentially breaking the integration during the update window.

ISC ships with 200+ pre-built connectors, and SailPoint maintains them. When ServiceNow releases a major update, SailPoint updates the ISC ServiceNow connector. When GitHub changes their OAuth flow, SailPoint updates the GitHub connector. The integration continues working.

For an organization with 30 connected systems, this eliminates 30 potential maintenance projects per year. The time savings compound as the connected system count grows.

### AI-Native Capabilities: Not Available On-Prem

ISC's AI engine — including peer group analysis, access recommendations, and risk scoring — runs on SailPoint's cloud infrastructure and requires the cloud architecture to function. These capabilities aren't available in IdentityIQ.

Concretely, this means:

**Certification recommendations**: During access certification, ISC analyzes peer groups and suggests which access items should be revoked based on what similar-role employees hold. Managers get data-driven recommendations rather than reviewing blind. Certification completion rates improve significantly, and revocation decisions are more defensible.

**Anomaly detection**: ISC compares each identity's access and activity patterns against their peer group. Outliers get flagged as risk signals. An engineer who suddenly holds Finance entitlements that no other engineer has gets surfaced for review — without a human having to know to look.

**Role mining**: ISC analyzes existing access patterns across the identity population to suggest logical role definitions. Instead of manually designing roles from scratch, organizations start with AI-generated suggestions based on what people actually have.

> 🎯 **KEY POINT:** The AI capabilities in ISC aren't add-ons — they're woven into core workflows (certifications, access requests, policy enforcement). An organization running ISC gets these capabilities as part of the platform, not as a separate project to implement.

## The Honest Trade-Offs

No architectural choice is free:

**Less customization than IdentityIQ**: IdentityIQ allows deeply custom BeanShell code throughout the platform. ISC provides a governed customization model — transforms, rules (BeanShell within defined boundaries), workflows, and REST APIs. Organizations that rely on extensive custom business logic in IdentityIQ may find ISC's customization model more constrained.

**SailPoint owns the upgrade schedule**: Continuous updates mean new features arrive automatically — but so do changes to existing behavior. ISC organizations need to monitor release notes and occasionally adjust configurations when platform behavior changes. On-prem organizations control exactly when they upgrade.

**Data residency considerations**: ISC runs in SailPoint's cloud infrastructure (primarily AWS). Organizations with strict data residency requirements need to verify SailPoint's regional hosting options and contractual commitments for their specific compliance context.

**Connectivity dependency**: ISC requires reliable internet connectivity for all governance functions. A significant network outage can interrupt governance workflows. On-premises deployments are fully local and more resilient to internet connectivity issues (though cloud system connectors still need connectivity).

## Key Takeaways

- ISC eliminates the operational overhead of managing identity governance infrastructure: no patching, no database administration, no server management
- Scaling is automatic — ISC handles growth from 100 to 100,000 employees without infrastructure expansion projects
- Connector currency is maintained by SailPoint — when connected systems update their APIs, SailPoint updates the connector
- AI capabilities (peer analysis, access recommendations, anomaly detection, role mining) are native to ISC and require the cloud architecture
- Trade-offs are real: less customization than IdentityIQ, SailPoint controls the upgrade schedule, and data residency requirements need verification

> 🔗 **CONNECTS TO:** Phase 1 established the problem and the platform. Phase 2 goes deeper into the building blocks: what exactly is an identity, an account, and an entitlement in ISC — and how do they relate to each other?
