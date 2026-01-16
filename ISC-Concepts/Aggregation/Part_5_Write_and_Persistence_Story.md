
# Part 5 – Write and Persistence

[⬅️ Back to Home](../README.md)

---

## Big Idea

This is the moment where data stops being “temporary” and becomes “real.”

Before this step:
Data was read.  
Data was shaped.  
Data could still be thrown away.

After this step:
Data becomes truth inside ISC.  
Every later decision depends on what is written here.

Think of this as memory formation.  
Once ISC remembers something, every future run will argue with that memory.

---

## Where This Fits in the Engine

Trigger → Extract → Normalize → **Persist** → Correlate → Evaluate → Recompute → Publish

Persistence is where the system answers one deep question:

“Do I already know this thing, or am I seeing it for the first time?”

Everything else flows from that answer.

---

## What ISC Is Thinking During Persistence

When data reaches this phase, ISC is not asking:
“Is this data correct?”

It is asking:
“Have I seen this identity before?”

It looks at one thing only: the Unique ID.

Then it thinks like this:

- If I have never seen this Unique ID → Create  
- If I have seen this Unique ID → Update  
- If I used to see this Unique ID but don’t anymore → Missing-from-feed  

This is not business logic.  
This is memory logic.

---

## Unique ID: The Memory Key

Unique ID is not just a field.  
It is how ISC remembers.

If this is stable:
ISC remembers people across time.

If this changes:
ISC forgets and starts over.

Example:

Run 1:
Alice has employeeId = 1001  
ISC stores account with key 1001.

Run 2:
Alice still has employeeId = 1001  
ISC says: I remember you → Update.

Run 3:
Unique ID changed to email.  
Alice’s email changed last week.

ISC now sees a “new” key.  
It says: I don’t remember you → Create.

Old Alice still exists.  
New Alice is born.  
Duplicates appear.

This is not a bug.  
This is memory being reset.

---

## A Slow Walk Through One Person

Source: HR  
Person: Bob  
Unique ID: employeeId

### First Run
HR sends: employeeId = 2002  
ISC has never seen 2002 → Create Bob.

### Second Run
HR sends: employeeId = 2002, department changed  
ISC recognizes 2002 → Update Bob.

### Third Run
HR does not send Bob at all  
ISC only knows one thing:  
Bob was not seen this time.

ISC does NOT know:
- Did Bob leave?
- Was Bob filtered?
- Did extraction fail?

It only knows: Bob is missing-from-feed.

What happens next depends on your rules.

---

## Missing-from-Feed: Uncertainty, Not Truth

Missing-from-feed does not mean:
“Person left.”

It means:
“I did not see this record in this run.”

Reasons can include:
- Person really left
- Source filtered them
- Connector lost permission
- Pagination broke
- Delta marker skipped data

So persistence asks:
“What should I do when I am unsure?”

Your answer can be:
- Disable
- Delete
- Ignore

This is a philosophy choice, not just a setting.

---

## Disable vs Delete: Two Worldviews

Disable says:
“I don’t see you now, but I remember you.”  
History stays. Audits stay. Context stays.

Delete says:
“I will forget you completely.”  
History may vanish. Audits lose meaning.

Most enterprises choose disable because memory is safer than amnesia.

---

## State Fields: The System’s Diary

Every account keeps memory of decisions:

- When it was created  
- When it was last seen  
- When it was last changed  
- Whether it is disabled  

These fields are not noise.  
They are the diary of persistence.

When something feels wrong, these fields tell you what ISC believed at each run.

---

## Full Story Example

Run 1:
Alice, Bob exist → Created

Run 2:
Alice, Bob exist with changes → Updated

Run 3:
Alice exists, Bob missing → Missing-from-feed  
Rule: Disable when missing

Result:
Alice updated  
Bob disabled

ISC now believes:
Alice is active.  
Bob exists but inactive.

That belief will guide all future logic.

---

## Why This Phase Is Dangerous

Mistakes here do not look small.

They look like:
- Duplicates everywhere
- Leavers still active
- People disappearing
- History becoming confusing

Because once memory is wrong, every future run argues with a lie.

---

## How to Think When It Breaks

Do not start with:
“Why is access wrong?”

Start with:
“What does ISC think is real?”

Ask in this order:

1) What is the Unique ID?  
2) Did it change?  
3) What does missing-from-feed do?  
4) What do state fields say happened?  

Only after that, look at correlation or identity logic.

---

## Classic Failure Story

Company changed unique ID from employeeId to email.

Many people had changed emails.

Next run:
ISC forgot everyone.  
Created everyone again.

Old people still existed.  
New people were born.

Security team panicked.  
But persistence just did what it was told.

Memory was erased. Then rebuilt wrongly.

---

## How to Learn This Phase

If you can answer these, you truly understand persistence:

- What question is ISC really answering here?  
- Why is unique ID actually “memory”?  
- Why is missing-from-feed uncertainty, not truth?  
- Why is disable safer than delete?  
- Why do most disasters start here?  

---

## Mindset

Persistence is not storage.  
It is belief.

Once ISC believes something,  
everything else follows that belief.

---

### Navigation
⬅️ Previous: [Part 4 – Normalization and Mapping](./Part_4_Normalization_and_Mapping.md)  
🏠 Home: [README – Aggregation Master Series](./README.md)  
➡️ Next: [Part 6 – Correlation (Accounts → Identities)](./Part_6_Correlation_Accounts_to_Identities.md)
