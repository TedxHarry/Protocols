# 🔍 ISC Provisioning – Phase 5: Verification and Aggregation (How the System Confirms Reality)

Phase 4 explained how provisioning plans are executed.  
Phase 5 explains how Identity Security Cloud confirms what actually happened.

This phase resolves one of the most common misunderstandings:  
Why provisioning can be marked complete but access is still not visible.

---

## 🧠 Verification in One Sentence

Verification is the process by which Identity Security Cloud re-reads target systems to confirm whether provisioning actions actually changed reality.

---

## 🎯 Why Verification Exists

Provisioning is an attempt.  
Verification is confirmation.

Target systems are external and authoritative for their own data.  
Identity Security Cloud must ask them what actually exists.

---

## 🔄 Provisioning vs Verification

Provisioning sends instructions.  
Verification reads results.

They are separate by design.  
This separation prevents false assumptions and silent failures.

---

## 💸 Mental Model: Bank Transfer

Sending money is not the same as seeing it credited.

Provisioning is sending the transfer.  
Verification is checking the account balance later.

Delays are expected.

---

## 🔍 How Verification Happens

Verification happens through **aggregation**.

Aggregation re-reads account and entitlement data from target systems.  
Identity Security Cloud updates identity state based on what it finds.

---

## 📊 Types of Aggregation

There are two common forms:

- **Targeted aggregation** – reads only affected accounts  
- **Full aggregation** – reads all accounts on a source  

Not all connectors support targeted aggregation.

---

## ⏳ Why Access Is Not Immediate

Even after successful execution:

- Aggregation may not have run yet  
- Target systems may apply changes asynchronously  
- Caching may delay visibility  

This is expected behavior.

---

## 🕰️ System State vs Event History (Revisited)

**Event history** answers:  
What was attempted?

**System state** answers:  
What exists now?

Verification updates system state, not event history.

---

## 😕 Common Beginner Confusions

- Provisioning shows success but access is missing  
- Manual task completed but identity unchanged  
- Aggregation removes manually added access  

All are explained by verification timing.

---

## 🔁 Drift Detection and Correction

When verification detects differences:

- State-based rules may re-trigger provisioning  
- Manual changes may be reverted  

This is not aggressive behavior.  
It is enforcement.

---

## 🧾 Auditor and Governance View

Auditors care about evidence.

Verification provides:

- Proof that execution succeeded or failed  
- A timeline of when access truly changed  

This supports compliance and investigations.

---

## 😅 This Feels Wrong but Is Expected

- Verification may remove access you manually added  
- Verification may re-add access you manually removed  
- Verification may lag behind execution  

These are signs the system is working correctly.

---

## 🛠️ Beginner Troubleshooting Mindset

When access looks wrong after provisioning, ask:

1. Has aggregation run?  
2. Was it targeted or full?  
3. Does the target system show the change?  
4. Is state-based enforcement correcting drift?  

---

## ✅ Pause and Confirm Understanding

You should now be able to:

- Explain why access is delayed after provisioning  
- Separate execution success from verification success  
- Predict when provisioning will re-run due to drift  

If this is clear, you are ready for Phase 6.

---

## ➡️ What Comes Next

**Phase 6** will focus on failure handling, retries, and error interpretation.

You will learn how to troubleshoot without guessing.

---

## 🔗 Navigation

- **Previous:** [Phase 4 – Fulfillment Channels](./ISC_Provisioning_Phase_4_Fulfillment_Channels.md)
- **Next:** [Phase 6 – Failures, Retries, and Error Interpretation](./ISC_Provisioning_Phase_6_Failures_Retries_and_Error_Interpretation.md)
