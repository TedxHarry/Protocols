
# Part 12 – Troubleshooting Playbook (Story-Driven)

[⬅️ Back to Home](../README.md)

---

## Big Idea

Troubleshooting is not fixing.  
Troubleshooting is understanding.

Most teams rush to “do something.”  
Rerun jobs. Change configs. Reset delta.

That feels active, but it often makes the story messier.

This playbook teaches one habit:

First learn what the system believes.  
Only then decide what to change.

---

## Where This Fits

Trigger → Extract → Normalize → Persist → Correlate → Evaluate → Recompute → Publish

Troubleshooting is not a step in the engine.  
It is how you walk through the engine when reality and expectation disagree.

---

## Four Layers of Truth

Every problem lives in one of four layers:

1) Source truth – What the real world says  
2) Account truth – What ISC stored from sources  
3) Identity truth – What ISC believes about people  
4) Governance truth – What access and rules say  

If you can’t name the broken layer, you are not troubleshooting. You are guessing.

---

## A Simple Story

HR says Alice is Engineering.  
ISC identity says Alice is Sales.  
Roles still give Sales access.

Where is the lie?

- HR might be wrong  
- Account might be wrong  
- Identity might be choosing wrong  
- Recompute might be late  

Troubleshooting is finding which story is false.

---

## The Golden Rule

Never change anything until you can answer:

Which layer is wrong?

Changing before you know this just creates new lies.

---

## How to Start: One Person

Never start with “everyone.”

Pick one person who shows the problem clearly.

Why?

If you cannot explain one person end to end,  
you will never understand ten thousand.

---

## Step 1: Prove Source Truth

Ask: What does reality say?

Check:
- HR screen
- Source UI
- Source API

Write it down.

If source is wrong, stop.  
Everything else is innocent.

---

## Step 2: Prove Account Truth

Ask: Did ISC read and store this correctly?

Check the account object:
- Is the person present?
- Are fields correct?
- Are values missing?

If account is wrong, the lie started in extraction or mapping.

Identity has no chance to be right if the account is wrong.

---

## Step 3: Prove Persistence Belief

Ask: What did ISC decide about this account?

Check:
- Unique ID match
- Created vs updated
- Disabled or missing-from-feed

If ISC is matching the wrong record, everything after this is poisoned.

---

## Step 4: Prove Correlation

Ask: Is this account linked to the right person?

If linked to wrong identity:
Stop. Fix this first.

Wrong correlation means:
One human may receive another human’s access.

This is the most dangerous failure.

---

## Step 5: Prove Identity Judgment

Ask: Why does identity show this value?

Check:
- Which sources feed this field
- Precedence order
- Derived logic
- Lifecycle influence

If account is right but identity is wrong, the rulebook is lying.

---

## Step 6: Prove Access Judgment

Ask: Given this identity, is access correct?

If identity is right but roles are wrong:
- Recompute may be late
- Role rules may not match identity
- Access profile logic may be wrong

Do not rerun aggregation to fix access logic.

---

## Step 7: Prove UI vs Reality

If API is right but UI is wrong:
- Indexing
- Cache
- Lag

Do not touch aggregation for a UI illusion.

---

## Walking the Engine

When something is wrong, walk like this every time:

Reality → Account → Persistence → Correlation → Identity → Access → UI

At each step, ask:

Is this still true?

The first “no” is where truth broke.

---

## Failure Story

Team saw wrong department on identity.

They reran aggregation three times.

Still wrong.

Finally checked:
Extraction never saw Alice because filter excluded her.

Reruns only repeated the same lie.

They were fixing without understanding.

---

## Why Reruns Are Dangerous

Reruns feel safe.  
But reruns:

- Hide root cause
- Create noise
- Change state repeatedly
- Make history confusing

Rerun only after you know what you are fixing.

---

## Smallest Safe Fix

Before touching anything, ask:

- How many people does this affect?
- Can I test with one person?
- Can I roll back?

Prefer fixes that are:
- Reversible
- Small
- Testable

Be careful with:
- Deleting accounts
- Mass uncorrelation
- Resetting delta

These change memory, not just behavior.

---

## After Fix: Prove Again

After you change something:

Walk the same path again:

Reality → Account → Identity → Access → UI

If you don’t re-prove, you don’t know.

---

## Mental Model

Troubleshooting is not action.  
It is investigation.

Fixing is easy.  
Understanding is hard.

But only understanding makes fixes safe.

---

## You Understand This If You Can Answer

- What are the four layers of truth?  
- Why start with one person?  
- Why is wrong correlation the most dangerous?  
- Why is UI always last?  
- Why are blind reruns harmful?  

---

### Navigation
⬅️ Previous: [Part 11 – Verification and Validation](../Part_11_Verification_and_Validation_Story.md)  
🏠 Home: [README – Aggregation Master Series](../README.md)  
➡️ Next: [Part 13 – Proof Paths and Fix Playbooks](../part_13_proof_paths_and_fix_playbooks.md)
