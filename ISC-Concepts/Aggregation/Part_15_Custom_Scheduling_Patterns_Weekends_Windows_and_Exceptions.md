# Part 15 – Custom Scheduling Patterns (Weekends, Windows, and Exceptions) — Teaching Mastery Edition

[⬅️ Back to Home](../README.md)

---

## The One Question This Part Answers

**How do I bend time to business needs without breaking truth?**

Custom schedules are not calendar tricks.  
They change how stale data is allowed to become.

Keep this in mind:

**Every exception is a promise about staleness.**

---

## Where This Lives in the Engine

Trigger → Extract → Normalize → Persist → Correlate → Evaluate → Recompute → Publish

Custom schedules control the Trigger,  
but they quietly reshape every step that follows.

---

## Mental Model

```
Business rule about time
   → Run gaps and bursts
     → Delta memory behavior
       → Recompute waves
         → How stale truth is allowed to be
```

If you don’t define staleness on purpose,  
you accept it by accident.

---

## A Real Story

Business says:
“Don’t run HR on weekends.”

What they usually mean:
“Nothing important changes on weekends.”

What the system hears:
“It is okay if identity is wrong for up to 48 hours.”

If that is not true, the schedule is lying before the engine even starts.

---

## Translate Requests Into Truth

Never accept a rule like:
- “Skip weekends”
- “Run only in window”
- “Pause during payroll”

Translate it to a truth statement:

- “Identity may be stale for 48 hours.”  
- “Access changes may wait until morning.”  
- “Leavers may keep access until window opens.”  

If the business is not okay with that sentence,  
the schedule is wrong.

---

## Delta and Gaps

Delta relies on memory.

When you skip days:
- Delta window becomes larger  
- Token risk increases  
- Monday runs become heavier  

Skipping does not stop change.  
It only delays when you see it.

So every gap must answer:

“How big a change set am I willing to process at once?”

---

## Common Patterns (Taught Through Reasoning)

### Pattern: Weekdays Only

Why people want it:
- Systems are slow on weekends
- Teams are not watching

What it really means:
- Joiners/leavers wait up to 48 hours

When it works:
- Business accepts weekend staleness

What usually breaks:
- Monday backlog
- Recompute storms

---

### Pattern: Weekdays Delta, Weekly Full

Why it exists:
- Delta is fast
- Full corrects drift

How to think:
- Delta handles daily noise  
- Full resets memory weekly  

This pattern accepts:
- Small daily risk  
- Big weekly cleanup  

It works when you plan the full carefully so it does not collide with Monday business traffic.

---

### Pattern: Twice Per Day

Why people want it:
- Faster joiners and leavers

What it risks:
- Overlap
- Recompute stacking
- Worker exhaustion

Before doing this, you must know:
Worst-case duration, not average.

---

### Pattern: Maintenance Window Only

Why it exists:
- Source is fragile
- Vendor restricts access

Hidden danger:
If a run slips, it may retry into forbidden time, or skip entirely.

So you must design:
What happens if today’s window fails?

Do you retry tomorrow?  
Do you force a full?

---

### Pattern: Blackout During Events

Example:
Payroll cutover, quarter close, mergers.

What blackout really means:
“I accept that truth will freeze for a while.”

Every blackout must come with a plan:
How do we safely wake truth back up?

That is the catch-up run.

---

## Catch-Up Runs: Waking Truth Back Up

When you skip or pause, you must choose:

1) Big delta catch-up  
2) Controlled full reset  

Big delta is lighter but riskier.  
Full reset is heavier but safer.

Choose based on:
- How critical missed changes are
- How risky delta memory feels

---

## Dependency Still Matters

Even with fancy calendars:

People-creating sources must run first.  
Access-attaching sources must follow.

If HR skips but AD runs:
AD will see stale people and mis-correlate.

So exceptions must respect dependency chains,  
not just calendars.

---

## The Hidden Enemy: Overlap by Accident

Custom rules often create overlap accidentally.

Example:
- Delta twice per day  
- Weekly full  
- Maintenance window  

If the full runs long, it may collide with the next delta.

Overlap is not a scheduling bug.  
It is a math bug.

You must space based on worst case, not hope.

---

## Interactive Pause

HR runs only weekdays.  
Monday delta runs at 5 AM.  
Worst-case HR job = 90 minutes.

Question:
Is a 6 AM AD run safe?

Pause. Think.

Answer:
No. If HR takes 90 minutes, AD will start before HR finishes.  
That breaks dependency and risks correlation.

---

## Why Custom Schedules Create Illusions

You may see:
- Jobs Completed
- UI green
- But identity stale

Because time silently changed truth expectations,  
not logic.

Time can lie more politely than code.

---

## Traps That Fool Smart People

- Accepting calendar rules without defining staleness  
- Skipping HR but not skipping downstream  
- Adding weekly full without adjusting spacing  
- Trusting average duration instead of worst case  

These are not beginner mistakes.  
They are rushed-operator mistakes.

---

## Debug Mindset for Custom Schedules

When timing feels wrong, ask:

1) What staleness did we promise?  
2) Is that acceptable?  
3) What is worst-case duration?  
4) Do any rules create overlap?  
5) Does dependency still flow correctly?  

Fix rhythm before fixing logic.

---

## Visual Rhythm Check

```
Business rule about time
   ↓
What staleness does it allow?
   ↓
Worst-case job duration
   ↓
Do any runs overlap?
   ↓
Do dependencies still flow?
```

---

## What This Phase Does NOT Do

- It does not fix mapping  
- It does not fix correlation  
- It does not fix access rules  

It only decides how long truth is allowed to sleep.

---

## Safe Design Principles

- Always define acceptable staleness  
- Space by worst-case, not average  
- Respect dependency order  
- Pair delta with periodic full  
- Plan catch-up for every blackout  

---

## The One Sentence That Defines Mastery

Before you accept a calendar rule, ask:

**What lie about freshness am I agreeing to?**

---

## Mastery Check

Answer without notes:

- What does “skip weekends” really mean in truth?  
- Why does delta become riskier after gaps?  
- Why must dependencies still flow during exceptions?  
- Why is worst-case duration more important than average?  
- Why does every blackout need a wake-up plan?  

---
### Navigation
⬅️ Previous: [Part 14 – Scheduling](./Part_14_Operational_Scheduling_and_Avoiding_Overlap.md)  
🏠 Home: [README](./README.md)  
➡️ Next: [Part 16 – Observability](./Part_16_Observability_and_Monitoring.md)

