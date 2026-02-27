# Integrating 10+ Systems — The Big Picture

Connecting one system to ISC is a technical exercise. Connecting 30 systems is an architecture exercise. The difference is substantial — and the engineers who understand the architectural dimension are the ones who get asked to lead enterprise-scale ISC implementations.

At scale, the questions that matter aren't "how do I configure this connector" — you already know that. The questions are: which systems should be integrated in which order, how do data flows between systems create dependencies, where does ISC sit in relation to other identity infrastructure, and how does a governance decision in one system propagate correctly through the rest?

GlobalTech's full environment has 47 connected systems. This document uses a representative financial services company — Meridian Capital, with $85B AUM, 3,200 employees — to show what enterprise-scale ISC architecture actually looks like.

---

## Meridian Capital's Environment

Meridian Capital's compliance requirements are stringent: SOX, SEC regulations, FINRA membership, and internal audit requirements that mandate quarterly access certifications for all systems touching financial data. When the CISO commissioned their ISC implementation, the landscape was:

- **HR systems**: Workday (primary), a legacy payroll system (Ceridian)
- **Directory infrastructure**: Active Directory (3 domains: corp, trading, compliance)
- **Financial applications**: Bloomberg terminal access, Advent portfolio management, Charles River OMS, Linedata, FactSet, Refinitiv Eikon (6 financial platforms, each with its own access model)
- **Corporate applications**: Salesforce, ServiceNow, Jira, Confluence, Zoom, Slack
- **Infrastructure**: AWS (4 accounts), Azure, on-prem servers (Unix/Linux)
- **Privileged access**: CyberArk PAM (manages 800+ service accounts and privileged credentials)
- **Security tools**: Splunk, Qualys, Palo Alto Panorama

Total: 28 systems in initial scope, with 12 more in a phase 2 roadmap.

---

## Dependency Mapping: What Needs to Exist Before What

The first architectural decision in a multi-system ISC deployment is sequence — which systems must be connected before others can work correctly.

**Dependencies flow from authoritative sources outward**:

**Tier 1 — Identity sources (must come first)**:
Workday must be connected and aggregating before any other system is correlated or provisioned. Workday is the authoritative HR record. Every identity in ISC derives its core attributes (employee ID, name, department, manager, lifecycle state) from Workday. Without Workday, ISC has no reliable identity data to govern.

**Tier 2 — Directory infrastructure (must come before applications)**:
Active Directory must be connected before any application that uses AD-based authentication. Many of Meridian's applications (Bloomberg terminal, Advent, Charles River) use AD group membership as their access control mechanism. ISC can't govern their access profiles correctly until AD entitlements are aggregated and visible.

**Tier 3 — High-priority governance targets (financial applications)**:
For Meridian, SOX requires quarterly certification of all access to financial systems. Bloomberg, Advent, Charles River, and Linedata must be connected to ISC before the first quarterly certification cycle.

**Tier 4 — Corporate applications**:
Salesforce, ServiceNow, Slack, Jira — these matter for efficiency but not for regulatory compliance. They can be integrated in later phases without compliance risk.

**Tier 5 — Infrastructure and privileged access**:
AWS, Azure, CyberArk — these are complex integrations that take longer to design. They're high-value but not blocking the initial compliance requirement.

> 🎯 **KEY POINT:** Integration sequence should be driven by governance priority, not technical complexity. Connect the systems that create the highest compliance and security risk first, even if they're harder to integrate. Prioritizing technically simple systems first (because it builds momentum) is a strategy that produces a beautiful demonstration environment and leaves your riskiest systems ungoverned the longest.

---

## The Hub-and-Spoke Architecture at Scale

For 28 connected systems, Meridian uses a hub-and-spoke architecture: ISC is the hub, every system is a spoke. All identity and access governance flows through ISC — no system talks to another system for access governance purposes.

```
            Workday (Identity Source)
                  │
                  ▼
    ┌─────── ISC Hub ───────────────────────────┐
    │                                           │
    ├──► Active Directory (3 domains)           │
    ├──► Bloomberg Terminal                     │
    ├──► Advent Portfolio Mgmt                  │
    ├──► Charles River OMS                      │
    ├──► Salesforce                             │
    ├──► ServiceNow                             │
    ├──► AWS (4 accounts)                       │
    ├──► CyberArk PAM                           │
    └──► 19 additional systems                  │
                                                │
    Ceridian Payroll ──[flat file]──► ISC       │
    (read-only, attributes only)                │
    └────────────────────────────────────────────┘
```

**What hub-and-spoke gives Meridian**:

Every quarterly SOX certification query: "Who has access to financial systems?" is answered from one place — ISC. Auditors don't interview 6 different system administrators; they review one ISC report covering all financial application access.

Every termination flows through one process: Workday termination → ISC lifecycle change → ISC governance decision → all 28 systems updated. The CISO sees one event trigger 28 automated actions, not 28 separate tickets.

