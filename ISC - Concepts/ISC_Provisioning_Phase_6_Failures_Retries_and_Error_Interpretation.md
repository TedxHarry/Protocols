# ❌ ISC Provisioning – Phase 6: Failures, Retries, and Error Interpretation

Phases 1–5 explained what provisioning is, why it runs, what it plans, how it executes, and how reality is verified.

Phase 6 explains what happens when things do not go as planned, and how to reason about it **without guessing**.

---

## 🧠 Failure Handling in One Sentence

Provisioning failures describe how an execution attempt ended, not whether the original decision was correct.

---

## 🎯 Why Failures Are Normal

Provisioning interacts with external systems that Identity Security Cloud does not control.

- Networks fail  
- APIs time out  
- Accounts already exist  
- Humans make mistakes  

Failure handling exists to absorb this reality safely and transparently.

---

## 🔁 Retryable vs Non-Retryable Failures

Not all failures are equal.

### Retryable failures indicate temporary problems, such as:
- Network issues  
- Target system unavailability  
- Transient connector errors  

### Non-retryable failures indicate logical or data problems, such as:
- Invalid attribute values  
- Missing required data  
- Unsupported operations  

---

## 📦 Mental Model: Package Delivery Attempt

A failed delivery attempt does not cancel the order.

- **Retryable failure:** road temporarily blocked, try again later  
- **Non-retryable failure:** address does not exist, needs correction  

Provisioning behaves the same way.

---

## 🔄 Automatic Retries

Identity Security Cloud automatically retries retryable failures.

- Retries do not require new approvals  
- Retries do not represent new intent  

They represent repeated execution attempts of the **same provisioning plan**.

---

## ⚖️ Partial Success

A single provisioning plan may contain multiple actions.

Some actions may succeed while others fail.

Partial success is expected and recorded at the **action level**, not the plan level.

---

## 🧑‍💻 Manual Failures

Manual provisioning tasks introduce a different failure mode.

A task can be marked complete even if the change was done incorrectly.

Verification later detects the mismatch.

---

## 🕰️ System State vs Error History

**Errors** describe execution attempts.  
**State** describes current reality.

- A failed execution does not always mean state is incorrect  
- A successful execution does not always mean state is correct  

---

## 😅 This Feels Wrong but Is Expected

- Errors may repeat without user action  
- Errors may disappear after retries  
- Errors may exist even when access looks correct  

This reflects separation of **intent**, **execution**, and **verification**.

---

## 🛠️ Beginner Troubleshooting Framework

When you see a failure, reason in this order:

1. Is this retryable or non-retryable?  
2. Is the failure temporary or data-related?  
3. Did any actions succeed?  
4. Has verification corrected or confirmed state?  
5. Does configuration need correction, or should the system retry?  

---

## 🗣️ Explaining Failures to Others

**User explanation:**  
“Your access is pending while the system retries a temporary issue.”

**Admin explanation:**  
“Execution failed due to a retryable connector error.”

**Auditor explanation:**  
“The system recorded execution attempts, outcomes, and verification results.”

---

## ✅ Pause and Confirm Understanding

You should now be able to:
- Classify failures correctly  
- Predict retry behavior  
- Explain partial success  
- Troubleshoot without guessing  

If this is clear, you are ready for the final phase.

---

## ➡️ What Comes Next

**Phase 7** will unify everything into end-to-end scenarios.

You will practice predicting outcomes **before they happen**.

---

## 🔗 Navigation

- **Previous:** [Phase 5 – Verification and Aggregation](./ISC_Provisioning_Phase_5_Verification_and_Aggregation.md)
- **Next:** [Phase 7 – End-to-End Scenarios](./ISC_Provisioning_Phase_7_End_to_End_Scenarios.md)
