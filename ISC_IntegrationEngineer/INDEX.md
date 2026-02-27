# SailPoint ISC Integration Engineer — Complete Learning Path

**50 documents · 8 phases · Foundation to mastery**

This series builds the knowledge to work as a SailPoint Identity Security Cloud (ISC) integration engineer — from understanding why identity governance exists through planning and leading enterprise-scale implementations. Each document is self-contained and cross-referenced. Read in sequence for a complete curriculum, or jump to a specific topic using the index below.

---

## How to Use This Series

**If you are new to identity governance** → start at Phase 1 and read in order through Phase 3. You will understand the problem space, the platform, and the role before any technical configuration begins.

**If you understand IAM conceptually but are new to ISC** → start at Phase 2 and read through Phase 5 for the ISC-specific data model and governance layer.

**If you have ISC experience and want practical implementation guidance** → go directly to Phase 6. Each document there is a standalone practical reference.

**If you are troubleshooting a specific problem** → see the Quick Reference table at the bottom of this document.

---

## Phase 1 — Foundation: Why Identity Governance Exists

*6 documents · Audience: everyone · No prerequisites*

The problem space. Why organizations need identity governance, what it does, and how ISC fits into the enterprise. No technical background required. Engineers who skip this phase tend to build technically correct integrations that solve the wrong problems.

| # | File | Title | What It Covers |
|---|---|---|---|
| 1 | `phase_1.1_why_identity_matters.md` | Why Organizations Need Identity Management | The cost of unmanaged access; regulatory drivers; the human factor in security breaches |
| 2 | `phase_1.2_identity_governance.md` | What is Identity Governance? | Governance vs. management; the principle of least privilege; audit and compliance foundation |
| 3 | `phase_1.3_seven_functions.md` | Seven Core Functions That Solve These Problems | The seven governance functions ISC delivers; how they map to real business problems |
| 4 | `phase_1.4_what_is_isc.md` | What is SailPoint Identity Security Cloud? | Platform overview; ISC's position in the identity market; what it is and is not |
| 5 | `phase_1.5_five_questions.md` | Five Critical Questions ISC Answers | The five governance questions ISC answers; how answers drive security posture |
| 6 | `phase_1.6_cloud_benefits.md` | Why Cloud Architecture Matters for Identity | Cloud vs. on-premises governance; what the SaaS model changes for integrators |

---

## Phase 2 — Core Concepts: The ISC Data Model

*10 documents · Audience: everyone · Prerequisite: Phase 1 or equivalent context*

The three fundamental objects in ISC (identity, account, entitlement) and how they move — aggregation, correlation, provisioning, lifecycle management. This is the vocabulary every subsequent phase uses.

| # | File | Title | What It Covers |
|---|---|---|---|
| 7 | `phase_2.1_what_is_identity.md` | What is an Identity in ISC? | The identity cube; how ISC creates and maintains identity records; identity vs. person |
| 8 | `phase_2.2_what_is_account.md` | What is an Account? | Accounts as system representations; correlated vs. uncorrelated; account vs. identity |
| 9 | `phase_2.3_what_are_entitlements.md` | What are Entitlements? | Groups, roles, permissions as entitlements; how ISC models and governs them |
| 10 | `phase_2.4_how_they_relate.md` | How Identity, Account, and Entitlement Connect | The complete object model; how the three objects compose into governed access |
| 11 | `phase_2.5_aggregation_flow.md` | Aggregation — How ISC Gets Data FROM Systems | Aggregation as the data ingestion mechanism; what happens to data once ISC receives it |
| 12 | `phase_2.6_correlation_challenge.md` | The Correlation Challenge — Linking Accounts to Identities | Why correlation is hard; the identity matching problem; uncorrelated account risk |
| 13 | `phase_2.7_provisioning_flow.md` | Provisioning — How ISC Sends Data TO Systems | Provisioning architecture; the provisioning plan; how ISC writes to target systems |
| 14 | `phase_2.8_automatic_assignment.md` | How ISC Automatically Assigns Access | Rule-based role assignment; birthright provisioning; the access assignment engine |
| 15 | `phase_2.9_lifecycle_changes.md` | What Happens When Employee Status Changes | Lifecycle events; how status changes drive access changes; the governance loop |
| 16 | `phase_2.10_integration_types.md` | Different Types of Integrations You'll Build | Source vs. target integrations; read vs. write; the taxonomy of ISC connections |

