
# Part 10 – Result Semantics (Story-Driven)

[⬅️ Back to Home](../README.md)

---

## Big Idea

Result labels are not truth.  
They are summaries of how far a worker ran.

Most people think:
Completed = correct  
Failed = broken

Reality:
A job can finish and still be wrong.  
A job can fail and still change data.

Result semantics teach you how to read the *signal* without being fooled by it.

---

## Where This Fits

Trigger → Extract → Normalize → Persist → Correlate → Evaluate → Recompute → Publish  
                                                                     ↑  
                                                             Result Labels

Results sit above the engine.  
They describe what the engine *reported*, not what actually happened in data.

---

## What ISC Is Thinking When It Reports a Result

ISC is not judging correctness.  
It is only asking:

“Did my worker reach the end of its planned steps?”

So it thinks like this:

- Reached the end → Completed  
- Some steps errored but worker continued → Partial / Warning  
- Worker stopped early due to error → Failed  

None of these mean “data is right.”  
They only mean “how far the worker ran.”

---

## A Simple Story

You run HR aggregation.

UI says: Completed.

You check Alice.  
Her role is still old.

What does Completed actually tell you?

Only this:
The worker reached the end of its execution path.

It does NOT tell you:
- Whether mapping was correct  
- Whether recompute finished  
- Whether downstream logic lagged  

So “Completed” answers the wrong question if you care about data.

---

## What “Completed” Really Means

Completed means:
The job did not crash before finishing its steps.

It does NOT guarantee:
- Every record was correct  
- Every page was read  
- Every transform worked  
- Every downstream job finished  

A job can complete even when:
- Some values were dropped silently  
- Some pages were skipped  
- Some transforms failed but were ignored  

So Completed means: finished running, not finished being right.

---

## What “Partial / Warning” Really Means

Partial or Warning means:
Some parts failed, but the job chose to continue.

This is dangerous because:
It looks successful at first glance.

Examples:
- Accounts succeeded, entitlements failed  
- Some pages failed, others worked  
- Some transforms errored, others ran  

Partial often hides data loss behind a green-looking status.

---

## What “Failed” Really Means

Failed means:
The job stopped early due to error.

But:
It may have already written some data before stopping.

So Failed does NOT mean:
“Nothing changed.”

It means:
“Not everything finished.”

Some damage may already be done.

---

## UI Summary vs Reality

UI shows:
- Status  
- Counts  
- Messages  

But this is a summary of a long story.

Logs and API show the real story:
- Which steps ran  
- Which records failed  
- Which values were dropped  

If UI and data disagree, trust data and logs, not the label.

---

## Running Example Revisited

Job: Completed  
Alice role: still old

Possible truths:
- Recompute still running  
- Recompute failed  
- Role rule doesn’t match identity  
- Mapping earlier dropped a field  

The result label alone cannot tell you which.

---

## Failure Story

Team saw “Completed” and relaxed.

But 30% of users lost department.

Root cause:
A transform silently dropped values due to type mismatch.  
Job still said Completed.

They trusted the label instead of checking data.

---

## How to Think When Results Confuse You

Do not ask:
“Did the job succeed?”

Ask:
“What data actually changed?”

Then walk like this:

1) Check real data in API/UI  
2) Check recompute / refresh status  
3) Read warning messages  
4) Read logs for silent drops  
5) Only then rerun anything  

---

## The Most Common Traps

- Treat Completed as Correct  
- Ignore warnings  
- Rerun jobs blindly  
- Never read logs  

These habits create bigger damage than the original problem.

---

## Mindset

Result labels are hints, not truth.

Never trust the word.  
Always trust the data.

The only real question is:

“What actually changed in the system?”

---

## You Understand This If You Can Answer

- What does Completed really mean?  
- Why is Partial more dangerous than Failed?  
- Why does Failed not mean “nothing changed”?  
- Why is UI never enough proof?  

---

### Navigation
⬅️ Previous: [Part 9 – Delta](./Part_9_Delta_State_and_Token_Lifecycle.md)  
🏠 Home: [README](./README.md)  
➡️ Next: [Part 11 – Verification and Validation](./Part_11_Verification_and_Validation.md)

