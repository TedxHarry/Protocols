# Part 1 of 4  
## Search Foundations and Data Flow in Identity Security Cloud

---

## 📌 What this part is about

This part builds the **foundation** for everything that follows in the Search & Reporting series.

Before learning how to write queries, investigate issues, or generate reports, it is important to understand:
- What Search actually represents inside Identity Security Cloud
- How data flows into Search behind the scenes
- Why Search is divided into categories
- Why delays, timing differences, and retention limits exist

This part focuses on **structure and reasoning**, not syntax.  
If these concepts are clear, later parts will feel intuitive instead of mechanical.

---

## 1️⃣ What Search is (and what it is not)

Search is often misunderstood as the place where data is created or controlled.  
That is not its role.

In Identity Security Cloud, data already exists before Search ever touches it.  
This data is produced by:
- identity aggregation
- access modeling and role logic
- provisioning actions
- access requests and approvals
- certifications and lifecycle processes

**Search does not create or modify this data.**

Instead, Search acts as an **indexing and visibility layer** that makes existing data:
- fast to retrieve
- easy to filter
- usable for investigations and reporting

A simple way to think about it:

> Data is created by system processes first.  
> Search makes that data visible and searchable later.

Understanding this prevents confusion when Search does not immediately reflect a recent change.

---

## 2️⃣ The end-to-end data flow behind Search

To understand Search behavior, it helps to visualize the full data pipeline.

```
External Sources (HR systems, AD, Applications)
            ↓
Aggregation, Provisioning, Requests, Lifecycle Processing
            ↓
ISC Internal Data Stores
  - Identity records
  - Access definitions (roles, profiles, entitlements)
  - Event history
  - Account activity (execution details)
            ↓
Propagation and Indexing
            ↓
Search Interface (categories and tabs)
```

Key points:
- Data must first **enter** Identity Security Cloud
- Then it must be **indexed**
- Only after indexing does Search return it

Any delay in this pipeline affects what Search can show.

---

## 3️⃣ Why Search is divided into categories

Search is not one flat dataset.

Identity Security Cloud manages different kinds of data, each with:
- different structure
- different fields
- different update frequency
- different retention rules

Because of this, Search organizes data into **categories**, each representing a distinct dataset.

The primary searchable categories are:
- Identities
- Roles
- Access Profiles
- Entitlements
- Events
- Account Activity

Each category answers a different type of question.  
Understanding this prevents mixing unrelated data during investigations.

---

## 4️⃣ Understanding each Search category

### 🔹 Identities

The Identities category represents **people and their current state** in the system.

It includes:
- identity attributes such as department, location, manager
- lifecycle state
- correlated accounts
- access relationships derived from roles and access profiles

Identity data is influenced by:
- authoritative sources
- correlation rules
- identity profiles and mappings
- identity refresh operations

This category answers questions like:
- Who is this person?
- What state are they in right now?
- What access do they currently have?

It reflects **current reality**, not history.

---

### 🔹 Roles, Access Profiles, and Entitlements

These categories represent the **access model**.

- Entitlements originate from connected sources
- Access Profiles bundle entitlements for provisioning and requests
- Roles group access for business or organizational use

Searching these categories helps you:
- understand what access exists
- see how access is grouped and exposed
- validate that access modeling behaves as expected

They describe **what access is defined**, not who has acted on it.

---

### 🔹 Events

Events record **actions and decisions** that occurred in the system.

Examples include:
- access request submission, approval, or rejection
- authentication events
- password changes
- certification decisions

Each event captures:
- what happened
- who performed the action
- which object was affected
- when it occurred

Events form a **historical timeline** and are commonly used for audits and reviews.

---

### 🔹 Account Activity

Account Activity records **what Identity Security Cloud attempted to do on source accounts**.

This includes:
- account creation
- attribute updates
- enable or disable actions
- password operations
- lifecycle-driven changes

Account Activity shows:
- whether an action succeeded or failed
- detailed error information returned by the source

This category provides the most detailed view of provisioning execution.

---

## 5️⃣ Current-state data vs historical data

Search categories naturally fall into two groups.

### Current-state data
- Identities
- Roles
- Access Profiles
- Entitlements

These describe what exists **right now**.

### Historical data
- Events
- Account Activity

These describe what happened **over time**.

This distinction is essential when interpreting results.  
Current-state data answers *what is true*.  
Historical data explains *how it became true*.

---

## 6️⃣ Default search behavior

When a search term is entered without filters:
- Search evaluates multiple categories
- Results are separated into tabs by category

This is why:
- the same keyword can appear under identities, access objects, and events
- switching tabs changes context without changing the query

Understanding this behavior avoids misinterpretation of results.

---

## 7️⃣ Why Search results may not appear immediately

There are two common causes.

### Indexing delay
After data enters the system:
- it must propagate
- it must be indexed

This can take minutes, especially in active environments.

### Scheduled processing
Some changes depend on scheduled jobs, such as:
- source configuration updates
- entitlement changes
- access application updates

These changes may not appear until scheduled synchronization completes.

Delays are expected behavior, not errors.

---

## 8️⃣ Time and retention considerations

### Time zones
- Search evaluates date filters using GMT
- Displayed timestamps follow the browser’s local time

### Retention
- Event data is retained for longer periods
- Account Activity data has shorter retention
- Pending items may persist beyond standard limits

This affects how far back investigations and audits can go.

---

## 9️⃣ Simple end-to-end example

A user reports that required access is missing.

A structured approach using Search:
1. Identity search shows the user’s current access state
2. Events show whether access was requested and approved
3. Account Activity shows whether provisioning succeeded or failed

Each category contributes a different piece of the explanation.

---

## 🔁 Part 1 recap

After completing this part, you should understand:
- Search is an indexing and visibility layer
- Data flows through multiple internal stages before appearing in Search
- Categories exist because data types differ
- Current-state and historical data serve different purposes
- Delays and retention limits are normal and expected

---

## ⏭️ What comes next

Part 2 focuses on **query building and operators**, showing how to ask precise questions once the structure is understood.

---

## 🔗 Navigation

⬅️ **Back:** _Start of Search & Reporting Series_  
➡️ **Next:** [Part 2 – Query Building and Operators](ISC_Search_Part_2_Query_Building.md)
