# Section 1.1.1: Valid Search Query Syntax and Operators

**Learning Objective:** By the end of this section, you will be able to find identities, accounts, and access in SailPoint ISC using search queries.

---

## What is Search in Identity Security Cloud?

Search is your primary tool for finding information in SailPoint ISC. Think of it like Google, but specifically for your organization's identity data.

**What can you search for?**
- **Identities** - People in your organization (employees, contractors, etc.)
- **Accounts** - Login accounts those people have in different systems
- **Access** - Permissions, roles, and entitlements people have
- **Sources** - Connected systems (Active Directory, Salesforce, etc.)

**Why is search important for an ISC Administrator?**
- Find specific users quickly
- Troubleshoot provisioning issues
- Generate reports
- Identify security risks (orphaned accounts, excessive permissions)
- Answer audit questions ("Who has access to X?")

---

## How to Access Search in ISC

### **Step 1: Log into SailPoint ISC**

Navigate to your tenant URL (usually `https://yourcompany.identitynow.com`)

### **Step 2: Locate the Search Feature**

**Method 1: Global Search Bar (Top of Screen)**
- Look at the top of your screen
- You'll see a search bar with a magnifying glass icon 🔍
- This is the **quick search** - fastest way to find something

**Method 2: Advanced Search (For Detailed Queries)**
- Click on the main menu (usually top-left)
- Navigate to **Search** or **Identities** → **Search**
- This gives you more filtering options

**What you'll see:**
- A search input box
- Filter options (Object Type, Status, etc.)
- Results area (shows matches below)

---

## Understanding Search Results

### **What Information is Displayed?**

When you search, you'll see results organized by **object type**:

**For Identities:**
```
Name: John Smith
Email: john.smith@company.com
Status: Active
Department: IT
Manager: Jane Doe
```

**For Accounts:**
```
Account Name: jsmith
Source: Active Directory
Identity: John Smith
Status: Enabled
```

**For Access (Roles/Entitlements):**
```
Name: VPN Access
Type: Access Profile
Source: Cisco VPN
Owner: IT Security
```

---

## Basic Search - Start Here

### **Simple Text Search**

**Example 1: Search by Name**

Just type a name in the search box:
```
John Smith
```

**What happens:** ISC searches across all fields and returns anything matching "John Smith"

**Example 2: Search by Email**
```
john.smith@company.com
```

**What happens:** Returns the identity with that email address

### **Try It Yourself: Exercise 1**

**Task:** Find an identity named "Sarah Johnson"

**Steps:**
1. Go to the search bar
2. Type: `Sarah Johnson`
3. Press Enter

**Expected Result:** You should see Sarah Johnson's identity profile

---

## Field-Specific Search (More Precise)

Sometimes you want to search ONLY in a specific field (like department or email). This is where **field operators** help.

### **The Colon `:` Operator**

**Syntax:** `fieldname:value`

This tells ISC: "Search for 'value' ONLY in the 'fieldname' field"

### **Common Fields You Can Search**

| Field Name | What It Searches | Example |
|------------|------------------|---------|
| `name` | Identity username | `name:john.smith` |
| `email` | Email address | `email:john@company.com` |
| `department` | Department name | `department:IT` |
| `cloudLifecycleState` | Active, Inactive, etc. | `cloudLifecycleState:active` |
| `manager` | Manager's name | `manager:Jane Doe` |

### **Examples: Field-Specific Searches**

**Example 1: Find all identities in IT department**
```
department:IT
```

**What happens:** Returns ONLY identities where the department field equals "IT"

**Example 2: Find all active identities**
```
cloudLifecycleState:active
```

**What happens:** Returns all identities with status "active"

**Example 3: Find identity by email**
```
email:john.smith@company.com
```

**What happens:** Returns the specific identity with that email

### **Try It Yourself: Exercise 2**

**Task:** Find all identities in the Finance department

**Question:** What search query should you use?

<details>
<summary>Click to see answer</summary>
```
department:Finance
```

</details>

---

## Using Wildcards (Finding Partial Matches)

Sometimes you don't know the complete name. Wildcards help you find partial matches.

### **The Star `*` Wildcard**

The `*` symbol means "match anything here"

**Syntax:** `fieldname:value*`

### **Wildcard Examples**

