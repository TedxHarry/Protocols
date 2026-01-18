# Part 6 – Correlation (Accounts → Identities) — Teaching Mastery Edition

[⬅️ Back to Home](../README.md)

---

## Why This Part Exists

Parts 3–5 decided:
- What data exists (Extraction),
- What it looks like (Normalization),
- What ISC remembers (Persistence).

Part 6 decides something more human:

**Who does this account belong to?**

Before correlation, we only have system records.  
After correlation, those records belong to real people.

If this step is wrong, the system doesn’t just break —  
it gives the **wrong person** the **right access**.

Keep this sentence in mind:

**Correlation is recognition.**

---

## Where This Fits in the End-to-End Flow

Trigger → Extract → Normalize → Persist → **Correlate** → Evaluate → Recompute → Publish

Persistence answers:
“Have I seen this account before?”

Correlation answers:
“Who is the human behind it?”

This is the bridge between data and people.

---

## The Mental Model

```
Account record
   → Look for a matching identity
     → Link, Create, or Leave Uncertain
```

Correlation is not about access.  
It is not about correctness of data.  
It is only about recognition.

---

## What ISC Is Really Doing

When an account reaches correlation, ISC is not asking:

- Should this person have access?
- Is this data valid?
- Is this user active?

It is asking only:

**Do I recognize the person behind this account?**

It checks one thing: the **match key**.

Then it decides:

- Exactly one identity matches → Link  
- No identity matches → Leave uncorrelated or create (if authoritative)  
- More than one matches → Mark ambiguous and refuse to guess  

This is identity logic, not business logic.

---

## Match Key: The Recognition Fingerprint

The match key is how ISC recognizes a person over time.

A good match key is:
- Unique (one person only)
- Stable (doesn’t change)
- Never reused

Strong examples:
- employeeId
- HR person ID

Weak examples:
- name
- department
- title
- sometimes email

If the fingerprint changes, the system stops recognizing the person.

---

## Guided Story: One Person Over Time

Source: HR (authoritative)  
Person: Bob  
Match key: employeeId

### First Run
HR sends Bob with employeeId = 2002  
No identity exists with 2002  
→ Create identity Bob, link account

### Second Run
HR sends Bob with employeeId = 2002, department changed  
ISC finds identity with 2002  
→ Link to same Bob

### Third Run
HR sends Bob with employeeId = 2002, email changed  
Match key unchanged  
→ Still Bob

Now change match key to email.

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

## Mini Checkpoint

Answer without looking:

- What is a match key really used for?  
- What happens if it changes?  

If your answer includes “recognition” and “forgetting,” you’re right.

---

## Uncorrelated: What It Really Means

Uncorrelated means:

“I have an account, but I don’t know who it belongs to.”

This can happen when:
- Identity doesn’t exist yet
- Match key is wrong
- Source data is bad

Uncorrelated is uncertainty, not failure.

Large numbers of uncorrelated accounts mean:
- Wrong match key
- Missing identities
- Dirty source data

---

## Ambiguous: Safer Than Guessing

Ambiguous means:

“More than one identity looks like a match.”

Example:
Two identities share the same email.  
Account arrives with that email.  
System refuses to guess.

Ambiguity is painful, but safe.  
Guessing would be dangerous.

---

## Authoritative vs Non-Authoritative

If the source is authoritative:
- Correlation may create new identities.

If the source is not authoritative:
- Correlation can only link to existing identities.

HR is usually authoritative.  
AD and apps usually are not.

This prevents random systems from inventing people.

---

## Interactive Scenario

HR is authoritative.  
Match key = employeeId.

Run 1: HR sends Alice(1001), Bob(2002)  
→ Two identities created.

Run 2: HR sends Alice(1001), Bob(2002) again  
→ Both linked correctly.

Run 3: Bob missing from HR  

Question:
Does correlation decide Bob left?

Pause. Think.

Answer:
No. Correlation only works on what exists.  
Persistence handles missing.  
Correlation only assigns ownership of accounts that appear.

---

## Why This Phase Is Dangerous

If correlation is wrong:

- One person may get another’s access
- Certifications approve the wrong human
- Audits become lies

This is where identity mistakes become security incidents.

---

## Debug Playbook

When something looks wrong, ask in this order:

1) What is the match key?  
2) Is it stable over time?  
3) Is it unique across people?  
4) Are there duplicate identity values?  
5) Are there ambiguous matches?  

Only after that, check roles or access.

---

## Visual Debug Flow

```
What is the match key?
   ↓
Is it stable?
   ↓
Is it unique?
   ↓
Any duplicates?
   ↓
Any ambiguous matches?
```

---

## Classic Failure Story

Company used email as match key.

Rebranding changed many emails.

Next run:
Old identities unmatched.  
New identities created.

People duplicated.  
Access exploded.  
Certifications broke.

System did exactly what it was told.  
Recognition was wrong.

---

## Proof Paths

To prove correlation health, check:

- Identity records and their match key values  
- Uncorrelated account list  
- Ambiguous match list  
- Duplicate values in identity data  

Truth is in recognition patterns, not access lists.

---

## What Must Not Happen

- Do not change match key casually  
- Do not use unstable attributes  
- Do not ignore ambiguity  
- Do not debug access before recognition  

---

## The One Sentence That Defines Mastery

Before you ask “Why is access wrong?”, ask:

**Is this account linked to the right person?**

---

## Mastery Check

Answer these without notes:

- What question is correlation really answering?  
- Why is match key recognition?  
- Why is uncorrelated uncertainty?  
- Why is ambiguous safer than guessing?  
- Why do disasters start here?  

---


### Navigation
⬅️ Previous: [Part 5 – Persistence](./Part_5_Write_and_Persistence_Story.md)  
🏠 Home: [README](./README.md)  
➡️ Next: [Part 7 – Identity Profile Evaluation](./Part_7_Identity_Profile_Evaluation_Engine.md)