---

## Phase 3 — The Engineer's Mindset: Role, Decisions, and Failure Modes

*5 documents · Audience: aspiring and early-career engineers · Prerequisite: Phases 1–2*

What integration engineers actually do day-to-day, how to make integration design decisions, what goes wrong most often, and how a complete integration project unfolds from requirements to production.

| # | File | Title | What It Covers |
|---|---|---|---|
| 17 | `phase_3.1_what_engineers_do.md` | What Does an Integration Engineer Actually Do? | The 40/60 split between technical work and discovery/communication; a real engineering day |
| 18 | `phase_3.2_decision_framework.md` | The Three Big Questions — How to Make Integration Decisions | What data, when, and how complex: the three-question framework for any integration decision |
| 19 | `phase_3.3_common_mistakes.md` | Common Mistakes Beginners Make (And How to Avoid Them) | Six failure patterns from real implementations: correlation shortcuts, over-engineering, missing deprovisioning |
| 20 | `phase_3.4_real_scenario.md` | A Day in the Life — Real Integration Scenario From Start to Finish | Workday HR system migration; all five project phases; the data problems that always appear |
| 21 | `phase_3.5_integration_approaches.md` | Decision Guide — Choosing Your Integration Approach | Hub-and-spoke, event-driven, and flat file patterns; when each applies; how all three coexist |

---

## Phase 4 — Technical Deep Dive: Connectors and Data Movement

*5 documents · Audience: engineers · Prerequisite: Phases 1–3*

The technical infrastructure: Virtual Appliance deployment and operations, the connector catalog, aggregation mechanics, the complete transform library, and correlation rule development.

| # | File | Title | What It Covers |
|---|---|---|---|
| 22 | `phase_4.1_virtual_appliance.md` | Virtual Appliance — What It Is, How It Works, and Why It Matters | VA architecture; outbound-only connectivity; clusters; deployment; health states; log files |
| 23 | `phase_4.2_connector_types.md` | Connector Types — Choosing the Right Connection Method | Pre-built connectors; direct vs. VA-required; SCIM, Web Services, JDBC, custom SDK, flat file |
| 24 | `phase_4.3_aggregation_deep_dive.md` | Aggregation — Full, Delta, and Everything That Can Go Wrong | Five aggregation phases; full vs. delta; scheduling; performance tuning; the aggregation log |
| 25 | `phase_4.4_transform_library.md` | The Transform Library — Data Manipulation Without Code | All built-in transforms with examples: string ops, lookup, date/time, conditional, utility; chaining |
| 26 | `phase_4.5_correlation_rules.md` | Correlation Rules — Linking Accounts to Identities | Attribute-based correlation; fallback chains; BeanShell rule patterns; testing; debugging |

---

## Phase 5 — The Governance Layer: Access, Roles, and Lifecycle

*5 documents · Audience: engineers and governance teams · Prerequisite: Phases 1–4*

The governance objects built on top of connected systems: access profiles, roles, lifecycle states, provisioning policies, and access certification campaigns.

