
# Part 3 – Extraction Phase (Reading Data)

[⬅️ Back to Home](../README.md)

---

## The Mental Model (Read This First)

```
Reality → Extraction Gate → ISC Reality
```

Extraction is the gatekeeper of truth.
It decides what data is even allowed to exist inside ISC.

If extraction is wrong, everything downstream works perfectly… on the wrong truth.

So the only real question of extraction is:

**What data gets a chance to exist inside ISC?**

---

## How Extraction Actually Works (The Machine)

Mechanically, extraction is simple:

- A job starts
- Connector asks for data
- Source returns data in pages
- ISC processes each page
- Connector asks for the next page
- Stops when no more pages exist

Sometimes it also says:

> “Give me only what changed since last time.”

So the real engine of extraction is:

- Pagination
- Filters and scope
- Delta memory (tokens, timestamps, watermarks)

Everything you see later is just a result of how well this engine runs.

---

## What Gets Extracted (Three Pipelines)

Extraction does not read one thing. It reads three independent streams.

| Pipeline | Reads | If Broken |
|--------|------|-----------|
| Account Extraction | Users / accounts | No identities possible |
| Entitlement Extraction | Groups / roles / permissions | Access looks empty |
| Membership Extraction | Who belongs to what | Nobody has access |

Each pipeline can succeed or fail on its own.

---

## Running Story

Source contains:

Accounts: Alice, Bob, Carol  
Entitlements: Engineering, HR, Finance  
Membership:
- Alice → Engineering  
- Bob → HR  
- Carol → Finance  

Extraction should do:

- Read 3 accounts  
- Read 3 entitlements  
- Read 3 membership links  

---

## Control Levers

- Page size
- Pagination logic
- Filters and scope
- Delta memory

Small change here = big reality change.

---

## Failure Patterns

### 🔴 Some Users Missing
Pagination or filter

### 🔴 No Entitlements
Entitlement extraction failed

### 🔴 Nobody Has Access
Membership extraction failed

### 🔴 Delta Missed Changes
Broken token or timestamp

---

## Debug Playbook

1) Is object in source?  
2) Did extraction read it?  
3) If not: filter, scope, pagination, delta  
4) Only then blame later phases  

---

## Visual Debug Flow

```
Is object in source?
   ↓
Did extraction read it?
   ↓
If no → filter / scope / pagination / delta
   ↓
If yes → move to next phase
```

---

## Deep Dives

### Pagination
Silent killer when broken.

### Delta
Fast but fragile.

### Filters
Wrong filter = invisible people.

---

## What Must Not Happen

- Completed ≠ complete
- Filters changed casually
- Trusting delta too early
- Ignoring pagination

---

## Safe Fixes

- Counts wrong → full run
- Filters wrong → simplify
- Pagination broken → fix connector
- Delta broken → reset token

---

## Operational Wisdom

First question is always:

**Did extraction even see the object?**

---

## Mastery Check

If you can explain extraction in one sentence, you own it.

---

### Navigation
⬅️ Previous: [Part 2 – Triggers and Job Lifecycle](./Part_2_Triggers_and_Job_Lifecycle.md)  
🏠 Home: [README – Aggregation Master Series](./README.md)  
➡️ Next: [Part 4 – Normalization and Mapping](./Part_4_Normalization_and_Mapping.md)
