# Part 0 – Correlation Mental Model, Configuration Meaning, and Runtime Flow

> **Purpose of this part**  
This chapter teaches correlation the way ISC actually thinks.
Not as clicks or settings, but as logic, flow, and decisions.

By the end, you should be able to:
- explain correlation without UI terms
- predict outcomes before aggregation runs
- understand *why* correlation succeeds or fails

This is the foundation for everything that follows.

---

## 0.1 Start with human intuition

Forget ISC for a moment.

You find a laptop on a desk.
It shows a username and an email sticker.

Before doing anything else, you ask:

> “Who does this belong to?”

That question is correlation.

Correlation is not technical at first.
It is human ownership logic translated into rules.

Only after ownership is clear do permissions, approvals, and governance matter.

---

## 0.2 What correlation really is

Now the formal definition:

**Correlation assigns a source account to an identity by matching account attribute values to identity attribute values using an ordered set of criteria.**

Notice what correlation does *not* do:
- it does not grant access
- it does not assign roles
- it does not provision anything

Correlation decides ownership. Nothing more.

---

## 0.3 Why correlation exists in ISC

ISC operates in two separate worlds.

### World 1: Sources
Sources provide accounts.
Accounts are raw records from systems.

Examples:
- HR platforms
- Active Directory
- SaaS applications

At this stage, accounts do not mean “person.”

### World 2: Identities
Identities represent people inside ISC.
Governance works here.

Examples:
- certifications
- access requests
- policies
- approvals

### Correlation is the bridge

Without correlation:
- accounts float without owners
- governance loses context
- approvals break
- risk increases silently

Correlation answers one question only:

> **Does this account belong to an identity I already know?**

---

## 0.4 The mental model that removes confusion

Many assume identity comes first.

ISC actually works like this:

1. accounts arrive from aggregation
2. ISC attempts correlation
3. ownership is assigned or left unknown

Think of correlation as a security checkpoint.

- match found, account enters the identity
- no match, account waits outside

No guessing. Only matching.

---

## 0.5 What correlation is not

Setting boundaries avoids wrong expectations.

Correlation is **not**:
- a merge engine
- lifecycle logic
- provisioning logic
- access decision logic

Correlation does not fix bad data.
It does not clean duplicates.
It does not make assumptions.

It only compares values.

---

## 0.6 The three objects involved

### Identity
An identity is how ISC understands a person.

Think of it as:
- a folder for accounts
- the target of governance

Without accounts, an identity has no system presence.

### Account
An account is how a system describes a user.

Think of it as:
- raw data
- possibly incomplete
- untrusted until linked

### Source
A source is where accounts originate.

Sources do not know identities.
ISC connects them.

---

## 0.7 Identity attributes vs account attributes

Correlation works by comparing values from different places.

### Identity attributes
- live on the identity
- populated by identity profiles
- represent canonical person data

Examples:
- employeeId
- email
- username

### Account attributes
- live on the account
- come from the source
- represent system-specific data

Examples:
- sAMAccountName
- uid
- mail

Correlation asks:

> “Does any identity have the same value as this account?”

---

## 0.8 What a correlation criteria actually means

A correlation rule is simple:

> Identity attribute equals account attribute

Key truths:
- equality only
- no partial match
- no fuzzy logic

One successful match is enough.

---

## 0.9 Order is the real engine

ISC evaluates criteria sequentially.

Its thinking is literal:

- try rule one
- if it matches, stop
- if not, try the next

This makes order critical.

Strong identifiers go first.
Weak identifiers go later.

If order is wrong, correlation feels random even when it is not.

---

## 0.10 Authoritative context changes impact

Matching logic stays the same.
Impact changes.

### Authoritative sources
- can create identities
- define initial identity truth

Mistakes here multiply quickly.

### Non-authoritative sources
- only attach accounts
- cannot create identities

Same logic, lower risk.

---

## 0.11 Configuration vs execution

Admins configure intent.
ISC executes literally.

### What you configure
An ordered list:
- identity attribute
- account attribute

Nothing more.

### What ISC executes
For each account:
1. pull data
2. normalize attributes
3. evaluate criteria in order
4. stop on first match
5. assign or leave uncorrelated

Configuration describes *what to try*.
Execution follows *exact order*.

---

## 0.12 Runtime flow, slowed down

For one account:

1. account arrives
2. attributes are mapped
3. rule one is evaluated
4. rule two is evaluated if needed
5. match succeeds or fails
6. result is written

Every account ends in one state:
- correlated
- uncorrelated

No partial outcomes exist.

---

## 0.13 Why correlation fails in real tenants

Failures are usually simple.

Common causes:
- missing values
- non-unique values
- weak identifiers
- wrong order

Clean data beats clever configuration every time.

---

## 0.14 How to troubleshoot like ISC

Stop guessing.

Ask:
1. what value did the account have
2. what values existed on identities
3. which rule ran first
4. why did it stop
5. why did none match

If you can answer these, correlation becomes predictable.

---

## 0.15 What this part prepares you for

After this part, you should:
- think in matching logic
- understand configuration meaning
- predict runtime behavior
- avoid trial-and-error correlation

Only now does deeper configuration make sense.

---

## Navigation

**⬅ Back:** README – Correlation Master Series  
**➡ Next:** - [Part 1 – Identity Readiness and Correlation Prerequisites](part_1_identity_readiness_and_correlation_prereqs.md)

---

*End of Part 0*
