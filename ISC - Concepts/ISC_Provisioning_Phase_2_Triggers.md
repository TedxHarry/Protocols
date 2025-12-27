# 🔄 ISC Provisioning – Phase 2: Triggers (Why Provisioning Runs)

Phase 1 answered what provisioning is.  
Phase 2 answers the next logical question: **why provisioning starts**.

Provisioning never runs randomly.  
It always runs because something changed or something was requested.

---

## 🧠 Provisioning Has Only One Rule

Provisioning runs when Identity Security Cloud detects a mismatch between:

- What access **should** exist  
- What access **currently** exists  

That mismatch is detected through **triggers**.

---

## 🧩 Two Families of Triggers

All provisioning triggers fall into two families:

- **Request-driven** (human-initiated)  
- **Data-driven** (system-initiated)  

Both create provisioning plans.  
The difference is only **how the change was discovered**.

---

## 👤 Request-Driven Triggers

Request-driven provisioning starts because a person explicitly asked for a change.

**Examples:**
- A user requests access  
- A manager requests access for a team member  
- An admin manually assigns or revokes access  

From the system’s perspective, these are intentional requests.

---

## ⚙️ What Happens Internally (Request-Driven)

1. A request is submitted  
2. Approvals are evaluated  
3. A provisioning plan is created  
4. Provisioning attempts execution  

Approval controls whether provisioning is allowed to start.  
Approval is **not** provisioning.

---

## 😕 Common Beginner Confusion (Request-Driven)

- Approval completed but access not visible yet  
- Approval denied but provisioning history still exists  

**Reason:**  
The system records **intent** and **outcomes** separately.

---

## 🔁 Data-Driven Triggers

Data-driven provisioning starts because identity data changed.

No one clicks a button.  
No one submits a request.

The system reacts to new information.

---

## 📊 Common Data-Driven Triggers

- Lifecycle state changes  
- Role assignment or removal  
- Certification revocation  
- Attribute synchronization mismatches  
- Identity refresh detecting differences  

---

## 🌡️ Mental Model: Thermostat

Data-driven provisioning works like a thermostat.

You don’t turn the heater on manually.  
You set a desired state.

When reality drifts, the system reacts automatically.

---

## ⏱️ Event-Based vs State-Based Triggers

Some triggers fire once.  
Others are continuously enforced.

**Event-based:**
- Lifecycle change from Active to Terminated  

**State-based:**
- Birthright access enforced as long as the user remains in a state  

---

## 😅 This Feels Wrong but Is Expected

- Provisioning can re-run even if nothing changed recently  
- Provisioning can remove access you manually added in the target system  
- Provisioning can re-grant access you manually removed  

**Reason:** state-based enforcement.

---

## 🔄 Multiple Triggers, One Engine

Requests, lifecycle changes, roles, and certifications all feel different.

Internally, they all feed the **same provisioning engine**.

Different trigger.  
Same execution path.

---

## ⏳ Trigger vs Timing

A **trigger** answers why provisioning should run.  
**Timing** answers when it actually runs.

Provisioning may run immediately, later, or after retries depending on:
- System load  
- Connector behavior  

---

## 🛠️ Beginner Troubleshooting Mindset

When provisioning surprises you, ask:

1. What changed?  
2. Was this request-driven or data-driven?  
3. Is this event-based or state-based?  
4. Is the system correcting drift?  

---

## ✅ Pause and Confirm Understanding

You should now be able to:

- Explain why provisioning ran without a request  
- Predict provisioning after identity data changes  
- Distinguish approval completion from trigger detection  

If this is clear, you’re ready for Phase 3.

---

## ➡️ What Comes Next

**Phase 3** will focus on provisioning plans.

You will learn what provisioning is actually trying to do once it starts.

---

## 🔗 Navigation

- **Previous:** [Phase 1 – Core Mental Model](./ISC_Provisioning_Phase_1_Core_Mental_Model.md)
- **Next:** [Phase 3 – Provisioning Plans](./ISC_Provisioning_Phase_3_Provisioning_Plans.md)
