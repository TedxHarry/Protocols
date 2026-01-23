# SailPoint ISC Correlation – One‑Page End‑to‑End Data Flow Lifecycle (Detailed)

> **Goal**  
This page explains **what actually happens to data** during correlation in SailPoint Identity Security Cloud (ISC), end to end.  
It follows the data from *source → aggregation → identity → correlation decision → persistence*.

If you understand this page, you understand correlation at runtime.

---

## 1. Source Emits Raw Account Data

Everything starts outside ISC.

A source system (HR, AD, Entra, App DB) provides:
- raw account objects
- raw attribute values
- raw identifiers (often inconsistent)

Important:
- ISC does **not** trust raw data
- nothing correlates at this stage

---

## 2. Connector Ingestion (Normalization Layer)

ISC connector:
- pulls raw accounts
- applies schema discovery
- normalizes attributes into ISC schema fields

Output:
- **account objects** with normalized attribute names
- still unowned
- still identity‑agnostic

No identity logic yet.

---

## 3. Account Object Creation or Update

ISC now decides:
- is this a **new account**?
- or an **existing account update**?

Decision is based on:
- account ID
- account name (as defined in schema)

At this point:
- correlation has not run
- ownership is unknown

---

## 4. Identity Profile Mapping (Attribute Projection)

Before correlation, ISC prepares identity data.

For authoritative sources:
- account attributes map to **identity attributes**
- identity attributes are staged for processing

Key point:
- mapping does not equal persistence yet
- identity processing still has to run

---

## 5. Identity Processing (Identity State Formation)

Identity processing:
- creates new identities (if authoritative)
- updates existing identities
- evaluates lifecycle state
- finalizes identity attribute values

After this step:
- identities exist
- identifiers are real
- identity state is stable

Correlation depends on this step.

---

## 6. Correlation Engine Invocation

Now correlation finally begins.

For **each account**, ISC asks:
> “Which identity should own this account?”

This question is evaluated **one account at a time**.

---

## 7. Correlation Decision Stack (Runtime)

For the current account, ISC evaluates **top to bottom**:

1. **Manual Correlation**
   - if present, stop immediately

2. **Correlation Rules**
   - if rule returns an identity, stop

3. **Correlation Criteria**
   - evaluated in configured order
   - first successful match wins

4. **Default Fallback**
   - account name ↔ identity name
   - case‑insensitive
   - last resort

If none match:
- account remains uncorrelated

---

## 8. Identity Lookup and Matching

During criteria or rule evaluation:
- ISC searches identity store
- uses current identity attribute values
- respects searchability constraints

Important:
- only identities that **exist now** can be matched
- late identities are invisible

---

## 9. Correlation Outcome Determination

Possible outcomes:
- account correlated to exactly one identity
- account remains uncorrelated
- (error only if configuration is invalid)

ISC does **not**:
- pick “best of many”
- guess ownership
- retry later automatically

---

## 10. Persistence of the Decision

Once decided:
- ownership link is written
- decision is persisted

Persistence means:
- future delta aggregations reuse it
- logic changes do not reapply automatically
- memory is intentional

This is why correlation feels “sticky”.

---

## 11. Manager Correlation (Parallel but Dependent)

Separately, for identities:
- manager identifier is evaluated
- ISC resolves **identity → identity** link

Requirements:
- manager identity must already exist
- manager identifier must be searchable
- sequencing must be correct

Failures here affect approvals, not account ownership.

---

## 12. Governance Consumption

After correlation:
- access requests use identity ownership
- certifications rely on correct managers
- policies assume identity truth
- reporting reflects correlated state

Governance trusts correlation completely.

---

## 13. Delta vs Full Data Flow Impact

Delta aggregation:
- skips unchanged accounts
- preserves ownership

Full aggregation:
- re‑evaluates correlation

Many “why didn’t it change?” issues are here.

---

## 14. End‑to‑End Data Flow Summary

**Raw Source Data**  
→ Schema Normalization  
→ Account Object  
→ Identity Attribute Mapping  
→ Identity Processing  
→ Correlation Decision  
→ Ownership Persistence  
→ Governance Usage  

---

## The One‑Sentence Runtime Truth

> **Correlation is a one‑time ownership decision made at aggregation, using the identity state that exists at that exact moment, and then remembered.**

---

**Use this page to debug data‑flow questions.**  
If something looks wrong, ask: *At which step did the data not exist yet?*
