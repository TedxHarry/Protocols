
# Part 9 – Delta State and Token Lifecycle (Story-Driven)

[⬅️ Back to Home](../README.md)

---

## Big Idea

Delta is memory.

Without delta, the system forgets everything and reads everything again.  
With delta, the system tries to remember where it stopped.

Delta is how ISC says:

“Last time, I read up to here.  
Next time, I will start from there.”

If this memory is wrong, everything after it is wrong — quietly.

---

## Where This Fits in the Engine

Trigger → **Extract (Delta Logic)** → Normalize → Persist → Correlate → Evaluate → Recompute → Publish

Delta lives inside extraction, but it decides what even gets a chance to exist downstream.

---

## What ISC Is Thinking During Delta

ISC is not thinking about people or access here.

It is thinking:

“Where did I stop last time?”  
“What is the next thing after that?”

It stores a marker called a token:
- Timestamp  
- ID  
- Vendor change token  

Next run, it asks:

“Give me everything after this marker.”

---

## A Slow Story: One Change Over Time

Source: HR  
People: Alice, Bob, Carol  

First run:
Full run.  
All read.  
Token stored = 10:00 AM

At 11:00 AM:
Bob changes department.

Next delta run:
Connector asks: “After 10:00 AM.”  
Source returns Bob only.

Everything else is ignored, not because it is wrong, but because it is old.

---

## How Delta State Is Born

Delta memory is born during a successful full run.

If first run is broken, memory is broken from the start.

From then on:
Each successful run updates the memory.  
Each failed run risks poisoning it.

Delta is only as healthy as the last good run.

---

## Where Memory Lives

Memory may live:
- In ISC job state  
- In the connector  
- In the source  

UI rarely shows it clearly.  
Logs are the real diary of delta.

---

## When Memory Lies: Acting Like Full

Sometimes delta forgets and rereads everything.

This happens when:
- Schema changes  
- Mapping changes  
- Connector upgrades  
- Permissions change  

ISC may decide:
“I no longer trust my memory.”

So it rereads everything.  
It may still call it “delta,” but behavior is full.

---

## When Memory Jumps: Skipping Data

More dangerous than rereading is skipping.

This happens when:
- Token jumps too far ahead  
- Time zone mismatch  
- Clock drift  
- Connector bug  
- Failed run saved wrong token  

Result:
Changes exist, but never arrive.

Everything looks stable but is actually stale.

---

## The Most Dangerous Scenario

Token = 12:00 PM  
Bob changed at 11:00 AM  

Next run:
Ask: “After 12:00 PM.”  
Source returns nothing.

Bob is lost forever, unless delta is reset.

---

## Resetting Memory

Reset delta means:

“Forget everything you remember.”

Next run becomes full.

This can:
- Fix missing data  
- Overload systems  
- Change behavior dramatically  

Never reset casually.

---

## Why This Phase Is Fragile

Delta breaks when:
- You change schema  
- You change mapping  
- You change filters  
- You upgrade connectors  
- Jobs fail mid-run  

Every change risks memory confusion.

---

## How to Think When It Breaks

Do not ask:
“Why didn’t the change arrive?”

Ask:
“Did delta allow the change in?”

Then check:
- What token was used  
- What query was sent  
- What source returned  

Logs tell the truth.

---

## Classic Failure Story

Bob changed department.

Jobs said: Delta Completed.  
Identity never updated.

Root cause:
Previous run failed but still saved token.  
Token jumped ahead.  
Bob was skipped forever.

Not correlation.  
Not identity logic.  
Memory was wrong.

---

## How to Know You Understand Delta

You understand delta if you can answer:

- What is delta really doing?  
- Where does memory come from?  
- How can memory skip silently?  
- Why is reset powerful and dangerous?  

---

## Mindset

Delta is not speed.  
Delta is trust in memory.

If memory lies,  
everything built on it lies quietly.

---

### Navigation
⬅️ Previous: [Part 8 – Recompute](./Part_8_Identity_Refresh_and_Recompute.md)  
🏠 Home: [README](./README.md)  
➡️ Next: [Part 10 – Result Semantics](./Part_10_Result_Semantics.md)

