# Part 2 – Account Correlation Configuration and Criteria Design

> **Purpose of this part**  
This chapter teaches how to *design* correlation configuration so it behaves predictably in production.
Not just where to click, but **why each decision matters**.

By the end of this part, you should:
- understand what correlation configuration actually represents
- know how ISC evaluates criteria internally
- design safe, intentional correlation rules
- avoid silent duplicate identity creation

---

## 2.1 Correlation configuration is a decision model

When you configure account correlation, you are not writing logic.
You are declaring **decision priority**.

Each row answers this question:

> “If this attribute matches, should ISC stop and assign ownership?”

The entire configuration is simply:
- a list of questions
- evaluated in order
- stopping at the first yes

---

## 2.2 Where correlation configuration lives

Correlation is configured **per source**.

Path:
Admin → Connections → Sources → Select Source → Account Management → Account Correlation

This matters because:
- each source can correlate differently
- correlation behavior is not global
- order can vary by source risk

---

## 2.3 What a single criteria row really means

A single correlation row means:

> “If the account’s value equals an identity’s value, assign ownership and stop.”

Key truths:
- equality only
- no OR logic inside a row
- no cross-attribute comparison
- no fallback inside the same row

One row equals one attempt.

---

## 2.4 Multiple criteria form a priority chain

Multiple rows form a **priority ladder**.

ISC does not combine rows.
It evaluates them one at a time.

Example order:
1. employeeId
2. username
3. email

Runtime behavior:
- if employeeId matches, ISC never checks username or email
- if employeeId fails, ISC tries username
- email is a last resort

This is why ordering is not cosmetic.

---

## 2.5 Designing criteria the right way

Design criteria based on **risk**, not convenience.

### Primary criteria (first)
Should be:
- immutable or near-immutable
- unique per person
- present early in lifecycle

Examples:
- employeeId
- workerId
- global unique ID

### Secondary criteria (fallback)
Should be:
- mostly unique
- stable enough
- understood to be risky

Examples:
- username
- email (only if lifecycle guarantees stability)

### Tertiary criteria (last resort)
Use only if necessary.
These increase risk.

Examples:
- displayName
- combinations derived externally

---

## 2.6 Why weak criteria create silent damage

Weak criteria do not usually fail loudly.
They succeed incorrectly.

This is more dangerous.

Examples:
- two identities share the same email temporarily
- contractor reused an old username
- display names collide

Result:
- wrong account assigned
- governance looks correct
- audit trail is polluted

Correlation success does not mean correctness.

---

## 2.7 Authoritative vs non-authoritative design impact

The same criteria have different consequences.

### Authoritative sources
Bad criteria can:
- create duplicate identities
- overwrite correct identity data
- poison downstream correlation

Design here must be conservative.

### Non-authoritative sources
Bad criteria usually:
- attach accounts incorrectly
- break governance visibility

Still bad, but easier to recover.

---

## 2.8 Changing correlation after aggregation

This is a common trap.

If you change correlation configuration:
- ISC does **not** automatically re-evaluate existing correlated accounts

Why:
- correlation is applied during aggregation
- existing links are preserved unless reprocessed

This explains:
> “I fixed the rule but nothing changed.”

Re-evaluation requires deliberate action later.

---

## 2.9 Criteria count does not equal quality

More rows do not mean better correlation.

Too many criteria:
- increase ambiguity
- increase risk
- hide data quality problems

Strong correlation usually needs:
- 1 or 2 solid identifiers
- clear ordering
- clean data

---

## 2.10 A safe design pattern

A proven pattern:

1. One strong immutable identifier
2. One fallback identifier
3. Stop

If you need more:
- reassess data quality
- reassess identity lifecycle
- do not blindly add rows

---

## 2.11 Reading correlation like a story

When reviewing a config, read it top to bottom and ask:

“If this row matches, am I comfortable assigning ownership forever?”

If the answer is no:
- that row is too risky
- or in the wrong position

---

## 2.12 Real-world example

HR source:
- primary: employeeId

Application source:
- primary: employeeId
- fallback: username

Result:
- identities created cleanly by HR
- applications attach safely
- email never used for ownership

This design avoids most duplication issues.

---

## 2.13 What this part prepares you for

After this part, you should:
- design correlation intentionally
- understand the cost of bad ordering
- recognize risky criteria immediately
- stop treating correlation as trial-and-error

Next, we will trace **exact runtime execution** with logs, states, and outcomes.

---

## Navigation

**⬅ Back:** [Part 1 – Identity Readiness and Correlation Prerequisites](part_1_identity_readiness_and_correlation_prereqs.md)
**➡ Next:** [Part 3 – Correlation Runtime Execution and Data Flow](part_3_correlation_runtime_execution_and_flow.md)

---

*End of Part 2*
