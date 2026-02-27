# The Three Big Questions — How to Make Integration Decisions

Experienced integration engineers make integration design look instinctive. They hear the requirements, ask a few questions, and within a meeting they've sketched out an architecture that the team will spend the next month implementing.

That's not instinct. It's a framework applied quickly, after enough repetition that it becomes automatic.

The framework is three questions. Every significant integration decision you make as an ISC engineer — whether you're designing a new connection from scratch or debugging a problem with an existing one — can be traced back to how you answered these three questions.

Master them, and you have a mental model that applies to any integration challenge you'll face.

---

## Question 1: What Data Needs to Move?

This is the foundational question. Before you think about technology, connectors, or configuration, you need to understand the data.

### Direction: Source to ISC or ISC to Source?

The first thing to determine: which direction does data primarily flow?

**FROM the system TO ISC** (source/authoritative): This system knows things ISC needs to know. Employee data, org structure, contractor records. ISC reads it.

**FROM ISC TO the system** (target/managed): ISC makes decisions that need to be reflected in this system. Account creation, entitlement assignment, account disabling. ISC writes it.

**Both directions**: Some systems are both source and target (Active Directory being the canonical example). Data flows in both directions, with defined rules about which direction is authoritative for each attribute.

Misidentifying the direction is one of the most expensive integration design mistakes. If you treat a system as a target when it's actually authoritative (or vice versa), you'll build the wrong provisioning flows, configure the wrong attribute authority rules, and potentially overwrite source-of-truth data.

### What: The Minimum Necessary Attributes

Once you know the direction, the next question is: which attributes, accounts, and entitlements specifically need to move?

**For source systems** — what attributes does ISC need to make governance decisions?

For a Workday integration, the governance-relevant attributes are:
- Employee ID (correlation anchor)
- Name (identity display)
- Department + Job Title (access assignment rules)
- Manager (approval routing, certification assignment)
- Employment status / lifecycle state (triggers provisioning/deprovisioning)
- Start date and end date (lifecycle timing)
- Employment type (employee vs. contractor — different governance rules)

What you DON'T need from Workday (and shouldn't aggregate):
- Salary, compensation, pay grade
- SSN, date of birth, personal contact information
- Banking information, tax withholding settings
- Medical leave details

> ⚠️ **COMMON MISTAKE:** Over-aggregating sensitive data because "it might be useful someday." Every attribute you pull into ISC expands the platform's data footprint and data security obligations. ISC's identity store becomes a target if it holds sensitive PII or compensation data unnecessarily. Aggregate the minimum necessary for governance decisions.

**For target systems** — what accounts and entitlements need to be provisioned/managed?

