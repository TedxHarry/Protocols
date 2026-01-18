# Part 18 – Governance Impacts (Certifications, Requests, and Policies)

[⬅️ Back to Home](../README.md)

---

# Part 18 – Governance Impacts (Certifications, Requests, and Policies)

## Purpose
This part explains why aggregation is not “just data.”

Aggregation feeds governance.
Governance is where people make decisions.

If aggregation truth is wrong or stale, governance becomes dangerous:
- certifications review the wrong access
- requests approve access for the wrong identity
- policies fire incorrectly or fail to fire

This part connects the engine to the human world.

---
## Where This Fits in the Master Flow

Trigger → Extract → Normalize → Persist → Correlate → Evaluate → Recompute → Publish

Governance consumes what the engine produces.
Once the engine publishes updated identity and access truth, governance features act on it.

---
## Mini‑Glossary

**Certification**  
A review of access where someone must approve or revoke.

**Access request**  
A workflow where someone asks for access.

**Policy**  
A rule that detects or prevents risky access.

**Entitlement**  
A granular access item like a group or permission.

**Role / Access profile**  
Access bundles built from entitlements.

---
## The Core Idea

Governance assumes identity and access truth is accurate.

When truth is wrong, governance doesn’t stop.
It simply produces the wrong decisions at scale.

So the quality of aggregation is the quality of governance.

---
## Step 1: Understand What Governance Uses

Governance features rely on:
- identity attributes (department, manager, location)
- account state (active/disabled)
- entitlements and memberships
- role and access profile assignments

These all come from the engine.

If any upstream phase is wrong, governance is wrong.

---
## Step 2: Certifications and Aggregation Truth

Certifications answer:
“Does this person still need this access?”

But reviewers can only judge what they can see.

If aggregation is stale:
- leavers still appear active
- terminated accounts still appear present
- old entitlements still appear assigned

Then reviewers approve access that should have been removed.

This is how staleness becomes risk.

---
## Step 3: Certification Scope Depends on Identity Data

Certification campaigns often use identity fields:
- by department
- by manager’s team
- by location
- by lifecycle state

If identity profile evaluation is wrong, campaign scope is wrong.

Example:
Alice moved to Engineering, but identity still says Sales.
Engineering access campaign misses her.
Sales campaign includes her incorrectly.

The wrong people review the wrong access.

---
## Step 4: Access Requests Depend on Correct Identity

Requests depend on:
- who the requester is
- what roles they are eligible for
- who should approve

All of these depend on identity truth.

If manager data is wrong, approvals go to the wrong person.
If department is wrong, eligibility logic may fail.

So even if the request workflow is perfect, the inputs can be wrong.

---
## Step 5: Requests Depend on Correct Access Model

Many request systems hide items you already have.

If access model is stale:
- user may request access they already have
- or cannot request access they should be able to see

This creates duplicate work and confused approvals.

Recompute health directly affects request quality.

---
## Step 6: Policies Depend on Clean Entitlements and Identity

Policies detect risk such as:
- separation of duties
- privileged access
- toxic combinations

Policies need two things:
- correct entitlement assignments
- correct identity context

If entitlements are missing, policy will not fire.
If identity attributes are wrong, policy may fire on the wrong people.

So policy tuning is useless if aggregation is incomplete.

---
## Step 7: How Aggregation Failures Show Up in Governance

### Pattern A: Stale access in certifications
Root cause is usually:
- aggregation schedule gaps
- delta skipping
- recompute storm

### Pattern B: Wrong reviewer or approval path
Root cause is usually:
- identity profile precedence
- manager attribute wrong

### Pattern C: Policies not triggering
Root cause is usually:
- missing entitlements
- membership not aggregated
- mapping errors

Governance symptoms are often downstream of upstream failures.

---
## Running Example: A Leaver Risk

Bob leaves the company on Friday.
HR marks him terminated.

But HR aggregation skips weekends.
Bob remains Active in ISC through Sunday.

On Saturday:
- Bob still appears in certifications
- policies still treat him as active
- access requests may still be possible

This is not a governance failure.
This is a scheduling and freshness failure.

---
## Step 8: The Right Way to Operate Governance

If governance decisions matter, you need freshness SLOs.

Examples:
- HR lifecycle changes must appear within 2 hours
- leaver removals must be same day

Then you design schedules and monitoring to meet those SLOs.

Governance is only as strong as your freshness discipline.

---
## Proof Paths

UI: campaign scope, reviewer assignments, policy violations  
API: identity fields, access model, entitlement assignments  
Logs: aggregation steps and recompute timing

When governance looks wrong, prove upstream truth first.

---
## What Must Not Happen

Do not run certifications on stale data.  
Do not trust policy silence if entitlements are missing.  
Do not assume approval flows are correct if manager data is wrong.

---
## Safe Fixes

Fix identity truth first, then rerun recompute.  
Validate entitlement coverage before policy tuning.  
Align HR schedules to governance freshness needs.

---
## Confidence Check

If you can answer these, you understand the governance impacts:
- Why is aggregation quality equal to governance quality?
- How does stale data corrupt certifications?
- Why do approvals go wrong when identity data is wrong?
- Why do policies fail when entitlement aggregation is incomplete?

---
### Navigation
⬅️ Previous: Part 17 – Performance Tuning and Scale
🏠 Home: README – Aggregation Master Series
➡️ Next: Part 19 – Change Management and Safe Updates

---
### Navigation
⬅️ Previous: [Part 17 – Performance Tuning and Scale](../Part_17_Performance_Tuning_and_Scale.md)  
🏠 Home: [README – Aggregation Master Series](../README.md)  
➡️ Next: [Part 19 – Change Management and Safe Updates](../part_19_change_management_and_safe_updates.md)
