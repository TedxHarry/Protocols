# SailPoint Identity Security Cloud (ISC)
## Access Modeling – Step 4: Role Insights

> **Goal of this step**  
Understand how **Role Insights** helps you improve *existing roles over time*, what signals ISC uses behind the scenes, and how to use insights safely without breaking trust in your access model.

Role Insights is about **refinement**, not creation.

---

## 1. What Role Insights Is (In Simple Terms)

Role Insights answers this question:

> “Given how people actually use access today, how should my existing roles be improved?”

It looks at:
- Roles that already exist
- The access held by users assigned to those roles
- Patterns that suggest roles are missing or carrying extra access

Then it provides **recommendations**, not changes.

> **Beginner clarity**  
Role Insights never changes a role automatically. It only suggests improvements.

---

## 2. How Role Insights Fits in the Big Picture

Think of Access Modeling as a lifecycle:

1. Define **Common Access**
2. Discover **new roles** (Role Discovery)
3. Operate roles in real life
4. Improve roles over time (**Role Insights**)

Role Insights only works **after roles exist and are in use**.

> **Experienced-admin mindset**  
If Role Discovery builds the house, Role Insights maintains it.

---

## 3. What Happens in the Background (Important Logic)

Role Insights analyzes three main things:

### A. Role Membership vs Actual Access
ISC compares:
- What access a role *claims* to provide
- What access role members *actually hold*

Gaps appear when:
- Users have extra access outside the role
- Users consistently need access that is missing from the role

---

### B. Entitlement Popularity Thresholds

One key concept is **popularity**.

Example:
- 100 users have Role A
- 85 of them hold Entitlement X
- Entitlement X has **85% popularity** in that role

ISC uses thresholds (for example, 80%) to decide:
- This entitlement might belong in the role

> **Beginner clarity**  
Popularity does not mean correctness. It means “many people have it”.

---

### C. Drift Detection

Over time, roles drift because:
- New tools are introduced
- Temporary access becomes permanent
- Managers approve exceptions

Role Insights detects this drift and surfaces it.

---

## 4. Types of Role Insight Recommendations

Role Insights typically suggests:

### 1. Add Entitlements to a Role
When an entitlement is held by a high percentage of role members but is not part of the role definition.

Use case:
- Role definition is outdated
- Access was added manually over time

---

### 2. Remove Entitlements from a Role
When entitlements in the role are rarely used or no longer needed by most members.

Use case:
- Role became over-provisioned
- Job responsibilities changed

---

### 3. Split Roles
When a role appears to cover multiple distinct access patterns.

Use case:
- One role serving two different job functions
- Growth or reorganization caused overlap

---

## 5. Role Insights Sessions (How You Review Safely)

Role Insights recommendations are reviewed in **sessions**, similar to discovery.

In a session, you:
- Review suggested changes
- Decide whether to accept or reject
- Apply changes as drafts first
- Publish only after validation

> **Beginner clarity**  
Always review insights as drafts. Never rush changes into production.

---

## 6. How Experienced Admins Evaluate Recommendations

Instead of asking:
> “Is this recommendation correct?”

Ask:
- Is this access **stable**?
- Is it **least privilege aligned**?
- Is it **job-defining** or just convenient?
- Would I feel safe granting this automatically later?

If the answer is unclear, defer the change.

---

## 7. Role Insights vs Role Discovery (Do Not Mix These)

| Area | Role Discovery | Role Insights |
|----|----|----|
| Purpose | Find new roles | Improve existing roles |
| Input | Direct access patterns | Role membership + access |
| Output | Suggested roles | Suggested role changes |
| Timing | Early modeling | Ongoing operations |

> Using Role Insights without Role Discovery leads to weak roles.

---

## 8. Common Mistakes with Role Insights

### Mistake 1: Blindly accepting popularity-based additions
Fix: Always apply business context.

### Mistake 2: Making large changes too frequently
Fix: Small, controlled updates build trust.

### Mistake 3: Treating insights as compliance requirements
Fix: Insights are guidance, not mandates.

### Mistake 4: Ignoring drift signals
Fix: Drift ignored today becomes audit pain tomorrow.

---

## 9. Practical Role Insights Workflow (Realistic)

1. Let roles run for a meaningful period
2. Review Role Insights periodically (monthly or quarterly)
3. Focus on:
   - High-popularity missing entitlements
   - Clear over-provisioned items
4. Apply changes as drafts
5. Communicate changes before publishing
6. Monitor impact
7. Repeat

> **Experienced-admin mindset**  
Consistency builds trust more than speed.

---

## 10. Quick Self-Check

1. Why is popularity alone not enough to add access to a role?
2. When should you split a role instead of modifying it?
3. Why should Role Insights changes be gradual?
4. How does Role Insights help prevent long-term role decay?

---

## What Comes After Step 4

At this point, you understand:
- How roles are discovered
- How common access is separated
- How roles are refined over time

Next logical connections (future steps):
- Connecting roles to **Access Requests**
- Using roles in **Certifications**
- Aligning roles with **Lifecycle automation**
- Preventing role explosion long term

---

### Navigation
⬅️ Previous: Step 3 – Role Discovery  
➡️ Next: *Operationalizing Roles (Requests, Certifications, Lifecycle)*
