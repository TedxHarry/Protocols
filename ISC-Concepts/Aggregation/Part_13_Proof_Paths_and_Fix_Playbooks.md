
# Part 13 – Proof Paths and Fix Playbooks (Story-Driven)

[⬅️ Back to Home](../README.md)

---

## Big Idea

Troubleshooting finds where truth broke.  
Proof paths and fix playbooks decide how you *prove* it and how you *fix* it safely.

This part turns:
“I think this is the problem”  
into  
“I can prove this is the problem.”

And it turns:
“Let’s try something”  
into  
“This fix is known, safe, and reversible.”

---

## Where This Lives

Trigger → Extract → Normalize → Persist → Correlate → Evaluate → Recompute → Publish

Proof paths run across the whole engine.  
They are how you walk the engine with evidence, not guesses.

Fix playbooks come after proof.  
Never before.

---

## What a Proof Path Really Is

A proof path is the story of evidence.

Not opinions.  
Not UI feelings.  
Not guesses.

A proof path answers:
“What is the strongest evidence for truth at this layer?”

Your default mindset:

UI shows the symptom.  
API shows the stored truth.  
Logs explain how it happened.

That is the spine of every proof path.

---

## The Story of One Problem

HR says Alice is Engineering.  
ISC identity says Alice is Sales.  
Roles still give Sales access.

Three truths cannot all be right.

Proof paths help you ask:
Which one is lying?

---

## Proof Path 1: Source Truth

Question: What does reality say?

Best proof:
The source itself.

You are not proving ISC yet.  
You are proving the world.

If HR says Alice is Engineering:
- Open HR
- Check Alice
- Capture timestamp

If this is wrong, stop.  
Everything else is innocent.

---

## Proof Path 2: Extraction

Question: Did ISC even see Alice?

Best proof:
Logs showing filters, pages, and returned records.

Supporting proof:
Job counts.

If Alice is not in logs:
She never entered the engine.  
Nothing after this matters.

---

## Proof Path 3: Normalization and Mapping

Question: Did ISC shape Alice correctly?

Best proof:
API view of account attributes.

Supporting proof:
Mapping preview.

If source is right but account is wrong:
The lie was born here.

---

## Proof Path 4: Persistence

Question: What did ISC *decide* about this record?

Best proof:
Account metadata in API:
- Created
- Updated
- Last seen
- Disabled

If ISC matched wrong record:
Everything after this is poisoned.

---

## Proof Path 5: Correlation

Question: Does this account belong to the right human?

Best proof:
API link from account → identity.

If this is wrong:
Stop everything. Fix this first.

Wrong correlation is the most dangerous lie.

---

## Proof Path 6: Identity Evaluation

Question: Why does identity show this value?

Best proof:
API identity attributes with source metadata.

If account is right but identity is wrong:
The rulebook is lying.

---

## Proof Path 7: Recompute

Question: Given this identity, is access correct?

Best proof:
API access model.

If identity is right but access is wrong:
Judgment failed, not aggregation.

---

## Proof Path 8: UI

Question: Is UI just late?

Best proof:
API already told you truth.

UI is only visibility, not reality.

---

## Walking the Full Proof Path

When something is wrong, walk like this:

Reality → Extraction → Mapping → Persistence → Correlation → Identity → Access → UI

At each step ask:

Is this still true?

The first “no” is where truth broke.

---

## What a Fix Playbook Is

A fix playbook is not creativity.  
It is memory.

It says:
“When this pattern appears, this fix is safe.”

Playbooks exist so you don’t invent danger every time.

---

## Playbook A – Missing Accounts

Symptom:
User exists in source, not in ISC.

Proof:
Source right → Extraction wrong.

Fix flow:
Check filters → check pagination → run full once → validate.

Never:
Touch identity or correlation first.

---

## Playbook B – Account Fields Wrong

Symptom:
Account exists but values empty or weird.

Proof:
Extraction right → Mapping wrong.

Fix flow:
Preview mapping → fix types → fix transforms → rerun small scope.

Never:
Touch identity profile first.

---

## Playbook C – Identity Field Wrong

Symptom:
Account correct, identity wrong.

Proof:
Mapping right → Evaluation wrong.

Fix flow:
Check precedence → check derived logic → re-evaluate identity.

Never:
Rerun aggregation blindly.

---

## Playbook D – Access Wrong

Symptom:
Identity right, access wrong.

Proof:
Evaluation right → Recompute wrong.

Fix flow:
Check recompute status → fix rule logic → rerun recompute.

Never:
Change correlation.

---

## Playbook E – Delta Skipped Changes

Symptom:
Source changed, ISC never updates.

Proof:
Source right → Delta memory wrong.

Fix flow:
Inspect token → reset delta once → validate → re-enable delta.

Never:
Reset delta repeatedly.

---

## Safety Rules

Before any fix, ask:

- How many people will this touch?
- Can I test with one?
- Can I undo it?

High-risk moves:
- Mass uncorrelate
- Delete accounts
- Reset delta on large sources

Treat these like production surgery.

---

## After Fix: Prove Again

Never trust your own fix.

Walk the proof path again:

Reality → Account → Identity → Access → UI

If you don’t re-prove, you don’t know.

---

## Mental Model

Proof paths are how you find truth.  
Playbooks are how you change truth safely.

Never fix without proof.  
Never prove without fixing.

---

## You Understand This If You Can Answer

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

