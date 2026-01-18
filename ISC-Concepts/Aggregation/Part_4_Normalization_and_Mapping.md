# Part 4 – Normalization and Mapping (Teaching Mastery Edition)

[⬅️ Back to Home](../README.md)

---

## Why This Part Exists

Part 3 (Extraction) answers:
**Did ISC read the data?**

Part 4 answers the next, more dangerous question:
**Did ISC understand the data correctly?**

Extraction copies raw truth from the source.
Normalization and mapping **shape that raw truth into ISC truth**.

If this step is wrong, everything downstream can still run perfectly…
but it will run on **mis-shaped data**, which creates the most confusing kind of bugs.

Keep this sentence in your head:

**If extraction is the camera, normalization is the translator, and mapping is the meaning.**

---

## Where This Fits in the End-to-End Flow

Trigger → Extract → **Normalize + Map** → Persist → Correlate → Evaluate → Recompute → Publish

- Extraction brings raw fields into the pipeline.
- Normalization converts raw values into ISC-friendly forms.
- Mapping decides **where each value belongs** inside ISC.

This is where “data” becomes “truth inside ISC.”

---

## The Mental Model (Use This Forever)

```
Raw source fields
  → Normalization (convert + clean + standardize)
    → Mapping (assign meaning + destination)
      → Stored account attributes
```

Think of it like translating a sentence:

- Normalization fixes spelling, converts units, standardizes the format.
- Mapping decides what each word means and where it belongs in the final sentence.

If you translate wrong, you don’t get a small error.
You get a **perfectly stored lie**.

---

## Definitions (Taught, Not Memorized)

### Normalization
Normalization is the step where ISC (and connector logic) converts raw source values into values ISC can reliably work with.

Examples of normalization:
- Convert a date string into a real date type
- Convert “Y/N” into true/false
- Trim spaces, fix casing, standardize separators
- Convert “1001” (text) into 1001 (number)
- Ensure a multi-value field stays multi-value

Normalization is about **shape and type**, not meaning.

### Mapping
Mapping is the step where you teach ISC what each source field means and where it should land.

Examples of mapping:
- employeeId → accountAttribute.employeeId
- email → accountAttribute.email
- department → accountAttribute.department
- hireDate → accountAttribute.hireDate

Mapping is about **meaning and destination**, not “cleaning.”

### Transform
A transform is logic applied during normalization/mapping to derive the final value.

Examples:
- firstName + lastName → displayName
- Trim spaces, change case, replace characters
- If department is blank, set “Unknown”
- Convert “ACTIVE/INACTIVE” into lifecycle states (where applicable)

Transforms are powerful because they can create new truth.
They’re risky because they can create wrong truth silently.

---

## A Guided Running Example (So the Concept Becomes Real)

Source: HR system  
People: Alice, Bob, Carol  
Fields: employeeId, email, department, hireDate

### Raw record for Alice (what extraction brought in)
- employeeId = "1001"
- email = "alice@company.com"
- department = "Engineering"
- hireDate = "2025-01-01"

At this moment, these are just raw strings coming from the source.
ISC has not decided what they *are* yet.

### Step 1: Normalization (make values usable)
- "1001" → number 1001
- "2025-01-01" → date 2025-01-01
- Trim/standardize strings (if needed)

### Step 2: Mapping (assign meaning + destination)
- 1001 → accountAttribute.employeeId
- alice@company.com → accountAttribute.email
- Engineering → accountAttribute.department
- 2025-01-01 → accountAttribute.hireDate

Now the data is not just present.
It is **structured truth inside ISC**.

---

## Why “Jobs Completed” Can Still Mean “Data Is Wrong”

A job status like Completed usually means:
- The worker finished
- The connector didn’t crash

It does **not** guarantee:
- Types were compatible
- Multi-values were preserved
- Transforms produced correct values
- Mappings pointed to the right fields

Normalization/mapping failures are often **silent**:
the job ends, but values are missing, malformed, or overwritten.

This is why beginners get confused and experienced operators get paranoid here.

---

## Data Types (The Silent Source of Most Problems)

Most “mystery bugs” here are type bugs.

### Text vs Number
- If employeeId is text in one place and number in another, comparisons can break.
- Matching/correlation logic that expects consistent types can fail quietly.

### Text vs Date
- If hireDate is stored as text, date logic (range, sorting, “older than”) behaves incorrectly.
- Some pipelines drop invalid dates instead of erroring loudly.

### Single-value vs Multi-value
This one causes the nastiest silent failure.

