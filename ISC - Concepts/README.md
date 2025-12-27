# ISC Provisioning – Learning & Troubleshooting Series

This repository helps you **understand, operate, and troubleshoot provisioning in SailPoint Identity Security Cloud** using a clear, step-by-step mental model.

It is designed to be **readable, practical, and reusable** for both learning and real-world support.

---

## How to Use This Series

If you are new to provisioning, **start from Phase 1 and read in order**.  
If you are troubleshooting, jump directly to the **Playbook** or **Decision Tree**.

The core mental loop used everywhere is:

**Decide → Plan → Execute → Verify → Enforce**

---

## Learning Path (Read in Order)

### Core Provisioning Phases

1. [Phase 1 – Core Mental Model](./ISC_Provisioning_Phase_1_Core_Mental_Model.md)  
2. [Phase 2 – Triggers](./ISC_Provisioning_Phase_2_Triggers.md)  
3. [Phase 3 – Provisioning Plans](./ISC_Provisioning_Phase_3_Provisioning_Plans.md)  
4. [Phase 4 – Fulfillment Channels](./ISC_Provisioning_Phase_4_Fulfillment_Channels.md)  
5. [Phase 5 – Verification and Aggregation](./ISC_Provisioning_Phase_5_Verification_and_Aggregation.md)  
6. [Phase 6 – Failures, Retries, and Error Interpretation](./ISC_Provisioning_Phase_6_Failures_Retries_and_Error_Interpretation.md)  
7. [Phase 7 – End-to-End Scenarios](./ISC_Provisioning_Phase_7_End_to_End_Scenarios.md)

---

## Practice and Troubleshooting

8. [Real-World Troubleshooting Drills](./ISC_Provisioning_Real_World_Troubleshooting_Drills.md)  
9. [Day-to-Day Admin Troubleshooting Playbook](./ISC_Provisioning_Day_to_Day_Admin_Troubleshooting_Playbook.md)  
10. [One-Page Troubleshooting Decision Tree](./ISC_Provisioning_One_Page_Troubleshooting_Decision_Tree.md)

---

## When to Use What

- **Learning provisioning** → Start at Phase 1  
- **Understanding strange behavior** → Read Phases 4–6  
- **Daily admin issues** → Use the Playbook  
- **Live incident** → Open the Decision Tree  

---

## Key Rules to Remember

- Approval does not mean access exists  
- Execution does not mean verification completed  
- Ticket closed does not mean access is correct  
- Manual changes are overwritten if enforcement exists  

If access comes back, **something is enforcing it**.

---

## Repository Structure

```
provisioning/
├── ISC_Provisioning_Phase_1_Core_Mental_Model.md
├── ISC_Provisioning_Phase_2_Triggers.md
├── ISC_Provisioning_Phase_3_Provisioning_Plans.md
├── ISC_Provisioning_Phase_4_Fulfillment_Channels.md
├── ISC_Provisioning_Phase_5_Verification_and_Aggregation.md
├── ISC_Provisioning_Phase_6_Failures_Retries_and_Error_Interpretation.md
├── ISC_Provisioning_Phase_7_End_to_End_Scenarios.md
├── ISC_Provisioning_Real_World_Troubleshooting_Drills.md
├── ISC_Provisioning_Day_to_Day_Admin_Troubleshooting_Playbook.md
└── ISC_Provisioning_One_Page_Troubleshooting_Decision_Tree.md
```

---

This series is meant to be **read, referenced, and reused**.  
Provisioning becomes simple once the loop is clear.
