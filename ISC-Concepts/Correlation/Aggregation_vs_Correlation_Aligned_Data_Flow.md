# SailPoint ISC – Aggregation vs Correlation  
## End-to-End Data Flow Alignment (Symmetric, Deep Dive)

> **Purpose**  
This page aligns **Aggregation** and **Correlation** data flows side by side so you can see:
- where aggregation ends
- where correlation begins
- what data exists at each moment
- why correlation issues are almost always aggregation or sequencing issues first

If you understand this page, you understand **ISC end to end**.

---

## 0. The Core Mental Split

**Aggregation answers:**  
> “What data exists in ISC right now?”

**Correlation answers:**  
> “Who owns this data?”

Aggregation builds **truth**.  
Correlation assigns **meaning**.

Correlation can never be better than aggregation.

---

## 1. Raw Source Emission (Outside ISC)

### Aggregation view
- Source emits raw accounts
- Attributes are source-native
- No ISC meaning yet

### Correlation view
- Correlation does not exist here
- Ownership is impossible at this stage

---

## 2. Connector Ingestion & Schema Normalization

### Aggregation
- Connector pulls raw objects
- Schema discovery maps attributes
- Account schema defines:
  - account ID
  - account name
  - attribute types

Result:
- Normalized **account objects**
- Still unowned

### Correlation
- Correlation still not active
- Only normalized account data exists

---

## 3. Account Object Persistence

### Aggregation
ISC decides:
- is this a new account?
- or an update to an existing account?

Decision based on:
- account ID
- account name

Result:
- Account object is created or updated
- No identity link yet

### Correlation
- Correlation still has not run
- Ownership is unknown

---

## 4. Identity Attribute Projection (Pre-Identity)

### Aggregation
For authoritative sources:
- account attributes map to identity attributes
- values are staged, not final

Important:
- mapping ≠ identity truth
- identity does not yet exist or update fully

### Correlation
- Correlation must wait
- Identity attributes are not reliable yet

---

## 5. Identity Processing (The Hard Boundary)

This is the **most important boundary** in ISC.

### Aggregation
Identity processing:
- creates identities
- updates identity attributes
- evaluates lifecycle state
- resolves authoritative truth

After this step:
- identities are real
- identifiers exist
- identity state is stable

### Correlation
This is the **earliest point correlation can safely run**.

Any correlation before this:
- will fail
- will fallback
- or will mis-attach

---

## 6. Correlation Engine Invocation

### Aggregation
- Aggregation job hands control to correlation
- One account at a time

### Correlation
For each account:
> “Which identity should own this account?”

Correlation now begins in earnest.

---

## 7. Decision Stack Execution

### Aggregation
Aggregation pauses logical control here.

### Correlation (runtime stack)
Evaluated top → down:

1. Manual correlation  
2. Correlation rules  
3. Correlation criteria  
4. Default fallback  

First decisive answer wins.

---

## 8. Identity Lookup (Dependency on Aggregation)

### Aggregation
- Identity store contains only identities created so far
- Late identities are invisible

### Correlation
- Searches identity store
- Uses current attribute values
- Cannot see future data

This is why sequencing matters more than configuration.

---

## 9. Correlation Outcome

### Aggregation
- Account object is updated with identity reference
- Or left uncorrelated

### Correlation
Possible outcomes:
- correlated to one identity
- uncorrelated
- never “best guess”

---

## 10. Persistence Boundary (Memory)

### Aggregation
- Account-identity link is persisted
- Becomes part of ISC truth

### Correlation
- Decision is remembered
- Delta aggregation reuses it
- Logic changes do not auto-reapply

This is intentional.

---

## 11. Manager Correlation (Parallel Identity Flow)

### Aggregation
- Identity attributes include manager reference

### Correlation
- Identity → identity resolution
- Depends entirely on aggregation order
- Manager identity must already exist

Failures here affect governance, not ownership.

---

## 12. Governance Consumption

### Aggregation
- Aggregation has finished its job

### Correlation
Governance trusts correlation blindly:
- access requests
- certifications
- policies
- reporting

If correlation is wrong, governance is wrong.

---

## 13. Delta vs Full Aggregation Impact

### Aggregation
- Delta skips unchanged accounts
- Full rebuilds state

### Correlation
- Delta preserves ownership
- Full allows re-evaluation

Most “why didn’t it change?” issues live here.

---

## 14. Unified End-to-End Timeline

**Source Data**  
→ Connector & Schema  
→ Account Object  
→ Identity Projection  
→ Identity Processing  
→ Correlation Decision  
→ Persistence  
→ Governance  

Aggregation owns everything **up to identity processing**.  
Correlation owns everything **after identity processing**.

---

## 15. The Golden Alignment Rule

> **If correlation surprises you, debug aggregation first.**  
Correlation only reveals problems that aggregation already created.

---

## One-Line Summary

> Aggregation creates identity reality.  
Correlation assigns ownership using whatever reality exists at that moment.

---

**Use this page when teaching, debugging, or designing ISC end to end.**  
It is the symmetry point between Aggregation and Correlation.
