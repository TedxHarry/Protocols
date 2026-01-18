
# Part 7 – Identity Profile Evaluation Engine (Story-Driven)

[⬅️ Back to Home](../README.md)

---

## Big Idea

Correlation decides **who owns an account.**  
Identity Profile Evaluation decides **who that person actually is.**

After correlation, ISC knows:
“These accounts belong to Alice.”

But it still does not know:
- What is Alice’s department?
- Which email is real?
- Who is her manager?
- What lifecycle stage is she in?

Identity Profile Evaluation answers:

“Given all the accounts linked to this person,  
who is this person supposed to be?”

This is where identity becomes a story, not just data.

---

## Where This Fits in the Engine

Trigger → Extract → Normalize → Persist → Correlate → **Evaluate** → Recompute → Publish

Persistence made data real.  
Correlation gave it an owner.  
Evaluation gives that owner meaning.

---

## What ISC Is Thinking During Evaluation

ISC is not asking:
“What did the source say?”

It is asking:
“When sources disagree, who do I trust?”

For every identity field, it thinks:

- Which sources are allowed to speak?
- In what order should I trust them?
- Should I copy or calculate this field?

Evaluation is not copying.  
Evaluation is choosing.

---

## Identity Profile: The Rulebook

The Identity Profile is the rulebook that answers:

For each identity attribute:
- Which sources can supply it
- Which one wins if there are many
- Whether it is copied or derived

It never reads raw data.  
It only reads already-linked accounts.

If the rulebook is wrong, the identity will be wrong even when accounts are perfect.

---

## A Slow Story: One Person, Two Systems

Sources:
- HR (authoritative)
- AD (non-authoritative)

Alice has two accounts:
- HR_Alice
- AD_Alice

HR says:
- department = Engineering
- email = alice@company.com

AD says:
- department = Sales
- email = alice@corp.local

So who is Alice?

The identity profile decides.

---

## Precedence: Trust Order

For each field, you define a trust order.

Example:

department:
1) HR  
2) AD  

email:
1) HR  

This means:

If HR has department, use it.  
Only if HR is empty, try AD.

So Alice becomes:
- department = Engineering  
- email = alice@company.com  

AD is ignored, not because it is wrong,  
but because it is less trusted.

---

## What Happens When Data Changes

Next month, HR removes Alice’s department.

Now evaluation thinks:

HR has no value.  
Try AD.  
AD says Sales.  
So identity.department becomes Sales.

Nothing “broke.”  
The rulebook worked exactly as written.

---

## Derived Fields: Identity That Is Calculated

Some fields are not copied at all.

Examples:
- displayName = firstName + lastName
- managerEmail = manager.identity.email
- isManager = true if has reports

These fields are built using logic, not copying.

If logic is wrong, identity is logically wrong, even if source data is fine.

---

## Lifecycle State: The Most Powerful Field

Lifecycle answers:
Where is this person in their journey?

Examples:
- Joiner
- Active
- Leave
- Terminated

Often based on HR data:
- Hire date
- Status
- Termination date

Lifecycle is dangerous because it drives:
- Provisioning
- Deprovisioning
- Certifications
- Policies

If lifecycle is wrong, automation becomes wrong.

---

## When Evaluation Runs

Evaluation runs:
- After correlation
- After account updates
- During identity refresh

It may not run instantly.  
So you may see:

Account changed → Identity still old → Later identity updates.

This delay often confuses people into thinking something is broken.

---

## Full Story Example

Accounts linked:
- HR_Alice
- AD_Alice

Profile rules:
- department: HR first, then AD
- email: HR
- displayName: firstName + lastName
- lifecycle: based on HR status

Evaluation result:
- department = Engineering
- email = alice@company.com
- displayName = Alice Smith
- lifecycle = Active

Even though AD said Sales, it was ignored by design.

---

## Why Identity Looks Wrong When Accounts Look Right

This is the classic trap.

You check the account.  
It has the right value.

But identity shows something else.

This usually means:
- Wrong precedence
- Wrong source allowed
- Bad derived logic
- Lifecycle overriding

The account is not wrong.  
The rulebook is.

---

## How to Think When It Breaks

Do not ask:
“Why is access wrong?”

Ask:
“Who does ISC think this person is?”

Then walk in this order:

1) Which sources feed this field?  
2) What is the precedence order?  
3) Are any sources empty?  
4) Is this field derived?  
5) Is lifecycle influencing behavior?  

Only after this, look at roles or access.

---

## Classic Failure Story

HR had correct department.  
AD had old department.

Profile trusted AD more than HR.

Identity showed wrong department.  
Roles assigned wrongly.  
Provisioning broke.

Nothing wrong with extraction.  
Nothing wrong with correlation.  
The rulebook was wrong.

---

## How to Know You Understand This Phase

You truly understand evaluation if you can answer:

- What question is evaluation really answering?  
- Why is it about choosing, not copying?  
- Why can identity be wrong when accounts are right?  
- Why is lifecycle the most dangerous field?  
- Why do most “mystery” bugs live here?  

---

## Mindset

Identity Profile Evaluation is not data flow.  
It is judgment.

Once judgment is wrong,  
everything built on it is wrong.

---

### Navigation
⬅️ Previous: [Part 6 – Correlation (Accounts → Identities)](../Part_6_Correlation_Story.md)  
🏠 Home: [README – Aggregation Master Series](../README.md)  
➡️ Next: [Part 8 – Identity Refresh and Recompute](../Part_8_Identity_Refresh_and_Recompute.md)
