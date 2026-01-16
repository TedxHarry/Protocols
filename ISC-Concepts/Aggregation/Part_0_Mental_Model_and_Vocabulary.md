# Part 0 – Mental Model and Vocabulary

[⬅️ Back to Home](../README.md)

---

## Purpose
This part teaches you how to *think* about aggregation before you ever touch buttons, schedules, or connectors.

If you understand this part, everything later will feel logical.
If you skip this part, every problem will feel random.

This is your mental map.

---

## What This Part Does and Does Not Teach

### What this part teaches
- A simple mental model for how data moves through ISC
- The key words you must understand (and what they *really* mean)
- How to debug without panic reruns
- How to locate the exact layer where things went wrong

### What this part does **not** teach
- How to configure connectors
- How to schedule jobs
- How to write rules or transforms
- Tenant-specific UI click paths

This is foundation. Configuration comes later.

---

## Where This Fits in the Master Flow
This part does not change data.
It changes how you understand data moving through the engine.

Master Flow:
Trigger → Extract → Normalize → Persist → Correlate → Evaluate → Recompute → Publish

You don’t need to memorize this yet.
You just need to understand that aggregation is a *chain of steps*, not one action.

---

## The Big Picture Map (Keep This in Your Head)

Here’s the entire aggregation story in one view:

```
Source System
  ↓
ISC stores Accounts (Account Truth)
  ↓
ISC links Accounts to an Identity (Identity Truth)
  ↓
ISC calculates Access (roles, access profiles, policies) (Governance Truth)
  ↓
Indexes refresh
  ↓
UI displays what’s indexed (UI view)
```

If something looks wrong, don’t guess.
Ask: **Which box is wrong?**

---

## Mini‑Glossary (Plain Language)
- Aggregation: Reading data from another system and loading it into ISC so it can be governed
- Source: The system that owns the data (HR, AD, Entra, App, etc.)
- Account: A user record in a source system
- Entitlement: A permission or group in a source
- Identity: The person object inside ISC
- Correlation: Linking an account to an identity
- Authoritative Source: The source that defines who a person is
- Non‑authoritative Source: A source that only adds access, not core identity details

---

## Vocabulary Table (Short, Blunt, Useful)

| Word | What it really means | When you care |
|---|---|---|
| **Aggregation** | Read data from a source and load it into ISC | When accounts or entitlements look missing/outdated |
| **Extract** | Connector pulls raw records | When the source has data but ISC never sees it |
| **Normalize** | Convert raw fields into ISC types and fields | When data looks “present” but wrong format/value |
| **Persist** | Create/update/disable/delete stored objects | When records exist but don’t update correctly |
| **Correlation** | Match an account to a person (identity) | When accounts are uncorrelated or linked to wrong person |
| **Evaluate** | Apply identity profile logic and set identity attributes | When identity fields are stale/wrong |
| **Recompute** | Recalculate access model (roles, access profiles, policies) | When roles/access don’t match identity reality |
| **Publish** | Indexing + eventual UI visibility | When API looks right but UI looks wrong |

---

## Running Example We Will Use Everywhere
We will reuse this same example in every part so nothing feels abstract.

Source: HR System (like Workday)
Users: Alice, Bob, Carol
Accounts: HR_Alice, HR_Bob, HR_Carol
Entitlements: Engineering, HR, Finance

Before aggregation:
- HR has Alice, Bob, Carol
- ISC has no HR accounts yet

After aggregation:
- ISC has HR_Alice, HR_Bob, HR_Carol as accounts
- They may or may not be linked to identities yet

---

## What Aggregation Really Is
Aggregation is not magic.
Aggregation is not “perfect sync.”

Aggregation is a pipeline that moves data through layers:

Source truth → Account truth → Identity truth → Governance truth

Each layer can disagree with the one before it.
That is normal.

Think of it like food delivery:
- Restaurant cooks the food = Source truth
- Delivery driver carries it = Account truth
- You receive and check it = Identity truth
- You eat and judge it = Governance truth

If food tastes wrong, you don’t blame the wrong step.
You ask: **where did it go wrong?**

---

## Wrong vs Right Thinking (Read This Twice)

### Aggregation
Wrong thinking:
Aggregation syncs everything perfectly.

Right thinking:
Aggregation moves data through layers that can disagree.

### Completed
Wrong thinking:
Job says Completed, so the data is correct.

Right thinking:
Completed only means the engine finished running. It says nothing about correctness.

### UI vs API
Wrong thinking:
UI is the truth.

Right thinking:
UI is a delayed view of indexed data. API is closer to stored reality.

### Account vs Identity
Wrong thinking:
An account is the same thing as the person.

Right thinking:
An account is just a record in a source. Identity is ISC’s person object that may link to many accounts.

---

## The Four Layers of Truth
Always ask: **which layer is wrong?**

### 1) Source Truth
What actually exists in the source system.
If it is wrong here, nothing later can fix it.

### 2) Account Truth
What ISC stored after reading and transforming the data.
Source can be right, but ISC can store it wrong.

### 3) Identity Truth
What ISC believes about a person after linking accounts and running profile logic.
Accounts can be right, but identity can still be wrong.

### 4) Governance Truth
What access, roles, and certifications are built from the identity.
Identity can be right, but access can still be wrong.

Most outages happen because people assume all layers are the same.
They are not.

---

## The Engine in Simple Words
Every aggregation run follows this thinking path:

1. Trigger
Something says “start.” Manual, schedule, or API.

2. Extract
Connector reads accounts and access from the source.

3. Normalize
Raw data is converted into ISC format and types.