| # | File | Title | What It Covers |
|---|---|---|---|
| 27 | `phase_5.1_access_profiles.md` | Access Profiles — The Unit of Access Management | What access profiles contain; granularity decisions; naming conventions; ownership; vs. direct assignment |
| 28 | `phase_5.2_roles.md` | Roles — Building the Access Model That Reflects the Business | IT Roles vs. Business Roles; birthright vs. requestable; role mining; role explosion prevention |
| 29 | `phase_5.3_lifecycle_states.md` | Lifecycle States — Modeling the Identity Journey Through ISC | Standard states; rule configuration; action matrices; pre-hire provisioning; LOA design |
| 30 | `phase_5.4_provisioning_policies.md` | Provisioning Policies — How ISC Creates and Manages Accounts | Provisioning plans; attribute sources; conflict resolution; plan operations; connector quirks |
| 31 | `phase_5.5_access_certifications.md` | Access Certifications — Designing Reviews That Mean Something | Four campaign types; rubber-stamp prevention; AI recommendations; remediation; key metrics |

---

## Phase 6 — Practical Implementation: Building, Failing, and Fixing

*6 documents · Audience: engineers · Prerequisite: Phases 1–5*

Hands-on implementation guides: how to build source and target integrations step-by-step, solve real correlation problems, diagnose provisioning failures, apply systematic troubleshooting, and design a proper testing framework.

| # | File | Title | What It Covers |
|---|---|---|---|
| 32 | `phase_6.1_building_source_integration.md` | Building Your First Source Integration | AD integration end-to-end: requirements, connector config, schema, correlation rules, staged testing |
| 33 | `phase_6.2_building_target_integration.md` | Building Your First Target Integration | Salesforce integration end-to-end: connector, entitlement aggregation, provisioning policy, four test scenarios |
| 34 | `phase_6.3_identity_correlation_scenarios.md` | Real Correlation Challenges and How to Solve Them | Five scenarios: clean match, email mismatch, name change, duplicate identities, no shared attribute |
| 35 | `phase_6.4_provisioning_failures.md` | Real Provisioning Failures and How to Fix Them | Six failure types with error messages, root causes, and diagnostic steps; data vs. system problem distinction |
| 36 | `phase_6.5_troubleshooting_decision_tree.md` | How to Diagnose Any Integration Problem | Six-step methodology; component isolation; log selection; ASCII decision tree; complete worked example |
| 37 | `phase_6.6_testing_validation_framework.md` | How to Test Integrations Properly | Three testing layers; real data safety; edge case checklist; pre-go-live checklist |

---

## Phase 7 — Advanced Scenarios: Enterprise Complexity

*7 documents · Audience: experienced engineers · Prerequisite: Phases 1–6*

The scenarios that appear in mature enterprise environments: multi-system architecture, organizational complexity, advanced certifications, separation of duties enforcement, event-driven integration, custom connector development, and performance optimization.

| # | File | Title | What It Covers |
|---|---|---|---|
| 38 | `phase_7.1_multi_system_architecture.md` | Integrating 10+ Systems — The Big Picture | Meridian Capital 28-system case; dependency sequencing; attribute authority matrix; governance ownership |
| 39 | `phase_7.2_complex_business_requirements.md` | Real Organizational Complexity | Six patterns: matrix orgs, contractors, geographies, role hierarchies, part-time, data isolation |
| 40 | `phase_7.3_access_certification_advanced.md` | Access Certifications at Scale | Compliance requirements by framework; reviewer fatigue solutions; privileged access certification; metrics |
| 41 | `phase_7.4_separation_of_duties.md` | Separation of Duties — Preventing Fraud Through Access Control | SoD policy design; ISC policy violations; detective vs. preventive mode; Meridian Capital banking example |
| 42 | `phase_7.5_event_driven_integration.md` | Event-Driven Integration — Real-Time Governance Response | Webhooks; ISC Trigger Framework; Workday termination flow; new hire pre-provisioning; reliability design |
| 43 | `phase_7.6_custom_connector_development.md` | When to Build Custom Connectors — And How to Decide | Seven-step decision framework; Connector SDK overview; maintenance planning; overlooked alternatives |
| 44 | `phase_7.7_performance_optimization.md` | Making Slow Integrations Fast — Performance Optimization | Aggregation bottleneck diagnosis; delta migration; VA tuning; BeanShell profiling; 6-hour→43-min case study |