**Example 1: Find anyone whose name starts with "John"**
```
name:john*
```

**Matches:**
- john.smith
- johnny.appleseed
- johnathan.doe

**Example 2: Find all emails from company.com**
```
email:*@company.com
```

**Matches:**
- john@company.com
- sarah@company.com
- mike@company.com

**Example 3: Find names containing "smith"**
```
name:*smith*
```

**Matches:**
- john.smith
- sarah.smithson
- blacksmith.joe

### **⚠️ Important Wildcard Rule**

**AVOID** starting searches with wildcards (slow performance):

❌ **BAD:**
```
name:*smith     # Very slow!
```

✅ **GOOD:**
```
name:smith*     # Much faster!
```

### **Try It Yourself: Exercise 3**

**Task:** Find all identities whose names start with "Sara"

**Question:** What search query should you use?

<details>
<summary>Click to see answer</summary>
```
name:sara*
```

This will find Sara, Sarah, Sarahbeth, etc.

</details>

---

## Combining Searches with AND

What if you want to find identities that match MULTIPLE conditions?

### **The `AND` Operator**

**Syntax:** `condition1 AND condition2`

Both conditions MUST be true.

### **AND Examples**

**Example 1: Find active identities in IT department**
```
department:IT AND cloudLifecycleState:active
```

**What happens:** Returns identities that are BOTH in IT AND active

**Example 2: Find John in IT department**
```
name:john* AND department:IT
```

**What happens:** Returns anyone named John (or Johnny, Johnathan) who works in IT

**Example 3: Find active employees with manager Jane Doe**
```
cloudLifecycleState:active AND manager:"Jane Doe"
```

**Note:** We use quotes `"Jane Doe"` because the manager name has a space

### **Try It Yourself: Exercise 4**

**Task:** Find all active identities in the HR department

**Question:** What search query should you use?

<details>
<summary>Click to see answer</summary>
```
department:HR AND cloudLifecycleState:active
```

Or:
```
cloudLifecycleState:active AND department:HR
```

(Order doesn't matter with AND)

</details>

---

## Combining Searches with OR

What if you want to find identities that match ANY of several conditions?

### **The `OR` Operator**

**Syntax:** `condition1 OR condition2`

At least ONE condition must be true.

### **OR Examples**

**Example 1: Find identities in IT or HR**
```
department:IT OR department:HR
```

**What happens:** Returns identities in EITHER IT OR HR (or both)

**Example 2: Find identities in multiple locations**
```
location:NYC OR location:Boston OR location:Chicago
```

**What happens:** Returns identities in any of these three cities

### **Try It Yourself: Exercise 5**

**Task:** Find all identities in either Finance or Accounting departments

**Question:** What search query should you use?

<details>
<summary>Click to see answer</summary>
```
department:Finance OR department:Accounting
```

</details>

---

## Excluding Results with NOT

What if you want to find everything EXCEPT something?

### **The `NOT` Operator**

**Syntax:** `NOT condition`

Excludes anything matching the condition.

### **NOT Examples**

**Example 1: Find all identities NOT in IT**
```
NOT department:IT
```

**What happens:** Returns all identities EXCEPT those in IT

**Example 2: Find active identities NOT in IT**
```
cloudLifecycleState:active AND NOT department:IT
```

**What happens:** Active identities in any department except IT

**Example 3: Find identities without a manager**
```
NOT _exists_:manager
```

**What happens:** Returns identities where the manager field is empty

### **Try It Yourself: Exercise 6**

**Task:** Find all active identities who are NOT in the IT department

**Question:** What search query should you use?

<details>
<summary>Click to see answer</summary>
```
cloudLifecycleState:active AND NOT department:IT
```

</details>

---

## Grouping with Parentheses

When combining AND, OR, and NOT, use parentheses `()` to group conditions.

### **Why Use Parentheses?**

Without parentheses, search might be confusing:
```
department:IT OR department:HR AND location:NYC
```

**Question:** Does this mean:
- A) (IT or HR) and NYC?
- B) IT or (HR and NYC)?

**Answer:** Without parentheses, ISC reads it as option B.

### **Using Parentheses for Clarity**

**Example 1: Find IT or HR employees in NYC**
```
(department:IT OR department:HR) AND location:NYC
```

