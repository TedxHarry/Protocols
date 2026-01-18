# Part 11 – Verification and Validation — Teaching Mastery Edition

[⬅️ Back to Home](../README.md)

---

## The One Question This Part Answers

**Do I *know* this is right, or am I just hoping?**

Verification and validation do not change data.  
They decide whether data deserves your trust.

Keep this line in mind:

**Jobs report progress. Verification proves truth. Validation proves meaning.**

---

## Where This Fits in the Engine

Trigger → Extract → Normalize → Persist → Correlate → Evaluate → Recompute → Publish  
                                                                ↑  
                                                     Verification & Validation

They sit above everything.  
They judge the engine and the business, not the buttons.

---

## Mental Model

```
Engine runs
   → It claims what it did
     → You prove what actually happened
       → Then you judge if that result makes sense
```

- Verification = Is the engine telling the truth?
- Validation   = Does that truth match reality and expectation?

---

## Verification vs Validation (Human Language)

Verification asks:
“Did each step do what it claimed?”

Validation asks:
“Even if it did, is the result what the business expects?”

Engine can be right. Business can still be wrong.  
Business can be right. Engine can still be wrong.

You need both.

---

## What the System Is Not Telling You

The system happily tells you:
- Status
- Counts
- Labels

It does not tell you:
- Which record was skipped
- Which value was dropped
- Which rule silently misfired

That silence is why this part exists.

---

## Guided Story

You run HR aggregation.

UI says: Completed.  
Alice shows department = Sales.

But HR says Alice is Engineering.

Now you must decide:

- Is ISC wrong?  
- Is HR wrong?  
- Or is your expectation wrong?  

This is the moment where hope must become proof.

---

## Rule 1: Start With One Person

Never start with “all users.”

Pick one person who:
- Recently changed
- You understand well
- Has clear source data

You are not testing scale.  
You are testing truth.

---

## Step 1 – Validate Against Reality

Ask first:

“What does the real world say?”

Check:
- Source UI
- Source API
- HR or business record

Now compare:
What does ISC show for the same person?

If they don’t match, you already know:
Truth broke somewhere.

You just don’t know where yet.

---

## Step 2 – Verify the Engine Path

Now you walk the engine like a detective.

Ask in this order:

1) Was this person extracted?  
2) Was their data shaped correctly?  
3) Was their account written correctly?  
4) Was it linked to the right identity?  
5) Did identity choose the right value?  
6) Did recompute apply the right access?  

You are retracing the system’s thinking.

---

## How to Ask Better Questions

Do not ask:
“Why is this wrong?”

Ask:
- Did extraction even see this person?  
- Did mapping drop this field?  
- Did persistence overwrite someone else?  
- Did correlation link wrongly?  
- Did evaluation choose wrong source?  
- Did recompute run and succeed?  

Each “yes” moves you forward.  
Each “no” shows where truth broke.

---

## Interactive Pause

Scenario:

Alice changed department in HR.  
ISC still shows old department.

Question:
Which do you check first?

Pause. Think.

Answer:
Did extraction even read Alice’s change?

If she never entered the engine, nothing else matters.

---

## Failure Story

Alice changed in HR.

Job said Completed.  
Identity still old.

Team blamed roles.

Verification showed:
Filter excluded Alice. She was never read.

Engine did exactly what it was told.  
Expectation was wrong.

---

## Verification Is About Proof

You are not guessing.  
You are proving.

You prove by:
- Checking one person end to end  
- Comparing with source  
- Reading logs  
- Checking stored objects via API  

If you cannot prove it, you do not know it.

---

## Validation Is About Meaning

Even when engine is perfect, business can still be wrong.

Example:
System says Alice is Active.  
Business says Alice left.

Engine followed rules.  
Rules were wrong.

Validation asks:
“Does this behavior make sense in real life?”

---

## Illusions This Phase Creates

- Completed means correct  
- Counts matching means truth  
- UI showing green means safe  
- Engine logic equals business logic  

All can be false.

---

## Traps That Fool Smart People

- Trusting counts more than samples  
- Debugging roles before identity  
- Rerunning before proving  
- Trusting UI without API or logs  

These are senior-level mistakes.

---

## Debug Mindset

When something feels wrong:

1) Pick one person  
2) Validate against source  
3) Verify engine path  
4) Fix first broken truth  
5) Then rerun if needed  

Never rerun before proving.

---

## Visual Debug Flow

```
Pick one person
   ↓
Validate with source
   ↓
Verify each engine phase
   ↓
Find first broken truth
   ↓
Fix, then rerun
```

---

## Proof Paths

To prove truth:

- Source UI or API  
- ISC account object  
- Identity object  
- Access assignments  
- Logs showing read, map, write  

Truth lives in objects and logs, not in labels.

---

## What This Phase Does NOT Do

- It does not fix data  
- It does not rerun jobs  
- It does not change rules  

It only decides whether you deserve to trust what you see.

---

## What Must Not Happen

- Do not rerun blindly  
- Do not debug with groups first  
- Do not trust labels  
- Do not skip proof  

---

## Safe Fixes

- Extraction wrong → fix source, filter, or connector  
- Mapping wrong → fix transform and preview  
- Correlation wrong → fix match logic  
- Evaluation wrong → fix identity profile  
- Recompute wrong → fix rules and recompute  

Always fix the first broken truth.

---

## The One Sentence That Defines Mastery

Before you act, ask:

**What proof do I have that this is wrong?**

---

## Mastery Check

Answer without notes:

- Difference between verification and validation?  
- Why start with one person?  
- Why “Completed” is never proof?  
- What is your debug order?  
- What does this phase absolutely not do?  

---
### Navigation
⬅️ Previous: [Part 10 – Result Semantics](./Part_10_Result_Semantics.md)  
🏠 Home: [README](./README.md)  
➡️ Next: [Part 12 – Troubleshooting](./Part_12_Troubleshooting_Playbook.md)

