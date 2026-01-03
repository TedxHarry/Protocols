# SailPoint Identity Security Cloud (ISC)
## Access Modeling – Step 3: Role Discovery

> **Goal of this step**  
Learn how Role Discovery finds **peer groups** and suggests **candidate roles**, what is happening in the background, and how to review suggestions like an experienced admin (not like someone clicking “accept” on everything).

This step is where Access Modeling starts to feel “real”, because you will see suggested roles built from your organization’s actual access patterns.

---

## 1. What Role Discovery Is Trying to Do

Role Discovery helps you answer this question:

> “Which groups of people already share a predictable set of access, and could that be managed as a role?”

It does **not** magically create perfect roles. It finds **patterns**. You decide whether those patterns should become real roles.

### What Role Discovery produces
- **Peer groups** (clusters of identities with similar access)
- **Suggested roles** (candidate bundles of access that look role-like)
- A **discovery session** (a repeatable workspace where you filter, review, and refine)

> **Beginner clarity**  
Role Discovery suggests. You approve. Nothing changes until you publish/assign roles through your governance process.

---

## 2. Why Step 2 Matters Here (Quick Reminder)

If you did not define Common Access first:
- Every suggested role will be bloated with baseline entitlements
- You will “discover” the same common items again and again
- You will create roles that look useful but are actually noisy and risky

> **Experienced-admin mindset**  
Clean baseline access first. Then discover specialized access on cleaner data.

---

## 3. The Mental Model (Simple but Accurate)

Think of Role Discovery like this:

1. **Start with a population** (a group of identities you choose)  
2. ISC analyzes their **entitlements** and how they overlap  
3. ISC finds **clusters** (peer groups) where overlap is high  
4. ISC proposes **candidate roles** for those clusters  
5. You review and decide:
   - Keep as a role
   - Split / merge / ignore
   - Extract “common” items you missed
   - Convert to Access Profiles (recommended)
   - Publish when ready

---

## 4. What Happens in the Background (So It Makes Sense)

### The core idea: similarity
ISC treats access as a pattern:

- Identity A holds entitlements: {E1, E2, E3, E7}
- Identity B holds entitlements: {E1, E2, E3, E8}
- Identity C holds entitlements: {E1, E2, E3, E9}

ISC sees a strong overlap {E1, E2, E3} and says:
> “This overlap is role-like.”

### Peer groups
Peer groups are essentially:
- A set of identities
- With highly similar access
- Detected by access overlap, not just org chart

> **Important**  
Departments and job titles help you **choose the population**, but the clustering is driven by access similarity.

### Timing reality
New identities and new entitlements don’t always show immediately. In practice, discovery is often “tomorrow clean”, not “right now perfect”.

---

## 5. Role Discovery Session (How You Start)

A **discovery session** is your controlled workspace. This is where you define:
- Which identities are in scope
- Which attributes you want to use to filter/group
- How broad or narrow the analysis should be

### Step 5.1: Pick the right population (most important decision)
Role Discovery begins by defining a group of identities.

Good populations:
- One department (Finance, HR)
- One region + department (Finance + US)
- One business unit
- One job family

Bad populations:
- “Everyone”
- Mixed departments with unrelated work
- A group with messy identity attribute values

> **Rule of thumb**  
If the population is too broad, you get role explosion.  
If the population is too narrow, you get roles that only fit 2 people.

### Step 5.2: Use identity attributes intentionally
You will typically filter based on identity attributes like:
- Department
- Job title
- Location
- Manager hierarchy (sometimes)

This is not just “nice to have”.
Identity attributes are how you avoid mixing unrelated access patterns.

> **Beginner clarity**  
If your identity attributes are inconsistent, fix data quality before trusting discovery results.

---

## 6. Understanding the Output (Peer Groups and Suggested Roles)

### Peer Group output
A peer group is:
- A cluster of identities discovered by access similarity
- Usually a subset of your population
- The “community” from which a role candidate is derived

What you do with peer groups:
- Inspect whether the group makes business sense
- Validate that the access pattern is real and repeatable

### Suggested Role output
A suggested role includes:
- A list of entitlements (and sometimes access profile suggestions depending on setup)
- A membership candidate list (who this role fits)
- An implied “popularity” of each entitlement within that peer group

> **Experienced-admin mindset**  
Don’t ask “Is this role correct?”  
Ask “Is this role stable, least-privilege aligned, and maintainable?”

---

## 7. The Most Important Review Skill: Separate “Core” vs “Edge” Entitlements

Within a peer group, you’ll often see:
- **Core entitlements**: held by most people in the group
- **Edge entitlements**: held by a few people, often exceptions

### What to do
- Keep **core** entitlements in the role
- Move edge items to:
  - A separate smaller role
  - An access profile
  - Direct requestable entitlements (if that matches your model)

> **Beginner clarity**  
A role should represent what’s *consistently needed*, not every exception.

---

## 8. Converting Suggestions Into Real Roles (Admin Choices)

When you accept a suggested role, you typically choose how to structure it:

### Option A: Put entitlements directly in the role
Pros:
- Fast
Cons:
- Hard to reuse
- Harder to maintain at scale
- Roles become “entitlement dumping grounds”

### Option B: Bundle into Access Profiles and assign those to the role (recommended)
Pros:
- Reusable bundles
- Cleaner role design
- Easier maintenance
Cons:
- Slightly more upfront design effort

> **Rule of thumb**  
If you expect reuse across multiple roles, use access profiles.

---

## 9. Practical Workflow (How Experienced Admins Iterate)

This is the “real life” loop:

1. Run discovery for a well-defined population  
2. Review peer groups:
   - Do they make business sense?
3. Review suggested roles:
   - Identify core access
   - Remove edge access
4. Check for missed common access:
   - If a baseline entitlement appears everywhere, extract it
5. Create roles as drafts:
   - Prefer access profiles when it makes sense
6. Publish gradually:
   - Pilot with a small set of identities
7. Observe drift:
   - Expect changes over time
8. Improve later using Role Insights (Step 4)

> **Experienced-admin mindset**  
You do not “finish” role design. You operationalize it.

---

## 10. Common Mistakes (And How to Avoid Them)

### Mistake 1: Discovering roles from “Everyone”
Result: role explosion and meaningless roles.

Fix: choose a narrow population.

### Mistake 2: Accepting every suggested role
Result: too many roles nobody trusts.

Fix: keep only roles with clear business meaning and stable access.

### Mistake 3: Including exception access in the main role
Result: over-provisioning and audit pain.

Fix: split edge access into separate roles/profiles.

### Mistake 4: Treating discovery as enforcement
Result: confusion and broken expectations.

Fix: remind stakeholders: discovery suggests, governance decides.

---

## 11. Quick Self-Check (If You Can Answer These, You’re Learning Well)

1. What makes a peer group different from a department?
2. Why is “core vs edge entitlements” the key skill in role review?
3. When should you bundle into access profiles instead of direct entitlements?
4. What is the biggest danger of running discovery for a very broad population?

---

## What’s Next

In **Step 4**, we go into **Role Insights**:
- How it improves existing roles
- Suggesting entitlement updates
- Using insights without creating role chaos

---

### Navigation
⬅️ Previous: Step 2 – Discover Common Access  
➡️ Next: Step 4 – Role Insights
