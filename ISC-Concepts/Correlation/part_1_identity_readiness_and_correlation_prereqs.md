# Part 1 – Identity Readiness and Correlation Prerequisites

> **Purpose of this part**  
This chapter explains *why correlation often fails before it even runs*.
Not because of bad rules, but because identities and attributes were not ready.

By the end of this part, you should:
- understand what must exist *before* correlation can succeed
- see how identity profiles silently control correlation
- know why “the criteria looks right” is often misleading

---

## 1.1 A hard truth upfront

Correlation does not create identity data.

It only **uses what already exists at the moment aggregation runs**.

If identities are missing, incomplete, or poorly shaped, correlation has nothing to work with.

Most correlation issues are not correlation problems.
They are **identity readiness problems**.

---

## 1.2 Correlation never runs in isolation

Correlation always runs **inside aggregation**.

This means correlation depends on everything that happened *before* it:

1. identity profile configuration
2. authoritative source behavior
3. attribute population timing
4. identity processing state

If any of these are wrong, correlation outcomes will be wrong.

---

## 1.3 The identity profile is the silent controller

The identity profile decides:
- which attributes exist on identities
- how those attributes are populated
- when those attributes become available

Correlation can only compare attributes that:
- exist on the identity
- are populated at runtime
- are searchable (when required)

If an attribute does not exist yet, correlation cannot magically use it.

---

## 1.4 Identity existence comes first

Correlation cannot assign an account to an identity that does not exist.

This sounds obvious, but it breaks many designs.

### Typical failure pattern

- HR source is authoritative
- identity creation happens later
- application source aggregates first
- correlation runs
- no identity exists yet
- account becomes uncorrelated

Nothing is wrong with correlation.
The identity simply did not exist at the time.

---

## 1.5 Timing matters more than configuration

Correlation is time-sensitive.

It evaluates values **as they exist at that exact run**.

If an attribute is:
- populated after aggregation
- derived by a transform that runs later
- dependent on identity processing that hasn’t happened

Then correlation will not see it.

This is why:
- “I see the value in the UI later” does not mean correlation saw it earlier

---

## 1.6 Authoritative sources shape readiness

Authoritative sources are special.

They:
- can create identities
- populate core attributes
- define initial identity truth

If the authoritative source:
- runs after application aggregation
- does not populate the right identifier
- uses inconsistent values

Then downstream correlation will struggle.

Good correlation starts with clean authoritative data.

---

## 1.7 Attribute quality beats attribute quantity

More attributes do not mean better correlation.

Correlation prefers attributes that are:
- stable
- unique
- present early in the lifecycle

Weak attributes cause silent damage.

Examples of risky primary identifiers:
- email (can change)
- displayName (not unique)
- manager name (not stable)

Use these only as fallback, never as primary.

---

## 1.8 Searchable attributes are a hidden requirement

Not every identity attribute can be used for correlation.

For an identity attribute to appear in correlation configuration:
- it must be marked **searchable**

This is not cosmetic.
Searchable means ISC can efficiently look up identities by that value during aggregation.

If an attribute is not searchable:
- it may exist
- it may be populated
- but correlation cannot reliably use it

---

## 1.9 Identity processing state matters

Identities are not always “fully processed”.

After changes such as:
- adding new identity attributes
- changing mappings
- modifying authoritative behavior

You may need identity processing to run **before** correlation sees those changes.

If identity processing has not caught up:
- correlation uses stale identity data
- results look inconsistent

---

## 1.10 A common false belief

> “If I fix the correlation rule and re-aggregate, it will work.”

Not always.

If identity readiness is still broken:
- same bad input
- same bad outcome

Correlation is deterministic.
It does not improve over time on its own.

---

## 1.11 How to evaluate readiness before touching correlation

Before configuring correlation, ask:

1. Do identities exist before this source aggregates?
2. Are the attributes I plan to use already populated?
3. Are those attributes stable and unique?
4. Are they searchable if required?
5. Does identity processing run before aggregation?

If any answer is “no”, correlation will struggle.

---

## 1.12 Real-world example

HR source creates identities using `employeeId`.

Application source tries to correlate using `email`.

But:
- email is populated later from another source
- identity exists, but email is blank at correlation time

Result:
- application accounts are uncorrelated

Fix:
- change correlation to use `employeeId`
- or fix attribute timing

The rule was fine.
The readiness was not.

---

## 1.13 What this part prepares you for

After this part, you should:
- stop blaming correlation rules prematurely
- diagnose timing and identity issues first
- design correlation based on lifecycle reality

Only now does correlation configuration become meaningful.

---

## Navigation

**⬅ Back:** Part 0 – Correlation Mental Model, Configuration Meaning, and Runtime Flow  
**➡ Next:** Part 2 – Account Correlation Configuration and Criteria Design

---

*End of Part 1*
