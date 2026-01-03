# SailPoint Identity Security Cloud (ISC)
## Step 6: Roles in Certifications

> **Goal of this step**  
Understand how roles behave during **Certifications**, why certifications often expose weaknesses in role design, and how experienced admins use certification outcomes to *improve* the access model instead of fighting audits every cycle.

This step explains why many teams hate certifications — and how roles can fix that.

---

## 1. What Certifications Are Really About

At a basic level, certifications answer one question:

> “Is this access still correct for this person?”

But certifications are **not just audits**. They are feedback loops.

Certifications help you:
- Validate access decisions
- Detect over-provisioning
- Identify outdated roles
- Spot modeling gaps

> **Beginner clarity**  
Certifications don’t fail because auditors are strict. They fail because access models are unclear.

---

## 2. Why Roles Matter So Much in Certifications

Without roles:
- Reviewers see hundreds of entitlements
- Managers don’t understand what they’re approving
- Reviews turn into rubber-stamping
- Audit risk increases

With roles:
- Reviewers certify **intent**, not raw permissions
- Decisions are faster and more confident
- Exceptions stand out clearly
- Reviews become meaningful

> **Experienced-admin mindset**  
If certifications feel painful, the problem is usually the access model — not the certification campaign.

---

## 3. Two Ways Roles Appear in Certifications

Roles can show up in certifications in **two distinct ways**.

---

### 3.1 Role Membership Reviews

This answers:
> “Should this person still have this role?”

Characteristics:
- High-level review
- Business-friendly
- Focused on job relevance

Example:
- “Does Alice still need the Finance Analyst role?”

When this works well:
- Roles are well-designed
- Roles represent stable job functions

---

### 3.2 Role Composition Reviews

This answers:
> “Does this role still contain the right access?”

Characteristics:
- Admin or owner-focused
- More technical
- Happens less frequently

Example:
- “Should this role still include this access profile?”

> **Beginner clarity**  
Membership reviews validate *who* has access.  
Composition reviews validate *what* the role contains.

---

## 4. What Happens When a Reviewer Takes Action

### Certifying (Approve)
- Access or role remains unchanged
- No provisioning action occurs

### Revoking a Role (Membership Review)
- Role assignment is removed
- Provisioning removes entitlements tied to that role
- Direct access remains untouched

### Revoking Access in Role Composition
- Role definition is updated (often via a task or draft)
- Future assignments change
- Existing users may be impacted depending on rollout

> **Important**  
Revoking a role does not automatically clean up unrelated direct entitlements.

---

## 5. Why Certifications Often Reveal Role Problems

Common signals during certifications:

- Managers frequently revoke *parts* of a role
- The same exceptions appear every cycle
- Reviewers don’t recognize role names
- Roles are consistently questioned

These signals usually mean:
- Roles are too broad
- Roles mix core and edge access
- Common access was not handled properly
- Role intent is unclear

> Certifications are mirrors. They show you the truth about your model.

---

## 6. How Experienced Admins Use Certification Results

Instead of treating certifications as pain:

Experienced admins:
- Track repeated revocations
- Identify access that is always removed
- Feed those findings into:
  - Role Insights
  - Role redesign
  - Common access refinement

This closes the loop:
Certification → Insight → Model improvement

---

## 7. Common Certification Design Patterns with Roles

### Pattern 1: Role-first certifications
- Review role membership first
- Review entitlements only when needed
- Keeps campaigns readable

### Pattern 2: Separate business and technical reviews
- Managers review roles
- App owners review entitlements
- Reduces confusion

### Pattern 3: Risk-based depth
- Low-risk roles reviewed lightly
- High-risk roles reviewed deeply

> **Beginner clarity**  
Not all access needs the same review depth.

---

## 8. Common Mistakes with Roles in Certifications

### Mistake 1: Certifying only entitlements
Fix: Introduce role-based reviews.

### Mistake 2: Over-certifying stable roles
Fix: Reduce frequency for well-governed roles.

### Mistake 3: Ignoring revocation patterns
Fix: Patterns are signals, not noise.

### Mistake 4: Assuming revocation fixes the model
Fix: Revocation fixes access. Modeling fixes future access.

---

## 9. Beginner Anchor

At this stage:
- Certifications validate the access model
- Revocations provide feedback
- Role improvements reduce future audit pain

Certifications are not enemies. They are teachers.

---

## 10. Quick Self-Check

1. Why do role-based certifications scale better than entitlement-based ones?
2. What is the difference between role membership and role composition reviews?
3. Why do repeated revocations point to modeling issues?
4. Why doesn’t revoking a role clean up all access automatically?

---

## What’s Next

In **Step 7**, we will cover **Roles in Lifecycle (Joiner, Mover, Leaver)**:
- How roles are automatically assigned and removed
- Why lifecycle automation magnifies modeling mistakes
- How to safely automate without breaking trust

---

### Navigation
⬅️ Previous: Step 5 – Roles in Access Requests  
➡️ Next: Step 7 – Roles in Lifecycle (JML)