For Salesforce as a target:
- Yes: User accounts, Profile assignments, Permission Set assignments
- Yes: Active/Inactive status
- Maybe: Custom fields if they're governance-relevant (department sync for Salesforce reports)
- No: Salesforce-specific configuration settings (territory models, forecast settings)
- No: Records and data within Salesforce (that's Salesforce's domain, not ISC's)

The decision rule: if it affects who can access what, it's in scope for governance. If it's business configuration that has nothing to do with access, it's out of scope.

> 🎯 **KEY POINT:** The minimum necessary principle applies to both attributes and entitlement scope. Every entitlement you choose to govern creates governance workload — certifications, policy evaluations, provisioning operations. Scope to what has meaningful access implications. Govern everything that matters; don't create noise around things that don't.

---

## Question 2: When Does Data Need to Move?

Timing isn't just a performance optimization — it's a governance decision with security implications.

### Three Timing Approaches

**Scheduled aggregation / batch processing**:
ISC runs aggregation on a fixed schedule (nightly, every 4 hours, etc.). Changes from the source system are reflected in ISC after the next scheduled run.

- When to use: Low-volatility data, long latency tolerance, systems that don't support webhooks or events
- Example: Jira project roles aggregated weekly — role changes don't require real-time governance response
- Latency: Hours to days

**Event-driven / near-real-time**:
The source system pushes events to ISC when changes occur (via webhook, message queue, or API callback). ISC responds to the event immediately.

- When to use: High-priority events, short latency requirements, systems that support event emission
- Example: Workday termination webhook → ISC receives event within seconds → deprovisioning triggers immediately
- Latency: Seconds to minutes

**Manual / on-demand**:
An administrator triggers aggregation manually when they know a change has occurred.

- When to use: One-time events, testing, emergency situations
- Example: After a system migration that moved 500 accounts, trigger a full aggregation to sync ISC
- Latency: As fast as you run it

### The Termination Latency Decision

This decision deserves special attention because the consequences of getting it wrong are significant.

If your termination processing uses nightly batch aggregation, you have up to a 24-hour window during which a terminated employee has active credentials. For standard terminations (voluntary departure, positive circumstances), some organizations accept this risk with a controlled offboarding process.

For privileged access users — IT admins, finance executives, security team members — a 24-hour window for a terminated employee with admin credentials is not acceptable.

The calculation:

| Termination Type | Max Acceptable Window | Required Timing Approach |
|---|---|---|
| Standard employee (low privilege) | 24 hours | Nightly batch aggregation |
| Standard employee (moderate privilege) | 4 hours | Frequent delta aggregation |
| IT admin / privileged user | Minutes | Event-driven webhook trigger |
| Involuntary termination (any level) | Minutes | Event-driven webhook trigger |
| Contractor offboarding | Hours | Scheduled aggregation or event |

> 💡 **REMEMBER:** Different termination paths can coexist in ISC. Standard terminations go through nightly aggregation. Flagged high-risk terminations trigger an event-driven workflow that runs immediately. ISC's workflow engine supports conditional logic — build the right response for the right situation.

### Applying the Timing Question

"ServiceNow is being added as a target system for IT ticket request access."

- What data moves: Account provisioning, group membership in ITSM roles
- When: Access request approval → immediate provisioning (ServiceNow accounts needed before IT can use the tool)
- Verdict: Event-triggered provisioning on access request approval. No need for real-time aggregation of ServiceNow data — weekly delta is fine for reading what accounts exist.

---

## Question 3: How Complex Is the Transformation?

Data from source systems rarely arrives in exactly the format ISC needs. Transformation is the work of converting what the system provides into what ISC needs to store and use.

### Three Levels of Transformation Complexity

**Simple attribute mapping — no transformation needed**:
Workday `Worker_ID` → ISC `employeeId`. The values are identical; only the attribute name changes. Zero transformation logic needed.

**ISC built-in transforms — handles most cases**:
ISC ships with a library of built-in transforms that handle common data manipulation without custom code:

- `Concat`: Combine `firstname` + "." + `lastname` → email format
- `Lookup`: Map department code "FIN-001" → "Finance" using a lookup table
- `DateFormat`: Convert "2024-11-15T00:00:00" → "2024-11-15"
- `Substring`: Extract characters from a string
- `Split`: Break a comma-separated string into individual values
- `E164phone`: Normalize phone number formats
- `Lower/Upper`: Case normalization

Chain these transforms together for moderate complexity: `Concat(Lower(firstname) + "." + Lower(lastname) + "@globaltech.com")`.

**Custom rules — for logic that transforms can't handle**:
When you need conditional logic, external lookups, or data manipulation that built-in transforms can't express, you write a BeanShell rule.

Example: Workday provides a department hierarchy code like `ENG.SW.BACKEND.PLATFORM`. ISC needs just the top-level department ("Engineering"). A custom rule parses the string, extracts the first segment, maps it against a lookup, and returns the normalized value — including handling cases where the format doesn't match the expected pattern.

> 🎯 **KEY POINT:** The transform library covers 80% of real-world transformation needs. Custom rules cover the remaining 20%. The mistake is skipping the transform library entirely and writing rules for everything. Rules are harder to maintain, harder to debug, and can break on ISC upgrades. Use built-in transforms until they genuinely can't do the job.

> ⚠️ **COMMON MISTAKE:** Over-engineering transforms. Engineers sometimes write 200-line BeanShell rules for transformations that a 3-step transform chain would handle. Complex rules have maintenance costs: the next engineer who works on this integration has to understand your code, bugs are harder to find, and ISC upgrades can break custom code in ways that built-in transforms don't suffer.

The decision rule: If a built-in transform can do it, use it. If a chain of built-in transforms can do it, use that. Write custom rules only when the transformation genuinely requires logic that transforms cannot express.

---

## Applying the Framework: A Real Integration Request

**Scenario**: GlobalTech is adding ServiceNow as a new managed system. The IT Director says: "We need ISC to manage ServiceNow access."

**Question 1: What data needs to move?**
- Direction: ISC → ServiceNow (it's a target — ISC provisions accounts and roles)
- Also: ServiceNow → ISC (aggregation, to see existing accounts and who has what)
- Attributes to provision: first name, last name, email, department, title, manager reference
- Entitlements to manage: ServiceNow roles (ITIL User, IT Manager, IT Admin, Asset Manager) — governance-relevant
- Attributes NOT to aggregate: ServiceNow-specific configurations, CMDB records, incident data

**Question 2: When does data need to move?**
- New employee provisioning: provision ServiceNow account as part of onboarding workflow — event-triggered when role is assigned
- Termination: ServiceNow account disabled immediately — event-triggered on termination lifecycle state
- Role changes: ServiceNow role updates within 24 hours — nightly delta aggregation is acceptable
- Verdict: Event-driven for onboarding and termination; nightly scheduled for ongoing sync

**Question 3: How complex are the transformations?**
- Most attributes: simple mapping (ISC `email` → ServiceNow `email`)
- ServiceNow user_name format: `firstname.lastname` — handle with built-in Concat transform
- Department mapping: ServiceNow uses different department names than ISC (ISC: "Engineering", ServiceNow: "IT-Engineering"). Three options: lookup table transform, custom rule, or update ServiceNow to match ISC naming. Choose lookup table transform — no custom code needed.
- Manager reference: ServiceNow wants the manager's ServiceNow user_id (not the email). Requires ISC to look up the manager's ServiceNow account. This requires a custom rule — transforms can't do cross-system lookups.
- Verdict: Built-in transforms for 80%, one custom rule for manager reference

**Design output**: ServiceNow integration as a managed target. Event-triggered provisioning/deprovisioning. Nightly aggregation for ongoing sync. Attribute mapping using Concat transform for username, lookup table for department, custom BeanShell rule for manager reference.

The team knows exactly what to build before touching ISC configuration.

## Key Takeaways

- The three-question framework applies to every integration: What moves (attributes, direction), When it moves (timing approach), How complex is the transformation (simple map, transform chain, or custom rule)
- Termination latency is the most governance-sensitive timing decision — match the approach to the risk level of the user population
- Over-aggregating sensitive data creates unnecessary risk; scope to governance-necessary attributes only
- Use built-in ISC transforms before writing custom rules — they're more maintainable, more reliable, and cover 80% of real cases
- Apply the framework systematically before implementation — it turns ambiguous requirements into a concrete design

> 🔗 **CONNECTS TO:** The framework tells you how to make decisions correctly. The next document tells you what happens when those decisions go wrong — the common mistakes that derail ISC implementations and how to avoid them.
