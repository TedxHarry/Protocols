# 🧾 ISC Provisioning – Phase 3: Provisioning Plans (What the System Tries to Do)

Phase 1 explained what provisioning is.  
Phase 2 explained why provisioning runs.  
Phase 3 explains what provisioning actually contains once it starts.

This phase removes the mystery behind phrases like **“provisioning attempted but nothing changed.”**

---

## 🧠 Provisioning Plan in One Sentence

A provisioning plan is a structured set of instructions that describes what Identity Security Cloud wants to change for one identity on one or more systems.

---

## 🎯 Why Provisioning Plans Exist

Identity Security Cloud does not directly execute decisions.

It first translates decisions into a neutral, structured format.

That format is the **provisioning plan**.

The plan allows the system to reason, retry, audit, and explain what it attempted.

---

## 🧩 Mental Model: Work Order

Think of a provisioning plan as a work order.

It does not do the work.  
It describes the work.

Execution happens later, but the work order always exists.

---

## 📦 What a Provisioning Plan Can Contain

A single provisioning plan may include:

- Create account  
- Enable or disable account  
- Add entitlements  
- Remove entitlements  
- Update account attributes  

One trigger can produce multiple actions.

---

## 👤 One Identity, Many Systems

Provisioning plans are identity-centric.

One identity may have:
- Multiple accounts  
- On multiple sources  

The plan coordinates changes across all of them.

---

## 🤔 Why “Nothing Happened” Still Creates a Plan

Provisioning plans are created based on **desired state versus actual state**.

If the system determines no change is required:
- The plan still exists  
- Execution may be a no-op  

This is expected behavior, not a failure.

---

## 😕 Common Beginner Confusions

- Seeing provisioning activity when access already exists  
- Seeing a plan with no executed changes  
- Assuming no plan means no evaluation occurred  

Evaluation and execution are separate steps.

---

## ⚙️ Plan Creation vs Plan Execution

**Plan creation** answers:  
What should change?

**Plan execution** answers:  
Can we make it happen?

A plan can exist even if execution never occurs.

---

## ✅ Approval vs Plan

Approvals control whether a plan is allowed to proceed.

They do not change the contents of the plan.

The plan describes desired changes.  
Approval controls execution permission.

---

## 🧾 System Recording and Audit View

From an audit perspective:

- The plan explains intent  
- Execution explains attempt  
- Result explains outcome  

All three are recorded.

---

## 😅 This Feels Wrong but Is Expected

- A provisioning plan can exist even when execution is blocked  
- A provisioning plan can include actions that later fail  
- A provisioning plan can partially succeed  

---

## ⚠️ Failure Scenarios at the Plan Level

- Invalid attribute values during account creation  
- Conflicting actions in the same plan  
- Missing account when an entitlement update is requested  

Failures are tied to actions, not the entire plan.

---

## 🛠️ Beginner Troubleshooting Mindset

When analyzing provisioning behavior, ask:

1. What did the plan intend to change?  
2. Which actions were included?  
3. Which actions executed?  
4. Which actions failed and why?  

---

## ✅ Pause and Confirm Understanding

You should now be able to:
- Explain why provisioning ran even if nothing changed  
- Predict multiple actions from one trigger  
- Separate intent from execution  

If this is clear, you are ready for Phase 4.

---

## ➡️ What Comes Next

**Phase 4** will focus on fulfillment channels.

You will learn how provisioning plans are actually carried out.

---

## 🔗 Navigation

- **Previous:** [Phase 2 – Triggers (Why Provisioning Runs)](./ISC_Provisioning_Phase_2_Triggers.md)
- **Next:** [Phase 4 – Fulfillment Channels](./ISC_Provisioning_Phase_4_Fulfillment_Channels.md)
