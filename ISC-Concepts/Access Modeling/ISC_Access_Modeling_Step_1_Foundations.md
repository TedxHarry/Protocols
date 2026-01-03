# SailPoint Identity Security Cloud (ISC)
## Access Modeling – Step 1: Foundations

> **Goal of this step**  
This step helps you build a clear mental picture of Access Modeling before touching any configuration. The goal is not speed. The goal is understanding how ISC *thinks* about access so that every advanced topic later feels logical and predictable.

If you understand this step well, you will avoid most real-world role design mistakes.

---

## 1. What Access Modeling Is (and What It Is Not)

### What Access Modeling *is*
Access Modeling helps you **analyze existing access**, understand patterns, and **suggest better access structures** using machine learning.

In simple words:
- ISC looks at who has what access today
- Finds patterns where many people share similar access
- Suggests how that access could be organized more cleanly using roles

It focuses on **insight and guidance**, not execution.

---

### What Access Modeling *is NOT*
This is one of the most important beginner clarifications.

Access Modeling does **not**:
- Automatically assign roles to users
- Automatically remove access from users
- Automatically enforce lifecycle changes
- Automatically fix bad access designs

Everything produced by Access Modeling is:
- A **suggestion**
- A **draft**
- Something that requires **human review and approval**

> **Beginner clarity**  
If you discover a role, nothing changes in the system until an admin reviews and applies it.

> **Experienced-admin mindset**  
Access Modeling informs decisions. Other ISC features execute them.

---

## 2. Core Building Blocks (Plain Language First)

### Identity
An **identity** represents a real person in ISC.
- Has attributes like department, job title, location, manager
- Can have multiple accounts

Clean identity data leads to meaningful roles.

### Account
A technical account on a system like AD, Entra ID, Salesforce.
- One identity can have many accounts
- Access lives on accounts

### Entitlement
A single unit of access.
Examples include groups, roles, licenses.

### Access Profile
A reusable collection of entitlements.
- Helps standardize access
- Makes roles easier to manage

**Mental model**: Entitlements = ingredients, Access Profile = recipe

### Role
Defines what access a type of user should have.
- Can include entitlements directly
- Or include access profiles (recommended)

**Mental model**: Role = meal plan

A role does nothing until it is assigned or requested.

---

## 3. Roles vs Lifecycle Rules

Lifecycle rules decide **when** access actions happen.  
Roles decide **what** access looks like.

Example:
- Lifecycle: New Hire → Active
- Role: Finance Analyst Role

---

## 4. Common Access vs Specialized Access

### Common Access
Shared by most users, not job-specific.
Examples: Email, VPN, Intranet

Once marked as common access, it is excluded from future role discovery.

### Specialized Access
Job-specific access held by peer groups.
This is where Role Discovery focuses.

---

## 5. How Role Discovery Thinks

ISC analyzes shared entitlements to find peer groups.
Peer groups become suggested roles.

Better identity data produces better roles.

---

## 6. Direct Access Will Always Exist

Some access will always be granted directly.
Access Modeling helps reduce chaos, not eliminate it.

---

## 7. Access Modeling Flow

Raw access  
→ Direct entitlements  
→ Pattern analysis  
→ Common access  
→ Peer groups  
→ Specialized roles  
→ Role Insights

---

## 8. Beginner Anchor

At this stage:
- You are not configuring anything
- You are not automating anything
- You are building understanding only

This is intentional.

---

## Navigation
➡️ Next: Step 2 – Discover Common Access
