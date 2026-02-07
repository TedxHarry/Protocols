# DAY 1: FOUNDATIONS & BASIC TRANSFORMS - MORNING SESSION

## Table of Contents
- [Part 1: Core Concepts (45 minutes)](#part-1-core-concepts-45-minutes)
  - [What Are Transforms?](#what-are-transforms-the-foundation)
  - [Transform vs Mapping vs Rule](#transform-vs-mapping-vs-rule---whats-the-difference)
  - [Where Are Transforms Used?](#where-are-transforms-used)
  - [When Does a Transform Execute?](#when-does-a-transform-execute)
  - [Transform Input/Output Model](#transform-inputoutput-model)
  - [Checkpoint Quiz 1](#checkpoint-quiz-1)
- [Part 2: Hands-On Setup (45 minutes)](#part-2-hands-on-setup-45-minutes)
  - [Step 1: Access Your Environment](#step-1-access-your-environment-5-min)
  - [Step 2: Navigate to Identity Profile Mappings](#step-2-navigate-to-identity-profile-mappings-10-min)
  - [Step 3: Explore the Transform Dropdown](#step-3-explore-the-transform-dropdown-15-min)
  - [Step 4: Use Identity Preview](#step-4-use-identity-preview-15-min)
  - [Checkpoint Quiz 2](#checkpoint-quiz-2)
- [Morning Session Complete](#morning-session-complete)

---

## Part 1: Core Concepts (45 minutes)

### What Are Transforms? (The Foundation)

Think of transforms as **calculation engines** that sit between your source data and your identity attributes in SailPoint.

#### The Simple Mental Model
```
SOURCE SYSTEM          TRANSFORM           IDENTITY IN ISC
(HR/AD/etc.)          (Calculator)        (What SailPoint knows)
--------------------------------------------------------------------------------

firstname: "John"   -->  [Concatenate]  -->  displayName: "John Smith"
lastname: "Smith"   -->  + " "          -->  

dept_code: "ENG"    -->  [Lookup]       -->  department: "Engineering"
                         ENG->Engineering

hire_date:          -->  [Date Compare] -->  lifecycleState: "Active"
"01/15/2024"             vs today
```

#### Why Do Transforms Exist?

Because source data is NEVER perfect:

- **Different formats** - dates, phone numbers, names
- **Messy data** - extra spaces, special characters
- **Missing data** - nulls, empty strings
- **Codes that need translation** - dept codes to full names
- **Data in wrong places** - need to combine or split

Transforms **clean, format, calculate, and translate** data before it becomes identity attributes.

---

### Transform vs Mapping vs Rule - What's the Difference?

Let me clarify these three concepts:

#### 1. DIRECT MAPPING (No transformation)
```
Source Attribute --> Identity Attribute (exactly as-is)

Example:
  Source: email = "john@company.com"
  Mapping: Direct
  Identity: email = "john@company.com"

When to use: When source data is already perfect
```

#### 2. TRANSFORM (Built-in calculations)
```
Source Attribute(s) --> TRANSFORM --> Identity Attribute (calculated value)

Example:
  Source: firstname = "John", lastname = "Smith"
  Transform: Concatenation (firstname + " " + lastname)
  Identity: displayName = "John Smith"

When to use: 80% of scenarios - data needs formatting/calculation
```

#### 3. RULE (Custom code)
```
Source Attribute(s) --> RULE (custom logic) --> Identity Attribute (complex calculation)

Example:
  Source: Multiple date fields, status flags, complex business logic
  Rule: Custom Java/BeanShell code with loops, external lookups, etc.
  Identity: lifecycleState = calculated by complex logic

When to use: When transforms can't handle the complexity (rare)
```

#### The Progression

1. Try **Direct Mapping** first (if data is perfect)
2. Use **Transforms** for 99% of data manipulation needs
3. Escalate to **Rule** only when transforms can't do it

---

### Where Are Transforms Used?

Transforms appear in several places in ISC:

#### 1. Identity Profile Attribute Mappings ⭐ MOST COMMON
```
Location: Admin > Identity Profiles > [Profile] > Mappings

This is where you transform source data into identity attributes:
- firstname, lastname --> displayName
- dept_code --> department (full name)
- hire_date, term_date --> cloudLifecycleState
- manager_id --> managerName (via correlation)
```

#### 2. Lifecycle State Mapping ⭐ CRITICAL
```
Location: Identity Profile > Mappings > cloudLifecycleState attribute

This special attribute determines which lifecycle state the identity is in:
- Pre-hire, Active, Leave, Terminated, etc.
- Drives access provisioning
```

#### 3. Provisioning Policies (Advanced)
```
Location: Source > Provisioning Policies

Transforms can modify data BEFORE it's provisioned to target systems
Less common for beginners
```

#### 4. Access Profiles (Advanced)
```
Dynamic access based on transformed attribute values
We'll cover this later
```

> **For Day 1, we focus on #1 and #2 - Identity Profile Mappings!**

---

### When Does a Transform Execute?

This is important to understand:

Transforms execute during **Identity Processing**, which happens:

#### 1. Event-Based (Immediate)
- Right after account aggregation
- When source data changes
- When you manually update an identity

#### 2. Scheduled (Twice Daily)
- Morning and evening for all identities
- If periodic refresh is needed

#### 3. Manual (When you trigger it)
- When you click "Apply Changes" on Identity Profile
- When you manually process identities

#### What This Means

- Change a transform → Identity processing runs → Attributes recalculate
- New data comes from source → Aggregation → Transform executes → Attributes update

---

### Transform Input/Output Model

Every transform follows this pattern:
```
INPUT (Attributes) --> TRANSFORM (Logic) --> OUTPUT (Value)
```

#### Example 1: Simple Transform
```
INPUT:
  firstname = "John"
  lastname = "Smith"

TRANSFORM: Concatenation
  Combine: firstname + " " + lastname

OUTPUT:
  "John Smith"
```

#### Example 2: Chained Transforms
```
INPUT:
  firstname = "John"
  lastname = "Smith"

TRANSFORM 1: Concatenation
  Result: "John Smith"
  ↓
TRANSFORM 2: Upper
  Input: "John Smith" (output from Transform 1)
  Result: "JOHN SMITH"
  ↓
OUTPUT:
  "JOHN SMITH"
```

> **Key Insight:** Output of Transform N becomes input to Transform N+1

This is called **TRANSFORM CHAINING** and it's incredibly powerful!

---

### Checkpoint Quiz 1

Before moving forward, answer these:

#### Q1: What's the difference between a direct mapping and a transform?

<details>
<summary>Click to reveal answer</summary>

**Answer:** 
- **Direct mapping** copies source data exactly as-is to identity attribute
- **Transform** calculates/modifies/formats the data before creating the identity attribute

Examples:
- Direct mapping: `source.email` → `identity.email` (no change)  
- Transform: `source.firstname + source.lastname` → `identity.displayName` (calculated)

</details>

#### Q2: When do transforms execute?

<details>
<summary>Click to reveal answer</summary>

**Answer:** 
During Identity Processing, which happens:
1. After account aggregation (event-based)
2. Twice daily for scheduled processing
3. When you manually trigger "Apply Changes"

</details>

#### Q3: Can transforms change data in the source system?

<details>
<summary>Click to reveal answer</summary>

**Answer:** 
**NO!** Transforms are **read-only**. They calculate identity attributes FROM source data, but never modify source data.

Source data stays in the source system unchanged.

</details>

#### Q4: What happens when you chain transforms?

<details>
<summary>Click to reveal answer</summary>

**Answer:** 
The output of Transform 1 becomes the input to Transform 2, which becomes input to Transform 3, etc.

Example: Concatenate → Lower → Substring  
Each step feeds into the next step.

</details>

---

## Part 2: Hands-On Setup (45 minutes)

Now let's get into your SailPoint environment and explore!

### STEP 1: Access Your Environment (5 min)

Open your SailPoint ISC tenant:
```
URL format: https://[your-tenant].identitynow.com
or: https://[your-tenant].api.cloud.sailpoint.com

Log in with your admin credentials
```

If you don't have access:
- Request sandbox from your SailPoint rep
- Use your company's dev/test environment
- Ask your team lead for access

> **Can't proceed without access!** Everything after this requires hands-on.

---

### STEP 2: Navigate to Identity Profile Mappings (10 min)

Let's explore where transforms live:

#### Navigation Path

1. Click **"Admin"** (top navigation)
2. Click **"Identities"** (left sidebar)
3. Click **"Identity Profiles"**
4. You'll see a list of existing Identity Profiles
5. Click on **ANY profile** (pick one that has identities associated)
6. Click **"Mappings"** in the left panel

#### What You're Looking At

You should see a table with columns:

| Column | Description |
|--------|-------------|
| **Attribute** | The identity attribute name |
| **Source** | Where the data comes from |
| **Transform** | If any transform is applied |
| **Example** | Sample value |

#### Explore This Interface

**Tasks:**
- [ ] Find the "displayName" attribute
- [ ] See how it's currently mapped
- [ ] Find the "email" attribute
- [ ] Find the "cloudLifecycleState" attribute (if mapped)
- [ ] Scroll through - see what other attributes exist

> **DON'T CLICK SAVE OR APPLY CHANGES YET!** We're just exploring for now.

---

### STEP 3: Explore the Transform Dropdown (15 min)

Now let's see what transforms are available:

#### Instructions

1. In the Mappings page, find any attribute (like displayName)
2. Click on the attribute row
3. In the overlay that appears, look for "Transform" dropdown
4. Click the Transform dropdown
5. Scroll through the list - see what's available!

#### You Should See Transforms Like
```
✓ Account Attribute
✓ Concatenation
✓ Conditional
✓ Date Compare
✓ Date Format
✓ Date Math
✓ E.164 Phone Format
✓ First Valid
✓ Index
✓ ISO3166
✓ Lookup
✓ Lower
✓ Random Alphanumeric
✓ Random Numeric
✓ Reference
✓ Replace
✓ Split
✓ Static
✓ Substring
✓ Trim
✓ Upper
... and many more!
```

#### Your Tasks

- [ ] Count how many transforms are available (there are 40+!)
- [ ] Read the names - try to guess what each one does
- [ ] Click on "Concatenation" - see what configuration options appear
- [ ] Click on "Conditional" - notice it has different inputs than Concatenation
- [ ] Click on "Lookup" - see it requires a lookup table
- [ ] Click "Cancel" - don't save anything yet!

> **Important:** Each transform has different inputs and configuration. We'll learn them one by one.

---

### STEP 4: Use Identity Preview (15 min)

This is **THE MOST IMPORTANT** debugging tool for transforms!

#### What is Identity Preview?

It shows you:
- Source data for a specific identity
- What transforms are doing to that data
- Final calculated values

#### How to Use It

1. Still in Mappings page
2. At the top, you'll see "Preview" button or a dropdown to select an identity
3. Select any identity from the dropdown
4. Click "Preview" or the identity automatically loads

#### What You'll See

The preview shows:
```
Attribute: displayName
├── Source Value: (shows raw source data)
├── Transform Output: (shows what transform calculated)
└── Final Value: (what will be in identity attribute)
```

#### Explore

**Tasks:**
- [ ] Pick an attribute that has a transform
- [ ] Look at the Source Value
- [ ] Look at the Transform Output
- [ ] See how the transform changed the data
- [ ] Try a different identity from the dropdown
- [ ] See how values differ per identity

> **THIS IS HOW YOU DEBUG TRANSFORMS!**
> 
> When a transform doesn't work, Identity Preview shows you exactly what's happening at each step.

---

### Checkpoint Quiz 2

#### Q1: Where do you configure transforms in ISC?

<details>
<summary>Click to reveal answer</summary>

**Answer:** 
```
Admin > Identities > Identity Profiles > [Select Profile] > Mappings
```

Each attribute mapping can have transforms applied.

</details>

#### Q2: What does Identity Preview show you?

<details>
<summary>Click to reveal answer</summary>

**Answer:** 

For a specific identity, it shows:
- Source attribute values (raw data)
- Transform output (what the transform calculated)
- Final identity attribute value

This is essential for debugging!

</details>

#### Q3: Should you click "Save" or "Apply Changes" while learning?

<details>
<summary>Click to reveal answer</summary>

**Answer:** 

**NO!** Not yet. 

- **Save** - saves the configuration but doesn't apply it
- **Apply Changes** - triggers identity processing for all identities (can take time)

While learning, explore but don't save until you're sure. Use Preview to test first!

</details>

---

## Morning Session Complete!

### You've Learned

- ✅ What transforms are and why they exist  
- ✅ Transform vs mapping vs rule  
- ✅ Where transforms are used  
- ✅ When transforms execute  
- ✅ Input/output model  
- ✅ How to navigate to transforms in ISC  
- ✅ How to use Identity Preview  

### Before Continuing to Afternoon, Ensure

- [ ] You can access your ISC tenant
- [ ] You can navigate to Identity Profile > Mappings
- [ ] You can see the transform dropdown
- [ ] You can use Identity Preview
- [ ] You understand the concepts above

---

## Next Steps

Continue to the **Afternoon Session** where we'll build actual transforms hands-on!

### What You'll Build

1. **Static** - Hardcoded values
2. **Concatenation** - Combine strings
3. **Substring** - Extract parts
4. **Upper/Lower** - Case conversion
5. **Trim** - Remove whitespace
6. **Replace** - Swap characters

You'll build **10+ working transforms** and master **transform chaining**!

---

**Ready to continue? Move on to the Afternoon Session!**

### Practice Tips
- Always use Identity Preview before applying changes
- Test with multiple identities to see different data scenarios
- Start simple, then add complexity
- Document your transform chains for future reference

---

*Part of the SailPoint ISC Transforms Mastery 7-Day Study Plan*