Every SoD policy (covered in Phase 7.4) is enforced across all 28 systems because ISC sees all of them. A policy that says "a person can't have trade execution access in Charles River AND trade approval access in Advent" requires ISC to see access in both systems — which hub-and-spoke provides.

---

## Managing Data Flow Between Systems

At scale, the question of data flow — which system is authoritative for which attributes — becomes critical. Attribute authority conflicts produce contradictory data and unpredictable provisioning behavior.

**Meridian's attribute authority matrix**:

| Attribute | Authority | Secondary Source |
|---|---|---|
| Employee ID | Workday | None |
| Name (legal) | Workday | None |
| Department | Workday | None |
| Job title | Workday | None |
| Manager | Workday | None |
| Lifecycle state | Workday (termination, LOA) | None |
| AD samAccountName | Active Directory | None |
| AD distinguishedName | Active Directory | None |
| Email address | Active Directory | Workday (authoritative format) |
| Cost center | Ceridian Payroll | Workday |
| Pay grade | Ceridian Payroll | None |

When Workday and Ceridian conflict on cost center (which happens after org changes that update one but not the other), ISC uses the Workday value (higher authority) and flags the discrepancy for HR to investigate.

**Circular authority problem**: One system writes to another which writes back. ISC provisions an attribute to Salesforce; Salesforce's admin changes the attribute in Salesforce; ISC aggregates the change back and overwrites the identity with the Salesforce value; the next provisioning cycle pushes the Workday value back to Salesforce. This loop generates noise, incorrect data, and support tickets.

Prevention: in ISC, for each attribute on each target system, explicitly configure whether ISC can update that attribute in the target. Turn off update for attributes that the target system owns. If Salesforce owns its own `Region` field, don't configure ISC to push `region` to Salesforce.

---

## Governance Ownership: Who Owns What System

At scale, the question of ownership becomes an operational issue. When ISC has 28 connected systems, 150 access profiles, and 40 certification campaigns, "the ISC team owns everything" breaks down. Someone needs to own each access profile, each certification campaign, and each system integration.

**Meridian's ownership model**:

| Role | Owns |
|---|---|
| ISC Platform Team (4 engineers) | ISC configuration, VA infrastructure, connector maintenance, identity profiles |
| Application Owner (1 per system) | Access profiles for their system, access request approvals, entitlement descriptions |
| Business Process Owner (1 per function) | Business roles, certification campaign review, access governance policy |
| Security Team | SoD policies, privileged access governance, certification campaign design |
| HR | Lifecycle state rules, authoritative source data quality |

Without this ownership model, changes to access profiles happen without review, certification campaigns are created without stakeholder alignment, and integration problems escalate to the ISC team even when they're application-level issues.

> ⚠️ **COMMON MISTAKE:** ISC platform team owning all access profiles and certification campaigns. At scale, this doesn't scale — you have 4 engineers who don't know each application as well as the application owner does, trying to write descriptions for access profiles they've never used. Distribute ownership to application owners. The ISC team's job is governance platform management, not governance content authorship.

---

## Planning for System Growth

Meridian's phase 2 roadmap has 12 additional systems. The architecture decisions made in phase 1 determine how painful phase 2 is.

**Architecture decisions that age well**:
- Consistent correlation attribute (employee ID) across all phase 1 systems — new systems correlate using the same attribute
- Standardized access profile naming (`[System] [Access Tier]`) — new systems follow the pattern without debate
- Documented attribute authority matrix — new systems' attribute ownership slots into the existing framework
- VA cluster sized for current load + 50% headroom — phase 2 systems don't require immediate VA scaling

**Architecture decisions that don't age well**:
- Custom correlation rules for each system with no shared logic — each new system requires custom engineering
- No naming standards — phase 2 access profiles get named by whoever sets them up, producing an inconsistent catalog
- Single VA in production — phase 2 systems all route through a single point of failure

> 🔗 **CONNECTS TO:** The architectural foundation handles the how of multi-system integration. The organizational structures those systems govern — contractors, matrix reporting, multiple geographies — shape what the governance model needs to represent. That's the subject of the next document.

---

## Key Takeaways

- Integration sequence should follow governance priority: identity sources → directory → high-risk applications → supporting systems
- Hub-and-spoke provides centralized governance, unified policy enforcement, and single-source audit data — it's the right default for 10+ connected systems
- Attribute authority matrix prevents data conflicts: define explicitly which system is authoritative for each attribute, and configure ISC to not overwrite target-owned attributes
- Governance ownership must be distributed at scale — ISC team owns the platform; application owners own their access profiles and certifications
- Phase 1 architecture decisions (correlation attribute consistency, naming standards, VA sizing) determine phase 2 expansion pain
- 28 connected systems with unified governance = single termination trigger disabling 28 systems; single certification campaign covering all financial access

> 🔗 **CONNECTS TO:** Multi-system architecture addresses the infrastructure layer. The next document addresses the organizational complexity layer — how ISC models the non-standard structures that real organizations have: matrix reporting, contractors, multiple geographies, and complex role hierarchies.
