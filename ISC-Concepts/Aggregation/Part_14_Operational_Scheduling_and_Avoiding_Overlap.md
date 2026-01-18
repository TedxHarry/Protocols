# Part 14 – Operational Scheduling and Avoiding Overlap

[⬅️ Back to Home](../README.md)

---

# Part 14 – Operational Scheduling and Avoiding Overlap

## Purpose
This part explains how to schedule aggregation like an operator, not like a gambler.

Scheduling sounds simple: pick a time and run daily.
In reality, scheduling is where many production issues are born:
- overlapping jobs
- slow sources causing backlog
- delta runs missing changes
- downstream recompute storms

The goal here is to build schedules that are predictable, calm, and safe.

---
## Where This Fits in the Master Flow

Trigger → Extract → Normalize → Persist → Correlate → Evaluate → Recompute → Publish

Scheduling controls Trigger.
But it also indirectly controls every later phase by deciding how often the engine runs and how much work it creates.

---
## Mini‑Glossary

**Schedule**  
A time rule that creates jobs automatically.

**Overlap**  
A new job starts before the previous job finished.

**Backlog**  
Jobs waiting in the queue because workers are busy.

**Burst**  
Too many jobs start at the same time.

**Downstream storm**  
A wave of recompute and indexing caused by many identity changes.

---
## The Core Scheduling Truth

Scheduling is not just about “when.”
Scheduling is about workload and dependency.

If you schedule without understanding job duration and downstream impact, you create fragile systems.

---
## Step 1: Start With a Simple Model

For each source, answer three questions:

1) How long does a typical run take?
2) How frequently does the source change?
3) How painful is it if it lags?

This gives you a schedule priority.

Example:
HR changes constantly and drives lifecycle states.
It deserves a reliable schedule.
A small app that changes weekly does not.

---
## Step 2: Understand Dependencies Between Sources

Sources are not independent.

Authoritative sources (like HR) often need to run before non‑authoritative sources.

Why:
If HR creates or updates identities, other sources need those identities to exist so they can correlate.

Example flow:
HR runs first → identities updated → AD runs second → correlates accounts → access model updated

If you reverse this:
AD runs first → accounts become uncorrelated → later HR creates identities → now you have clean‑up work.

---
## Step 3: Avoid Overlap by Design

Overlap happens when:
- a job takes longer than expected
- schedule frequency is too high
- multiple sources share the same time slot

Overlap is dangerous because jobs compete for:
- connector capacity
- worker threads
- shared state in delta tokens

A simple rule:
Never schedule a source to start again before its longest expected run time.

If a source can take 45 minutes on a bad day, don’t schedule it every 30 minutes.

---
## Step 4: Stagger Jobs, Don’t Burst

A common beginner mistake:
Run everything at midnight.

That creates:
- job queue spikes
- slow response
- recompute storms
- UI lag

Instead, stagger sources.

Think of it like traffic signals.
You don’t release every lane at once.
You sequence lanes to keep flow stable.

---
## Step 5: Respect Delta Fragility

Delta schedules look attractive because they are fast.
But delta has a memory and can break.

If you schedule delta too aggressively:
- you create constant token churn
- you increase risk of silent skips
- you create frequent downstream recompute waves

A healthier pattern:
- delta frequently
- full occasionally

Example:
Delta daily, full weekly.

This catches drift.

---
## Step 6: Plan for Downstream Work

Aggregation does not end at “accounts updated.”

Every run can trigger:
- identity evaluation
- recompute
- indexing

If you schedule heavy aggregations back‑to‑back, you can create a recompute storm.

Symptoms:
- roles appear late
- UI feels stale
- job queue grows

Scheduling must include this downstream cost.

---
## Step 7: Scheduling for Business Needs

You schedule based on what the business cares about.

Examples:
Joiners need access early in the day.
Leavers should lose access quickly.
Certifications may need fresh data before launch.

So schedule HR early.
Schedule access sources after HR.
Schedule non‑critical sources off‑peak.

---
## Running Example Schedule

Sources:
- HR (authoritative)
- AD (non‑authoritative)
- A small SaaS app

A calm schedule could be:
- HR at 5:00 AM daily
- AD at 6:00 AM daily
- SaaS app at 2:00 PM daily

No overlap, clear dependency order.

---
## Why This Matters

Bad scheduling produces:
- uncorrelated accounts
- stale identities
- delayed access removal
- backlogs that look like “ISC is broken”

Good scheduling produces:
- predictable identity freshness
- calm recompute
- fewer incidents

---
## If You See This → Do This

If queue is always full, reduce bursts and stagger.
If jobs overlap, increase spacing or reduce frequency.
If access changes lag, check recompute storms.
If non‑auth accounts uncorrelated, check source order.

---
## How People Usually Fail Here

They schedule everything at midnight.
They ignore dependency order.
They schedule delta too aggressively.
They forget downstream recompute cost.

---
## Proof Paths

UI: Job history and duration trends  
API: Job objects and timestamps  
Logs: Worker usage, queue behavior

---
## What Must Not Happen

Do not schedule overlapping jobs.  
Do not burst all sources at once.  
Do not rely on delta without periodic full runs.  
Do not ignore downstream recompute impact.

---
## Safe Fixes

Stagger schedules.  
Run authoritative sources first.  
Add periodic full runs.  
Measure job duration and adjust.

---
## Confidence Check

If you can answer these, you can design safe schedules:
- Why is overlap dangerous?
- Why should authoritative sources run first?
- Why do bursts create recompute storms?
- Why pair delta with occasional full?

---
### Navigation
⬅️ Previous: Part 13 – Proof Paths and Fix Playbooks
🏠 Home: README – Aggregation Master Series
➡️ Next: Part 15 – Custom Scheduling Patterns (Weekends, Windows, and Exceptions)

---
### Navigation
⬅️ Previous: [Part 13 – Proof Paths and Fix Playbooks](../Part_13_Proof_Paths_and_Fix_Playbooks.md)  
🏠 Home: [README – Aggregation Master Series](../README.md)  
➡️ Next: [Part 15 – Custom Scheduling Patterns (Weekends, Windows, and Exceptions)](../part_15_custom_scheduling_patterns.md)
