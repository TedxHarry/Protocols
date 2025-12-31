# Part 2 of 4  
## Query Building and Operators in Identity Security Cloud

---

## 📌 What this part is about

In Part 1, you learned:
- What Search indexes
- How data flows into Search
- Why Search is divided into categories
- The difference between current-state and historical data

Now the focus shifts to **how Search understands questions**.

This part explains:
- How queries are structured
- Why some searches feel imprecise
- How operators change results
- Why nested queries are required for real-world questions

The goal is not memorization, but confidence.

---

## 1️⃣ How Search interprets a query

Every Search query is built from three elements:

1. **Field** – where Search looks  
2. **Value** – what Search matches  
3. **Logic** – how multiple conditions relate  

At its simplest, the structure is:

```
field:value
```

Think of it like asking a very specific question:
- Not “find users”
- But “find users where *this exact attribute* equals *this value*”

This structure applies consistently across identities, access objects, events, and account activity.

---

## 2️⃣ Free-text search vs field-based search

### Free-text search

When you type a word without specifying a field:
- Search looks across many fields
- Across multiple categories
- And returns anything that loosely matches

Example situation:
> You search for **Finance** and get identities, roles, access profiles, and events.

This is useful when:
- You are exploring data
- You are not sure where information lives yet

But free-text search is **broad by design**.

---

### Field-based search (precision searching)

Field-based search tells Search:
- Exactly which attribute to evaluate
- Exactly what value must match

Conceptually:
```
specific attribute : specific value
```

Example in plain language:
> Look only at the department attribute and return identities where department equals Finance.

Why this matters:
- Results are predictable
- Queries scale as data grows
- Troubleshooting becomes reliable

As environments mature, field-based search becomes the default approach.

---

## 3️⃣ Understanding fields and hierarchy

Fields represent **attributes of an object**.

Some fields:
- Belong directly to the object (for example, identity name)
- Belong to related objects (for example, manager name)

This is why fields often look hierarchical:

```
object.subfield
```

Plain explanation:
> You are telling Search exactly *which piece of data* to inspect.

If a field does not exist for a category, the query will not work there.

This reinforces why understanding categories from Part 1 is essential.

---

## 4️⃣ Phrase searching (why quotes change results)

Search treats words differently depending on quotation marks.

Without quotes:
- Words are evaluated independently
- Partial matches are common

With quotes:
- The entire phrase is treated as one value
- Matches are exact

Example concept:
> Searching for New York without quotes may match New or York separately.  
> Using quotes forces Search to match the full phrase.

Use quotes whenever:
- Values contain spaces
- Accuracy matters

---

## 5️⃣ Logical operators: shaping results

Search supports three operators:
- **AND**
- **OR**
- **NOT**

They must be written in uppercase.

---

### AND (narrowing)

AND means **all conditions must be true**.

Plain-language example:
> Find identities in Finance **and** in Active lifecycle state.

Each additional AND condition reduces results.

Use AND when refining results step by step.

---

### OR (broadening)

OR means **any condition can be true**.

Plain-language example:
> Find identities in Finance **or** HR.

OR expands the result set.

Use OR when alternatives are acceptable.

---

### NOT (excluding)

NOT removes matches.

Plain-language example:
> Find identities in Finance **but not** contractors.

NOT is powerful but should be used carefully to avoid unintended exclusions.

---

## 6️⃣ Parentheses: controlling logic order

When multiple operators are used together:
- Search evaluates expressions from left to right
- Parentheses override that order

Conceptual example:
> (Finance OR HR) AND Active

Without parentheses, Search might interpret the logic differently.

Parentheses ensure your intent is clear and unambiguous.

---

## 7️⃣ Categories and field availability

Fields are **category-specific**.

That means:
- A field valid in Identity search may not exist in Event search
- The same query can fail or succeed depending on category

Before writing complex queries:
- Confirm which category you are searching
- Confirm which fields belong to that category

This prevents silent failures and confusion.

---

## 8️⃣ Why nested queries exist (slow and clear explanation)

Some attributes are **single-value**:
- lifecycle state
- department

Others are **multi-value lists**:
- accounts
- access items
- provisioning requests

Example:
> One identity can have multiple accounts on different sources.

If identities had only one account:
- Simple field searches would be enough

But because identities can have many accounts:
- Search must evaluate **each account individually**

That is why nested queries exist.

---

### Conceptual difference

Non-nested thinking:
> Check the identity

Nested thinking:
> Check every account inside the identity

If **any one item** inside the list matches:
- The identity is returned

This is critical for accurate access and provisioning analysis.

---

## 9️⃣ When nested queries are required

You should think about nesting when your question includes:
- “Has any account…”
- “Has at least one access…”
- “Any provisioning attempt…”

Examples in plain language:
- Does this identity have **any failed account update**?
- Does this identity have **any privileged access**?

These questions cannot be answered without nesting.

---

## 10️⃣ Wrong approach vs correct approach (conceptual contrast)

### Common mistake
> Try to search account details as if they were single identity attributes.

Result:
- No matches
- Confusing results

### Correct approach
> Acknowledge that accounts and access are lists  
> Use nested logic to evaluate each item

This shift in thinking is essential for mastery.

---

## 🔁 Part 2 recap

After this part, you should understand:
- How Search evaluates queries
- Why field-based search is more reliable
- How AND, OR, and NOT change results
- Why parentheses matter
- Why nested queries exist and when they are required

---

## 🧠 Self-check (confidence test)

You should be able to answer:
1. When is free-text search appropriate?
2. Why might a field work in one category but not another?
3. Why do identities with multiple accounts require nested queries?
4. How can parentheses change query meaning?

If these make sense, you’re ready to move on.

---

## ⏭️ What comes next (Part 3 preview)

Part 3 applies everything learned so far to:
- Access request tracing
- Event timelines
- Provisioning failure analysis

This is where understanding becomes problem-solving.

---

## 🔗 Navigation

⬅️ **Back:** [Part 1 – Search Foundations and Data Flow](ISC_Search_Part_1_Foundations.md)  
➡️ **Next:** Part 3 – Investigation Workflows

