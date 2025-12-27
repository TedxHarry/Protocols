# 🚦 ISC Provisioning – Phase 4: Fulfillment Channels (How Execution Actually Happens)

Phase 1 explained what provisioning is.  
Phase 2 explained why provisioning runs.  
Phase 3 explained what provisioning plans contain.

Phase 4 explains **how those plans are actually carried out in the real world**.

This phase removes confusion around words like *pending*, *manual task*, *retry*, and *completed*.

---

## 🧠 Fulfillment in One Sentence

Fulfillment is the method Identity Security Cloud uses to deliver a provisioning plan to a downstream system and record the outcome.

---

## 🎯 Why Fulfillment Channels Exist

Not all systems can be changed the same way.

- Some systems support APIs  
- Some require human action  
- Some are governed by ticketing workflows  

Fulfillment channels exist to adapt the **same provisioning plan** to different execution realities.

---

## 🔀 The Three Fulfillment Channels

Identity Security Cloud uses three primary fulfillment channels:

1. **Direct connector execution**  
2. **Manual provisioning tasks**  
3. **Ticketing system integration**  

---

## ⚡ Channel 1: Direct Connector Execution

In this channel, Identity Security Cloud communicates directly with the target system.

The provisioning plan is translated into API calls or system-specific operations.

The target system responds with success or failure, which ISC records.

### System Behavior (Direct Connector)

- Execution can be immediate or queued  
- Errors may be retryable  
- Partial success is possible  
- The connector determines what is technically allowed  

### Common Confusions (Direct Connector)

- Success does not always mean access is visible immediately  
- Failure may trigger automatic retries  
- A successful API call does not guarantee business correctness  

---

## 🧑‍💻 Channel 2: Manual Provisioning Tasks

Some systems cannot be changed automatically.

In these cases, Identity Security Cloud creates a **manual task** describing the required change.

A human completes the change outside the system and marks the task complete.

### Admin Perspective (Manual Tasks)

- Tasks describe what must be done, not how  
- Completion is a declaration, not a verification  
- Delays are expected and visible  

### 😅 This Feels Wrong but Is Expected (Manual Tasks)

- Marking a task complete does not verify access  
- Provisioning shows success even before aggregation  
- Incorrect manual completion leads to drift  

---

## 🎫 Channel 3: Ticketing System Fulfillment

Some organizations route provisioning through ticketing systems.

Identity Security Cloud creates a ticket containing the provisioning plan.

Execution happens according to external workflows.

### System vs Ticket Status

- Ticket closed does not guarantee access exists  
- Ticket open does not mean provisioning failed  
- ISC tracks the ticket state, not the work quality  

---

## 🔄 One Plan, Different Outcomes

The same provisioning plan may:

- Execute instantly via a connector  
- Wait hours or days as a manual task  
- Follow a multi-step ticket workflow  

The plan is the same.  
Only the delivery differs.

---

## 🔁 Retries and Queues

Provisioning failures may be retryable.

- Retries do not indicate new intent  
- They indicate repeated execution attempts  

Queued actions are normal under load.

---

## 👥 User, Admin, and Auditor Perspectives

**User:**  
“Why don’t I have access yet?”

**Admin:**  
“The task is still pending.”

**Auditor:**  
“I can see when execution was attempted and through which channel.”

---

## 🛠️ Beginner Troubleshooting Mindset

When provisioning is pending or delayed, ask:

1. Which fulfillment channel is being used?  
2. Has execution started or is it queued?  
3. Is this waiting on a human or an external system?  
4. Has verification occurred yet?  

---

## ✅ Pause and Confirm Understanding

You should now be able to:

- Explain why provisioning is pending  
- Distinguish execution from verification  
- Predict delays based on fulfillment channel  

If this is clear, you are ready for Phase 5.

---

## ➡️ What Comes Next

**Phase 5** will focus on verification and aggregation.

You will learn how Identity Security Cloud confirms what actually happened.

---

## 🔗 Navigation

- **Previous:** [Phase 3 – Provisioning Plans](./ISC_Provisioning_Phase_3_Provisioning_Plans.md)
- **Next:** [Phase 5 – Verification and Aggregation](./ISC_Provisioning_Phase_5_Verification_and_Aggregation.md)
