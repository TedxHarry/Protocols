# Separation of Duties — Preventing Fraud Through Access Control

In 2002, a single WorldCom accountant named David Myers instructed his team to book $3.8 billion in fraudulent entries. The fraud was possible in part because the same people who could authorize journal entries could also execute them without independent review. This is the failure that Separation of Duties exists to prevent: no single person should have enough access to commit, conceal, and benefit from fraud undetected.

SoD is not a compliance checkbox. It's a fundamental access control principle — and ISC is the mechanism that makes it enforceable across dozens of systems simultaneously. Without ISC, SoD is a policy document and a prayer. With ISC, it's a continuously enforced technical control.

---

## What Separation of Duties Means in Practice

A SoD conflict exists when a single identity has access to two (or more) capabilities that, when combined, enable a harmful action without requiring another person's involvement.

**Classic financial SoD conflicts**:
- **Approve payments + Execute payments**: Can create and approve their own fraudulent payment
- **Create vendors + Approve vendor invoices**: Can create fake vendor, approve invoices to themselves
- **Access payroll + Approve payroll changes**: Can modify their own salary
- **Record journal entries + Approve journal entries**: WorldCom scenario — can book any entry without oversight

**IT SoD conflicts** (often overlooked):
- **Create user accounts + Assign privileges**: Can create admin accounts without oversight
- **Deploy to production + Approve deployments**: Can deploy unauthorized code changes
- **Access backup systems + Manage audit logs**: Can restore without detection, modify logs
- **Write firewall rules + Approve firewall changes**: Can open access without review

**Why SoD spans multiple systems**: The vendor payment conflict doesn't live in one application. "Create vendor" might be in SAP. "Approve vendor invoice" might be in an ERP module. "Execute payment" might be in a treasury system. A person who has access in all three systems can complete the fraud chain. ISC is the only component in the environment that can see access across all three systems simultaneously and evaluate the conflict.

> 🎯 **KEY POINT:** SoD enforcement that only looks at access within one system misses the most dangerous conflicts. Real fraud scenarios span multiple systems. Enterprise SoD requires cross-system visibility — which is exactly what ISC provides when all relevant systems are connected.

---

## ISC's SoD Implementation

ISC implements SoD through **Policy Violations** — rules that define which access combinations are conflicts, and a continuous evaluation engine that detects when identities hold conflicting access.

**Policy violation structure**:

A policy violation rule defines:
- **Left side**: a set of access (one or more access profiles, roles, or entitlements)
- **Right side**: a conflicting set of access
- **Violation**: any identity that holds access on both sides simultaneously

Example — Meridian Capital SoD policy:

```
Policy: Trade Execution + Trade Approval Conflict
Left side: Charles River OMS - Trade Execution (access profile)
           OR Advent - Trading Desk (access profile)
Right side: Advent - Trade Approval (access profile)
            OR Charles River - Trade Approval (access profile)
Violation: Any identity with ANY left-side AND ANY right-side access
```

This policy fires if any single analyst has trade execution capability in any system AND trade approval capability in any system — the full conflict chain, across two different applications.

**Policy types in ISC**:

| Policy Type | What It Does |
|---|---|
| Conflicting Access | Detects identities with access on both sides (standard SoD) |
| Prohibited Activity | Access that nobody should have (no valid business case) |
| Risk Score Threshold | Identities whose cumulative risk score exceeds a threshold |

---

## Designing SoD Policies

The technical configuration of an SoD policy takes 15 minutes. Designing the right policies takes weeks of business stakeholder engagement.

**Step 1 — Identify the financial/operational flows that require segregation**

Work with Finance, HR, IT security, and Internal Audit to answer: "What combinations of access would allow a single person to commit an undetected harmful act?"

For Meridian Capital, the initial SoD policy inventory (developed with Internal Audit over 4 workshops):
1. Trade execution + Trade approval (financial)
2. Vendor creation + Invoice approval + Payment authorization (financial)
3. Payroll data access + Payroll change approval (HR/financial)
4. Journal entry creation + Journal entry approval (accounting)
5. User account creation + Privilege assignment (IT)
6. Production deployment + Change approval (IT)
7. Audit log access + System configuration (IT security)
8. Client data access + Data deletion capability (data governance)

**Step 2 — Map business functions to ISC access**

Each business function in the SoD policy corresponds to specific ISC access profiles or entitlements. "Trade execution capability" maps to: Charles River Trade Execution profile, Advent Trading Desk profile, and 2 Bloomberg terminal permission groups. Build this mapping in a spreadsheet before configuring in ISC.

**Step 3 — Define violation severity and response**

Not all violations are equal:
- **Critical**: Immediate access change required. "One employee can both create and execute their own payments." Requires resolution before the end of business day.
- **High**: Resolution required within 5 business days. Manager notification, risk owner notification, remediation tracked.
- **Medium**: Resolution required within 30 days. Documented exception permitted with management approval.
- **Low**: Advisory — flagged for review at next certification. No immediate action required.

