# Part 0 – Mental Model and Vocabulary (Teaching Mastery Edition)

[⬅️ Back to Home](README_Aggregation_Master_Series_FINAL_v3.md)

---

## Why This Part Exists

Before you touch buttons, schedules, or connectors, you need the right way to **think**.

Aggregation is not a button.  
It is not a sync.  
It is a chain of decisions.

If you learn the chain first, everything later feels logical.  
If you skip this, every bug will feel random.

This part gives you the map.

---

## The Big Journey (One Line First)

```
Source Truth → Account Truth → Identity Truth → Governance Truth → UI View
```

Aggregation is not one truth.  
It is a journey of truth through layers.

When something looks wrong, never guess.  
Always ask:

**Which layer is lying to me?**

---

## Where This Fits in the Engine

Master Flow:

Trigger → Extract → Normalize → Persist → Correlate → Evaluate → Recompute → Publish

You don’t need to memorize this yet.  
Just know this:

Aggregation is not one step.  
It is a chain.

If one link lies, everything after it lies politely.

---

## The Four Layers of Truth

### 1) Source Truth
What actually exists in the source system.

If it is wrong here, nothing later can fix it.

### 2) Account Truth
What ISC stored after reading and shaping source data.

Source can be right.  
ISC can still store it wrong.

### 3) Identity Truth
What ISC believes about a person after linking accounts and applying rules.

Accounts can be right.  
Identity can still be wrong.

### 4) Governance Truth
What access, roles, and certifications are built from identity.

Identity can be right.  
Access can still be wrong.

Most outages happen because people assume all four are the same.  
They are not.

---

## The Engine in Simple Words

Every aggregation run thinks like this:

1) Something triggers the run  
2) Data is read from the source  
3) Data is cleaned and shaped  
4) Data is written into memory  
5) Accounts are linked to people  
6) Identity rules decide who the person is  
7) Access is calculated  
8) UI eventually shows it  

This is not one action.  
It is a story in eight chapters.

---

## Running Example We Will Use Everywhere

We will reuse the same example in every part.

Source: HR System  
People: Alice, Bob, Carol  
Accounts: HR_Alice, HR_Bob, HR_Carol  
Entitlements: Engineering, HR, Finance  

Before aggregation:
- HR has Alice, Bob, Carol
- ISC has nothing yet

After aggregation:
- ISC has HR_Alice, HR_Bob, HR_Carol as accounts
- They may or may not be linked to identities yet

This example will grow with you.

---

## What Aggregation Really Is

Aggregation is not magic.  
Aggregation is not “perfect sync.”

Aggregation is a pipeline:

```
Source Truth → Account Truth → Identity Truth → Governance Truth
```

Each layer can disagree with the one before it.  
That is normal.

Think like food delivery:

- Restaurant cooks = Source truth  
- Driver carries = Account truth  
- You receive = Identity truth  
- You eat and judge = Governance truth  

If food tastes wrong, you don’t blame randomly.  
You ask: where did it go wrong?

---

## Vocabulary (Taught, Not Memorized)

### Aggregation
Reading data from another system and loading it into ISC so it can be governed.

### Source
The system that owns the data (HR, AD, Entra, App, etc.).

### Account
A user record in a source system.

### Entitlement
A permission or group in a source.

### Identity
The person object inside ISC.

### Correlation
Linking an account to an identity.

### Authoritative Source
The source that defines who a person is.

### Non‑Authoritative Source
A source that only adds access, not identity.

---

## Vocabulary Table (Short and Practical)

| Word | What it really means | When you care |
|---|---|---|
| Aggregation | Read data from a source | When data is missing or stale |
| Extract | Pull raw records | When ISC never sees source data |
| Normalize | Fix format and type | When data exists but looks wrong |
| Persist | Save as memory | When data doesn’t update correctly |
| Correlate | Link to person | When accounts are uncorrelated |
| Evaluate | Build identity | When identity fields are wrong |
| Recompute | Build access | When roles/access are wrong |
| Publish | Show in UI | When API is right but UI is wrong |

---

## Wrong vs Right Thinking

### Aggregation
Wrong: Aggregation syncs everything perfectly.  
Right: Aggregation moves truth through layers that can disagree.

### Completed
Wrong: Job says Completed, so data is correct.  
Right: Completed only means the engine stopped running.

### UI vs API
Wrong: UI is truth.  
Right: UI is delayed. API is closer to memory.

### Account vs Identity
Wrong: Account equals person.  
Right: Account is a record. Identity is the person model.

---

## Walk the Example Through the Engine

Alice in HR:
- employeeId = 1001
- department = Engineering

### Extract
Connector reads HR_Alice.

### Normalize
Fields become ISC-style fields.

### Persist
HR_Alice is stored.

### Correlate
employeeId = 1001 links to Identity Alice.

### Evaluate
Identity Alice.department = Engineering.

### Recompute
Engineering role assigned.

### Publish
UI eventually shows Alice has Engineering role.

If one step lies, the final story lies.

---

## When Things Break

Scenario:
Alice is Engineering in HR.  
UI shows no Engineering role.

Trace layers:

1) Source: Is Alice Engineering in HR?  
2) Account: Does ISC show department correctly?  
3) Correlation: Is HR_Alice linked to Alice?  
4) Identity: Does identity.department = Engineering?  
5) Governance: Was role recalculated?  
6) Publish: Is UI just delayed?  

Never rerun first.  
Locate the lying layer first.

---

## Debug Playbook

When something looks wrong:

1) Prove source truth  
2) Prove account truth  
3) Prove correlation  
4) Prove identity truth  
5) Prove governance truth  
6) Prove publish  

Only after this, consider rerunning.

---

## Mini Checkpoint

If API looks right but UI looks wrong, which step is likely involved?  
Answer: Publish / indexing.

---

## Real Failure Lessons

- Reruns don’t fix logic.  
- UI can lie temporarily.  
- Correlation mistakes look like access bugs.  
- Identity logic mistakes look like source bugs.  

Most chaos comes from not separating layers.

---

## If You Remember One Line

Never ask:  
“Which job should I rerun?”

Always ask:  
**Which layer is lying to me?**

---
### Navigation
⬅️ Previous: None – This Is the Beginning
🏠 Home: [README – Aggregation Master Series](README_Aggregation_Master_Series_FINAL_v3.md)
➡️ Next: [Part 1 – Before Aggregation Can Run (Pre‑Flight)](Part_1_Before_Aggregation_Can_Run_Pre_Flight.md)