If you map a multi-value field into a single-value target:
- ISC may keep only one value (often last seen)
- Or drop values depending on the target field rules

Symptom:
- “Source has many values, ISC shows one.”

Root cause is rarely correlation.
It’s usually **multi vs single** mismatch during mapping.

---

## Transforms (When You’re Not Copying, You’re Creating)

Transforms exist because sources rarely give you values in the exact shape you want.

Good transforms:
- Build displayName from firstName + lastName
- Standardize email casing
- Convert “Y/N” into boolean
- Replace empty department with a default

Risky transforms:
- Changing identifiers used by correlation
- Deriving values with incomplete rules
- Copying transforms from another source without understanding

Master rule:
**If a transform touches an identifier (employeeId, email, username), treat it like production code.**

---

## Failure Patterns (Symptom → Most Likely Cause)

Use this to avoid wasting days.

### 🔴 Pattern: Field is always empty in ISC
Most likely:
- wrong mapping (pointing to wrong source field), or
- type mismatch causing silent drop, or
- transform produced null/blank

### 🔴 Pattern: Field looks “weird” (spacing, casing, formatting)
Most likely:
- missing/incorrect normalization or transform

### 🔴 Pattern: Only one value shows up when source has many
Most likely:
- multi-value mapped into a single-value target

### 🔴 Pattern: Correlation suddenly fails after a mapping change
Most likely:
- you changed the shape/type/value of a correlation key (email, employeeId, etc.)

---

## A Real Failure Story (So You Remember This)

HR had hire dates for everyone.
ISC showed hireDate empty for all accounts.

People blamed identity logic and correlation.

Root cause:
hireDate came in as text and was mapped into a date field.
Normalization dropped it quietly.

Lesson:
**Empty in ISC does not mean empty in source. It often means “dropped during normalization.”**

---

## Debug Playbook (The Order Masters Use)

When values look wrong, debug in this order:

### Step 1: Confirm the source value
Open the source record and confirm the exact raw value.
This prevents guessing.

### Step 2: Confirm extraction saw the raw field
If extraction never read the field, Part 3 is your problem.

### Step 3: Check mapping preview (your best friend)
Mapping preview shows what ISC believes the value becomes **after transforms**.
If preview is wrong, the stored value will be wrong.

### Step 4: Verify target field type and cardinality
Ask:
- Is this a date or text?
- Single or multi-value?
- Boolean or string?

### Step 5: Validate transform logic with one record
Test one user (Alice) and make sure the transform output matches expectation.

### Step 6: Only then move downstream
Correlation, identity profile logic, lifecycle state evaluation, access modeling.

Master question:
**Was the data shaped correctly before persistence?**

---

## Visual Debug Flow (Use This During Incidents)

```
Source value correct?
   ↓
Extraction read the field?
   ↓
Mapping preview shows correct transformed value?
   ↓
Target type + single/multi matches?
   ↓
If no → mapping/type/transform issue
   ↓
If yes → move downstream (persist/correlate)
```

---

## Proof Paths (How You Prove This Phase Is Healthy)

Use more than one proof path:

- **Mapping preview** (truth before it is stored)
- **API account attributes** (truth after persistence)
- **Logs** (warnings about types, transforms, drops)

If preview is correct but stored value is wrong, persistence is suspect (next part).
If preview is wrong, this part is your root cause.

---

## Safe Fixes (Do This Without Creating New Damage)

- Fix mapping → preview again before running full
- Fix transform → test on one record first
- Fix type mismatch → correct the target field, then rerun a small scope
- If you changed a correlation key → expect correlation impacts and re-test intentionally

---

## What Must Not Happen (Hard Rules)

- Do not change mappings mid-run
- Do not ignore type warnings
- Do not copy transforms blindly
- Do not “fix correlation” until you confirm normalization/mapping is correct

---

## The One Sentence That Defines Mastery

Before you blame correlation, ask:

**Was the data shaped correctly before it was stored?**

---

## Mastery Check (If You Can Teach This, You Own It)

Answer these without notes:

- What is the difference between normalization and mapping?
- Why can a job complete even when values are wrong?
- What are the three most dangerous type mismatches?
- Why is multi vs single-value a silent killer?
- What makes transforms risky?
- What is your debug order and why?

---
### Navigation
⬅️ Previous: [Part 3 – Extraction Phase](./Part_3_Extraction_Phase_Reading_Data.md)  
🏠 Home: [README – Aggregation Master Series](./README.md)  
➡️ Next: [Part 5 – Write and Persistence](./Part_5_Write_and_Persistence.md)
