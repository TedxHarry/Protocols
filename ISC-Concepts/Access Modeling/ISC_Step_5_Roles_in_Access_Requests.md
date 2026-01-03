# SailPoint Identity Security Cloud (ISC)
## Step 5: Roles in Access Requests

> **Goal of this step**  
Understand how roles are used in **Access Requests**, what actually happens when a role is requested, and how experienced admins design role requests to stay clean, scalable, and auditable.

This step connects your **access model** to real user behavior.

---

## 1. Why Access Requests Matter for Roles

Access Requests answer a simple but critical question:

> “How does someone *ask* for access in a controlled way?”

When roles are requestable:
- Users don’t request dozens of entitlements
- Managers don’t approve technical permissions blindly
- Governance teams review *meaningful access*, not noise

> **Beginner clarity**  
Access Requests are not about convenience. They are about control and traceability.

---

## 2. What Happens When a Role Is Requested (Behind the Scenes)

When a user requests a role, ISC does **not** grant access instantly.

The flow looks like this:

1. User selects a **role** in Request Center  
2. ISC evaluates:
   - Who is requesting
   - Who the access is for
   - Whether the role is requestable
3. Approval workflow starts (manager, owner, governance, etc.)  
4. Once approved:
   - Role assignment is created
   - Provisioning engine executes changes
5. Entitlements inside the role are provisioned to accounts

> **Important**  
Users never receive “a role” technically. They receive the **entitlements inside the role**.

---

## 3. Why Roles Are Better Than Requesting Entitlements

### Without roles
- Users request individual entitlements
- Managers approve what they don’t understand
- Over-provisioning is common
- Reviews become painful

### With roles
- Users request job-relevant access
- Managers approve recognizable bundles
- Access stays consistent
- Reviews focus on intent

> **Experienced-admin mindset**  
Requests should express *business need*, not technical detail.

---

## 4. Making a Role Requestable (Conceptual View)

For a role to appear in Access Requests:
- It must be marked as **requestable**
- It must be visible to the requester’s scope
- It must not be restricted by policy or lifecycle rules

Key idea:
- Not every role should be requestable
- Some roles are birthright or lifecycle-driven only

> **Beginner clarity**  
A role can exist without ever being requestable.

---

## 5. Approval Design: Why It Matters

When a role is requested, approvals typically answer:
- Does the user need this role?
- Is it appropriate for their job?
- Is there any conflict or risk?

Common approvers:
- Manager
- Application owner
- Governance team

> **Good practice**  
Roles reduce approval complexity because approvers review *intent*, not raw permissions.

---

## 6. Role Requests vs Entitlement Requests

| Area | Role Request | Entitlement Request |
|----|----|----|
| User experience | Simple | Complex |
| Manager understanding | High | Low |
| Governance clarity | High | Low |
| Risk of over-provisioning | Lower | Higher |
| Audit readiness | Better | Worse |

> Entitlements are implementation details. Roles are business access.

---

## 7. Common Design Patterns (Real-World)

### Pattern 1: Job Role Requests
- Example: “Finance Analyst”, “HR Partner”
- Used when someone changes jobs
- Often manager-approved

### Pattern 2: Functional Add-On Roles
- Example: “Reporting Access”, “Production Read-Only”
- Used for extra responsibilities
- Often owner-approved

### Pattern 3: Temporary Role Requests
- Example: “Project X Contributor”
- Time-bound
- Automatically expires

> **Beginner clarity**  
Temporary access is safer when modeled as roles, not one-off entitlements.

---

## 8. How Access Requests Affect Role Quality

Access Requests provide feedback to your model:
- Frequently requested roles are usually well-designed
- Rarely requested roles may be obsolete
- Frequent exceptions signal modeling gaps

This feedback later feeds **Role Insights**.

---

## 9. Common Mistakes With Role Requests

### Mistake 1: Making every role requestable
Fix: Separate birthright roles from requestable roles.

### Mistake 2: Allowing entitlements to bypass roles
Fix: Encourage role-first requests wherever possible.

### Mistake 3: Overloading roles to reduce approvals
Fix: Keep roles clean. Reduce noise elsewhere.

### Mistake 4: Ignoring denial patterns
Fix: Denials often reveal bad role design.

---

## 10. Beginner Anchor

At this stage:
- Roles define *what* is requested
- Access Requests define *how* it is requested
- Provisioning defines *how it is applied*

Each layer has a different responsibility.

---

## 11. Quick Self-Check

1. Why are roles better request targets than entitlements?
2. When should a role NOT be requestable?
3. How do access requests provide feedback into role design?
4. What risk increases when entitlements bypass roles?

---

## What’s Next

In **Step 6**, we will cover **Roles in Certifications**:
- How roles simplify access reviews
- Role composition vs role membership reviews
- Why certification pain often points to modeling issues

---

### Navigation
⬅️ Previous: Step 4 – Role Insights  
➡️ Next: Step 6 – Roles in Certifications
