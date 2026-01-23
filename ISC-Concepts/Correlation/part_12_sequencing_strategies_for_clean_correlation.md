# Part 12 – Sequencing Strategies for Clean Correlation and Manager Resolution (Deep Dive)

> **Purpose of this part**  
Most correlation and manager issues are not logic problems.
They are **sequencing problems**.

This chapter teaches how to design aggregation and processing order so:
- identities exist when they are needed
- attributes are populated before they are used
- managers resolve cleanly
- correlation behaves predictably from Day 1

By the end of this part, you should:
- understand every moving engine involved in sequencing
- design safe aggregation order across sources
- avoid timing-based nulls and wrong links
- reason about sequencing like an operator, not a UI clicker

---

## 12.1 The hidden truth about ISC timing

ISC is not a single pipeline.
It is a **set of independent engines** that run in sequence or in parallel.

Key engines:
- Source aggregation
- Identity profile mapping
- Identity processing
- Correlation (account and manager)
- Governance evaluation

If you design without respecting engine order, correlation will fail even with perfect configuration.

---

## 12.2 The golden rule of sequencing

> **You cannot correlate against data that does not exist yet.**

Everything in this chapter flows from that rule.

---

## 12.3 Identity-first vs application-first worlds

### Identity-first (recommended)
- Authoritative source aggregates first
- Identities are created early
- Core identifiers are populated
- Applications correlate cleanly

### Application-first (risky)
- Applications aggregate before identities exist
- Correlation fails or falls back
- Manual fixes creep in
- Governance trust erodes

Always design for identity-first flow unless you have a strong reason not to.

---

## 12.4 The minimum safe sequencing model

A safe baseline sequence looks like this:

1. Aggregate authoritative HR source  
2. Run identity processing  
3. Verify identity creation and identifiers  
4. Aggregate downstream application sources  
5. Run correlation using stable identity state  

Skipping step 2 is the most common mistake.

---

## 12.5 Where manager correlation fits

Manager correlation depends on **both identities** existing.

Safe manager sequencing:
- HR aggregates managers early
- Manager identities exist before employees reference them
- Identity processing completes before manager resolution

If managers are created late, manager links will lag or fail.

---

## 12.6 Full sequencing flow (slow motion)

Think of one daily cycle:

1. HR aggregation starts
2. Employee accounts are pulled
3. Identity attributes populate
4. Identity processing runs
5. Manager references resolve
6. Application aggregations begin
7. Account correlation runs
8. Governance consumes results

If you change this order, outcomes change.

---

## 12.7 Parallel aggregation pitfalls

Running aggregations in parallel feels efficient.
It is also dangerous.

Risks:
- application aggregates before HR finishes
- identity attributes partially populated
- correlation sees incomplete identity state

Parallelism without dependency control creates non-deterministic behavior.

---

## 12.8 Delta aggregation and sequencing

Delta aggregation optimizes performance, not correctness.

Sequencing risk:
- HR delta misses manager updates
- application delta runs
- correlation uses stale manager links

Periodic full aggregation or identity processing is required to realign state.

---

## 12.9 Rehire and lifecycle sequencing

Rehire scenarios stress sequencing hard.

Common failure:
- old identity exists
- new hire record arrives
- application aggregates before identity cleanup
- accounts attach incorrectly

Safe pattern:
- resolve rehire identity state first
- complete identity processing
- then allow application correlation

---

## 12.10 Sequencing across environments

Lower environments often hide sequencing problems because:
- data volume is small
- runs are manual
- timing differences are invisible

Production exposes sequencing flaws brutally.

Always validate sequencing assumptions in prod-like conditions.

---

## 12.11 Designing schedules intentionally

Do not accept default schedules blindly.

Design schedules by asking:
- which source creates identities?
- which attributes must exist before correlation?
- which sources depend on those identities?
- which jobs must never overlap?

Document these answers.

---

## 12.12 Observability for sequencing

Good sequencing design includes visibility.

Track:
- aggregation start and end times
- identity processing completion
- correlation success rates per run
- manager resolution lag

Without observability, sequencing bugs look random.

---

## 12.13 Real-world example

Problem:
- HR runs at 6 AM
- Identity processing runs at 9 AM
- Applications run at 7 AM

Result:
- applications see half-baked identities
- correlation fails or falls back
- managers are null

Fix:
- move identity processing immediately after HR
- delay applications until identities settle

No config changes needed.

---

## 12.14 Operational mindset shift

Stop thinking:
> “Which setting fixes this?”

Start thinking:
> “Which engine ran too early?”

That question solves most correlation mysteries.

---

## 12.15 What this part prepares you for

After this part, you should:
- design correlation flows intentionally
- stop firefighting timing issues
- build tenants that self-heal over time

Next, we move into **end-to-end correlation troubleshooting**, tying every previous part into a single diagnostic playbook.

---

## Navigation

**⬅ Back:** Part 11 – Manager Correlation and Hierarchical Identity Links  
**➡ Next:** Part 13 – End-to-End Correlation Troubleshooting Playbook

---

*End of Part 12*
