# Part 10 – Result Semantics (Teaching Mastery Edition)

[⬅️ Back to Home](../README.md)

---

## Why This Part Exists

This part answers one sharp question:

**What does the system think happened?**

Result labels are not truth.  
They are summaries of how far a worker ran.

If you confuse labels with truth, you will relax when you should worry —  
and panic when nothing is actually wrong.

Keep this sentence in mind:

**Results describe execution, not correctness.**

---

## Where This Fits in the Engine

Trigger → Extract → Normalize → Persist → Correlate → Evaluate → Recompute → Publish  
                                                                     ↑  
                                                             Result Semantics

Results sit above the engine.  
They describe what the worker *reported*, not what the data *became*.

---

## The Mental Model

```
Worker runs steps
   → Worker reports how far it went
     → UI shows a label
```

Result labels answer:

“How far did the worker run?”

They do NOT answer:

“Is the data right?”

---

## What the System Is Really Asking

When ISC reports a result, it is not judging truth.

It is asking only:

**Did my worker reach the end of its planned steps?**

So the logic is:

- Reached end → Completed  
- Some steps errored but worker continued → Warning / Partial  
- Worker stopped early → Failed  

None of these mean “data is correct.”  
They only mean “how far execution went.”

---

## Guided Story

You run HR aggregation.

UI says: Completed.

You check Alice.  
Her department is wrong.

What does Completed tell you?

Only this:
The worker reached the end of its execution path.

It does NOT tell you:
- Mapping was right  
- Identity updated  
- Recompute finished  
- Indexing completed  

Completed answered the wrong question for your problem.

---

## What “Completed” Really Means

Completed means:
- Worker finished its planned path  
- No fatal error stopped it  

It does NOT guarantee:
- All records were correct  
- All pages were read  
- All transforms worked  
- All downstream engines finished  

A job can complete even when:
- Some values were dropped  
- Some pages were skipped  
- Some transforms returned null  

So Completed means: finished running, not finished being right.

---

## What “Warning / Partial” Really Means

Warning or Partial means:
- Some steps failed  
- Worker chose to continue  

This is dangerous because:
It looks mostly green.

Examples:
- Accounts succeeded, entitlements failed  
- Some pages failed, others worked  
- Some transforms errored silently  

Partial often hides data loss behind a “mostly okay” label.

---

## What “Failed” Really Means

Failed means:
- Worker stopped early due to error  

But:
It may have already written data before stopping.

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

Logs and APIs show the real story:
- Which steps ran  
- Which records failed  
- Which values were dropped  

If UI and data disagree, trust data and logs — not the label.

---

## Interactive Pause

Scenario:

Job = Completed  
Alice role = old

Question:
What should you suspect first?

Pause. Think.

Answer:
- Recompute may not have run  
- Recompute may have failed  
- Rules may not match identity  
- Mapping earlier may have dropped a field  

Not: “Aggregation is broken.”

---

## Failure Story

Team saw “Completed” and relaxed.

But 30% of users lost department.

Root cause:
Transform dropped values due to type mismatch.  
Worker still reached the end.

Job said Completed.  
Data was wrong.

They trusted the label instead of the data.

---

## Illusions This Phase Creates

- Completed means correct  
- Failed means nothing changed  
- Partial is mostly safe  
- UI tells the full story  

All four can be false.

---

## Traps That Fool Smart People

- Treating Completed as success  
- Ignoring warnings  
- Rerunning jobs blindly  
- Never reading logs  

These habits create bigger disasters than the original bug.

---

## Debug Playbook

When results confuse you, debug like this:

1) What data actually changed?  
2) Did identity update?  
3) Did recompute run?  
4) Are warnings hiding failures?  
5) What do logs say about drops or skips?  

Only after this, decide whether to rerun anything.

---

## Visual Debug Flow

```
What data changed?
   ↓
Did identity update?
   ↓
Did recompute run?
   ↓
Any warnings?
   ↓
Read logs for silent drops
```

---

## Proof Paths

To prove result truth:

- API data (accounts, identities, access)  
- Job messages and warnings  
- Worker logs  
- Recompute job history  

Truth lives in data, not labels.

---

## What Must Not Happen

- Do not trust Completed blindly  
- Do not ignore warnings  
- Do not rerun without proof  
- Do not debug only from UI  

---

## Safe Fixes

- Data wrong → find which phase lied  
- Partial → identify exactly what failed  
- Failed → check what already changed  
- UI wrong → wait or check indexing  

---

## The One Sentence That Defines Mastery

Before you ask “Did the job succeed?”, ask:

**What actually changed in the system?**

---

## Mastery Check

Answer without notes:

- What do result labels really describe?  
- Why can Completed still mean wrong data?  
- Why is Partial more dangerous than Failed?  
- Why does Failed not mean “nothing changed”?  
- What is your debug order when labels mislead you?  

---

### Navigation
⬅️ Previous: [Part 9 – Delta](./Part_9_Delta_State_and_Token_Lifecycle.md)  
🏠 Home: [README](./README.md)  
➡️ Next: [Part 11 – Verification and Validation](./Part_11_Verification_and_Validation.md)

