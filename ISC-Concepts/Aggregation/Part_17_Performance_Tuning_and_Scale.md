
# Part 17 – Performance Tuning and Scale

[⬅️ Back to Home](../README.md)

---

## Purpose
This part explains how aggregation performance behaves at scale, and how to tune it without breaking correctness.

When teams grow, the system grows too:
More identities.
More accounts.
More entitlements.
More joins and leavers.

Aggregation that felt fine at 1,000 accounts can become painful at 100,000.

Performance tuning is not just “make it faster.”
It is “make it faster without losing truth.”

---
## Where This Fits in the Master Flow

Trigger → Extract → Normalize → Persist → Correlate → Evaluate → Recompute → Publish

Performance can bottleneck any phase.
But the most common bottlenecks happen in:
- Extract (network, pagination)
- Persist (writes)
- Correlate (matching)
- Recompute (access model)
- Publish (indexing)

---
## Mini‑Glossary

**Throughput**  
How much data you can process per minute.

**Latency**  
How long one run takes end‑to‑end.

**Backlog**  
Jobs waiting because workers are busy.

**Hotspot**  
A phase or object type consuming most time.

**Scope**  
Limiting what you read or process.

---
## The Core Principle

Do not tune blindly.

First, prove where the time is going.
Then tune the smallest part.

Most “performance tuning” disasters happen when people change multiple variables without measurement.

---
## Step 1: Measure Before You Tune

Pick one source.
Collect these metrics over time:
- job duration
- queue time
- accounts read
- entitlements read
- memberships processed
- identities touched
- roles changed

Now you can see whether slowdown is:
- more data
- slower source
- slower workers
- downstream recompute

Without this baseline, you are guessing.

---
## Step 2: Understand the Three Real Cost Drivers

### Cost driver 1: How much you read
Accounts, entitlements, and memberships volume.

### Cost driver 2: How much you change
A run that reads 10,000 records but updates only 10 is cheaper than a run that updates 5,000.
Writes are expensive.

### Cost driver 3: How many identities you touch
Identity evaluation and recompute cost grows with identities touched.

Many teams focus on reading speed, but the real cost is often recompute.

---
## Step 3: Tune Extraction Safely

Extraction speed depends on:
- API rate limits
- pagination size
- network latency
- filtering

Safe tuning moves:
- Adjust page size carefully
- Use filters only when business allows
- Prefer delta when it is stable

Dangerous tuning moves:
- Aggressive filtering that hides data
- Over‑tight delta windows

The fastest extraction is useless if it skipped the joiner.

---
## Step 4: Reduce Write Pressure

Persistence cost increases when you update many objects.

Common causes of unnecessary writes:
- mapping that changes formatting every run (case, whitespace)
- transforms that generate new values each time
- unstable fields used as unique ID

If you see high update counts every run, ask:
“Are these real changes, or noisy changes?”

Noisy changes create recompute storms.

---
## Step 5: Make Correlation Cheaper and Safer

Correlation becomes expensive when match keys are weak or inconsistent.

Good match keys:
- unique
- stable
- present on all accounts

Bad match keys:
- email when email changes frequently
- usernames that vary by system

A strong match key improves:
- performance
- correctness

This is one of the rare places where the safe option is also the fast option.

---
## Step 6: Control Recompute Storms

Recompute is often the biggest performance cost.

Why:
Every identity change can trigger:
- role evaluation
- access profile evaluation
- provisioning decisions
- indexing

If your mapping causes thousands of identities to change every day, recompute will drown.

Ways to control storms:
- Fix noisy attribute changes
- Stagger source schedules
- Recompute only when needed

The goal is fewer unnecessary identity changes.

---
## Step 7: Scale Scheduling, Not Frequency

A common mistake is to run more frequently to “stay fresh.”

If the system is already behind, running more often makes backlog worse.

Better approach:
- reduce overlap
- stagger heavy sources
- run authoritative sources first
- monitor freshness SLOs

Freshness comes from stability, not from panic frequency.

---
## Step 8: Use Scope as a Tool

When troubleshooting or tuning, do not run full population.

Use scope:
- test on one identity
- test on one department
- test on small OU

This reduces blast radius and gives faster feedback.

---
## Running Example: Monday Storm

You skip HR on weekends.
Monday delta reads a large change set.
Thousands of identities update.
Recompute storms.
UI lags.

Fix is not “run again.”
Fix is:
- add a Sunday full or light run
- stagger downstream sources
- reduce noisy updates

---
## Why This Matters

Performance problems become business problems:
- joiners delayed
- leavers not removed
- access changes late
- certifications stale

The fastest pipeline is useless if it is not trustworthy.

---
## If You See This → Do This

If duration grows slowly, suspect volume growth or worker contention.
If update counts are huge daily, suspect noisy mapping.
If roles lag, suspect recompute storm.
If queue time grows, stagger schedules.

---
## How People Usually Fail Here

They tune without measurement.
They filter aggressively and lose truth.
They ignore recompute as a cost driver.
They increase frequency and create backlog.

---
## Proof Paths

UI: duration trends and job history  
API: counts of updates and identities touched  
Logs: phase timings, pagination, rate limit hits

---
## What Must Not Happen

Do not trade correctness for speed.  
Do not tune without baseline metrics.  
Do not create performance “fixes” that hide data.

---
## Confidence Check

If you can answer these, you can tune safely:
- What are the three real cost drivers?
- Why writes and recompute matter more than reads?
- What is a recompute storm?
- How do you tune without losing truth?

---
### Navigation
⬅️ Previous: [Part 16 – Observability and Monitoring](../Part_16_Observability_and_Monitoring.md)  
🏠 Home: [README – Aggregation Master Series](../README.md)  
➡️ Next: [Part 18 – Governance Impacts (Certifications, Requests, and Policies)](../part_18_governance_impacts.md)