**Step 4 — Define the exception process**

Some SoD violations are legitimate. A manager covering for a vacant role may temporarily need access that creates a conflict. The process for approved exceptions:
- Request submitted with business justification
- Risk owner approval required
- Time-limited exception (maximum 90 days)
- Compensating control documented (e.g., enhanced monitoring during the exception period)
- ISC marks the violation as approved exception — removes from the outstanding violations queue

---

## Detecting Violations — Preventive vs Detective Controls

ISC SoD enforcement operates in two modes:

### Detective Control — Post-Facto Violation Detection

ISC continuously evaluates all identities against all SoD policies. When a violation is detected (because access was granted that created a conflict), ISC:
1. Marks the identity as a policy violator
2. Notifies the policy owner, the identity's manager, and the risk team
3. Creates a violation work item requiring remediation
4. Tracks the violation until it's resolved (one side of the conflict removed) or approved as an exception

Detective control catches violations after they occur. This is appropriate for violations that arise from gradual access accumulation, organizational changes, or role assignments that individually don't look problematic.

### Preventive Control — Block Access Before It Creates a Conflict

When an identity requests access that would create a SoD conflict, ISC can block the request before it's granted.

**Configuration**: Access request policy → SoD check enabled. When an identity requests access, ISC evaluates whether granting it would violate any SoD policy. If yes:
- Option 1: Deny the request automatically with a message explaining the conflict
- Option 2: Route to a risk owner for manual review with a warning
- Option 3: Allow but mark as a violation requiring immediate review

Preventive control is more secure — it stops the violation before it exists. But it requires more careful policy design, because overly broad policies block legitimate access requests and create business disruption.

> ⚠️ **COMMON MISTAKE:** Enabling preventive SoD checks without first calibrating policies against existing access patterns. If you turn on preventive SoD blocking and the policies aren't refined, every access request for commonly combined access is blocked, and the IT help desk gets flooded with "ISC blocked my access request" tickets. Run detective mode first for 30–60 days to understand actual violation patterns before enabling preventive blocking.

---

## Remediating Violations

When ISC identifies an SoD violation, someone needs to make a decision. Three resolution paths:

**Remove one side of the conflict**: The most common resolution. Determine which access is less necessary for the person's role and revoke it. ISC provides side-by-side visibility of both conflicting access sets to help the decision.

**Transfer responsibility**: Move one business function to a different person. "James Rivera has both trade execution and trade approval access — reassign trade approval to another team member." ISC tracks both access sets, so the transfer is visible.

**Approve as exception with compensating control**: When a violation is legitimate but unavoidable (small team, key person covering multiple functions), document the exception with a compensating control (e.g., manager daily review of all transactions, enhanced audit logging) and set an expiry on the exception.

**Violation aging**: Track how long violations remain open. Meridian Capital's policy: critical violations unresolved for more than 5 business days generate an automatic escalation to the CISO. High violations unresolved for 30 days go to the CFO (for financial SoD) or CIO (for IT SoD). Time pressure produces resolution.

---

## Building the SoD Policy Inventory for a New Implementation

Starting from scratch at a new organization:

**Start with Internal Audit's risk inventory**: Every organization with an audit function has some version of "sensitive access combinations" documented. This is your starting point — it reflects what the audit team already considers a risk.

**Add IT security's SoD list**: IT often has independent concerns (privileged access combinations, change management controls) that aren't in the audit inventory.

**Cross-reference against connected systems**: For each SoD conflict in the business policy, map it to specific ISC access profiles. Conflicts that span unconnected systems are policy gaps — you can document them but can't enforce them until those systems are connected.

**Start with fewer, higher-confidence policies**: 8 well-calibrated policies enforced reliably are better than 40 noisy policies generating constant false positives. Add policies incrementally as you verify each one catches real violations and doesn't trigger excessive false alarms.

**Review policies quarterly**: Business processes change. New systems are added. Existing SoD policies may need updates when the systems that host the relevant access change, when business processes are restructured, or when audit findings reveal gaps.

---

## Key Takeaways

- SoD prevents single individuals from having complete control of a harmful action by requiring multiple people to be involved — ISC enforces this continuously across all connected systems
- Real SoD conflicts span multiple systems — trade execution in one system + trade approval in another; only ISC can see both simultaneously
- Design SoD policies through stakeholder workshops with Finance, HR, IT security, and Internal Audit — the technical configuration is fast; the policy design is where the work is
- Run detective mode (post-facto detection) before enabling preventive mode (block-before-grant) — calibrate policies against real access patterns first
- Violations need defined remediation paths: remove access, transfer responsibility, or approve exception with compensating control and expiry
- 8 well-calibrated policies beat 40 noisy ones — start narrow, add coverage as confidence grows

> 🔗 **CONNECTS TO:** SoD enforces governance through policy rules applied to existing access. Event-driven integration enforces governance through real-time response to identity events. The next document covers how to move from scheduled batch processing to real-time governance triggers.
