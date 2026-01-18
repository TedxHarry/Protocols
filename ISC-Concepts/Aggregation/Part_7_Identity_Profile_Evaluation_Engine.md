# Part 7 – Identity Profile Evaluation Engine — Teaching Mastery Edition

[⬅️ Back to Home](../README.md)

---

## Why This Part Exists

Parts before this decided:

- What data exists (Extraction),
- What it looks like (Normalization),
- What ISC remembers (Persistence),
- Who owns which account (Correlation).

Part 7 answers a deeper question:

**Given all the linked accounts, who is this person supposed to be?**

Correlation says: “These accounts belong to Alice.”  
Evaluation says: “So… who is Alice?”

Keep this sentence in mind:

**Evaluation is judgment. Not copying.**

---

## Where This Fits in the Engine

Trigger → Extract → Normalize → Persist → Correlate → **Evaluate** → Recompute → Publish

Persistence made data real.  
Correlation gave it an owner.  
Evaluation gives that owner meaning.

This is where identity becomes a story, not just records.

---

## The Mental Model

```
Linked accounts
   → Apply identity profile rules
     → Choose, derive, and resolve conflicts
       → Final identity attributes
```

Evaluation never reads raw sources.  
It only reads already-linked accounts and applies rules to them.

---

## What ISC Is Really Doing

During evaluation, ISC is not asking:

- What did the source say?
- Which value is newest?
- Which system is loudest?

It is asking:

**When sources disagree, who do I trust?**

For every identity attribute, ISC checks:

- Which sources are allowed to speak?
- In what order should they be trusted?
- Should the value be copied or calculated?

Evaluation is choosing. Not copying.

---

## The Identity Profile: The Rulebook

The Identity Profile is the rulebook that answers:

For each identity attribute:

- Which sources can supply it
- Which one wins if many exist
- Whether it is copied or derived

It does not read raw data.  
It reads linked account attributes and decides what becomes “truth.”

If the rulebook is wrong, identity will be wrong — even when accounts are perfect.

---

## Guided Story: One Person, Two Systems

Sources:
- HR (authoritative)
- AD (non-authoritative)

Alice has two linked accounts:

- HR_Alice
- AD_Alice

HR says:
- department = Engineering
- email = alice@company.com

AD says:
- department = Sales
- email = alice@corp.local

Who is Alice?

Evaluation decides.

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

- If HR has a value, use it.
- Only if HR is empty, try AD.

So Alice becomes:

- department = Engineering  
- email = alice@company.com  

AD is ignored — not because it is wrong,  
but because it is less trusted.

---

## What Happens When Data Changes

Next month, HR removes Alice’s department.

Evaluation logic:

- HR has no value.
- Try AD.
- AD says Sales.

So identity.department becomes Sales.

Nothing broke.  
The rulebook worked exactly as written.

---

## Derived Fields: Identity That Is Calculated

Some fields are not copied at all.

Examples:

- displayName = firstName + lastName
- managerEmail = manager.identity.email
- isManager = true if has reports

These fields are created using logic.

If the logic is wrong, identity becomes logically wrong —  
even if all source data is perfect.

---

## Lifecycle State: The Most Powerful Field

Lifecycle answers:

**Where is this person in their journey?**

Examples:
- Joiner
- Active
- Leave
- Terminated

Often based on HR fields:
- Status
- Hire date
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
- During identity refresh/recompute

It may not run instantly.

So you might see:

Account changed → Identity still old → Later identity updates

This delay often tricks people into thinking something is broken.

---

## Interactive Scenario

Profile rules:

- department: HR first, then AD
- email: HR only
- displayName: firstName + lastName
- lifecycle: from HR status

Accounts linked:
- HR_Alice: department = Engineering, status = Active
- AD_Alice: department = Sales

Question:
What does identity look like?

Pause. Think.

Answer:

- department = Engineering  
- email = from HR  
- displayName = derived  
- lifecycle = Active  

AD’s department is ignored by design.

---

## Why Identity Looks Wrong When Accounts Look Right

This is the classic trap.

You check the account — it looks correct.  
But identity shows something else.

This usually means:

- Wrong precedence
- Wrong source allowed
- Bad derived logic
- Lifecycle overriding behavior

The account is not wrong.  
The rulebook is.

---

## Debug Playbook

When identity looks wrong, debug in this order:

1) Which sources feed this field?  
2) What is the precedence order?  
3) Are higher-priority sources empty?  
4) Is this field derived?  
5) Is lifecycle influencing behavior?  

Only after this, look at roles or access.

---

## Visual Debug Flow

```
Which sources feed the field?
   ↓
What is precedence order?
   ↓
Is top source empty?
   ↓
Is field derived?
   ↓
Is lifecycle involved?
```

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

## Proof Paths

To validate evaluation:

- Check identity profile rules  
- Check linked account values  
- Check precedence order  
- Check derived logic  
- Check lifecycle rules  

Truth lives in the rulebook, not the source alone.

---

## What Must Not Happen

- Do not change precedence casually  
- Do not ignore empty higher-priority sources  
- Do not write complex derived logic blindly  
- Do not debug access before judgment  

---

## The One Sentence That Defines Mastery

Before you ask “Why is access wrong?”, ask:

**Who does ISC think this person is?**

---

## Mastery Check

Answer these without notes:

- What question is evaluation really answering?  
- Why is it about choosing, not copying?  
- Why can identity be wrong when accounts are right?  
- Why is lifecycle the most dangerous field?  
- Why do mystery bugs live here?  

---

### Navigation
⬅️ Previous: [Part 6 – Correlation](./Part_6_Correlation_Accounts_to_Identities.md)  
🏠 Home: [README](./README.md)  
➡️ Next: [Part 8 – Identity Refresh and Recompute](./Part_8_Identity_Refresh_and_Recompute.md)

