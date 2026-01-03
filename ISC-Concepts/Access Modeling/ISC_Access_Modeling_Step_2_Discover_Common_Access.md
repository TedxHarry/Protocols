# SailPoint Identity Security Cloud (ISC)
## Access Modeling – Step 2: Discover Common Access

> **Goal of this step**  
Learn how SailPoint ISC identifies *Common Access*, why it must come **before** role discovery, and how experienced admins decide what truly belongs in common access versus what does not.

This step turns theory from Step 1 into your **first real modeling decision**.

---

## 1. What “Common Access” Really Means

Common Access represents **baseline access** that:
- Most identities in your organization need
- Is not tied to a specific job function
- Is generally low risk
- Rarely changes with role or team

Think of it as:
> “If someone works here, they almost certainly need this.”

Examples:
- Corporate email
- VPN
- Collaboration tools
- Intranet portals

> **Beginner clarity**  
Common access is not about convenience. It is about *predictability and stability*.

---

## 2. Why Common Access Must Come First

If you skip this step and jump straight to role discovery:
- Common entitlements appear in every role
- Roles become bloated
- Role discovery results become noisy
- Certifications become painful

ISC expects this order:
1. Identify and confirm common access
2. Exclude it from discovery
3. Discover specialized roles on cleaner data

> **Experienced-admin insight**  
Good common access makes role discovery boring. That’s a good thing.

---

## 3. What Happens in the Background

When entitlements are marked as common access:
- ISC treats them as **baseline**
- They are **excluded from future role discovery**
- They are excluded from access request recommendations
- They stop polluting specialized roles

This is a filtering mechanism, not just a label.

---

## 4. How ISC Identifies Common Access

ISC can surface common access in **three ways**.

---

### Method 1: Automatic Suggestions (Banner)

ISC may show a banner suggesting common access based on:
- High entitlement usage
- Broad distribution across identities

You can:
- Review the suggestion
- Confirm it as common access
- Ignore it if it feels risky

> Never blindly accept suggestions.

---

### Method 2: During Role Discovery

While reviewing role discovery results:
- You may notice entitlements present in almost every suggested role
- These are strong candidates for common access

Experienced admins pause discovery here and:
- Pull those entitlements out
- Define them as common access
- Rerun discovery

---

### Method 3: Manual Identification

Admins can proactively mark access as common when:
- Business already defines birthright access
- Security teams have approved baseline tools
- Historical data confirms near-universal usage

This is common in mature organizations.

---

## 5. What Belongs in Common Access (Decision Guide)

Good candidates:
- Access held by a large percentage of users
- Access required regardless of department
- Access that rarely changes

Bad candidates:
- Admin access
- Finance-only or HR-only systems
- Temporary or project-based access
- Anything regulated or high risk

> **Rule of thumb**  
If removing it would surprise most managers, it might be common access.

---

## 6. Common Access Is Not “Everyone Roles”

This is a critical distinction.

Bad design:
- One massive “Everyone Role” with everything inside

Good design:
- Small, focused common access bundles
- Clear intent
- Minimal risk

> Common access should be boring, not powerful.

---

## 7. How Common Access Improves Everything Else

Once common access is defined:
- Role discovery becomes cleaner
- Specialized roles become smaller and clearer
- Access requests become more meaningful
- Certifications focus on real risk

This is leverage, not busy work.

---

## 8. Beginner Anchor

At this stage:
- You are making **modeling decisions**, not automation
- You are allowed to be conservative
- Smaller common access is better than oversized common access

You can always expand later.

---

## 9. Quick Self-Check

Ask yourself:
1. Would most users lose productivity without this access?
2. Is this access stable across roles?
3. Would I feel safe auto-assigning this in the future?

If all three are “yes”, it is a strong candidate.

---

## What’s Next

In **Step 3**, we will move into **Role Discovery**:
- Peer groups
- Discovery sessions
- Interpreting suggested roles correctly
- Avoiding role explosion

---

### Navigation
⬅️ Previous: Step 1 – Foundations  
➡️ Next: Step 3 – Role Discovery
