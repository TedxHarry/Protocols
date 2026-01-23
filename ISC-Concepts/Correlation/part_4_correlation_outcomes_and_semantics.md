# Part 4 – Correlation Outcomes and Result Semantics

> **Purpose of this part**  
This chapter explains what correlation *produces* and why those outcomes matter long after aggregation finishes.
We focus on meaning, not mechanics.

By the end of this part, you should:
- understand what “correlated” really implies
- understand what “uncorrelated” really breaks
- predict downstream behavior in governance, requests, and audits

---

## 4.1 Correlation has only two outcomes

No matter how complex the setup looks, correlation always ends the same way.

For every account, ISC records **one of two states**:
- correlated
- uncorrelated

There are no partial matches.
There is no “almost correlated”.

This simplicity is intentional.

---

## 4.2 What “correlated” actually means

When an account is correlated, ISC has decided:

> “This account belongs to this identity.”

This single decision causes multiple downstream effects.

### Practical implications
A correlated account:
- appears under the identity
- participates in certifications
- is included in access requests
- follows identity-based approval paths
- is evaluated by policies using identity context

Correlation is what makes governance personal.

---

## 4.3 What correlation does *not* guarantee

Correlation does **not** mean:
- the account is correct
- the account is active
- the account should exist
- the account should have access

Correlation only establishes ownership.
Correctness is a separate concern.

This distinction is critical during audits.

---

## 4.4 What “uncorrelated” really means

An uncorrelated account means:

> “ISC cannot confidently assign this account to any identity.”

It does **not** mean:
- the account is invalid
- the account is unused
- the account is ignored

It means ownership is unknown.

---

## 4.5 Why uncorrelated accounts are dangerous

Uncorrelated accounts are risky because they sit outside identity context.

Common consequences:
- missed certifications
- incomplete access reviews
- policies not firing correctly
- audit findings with no clear owner

The risk is silent.
Everything looks fine until someone asks, “Who owns this?”

---

## 4.6 Governance behavior with correlated accounts

When an account is correlated:

### Certifications
- account access is reviewed by identity owner or manager
- ownership chain is clear

### Access requests
- approvals follow identity hierarchy
- request context is preserved

### Policies
- identity attributes are available
- SoD and lifecycle rules apply correctly

Correlation enables governance logic to work as designed.

---

## 4.7 Governance behavior with uncorrelated accounts

When an account is uncorrelated:

### Certifications
- account may appear in special “uncorrelated” scopes
- ownership is ambiguous

### Access requests
- identity-based approvals may not apply
- request routing can be limited or blocked

### Policies
- identity attributes are unavailable
- policy evaluation becomes weaker or skipped

Governance still runs, but without identity intelligence.

---

## 4.8 Why “it shows up in the UI” is misleading

A common misunderstanding:

> “I see the account in ISC, so it must be governed.”

Visibility is not ownership.

An uncorrelated account can be visible:
- in source views
- in reports
- in raw aggregation data

But without correlation, it is not fully governable.

---

## 4.9 Correlation permanence and memory

Once an account is correlated:
- ISC remembers the assignment
- future aggregations often preserve it
- correlation does not reset automatically

This persistence explains why:
- fixes don’t apply retroactively
- mistakes linger quietly

Correlation is durable by design.

---

## 4.10 Downstream impact of wrong correlation

Wrong correlation is worse than no correlation.

Effects include:
- access reviewed by the wrong manager
- incorrect approvals
- audit trails pointing to the wrong person
- trust erosion in governance reports

Incorrect ownership poisons everything built on top of it.

---

## 4.11 Why uncorrelated is sometimes safer

In some scenarios, uncorrelated is preferable.

Examples:
- shared service accounts
- orphaned legacy accounts
- accounts with unreliable identifiers

Leaving them uncorrelated:
- avoids false ownership
- flags them for investigation
- keeps governance honest

Correlation should be confident, not forced.

---

## 4.12 Operational mindset shift

Stop thinking:
> “How do I correlate everything?”

Start thinking:
> “Which accounts can I correlate *correctly*?”

Accuracy beats coverage.

---

## 4.13 Real-world example

Two users share a contractor email alias.

Correlation uses email.
Both accounts attach to one identity.

Result:
- certifications look clean
- access reviews approve incorrectly
- audit fails months later

If correlation had failed:
- risk would have been visible earlier

---

## 4.14 How to evaluate correlation health

Ask:
- are uncorrelated accounts explainable?
- are correlated accounts trustworthy?
- do certifications align with real ownership?
- can auditors trace ownership confidently?

Correlation quality is a governance maturity signal.

---

## 4.15 What this part prepares you for

After this part, you should:
- respect uncorrelated states
- fear incorrect correlation more than missing correlation
- understand why resolution requires care

Next, we move into **how to resolve uncorrelated accounts safely**.

---

## Navigation

**⬅ Back:** - [Part 3 – Correlation Runtime Execution and Data Flow](part_3_correlation_runtime_execution_and_flow.md)

**➡ Next:** - [Part 5 – Uncorrelated Accounts Resolution and Safe Fixes](part_5_uncorrelated_accounts_resolution.md)

---

*End of Part 4*
