
# Part 2 – Triggers and Job Lifecycle

[⬅️ Back to Home](../README.md)

---

## Purpose
This part explains how aggregation actually starts and what really happens from the moment you press Run until the system says Completed.

Many beginners think aggregation is one action: click Run and data appears.  
In reality, a job is created, queued, executed, monitored, and only then closed.

Understanding this lifecycle lets you answer:
- Why did my job not start yet?
- Why does it say Running forever?
- Why does it say Completed but data is still wrong?

---

## Where This Fits in the Master Flow

Master Flow:  
Trigger → Extract → Normalize → Persist → Correlate → Evaluate → Recompute → Publish

This part owns the very first word: Trigger.  
If trigger or job handling is wrong, nothing else even gets a chance.

---

## Mini‑Glossary

Trigger: the action that asks ISC to start aggregation.  
Job: the background process that does the work.  
Queue: the waiting line for jobs.  
Worker: the engine that actually runs jobs.  
Status: the current life stage of a job.

---

## Running Example

Source: HR system  
People: Alice, Bob, Carol  
Accounts: HR_Alice, HR_Bob, HR_Carol  

We are ready to press Run.  
What actually happens behind the scenes?

---

## How Aggregation Gets Triggered

Three ways aggregation can start:

Manual: a human clicks Run.  
Scheduled: ISC starts it by time rules.  
API: another system triggers it.

All three do the same thing:  
They create a job request.

ISC does not start reading data immediately.  
It first creates a job object and places it in line.

In our example, we click Run for HR. A job is created and placed in the queue.

---

## What a Job Really Is

A job is not the data move.  
A job is a plan that says:
- Which source
- Full or delta
- Which connector
- Who requested it
- When it was created

After creation, it enters the queue.  
If workers are busy, it waits.

This is why you click Run and nothing seems to happen.  
Something did happen: the job exists, but it is waiting.

---

## Job States in Human Language

Queued: waiting for a worker.  
Running: a worker picked it up.  
Completed: the job finished its tasks.  
Failed: it stopped because of an error.  
Warning/Partial: some parts worked, some didn’t.

In our example: Queued → Running → Completed.

---

## Workers and Concurrency

Workers are the engines that do the work.  
There are only so many.

If many jobs start at once, some must wait.  
If two jobs run on the same source, they can fight and corrupt state.

This is why overlapping schedules are dangerous.  
It’s not about time, it’s about shared state.

---

## What “Completed” Really Means

Completed only means the job finished running.  
It does not mean the data is correct.

A job can complete even if:
- Some records failed
- Some data was skipped
- Recompute and indexing are still running

In our example, HR job may say Completed, but Alice’s access may not show yet because background work is still catching up.

---

## Lifecycle Walkthrough

We click Run on HR.

Trigger → job request created.  
Job creation → job object stored.  
Queue → job waits for worker.  
Running → worker reads and writes data.  
Completion → job ends and shows Completed.  

After this, other background processes may still be updating identity and UI.

---

## Why Jobs Get “Stuck”

Usually three reasons:

Queued too long: no free workers.  
Running too long: network or connector blocked.  
Running slowly: source is huge or slow.

Killing and restarting blindly usually makes things worse by adding more jobs.

---

## Why This Matters

Without lifecycle understanding you will:
- Think nothing is happening when it’s just queued.
- Think something is broken when it’s just slow.
- Trust Completed when the real work isn’t visible yet.

Understanding this saves panic actions.

---

## If You See This → Do This

Queued too long → check how many jobs are running.  
Running too long → check logs and network.  
Completed but wrong data → check API and recompute, not just UI.

---

## How People Fail Here

They click Run many times.  
They schedule overlapping jobs.  
They kill jobs without understanding.  
They trust Completed as Correct.

---

## Proof Paths

UI: job list, timestamps, status.  
API: real job objects and states.  
Logs: what the worker is actually doing.

---

## What Must Not Happen

Do not run same source twice at once.  
Do not spam Run when queued.  
Do not assume Completed means finished everywhere.

---

## Safe Fixes

Overlapping jobs → fix schedules.  
Queue too long → stagger sources.  
Slow jobs → test smaller scope.

---

## Confidence Check

If you can answer these, you understand jobs:

What creates a job?  
Why can a job wait after you click Run?  
What does Completed really mean?  
Why are overlapping jobs dangerous?

---

### Navigation
⬅️ Previous: [Part 1 – Before Aggregation Can Run (Pre‑Flight)](./Part_1_Before_Aggregation_Can_Run_Pre_Flight.md)  
🏠 Home: [README – Aggregation Master Series](./README.md)  
➡️ Next: [Part 3 – Extraction Phase](./Part_3_Extraction_Phase_Reading_Data.md)
