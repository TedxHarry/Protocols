# Part 5 – Write and Persistence (Teaching Mastery Edition)

[⬅️ Back to Home](../README.md)

---

## Why This Part Exists

Part 4 shaped data so ISC could understand it.  
Part 5 decides something more permanent:

**Should ISC remember this as truth?**

Before this step, data is temporary.  
After this step, data becomes memory.

Once ISC remembers something, every future run will argue with that memory.

Keep this sentence in mind:

**Persistence is not storage. It is belief.**

---

## Where This Fits in the End-to-End Flow

Trigger → Extract → Normalize → **Persist** → Correlate → Evaluate → Recompute → Publish

Extraction copies reality.  
Normalization shapes it.  
Persistence decides:

“Have I seen this before, or is this new?”

Everything later depends on that answer.

---

## The Mental Model (Use This Forever)

```
Incoming shaped data
   → Compare with memory (unique ID)
     → Create / Update / Missing-from-feed
       → Memory is updated
```

Persistence is a memory engine.  
It does not judge correctness.  
It only judges: **Have I seen you before?**

---

## What ISC Is Really Doing Here

When data reaches persistence, ISC is not asking:

- Is this data right?
- Is this person valid?
- Should they have access?

It is asking only:

**Do I already know this thing?**

It checks one key: the **Unique ID**.

Then it decides:

- New Unique ID → Create  
- Known Unique ID → Update  
- Previously known but not seen now → Missing-from-feed  

This is memory logic, not business logic.

---

## Unique ID: The Memory Key

Unique ID is how ISC remembers.

If it stays stable, ISC remembers people across time.  
If it changes, ISC forgets and starts over.

### Guided Example

Run 1:
Alice has employeeId = 1001  
ISC stores account with key = 1001

Run 2:
Alice still has employeeId = 1001  
ISC says: I remember you → Update

Run 3:
Unique ID changed to email  
Alice’s email changed last week

ISC now sees a new key.  
It says: I don’t remember you → Create

Old Alice still exists.  
New Alice is born.  
Duplicates appear.

This is not a bug.  
This is memory being reset.

---

## Mini Checkpoint

Answer without looking:

- What does Unique ID really represent?  
- What happens if it changes?  

If your answer is “memory,” you’re on track.

---

## A Slow Walk Through One Person

Source: HR  
Person: Bob  
Unique ID: employeeId

### First Run
HR sends: employeeId = 2002  
ISC has never seen 2002 → Create Bob

### Second Run
HR sends: employeeId = 2002, department changed  
ISC recognizes 2002 → Update Bob

### Third Run
HR does not send Bob at all  

ISC does not know why.  
It only knows: Bob was not seen this time.

So Bob becomes: **Missing-from-feed**

---

## Missing-from-Feed: Uncertainty, Not Truth

Missing-from-feed does NOT mean:
“Bob left.”

It means:
“I did not see Bob in this run.”

Possible reasons:
- Bob really left
- Filter excluded Bob
- Pagination broke
- Delta skipped Bob
- Connector lost permission

Persistence now asks:

“What should I do when I am unsure?”

Your answer becomes policy.

---

## Disable vs Delete: Two Worldviews

### Disable says:
“I don’t see you now, but I remember you.”  
History stays. Audits stay. Context stays.

### Delete says:
“I will forget you completely.”  
History may vanish. Audits lose meaning.

Most enterprises choose disable because memory is safer than amnesia.

---

## Interactive Scenario

Source behavior:

Run 1: Alice, Bob  
Run 2: Alice only  

Policy: Disable when missing

Question: What does ISC believe after Run 2?

Pause. Think.

Answer:
- Alice → active and updated  
- Bob → exists but disabled  

ISC now believes Bob is real, but inactive.  
All later logic will trust that belief.

---

## State Fields: The System’s Diary

Every account carries memory:

- When it was created  
- When it was last seen  
- When it was last changed  
- Whether it is disabled  

These fields tell the story of what ISC believed at each run.

When something looks wrong, read the diary before guessing.

---

## Why This Phase Is Dangerous

Mistakes here don’t look small.

They look like:
- Duplicates everywhere  
- Leavers still active  
- People disappearing  
- History becoming confusing  

Because once memory is wrong, every future run argues with a lie.

---

## Classic Failure Story

Company changed unique ID from employeeId to email.

Many people had changed emails.

Next run:
ISC forgot everyone.  
Created everyone again.

Old people still existed.  
New people were born.

Security team panicked.  
Persistence just did what it was told.

Memory was erased. Then rebuilt wrongly.

---

## Debug Playbook (The Order Masters Use)

When things look wrong, ask in this order:

1) What is the Unique ID?  
2) Did it change?  
3) What does missing-from-feed do?  
4) What do state fields say?  

Only after that, check correlation or access logic.

---

## Visual Debug Flow

```
What is the Unique ID?
   ↓
Did it change recently?
   ↓
Is object missing-from-feed?
   ↓
What did policy do (disable/delete/ignore)?
   ↓
Check state fields for history
```

---

## Proof Paths

Use more than one view:

- Account record (created, last seen, disabled flags)  
- Job history  
- API or UI state fields  

Truth lives in memory fields, not assumptions.

---

## Safe Fixes

- Unique ID wrong → fix and plan cleanup carefully  
- Too many missing → check extraction first  
- Disable/delete wrong → change policy and retest  
- Duplicates exist → fix key, then reconcile  

---

## What Must Not Happen

- Do not change Unique ID casually  
- Do not delete unless legally required  
- Do not ignore missing-from-feed patterns  
- Do not debug access before memory  

---

## The One Sentence That Defines Mastery

Before you ask “Why is access wrong?”, ask:

**What does ISC think is real?**

---

## Mastery Check

Answer these without notes:

- What question is persistence really answering?  
- Why is Unique ID actually memory?  
- Why is missing-from-feed uncertainty?  
- Why is disable safer than delete?  
- Why do most disasters start here?  

---

### Navigation
⬅️ Previous: [Part 4 – Normalization and Mapping](./Part_4_Normalization_and_Mapping.md)  
🏠 Home: [README – Aggregation Master Series](./README.md)  
➡️ Next: [Part 6 – Correlation (Accounts → Identities)](./Part_6_Correlation_Accounts_to_Identities.md)