---

## Phase 8 — Mastery: Strategy, Leadership, and Career

*6 documents · Audience: senior engineers and leads · Prerequisite: Phases 1–7*

Enterprise implementation planning, maturity assessment, expert-level troubleshooting, real-world case studies, staying current with the evolving platform, and building a professional portfolio that demonstrates expertise.

| # | File | Title | What It Covers |
|---|---|---|---|
| 45 | `phase_8.1_enterprise_implementation_planning.md` | Planning a Large-Scale ISC Rollout | Six-phase 18-month plan; exit criteria per phase; risk table; Meridian Capital end-to-end example |
| 46 | `phase_8.2_system_maturity_assessment.md` | Measuring ISC Implementation Maturity | Four maturity levels; self-assessment matrix; the Level 2 plateau problem; roadmap by level |
| 47 | `phase_8.3_expert_troubleshooting.md` | Advanced Problem-Solving — The Hard 20% | Three case studies: intermittent correlation, single-identity failure, upgrade performance regression |
| 48 | `phase_8.4_real_case_studies.md` | Learning from Real Implementation Stories | Startup rapid scale; legacy manufacturing modernization; financial services SEC compliance |
| 49 | `phase_8.5_continuous_learning.md` | Staying Current With SailPoint ISC | Quarterly release process; official/developer/community resources; certifications; learning cadence |
| 50 | `phase_8.6_building_expertise_portfolio.md` | Building Your Expertise Portfolio | Documentation; community contribution; GitHub; LinkedIn outcomes language; case studies; mentoring |

---

## Quick Reference: Find the Right Document for Your Problem

| Situation | Go to |
|---|---|
| "I need to explain what ISC does to a non-technical stakeholder" | 1.4, 1.5 |
| "I don't understand what correlation is" | 2.6, 4.5 |
| "I'm gathering requirements for a new integration" | 3.1, 3.2 |
| "I need to choose a connector type" | 4.2 |
| "My aggregation has too many uncorrelated accounts" | 4.5, 6.3 |
| "I need to build an AD source integration" | 6.1 |
| "I need to build a Salesforce target integration" | 6.2 |
| "A provisioning operation is failing" | 6.4, 6.5 |
| "I need to design a testing plan" | 6.6 |
| "An integration is running too slowly" | 7.7 |
| "I need to implement SoD policies" | 7.4 |
| "I need real-time termination processing" | 7.5 |
| "I'm deciding whether to build a custom connector" | 7.6 |
| "I need to design an access certification campaign" | 5.5, 7.3 |
| "I need to model a complex org structure" | 7.2 |
| "I'm planning a multi-system enterprise deployment" | 7.1, 8.1 |
| "I need to assess my ISC implementation maturity" | 8.2 |
| "I'm troubleshooting something unusual and hard" | 8.3 |
| "I want to understand the VA architecture" | 4.1 |
| "I need to configure lifecycle states" | 5.3 |
| "I need to explain how to build a role model" | 5.2 |
| "I want to understand ISC transforms" | 4.4 |
| "I need to design provisioning policies" | 5.4 |
| "I want to build my professional profile in ISC" | 8.5, 8.6 |

---

## The Running Example

All 50 documents use a consistent fictional organization — **GlobalTech** — a 12,000-employee manufacturing and software company. GlobalTech's ISC environment grows across the series: by Phase 7 it has 47 connected systems. A second fictional organization — **Meridian Capital**, an $85B asset management firm — appears in Phases 7 and 8 for financial services governance scenarios.

Using consistent fictional organizations across the series means concepts introduced in early phases are referenced and extended in later ones, rather than each document starting from a blank example.

---

*Series covers SailPoint ISC as of 2025. SailPoint releases quarterly platform updates; verify current feature availability against official documentation at `documentation.sailpoint.com`.*