**What happens:** Must be in NYC, and must be in either IT or HR

**Example 2: Active employees in IT, HR, or Finance**
```
cloudLifecycleState:active AND (department:IT OR department:HR OR department:Finance)
```

**What happens:** Active identities in any of the three departments

### **Try It Yourself: Exercise 7**

**Task:** Find active identities in either IT or HR department who are located in either NYC or Boston

**Question:** What search query should you use?

<details>
<summary>Click to see answer</summary>
```
cloudLifecycleState:active AND (department:IT OR department:HR) AND (location:NYC OR location:Boston)
```

</details>

---

## Searching with Quotes (Exact Phrases)

Use quotes `""` when searching for exact phrases with spaces.

### **When to Use Quotes**

**Example 1: Search for exact name**

❌ **Without quotes:**
```
name:John Smith
```
This searches for "John" in the name field and "Smith" anywhere globally (wrong!)

✅ **With quotes:**
```
name:"John Smith"
```
This searches for the exact phrase "John Smith" in the name field (correct!)

**Example 2: Search for department with spaces**
```
department:"Information Technology"
```

**Example 3: Search for manager name**
```
manager:"Jane Doe"
```

### **Try It Yourself: Exercise 8**

**Task:** Find the identity with the exact name "Mary Jane Watson"

**Question:** What search query should you use?

<details>
<summary>Click to see answer</summary>
```
name:"Mary Jane Watson"
```

</details>

---

## Special Operators: Checking if Field Exists

