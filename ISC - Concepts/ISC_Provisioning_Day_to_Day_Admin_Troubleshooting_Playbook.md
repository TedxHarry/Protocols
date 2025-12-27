# 🛠️ ISC Provisioning – Day-to-Day Admin Troubleshooting Playbook

This playbook is written from the perspective of a working ISC administrator.

These are not theoretical problems. These are the issues that show up in real environments, with clear thinking steps to resolve them confidently.

Always reason using the provisioning loop:

**Decide → Plan → Execute → Verify → Enforce**

---

## 🚨 Issue 1: User Says “I Got Approval but Still No Access”

**Symptoms:**
- Request approved  
- Provisioning shows success  
- User still blocked  

**Root Cause Pattern:**  
Execution completed, verification has not yet confirmed reality.

**Troubleshooting Steps:**
1. Confirm provisioning execution status  
2. Identify fulfillment channel used  
3. Check if aggregation has run  
4. Verify access directly in target system  

**Key Insight:**  
Approval allows execution. Verification confirms reality.

---

## 🌙 Issue 2: Provisioning Runs Overnight Without Requests

**Symptoms:**
- Provisioning activity overnight  
- No requests submitted  

**Root Cause Pattern:**  
Data-driven triggers such as role recalculation or lifecycle enforcement.

**Troubleshooting Steps:**
1. Identify what identity attributes changed  
2. Check lifecycle state enforcement  
3. Review role membership criteria  
4. Confirm scheduled identity refresh  

**Key Insight:**  
Provisioning reacts to state drift, not user clicks.

---

## 🔁 Issue 3: Access Removed but Comes Back After Aggregation

**Symptoms:**
- Admin removes access manually  
- Access reappears after aggregation  

**Root Cause Pattern:**  
State-based enforcement by role or lifecycle state.

**Troubleshooting Steps:**
1. Identify access source (role vs lifecycle)  
2. Check active lifecycle state requirements  
3. Verify role assignment logic  
4. Remove enforcing source, not the entitlement  

**Key Insight:**  
You must change the rule, not the result.

---

## 🧑‍💻 Issue 4: Manual Task Marked Complete but Access Missing

**Symptoms:**
- Task completed  
- Provisioning marked complete  
- Access missing  

**Root Cause Pattern:**  
Manual fulfillment is declarative, not verified.

**Troubleshooting Steps:**
1. Verify change directly in target system  
2. Run aggregation  
3. Confirm correct entitlement was applied  
4. Correct and re-run provisioning if needed  

**Key Insight:**  
Humans can mark tasks complete incorrectly.

---

## ⏱️ Issue 5: Same Error Repeats Every Hour

**Symptoms:**
- Identical error recurring  
- No new approvals  

**Root Cause Pattern:**  
Retryable execution failure.

**Troubleshooting Steps:**
1. Identify error type  
2. Check connector health  
3. Validate network connectivity  
4. Wait for retries or fix blocking issue  

**Key Insight:**  
Retries repeat execution, not intent.

---

## ⚖️ Issue 6: Partial Success Across Systems

**Symptoms:**
- Access exists in one system  
- Missing in another  

**Root Cause Pattern:**  
Multi-action provisioning plan with mixed outcomes.

**Troubleshooting Steps:**
1. Identify which action failed  
2. Determine retryable vs non-retryable  
3. Correct data if needed  
4. Allow retry or re-trigger provisioning  

**Key Insight:**  
Provisioning success is action-level, not plan-level.

---

## 🎫 Issue 7: Ticket Closed but Access Still Missing

**Symptoms:**
- Ticket closed  
- User lacks access  

**Root Cause Pattern:**  
Ticket workflow completed, not technical verification.

**Troubleshooting Steps:**
1. Confirm what work was performed  
2. Verify access in target system  
3. Run aggregation  
4. Correct and re-open if needed  

**Key Insight:**  
ISC tracks workflow state, not correctness.

---

## 👤 Issue 8: Account Not Created When Access Granted

**Symptoms:**
- Access granted  
- No account exists  

**Root Cause Pattern:**  
Create Account definition incomplete or invalid.

**Troubleshooting Steps:**
1. Validate create account attribute mappings  
2. Check required attributes  
3. Review identity data completeness  
4. Re-trigger provisioning  

**Key Insight:**  
Access cannot exist without an account container.

---

## 🔄 Issue 9: Lifecycle Change Does Not Remove Access

**Symptoms:**
- Lifecycle state changed  
- Access still present  

**Root Cause Pattern:**  
Access granted by overlapping lifecycle or role.

**Troubleshooting Steps:**
1. Identify all access-granting sources  
2. Compare old vs new lifecycle state  
3. Review role overlap  
4. Adjust access model  

**Key Insight:**  
Removal requires all grant paths to be removed.

---

## 🧬 Issue 10: Attribute Sync Keeps Overwriting Values

**Symptoms:**
- Attribute changes revert automatically  

**Root Cause Pattern:**  
Attribute sync enforcing identity as source of truth.

**Troubleshooting Steps:**
1. Identify synced attributes  
2. Check identity attribute source  
3. Confirm business ownership of data  
4. Adjust sync configuration  

**Key Insight:**  
Attribute sync enforces consistency, not flexibility.

---

## 🧠 Final Admin Mental Checklist

Before escalating any issue, always ask:

- What triggered provisioning?  
- What was the intended state?  
- How was execution performed?  
- Has verification occurred?  
- Is enforcement correcting drift?  

If you can answer these, you are operating at a **senior level**.

---

## 🔗 Navigation

- **Previous:** [Real-World Troubleshooting Drills](./ISC_Provisioning_Real_World_Troubleshooting_Drills.md)
- **Next:** [One-Page Troubleshooting Decision Tree](./ISC_Provisioning_One_Page_Troubleshooting_Decision_Tree.md)
