# Part 8 – Manual Correlation, Permanence, and Risk (Deep Dive)

> **Purpose of this part**  
Manual correlation is the most powerful “break glass” tool in ISC correlation.
It is also the easiest way to silently destroy identity truth if used casually.

This part teaches manual correlation like an operator:
- what it really does under the hood
- why it persists
- how it interacts with aggregation and rule-based correlation
- how to use it safely, with proofs and guardrails

By the end of this part, you should:
- understand manual correlation semantics and permanence
- know when manual correlation is justified
- know when manual correlation is a trap
- follow a safe workflow that you can defend in audits

---

## 8.1 Start with a simple truth

Manual correlation is not “helping correlation”.

Manual correlation is **overriding correlation**.

It is ISC saying:

> “Do not try to infer ownership. I am explicitly telling you who owns this account.”

This is why it is powerful.
This is why it must be treated like a controlled exception.

---

## 8.2 What manual correlation produces

When you manually correlate an account:
- you create a **hard link** between an account and an identity
- ISC records that this link was created intentionally

This hard link behaves differently from rule-based correlation.

Rule-based correlation is inference.
Manual correlation is a directive.

---

## 8.3 The permanence principle

Manual correlation is sticky by design.

Once you manually correlate:
- aggregation can re-run
- account attributes can change
- identity attributes can change

The link remains.

Why?
Because ISC assumes you verified ownership.

That is the entire point of “manual”.

---

## 8.4 Why this stickiness exists

If manual correlation could be undone automatically:
- every future aggregation would risk breaking an operator’s intentional decision
- audits would become unreliable
- exceptions would be impossible to manage

So ISC treats manual correlation as higher confidence than inference.

In other words:

- rule-based correlation: “I think this belongs to Ted”
- manual correlation: “I know this belongs to Ted”

ISC trusts the second more.

---

## 8.5 Manual correlation is not a data quality fix

A common anti-pattern:

> “We have messy identifiers, so we will manually correlate everything.”

This creates a tenant that cannot self-heal.

If you manually correlate a large population:
- correlation rules stop being meaningful
- upstream fixes stop improving governance
- audits become dependent on human memory

Manual correlation should never replace good identifiers.

---

## 8.6 When manual correlation is justified (good use cases)

Manual correlation makes sense when **ownership is real but unprovable automatically**.

Examples:
- service accounts with a known business owner
- shared accounts that must be governed via a responsible identity
- legacy systems with inconsistent identifiers
- temporary exceptions during migrations
- newly onboarded apps where identifiers are not ready yet

The key requirement is always:

> You can prove ownership outside of ISC.

---

## 8.7 When manual correlation is a trap (danger patterns)

Manual correlation is dangerous when used to increase coverage metrics.

Red flags:
- “We need to reduce uncorrelated accounts quickly”
- “Email is missing, so just link it manually”
- “We will fix it later”
- “It’s only a few hundred accounts”

These decisions become permanent technical debt.

---

## 8.8 Manual correlation vs rule-based correlation (priority)

Think of correlation like a hierarchy of confidence:

1. manual correlation (explicit directive)
2. correlation rule logic (custom deterministic logic)
3. configured criteria list (ordered inference)
4. default correlation fallback (system heuristic)

Manual correlation sits at the top because it is the most explicit form of ownership.

This explains why changing criteria later may not affect manually correlated accounts.

---

## 8.9 Why manually correlated accounts “ignore” rule changes

If an account is manually correlated:
- changing correlation order does nothing
- changing attributes does nothing
- rerunning aggregation does nothing

Because ISC is not trying to infer ownership anymore.

If you want rule-based logic to take over:
- you must remove the manual correlation link first

Manual correlation is not a hint.
It is a lock.

---

## 8.10 Operational workflow: manual correlation done safely

Manual correlation should follow a strict sequence.

### Step 1: Confirm the account is truly uncorrelated
Do not manually correlate an account that is already correctly correlated.

First confirm:
- the account is uncorrelated
- the correlation rule did not match
- the identity exists and is correct

### Step 2: Prove ownership outside ISC
Proof examples:
- HR record mapping to the account
- system audit logs showing last use by the identity owner
- application owner sign-off
- ticket approval trail

No proof, no manual correlation.

### Step 3: Correlate only the minimum set
Manual correlation should be small and targeted.

If you are correlating hundreds:
- pause
- reassess readiness and identifiers
- you are likely fixing the wrong layer

### Step 4: Document the exception
Always record:
- why correlation could not infer ownership
- what proof was used
- who approved the manual link
- when it should be revisited

Manual correlation without documentation is future audit pain.

### Step 5: Monitor and revalidate
Manual correlations should be revalidated on a schedule.
Ownership can change.

---

## 8.11 The “reassignment” problem

A common scenario:

You manually correlated the account to Identity A.
Later, you discover it belongs to Identity B.

This is where manual correlation can hurt.

Because the link is sticky:
- correlation will not “self-correct”
- ownership remains wrong until you intervene

Operational rule:
- if you change ownership, remove the manual link first, then re-correlate correctly

Never assume aggregation will fix it.

---

## 8.12 The “migration” pattern

Manual correlation is most safely used as a temporary bridge during migrations.

Example:
- app onboarding begins before HR identifiers are stable
- you manually correlate critical users
- you fix identifiers and rules
- you gradually remove manual links as automation becomes trustworthy

This turns manual correlation into a controlled transition, not permanent debt.

---

## 8.13 Service accounts: the special case

Service accounts create the hardest ownership problem.

Because:
- they often do not map to a person
- identifiers are generic
- lifecycle is different

Safe approaches:
- manual correlate to a business owner identity
- or keep uncorrelated and govern via separate policy scopes

What you must avoid:
- correlating service accounts to “whoever created them” without ownership proof

---

## 8.14 Wrong manual correlation is worse than uncorrelated

Uncorrelated is honest uncertainty.
Wrong correlation is false certainty.

Wrong manual correlation causes:
- wrong certifications
- wrong approvals
- wrong audit ownership
- long-lived trust damage

If unsure, do not manually correlate.
Leave uncorrelated and investigate.

---

## 8.15 Debugging manual correlation issues

If results look “stuck”, ask:

1. was this account manually correlated?
2. is there a correlation rule overriding normal behavior?
3. was the account reprocessed by aggregation?
4. is the identity still correct?

Many “why won’t it change” cases are manual correlation locks.

---

## 8.16 What this part prepares you for

After this part, you should:
- treat manual correlation like a controlled exception
- use proofs and documentation
- avoid metric-driven manual linking
- understand why some links never change without intervention

Next, we go into **default correlation fallback and case sensitivity**, which explains many “mystery correlations”.

---

## Navigation

**⬅ Back:** Part 7 – Custom Identity Attributes and Searchability  
**➡ Next:** Part 9 – Default Correlation Fallback and Case Sensitivity

---

*End of Part 8*
