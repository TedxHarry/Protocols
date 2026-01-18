# Part 18 – Governance Impacts (Certifications, Requests, and Policies) — Teaching Mastery Edition

[⬅️ Back to Home](../README.md)

---

## The One Question This Part Answers

**How does bad data become bad decisions?**

Governance is where humans act on what the engine believes.  
If the engine is wrong or late, people make confident mistakes.

Keep this in mind:

**Aggregation creates truth. Governance spends it.**

---

## Where This Lives in the Engine

Trigger → Extract → Normalize → Persist → Correlate → Evaluate → Recompute → Publish

Governance does not change the engine.  
It consumes what the engine produces and asks humans to judge it.

So governance quality is always downstream of aggregation quality.

---

## Mental Model

```
Engine creates truth
   → Governance shows it to humans
     → Humans approve, revoke, or block
```

If the truth is wrong, governance does not stop.  
It simply spreads the wrong truth through human decisions.

---

## A Simple Story

Bob leaves the company Friday evening.  
HR updates him to Terminated.

But HR aggregation runs only on weekdays.

On Saturday:
- Bob still looks Active
- Bob still appears in certifications
- Bob can still request access
- Policies still treat Bob as valid

No button is broken.  
Only truth is late.

Governance didn’t fail.  
Time failed first.

---

## What Governance Really Uses

Governance never reads raw sources.  
It uses only what the engine already decided:

- Identity attributes like department, manager, location
- Account states like active or disabled
- Entitlements and memberships
- Roles and access profiles

So every governance feature is standing on aggregation’s shoulders.

If those shoulders are weak, governance falls.

---

## Certifications: Reviewing What the Engine Believes

Certifications answer:

“Does this person still need this access?”

But reviewers can only judge what they are shown.

If aggregation is stale:
- Leavers still appear active
- Old entitlements still appear assigned
- Wrong identities appear in scope

So reviewers approve access that should already be gone.

That is not a reviewer failure.  
It is a truth failure.

---

## Scope Depends on Identity Truth

Certification campaigns are often built using identity fields:
- Department
- Manager
- Location
- Lifecycle state

If identity evaluation is wrong, scope is wrong.

Example:
Alice moved from Sales to Engineering.  
Identity still says Sales.

Engineering campaign misses Alice.  
Sales campaign includes her wrongly.

Right people review wrong access.  
Wrong people review right access.

---

## Requests: Asking Based on Identity

Access requests depend on three truths:

- Who the requester is
- What they are eligible for
- Who should approve

All three depend on identity truth.

If manager is wrong:
Approvals go to the wrong person.

If department is wrong:
Eligibility logic fails.

So even a perfect workflow produces wrong outcomes when identity is wrong.

---

## Requests Depend on Recompute

Request systems usually hide access you already have.

If recompute is late or broken:
- You may request access you already have
- Or you may not see access you should request

This creates:
- Duplicate work
- Confused reviewers
- Lost trust

So request quality depends on recompute health, not just UI design.

---

## Policies: Logic Without Truth Is Noise

Policies detect risk:
- Separation of duties
- Toxic combinations
- Privileged access

Policies need two clean inputs:

- Correct entitlements
- Correct identity context

If entitlements are missing, policies do not fire.  
If identity attributes are wrong, policies fire on the wrong people.

So policy tuning is useless if aggregation is incomplete.

---

## How Aggregation Failures Appear as Governance Pain

Governance symptoms often look like governance bugs, but aren’t.

Pattern:
- Certifications show wrong people  
Root cause:
Identity evaluation or schedule

Pattern:
- Approvals go to wrong manager  
Root cause:
Manager attribute wrong

Pattern:
- Policies not triggering  
Root cause:
Entitlements not aggregated

Governance is the mirror.  
Aggregation is the face.

---

## The Leaver Risk Revisited

Bob leaves Friday.  
HR updates him.

But HR aggregation skips weekends.

Saturday:
- Bob still appears active
- Certifications include him
- Policies ignore him
- Requests still possible

This is not a governance bug.  
It is a freshness decision that became a security decision.

---

## Governance Needs Freshness Promises

If governance matters, you must define freshness like a contract.

Examples:
- Joiners must appear within 2 hours
- Leavers must lose access same day
- Manager changes must reflect by next business morning

Then you design schedules, delta, and monitoring to honor those promises.

Governance is only as strong as your freshness discipline.

---

## Proof When Governance Looks Wrong

Never start by blaming governance features.

Prove upstream first:

- Source truth
- Account truth
- Identity truth
- Recompute truth

If those are right, governance will usually be right too.

---

## Why Governance Creates Illusions

You may see:
- Campaign running
- Approvals flowing
- Policies quiet

And assume:
“Everything is safe.”

But silence can mean:
Truth never arrived.

Governance can only act on what it knows.

---

## Traps That Fool Smart People

- Running certifications on stale data  
- Tuning policies before fixing entitlements  
- Blaming workflow when identity is wrong  
- Trusting silence as safety  

These are not beginner mistakes.  
They are comfort mistakes.

---

## What This Phase Does NOT Do

- It does not fix data  
- It does not change identity  
- It does not repair aggregation  

It only shows you what wrong truth looks like when humans act on it.

---

## Debug Mindset for Governance

When governance looks wrong, ask:

1) Is identity fresh?  
2) Is account data complete?  
3) Did recompute run?  
4) Are entitlements correct?  
5) Is UI just late?  

Fix truth before fixing governance logic.

---

## Visual Truth-to-Governance Flow

```
Source truth
   ↓
Account truth
   ↓
Identity truth
   ↓
Access truth
   ↓
Governance decisions
```

If any arrow is broken, humans will decide wrongly.

---

## The One Sentence That Defines Mastery

Before you trust a governance decision, ask:

**How fresh and how correct was the truth it used?**

---

## Mastery Check

Answer without notes:

- Why is aggregation quality equal to governance quality?
- How does stale data corrupt certifications?
- Why do approvals fail when identity is wrong?
- Why do policies fail when entitlements are missing?
- Why is freshness a security decision?

---
### Navigation
⬅️ Previous: [Part 17 – Performance](./Part_17_Performance_Tuning_and_Scale.md)  
🏠 Home: [README](./README.md)  
➡️ Next: [Part 19 – Change Management](./Part_19_Change_Management_and_Safe_Updates.md)

