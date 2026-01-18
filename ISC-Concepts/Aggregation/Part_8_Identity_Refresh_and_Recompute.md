# Part 8 – Identity Refresh and Recompute — Teaching Mastery Edition

[⬅️ Back to Home](../README.md)

---

## Why This Part Exists

Up to now, the engine has answered:

- What data exists (Extraction)  
- What it looks like (Normalization)  
- What is remembered (Persistence)  
- Who owns which account (Correlation)  
- Who the person is (Evaluation)  

Part 8 answers the business question:

**Given who this person is, what access should they have now?**

This is where identity turns into power.

Keep this sentence in mind:

**Evaluation builds the person. Recompute builds their power.**

---

## Where This Fits in the Big Engine

Trigger → Extract → Normalize → Persist → Correlate → Evaluate → **Recompute** → Publish

Evaluation answered: “Who is this person?”  
Recompute answers: “What should this person have?”

They often run close together, but they are not the same job.

---

## The Mental Model

```
Identity (who you are)
   → Rules and policies
     → Access decisions (what you get)
```

Recompute does not copy data.  
It judges data.

It looks at identity and asks:

- Which roles apply?
- Which access profiles apply?
- Which access must be removed?

---

## Identity Refresh vs Recompute

They sound similar, but they think differently.

### Identity Refresh thinks:
“Do I still know who this person is?”  
It re-evaluates identity attributes.

### Recompute thinks:
“Given who this person is, what should they have?”  
It recalculates access.

Refresh builds the person.  
Recompute builds their access.

---

## A Guided Story: Alice Changes

Source: HR  
Alice moves from Sales to Engineering.

Earlier phases already did:

- HR account updated  
- Correlation still correct  
- Identity evaluation set department = Engineering  

So ISC now knows:

Alice is an Engineering person.

But she still has Sales access.

Recompute is now responsible for fixing that.

---

## What Recompute Does With Alice

Role rules:

- Sales role if department = Sales  
- Engineering role if department = Engineering  

Recompute logic:

- Identity says: Engineering  
- Remove Sales role  
- Add Engineering role  

Now Alice’s access matches who she is.

---

## Why Access Can Lag Behind Identity

Recompute is often asynchronous.

That means:

- Aggregation job may say Completed  
- Identity shows new department  
- Access still shows old roles  

This is not always a failure.  
It often just means recompute has not finished yet.

This delay tricks people into rerunning aggregation, which usually makes things worse.

---

## What Recompute Never Does

Recompute does NOT:

- Re-read sources  
- Re-link accounts  
- Change identity attributes  

It only changes access.

So if identity is wrong, recompute cannot fix it.  
And if access is wrong, rerunning aggregation is usually the wrong instinct.

---

## Interactive Pause

Scenario:

- Identity shows: Alice.department = Engineering  
- Access still shows: Sales role  

Question:
What should you check first?

Pause. Think.

Answer:
- Did recompute run?
- Did recompute succeed?
- Are role rules correct?

Not: “Let me rerun aggregation.”

---

## Full Story Example

Run 1:
Alice = Sales  
→ Sales role granted

Run 2:
Alice becomes Engineering  
Identity updates first  
Access still Sales for a while

Recompute runs later:
- Remove Sales role  
- Add Engineering role  

Only now does access reflect truth.

---

## Why This Phase Is Risky

If recompute is slow or broken:

- Leavers keep access  
- Movers keep old access  
- Joiners wait too long  

Business risk lives here, not in extraction.

You can have perfect data and still have dangerous access.

---

## What Goes Wrong Here

When access is wrong but identity is right, usually:

- Recompute never ran  
- Recompute failed  
- Role or access rules don’t match identity fields  
- Conditions are too strict or too loose  

The problem is rarely aggregation at this point.

---

## Classic Failure Story

HR changed department for 200 users.

- Identities updated correctly  
- Roles stayed old  

Team reran aggregation many times.

Real issue:
Recompute queue was stuck.

Aggregation was never broken.  
Judgment was.

---

## Debug Playbook

When access looks wrong, debug in this order:

1) Is identity correct?  
2) Did recompute run?  
3) Did recompute succeed?  
4) Do rules match identity fields?  
5) Are conditions too strict or too broad?  

Only after this, look at aggregation again.

---

## Visual Debug Flow

```
Is identity correct?
   ↓
Did recompute run?
   ↓
Did it succeed?
   ↓
Do rules match identity?
   ↓
Fix rules or rerun recompute
```

---

## Proof Paths

To prove recompute health:

- Identity view (who the person is)  
- Role and access assignments  
- Recompute job history  
- Logs for rule evaluation  

Truth lives in judgment results, not raw data.

---

## What Must Not Happen

- Do not rerun aggregation to fix access  
- Do not ignore recompute failures  
- Do not write rules without testing  
- Do not assume identity change means access change  

---

## Safe Fixes

- Access wrong → rerun recompute  
- Rule wrong → fix rule and recompute  
- Identity wrong → go back to evaluation phase  
- Queue stuck → fix recompute engine, not aggregation  

---

## The One Sentence That Defines Mastery

Before you ask “Why is access wrong?”, ask:

**Did the judgment that grants access run correctly?**

---

## Mastery Check

Answer without notes:

- What question does recompute answer?  
- Why can identity be right while access is wrong?  
- Why is rerunning aggregation often wrong?  
- Why does business risk live here?  
- What is your debug order for access issues?  

---

### Navigation
⬅️ Previous: [Part 7 – Identity Evaluation](./Part_7_Identity_Profile_Evaluation_Engine.md)  
🏠 Home: [README](./README.md)  
➡️ Next: [Part 9 – Delta State](./Part_9_Delta_State_and_Token_Lifecycle.md)

