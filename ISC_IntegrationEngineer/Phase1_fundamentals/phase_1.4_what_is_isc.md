# What is SailPoint Identity Security Cloud?

When SailPoint released IdentityIQ in 2007, it ran on servers in your data center. You bought the hardware, managed the database, applied the patches, and hired the specialized engineers to keep it running. It was powerful — and it was yours to maintain indefinitely.

In 2019, SailPoint launched Identity Security Cloud (ISC). Same governance capabilities. No servers to manage. No patches to apply. No database to tune. Delivered as Software-as-a-Service from SailPoint's cloud infrastructure.

That's not just a deployment difference — it changes the economics, the operational burden, and the pace at which organizations can actually govern identity. Understanding *what* ISC is requires understanding what changed when identity governance moved to the cloud.

## What ISC Actually Is

**SailPoint Identity Security Cloud** is a SaaS-delivered identity governance platform. Organizations connect their systems to it, and it becomes the central control point for managing who has access to what — across every connected system.

Here's the concrete picture: ISC sits between your organization's **source systems** (HR platforms like Workday or SAP SuccessFactors that define who employees are) and your **target systems** (everything people need access to: Active Directory, Salesforce, GitHub, AWS, ServiceNow, SAP, and hundreds of others).

Data flows in both directions:
- **In**: ISC reads account and entitlement data from all connected systems (called aggregation)
- **Out**: ISC writes changes to target systems — creating accounts, adding entitlements, disabling accounts (called provisioning)

ISC itself holds the authoritative picture: a unified identity record for every person in the organization, showing all their accounts, all their entitlements, their attributes, their history, and their current access status. This is the single source of truth that makes governance possible.

> 🎯 **KEY POINT:** ISC is the connective tissue between your HR system (which defines identity) and every other system your organization uses. Without it, each system manages its own access independently. With it, access is governed centrally through consistent policies applied everywhere simultaneously.

## ISC vs. IdentityIQ: Why the Distinction Matters

You'll encounter both products in the SailPoint ecosystem. Many large enterprises still run IdentityIQ (on-premises). Understanding the difference matters for your work as an integration engineer.

| Dimension | IdentityIQ (On-Prem) | Identity Security Cloud (SaaS) |
|---|---|---|
| Deployment | Customer's data center | SailPoint's cloud infrastructure |
| Upgrades | Customer-managed, disruptive | Automatic, continuous |
| Customization | Extensive (BeanShell rules, custom code) | Governed (rules + transforms + workflows) |
| Connector updates | Manual process | Automatic via SailPoint |
| AI/ML capabilities | Limited | Native (AI-driven recommendations, risk scoring) |
| Time to value | Months (implementation) | Weeks (configuration) |
| Operational overhead | High (DBAs, sysadmins, patch management) | Minimal (SailPoint manages infrastructure) |

The key trade-off: IdentityIQ is more customizable, ISC is more operable. Organizations that need deeply custom governance logic often stay on IIQ. Organizations prioritizing operational simplicity and modern AI capabilities choose ISC.

As an integration engineer, you'll need to know both — many enterprises have IIQ on-prem and are evaluating or migrating to ISC. But this curriculum focuses on ISC.

> 💡 **REMEMBER:** When someone says "SailPoint," they might mean IdentityIQ (on-prem) OR Identity Security Cloud (SaaS). Always clarify which platform a conversation is about — the architecture, configuration approach, and connector types differ significantly.

## How ISC Automates the Seven Functions

In the previous document, you saw the seven functions of identity governance. ISC automates every one of them. Here's how that looks for a mid-size company — Cortex Technologies, 2,400 employees, 28 connected systems:

**Provisioning**: When Cortex's HR team marks a new hire in Workday, ISC receives that event within minutes. It evaluates the employee's department, job title, and level against defined access policies. It creates the AD account, Slack workspace, Jira instance access, and Salesforce profile — all before the employee's first day. No ticket. No sysadmin intervention.

**Deprovisioning**: When an employee is terminated in Workday, ISC disables all 28 connected system accounts within the same automated trigger. The total elapsed time from HR action to account disable: under 15 minutes. Previously: average 12 days.

**Access Request**: Employees access the ISC portal to request additional access beyond their standard provisioning. Requests route through configured approval chains. Access is provisioned only after all required approvals are recorded. Cortex processes 340 access requests per month this way.

