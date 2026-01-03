# SailPoint Identity Security Cloud (ISC)
## Step 7: Roles in Lifecycle (Joiner, Mover, Leaver)

> **Goal of this step**  
Understand how roles behave during **Joiner, Mover, and Leaver (JML)** events, how lifecycle automation actually applies and removes access, and why lifecycle rules amplify both good and bad role design.

This step is where roles move from *governed* to *automated*.

---

## 1. What Lifecycle Means in ISC (Plain Language)

Lifecycle answers one question:

> “When should access be added, changed, or removed?”

In ISC, lifecycle decisions are driven by:
- Identity attributes (status, department, job title, location)
- Lifecycle states (Joiner, Active, Mover, Leaver)
- Rules that map identity changes to access actions

> **Beginner clarity**  
Lifecycle controls *timing*. Roles control *content*.

---

## 2. Why Roles Are Central to Lifecycle Automation

Lifecycle automation works best when:
- Access is bundled into roles
- Roles represent stable job intent
- Direct entitlements are minimized

Why:
- Adding one role can provision many entitlements safely
- Removing one role can deprovision access consistently
- Automation stays readable and auditable

> **Experienced-admin mindset**  
Automate roles, not raw permissions.

---

## 3. Joiner: How New Hires Get Access

### What typically happens
When a new identity joins:
1. Identity is created from an authoritative source
2. Lifecycle state becomes **Joiner** or **Active**
3. Lifecycle rules evaluate identity attributes
4. Roles are assigned automatically
5. Provisioning applies entitlements

### Types of roles assigned at Joiner
- **Common (birthright) roles**  
  Access everyone needs from day one

- **Job-based roles**  
  Based on department, job title, or location

> **Beginner clarity**  
Joiner automation should feel boring and predictable.

---

## 4. Mover: When Attributes Change

Mover events happen when:
- Department changes
- Job title changes
- Location changes
- Manager changes

### What lifecycle rules do
1. Detect attribute change
2. Reevaluate role eligibility
3. Remove roles no longer applicable
4. Assign new roles if conditions match

### Why clean roles matter here
If roles are too broad:
- Users retain access they no longer need
If roles are too granular:
- Users lose access constantly

> **Experienced-admin mindset**  
Mover logic is the fastest way to expose weak role design.

---

## 5. Leaver: Removing Access Safely

Leaver events occur when:
- Employment ends
- Identity is disabled or terminated

### What typically happens
1. Identity enters **Leaver** state
2. Role assignments are removed
3. Provisioning deprovisions access
4. Accounts may be disabled or deleted

Important detail:
- Only access tied to roles or lifecycle rules is removed automatically
- Direct entitlements may require separate handling

> **Beginner clarity**  
Lifecycle automation is only as complete as your role coverage.

---

## 6. Common Lifecycle Design Patterns with Roles

### Pattern 1: Birthright via lifecycle
- Common access roles assigned at Joiner
- Never requestable
- Stable and predictable

### Pattern 2: Job-based roles via lifecycle
- Assigned and removed automatically on attribute change
- Minimal manager involvement

### Pattern 3: Optional roles via request
- Not lifecycle-driven
- Added through Access Requests
- Often time-bound

> Good designs clearly separate these patterns.

---

## 7. What Happens When Things Go Wrong

### Symptom: Access not removed at Mover
Likely cause:
- Role eligibility conditions too broad
- Direct entitlements bypassing roles

### Symptom: Access flapping
Likely cause:
- Overly granular roles
- No buffer between attribute changes and automation

### Symptom: Leaver retains access
Likely cause:
- Access not tied to roles
- Manual grants outside lifecycle control

> **Rule of thumb**  
Automation exposes modeling mistakes faster than humans do.

---

## 8. How Experienced Admins Roll Out Lifecycle Automation

They do not automate everything at once.

Typical approach:
1. Start with common access roles
2. Automate only stable job roles
3. Observe joiner and mover behavior
4. Fix edge cases
5. Expand gradually

> **Experienced-admin mindset**  
Trust is built through predictability, not speed.

---

## 9. How Lifecycle Feeds Back Into the Model

Lifecycle outcomes provide feedback:
- Frequent manual fixes indicate bad roles
- Repeated exceptions highlight modeling gaps
- Automation failures point to unclear role intent

This feedback loops back into:
- Role Discovery
- Role Insights
- Role redesign

---

## 10. Beginner Anchor

At this stage:
- Lifecycle rules decide *when*
- Roles decide *what*
- Provisioning decides *how*

Never mix these responsibilities.

---

## 11. Quick Self-Check

1. Why should lifecycle rules assign roles instead of entitlements?
2. What makes mover events more complex than joiner events?
3. Why doesn’t leaver automation always remove all access?
4. Why should lifecycle automation be rolled out gradually?

---

## What Comes After Step 7

You now understand:
- Modeling roles
- Governing roles
- Requesting roles
- Reviewing roles
- Automating roles

Next advanced topics (optional but powerful):
- Preventing role explosion long term
- Role ownership and accountability
- Aligning roles with SoD and risk
- Operating at scale

---

### Navigation
⬅️ Previous: Step 6 – Roles in Certifications  
➡️ Next: *Advanced Role Operations & Scaling*
