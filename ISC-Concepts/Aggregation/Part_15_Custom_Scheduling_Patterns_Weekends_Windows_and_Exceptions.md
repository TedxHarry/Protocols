# Part 15 – Custom Scheduling Patterns (Weekends, Windows, and Exceptions)

[⬅️ Back to Home](../README.md)

---

# Part 15 – Custom Scheduling Patterns (Weekends, Windows, and Exceptions)

## Purpose
This part explains how to handle real‑world scheduling rules.

In theory, schedules are clean: run daily, same time.
In reality, businesses ask for things like:
- do not run on weekends
- run only during a maintenance window
- run twice a day on weekdays
- run full on Sunday and delta on weekdays
- pause runs during payroll cutover

This part teaches how to think through these requests safely.

---
## Where This Fits in the Master Flow

Trigger → Extract → Normalize → Persist → Correlate → Evaluate → Recompute → Publish

Custom scheduling still controls Trigger.
But the risk is bigger because the more rules you add, the easier it is to create gaps, overlaps, or delta drift.

---
## Mini‑Glossary

**Window**  
A time range where jobs are allowed.

**Blackout**  
A time range where jobs must not run.

**Exception**  
A special rule like “skip weekends” or “run full on Sunday.”

**Catch‑up run**  
A run that compensates for skipped days.

---
## The Core Principle

Custom schedules are not just calendar rules.
They change what data freshness means.

Every exception creates a data gap.
Your job is to decide whether that gap is acceptable and how to manage the consequences.

---
## Step 1: Ask What the Business Really Wants

A request like “don’t run on weekends” usually has a reason:
- source maintenance
- cost control
- performance concerns
- vendor rate limits

You should translate the request into a technical question:
“Is it acceptable if identity and access changes are stale for up to 48 hours?”

Because that is what “skip weekends” really means.

---
## Step 2: Decide Whether Delta Can Survive the Gap

Delta depends on a token.
When you skip runs for two days, delta has to cover a bigger change window.

This is usually fine, but it increases risk:
- larger change sets
- higher chance of token issues
- bigger recompute waves

If the source is large, a Monday delta might become heavy enough to behave like a full run.

---
## Step 3: Choose a Pattern That Matches Reality

Here are the patterns that work well in real ops.

### Pattern A: Weekdays Only, Same Time
Use when:
The business can tolerate weekend staleness.

What to watch:
Monday backlog and recompute storm.

---
### Pattern B: Weekdays Delta, Weekly Full
Use when:
You want speed daily but also want drift correction.

Example:
Delta Monday–Saturday, full Sunday.

This catches:
- missed delta changes
- connector drift
- token weirdness

---
### Pattern C: Two Runs Per Day
Use when:
Joiners and leavers need faster turnaround.

Example:
HR at 6 AM and 2 PM.

What to watch:
Overlap risk and downstream recompute load.

---
### Pattern D: Maintenance Windows Only
Use when:
Source system is sensitive and only allows reads during a window.

Example:
Run between 1 AM and 3 AM.

What to watch:
If a job slips outside window, it must not retry endlessly.

---
### Pattern E: Blackout During Critical Events
Use when:
There is a known business event like payroll cutover.

What to watch:
Plan a catch‑up run after blackout ends.

---
## Step 4: Plan the Catch‑Up Run

If you skip runs, you need to decide how to catch up.

Two common choices:

1) Monday delta catch‑up
You run delta and accept the larger change set.

2) Monday full reset
You force a full to rebuild truth.

Full reset is heavier but safer when you suspect delta drift.

---
## Step 5: Respect Dependency Order Even With Exceptions

Skipping weekends often breaks the normal “HR then AD” chain.

Example:
If HR doesn’t run Saturday and Sunday, but AD does run, AD will see stale identities.

So you should apply exceptions consistently:
If HR skips weekends, downstream sources that depend on HR should also skip or adjust.

---
## Step 6: Prevent Overlap With Custom Rules

Custom rules often create accidental overlaps.

Example:
Two runs per day plus a weekly full.

If the full run takes longer, it can overlap with the next delta.

So you must build spacing based on worst‑case duration, not average duration.

---
## Running Example: “Skip Weekends for HR”

Business request:
Do not aggregate HR on Saturday and Sunday.

Operator translation:
Identity freshness can be up to 48 hours stale.

Safe schedule:
- HR runs Monday–Friday at 5 AM
- HR runs a full on Sunday night or early Monday once
- AD runs after HR on weekdays only

Why this works:
HR stays authoritative and fresh on business days.
AD only runs when identities are updated.
Full run catches drift.

---
## Why This Matters

Custom schedules create real risk:
- joiners delayed
- leavers keep access longer
- delta drift grows
- Monday storms happen

If you design the pattern intentionally, these risks become manageable.

---
## If You See This → Do This

If Monday runs are heavy, add a weekly full or add a Saturday light run.
If access changes lag after weekends, align dependent sources.
If delta starts skipping changes, schedule periodic full and monitor token behavior.

---
## How People Usually Fail Here

They implement calendar rules without thinking about freshness.
They skip HR but keep downstream sources running.
They don’t plan catch‑up.
They create overlaps with weekly full runs.

---
## Proof Paths

UI: Job history and durations across week  
API: Job timestamps and run types  
Logs: Delta token usage across gaps

---
## What Must Not Happen

Do not add exceptions without understanding data staleness.  
Do not break dependency order.  
Do not create overlap with full runs.  
Do not ignore Monday backlog.

---
## Safe Fixes

Align exceptions across dependent sources.  
Add weekly full runs.  
Stagger Monday schedules.  
Measure worst‑case duration.

---
## Confidence Check

If you can answer these, you can design custom schedules:
- What does “skip weekends” really mean in data freshness?
- Why can Monday delta become heavy?
- Why must dependent sources follow the same exceptions?
- When should you add a catch‑up full run?

---
### Navigation
⬅️ Previous: Part 14 – Operational Scheduling and Avoiding Overlap
🏠 Home: README – Aggregation Master Series
➡️ Next: Part 16 – Observability and Monitoring

---
### Navigation
⬅️ Previous: [Part 14 – Operational Scheduling and Avoiding Overlap](../Part_14_Operational_Scheduling_and_Avoiding_Overlap.md)  
🏠 Home: [README – Aggregation Master Series](../README.md)  
➡️ Next: [Part 16 – Observability and Monitoring](../part_16_observability_and_monitoring.md)
