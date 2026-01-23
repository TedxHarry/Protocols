# Part 5 – Uncorrelated Accounts Resolution and Safe Fixes

> **Purpose of this part**  
This chapter teaches how to deal with uncorrelated accounts **without creating bigger problems**.
The goal is not to eliminate all uncorrelated accounts, but to resolve them *correctly*.

By the end of this part, you should:
- understand why uncorrelated accounts exist
- know when to fix data vs fix configuration
- apply safe resolution patterns
- avoid permanent correlation mistakes

---

## 5.1 First rule: uncorrelated is not an error

An uncorrelated account means:

> “ISC could not confidently assign ownership.”

That is not a failure.
That is honesty.

Treating uncorrelated accounts as defects leads to rushed fixes and bad correlation.

---

## 5.2 Why accounts end up uncorrelated

Uncorrelated accounts usually exist for predictable reasons:

- identity does not exist yet
- required attribute was missing at runtime
- attribute value was not unique
- ordering skipped the correct rule
- account truly has no owner (shared or service)

Understanding the cause always comes before fixing.

---

## 5.3 Start with classification, not correction

Before changing anything, classify uncorrelated accounts.

Typical buckets:
- future employees (identity not created yet)
- former employees (orphaned accounts)
- service or shared accounts
- data quality issues
- configuration gaps

Each bucket needs a different response.

---

## 5.4 The safest fix is almost always data

If correlation failed because:
- value was missing
- value was wrong
- value was delayed

Then the fix belongs in the **source**, not ISC.

Fixing data upstream:
- improves correlation everywhere
- avoids permanent overrides
- preserves audit integrity

---

## 5.5 When configuration is the right fix

Change correlation configuration only when:
- the chosen identifier does not reflect reality
- ordering does not match risk
- identity lifecycle changed

Configuration changes should be deliberate and minimal.

Never “add another row just in case”.

---

## 5.6 The danger of manual correlation

Manual correlation feels fast.
It is also permanent.

Manual correlation:
- creates a hard link
- bypasses normal matching logic
- survives future aggregation changes

Use it only when:
- ownership is verified
- automation cannot safely determine ownership
- the exception is documented

---

## 5.7 Manual correlation is not a cleanup tool

Do **not** use manual correlation to:
- hide data quality problems
- compensate for missing identifiers
- speed through onboarding

If manual correlation is frequent, the design is wrong.

---

## 5.8 Service and shared accounts

Some accounts should remain uncorrelated.

Examples:
- generic admin accounts
- shared mailboxes
- integration users

Forcing correlation here creates false ownership.

Better patterns:
- leave uncorrelated
- classify separately
- govern with alternate controls

---

## 5.9 Timing-based uncorrelation

Some uncorrelated accounts are temporary.

Example:
- application aggregates before HR
- identity is created later
- correlation succeeds on next run

Do not rush to fix temporary states.

---

## 5.10 A safe resolution workflow

A recommended sequence:

1. review uncorrelated report
2. classify accounts
3. verify data availability and timing
4. fix upstream data
5. adjust configuration if justified
6. use manual correlation only as exception

This workflow avoids long-term damage.

---

## 5.11 What not to do

Avoid these mistakes:
- forcing correlation for metrics
- using weak identifiers to reduce counts
- manual correlation without documentation
- assuming uncorrelated equals unused

---

## 5.12 Real-world example

An app has 200 uncorrelated accounts.

Investigation shows:
- HR aggregates at 6 AM
- app aggregates at 2 AM
- identities don’t exist yet

Fix:
- change aggregation order
- no rule changes
- uncorrelated count drops naturally

---

## 5.13 Operational mindset shift

Stop asking:
> “How do I eliminate uncorrelated accounts?”

Start asking:
> “Which uncorrelated accounts are expected, and which are not?”

---

## 5.14 What this part prepares you for

After this part, you should:
- trust uncorrelated states
- fix causes, not symptoms
- use manual correlation responsibly

Next, we move into recommendations and optimization logic.

---

## Navigation

**⬅ Back:** - [Part 4 – Correlation Outcomes and Result Semantics](part_4_correlation_outcomes_and_semantics.md)

**➡ Next:** - [Part 6 – Correlation Recommendations and Optimization Logic](part_6_correlation_recommendations_and_optimization.md)

---

*End of Part 5*
