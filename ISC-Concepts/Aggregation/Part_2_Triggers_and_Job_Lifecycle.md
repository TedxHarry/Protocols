# Part 2 – Triggers and Job Lifecycle — Teaching Mastery Edition

[⬅️ Back to Home](../README.md)

---

## Why This Part Exists

Most people think aggregation starts when data starts moving.

That’s wrong.

Aggregation really starts when a **job** is born.

If you don’t understand jobs, you will:
- Click Run again and again,
- Panic when nothing seems to happen,
- Trust “Completed” when the real work is still happening elsewhere.

This part teaches you to see what actually happens
from the moment you press Run to the moment ISC truly finishes.

Keep this sentence in mind:

**Aggregation doesn’t start with data. It starts with a job.**

---

## Where This Fits in the Big Engine

Master Flow:

Trigger → Extract → Normalize → Persist → Correlate → Evaluate → Recompute → Publish

This part owns the very first word:

**Trigger.**

If trigger or job handling is wrong,
none of the later steps even get a chance to run correctly.

---

## The Mental Model

```
Trigger
   → Job is created
     → Job waits
       → Worker runs it
         → Job ends
           → Other background work may still continue
```

So when you click Run, you are not starting extraction.
You are asking the system:

“Please create a job and put it in line.”

---

## Running Story (We Will Use This)

Source: HR system  
People: Alice, Bob, Carol  
Accounts: HR_Alice, HR_Bob, HR_Carol  

We are ready. We click Run.

What actually happens next?

---

## What a Trigger Really Does

There are three ways to trigger aggregation:

- Manual: you click Run  
- Scheduled: time rule starts it  
- API: another system calls it  

All three do the same thing:

They **do not** start reading data.  
They **create a job request.**

In our story:
We click Run for HR.  
ISC creates a job that says:

- Source = HR  
- Type = full or delta  
- Requested by = you  
- Time = now  

Then that job is placed in a queue.

---

## What a Job Really Is

A job is not the work.
A job is a **plan** for work.

It contains:
- Which source to run
- What type of run
- Which connector
- Who asked for it
- When it was created

After creation, the job just waits.

This is why you click Run and see nothing happening.
Something did happen:
A job exists, but no worker has picked it up yet.

---

## Workers and the Queue

Workers are the engines that actually do work.

There are only so many workers.

If many jobs exist:
- Some run
- Some wait

This creates a queue.

In our story:
If two other jobs are already running,
our HR job will sit in Queued state.

Nothing is broken.
It is just waiting its turn.

---

## Job States in Human Language

Job status is not technical.
It’s just life stages.

- Queued → waiting for a worker  
- Running → a worker is executing it  
- Completed → worker finished  
- Failed → worker stopped due to error  
- Warning/Partial → some parts failed  

So our HR job goes:

Queued → Running → Completed

---

## What “Completed” Really Means

Completed means only one thing:

“The worker finished its assignment.”

It does **not** guarantee:
- Data is correct
- Identity is updated
- UI is refreshed
- Recompute is done

Many things run after the job ends:
- Identity evaluation
- Access recompute
- Indexing
- UI refresh

So you may see:

Job = Completed  
UI = Still old

That is not always a bug.
It is often timing.

---

## Walk the Story Slowly

We click Run on HR.

1) Trigger creates job  
2) Job enters queue  
3) Worker becomes free  
4) Worker picks job  
5) Job status = Running  
6) Worker extracts and writes data  
7) Worker finishes  
8) Job status = Completed  
9) Other engines continue working in background  

Only after all background work finishes
does the UI fully reflect reality.

---

## Why Jobs Look “Stuck”

Usually for three reasons:

### Queued Too Long
- All workers are busy
- Too many jobs at once

### Running Too Long
- Source is slow
- Network is slow
- Connector is blocked

### Looks Finished but UI Wrong
- Job finished
- Recompute or indexing still running

Killing jobs blindly usually makes things worse.

---

## Interactive Pause

Scenario:

You click Run.  
Status shows Queued for 20 minutes.

Question:
Is something broken?

Pause. Think.

Answer:
Not necessarily.
It probably means no worker is free.
Check how many jobs are already running.

---

## Overlapping Jobs: The Silent Killer

Running two jobs on the same source at the same time is dangerous.

They can:
- Overwrite each other’s state
- Break delta memory
- Cause missing-from-feed errors

This is why overlapping schedules are risky.

It’s not about time.
It’s about shared memory.

---

## Why People Panic Here

Because they:
- Click Run many times
- Schedule too often
- Kill jobs too fast
- Trust Completed too much

This creates chaos that looks like data bugs,
but really started as job chaos.

---

## Debug Playbook

When something looks wrong, ask in this order:

1) Was a job created?  
2) What is its status?  
3) Is it queued or running?  
4) Are other jobs blocking it?  
5) Did it complete recently?  
6) Are background engines still running?  

Only then blame extraction or mapping.

---

## Visual Debug Flow

```
Did trigger create a job?
   ↓
Is job queued or running?
   ↓
Are workers busy?
   ↓
Did job complete?
   ↓
Is recompute/indexing still running?
```

---

## Proof Paths

Use more than UI:

- UI → job list and timestamps  
- API → real job objects  
- Logs → what worker is actually doing  

UI alone lies by delay.

---

## What Must Not Happen

- Do not spam Run  
- Do not overlap same-source jobs  
- Do not trust Completed blindly  
- Do not kill jobs without reason  

---

## Safe Fixes

- Too many queued jobs → stagger schedules  
- Overlap → fix timing rules  
- Slow jobs → test with smaller scope  

---

## The One Sentence That Defines Mastery

Before you ask “Why is my data wrong?”, ask:

**What is happening with my job right now?**

---

## Mastery Check

Answer without notes:

- What does a trigger really do?  
- What is a job actually?  
- Why can a job wait after you click Run?  
- What does Completed really mean?  
- Why are overlapping jobs dangerous?  

---
### Navigation
⬅️ Previous: [Part 1 – Before Aggregation Can Run (Pre‑Flight)](./Part_1_Before_Aggregation_Can_Run_Pre_Flight.md)  
🏠 Home: [README – Aggregation Master Series](./README.md)  
➡️ Next: [Part 3 – Extraction Phase](./Part_3_Extraction_Phase_Reading_Data.md)
