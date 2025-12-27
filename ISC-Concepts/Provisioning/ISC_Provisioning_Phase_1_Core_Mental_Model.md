# 🧠 ISC Provisioning – Phase 1: Core Mental Model

This phase builds the foundation for understanding provisioning in Identity Security Cloud. Everything you learn later depends on this mental model. If this is clear, provisioning becomes predictable rather than confusing.

---

## ✅ Provisioning in One Sentence

Provisioning is the moment Identity Security Cloud sends instructions to a target system to make access match a decision.

---

## ⚙️ What Provisioning Really Is

Provisioning is execution.

It is not approval.  
It is not policy.  
It is not governance.

Provisioning happens when Identity Security Cloud attempts to create, modify, enable, disable, or remove something in a real downstream system.

---

## 🎯 Why Provisioning Exists

Organizations are usually good at deciding who should have access.

They are not good at making every system consistently reflect those decisions.

Provisioning exists to close that gap between decision and reality.

---

## 🧩 Where Provisioning Fits in the Bigger Picture

Provisioning is one step in a larger flow:

**Decision → Change Plan → Provisioning → Verification → Identity State Update**

Provisioning is responsible only for the execution step.  
Everything before it decides what should happen.  
Everything after it checks what actually happened.

---

## 🚚 Mental Model: The Shipping Company

Think of Identity Security Cloud as a shipping company.

A business decision creates a shipping order.  
Provisioning is the truck leaving the warehouse.

Approvals decide whether a package should be shipped.  
Provisioning is the act of delivering it.

If the road is blocked or the address is wrong, the truck still attempted delivery. That attempt is always recorded.

---

## 📦 What Provisioning Actually Sends

Provisioning does not magically change systems.

It sends structured instructions through:
- A connector  
- A task  
- A ticket  

The target system decides whether it can accept and apply those instructions. Provisioning records the attempt and the response.

---

## 🔁 A Simple End-to-End Scenario

- A user requests access  
- The request is approved  
- Identity Security Cloud builds a change plan  
- Provisioning sends instructions to the target system  
- The target system responds  
- Identity Security Cloud records the result and later verifies the outcome  

---

## 👥 Four Perspectives on the Same Event

**User:**  
“I got access” or “I lost access.”

**Admin:**  
“The system attempted a change.”

**System:**  
“I sent instructions and received a response.”

**Auditor or Governance:**  
“I can see when the change was attempted, why it happened, and what the outcome was.”

---

## 🧭 Trigger vs Action

Trigger answers why provisioning started.  
Action answers what provisioning tried to do.

Most confusion comes from mixing these two ideas.

---

## 🧱 Configuration vs Behavior

Configuration defines what should happen.  
Behavior shows what actually happened.

Provisioning issues usually appear when configuration and behavior drift apart.

---

## 🕰️ System State vs Event History

**System state** answers:  
What access exists right now?

**Event history** answers:  
How did we get here?

Provisioning always writes history, even if the final state does not change.

---

## 🚫 What Provisioning Is Not

Provisioning does not decide who should have access.  
Provisioning does not guarantee access appears instantly.  
Provisioning does not mean verification has already happened.

---

## 😅 This Feels Wrong but Is Expected

Provisioning can succeed even if the target system later rejects the change.  
Provisioning can fail even if access already exists.  
Provisioning can retry automatically without human involvement.

---

## ⚠️ Common Failure Scenarios

- Target system is unreachable  
- Account creation attributes are invalid  
- Access already exists or is already missing  
- Manual provisioning tasks are not completed  

Each attempt and outcome is recorded.

---

## 🛠️ Beginner Troubleshooting Mindset

When something looks wrong, always reason in this order:

1. Was there a decision?  
2. Did provisioning attempt execution?  
3. Did the instruction reach the target?  
4. Did the target accept or reject it?  
5. Has verification updated the system state yet?  

---

## ✅ Pause and Confirm Understanding

If you can confidently explain:
- Why provisioning ran  
- What it attempted to change  
- Why the current access state may or may not reflect that attempt  

You are ready to move forward.

---

## ➡️ What Comes Next

Once you understand what provisioning is, the next logical question is:

**What causes provisioning to run?**

That is the focus of **Phase 2**.

---

## 🔗 Navigation

- **Next:** [Phase 2 – Triggers (Why Provisioning Runs)](./ISC_Provisioning_Phase_2_Triggers.md)