**Access Certification**: Every quarter, ISC generates certification campaigns automatically. Managers review their team's access in the ISC portal, certifying or revoking each entitlement. ISC tracks completion, sends reminders, and automatically revokes access when a manager doesn't respond within the deadline. No spreadsheets.

**Role Management**: Cortex has defined 67 role definitions in ISC — one for each job function. When HR adds a new employee with a known job title, ISC matches them to the right role and provisions all role-based access automatically. Role definitions are reviewed and updated quarterly.

**Policy Management**: ISC evaluates entitlements against SoD policy rules continuously. When David in Finance (from the previous document's example) accumulated conflicting journal entry entitlements, ISC generated a policy violation. The compliance team remediated it within 48 hours.

**Reporting**: Cortex's last SOC 2 audit required 14 specific reports on access controls. The ISC admin generated all 14 in 90 minutes. Previously, gathering this data manually took three people two weeks.

> ⚠️ **COMMON MISTAKE:** Expecting ISC to work perfectly out of the box. ISC provides the *capability* to automate these seven functions — but it requires proper configuration: connected sources, defined roles, configured policies, built access profiles, and trained administrators. It's a platform, not a plug-and-play solution. The integration engineer's job is building the connections and configurations that make automation work.

## The ISC Architecture: Key Components

Understanding what ISC is made of helps you understand what you're configuring as an engineer:

**Tenant**: Your organization gets a dedicated ISC tenant — a logically isolated instance of the platform. It's multi-tenant infrastructure (shared hardware, isolated data). Your data never mingles with another organization's.

**Virtual Appliance (VA)**: A small virtual machine that runs inside your network and brokers communication between ISC (in the cloud) and on-premises systems (like Active Directory). ISC can't reach inside your firewall directly — the VA creates a secure, outbound-initiated tunnel. For cloud-to-cloud integrations (ISC to Salesforce, ISC to Workday), no VA is needed.

**Connectors**: Pre-built integration modules for specific systems. ISC ships with 200+ connectors covering Active Directory, LDAP, Salesforce, GitHub, AWS, ServiceNow, SAP, Workday, and many others. Connectors handle the system-specific communication protocols, authentication, and data mapping. You configure them; you don't write them from scratch (usually).

**Identity Cube**: ISC's term for the unified identity record. It aggregates account data from all connected sources, correlates it back to the person, and builds a comprehensive picture of that person's access across the entire environment.

**Workflows**: Automation sequences triggered by events (employee hired, access requested, certification failed, violation detected). ISC's workflow engine orchestrates approval chains, notifications, and automated actions.

**AI Engine**: ISC's AI capabilities — peer group analysis (flagging access that looks anomalous compared to similar roles), access recommendations during certifications, risk scoring for identities and entitlements. This is native to ISC and not available in IdentityIQ.

> 🎯 **KEY POINT:** The Virtual Appliance is often the first technical hurdle in ISC implementations. It must be deployed in your network, configured to communicate with both ISC and your internal systems, and kept running. VA downtime means aggregation stops. Understanding the VA architecture is essential for diagnosing connectivity problems.

## What ISC Is Not

Setting accurate expectations saves significant frustration:

ISC is **not** an SSO platform. It doesn't handle authentication (logging users in). That's Okta, Azure AD, Ping Identity's job. ISC governs *which* access exists — SSO manages *how* users access it. They're complementary but distinct.

ISC is **not** a SIEM. It doesn't analyze security events, correlate logs, or detect attacks in real time. That's Splunk, Microsoft Sentinel, or CrowdStrike's domain. ISC generates audit logs that a SIEM might consume — but they serve different purposes.

ISC is **not** automatic without configuration. The automation is powerful, but it requires integration engineers to connect systems, define policies, build roles, and configure workflows. The platform is the engine — integration engineers build the roads.

## Key Takeaways

- ISC is a SaaS identity governance platform — no on-premise servers, automatic updates, AI-native capabilities
- It acts as the central control point between HR systems (which define who people are) and all other systems (which provide what they can access)
- ISC automates all seven identity governance functions: provisioning, deprovisioning, access request, certification, role management, policy management, and reporting
- Key components: tenant, Virtual Appliance, connectors, Identity Cube, workflows, AI engine
- ISC is not SSO, not SIEM — it governs access, it doesn't authenticate users or detect attacks

> 🔗 **CONNECTS TO:** Now that you know what ISC does, the next document maps its specific capabilities to the five critical questions that every identity governance program must answer.
