# Part 4 of 4  
## Reporting and Saved Searches in Identity Security Cloud

---

## 📌 What this part is about

In Parts 1–3, you learned:
- How Search is structured and indexed
- How to build precise queries
- How to investigate real scenarios using Search

Part 4 completes the learning by explaining **how Search results are turned into reusable assets and evidence**.

This part covers:
- Saved Searches and why they matter
- Subscriptions and automated monitoring
- Search-based reporting
- How to produce audit-ready evidence
- Common pitfalls in reporting

The goal is to move from *investigating once* to *monitoring continuously*.

---

## 1️⃣ Why saved searches exist

Running the same query repeatedly is inefficient.

Saved Searches allow you to:
- Preserve complex queries
- Reuse them consistently
- Reduce errors caused by rewriting queries
- Establish repeatable checks

Saved searches represent **known questions with known answers**.

---

## 2️⃣ What a saved search really is

A saved search is:
- A stored query
- Bound to a specific search category
- Visible only to the user who created it

Important characteristics:
- The query itself cannot be edited after saving
- To change logic, a new saved search must be created
- Saved searches are personal, not global

This design prevents accidental changes to trusted queries.

---

## 3️⃣ When to use saved searches

Saved searches are useful when:
- You repeatedly check the same condition
- You monitor risk-prone access
- You validate governance controls
- You prepare recurring reports

Examples:
- Identities with privileged access
- Failed provisioning attempts
- Users bypassing lifecycle rules
- Accounts still active after termination

---

## 4️⃣ Subscriptions: automated awareness

Saved searches can be **subscribed** on a schedule.

Subscription options typically include:
- Daily
- Weekly
- Monthly

Subscriptions:
- Automatically run the saved search
- Email the results to recipients

This allows Search to act as a **monitoring mechanism**, not just a lookup tool.

---

## 5️⃣ Subscription considerations

Subscriptions require careful handling because:
- Search results may include sensitive access data
- Email distribution increases exposure

Best practices:
- Limit recipients
- Avoid unnecessary detail
- Use summaries where possible

Report links sent by email typically have an expiration period.

---

## 6️⃣ Search-based reporting

Reports in Identity Security Cloud are built directly from Search queries.

General flow:
1. Run a search
2. Select a category
3. Choose result columns
4. Generate a report
5. Download results

Reports are not separate from Search.  
They are **Search results formatted for export**.

---

## 7️⃣ On-screen results vs downloaded reports

Important difference:
- On-screen results may be limited
- Downloaded reports contain the full dataset

When exporting:
- Large result sets are common
- Some reports expand rows (for example, access details)

Understanding this prevents confusion when row counts change.

---

## 8️⃣ Common report types

Common search-based reports include:
- Access Request Activity
- Authentication Activity
- Password Changes
- Provisioning Activity
- Certification Outcomes

These are predefined queries that can be refined as needed.

---

## 9️⃣ Reporting for audits

Audit-focused reports emphasize:
- Accuracy
- Completeness
- Time range
- Traceability

Auditors typically care about:
- Who approved access
- When it was approved
- Whether provisioning succeeded
- Evidence of enforcement

Search-based reports provide this evidence.

---

## 10️⃣ Retention and reporting limits

Not all data is retained equally.

Key reminders:
- Event data is retained longer
- Account Activity data has shorter retention
- Older execution details may not be available

For long-term audits:
- Reports should be generated periodically
- Saved searches help preserve evidence

---

## 11️⃣ Common reporting mistakes

- Exporting without verifying filters
- Misinterpreting expanded rows
- Assuming on-screen counts equal exported counts
- Using Account Activity for long-term audits

Avoiding these mistakes improves credibility.

---

## 🔁 Part 4 recap

After this part, you should understand:
- Why saved searches exist
- How subscriptions enable monitoring
- How reports are generated from Search
- How to prepare audit-ready evidence
- Where reporting limitations exist

---

## 🧠 Final self-check

You should be able to answer:
1. When should a query be saved?
2. What are the risks of subscriptions?
3. Why can exported row counts differ from on-screen results?
4. Which data type is better for audits and why?

---

## 🎯 Series completion

You have now completed all four parts of **Search & Reporting in Identity Security Cloud**:

- Part 1 – Foundations and Data Flow  
- Part 2 – Query Building and Operators  
- Part 3 – Investigation Workflows  
- Part 4 – Reporting and Saved Searches  

Together, these parts provide a complete, structured understanding of the topic.

---

## 🔗 Navigation

⬅️ **Back:** [Part 3 – Investigation Workflows](ISC_Search_Part_3_Investigation_Workflows.md)  
➡️ **Next:** _End of Search & Reporting Series_
