# Part 3 of 4  
## Investigation Workflows Using Search in Identity Security Cloud

---

## 📌 What this part is about

In Part 1, you learned **how Search is structured**.  
In Part 2, you learned **how to ask precise questions** using queries.

Part 3 focuses on **using Search to investigate real situations**.

This part explains:
- How to trace access requests end to end
- How to read Events as a timeline
- How to use Account Activity to understand provisioning behavior
- How different Search categories work together to tell one story

The goal is to move from “I can write queries” to “I can explain what happened.”

---

## 1️⃣ The investigation mindset

Every investigation starts with a **clear question**.

Examples:
- Why does this user not have access?
- Did this request get approved?
- Why did provisioning fail?
- Who changed this access and when?

Search helps answer these questions by showing:
- Current state (what exists now)
- Historical actions (what happened)
- Execution details (what the system tried to do)

No single category gives the full picture.  
Investigations always involve **multiple categories**.

---

## 2️⃣ The three core investigation categories

Almost all investigations revolve around these three:

1. **Identities** – current access state  
2. **Events** – actions and decisions  
3. **Account Activity** – provisioning execution  

Understanding how these fit together is critical.

---

## 3️⃣ Workflow 1: User reports missing access

### Step 1: Start with the Identity

The identity view answers:
- What access does the user currently have?
- Which roles or access profiles are present?
- Are there correlated accounts?

This establishes the **current state**.

If access is already present here, the issue may be outside Search.

---

### Step 2: Check Events (history)

If access is missing:
- Search Events for access request activity
- Look for request submission, approval, or rejection

Events answer:
- Was access requested?
- Was it approved or denied?
- Who took the action?
- When did it happen?

This builds a **timeline of decisions**.

---

### Step 3: Check Account Activity (execution)

If Events show approval:
- Move to Account Activity

Account Activity answers:
- Was provisioning attempted?
- Did it succeed or fail?
- What error was returned by the source?

This is where technical failures are visible.

---

## 4️⃣ Workflow 2: Investigating failed provisioning

Provisioning failures are best understood through Account Activity.

Account Activity records:
- operation type (create, modify, disable)
- status (success, failure)
- detailed error messages

Steps:
1. Identify the identity
2. Review recent Account Activity
3. Look for failed actions
4. Correlate timestamps with Events

Events provide context.  
Account Activity provides detail.

---

## 5️⃣ Workflow 3: Verifying access request processing

Access requests generate **multiple Events**.

Typical sequence:
1. Request submitted
2. Approval or rejection
3. Request processed
4. Provisioning attempt

By reviewing Events:
- You can confirm each stage
- Identify where the process stopped

If processing completed but access is missing:
- Account Activity explains why

---

## 6️⃣ Understanding Event timelines

Events are chronological records.

Each event includes:
- action type
- actor
- target
- timestamp

Reading Events as a sequence helps you:
- reconstruct decisions
- explain outcomes
- provide audit evidence

Events answer *what happened*, not *why it failed*.

---

## 7️⃣ When to switch from Events to Account Activity

A simple rule:

- If the question is **“Did it happen?”** → Events  
- If the question is **“Why didn’t it work?”** → Account Activity  

Switching too early or too late leads to confusion.

---

## 8️⃣ Workflow 4: Investigating unexpected access

Sometimes access exists but should not.

Steps:
1. Confirm access via Identity search
2. Identify how access was granted
   - role membership
   - access profile
   - direct entitlement
3. Use Events to trace the grant
4. Use timestamps to identify the source

This helps determine whether access came from:
- role logic
- manual request
- automated lifecycle action

---

## 9️⃣ Common investigation mistakes

- Looking only at current state and ignoring history
- Relying on Events alone for provisioning failures
- Ignoring timestamps
- Forgetting that indexing delays exist

Avoiding these mistakes saves time and confusion.

---

## 🔁 Part 3 recap

After this part, you should be able to:
- Trace access requests end to end
- Read Events as a timeline
- Use Account Activity to diagnose failures
- Combine multiple categories into one explanation

---

## 🧠 Self-check

You should be able to answer:
1. Why are Events and Account Activity both needed?
2. What is the first category to check for missing access?
3. Where do provisioning error details live?
4. How do timestamps help investigations?

---

## ⏭️ What comes next (Part 4 preview)

Part 4 covers:
- Saved Searches
- Subscriptions
- Reporting and exports
- Audit-ready evidence

This completes the Search & Reporting series.

---

## 🔗 Navigation

⬅️ **Back:** [Part 2 – Query Building and Operators](ISC_Search_Part_2_Query_Building.md)  
➡️ **Next:** Part 4 – Reporting and Saved Searches
