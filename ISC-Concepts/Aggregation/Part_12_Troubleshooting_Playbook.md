# Part 12 – Troubleshooting Playbook — Teaching Mastery Edition

[⬅️ Back to Home](../README.md)

---

## The One Question This Part Answers

**How do I think when reality and expectation disagree?**

Troubleshooting is not fixing.  
Troubleshooting is learning the truth before you touch anything.

Keep this line in mind:

**Action without understanding creates new lies.**

---

## Where This Fits in the Engine

Trigger → Extract → Normalize → Persist → Correlate → Evaluate → Recompute → Publish

Troubleshooting is not a step in the engine.  
It is how you walk the engine when something feels wrong.

---

## The Mental Model

```
Expectation
   vs
Reality
   → Find which layer is lying
     → Prove it
       → Fix only that
```

If you can’t name the lying layer, you are guessing.

---

## The Four Layers of Truth

Every problem lives in one of four layers:

1) Source truth – What the real world says  
2) Account truth – What ISC stored from sources  
3) Identity truth – What ISC believes about people  
4) Governance truth – What access and rules say  

Most chaos happens because people mix these layers.

---

## A Simple Story

HR says Alice is Engineering.  
ISC identity says Alice is Sales.  
Roles still give Sales access.

Which story is false?

- HR might be wrong  
- Account might be wrong  
- Identity judgment might be wrong  
- Recompute might be late  

Troubleshooting is finding which story is lying.

---

## The Golden Rule

Never change anything until you can answer:

**Which layer is wrong?**

Changing before knowing this just creates new lies.

---

## Start With One Person

Never start with “everyone.”

Pick one person who clearly shows the problem.

Why?

If you cannot explain one person end to end,  
you will never understand ten thousand.

---

## Step 1 – Prove Source Truth

Ask: What does reality say?

Check:
- Source UI
- Source API
- Business or HR record

Write it down.

If source is wrong, stop.  
Everything else is innocent.

---

## Step 2 – Prove Account Truth

Ask: Did ISC read and store this correctly?

Check the account object:
- Is the person present?
- Are fields correct?
- Are values missing?

If account is wrong, the lie started in extraction or mapping.

Identity has no chance if account is wrong.

---

## Step 3 – Prove Persistence Behavior

Ask: What did ISC do with this account?

Check:
- Unique ID match
- Created vs updated
- Disabled or missing-from-feed

If memory is wrong, everything after it is poisoned.

---

## Step 4 – Prove Correlation

Ask: Is this account linked to the right identity?

If linked to wrong identity:
Stop. Fix this first.

Wrong correlation means:
One human may receive another human’s access.

This is the most dangerous failure.

---

## Step 5 – Prove Identity Judgment

Ask: Why does identity show this value?

Check:
- Which sources feed this field
- Precedence order
- Derived logic
- Lifecycle influence

If account is right but identity is wrong, the rulebook is lying.

---

## Step 6 – Prove Access Judgment

Ask: Given this identity, is access correct?

If identity is right but roles are wrong:
- Recompute may be late
- Rules may not match identity
- Access profile logic may be wrong

Do not rerun aggregation to fix access logic.

---

## Step 7 – Prove UI vs Reality

If API is right but UI is wrong:
- Indexing
- Cache
- Lag

Do not touch aggregation for a UI illusion.

---

## Walking the Engine

Always walk in this order:

Reality → Account → Memory → Correlation → Identity → Access → UI

At each step ask:

Is this still true?

The first “no” is where truth broke.

---

## Interactive Pause

Scenario:

Alice changed department in HR.  
ISC still shows old department.

Question:
What do you check first?

Pause. Think.

Answer:
Did extraction even read Alice’s change?

If she never entered the engine, nothing else matters.

---

## Failure Story

Team saw wrong department.

They reran aggregation three times.

Still wrong.

Finally checked:
Filter excluded Alice. She was never read.

Reruns only repeated the same lie.

They fixed without understanding.

---

## Why Reruns Are Dangerous

Reruns feel safe.  
But reruns:

- Hide root cause
- Create noise
- Change memory repeatedly
- Make history confusing

Rerun only after you know what you are fixing.

---

## The Smallest Safe Fix

Before touching anything, ask:

- How many people does this affect?
- Can I test with one person?
- Can I roll back?

Prefer fixes that are:
- Small
- Reversible
- Testable

Be careful with:
- Deleting accounts
- Mass uncorrelation
- Resetting delta

These change memory, not just behavior.

---

## After Fix: Prove Again

After any change, walk again:

Reality → Account → Identity → Access → UI

If you don’t re-prove, you don’t know.

---

## Illusions This Phase Creates

- Rerunning is fixing  
- Green UI means safe  
- More action means more progress  

All can be false.

---

## Traps That Fool Smart People

- Debugging access before identity  
- Rerunning before proving  
- Trusting UI over API  
- Changing many things at once  

These are senior-level mistakes.

---

## What This Phase Does NOT Do

- It does not change data  
- It does not fix logic  
- It does not rerun jobs  

It only teaches you where truth broke.

---

## Debug Mindset

When something feels wrong:

1) Pick one person  
2) Prove reality  
3) Walk the engine  
4) Find first lie  
5) Fix only that  
6) Prove again  

---

## Visual Debug Flow

```
Pick one person
   ↓
Prove source truth
   ↓
Prove account truth
   ↓
Prove memory
   ↓
Prove correlation
   ↓
Prove identity
   ↓
Prove access
   ↓
Prove UI
```

---

## The One Sentence That Defines Mastery

Before you touch anything, ask:

**Which layer is lying to me?**

---

## Mastery Check

Answer without notes:

- What are the four layers of truth?  
- Why start with one person?  
- Why is wrong correlation the most dangerous?  
- Why is UI always last?  
- Why are blind reruns harmful?  

---
### Navigation
⬅️ Previous: [Part 11 – Verification](./Part_11_Verification_and_Validation.md)  
🏠 Home: [README](./README.md)  
➡️ Next: [Part 13 – Proof Paths](./Part_13_Proof_Paths_and_Fix_Playbooks.md)

