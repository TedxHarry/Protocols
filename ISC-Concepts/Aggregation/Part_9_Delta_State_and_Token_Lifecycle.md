# Part 9 – Delta State and Token Lifecycle — Teaching Mastery Edition

[⬅️ Back to Home](../README.md)

---

## Why This Part Exists

This part answers one sharp question:

**What does the system remember between runs?**

Delta is not about speed.  
Delta is about memory.

If memory is wrong, the system will be wrong — politely, quietly, and consistently.

Keep this sentence in mind:

**Delta is not optimization. Delta is trust in memory.**

---

## Where This Fits in the Engine

Trigger → **Extract (Delta Logic)** → Normalize → Persist → Correlate → Evaluate → Recompute → Publish

Delta lives inside extraction.  
But it decides what even gets a chance to exist downstream.

If delta blocks data, nothing later can fix it.

---

## The Mental Model

```
Last run memory
   → Ask source: “What changed after this?”
     → Only those changes enter the engine
```

Delta is not looking at people, identity, or access.  
It is only asking:

“Where did I stop last time?”

---

## What Delta Really Is

Delta is a remembered marker.

That marker might be:
- A timestamp  
- An ID  
- A vendor change token  

On every run, the connector says:

“Give me everything that changed after this marker.”

So delta is a conversation:

- System: “I last read up to here.”  
- Source: “Here is everything after that.”  

If either side lies, truth disappears.

---

## Guided Story: One Change Over Time

Source: HR  
People: Alice, Bob, Carol  

### First Run
Full run.  
Everything is read.  
Delta memory stored = 10:00 AM

### At 11:00 AM
Bob changes department.

### Next Delta Run
Connector asks: “After 10:00 AM.”  
Source returns Bob only.

Bob enters the engine.  
Alice and Carol are ignored — not because they are wrong, but because they are old.

---

## How Delta Memory Is Born

Delta memory is created only when a run finishes successfully.

That first good run is sacred.

If the first run is broken:
- Memory is broken from birth
- Every delta after it will be unreliable

From then on:
- Each successful run updates memory
- Each failed run risks poisoning memory

Delta is only as healthy as the last good run.

---

## Where Memory Lives

Delta memory may live in:
- ISC job state  
- Connector state  
- Source-side tokens  

UI rarely shows it clearly.  
Logs are the real diary of delta.

If you cannot see the token or timestamp, you are blind.

---

## Two Ways Memory Can Lie

### 1) Memory Forgets (Acts Like Full)

Delta suddenly rereads everything.

This happens when:
- Schema changes  
- Mapping changes  
- Connector upgrades  
- Permissions change  

System says:
“I don’t trust my memory anymore.”

So it rereads everything.

This is loud, slow, but usually safe.

---

### 2) Memory Jumps (Skips Data)

More dangerous.

This happens when:
- Token jumps forward incorrectly  
- Time zone mismatch  
- Clock drift  
- Connector bug  
- Failed run saved wrong token  

Result:
Changes exist but never arrive.

Everything looks stable.  
But truth is frozen.

---

## The Most Dangerous Scenario

Token = 12:00 PM  
Bob changed at 11:00 AM  

Next run asks:
“After 12:00 PM.”

Source returns nothing.

Bob is skipped forever — unless delta is reset.

This is silent data loss.

---

## Resetting Delta: Forgetting on Purpose

Reset delta means:

“Forget everything you remember.”

Next run becomes full.

This can:
- Fix missing data  
- Stress the source  
- Change behavior dramatically  

Never reset casually.

Reset is amnesia.  
Sometimes necessary. Always dangerous.

---

## Why This Phase Is Fragile

Delta breaks when:

- You change schema  
- You change mapping  
- You change filters  
- You upgrade connectors  
- Jobs fail mid-run  

Every change risks memory confusion.

Delta is the most fragile phase because it depends on history, not just logic.

---

## Interactive Pause

Scenario:

Last token = 10:00  
Bob changed at 10:30  
Next run shows no Bob change.

Question:
What is your first suspicion?

Pause. Think.

Answer:
Token probably jumped forward or was saved wrong.

Not correlation.  
Not identity logic.  
Memory.

---

## How to Think When Delta Breaks

Do not ask:
“Why didn’t the change arrive?”

Ask:
“Did delta even allow it in?”

Then check:
- What token was used  
- What query was sent  
- What source returned  

Logs are the truth.  
UI is a rumor.

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
Not mapping.  
Memory lied.

---

## Illusions This Phase Creates

- Delta is faster, so it must be better  
- Completed means nothing was missed  
- If no changes arrived, nothing changed  

All three can be false.

---

## Traps That Fool Smart People

- Trusting delta after major config changes  
- Ignoring failed runs that still update memory  
- Assuming source and ISC clocks agree  
- Resetting delta casually  

These are senior-level mistakes.

---

## Debug Playbook

When changes don’t arrive:

1) What token was used?  
2) What time/ID does it represent?  
3) What query was sent to the source?  
4) Did source have changes after that?  
5) Did token update correctly after run?  

Only then look downstream.

---

## Visual Debug Flow

```
What token was used?
   ↓
What does it represent?
   ↓
What query was sent?
   ↓
Did source have changes after that?
   ↓
Was token updated correctly?
```

---

## Proof Paths

To prove delta health:

- Job logs showing token before and after  
- Connector logs showing query  
- Source logs showing what was returned  
- Comparison with full run results  

Truth lives in history, not UI labels.

---

## What Must Not Happen

- Do not trust delta after big changes  
- Do not ignore failed runs  
- Do not reset casually  
- Do not debug downstream first  

---

## Safe Fixes

- Data missing → consider controlled full run  
- Token wrong → reset with planning  
- Source slow → widen delta window if supported  
- Too many failures → stabilize before trusting delta  

---

## The One Sentence That Defines Mastery

Before you ask “Why didn’t this change come?”, ask:

**What does the system remember about last time?**

---

## Mastery Check

Answer without notes:

- What is delta really doing?  
- Why is delta memory fragile?  
- What are the two ways memory lies?  
- Why is reset both powerful and dangerous?  
- What is your debug order for delta issues?  

---
### Navigation
⬅️ Previous: [Part 8 – Recompute](./Part_8_Identity_Refresh_and_Recompute.md)  
🏠 Home: [README](./README.md)  
➡️ Next: [Part 10 – Result Semantics](./Part_10_Result_Semantics.md)

