
# Part 1 – Before Aggregation Can Run (Pre‑Flight)

[⬅️ Back to Home](../README.md)

---

## Purpose
Pre‑Flight is the thinking and setup that happens before you ever press Run.  
It is not about speed. It is about certainty.

When pre‑flight is weak, aggregation does not always fail loudly. It often fails quietly, and you only notice later when identities, access, or certifications look wrong.

Think of this like cooking. If ingredients are missing or mislabeled, the dish will taste wrong even if you follow the recipe perfectly.

---

## Where This Fits in the Master Flow
This part lives before the engine even starts.

Master Flow:
Trigger → Extract → Normalize → Persist → Correlate → Evaluate → Recompute → Publish

Pre‑Flight decides whether pressing Trigger even makes sense. If the setup is wrong, the engine will only amplify the mistakes.

Mental picture:

Pre‑Flight  
   ↓  
Trigger  
   ↓  
Extract / Normalize / Persist  
   ↓  
Accounts stored in ISC  
   ↓  
Correlation  
   ↓  
Identity profile evaluation  
   ↓  
Access recompute  
   ↓  
Publish (UI catches up)

Pre‑Flight is everything that makes the rest of the chain predictable.

---

## Mini‑Glossary (Sharp and Practical)

| Term | What it really means |
|------|----------------------|
| Authoritative source | The system ISC should trust for who a person is |
| Non‑authoritative source | A system that mostly adds access, not identity truth |
| Direct connection | ISC can reach the source over the internet |
| VA (Virtual Appliance) | A bridge when the source is inside a private network |
| Schema | Fields and objects the connector can see |
| Mapping | Where each source field lands inside ISC |
| Unique ID | The anchor that tells ISC “same account as last time” |
| Identity profile | Logic that builds the person from linked accounts |

---

## Wrong Thinking vs Right Thinking

Wrong: “If the job runs, the data is correct.”  
Right: A job can run perfectly and still build wrong identities.

Wrong: “Unique ID is just a required field.”  
Right: Unique ID is the anchor. If it’s wrong, everything becomes unstable.

Wrong: “Identity will match whatever the account shows.”  
Right: Accounts can be right and identities can still be wrong because identity logic chooses winners.

Wrong: “Let’s schedule first.”  
Right: Scheduling bad logic only makes bad results arrive faster.

---

## Running Example (We Use This Story Everywhere)

Source: HR system (Workday‑like)  
People: Alice, Bob, Carol  
Accounts: HR_Alice, HR_Bob, HR_Carol  
Entitlements: Engineering, HR, Finance  

Goal: When aggregation runs, ISC should read Alice, Bob, and Carol from HR and prepare their accounts so they become clean, correct identities.

---

## The Pre‑Flight Checklist (One Page)

1) Decide what this source represents  
2) Decide how ISC will reach it  
3) Verify credentials and permissions  
4) Understand schema  
5) Choose a safe unique ID  
6) Map fields carefully  
7) Prepare identity profile logic  
8) Manual run and prove  
9) Only then: schedule  

---

## Step 1: Decide What This Source Represents

Every source must answer one question:  
Does this system define who a person is, or does it only give them access?

If it defines who the person is → Authoritative  
HR systems usually live here.

If it mostly gives access → Non‑authoritative  
AD, Entra, and applications usually live here.

In our example, HR is authoritative.  
So if Alice’s department changes in HR, ISC must trust HR more than any other system.

If you mark this wrong, identity data will come from the wrong place and everything will feel random later.

---

## Step 2: Decide How ISC Will Reach the Source

If the source is on the internet → Direct connection.  
If the source is inside a private network → Use a VA.

In our example, HR is SaaS, so we use direct connection.

If this choice is wrong, jobs may start but never really extract anything.

---

## Step 3: Prepare Credentials and Permissions

The connector logs in as a real account.

That account must read:
- Accounts
- Entitlements or groups
- Memberships

In our example, the HR connector must read employees and their departments.

A job can complete while silently skipping what it cannot read.  
Missing access often starts here.

---

## Step 4: Understand the Schema

Schema is the connector’s view of the source:
What objects exist, what fields exist, and their types.

Do not rush this screen.

### Choosing the Unique ID (High‑stakes decision)

This field tells ISC:
“This is the same account as last time.”

In our example, we choose employeeId.

Good unique ID:
- Stable
- Truly unique
- Present for everyone

Bad unique ID:
- Email
- Display name
- Reusable usernames

Classic failure:  
Unique ID = email. People change email. ISC creates duplicates.

---

## Step 5: Map Accounts Carefully

Mapping decides where raw source data lands in ISC.

Example:
employeeId → accountAttribute.employeeId  
email → accountAttribute.email  
department → accountAttribute.department  

Types must match. Multi‑value must not be forced into single.

Always use mapping preview.  
Preview is your first lie detector.

---

## Step 6: Prepare the Identity Profile

Accounts are not identities.

Identity profile logic decides:
- Which source sets which field
- What happens when sources disagree

In our example, HR wins for department and email.

So even if AD says Alice is Sales, but HR says Engineering, identity follows HR.

If this logic is wrong, accounts look perfect and identities still look wrong.

---

## Step 7: Think About Scheduling (Slowly)

Start with manual runs.  
Prove one clean run.

Then schedule once per day.  
Avoid overlapping jobs.

Scheduling bad logic only makes bad results arrive faster.

---

## Broken‑Flow Walkthrough

Scenario:
Job says Completed.  
Accounts look fine.  
Identity department is wrong.

Logic:
Accounts right + identity wrong = identity profile problem.

Proof order:
1) Source truth  
2) Account truth  
3) Correlation  
4) Identity truth  

Fix identity profile. Re‑evaluate identities.

Completed is not the same as correct.

---

## Fast Triage

Duplicate accounts → Check unique ID  
Empty fields → Check mapping and permissions  
Identity wrong but accounts right → Check identity profile  
Job never really starts → Check network and credentials  

---

## Proof Paths

UI: shows configuration  
API: shows what ISC stored  
Logs: show connection truth  

Do not debug only from UI symptoms.

---

## What Must Not Happen

Do not schedule before a good manual run.  
Do not change schema during a run.  
Do not guess unique ID.

---

## Safe Fixes

Schema wrong → Fix and rediscover  
Mapping wrong → Fix and preview  
Identity profile wrong → Fix and re‑evaluate identities  

---

## Confidence Check

If you can answer these, you’re ready:
1) What makes a source authoritative?
2) Why is unique ID critical?
3) Why can identity be wrong if accounts are right?
4) Why start with manual runs?
5) If job completes but identity is wrong, which layer do you check first?

---

### Navigation
⬅️ Previous: [Part 0 – Mental Model and Vocabulary](./Part_0_Mental_Model_and_Vocabulary.md)

🏠 Home: [README – Aggregation Master Series](./README.md)  

➡️ Next: [Part 2 – Triggers and Job Lifecycle](./Part_2_Triggers_and_Job_Lifecycle.md)
