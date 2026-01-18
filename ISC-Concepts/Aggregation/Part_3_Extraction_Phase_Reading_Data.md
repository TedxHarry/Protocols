# Part 3 – Extraction Phase (Reading Data)

[⬅️ Back to Home](../README.md)

---

## Why This Part Exists

Extraction is the moment ISC **decides what is real**.

Before correlation, before identity updates, before access modeling, ISC must first *see* data from the source.
If extraction misses an object, nothing downstream can “fix it” because ISC never had the raw input to work with.

Keep this sentence in your head as you read:

**If ISC didn’t extract it, ISC can’t govern it.**

---

## Where Extraction Fits in the End-to-End Flow

Extraction is early in the pipeline:

Trigger → Job → **Extract** → Normalize → Persist → Correlate → Evaluate → Recompute → Publish

Extraction answers one question:

**What data gets a chance to exist inside ISC?**

Everything after extraction is interpretation and governance.
Extraction is the “camera.” If the camera never captured the scene, you can’t reconstruct it later.

---

## The Mental Model (The One You Should Use Forever)

```
Reality in the source → Extraction Gate → Reality inside ISC
```

- The source has real objects (accounts, groups, memberships).
- Extraction is the gate that brings some of that reality into ISC.
- Everything downstream works only on what passed through that gate.

So when something looks wrong later, a master asks early:

**Did extraction even see it?**

---

## What Extraction Really Is (Not a Fact List, the Actual Concept)

Many people imagine extraction like a big download:
“ISC pulls all data from the source.”

That’s not how it behaves in practice.

Extraction is a **conversation** between ISC (through a connector) and the source:

- “Give me the first page of accounts.”
- “Now give me the next page.”
- “Now give me only what changed since last time.”

It’s repetitive, page-by-page, object-by-object.
And because it’s a conversation, it can fail in quiet ways:
pagination can stop early, filters can exclude people, delta memory can skip changes.

This is why jobs can show **Completed** while the truth is still incomplete.

---

## The Three Things Extraction Reads (Three Pipelines)

Extraction isn’t one stream. It’s three distinct pipelines.
Treat them as separate systems with separate failure modes.

| Pipeline | What it reads | What you’ll see if it fails |
|---|---|---|
| **Account extraction** | The account objects themselves (users, identities in the source) | People missing in ISC. Correlation never gets a chance. |
| **Entitlement extraction** | Groups/roles/permissions (the “things you can have”) | Access modeling and certifications look empty or incomplete. |
| **Membership extraction** | Links between accounts and entitlements (who has what) | Accounts exist, entitlements exist, but nobody appears to have access. |

Mastery trick:
If your symptom is “missing access,” don’t jump to correlation.
First decide which pipeline is lying.

---

## A Guided Running Example (So the Terms Become Real)

Source: HR system

### Step 0: What exists in the source (Reality)
Accounts:
- Alice
- Bob
- Carol

Entitlements:
- Engineering
- HR
- Finance

Memberships:
- Alice → Engineering
- Bob → HR
- Carol → Finance

### Step 1: What extraction does (Copy reality into ISC)
Extraction should read:
- 3 accounts
- 3 entitlements
- 3 membership links

### Step 2: What ISC does NOT do yet
At extraction time, ISC is not deciding:
- Who is the “identity”
- Who matches whom (correlation)
- What lifecycle state wins
- Whether access is compliant

Extraction is simply: **read raw truth**.

If raw truth is wrong, every later “smart” step becomes confidently wrong.

---

## The “Machine” Under the Hood (How It Runs)

Here’s the simplest accurate picture:

1) A job starts
2) Connector requests a page of data
3) Source returns a page (100, 500, etc.)
4) ISC processes the page
5) Connector requests the next page
6) Repeat until the source returns “no more”

Sometimes the request includes a condition:

- “Only give me objects changed since my last successful run.”

So extraction is governed by three control areas:

- **Pagination** (how the connector walks pages)
- **Filters and scope** (which objects are included/excluded)
- **Delta memory** (how the connector remembers “since when”)

If any of these are wrong, extraction can produce partial truth.

---

## The Control Levers (What You Are Actually Changing)

These settings do not just change performance.
They can change what exists in ISC.

### 1) Pagination
Pagination is how you get from page 1 to page N.

If pagination breaks after page 1:
- the job might still end “cleanly”
- counts will be wrong
- large parts of the population will be invisible

### 2) Filters and scope
Filters decide who is eligible to be read:
- only active users
- only a certain OU/container
- only users matching a condition

A bad filter doesn’t create an error.
It creates **invisible people**.

### 3) Delta memory (tokens, timestamps, watermarks)
Delta extraction depends on a remembered checkpoint:
- last success timestamp
- change token
- watermark

If that checkpoint is wrong, delta can:
- skip real changes
- behave like full (slow)
- create partial truth (most dangerous)

---

## Failure Patterns (Symptoms → Most Likely Extraction Cause)

