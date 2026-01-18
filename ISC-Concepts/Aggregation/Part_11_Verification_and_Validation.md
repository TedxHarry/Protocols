
# Part 11 – Verification and Validation (Story-Driven)

[⬅️ Back to Home](../README.md)

---

## Big Idea

Verification and validation are how you separate truth from illusion.

A job can say Completed.  
The UI can look fine.  
Access can “mostly” work.

And still, the system can be wrong.

Verification and validation answer one question:

“Do I *know* this is right, or am I just hoping?”

---

## Where This Fits in the Engine

Trigger → Extract → Normalize → Persist → Correlate → Evaluate → Recompute → Publish  
                                                                ↑  
                                                        Verification & Validation

Verification and validation do not change data.  
They judge whether data deserves to be trusted.

---

## Verification vs Validation

They sound similar, but they think differently.

Verification asks:
“Did each step do what it claimed to do?”

Validation asks:
“Does the final result match reality and expectation?”

Verification is about the engine.  
Validation is about the business.

---

## What the System Is Not Telling You

The system tells you:
- Job status  
- Counts  
- Labels  

It does not tell you:
- Which record was skipped  
- Which value was dropped  
- Which rule misfired  

Verification and validation are how *you* become the judge.

---

## A Simple Story

You run HR aggregation.

UI says: Completed.  
Alice shows department = Sales.

But HR says Alice is Engineering.

Now you must decide:

Is the system wrong?  
Or is your expectation wrong?

This is where verification and validation begin.

---

## Step 1: Start With One Person

Never start with “all users.”

Pick one person:
- Someone you know well  
- Someone who recently changed  
- Someone with clear source data  

You are not testing scale.  
You are testing truth.

---

## Step 2: Validate Against Reality

Ask:
What does the real world say?

Check:
- HR record  
- Source system UI  
- Source API  

Now compare:
What does ISC show for the same person?

If they do not match, you already know:
Something is wrong.

You just don’t know where yet.

---

## Step 3: Verify the Engine Path

Now walk the engine in order:

1) Was the person read?  
2) Was the data shaped correctly?  
3) Was the account written correctly?  
4) Was it linked to the right identity?  
5) Did identity choose the right value?  
6) Did recompute apply the right access?  

You are retracing the system’s thinking.

---

## How to Ask the Right Questions

Do not ask:
“Why is this wrong?”

Ask:
- Did extraction even see this person?  
- Did mapping drop the value?  
- Did persistence match the right record?  
- Did correlation link to the right identity?  
- Did evaluation choose the right source?  
- Did recompute run and succeed?  

Each “yes” moves you forward.  
Each “no” tells you where truth broke.

---

## Failure Story

Alice changed department in HR.

UI shows Completed.  
Identity still shows old department.

Team blamed roles.

But verification showed:
Extraction never read Alice, because filtering excluded her.

If she was not read, nothing else mattered.

---

## Verification Is About Proof

You are not guessing.  
You are proving.

You prove by:
- Comparing counts  
- Checking single records  
- Reading logs  
- Tracing one person end to end  

If you cannot prove it, you do not know it.

---

## Validation Is About Meaning

Even if engine is correct, business can still be wrong.

Example:
System says Alice is Active.  
Business says Alice left.

Engine did what rules said.  
Rules were wrong.

Validation asks:
“Does this behavior make sense in real life?”

---

## Why People Skip This

People skip verification and validation because:

- It feels slow  
- It feels manual  
- It feels obvious  

But skipping it creates:
- Blind trust  
- Silent failures  
- Big surprises later  

---

## How to Think When It Feels Wrong

Do not rerun jobs blindly.

Do this instead:

1) Pick one person  
2) Validate against source  
3) Verify each engine step  
4) Fix the first broken truth  
5) Only then rerun anything  

---

## Mental Model

Verification = Is the engine telling the truth?  
Validation = Does the truth match reality?

Without both, you are just hoping.

---

## You Understand This If You Can Answer

- Why start with one person?  
- Difference between verification and validation?  
- Why UI is never enough?  
- How to walk the engine logically?  
- Why “Completed” is not proof?  

---

### Navigation
⬅️ Previous: [Part 10 – Result Semantics](../Part_10_Result_Semantics_Story_v2.md)  
🏠 Home: [README – Aggregation Master Series](../README.md)  
➡️ Next: [Part 12 – Troubleshooting Playbook](../part_12_troubleshooting_playbook.md)
