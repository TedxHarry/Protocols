
# Part 6 – Correlation (Accounts → Identities) — Story-Driven

[⬅️ Back to Home](../README.md)

---

## Big Idea

Correlation is the moment where accounts stop being “just records” and become “someone.”

Before this step:
We only have accounts from systems.

After this step:
Those accounts belong to real people inside ISC.

Correlation does not give access.  
Correlation does not change data.  
Correlation only answers one dangerous question:

“Who does this account belong to?”

If this answer is wrong, everything that follows becomes dangerous.

---

## Where This Fits in the Engine

Trigger → Extract → Normalize → Persist → **Correlate** → Evaluate → Recompute → Publish

Persistence decided what exists.  
Correlation decides who owns it.

This is the bridge between data and people.

---

## What ISC Is Thinking During Correlation

When an account arrives here, ISC is thinking:

“Do I already know the person behind this account?”

It looks at one thing: the match key.

Then it reasons like this:

- If exactly one identity matches → Link  
- If no identity matches → Leave uncorrelated or create (if authoritative)  
- If more than one identity matches → Mark ambiguous and refuse to guess  

This is not business logic.  
This is identity logic.

---

## Match Key: The Identity Fingerprint

The match key is how ISC recognizes a person.

A good match key is:
- Unique
- Stable
- Never reused

Examples of strong keys:
- employeeId
- HR person ID

Examples of weak keys:
- name
- department
- title
- sometimes email

If the fingerprint changes, the system stops recognizing the person.

---

## A Slow Story: One Person Over Time

Source: HR (authoritative)  
Person: Bob  
Match key: employeeId

### First Run
HR sends Bob with employeeId = 2002  
ISC finds no identity with 2002  
→ Creates Identity Bob and links account

### Second Run
HR sends Bob with employeeId = 2002, department changed  
ISC finds identity with 2002  
→ Links account to same Bob

### Third Run
HR sends Bob with employeeId = 2002, email changed  
ISC still matches on 2002  
→ Identity remains Bob

Now change the match key to email.

Next run:
Bob’s email changed last week.  
ISC looks for identity with new email.  
Finds none.  
→ Creates new identity.

Old Bob still exists.  
New Bob is born.

This is not a bug.  
This is broken recognition.

---

## What “Uncorrelated” Really Means

Uncorrelated means:
“The system has an account, but does not know who it belongs to.”

This might be okay temporarily, but large numbers mean:
- Wrong match key
- Identity not created yet
- Bad data in source

Uncorrelated is uncertainty.

---

## What “Ambiguous” Really Means

Ambiguous means:
“More than one identity looks like a match.”

ISC refuses to guess.

Example:
Two identities share same email.  
Account arrives with that email.  
System cannot decide.

Ambiguity is safer than guessing, but it blocks automation until fixed.

---

## Authoritative vs Non-Authoritative

If the source is authoritative:
- Correlation may create new identities.

If the source is not authoritative:
- Correlation only links to existing identities.

HR is usually authoritative.  
AD and apps usually are not.

This is how ISC avoids creating people from random systems.

---

## Full Story Example

HR sends:
Alice (1001), Bob (2002), Carol (3003)

First run:
No identities exist.  
HR is authoritative.  
→ Three identities created.

Second run:
All three still exist.  
→ All three accounts linked again.

Third run:
Bob missing from HR.  
→ Persistence handles missing.  
Correlation only works with what exists.

Correlation never decides leavers.  
It only decides ownership of accounts that exist.

---

## Why This Phase Is So Risky

If correlation is wrong:
- One person may get another person’s access
- Certifications approve the wrong human
- Audits become meaningless

This is where identity mistakes become security incidents.

---

## How to Think When It Breaks

Do not start with:
“Why is access wrong?”

Start with:
“Is this account linked to the right person?”

Ask in this order:

1) What is the match key?  
2) Is it stable over time?  
3) Is it unique across people?  
4) Are there duplicates in identity data?  
5) Are there ambiguous matches?  

Only after that, look at roles or access.

---

## Classic Failure Story

Company used email as match key.

Many people changed email during rebranding.

Next run:
Old identities unmatched.  
New identities created.  
People duplicated.

Access exploded.  
Certifications broke.

System did exactly what it was told.  
Recognition was wrong.

---

## How to Learn Correlation

If you can answer these, you understand correlation:

- What question is ISC really answering here?  
- Why is match key actually “recognition”?  
- Why is uncorrelated uncertainty, not error?  
- Why is ambiguous safer than guessing?  
- Why do security disasters start here?  

---

## Mindset

Correlation is not logic.  
It is recognition.

Once recognition is wrong,  
everything built on top is wrong.

---

### Navigation
⬅️ Previous: [Part 5 – Persistence](./Part_5_Write_and_Persistence_Story.md)  
🏠 Home: [README](./README.md)  
➡️ Next: [Part 7 – Identity Profile Evaluation](./Part_7_Identity_Profile_Evaluation_Engine.md)

