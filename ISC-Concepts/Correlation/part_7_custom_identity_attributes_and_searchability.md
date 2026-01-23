# Part 7 – Custom Identity Attributes and Searchability

> **Purpose of this part**  
This chapter explains why correlation sometimes feels blocked even when attributes clearly exist.
The issue is usually not configuration logic, but **attribute availability and searchability**.

By the end of this part, you should:
- understand what “searchable” really means
- know why some attributes never appear in correlation dropdowns
- understand the timing between identity processing and correlation
- design custom attributes safely for correlation use

---

## 7.1 The confusion most teams hit

A common experience:

> “The attribute exists on the identity, I can see it in the UI, but I can’t use it for correlation.”

This is not a UI bug.
It is a **searchability and timing issue**.

Correlation does not use every identity attribute.
It uses only attributes that are designed to be searched efficiently at runtime.

---

## 7.2 What searchability actually means

When an identity attribute is marked **searchable**, ISC can:

- index it
- efficiently query identities by that value
- use it during aggregation-time correlation

Searchability is not cosmetic.
It directly controls whether correlation can use the attribute.

If an attribute is not searchable:
- it may exist
- it may be populated
- but correlation cannot reliably use it

---

## 7.3 Why correlation needs searchable attributes

During aggregation, correlation must answer:

> “Which identity has this value?”

This is a search problem.

Without searchability:
- lookups are expensive
- results are unreliable
- correlation performance degrades

So ISC restricts correlation to searchable attributes by design.

---

## 7.4 Built-in vs custom identity attributes

### Built-in attributes
Most core identity attributes:
- employeeId
- username
- email

are already searchable.

They are safe to use once lifecycle timing is correct.

### Custom attributes
Custom attributes:
- are not searchable by default
- must be explicitly configured
- require processing before use

This is where most issues arise.

---

## 7.5 Making a custom identity attribute usable

For a custom identity attribute to work in correlation, it must:

1. exist on the identity
2. be populated before aggregation
3. be marked searchable
4. be processed and indexed

Missing any step makes the attribute invisible to correlation.

---

## 7.6 The timing trap

Another common misunderstanding:

> “I added the attribute and ran aggregation, why didn’t it work?”

Because correlation saw the identity **before** the attribute was indexed.

Correct sequence:
1. create or update identity attribute
2. mark it searchable
3. run identity processing
4. verify values exist and are indexed
5. run aggregation

Skipping step 3 breaks correlation.

---

## 7.7 Identity processing vs aggregation

These are separate engines.

- identity processing updates identities
- aggregation evaluates accounts

Correlation happens during aggregation.
It uses the identity state produced by identity processing.

If identity processing hasn’t run:
- correlation sees old data
- new attributes appear “ignored”

---

## 7.8 Searchable does not mean safe

Just because an attribute is searchable does not mean it is safe.

Before using a custom attribute for correlation, verify:
- uniqueness guarantees
- lifecycle stability
- population timing

Searchability enables correlation.
Design determines correctness.

---

## 7.9 Limits and practical considerations

Custom searchable attributes should be used carefully.

Too many searchable attributes:
- increase index size
- slow searches
- complicate correlation design

Prefer:
- few, strong identifiers
- clear lifecycle ownership

---

## 7.10 Real-world example

A company creates a custom attribute `globalWorkerId`.

Mistake:
- attribute added
- immediately used in correlation
- aggregation runs
- no matches occur

Fix:
- run identity processing
- verify values indexed
- rerun aggregation

Correlation works without changing rules.

---

## 7.11 Debugging attribute availability

When an attribute does not work in correlation, check:

1. does the attribute exist on the identity?
2. is it populated before aggregation?
3. is it marked searchable?
4. has identity processing run?
5. does it appear in search results?

If any answer is no, correlation will fail.

---

## 7.12 Design guidance

Best practice:
- prefer built-in identifiers when possible
- introduce custom attributes only when necessary
- document why each searchable attribute exists
- review correlation usage periodically

Searchability is powerful.
Use it intentionally.

---

## 7.13 What this part prepares you for

After this part, you should:
- understand why attributes “disappear” from correlation
- design custom attributes with lifecycle awareness
- fix correlation blocks without changing rules

Next, we move into **manual correlation behavior and permanence**.

---

## Navigation

**⬅ Back:** - [Part 6 – Correlation Recommendations and Optimization Logic](part_6_correlation_recommendations_and_optimization.md)

**➡ Next:**  [Part 8 – Manual Correlation, Permanence, and Risk](part_8_manual_correlation_permanence_and_risk.md)

---

*End of Part 7*
