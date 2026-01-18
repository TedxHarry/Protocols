
# Part 8 – Identity Refresh and Recompute (Story-Driven)

[⬅️ Back to Home](../README.md)

---

## Big Idea

After identity is built, access is still not decided.

At this point ISC knows:
- Who the person is
- What their attributes are

But it does NOT yet know:
- What access they should have

Recompute is the phase where ISC asks:

“Given who this person is now, what should they be allowed to do?”

This is where identity turns into access.

---

## Where This Fits in the Engine

Trigger → Extract → Normalize → Persist → Correlate → Evaluate → **Recompute** → Publish

Evaluation answered: Who is this person?  
Recompute answers: What should this person have?

---

## What ISC Is Thinking During Recompute

ISC is not looking at raw accounts anymore.

It is thinking:

- Based on identity fields…  
- Based on role rules…  
- Based on access profile rules…  

What access should exist now?

It is making decisions like:
- Add this role
- Remove that role
- Grant this access profile
- Revoke that one

Recompute is judgment, not copying.

---

## Identity Refresh vs Recompute

They sound similar but think differently.

Identity Refresh thinks:
“Do I still know who this person is?”

Recompute thinks:
“Given who they are, what should they have?”

Refresh builds the person.  
Recompute builds their access.

They often run together, but they solve different problems.

---

## A Slow Story: Alice Changes Department

Alice moves from Sales to Engineering in HR.

Earlier phases already did:
- Account updated
- Correlation still correct
- Identity profile updated department = Engineering

So now ISC knows:
Alice is now an Engineering person.

But she still has Sales access.

Recompute now asks:
What does an Engineering person get?

---

## What Recompute Does With Alice

Role rules say:
- Sales role if department = Sales
- Engineering role if department = Engineering

Recompute logic:
- Remove Sales role
- Add Engineering role

Now Alice’s access matches her identity.

---

## Why Access Can Lag Behind Identity

Recompute is often asynchronous.

That means:
- Aggregation job may say Completed
- Identity shows new department
- Access still shows old roles

This does not always mean failure.  
It often just means recompute has not finished yet.

This delay confuses people into rerunning aggregation unnecessarily.

---

## What Goes Wrong Here

When access is wrong but identity is right, usually:

- Recompute has not run yet
- Recompute failed
- Role rules do not match identity fields

The problem is rarely aggregation at this point.

---

## What Recompute Never Does

Recompute does NOT:
- Re-read sources
- Re-link accounts
- Change identity fields

It only changes access.

So rerunning aggregation to fix access is usually the wrong instinct.

---

## Full Story Example

Run 1:
Alice = Sales  
→ Sales role granted

Run 2:
Alice becomes Engineering  
Identity updated first  
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

Business risk lives here, not in aggregation.

---

## How to Think When It Breaks

Do not start with:
“Why is aggregation broken?”

Ask:
- Is identity correct?
- Did recompute run?
- Did recompute succeed?
- Do role rules match identity?

Only then change anything.

---

## Classic Failure Story

HR changed department for 200 users.

Identities updated correctly.  
Roles stayed old.

Team reran aggregation many times.

Real issue:
Recompute queue was stuck.

Aggregation was never broken.

---

## How to Know You Understand Recompute

You understand this phase if you can answer:

- What question is recompute answering?  
- Why can identity be right while access is wrong?  
- Why is rerunning aggregation often the wrong fix?  
- Why does business risk live here?  

---

## Mindset

Aggregation builds truth.  
Evaluation builds the person.  
Recompute builds access.

If access is wrong,  
do not rebuild truth blindly.  
Fix the judgment that gives access.

---

### Navigation
⬅️ Previous: [Part 7 – Identity Evaluation](./Part_7_Identity_Profile_Evaluation_Engine.md)  
🏠 Home: [README](./README.md)  
➡️ Next: [Part 9 – Delta State](./Part_9_Delta_State_and_Token_Lifecycle.md)

