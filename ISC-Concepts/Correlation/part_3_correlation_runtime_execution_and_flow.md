# Part 3 – Correlation Runtime Execution and Data Flow

> **Purpose of this part**  
This chapter walks through *exactly what happens at runtime* when correlation executes.
No configuration theory, no UI focus.
Just **what ISC does, in what order, and why outcomes look the way they do**.

By the end of this part, you should:
- trace correlation step by step for a single account
- understand why re-aggregation often changes nothing
- know precisely where correlation succeeds, stops, or fails

---

## 3.1 Correlation only runs during aggregation

Correlation **does not run continuously**.
It runs **only when aggregation runs**.

This means:
- no aggregation, no correlation
- no reprocessing, no reevaluation

If aggregation does not touch an account, correlation does not revisit it.

---

## 3.2 Think one account at a time

ISC correlates **one account at a time**.

Each account goes through the same decision tunnel independently.

Never debug correlation globally.
Always debug **one account’s journey**.

---

## 3.3 The full runtime flow (high level)

For each account, ISC follows this sequence:

1. pull account from source  
2. normalize attributes via schema  
3. load correlation configuration  
4. evaluate criteria in order  
5. assign or leave uncorrelated  
6. persist the result  

Nothing else happens.

---

## 3.4 Step 1 – Account retrieval

Aggregation begins by pulling raw account data from the source.

At this moment:
- ISC sees only source-side values
- identity data has not been consulted yet

If the account never arrives:
- correlation never happens

---

## 3.5 Step 2 – Attribute normalization

Raw source data is normalized into ISC account attributes.

This step uses:
- source schema
- attribute mappings
- transforms (if any)

If the value is not present **after normalization**, correlation will never see it.

---

## 3.6 Step 3 – Load correlation configuration

ISC loads the correlation configuration **for this specific source**.

This includes:
- ordered criteria list
- source-specific correlation behavior

No identity matching has happened yet.

---

## 3.7 Step 4 – Identity lookup and matching

For each criteria, in order:

1. take the account attribute value  
2. search identities for a matching identity attribute value  
3. if a match is found, stop evaluation  
4. if no match, continue to the next criteria  

Rules:
- equality comparison
- stop on first match
- no backtracking

---

## 3.8 What “match found” really means

A match does **not** mean:
- best match
- newest identity
- authoritative identity

It means:
- value equality
- searchability at runtime
- identity exists at that moment

Uniqueness matters.

---

## 3.9 Step 5 – Assignment decision

If a match exists:
- account is assigned to the identity
- ownership is recorded

If no match exists:
- account remains uncorrelated

There is no retry later in the same run.

---

## 3.10 Step 6 – Persistence and memory

Correlation results are persisted.

Once correlated:
- ISC remembers the link
- future runs may skip reevaluation
- rule changes alone may not affect existing links

Correlation is not stateless.

---

## 3.11 Why re-aggregation often changes nothing

Common reason:
- the account was not reprocessed
- optimization skipped unchanged accounts
- correlation state was preserved

Correlation changes only when accounts are re-evaluated.

---

## 3.12 Delta aggregation impact

Delta aggregation processes **only changed accounts**.

Unchanged accounts:
- skip correlation
- keep previous assignment

Full aggregation or forced reprocessing is required for reevaluation.

---

## 3.13 Identity-side changes do not trigger correlation

Updating identity attributes does **not** trigger correlation.

Correlation runs on account aggregation, not identity change.

---

## 3.14 Visualizing the flow

Account arrives → attributes mapped → rules evaluated → stop or pass → result stored

ISC does not rewind this automatically.

---

## 3.15 How to debug runtime flow correctly

Capture:
1. raw account value  
2. normalized account value  
3. identity attribute values at runtime  
4. criteria order  
5. aggregation type  

Without runtime context, debugging fails.

---

## 3.16 What this part prepares you for

After this part, you should:
- understand why correlation feels sticky
- know why timing matters
- predict when correlation will or will not change

Next, we cover correlation outcomes and downstream impact.

---

## Navigation

**⬅ Back:** Part 2 – Account Correlation Configuration and Criteria Design  
**➡ Next:** Part 4 – Correlation Outcomes and Result Semantics

---

*End of Part 3*
