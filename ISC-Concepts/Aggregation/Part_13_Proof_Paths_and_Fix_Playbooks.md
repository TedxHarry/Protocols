# Part 13 – Proof Paths and Fix Playbooks — Teaching Mastery Edition

[⬅️ Back to Home](../README.md)

---

## The One Question This Part Answers

**How do I turn suspicion into proof, and proof into a safe fix?**

Troubleshooting finds where truth broke.  
Proof paths prove it.  
Fix playbooks change it safely.

Keep this line in mind:

**Never fix what you cannot prove.**

---

## Where This Fits in the Engine

Trigger → Extract → Normalize → Persist → Correlate → Evaluate → Recompute → Publish

Proof paths run across the whole engine.  
Playbooks come after proof, never before.

---

## Mental Model

```
Symptom
   → Proof path (evidence)
     → First broken truth
       → Fix playbook (safe change)
         → Prove again
```

No guessing. No hope. Evidence only.

---

## Proof Path: What It Really Is

A proof path is the story of evidence from reality to UI.

Default rule:

- Reality is strongest  
- API is stronger than UI  
- Logs explain how it happened  

UI shows feelings.  
API shows memory.  
Logs show history.

---

## Guided Story

HR says Alice is Engineering.  
ISC identity says Alice is Sales.  
Roles still give Sales access.

Three statements cannot all be true.

Proof path will decide who is lying.

---

## Proof Path 1: Source Truth

Question:
What does the real world say?

Best proof:
- Source UI
- Source API
- Business or HR record

If source is wrong:
Stop. Everything else is innocent.

---

## Proof Path 2: Extraction

Question:
Did ISC even see Alice?

Best proof:
- Connector logs
- Filter and pagination logs

If Alice never appears in logs:
She never entered the engine.

Nothing after this matters.

---

## Proof Path 3: Normalization and Mapping

Question:
Did ISC shape Alice correctly?

Best proof:
- Account object via API

If source is right but account fields are wrong:
The lie was born here.

---

## Proof Path 4: Persistence

Question:
What did ISC decide about this record?

Best proof:
- Account metadata:
  - Unique ID
  - Created vs updated
  - Last seen
  - Missing-from-feed

If memory is wrong:
Everything after it is poisoned.

---

## Proof Path 5: Correlation

Question:
Does this account belong to the right human?

Best proof:
- Account → identity link in API

If wrong:
Stop everything. Fix this first.

Wrong correlation is the most dangerous lie.

---

## Proof Path 6: Identity Evaluation

Question:
Why does identity show this value?

Best proof:
- Identity API with source attribution
- Identity profile rules

If account is right but identity is wrong:
The rulebook is lying.

---

## Proof Path 7: Recompute

Question:
Given this identity, is access correct?

Best proof:
- Access assignments in API
- Recompute job history

If identity is right but access is wrong:
Judgment failed, not aggregation.

---

## Proof Path 8: UI

Question:
Is UI just late?

Best proof:
- Compare UI vs API

If API is right and UI is wrong:
Indexing or cache is lying, not data.

---

## Walking the Full Proof Path

Always walk in this order:

Reality → Extraction → Mapping → Persistence → Correlation → Identity → Access → UI

At each step ask:

Is this still true?

The first “no” is where truth broke.

---

## What a Fix Playbook Is

A fix playbook is memory, not creativity.

It says:
“When this pattern appears, this fix is safe.”

Playbooks exist so you don’t invent danger every time.

---

## Playbook A – Missing Accounts

Symptom:
User exists in source, not in ISC.

Proof:
Source right → Extraction wrong.

Fix:
- Check filters
- Check pagination
- Run one controlled full
- Validate

Never:
Touch identity or correlation first.

---

## Playbook B – Account Fields Wrong

Symptom:
Account exists but values empty or weird.

Proof:
Extraction right → Mapping wrong.

Fix:
- Preview mapping
- Fix type mismatch
- Fix transforms
- Rerun small scope

Never:
Change identity profile first.

---

## Playbook C – Identity Field Wrong

Symptom:
Account correct, identity wrong.

Proof:
Mapping right → Evaluation wrong.

Fix:
- Check precedence
- Check derived logic
- Re-evaluate identity

Never:
Rerun aggregation blindly.

---

## Playbook D – Access Wrong

Symptom:
Identity right, access wrong.

Proof:
Evaluation right → Recompute wrong.

Fix:
- Check recompute status
- Fix rule logic
- Rerun recompute

Never:
Change correlation.

---

## Playbook E – Delta Skipped Changes

Symptom:
Source changed, ISC never updates.

Proof:
Source right → Delta memory wrong.

Fix:
- Inspect token
- Controlled reset once
- Validate
- Re-enable delta

Never:
Reset repeatedly.

---

## Safety Rules

Before any fix, ask:

- How many people does this touch?
- Can I test with one?
- Can I undo it?

High-risk moves:
- Mass uncorrelation
- Delete accounts
- Reset delta on large sources

Treat these like production surgery.

---

## After Fix: Prove Again

Never trust your own fix.

Walk again:

Reality → Account → Identity → Access → UI

If you don’t re-prove, you don’t know.

---

## Illusions This Phase Creates

- Fixing fast is better than fixing right  
- UI success means real success  
- More changes means more progress  

All can be false.

---

## Traps That Fool Smart People

- Fixing before proving  
- Touching many things at once  
- Trusting UI over API  
- Ignoring rollback paths  

Senior mistakes are usually rushed mistakes.

---

## What This Phase Does NOT Do

- It does not create data  
- It does not decide identity  
- It does not grant access  

It only proves truth and changes it safely.

---

## Debug Mindset

When something is wrong:

1) Pick one person  
2) Walk proof path  
3) Find first lie  
4) Apply matching playbook  
5) Prove again  

---

## Visual Proof-to-Fix Flow

```
Symptom
   ↓
Walk proof path
   ↓
Find first lie
   ↓
Choose playbook
   ↓
Fix safely
   ↓
Prove again
```

---

## The One Sentence That Defines Mastery

Before you fix anything, ask:

**What is my proof, and how can I undo this?**

---

## Mastery Check

Answer without notes:

- What is a proof path?  
- Why is API stronger than UI?  
- Why is correlation the most dangerous lie?  
- When do you reset delta?  
- Why must fixes be reversible?  

---

### Navigation
⬅️ Previous: [Part 12 – Troubleshooting](./Part_12_Troubleshooting_Playbook.md)  
🏠 Home: [README](./README.md)  
➡️ Next: [Part 14 – Operational Scheduling](./Part_14_Operational_Scheduling_and_Avoiding_Overlap.md)