When something looks wrong, pattern-match first.
This saves hours.

### 🔴 Pattern: “Some users are missing”
Most likely:
- pagination stopped early, or
- a filter/scope excluded them, or
- delta checkpoint skipped them

### 🔴 Pattern: “Entitlements are missing”
Most likely:
- entitlement extraction didn’t run correctly, or
- the connector lacks permission to read entitlements

### 🔴 Pattern: “Accounts exist, entitlements exist, but nobody has access”
Most likely:
- membership extraction failed or returned empty links

### 🔴 Pattern: “Delta ran, but it didn’t pick up changes”
Most likely:
- delta memory (token/timestamp) is incorrect or stale

---

## A Real Failure Story (Why This Matters in Production)

Source had 5,000 users.
ISC showed 500.

People blamed correlation for days because correlation is “where identities happen.”

But extraction never read beyond page 1.
The job said Completed because the worker stopped, not because it read everything.

Lesson:
**Completed is a job status, not a truth guarantee.**

Truth comes from counts, logs, and proof paths.

---

## Debug Playbook (The Order Masters Use)

When data looks wrong, debug in this exact order.

### Step 1: Confirm reality in the source
Is the object actually there?
Is it in scope (OU/container/active status)?

### Step 2: Confirm extraction visibility
Did the extraction logs or job results ever mention the object?
If the object is never referenced, it likely never got read.

### Step 3: Identify the extraction failure class
- Filter/scope issue?
- Pagination issue?
- Delta memory issue?
- Permission issue (especially for entitlements/memberships)?

### Step 4: Only then move downstream
Correlation, identity profile logic, lifecycle overrides, access model updates.

If extraction didn’t see it, downstream analysis is wasted time.

---

## Visual Debug Flow (Use This During Incidents)

```
Is the object in the source?
   ↓
Is the object in extraction scope (filters/OUs/status)?
   ↓
Do logs show it being read?
   ↓
If no → filter/scope OR pagination OR delta memory
   ↓
If yes → move downstream (normalize/persist/correlate)
```

---

## Proof Paths (How You Prove Extraction Is Telling the Truth)

Use multiple proof paths, not one.

- **UI job counts** (what ISC thinks it read)
- **API object counts** (accounts, entitlements, memberships)
- **Logs** (page-by-page evidence, filter evidence, delta checkpoint evidence)

If the counts don’t match the source, assume extraction first.

---

## Safe Fixes (Fix the Truth Before You Fix the Model)

- Counts wrong → run **full extraction once** to reset the baseline
- Filters wrong → simplify filters, test, then tighten carefully
- Pagination broken → reduce page size, validate page traversal in logs
- Delta broken → reset checkpoint and run full once, then re-enable delta

---

## Deep Dives (Optional, Expand When You Want More)

<details>
<summary><strong>Deep Dive: Pagination Failure Modes</strong></summary>

Pagination failures often look like “random missing people” but the pattern is usually size-related.

Common causes:
- Connector stops after first page due to an error handling bug
- Next-page token not handled correctly
- Timeouts that stop paging early
- Source rate limiting that ends the loop

What to look for:
- log lines that show page 1, but not page 2
- total counts that are near your page size (100, 500, 1000)
</details>

<details>
<summary><strong>Deep Dive: Delta Checkpoint Pitfalls</strong></summary>

Delta extraction relies on a remembered checkpoint.

If the checkpoint moves forward incorrectly:
- changes can be skipped permanently until a full run resets reality

If the checkpoint never moves:
- delta behaves like full (slow and expensive)

What to look for:
- last successful delta timestamp/token
- gaps where changes existed but weren’t captured
</details>

<details>
<summary><strong>Deep Dive: Membership Extraction Gotchas</strong></summary>

Membership is the link layer.
If membership fails, it produces the most confusing symptom:
“Everything exists but nobody has access.”

What to validate:
- does the source actually expose membership links?
- does the connector have permission to read group membership?
- are membership objects returned as separate endpoints/pages?
</details>

---

## What Must Not Happen (Hard Rules)

- Do not trust Completed as complete
- Do not change filters casually without measuring counts
- Do not rely on delta until full is proven stable
- Do not ignore pagination hints in logs

---

## The One Sentence That Defines Mastery

Before you blame correlation, mappings, or governance logic, ask:

**Did extraction even see the object?**

---

## Mastery Check (If You Can Teach This, You Own It)

Answer these without notes:

- What is extraction in one sentence?
- Why is extraction a “gatekeeper of truth”?
- What are the three pipelines and their symptoms?
- Why can Completed still be incomplete?
- What are the three main control levers?
- What is your debug order, and why?

---

### Navigation
⬅️ Previous: [Part 2 – Triggers and Job Lifecycle](./Part_2_Triggers_and_Job_Lifecycle.md)  
🏠 Home: [README – Aggregation Master Series](./README.md)  
➡️ Next: [Part 4 – Normalization and Mapping](./Part_4_Normalization_and_Mapping.md)
