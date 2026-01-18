# Part 16 – Observability and Monitoring

[⬅️ Back to Home](../README.md)

---

# Part 16 – Observability and Monitoring

## Purpose
This part explains how to *see* aggregation health before users feel pain.

Most teams notice aggregation problems only when:
- a joiner can’t get access
- a leaver still has access
- certifications look wrong
- someone says “data is stale again”

That is too late.

Observability means you can answer, any time:
- Are jobs running on schedule?
- Are they getting slower?
- Are they skipping data?
- Are they producing more uncorrelated accounts?
- Are recompute and publish keeping up?

---
## Where This Fits in the Master Flow

Trigger → Extract → Normalize → Persist → Correlate → Evaluate → Recompute → Publish

Monitoring sits across the whole engine.
It watches the system and tells you when one phase is drifting.

---
## Mini‑Glossary

**Observability**  
The ability to understand system health from signals.

**Monitoring**  
Alerts and dashboards based on those signals.

**Lag**  
The delay between source change and ISC reflecting it.

**Drift**  
Gradual loss of correctness over time, usually from delta issues.

**SLO**  
A promised target like “HR data must be < 2 hours old.”

---
## The Core Mindset

Do not monitor only job status.
Status is the weakest signal.

Instead, monitor the outcomes that matter:
- freshness
- counts
- correlation health
- failure patterns

A job can be Completed and still be unhealthy.

---
## Step 1: Define What “Healthy” Means

Every source needs a health definition.

For HR, health might mean:
- runs daily before 7 AM
- changes appear in ISC within 2 hours
- uncorrelated count stays near zero

For a small SaaS app, health might mean:
- runs weekly
- drift acceptable

If you don’t define health, you can’t monitor it.

---
## Step 2: Monitor Job Execution Health

This is the basic layer.

Watch:
- job start time vs scheduled time
- job duration trend
- queue time
- failure frequency

Why this matters:
If duration grows slowly over weeks, you will hit overlap and backlog later.

---
## Step 3: Monitor Data Freshness

Freshness is the most business‑meaningful signal.

A simple idea:
Measure the time between a known change in the source and when it appears in ISC.

You can track freshness via:
- timestamps on accounts
- lastSeen / lastUpdated metadata
- identity attribute timestamps

This tells you whether your pipeline is actually delivering.

---
## Step 4: Monitor Volume and Counts

Counts reveal silent failures.

Track:
- number of accounts read
- number of accounts updated
- number of entitlements read
- number of memberships processed

If pagination breaks, counts suddenly drop.
If delta acts like full, counts suddenly spike.

Counts are your early warning.

---
## Step 5: Monitor Correlation Health

Correlation health tells you whether your identity model is stable.

Track:
- number of uncorrelated accounts
- number of ambiguous accounts
- number of identities created from authoritative sources

Spikes here usually indicate:
- match key drift
- duplicate keys
- identity profile changes
- source order changes

Correlation metrics are security metrics.

---
## Step 6: Monitor Identity Evaluation Health

Identity profile issues often appear as:
- precedence mistakes
- derived fields breaking
- lifecycle state wrong

Track:
- number of identities with empty critical fields
- lifecycle state distribution changes
- sudden flips in attribute source

These signals tell you your identity truth is degrading.

---
## Step 7: Monitor Recompute and Downstream Load

Recompute is where the pipeline turns into access change.

Track:
- time between identity update and role update
- backlog of recompute work
- spikes in role changes

If recompute cannot keep up, access will lag even if aggregation is perfect.

---
## Step 8: Monitor Publish and UI Visibility

UI delay is normal sometimes.
But extreme delay becomes operational risk.

Track:
- API shows correct state but UI stale
- time to appear in search
- indexing backlog indicators

This helps you distinguish:
Data wrong vs data not visible.

---
## Practical Monitoring Dashboard (What to Put on One Page)

For each source, a dashboard should show:

Job:
- last run time
- duration trend
- queue time
- failures

Data:
- accounts read
- entitlements read
- memberships processed
- update counts

Identity:
- uncorrelated count
- ambiguous count
- identity creation count

Freshness:
- age of latest account update
- age of latest identity update
- age of latest access model update

One page, calm signals.

---
## Alerts That Actually Matter

Avoid alert spam.

Good alerts are about meaningful thresholds:

- HR job did not run by 7 AM
- HR duration increased by 2x
- accounts read dropped by 50%
- uncorrelated accounts spiked
- freshness exceeded SLO

These alerts catch silent failures early.

---
## Running Example: Detecting Pagination Failure

Yesterday:
HR accounts read = 10,000

Today:
HR accounts read = 500

Job status = Completed

Monitoring catches it because counts dropped.
You investigate extraction logs and find pagination stopped after first page.

Without monitoring, you would discover this only when joiners are missing.

---
## Why This Matters

Monitoring is not a luxury.
It prevents:
- stale identities
- delayed leaver removal
- broken certifications
- access drift

If you monitor the right things, incidents become predictable instead of surprising.

---
## What Must Not Happen

Do not monitor only status.  
Do not ignore counts.  
Do not ignore correlation health.  
Do not accept staleness without an SLO.

---
## Safe Next Steps

Define health per source.
Start with 5 metrics:
- last run time
- duration
- accounts read
- uncorrelated count
- freshness

Then expand once those are stable.

---
## Confidence Check

If you can answer these, you understand observability:
- Why status is weak signal?
- Which metrics catch silent failures?
- How do you measure freshness?
- Why is correlation monitoring a security measure?

---
### Navigation
⬅️ Previous: Part 15 – Custom Scheduling Patterns (Weekends, Windows, and Exceptions)
🏠 Home: README – Aggregation Master Series
➡️ Next: Part 17 – Performance Tuning and Scale

---
### Navigation
⬅️ Previous: [Part 15 – Custom Scheduling](./Part_15_Custom_Scheduling_Patterns_Weekends_Windows_and_Exceptions.md)  
🏠 Home: [README](./README.md)  
➡️ Next: [Part 17 – Performance Tuning](./Part_17_Performance_Tuning_and_Scale.md)

