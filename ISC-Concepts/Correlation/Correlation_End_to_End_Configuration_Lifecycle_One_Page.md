# SailPoint ISC Correlation – One-Page End-to-End Configuration Lifecycle

> **Goal**  
This page shows the *entire correlation configuration lifecycle* from Day 0 to steady-state operations.  
If you follow this flow, correlation will be **predictable, auditable, and maintainable**.

---

## 1. Identity Foundation (Day 0 – Non‑Negotiable)

Everything starts here. If this layer is weak, nothing above it is trustworthy.

Configure:
- Exactly **one authoritative source** to *create identities*
- Identity profile with:
  - immutable / stable identifiers (employeeId, workerId)
  - lifecycle-critical attributes
- Identity processing enabled and scheduled immediately after HR aggregation

Outcome:
- Identities exist early
- Core identifiers are populated before anything else runs

---

## 2. Attribute Readiness Design

Before touching correlation, decide:

For every identifier you plan to use:
- where it comes from
- when it is populated
- whether it is stable over time
- whether it must be searchable

Configure:
- identity attribute mappings
- custom identity attributes (if needed)
- searchability only where required

Outcome:
- identifiers exist **at correlation runtime**
- no late or drifting attributes

---

## 3. Correlation Criteria Design (Primary Logic)

Now design how accounts *infer ownership*.

Configure:
- correlation criteria on each source
- strongest identifier first
- weakest last
- avoid unnecessary rows

Rules:
- order criteria by **confidence**
- document why each row exists
- never rely on default fallback as a strategy

Outcome:
- most accounts correlate cleanly
- logic is explainable

---

## 4. Correlation Rules (Only If Needed)

Use rules only for cases criteria cannot express.

Configure:
- rules that handle **exceptions**
- return “no decision” for normal accounts
- document intent clearly

Avoid:
- rules that always return an identity
- replacing criteria entirely with code

Outcome:
- edge cases handled
- core logic remains visible

---

## 5. Manager Correlation Configuration

Manager correlation is identity‑to‑identity resolution.

Configure:
- stable manager identifier (prefer immutable HR IDs)
- mappings to identity attributes
- sequencing so manager identities exist first

Validate:
- no self‑managers
- no obvious loops
- expected nulls are intentional

Outcome:
- approvals and certifications route correctly

---

## 6. Sequencing and Scheduling (The Hidden Control Plane)

This decides whether your configuration actually works.

Design schedule:
1. Authoritative source aggregation
2. Identity processing
3. Manager resolution
4. Application aggregations
5. Correlation execution

Avoid:
- unsafe parallel runs
- application aggregation before identity stabilization

Outcome:
- deterministic behavior
- no timing‑based surprises

---

## 7. Initial Validation (First Runs)

Validate with **real accounts**, not percentages.

Check:
- which criteria matched
- which attributes existed at runtime
- whether fallback triggered
- whether identities existed when needed

Do not:
- manually correlate to “fix” early issues
- optimize before understanding behavior

Outcome:
- confidence in real behavior

---

## 8. Steady-State Operations

Once stable, operate intentionally.

Monitor:
- uncorrelated accounts (expected vs unexpected)
- manager nulls
- rule usage
- manual correlations count

Educate teams:
- delta vs full aggregation behavior
- persistence of decisions

Outcome:
- correlation becomes boring (this is good)

---

## 9. Change Management (Apps, Rehires, M&A)

Before changes, always ask:
- does the identifier still mean the same thing?
- does sequencing still hold?
- will old decisions persist?

During change:
- reduce automation temporarily
- accept higher uncorrelated counts
- favor traceability over coverage

Outcome:
- governance integrity preserved during change

---

## 10. Periodic Design Review (Governance Readiness)

On a schedule, review:
- authoritative sources
- identifier stability
- criteria relevance
- rule necessity
- manual correlation debt

Remove:
- obsolete rules
- historical workarounds
- assumptions that no longer hold

Outcome:
- correlation stays defensible over time

---

## The One‑Line Lifecycle Summary

> **Prepare identities → Prepare attributes → Infer carefully → Override rarely → Sequence intentionally → Operate defensibly**

---

**Use this page as the configuration lifecycle map.**  
If a problem appears, ask: *Which step did we skip or rush?*