4. Persist
ISC decides to create, update, disable, or delete.

5. Correlate
Accounts are linked (or not) to identities.

6. Evaluate
Identity profile logic runs and sets identity fields.

7. Recompute
Access model recalculates roles and access profiles.

8. Publish
Indexes update and UI eventually shows the result.

This is not one action.
It is a chain of decisions.

---

## Learning Checkpoint 1
Stop and think:

If the UI looks wrong but the API looks right, which step is most likely involved?

Hint: it’s the step that makes UI visibility happen.

---

## Running Example Through the Engine (Happy Path)
Alice in HR has:
- employeeId = 1001
- department = Engineering

After aggregation:

Extract:
- Connector reads HR_Alice

Normalize:
- employeeId becomes accountAttribute.employeeId
- department becomes accountAttribute.department

Persist:
- HR_Alice is created or updated in ISC

Correlate:
- employeeId = 1001 matches Identity Alice
- HR_Alice links to Identity Alice

Evaluate:
- Identity Alice.department = Engineering

Recompute:
- Engineering role is assigned

Publish:
- UI shows Alice has Engineering role

If any step breaks, the final result lies to you.

---

## The Same Example When Things Break (Broken Flow Walkthrough)
Scenario: Alice is in Engineering in HR, but **ISC UI shows no Engineering role**.

Don’t rerun blindly. Trace the layers.

### Step A: Source Truth
Question: Is Alice actually Engineering in HR?
- If no, fix HR. Everything else is downstream noise.

### Step B: Account Truth
Question: Did ISC store Alice’s department correctly on the HR account?
- If the account in ISC shows department missing or wrong, the issue is Extract, Normalize, or Persist.

### Step C: Identity Truth
Question: Is the HR account correlated to the right identity?
- If the account is uncorrelated, identity logic cannot apply.
- If it’s correlated to the wrong identity, the “right” person won’t get access.

### Step D: Evaluate
Question: Did identity profile logic set identity.department correctly?
- If account is right but identity.department is wrong, the issue is identity profile logic, mapping, or evaluation rules.

### Step E: Governance Truth
Question: Did access recompute run and assign roles correctly?
- If identity.department is correct but role is missing, the issue is recompute triggers or access modeling.

### Step F: Publish
Question: Is API correct but UI wrong?
- That usually means indexing or publish delay.

This is why reruns don’t fix most issues.
They just replay the same broken logic.

---

## Real-World Failure Stories (So You Remember)

### Story 1: The Panic Rerun Loop
UI showed wrong department.
Team reran full aggregation multiple times.
Root cause was identity profile logic using the wrong attribute name.
Reruns changed nothing, but created noise and confusion.

### Story 2: The Correlation Trap
Accounts were present and correct.
But they were uncorrelated due to a missing identifier.
Identity never updated, so access never recalculated.
Problem looked like “roles are broken,” but it was correlation.

### Story 3: The UI Mirage
API showed correct role assignments.
UI showed missing roles for hours.
Root cause was index delay.
Teams almost rolled back changes that were actually correct.

---

## Why This Matters
If you don’t separate layers, you will:
- Fix the wrong thing
- Create bigger outages
- Panic rerun jobs blindly

Example mistake:
UI shows wrong department → You rerun full aggregation
Real issue was identity profile logic
Now delta breaks and noise increases

Understanding layers prevents panic fixing.

---

## If You See This → Do This
- UI wrong → Check API
- API wrong → Check mapping and persist logic
- Accounts uncorrelated → Check correlation rules
- Identity fields stale → Check evaluation logic
- Access wrong → Check recompute triggers

Never start with rerun.
Start with locating the broken layer.

---

## The Debugging Playbook (Print This in Your Brain)

When something looks wrong, follow this order:

1) Prove the source truth
- Confirm the value exists in the source

2) Prove account truth in ISC
- Confirm the account exists and attributes are correct

3) Prove correlation
- Confirm account is linked to the right identity

4) Prove identity truth
- Confirm identity attributes reflect the account truth

5) Prove governance truth
- Confirm access model recalculated and produced expected roles/access

6) Prove publish
- Confirm indexing/UI visibility is caught up

Only after this, consider rerunning.

---

## How Beginners Usually Fail
- Treat aggregation as magic
- Trust “Completed” as “Correct”
- Skip proof and go straight to rerun
- Think account = identity
- Think UI = truth

These habits cause long outages.

---

## Proof Paths You Will Use Everywhere
- UI shows symptoms
- API shows stored state
- Logs show what really happened

You will use all three in every part.

---

## What Must Not Happen
- No blind reruns
- No delta reset without reason
- No mass uncorrelate without rollback

---

## Safe Habits to Build Now
- Always ask “which layer is wrong?”
- Always prove before fixing
- Always think in steps, not screens

---

## Learning Checkpoint 2
Stop and think:

If the account attribute is correct in ISC, but the identity attribute is stale, which step is likely the issue?

Hint: it’s the step that applies identity profile logic.

---

## Confidence Check
If you can answer these, you got it:
1. What are the four layers of truth?
2. Which step links accounts to identities?
3. Why can API be right but UI be wrong?
4. Why does “Completed” not mean “Correct”?
5. When should you rerun, and when should you not?

---

## If You Remember Only One Line
Never ask:
Which job should I rerun?

Always ask:
**Which layer is lying to me?**

---

### Navigation
⬅️ Previous: None – This Is the Beginning
🏠 Home: [README – Aggregation Master Series](../README.md)
➡️ Next: [Part 1 – Before Aggregation Can Run (Pre‑Flight)](../part_1_before_aggregation_can_run.md)
