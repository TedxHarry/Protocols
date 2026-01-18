# Part 20 – Security and Compliance Considerations

[⬅️ Back to Home](../README.md)

---

# Part 20 – Security and Compliance Considerations

## Purpose
This part explains how aggregation affects security and compliance.

Aggregation is not neutral plumbing.
It decides:
- who appears to exist
- what access they appear to have
- when access is removed

Security and compliance trust that this truth is accurate and timely.
If aggregation lies or lags, security decisions become wrong even if policies are perfect.

---
## Where This Fits in the Master Flow

Trigger → Extract → Normalize → Persist → Correlate → Evaluate → Recompute → Publish

Security and compliance sit on top of the engine.
They consume the truth the engine produces and turn it into controls, reviews, and evidence.

---
## Mini‑Glossary

**Least privilege**  
Only the access a person truly needs.

**Joiner / Mover / Leaver**  
Lifecycle events that drive access change.

**Audit trail**  
Evidence showing what happened and when.

**Attestation**  
Formal review of access (certification).

**SLA / SLO**  
Time promise for how fast changes must appear.

---
## The Core Security Truth

Security depends on freshness and correctness.

If a leaver still appears active for two days, that is a security gap.
If a joiner does not appear, that is an operational gap.
If access is missing or extra, that is a risk gap.

Aggregation quality is security quality.

---
## Step 1: Joiners, Movers, Leavers Are Security Events

Lifecycle changes are not HR events.
They are security triggers.

Joiner:
- new identity appears
- baseline access must be granted

Mover:
- identity attributes change
- some access must be removed
- some access must be added

Leaver:
- identity must become inactive
- all access must be removed

If aggregation delays these, security is delayed.

---
## Step 2: Staleness Equals Risk

Every hour of staleness is a window of risk.

Examples:
- Leaver access remains active
- Contractor end date passes but account still active
- Department change not reflected, so toxic access stays

So you must define freshness SLOs for security‑critical sources like HR.

---
## Step 3: Identity Truth Drives Policy Truth

Policies use:
- identity attributes
- lifecycle state
- access assignments

If identity truth is wrong:
- SoD policies fire on wrong people
- privileged access not flagged
- risky combinations missed

So policy tuning without fixing aggregation is fake security.

---
## Step 4: Correlation Is a Security Control

Correlation decides which account belongs to which person.

If correlation is wrong:
- one person may “own” another person’s access
- access reviews go to the wrong manager
- audit evidence is misleading

Weak match keys are not just technical debt.
They are security debt.

---
## Step 5: Deprovisioning Depends on Aggregation

Automated deprovisioning relies on:
- lifecycle state
- account state
- access model updates

If aggregation does not mark someone inactive:
Provisioning cannot remove access.

So many “provisioning failures” are actually aggregation failures.

---
## Step 6: Evidence and Audit Trails

Auditors ask:
- When did Bob leave?
- When was his access removed?
- Who approved what and when?

To answer, you need:
- source timestamps
- aggregation timestamps
- recompute timestamps
- provisioning timestamps

If aggregation timing is unclear, your audit story is weak.

---
## Step 7: Certifications as Compliance Control

Certifications assume data is correct.

If aggregation is stale:
- reviewers approve wrong access
- compliance reports show false comfort

So before launching a campaign, you must prove data freshness.

---
## Step 8: Security‑Driven Scheduling

Security should influence schedules.

Examples:
- HR must run early in the day
- Leaver updates must not wait until next morning
- Delta may be too risky for leaver‑heavy sources

Scheduling is a security decision, not just an ops decision.

---
## Running Example: Weekend Leaver Risk

Bob leaves Friday night.
HR updates status Friday.
Aggregation skips weekends.
Bob still appears active until Monday.

For two days:
- access not removed
- policies think Bob is active
- certifications may include him

This is not a tooling issue.
This is a scheduling and freshness decision that created risk.

---
## Step 9: Designing for Compliance

To support compliance, you need:
- defined freshness SLOs
- monitoring of staleness
- proof paths for timing
- repeatable deprovisioning evidence

Compliance is not about passing audits once.
It is about being able to prove truth any day.

---
## Proof Paths

UI: lifecycle state, certification scope, policy violations  
API: identity attributes, timestamps, access model  
Logs: job timing, recompute timing, provisioning actions

Security questions always start by proving aggregation truth first.

---
## What Must Not Happen

Do not treat lifecycle as “just HR data.”  
Do not accept staleness for leaver data.  
Do not tune policies while identity truth is broken.  
Do not ignore correlation quality.

---
## Safe Practices

Run authoritative sources early and often.
Monitor freshness for security‑critical data.
Use strong, stable match keys.
Test deprovisioning paths regularly.

---
## Confidence Check

If you can answer these, you understand security impacts:
- Why is aggregation quality equal to security quality?
- Why is staleness a risk window?
- Why is correlation a security control?
- Why is scheduling a security decision?

---
### Navigation
⬅️ Previous: Part 19 – Change Management and Safe Updates
🏠 Home: README – Aggregation Master Series
➡️ Next: Part 21 – Disaster Scenarios and Recovery

---
### Navigation
⬅️ Previous: [Part 19 – Change Management](./Part_19_Change_Management_and_Safe_Updates.md)  
🏠 Home: [README](./README.md)  
➡️ Next: [Part 21 – Disaster Scenarios](./Part_21_Disaster_Scenarios_and_Recovery.md)

