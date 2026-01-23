# SailPoint ISC Correlation – One‑Page Mental Model Cheat Sheet

> **Goal**  
This page gives you a *single mental picture* of how correlation works in SailPoint Identity Security Cloud (ISC).  
If you remember this page, correlation will never feel random again.

---

## 1. What Correlation Really Is

**Correlation answers one question only:**

> *“Which identity should own this account?”*

It does **not**:
- validate access
- fix bad data
- invent ownership
- correct history

Correlation is **inference + overrides + memory**.

---

## 2. The Four Decision Layers (Highest Priority First)

Correlation decisions follow a strict hierarchy:

1. **Manual Correlation**  
   Human decision. Absolute. Persistent.

2. **Correlation Rules**  
   Programmable logic. Overrides configuration.

3. **Correlation Criteria**  
   Ordered inference based on identifiers.

4. **Default Fallback**  
   Name-based, case-insensitive last resort.

Higher layers **short‑circuit** lower ones.

---

## 3. The Core Runtime Question

For each account, ISC asks:

> *“Do I already know who owns this?”*

Then proceeds **top → down** through the layers until a decision is made.

---

## 4. Correlation Is Per‑Account, Per‑Run

Important realities:
- Correlation runs **one account at a time**
- Decisions are made **during aggregation**
- ISC does **not** rethink ownership unless triggered
- Old decisions persist unless explicitly changed

Correlation is **not continuously re‑evaluated**.

---

## 5. Identity Readiness Is Everything

Correlation cannot succeed if the identity is not ready.

Before correlation:
- identity must exist
- identifiers must be populated
- identity processing must complete

Seeing values later in the UI ≠ values existed at runtime.

---

## 6. Attribute Readiness Rules

A correlation attribute must be:
- populated **before** aggregation
- stable over time
- unique in practice
- searchable if required

Late attributes cause:
- failed correlation
- fallback correlation
- wrong ownership

---

## 7. Default Fallback (The Silent Actor)

If nothing matches:
- ISC tries account name ↔ identity name
- comparison is **case‑insensitive**
- this can cause “mystery correlations”

Fallback should be:
- understood
- tolerated
- never relied on as a strategy

---

## 8. Manual Correlation Is a Lock

Manual correlation:
- overrides all logic
- survives config changes
- survives aggregation
- survives rules

It must be **removed** before logic can reapply.

---

## 9. Delta vs Full Aggregation

Delta aggregation:
- skips unchanged accounts
- preserves previous decisions

Full aggregation:
- re‑evaluates correlation

Many “why didn’t it change?” issues are delta behavior.

---

## 10. Manager Correlation Is Different

Account correlation:
> Account → Identity

Manager correlation:
> Identity → Identity

Manager identity must exist **first** or resolution fails.

Most manager issues are **sequencing**, not mapping.

---

## 11. Sequencing Mental Model

Safe order:

1. Authoritative source aggregates  
2. Identity processing runs  
3. Identities stabilize  
4. Applications aggregate  
5. Correlation executes  

Wrong order = unpredictable outcomes.

---

## 12. Troubleshooting Order (Never Skip)

When correlation surprises you, think in this order:

1. Manual?
2. Rule?
3. Criteria?
4. Fallback?
5. Attribute readiness?
6. Identity existence?
7. Sequencing?
8. Persistence?

This sequence never lies.

---

## 13. The Golden Rule

> **Correlation never fails randomly.**  
If the result is wrong, something happened **too early**, **too late**, or **once and never undone**.

---

## 14. One‑Sentence Executive Explanation

> *“We only automate ownership when we can prove it.  
When we can’t, we make the risk visible instead of guessing.”*

---

**Use this page as your anchor.**  
Everything else in correlation is detail built on this model.