Sometimes you want to find identities where a field has a value (or doesn't).

### **The `_exists_` Operator**

**Syntax:** `_exists_:fieldname`

Finds records WHERE the field HAS a value.

### **EXISTS Examples**

**Example 1: Find identities WITH a manager**
```
_exists_:manager
```

**Example 2: Find identities WITHOUT an email**
```
NOT _exists_:email
```

**Example 3: Find active identities without a manager**
```
cloudLifecycleState:active AND NOT _exists_:manager
```

### **Try It Yourself: Exercise 9**

**Task:** Find all active identities who do NOT have an email address

**Question:** What search query should you use?

<details>
<summary>Click to see answer</summary>
```
cloudLifecycleState:active AND NOT _exists_:email
```

</details>

---

## Searching Accounts (Not Just Identities)

So far we've focused on identities. You can also search for accounts.

### **Common Account Fields**

| Field Name | What It Searches | Example |
|------------|------------------|---------|
| `name` | Account username | `name:jsmith` |
| `source.name` | Source system name | `source.name:"Active Directory"` |
| `uncorrelated` | Orphaned accounts | `uncorrelated:true` |
| `disabled` | Account status | `disabled:true` |
| `identity.name` | Associated identity | `identity.name:"John Smith"` |

### **Account Search Examples**

**Example 1: Find uncorrelated accounts**
```
uncorrelated:true
```

**What this means:** Accounts that exist in a source system but aren't linked to any identity in ISC

**Example 2: Find disabled accounts in Active Directory**
```
disabled:true AND source.name:"Active Directory"
```

**Example 3: Find all accounts for John Smith**
```
identity.name:"John Smith"
```

### **Try It Yourself: Exercise 10**

**Task:** Find all uncorrelated accounts in the "Salesforce" source

**Question:** What search query should you use?

<details>
<summary>Click to see answer</summary>
```
uncorrelated:true AND source.name:Salesforce
```

</details>

---

## Common Beginner Mistakes

### **Mistake 1: Forgetting Quotes for Multi-Word Values**

❌ **Wrong:**
```
department:Information Technology
```

✅ **Correct:**
```
department:"Information Technology"
```

### **Mistake 2: Using Uppercase Field Names**

❌ **Wrong:**
```
Department:IT
```

✅ **Correct:**
```
department:IT
```

**Remember:** Field names are lowercase (but values can be any case)

### **Mistake 3: Starting with a Wildcard**

❌ **Slow:**
```
name:*smith
```

✅ **Fast:**
```
name:smith*
```

### **Mistake 4: Forgetting Parentheses with OR and AND**

❌ **Confusing:**
```
department:IT OR department:HR AND location:NYC
```

✅ **Clear:**
```
(department:IT OR department:HR) AND location:NYC
```

---

## Real-World Search Scenarios

### **Scenario 1: New Hire Onboarding**

**Question:** "Show me all identities who started in the last 30 days and are in IT"

**Search Query:**
```
department:IT AND created:[now-30d TO now]
```

(We'll cover date ranges in detail later)

### **Scenario 2: Security Audit**

**Question:** "Find all active identities without a manager assigned"

**Search Query:**
```
cloudLifecycleState:active AND NOT _exists_:manager
```

### **Scenario 3: Offboarding**

**Question:** "Find all inactive identities in the Finance department"

**Search Query:**
```
cloudLifecycleState:inactive AND department:Finance
```

### **Scenario 4: Troubleshooting**

**Question:** "Find all accounts in Active Directory that aren't linked to an identity"

**Search Query:**
```
uncorrelated:true AND source.name:"Active Directory"
```

---

## Quick Reference Card

### **Basic Operators**

| Operator | Purpose | Example |
|----------|---------|---------|
| `:` | Field search | `department:IT` |
| `*` | Wildcard | `name:john*` |
| `AND` | Both must match | `department:IT AND cloudLifecycleState:active` |
| `OR` | Either can match | `department:IT OR department:HR` |
| `NOT` | Exclude | `NOT department:IT` |
| `()` | Group conditions | `(department:IT OR department:HR) AND location:NYC` |
| `""` | Exact phrase | `name:"John Smith"` |
| `_exists_:` | Field has value | `_exists_:manager` |

### **Common Identity Fields**

| Field | Example |
|-------|---------|
| `name` | `name:john.smith` |
| `email` | `email:john@company.com` |
| `department` | `department:IT` |
| `cloudLifecycleState` | `cloudLifecycleState:active` |
| `manager` | `manager:"Jane Doe"` |
| `location` | `location:NYC` |

### **Common Account Fields**

| Field | Example |
|-------|---------|
| `name` | `name:jsmith` |
| `source.name` | `source.name:"Active Directory"` |
| `uncorrelated` | `uncorrelated:true` |
| `disabled` | `disabled:true` |
| `identity.name` | `identity.name:"John Smith"` |

---

## Practice Exercises - Test Your Knowledge

**Exercise 11:** Find all identities named Sarah in the HR department

<details>
<summary>Click to see answer</summary>
```
name:sarah* AND department:HR
```

</details>

---

**Exercise 12:** Find all active identities in either IT, HR, or Finance departments

<details>
<summary>Click to see answer</summary>
```
cloudLifecycleState:active AND (department:IT OR department:HR OR department:Finance)
```

</details>

---

**Exercise 13:** Find all disabled accounts in Active Directory

<details>
<summary>Click to see answer</summary>
```
disabled:true AND source.name:"Active Directory"
```

</details>

---

**Exercise 14:** Find all active identities who do NOT have an email address and are in the IT department

<details>
<summary>Click to see answer</summary>
```
cloudLifecycleState:active AND NOT _exists_:email AND department:IT
```

</details>

---

**Exercise 15:** Find all identities whose email ends with @contractor.com

<details>
<summary>Click to see answer</summary>
```
email:*@contractor.com
```

</details>

---

## What's Next?

In the next section (**1.1.2 Understanding Search Results and Filters**), you'll learn:
- How to read and interpret search results
- Using filters to narrow results
- Pagination and result limits
- Exporting search results

---

## Key Takeaways

✅ **Search is your most-used tool** - Master it early  
✅ **Field-specific searches** are more precise than general searches  
✅ **Use wildcards (`*`)** for partial matches, but avoid leading wildcards  
✅ **Combine with AND, OR, NOT** for complex queries  
✅ **Use parentheses** to group conditions clearly  
✅ **Use quotes** for multi-word values  
✅ **Practice makes perfect** - Try these searches in your test tenant  

---

## Additional Resources

📚 **Official SailPoint Documentation:**
- [SailPoint Search Documentation](https://documentation.sailpoint.com/saas/help/search/index.html)
- [Search Query Syntax Guide](https://developer.sailpoint.com/idn/docs/search)

🎥 **Recommended Practice:**
- Log into your ISC tenant
- Try each example query in this guide
- Modify the queries to match your organization's data
- Create your own search queries for common tasks

---
