# 🧠 ISC Provisioning – Real-World Troubleshooting Drills

These drills are designed to train your thinking.

Do not jump to the answers.  
Read each scenario, pause, and predict what is happening using the provisioning loop:

**Decide → Plan → Execute → Verify → Enforce**

---

## 🔎 Drill 1: Approved but No Access

**Scenario:**  
A user requests an access profile.  
The request is approved.  
Provisioning shows success.  
The user logs in immediately and does not see access.

**Question:**  
What is the most likely reason?

---

## 🔎 Drill 2: Provisioning Ran Without Any Request

**Scenario:**  
No user submitted a request.  
No admin made a change.  
Provisioning activity appears for several users overnight.

**Question:**  
What triggered provisioning?

---

## 🔎 Drill 3: Manual Task Completed but Access Missing

**Scenario:**  
A manual provisioning task was assigned to an admin.  
The admin marked the task complete.  
Provisioning shows completed.  
Aggregation later removes the access.

**Question:**  
Why did this happen?

---

## 🔎 Drill 4: Error Repeats Every Hour

**Scenario:**  
A provisioning action fails.  
The same error appears again every hour.  
No new approvals were given.

**Question:**  
Why is the error repeating?

---

## 🔎 Drill 5: Access Keeps Reappearing After Manual Removal

**Scenario:**  
An admin removes access directly in the target system.  
After aggregation, the access comes back.

**Question:**  
Why is ISC re-adding the access?

---

## 🔎 Drill 6: Partial Success

**Scenario:**  
A role assignment provisions access to two systems.  
System A succeeds.  
System B fails.

**Question:**  
How does ISC record this and what happens next?

---

## 🔎 Drill 7: Ticket Closed but Access Missing

**Scenario:**  
Provisioning is routed through a ticketing system.  
The ticket is marked closed.  
Access is still missing.

**Question:**  
Why does ISC still show provisioning complete?

---

# ✅ Answers and Explanations

---

## ✔️ Drill 1 – Answer

Most likely, verification has not occurred yet.

Approval allowed execution, but aggregation has not confirmed the change.  
Provisioning success does not guarantee immediate visibility.

---

## ✔️ Drill 2 – Answer

This was data-driven provisioning.

Common triggers include lifecycle state changes, role recalculation, or identity refresh.  
No human action is required for provisioning to run.

---

## ✔️ Drill 3 – Answer

Manual task completion is a declaration, not verification.

Aggregation later detected that the required access was not actually present,  
so enforcement corrected the drift.

---

## ✔️ Drill 4 – Answer

This is a retryable failure.

ISC automatically retries execution without new approvals.  
The intent has not changed, only execution is repeating.

---

## ✔️ Drill 5 – Answer

The access is enforced by a role or lifecycle state.

Verification detected drift and provisioning re-ran to restore the desired state.

---

## ✔️ Drill 6 – Answer

ISC records partial success at the action level.

System A is marked successful.  
System B is marked failed and may retry depending on error type.

---

## ✔️ Drill 7 – Answer

ISC tracks ticket state, not access reality.

Ticket closure signals workflow completion, not technical verification.  
Aggregation later determines the actual access state.

---

## 🔗 Navigation

- **Previous:** [Phase 7 – End-to-End Scenarios](./ISC_Provisioning_Phase_7_End_to_End_Scenarios.md)
- **Next:** [Day-to-Day Admin Troubleshooting Playbook](./ISC_Provisioning_Day_to_Day_Admin_Troubleshooting_Playbook.md)
