# 🌳 ISC Provisioning – One-Page Troubleshooting Decision Tree

Use this decision tree when something looks wrong.  
Follow it **top to bottom**. Do not jump steps.

---

## 🔁 Core Loop

**Decide → Plan → Execute → Verify → Enforce**

Keep this loop in mind while walking through the questions.

---

## ▶️ START HERE

### ❓ Q1. Did something trigger provisioning?

- **Yes** → Go to **Q2**  
- **No / Unsure** → Check identity changes (attributes, lifecycle, role evaluation, certification)

---

## 🚦 TRIGGER

### ❓ Q2. What kind of trigger was it?

- **Request-driven** (user/admin request) → Go to **PLAN**  
- **Data-driven** (lifecycle, role, refresh, certification) → Go to **PLAN**  

---

## 🧩 PLAN

### ❓ Q3. Was a provisioning plan created?

- **Yes** → Go to **EXECUTION**  
- **No** → No mismatch detected (no-op). **Stop here.**

---

## ⚙️ EXECUTION

### ❓ Q4. How was the plan executed?

- **Direct connector** → Go to **Q5**  
- **Manual task** → Go to **Q6**  
- **Ticketing system** → Go to **Q7**  

---

## 🔌 DIRECT CONNECTOR PATH

### ❓ Q5. Did execution succeed?

- **Yes** → Go to **VERIFICATION**  
- **Failed (retryable)** → Wait or fix blocker  
- **Failed (non-retryable)** → Fix data/config, then re-trigger  

---

## 🧑‍💻 MANUAL TASK PATH

### ❓ Q6. Was the task completed correctly?

- **Yes** → Go to **VERIFICATION**  
- **No / Unsure** → Fix manually, then re-run aggregation  

---

## 🎫 TICKETING PATH

### ❓ Q7. Is the ticket closed?

- **Yes** → Go to **VERIFICATION**  
- **No** → Execution still pending. **Stop here.**

---

## 🔍 VERIFICATION

### ❓ Q8. Has aggregation run?

- **Yes** → Go to **Q9**  
- **No** → Run or wait for aggregation  

---

## 🧪 STATE CHECK

### ❓ Q9. Does the target system reflect the expected access?

- **Yes** → ✅ **Done**  
- **No** → Go to **ENFORCEMENT**  

---

## 🔁 ENFORCEMENT

### ❓ Q10. Is access enforced by role, lifecycle, or attribute sync?

- **Yes** → Change the **rule**, not the entitlement  
- **No** → Investigate execution failure or incorrect data  

---

## 🛑 FINAL RULES (READ EVERY TIME)

- Approval ≠ Execution  
- Execution ≠ Verification  
- Ticket closed ≠ Access exists  
- Manual change ≠ Desired state  
- If access comes back, **something is enforcing it**  

---

## 🔗 Navigation

- **Previous:** [Day-to-Day Admin Troubleshooting Playbook](./ISC_Provisioning_Day_to_Day_Admin_Troubleshooting_Playbook.md)
