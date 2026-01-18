# Part 1 – Before Aggregation Can Run (Pre‑Flight) — Teaching Mastery Edition

[⬅️ Back to Home](../README.md)

---

## Why This Part Exists

Pre‑Flight is everything you do **before** you press Run.

Not to make things faster.  
To make things **predictable**.

A bad run usually doesn’t fail loudly.  
It finishes politely… and builds the wrong truth.

Think of cooking:

- Wrong ingredients → bad dish  
- Even with a perfect recipe

Pre‑Flight is checking ingredients before cooking.

---

## Where This Fits in the Big Engine

Pre‑Flight lives **before** the engine starts.

Master Flow:

Trigger → Extract → Normalize → Persist → Correlate → Evaluate → Recompute → Publish

Pre‑Flight decides whether pressing Trigger even makes sense.

If Pre‑Flight is wrong, the engine just amplifies the mistake.

Mental picture:

Pre‑Flight  
   ↓  
Trigger  
   ↓  
Extract → Normalize → Persist  
   ↓  
Accounts in ISC  
   ↓  
Correlation  
   ↓  
Identity Evaluation  
   ↓  
Access Recompute  
   ↓  
Publish (UI)

Pre‑Flight is what makes the chain reliable.

---

## The Mental Model

```
Good inputs → Predictable engine → Trustworthy results
Bad inputs  → Perfect engine     → Perfect lies
```

The engine is not your enemy.  
Bad setup is.

---

## Running Example (We Use This Everywhere)

Source: HR system (Workday‑like)  
People: Alice, Bob, Carol  
Accounts: HR_Alice, HR_Bob, HR_Carol  
Entitlements: Engineering, HR, Finance  

Goal:

When aggregation runs:
- ISC reads Alice, Bob, Carol from HR
- Accounts are clean
- They become correct identities later

Pre‑Flight decides whether this goal is even possible.

---

## The Pre‑Flight Journey

Instead of a checklist, think of this as a story:

1) What does this system represent?  
2) How will ISC reach it?  
3) Can ISC really read what it needs?  
4) What does the data look like?  
5) How will ISC remember accounts?  
6) Where will fields land?  
7) How will identity be built?  
8) Prove with a manual run  
9) Only then: schedule

Each step protects the next one.

---

## Step 1: What Does This Source Represent?

Every source must answer one question:

Does this system define **who a person is**,  
or does it mostly give them **access**?

If it defines the person → Authoritative  
Usually HR systems.

If it mostly gives access → Non‑authoritative  
Usually AD, Entra, apps.

In our example:
HR defines who Alice is.  
So HR must be authoritative.

If you mark this wrong, identity data will come from the wrong place and everything later will feel random.

---

## Step 2: How Will ISC Reach the Source?

Two choices:

- Direct connection → source is on the internet  
- VA (Virtual Appliance) → source is inside a private network  

In our example:
HR is SaaS, so direct connection works.

If this is wrong:
- Jobs may start
- But extraction will be empty or partial

Connectivity mistakes often look like “no data” bugs later.

---

## Step 3: Credentials and Permissions

The connector logs in as a real account.

That account must read:

- Accounts
- Entitlements/groups
- Memberships

In our example:
HR connector must read employees and departments.

A job can say Completed even if:
- It could not read groups
- It could not read memberships

Silent permission issues are very common.

---

## Step 4: Understand the Schema

Schema is the connector’s view of the source:

- What objects exist?
- What fields exist?
- What types are they?

Do not rush this screen.

This is where you choose the most dangerous field of all:

### Unique ID — The Memory Anchor

Unique ID tells ISC:

“This is the same account as last time.”

In our example:
We choose employeeId.

Good unique ID:
- Stable
- Truly unique
- Present for everyone

Bad unique ID:
- Email
- Display name
- Reusable usernames

Classic disaster:
Unique ID = email.  
People change email.  
ISC creates duplicates.

This is not a bug.  
This is memory being reset.

---

## Mini Checkpoint

Answer without looking:

What is Unique ID really used for?

If your answer includes “memory” or “recognition,” you’re right.

---

## Step 5: Map Accounts Carefully

Mapping decides where source fields land in ISC.

Example:

employeeId → accountAttribute.employeeId  
email → accountAttribute.email  
department → accountAttribute.department  

Two dangers here:

- Wrong destination field  
- Wrong type (text vs date, single vs multi)

Always use mapping preview.  
Preview is your first lie detector.

---

## Step 6: Prepare the Identity Profile

Accounts are not people.  
Identity profile logic builds the person.

It decides:

- Which source can set each identity field  
- Which source wins when they disagree  
- Whether a field is copied or derived  

In our example:
HR wins for department and email.

So even if AD says Alice is Sales, but HR says Engineering, identity follows HR.

If identity profile is wrong:
- Accounts look perfect
- Identity still looks wrong

---

## Step 7: Think About Scheduling (Slowly)

Do not start with schedules.

Start with:

- Manual run
- Fix problems
- Run again
- Prove stability

Then:

- Schedule once per day
- Avoid overlapping jobs

Scheduling bad logic only makes bad results arrive faster.

---

## Broken‑Flow Story

Scenario:

Job says Completed.  
Accounts look fine.  
Identity department is wrong.

Logic:

Accounts right + identity wrong  
→ Identity profile problem.

Proof order:

1) Source truth  
2) Account truth  
3) Correlation  
4) Identity truth  

Fix identity profile. Re‑evaluate identities.

Completed is not the same as correct.

---

## Fast Triage Patterns

- Duplicate accounts → Check unique ID  
- Empty fields → Check mapping and permissions  
- Identity wrong but accounts right → Check identity profile  
- Job never really extracts → Check network and credentials  

---

## Proof Paths

Use more than UI:

- UI → configuration view  
- API → stored reality  
- Logs → connection and read truth  

Never debug only from UI symptoms.

---

## What Must Not Happen

- Do not schedule before one clean manual run  
- Do not change schema during a run  
- Do not guess Unique ID  

---

## Safe Fixes

- Schema wrong → Fix and rediscover  
- Mapping wrong → Fix and preview  
- Identity profile wrong → Fix and re‑evaluate identities  

---

## Mastery Check

Answer these without notes:

1) What makes a source authoritative?  
2) Why is Unique ID dangerous?  
3) Why can identity be wrong when accounts are right?  
4) Why start with manual runs?  
5) If job completes but identity is wrong, which layer do you check first?  

---

### Navigation
⬅️ Previous: [Part 0 – Mental Model and Vocabulary](./Part_0_Mental_Model_and_Vocabulary.md)

🏠 Home: [README – Aggregation Master Series](./README.md)  

➡️ Next: [Part 2 – Triggers and Job Lifecycle](./Part_2_Triggers_and_Job_Lifecycle.md)
