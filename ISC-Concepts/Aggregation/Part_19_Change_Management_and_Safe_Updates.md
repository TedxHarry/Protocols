# Part 19 – Change Management and Safe Updates

[⬅️ Back to Home](../README.md)

---

# Part 19 – Change Management and Safe Updates

## Purpose
This part explains how to make changes to aggregation safely.

In real life, you will constantly change things:
- add new attributes
- change mappings
- adjust filters
- change match keys
- update identity profile precedence
- upgrade connectors

Each change can silently rewrite truth.
So change management is not a “process thing.”
It is how you prevent accidents.

---
## Where This Fits in the Master Flow

Trigger → Extract → Normalize → Persist → Correlate → Evaluate → Recompute → Publish

Changes can affect any phase.
But most risky changes touch:
- extraction scope
- mapping and transforms
- unique ID and match keys
- identity profile precedence
- delta state

---
## Mini‑Glossary

**Change**  
Any modification to configuration, schema, mapping, logic, or connector.

**Blast radius**  
How many identities and systems the change can affect.

**Rollback**  
Your ability to undo the change safely.

**Dry run**  
Testing on a small scope before full rollout.

**Drift**  
Slow loss of correctness after repeated changes.

---
## The Core Principle

Never push a change directly into full population without proof.

Every change must answer:
- What will change?
- Who will it impact?
- How will I prove it worked?
- How will I undo it?

This is the difference between clean operations and chaos.

---
## Step 1: Classify the Change Type

Not all changes are equal.

### Low risk changes
- Adding a new non‑critical attribute
- Cosmetic formatting changes

### Medium risk changes
- Adding new entitlements
- Modifying transforms
- Adjusting schedules

### High risk changes
- Changing unique ID
- Changing match keys
- Changing identity profile precedence for critical fields
- Resetting delta
- Connector upgrades

Classification matters because it tells you how much proof you need.

---
## Step 2: Predict What Will Change

Before you touch anything, predict the impact.

Examples:

If you change mapping for department:
- Many identities may update department
- Roles may change
- Recompute storm possible

If you change match key:
- Accounts may uncorrelate
- Accounts may relink to different identities
- Certifications may shift

If you upgrade connector:
- delta behavior might reset
- schema discovery might change
- pagination behavior might change

The best operators predict before they push.

---
## Step 3: Reduce Blast Radius With Scope

Never test on full population first.

Use scope to test:
- one identity
- one department
- one OU
- small filter set

This gives fast feedback and limits damage.

---
## Step 4: Use a Proof Plan

Every change needs a proof plan.

A proof plan is simply:
- what you will check in source
- what you will check in account attributes
- what you will check in identity
- what you will check in access model

If you can’t describe your proof plan, you are not ready to change.

---
## Step 5: Implement the Change in the Safest Order

Safer order means:
- change one thing at a time
- validate the layer it affects
- then move to next

Example: adding a new identity attribute
1) update schema and mapping
2) verify account attributes
3) verify identity profile pulls it
4) recompute if needed

Do not change mapping, identity profile, and role rules all together.
You won’t know what caused the outcome.

---
## Step 6: Plan Rollback Before You Need It

Rollback is not an emergency option.
It is part of the change.

Rollback might be:
- restore old mapping
- restore old precedence
- restore old connector version

If rollback is hard, treat the change as higher risk.

---
## Step 7: Understand “Changes That Rewrite History”

Some changes do not just affect future runs.
They can rewrite past truth.

Examples:
- changing unique ID can duplicate or orphan accounts
- changing match key can relink accounts
- changing mapping can change stored account attributes

These changes can reshape what governance believes about history.

Treat them with extreme caution.

---
## Running Example: Changing Department Mapping

You realize HR department is stored in a different field.
You want to fix mapping.

Risk prediction:
- department will change for many identities
- role assignments may change
- recompute storm possible

Safe approach:
- scope test on one department
- verify account attribute change
- verify identity field updates
- measure how many identities changed
- stagger recompute

Wrong approach:
Change mapping and run full immediately.

---
## Running Example: Changing Match Key

You want to move from email to employeeId.

This is high risk.

Safe approach:
- prove employeeId exists everywhere
- test correlation on small scope
- check ambiguous and uncorrelated counts
- only then migrate gradually

Wrong approach:
Flip match key and hope.

---
## Why This Matters

Most aggregation incidents are not random.
They are caused by unplanned changes.

Bad change management produces:
- silent drift
- unexpected recompute storms
- broken correlation
- governance confusion

Good change management produces predictable evolution.

---
## Proof Paths

UI: mapping previews, job summaries, campaign scope changes  
API: account attributes, identity attributes, access model  
Logs: schema changes, token resets, correlation changes

---
## What Must Not Happen

Do not change high‑risk keys without scoped testing.  
Do not change multiple layers at once.  
Do not reset delta without a proof plan.  
Do not upgrade connectors without monitoring impact.

---
## Safe Fixes

If drift appears after a change, rollback first, investigate second.
If recompute storm appears, stagger and reduce noisy updates.
If correlation breaks, restore previous key and reprocess slowly.

---
## Confidence Check

If you can answer these, you can manage changes safely:
- Which changes are high risk and why?
- How do you reduce blast radius?
- What is a proof plan?
- Why must rollback be planned before change?

---
### Navigation
⬅️ Previous: Part 18 – Governance Impacts (Certifications, Requests, and Policies)
🏠 Home: README – Aggregation Master Series
➡️ Next: Part 20 – Security and Compliance Considerations

---
### Navigation
⬅️ Previous: [Part 18 – Governance Impacts (Certifications, Requests, and Policies)](../Part_18_Governance_Impacts_Certifications_Requests_and_Policies.md)  
🏠 Home: [README – Aggregation Master Series](../README.md)  
➡️ Next: [Part 20 – Security and Compliance Considerations](../part_20_security_and_compliance_considerations.md)
