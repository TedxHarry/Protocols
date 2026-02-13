# Section 1.1: Search and Reporting

**Master the essential skills for finding information and generating reports in SailPoint Identity Security Cloud**

---

## Table of Contents

- [1.1.1 Valid Search Query Syntax and Operators](#111-valid-search-query-syntax-and-operators)
- [1.1.2 Understanding Search Results and Filters](#112-understanding-search-results-and-filters)
- [1.1.3 Creating Custom Search Reports](#113-creating-custom-search-reports)
- [1.1.4 Scheduling and Managing Reports](#114-scheduling-and-managing-reports)

---

# 1.1.1 Valid Search Query Syntax and Operators

**Learning Objective:** Find identities, accounts, and access in SailPoint ISC using search queries.

---

## What is Search in Identity Security Cloud?

Search is your primary tool for finding information in SailPoint ISC. Think of it like Google, but specifically for your organization's identity data.

**What can you search for?**
- **Identities** - People in your organization (employees, contractors)
- **Accounts** - Login accounts in different systems
- **Access** - Permissions, roles, and entitlements
- **Sources** - Connected systems (Active Directory, Salesforce)

**Why is search important?**
- Find specific users quickly
- Troubleshoot provisioning issues
- Generate compliance reports
- Identify security risks
- Answer audit questions

---

## How to Access Search in ISC

### **Opening the Search Interface**

**Method 1: Global Search Bar (Quick Search)**
1. Look at the top of your ISC screen
2. Find the search bar with magnifying glass icon 🔍
3. Click and start typing

**Method 2: Advanced Search**
1. Click the main menu (top navigation)
2. Select **Identities** or **Search**
3. Use the full search interface with filters

**What you'll see:**
- Search input box
- Object type filters (Identities, Accounts, Access)
- Results display area
- Export and action buttons

---

## Basic Search - Start Here

### **Simple Text Search**

Just type what you're looking for:
```
John Smith
```

**What happens:** ISC searches all fields and returns matches

**Examples:**
```
Sarah Johnson          → Finds identity named Sarah Johnson
john.smith@company.com → Finds identity with this email
IT                     → Finds anything containing "IT"
```

---

## Field-Specific Search

Search ONLY in specific fields for more precise results.

### **The Colon `:` Operator**

**Format:** `fieldname:value`

**Common Identity Fields:**

| Field | What It Searches | Example |
|-------|------------------|---------|
| `name` | Username | `name:john.smith` |
| `email` | Email address | `email:john@company.com` |
| `department` | Department | `department:IT` |
| `cloudLifecycleState` | Status | `cloudLifecycleState:active` |
| `manager` | Manager name | `manager:"Jane Doe"` |

### **Examples**
```
department:IT
→ All identities in IT department

cloudLifecycleState:active
→ All active identities

email:john.smith@company.com
→ Identity with this specific email
```

---

## Using Wildcards

Find partial matches when you don't know the complete value.

### **The Star `*` Wildcard**

**Format:** `fieldname:value*`

The `*` means "match anything here"

### **Examples**
```
name:john*
→ Matches: john.smith, johnny.appleseed, johnathan.doe

email:*@company.com
→ Matches: anyone@company.com

name:*smith*
→ Matches: john.smith, sarah.smithson, blacksmith
```

### **⚠️ Performance Rule**

❌ **AVOID (Very Slow):**
```
name:*smith
```

✅ **USE (Much Faster):**
```
name:smith*
```

**Tip:** Put wildcards at the END, not the beginning

---

## Combining Searches with AND

Find items matching MULTIPLE conditions.

### **The `AND` Operator**

**Format:** `condition1 AND condition2`

Both conditions MUST be true.

### **Examples**
```
department:IT AND cloudLifecycleState:active
→ Active identities in IT

name:john* AND department:IT
→ Anyone named John in IT department

manager:"Jane Doe" AND cloudLifecycleState:active
→ Active identities managed by Jane Doe
```

---

## Combining Searches with OR

Find items matching ANY of several conditions.

### **The `OR` Operator**

**Format:** `condition1 OR condition2`

At least ONE condition must be true.

### **Examples**
```
department:IT OR department:HR
→ Identities in either IT or HR

location:NYC OR location:Boston
→ Identities in either city

department:IT OR department:HR OR department:Finance
→ Identities in any of these three departments
```

---

## Excluding Results with NOT

Find everything EXCEPT something.

### **The `NOT` Operator**

**Format:** `NOT condition`

Excludes matching results.

### **Examples**
```
NOT department:IT
→ All identities NOT in IT

cloudLifecycleState:active AND NOT department:IT
→ Active identities in any department except IT

NOT _exists_:manager
→ Identities without a manager assigned
```

---

## Grouping with Parentheses

Control the order of operations in complex queries.

### **Using `()` for Clarity**

**Format:** `(condition1 OR condition2) AND condition3`

### **Examples**
```
(department:IT OR department:HR) AND location:NYC
→ IT or HR employees in NYC

cloudLifecycleState:active AND (department:IT OR department:HR OR department:Finance)
→ Active identities in any of three departments

(name:john* OR name:jane*) AND department:IT
→ Anyone named John or Jane in IT
```

---

## Using Quotes for Exact Phrases

Search for exact multi-word values.

### **The `""` Quote Operator**

**Format:** `fieldname:"exact value"`

### **When to Use Quotes**

**Names with spaces:**
```
name:"John Smith"
manager:"Jane Doe"
```

**Departments with spaces:**
```
department:"Information Technology"
department:"Human Resources"
```

**Without quotes (WRONG):**
```
name:John Smith
→ Searches for "John" in name field and "Smith" everywhere
```

**With quotes (CORRECT):**
```
name:"John Smith"
→ Searches for exact phrase "John Smith" in name field
```

---

## Checking if Fields Exist

Find records where fields have values (or don't).

### **The `_exists_` Operator**

**Format:** `_exists_:fieldname`

Finds records WHERE field HAS a value.

### **Examples**
```
_exists_:manager
→ Identities WITH a manager

NOT _exists_:manager
→ Identities WITHOUT a manager

NOT _exists_:email
→ Identities missing email addresses

cloudLifecycleState:active AND NOT _exists_:manager
→ Active identities without assigned manager
```

---

## Searching Accounts

Search for accounts in source systems.

### **Common Account Fields**

| Field | What It Searches | Example |
|-------|------------------|---------|
| `name` | Account username | `name:jsmith` |
| `source.name` | Source system | `source.name:"Active Directory"` |
| `uncorrelated` | Orphaned status | `uncorrelated:true` |
| `disabled` | Account status | `disabled:true` |
| `identity.name` | Linked identity | `identity.name:"John Smith"` |

### **Examples**
```
uncorrelated:true
→ Accounts not linked to any identity

disabled:true AND source.name:"Active Directory"
→ Disabled accounts in Active Directory

source.name:Salesforce
→ All accounts in Salesforce

identity.name:"John Smith"
→ All accounts belonging to John Smith
```

---

## Common Search Mistakes

### **Mistake 1: Forgetting Quotes**

❌ **Wrong:**
```
department:Information Technology
```

✅ **Correct:**
```
department:"Information Technology"
```

### **Mistake 2: Wrong Field Name Case**

❌ **Wrong:**
```
Department:IT
Name:john
```

✅ **Correct:**
```
department:IT
name:john
```

**Remember:** Field names are LOWERCASE

### **Mistake 3: Leading Wildcards**

❌ **Slow:**
```
name:*smith
```

✅ **Fast:**
```
name:smith*
```

### **Mistake 4: Missing Parentheses**

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

### **Scenario 1: Find New Hires**

**Question:** "Show me IT employees who started recently"
```
department:IT AND cloudLifecycleState:active
```

### **Scenario 2: Security Audit**

**Question:** "Find active users without managers"
```
cloudLifecycleState:active AND NOT _exists_:manager
```

### **Scenario 3: Offboarding**

**Question:** "Find inactive Finance employees"
```
cloudLifecycleState:inactive AND department:Finance
```

### **Scenario 4: Orphaned Accounts**

**Question:** "Find AD accounts not linked to identities"
```
uncorrelated:true AND source.name:"Active Directory"
```

---

## Quick Reference Guide

### **Operators**

| Operator | Purpose | Example |
|----------|---------|---------|
| `:` | Field search | `department:IT` |
| `*` | Wildcard | `name:john*` |
| `AND` | Both conditions | `department:IT AND cloudLifecycleState:active` |
| `OR` | Either condition | `department:IT OR department:HR` |
| `NOT` | Exclude | `NOT department:IT` |
| `()` | Group | `(department:IT OR department:HR) AND location:NYC` |
| `""` | Exact phrase | `name:"John Smith"` |
| `_exists_:` | Has value | `_exists_:manager` |

### **Common Fields**

**Identities:**
- `name` - Username
- `email` - Email address
- `department` - Department
- `cloudLifecycleState` - active, inactive
- `manager` - Manager name
- `location` - Location

**Accounts:**
- `name` - Account name
- `source.name` - Source system
- `uncorrelated` - true/false
- `disabled` - true/false
- `identity.name` - Linked identity

---

## Practice Exercises

**Exercise 1:** Find all active HR employees
<details>
<summary>Answer</summary>
```
department:HR AND cloudLifecycleState:active
```
</details>

---

**Exercise 2:** Find anyone named Sarah in Finance
<details>
<summary>Answer</summary>
```
name:sarah* AND department:Finance
```
</details>

---

**Exercise 3:** Find active employees in IT, HR, or Finance
<details>
<summary>Answer</summary>
```
cloudLifecycleState:active AND (department:IT OR department:HR OR department:Finance)
```
</details>

---

**Exercise 4:** Find disabled accounts in Salesforce
<details>
<summary>Answer</summary>
```
disabled:true AND source.name:Salesforce
```
</details>

---

**Exercise 5:** Find active employees without email addresses
<details>
<summary>Answer</summary>
```
cloudLifecycleState:active AND NOT _exists_:email
```
</details>

---

# 1.1.2 Understanding Search Results and Filters

**Learning Objective:** Read, interpret, and filter search results effectively.

---

## Understanding Search Results

When you run a search, ISC displays results organized by object type.

### **Result Display Components**

**Top Section:**
- Total results count
- Object type tabs (Identities, Accounts, Access)
- Filter panel
- Export options

**Results Area:**
- List of matching items
- Key information for each item
- Actions you can perform
- Pagination controls

---

## Reading Identity Results

### **What You See for Each Identity**
```
Name: John Smith
Email: john.smith@company.com
Department: IT
Status: Active
Manager: Jane Doe
Last Modified: 2024-02-10
```

### **Available Information**

| Field | Description |
|-------|-------------|
| **Name** | Display name / username |
| **Email** | Contact email |
| **Department** | Organizational unit |
| **Status** | Active, Inactive, etc. |
| **Manager** | Direct supervisor |
| **Accounts** | Number of linked accounts |
| **Last Modified** | Recent change date |

### **Clicking on a Result**

Click any identity to see:
- Full identity details
- All accounts
- Current access (roles, entitlements)
- Access history
- Lifecycle events

---

## Reading Account Results

### **What You See for Each Account**
```
Account: jsmith
Source: Active Directory
Identity: John Smith
Status: Enabled
Created: 2024-01-15
```

### **Available Information**

| Field | Description |
|-------|-------------|
| **Account Name** | Username in source |
| **Source** | System (AD, Salesforce) |
| **Identity** | Linked person |
| **Status** | Enabled/Disabled |
| **Correlation** | Linked or orphaned |
| **Entitlements** | Account permissions |

---

## Reading Access Results

### **What You See for Access Items**
```
Name: VPN Access
Type: Access Profile
Source: Cisco VPN
Owner: IT Security Team
Requestable: Yes
```

### **Available Information**

| Field | Description |
|-------|-------------|
| **Name** | Access item name |
| **Type** | Role, Access Profile, Entitlement |
| **Source** | Origin system |
| **Owner** | Responsible person/team |
| **Requestable** | Can users request it? |
| **Description** | What it grants |

---

## Using Filters

Filters help narrow search results without writing complex queries.

### **How to Access Filters**

1. Run your search
2. Look for **Filter** panel (usually left side)
3. Click filter categories to expand
4. Select filter values

### **Common Filter Types**

**Status Filters:**
- Active
- Inactive
- Pending
- Disabled

**Source Filters:**
- Active Directory
- Salesforce
- Workday
- (All connected sources)

**Type Filters:**
- Identities
- Accounts
- Roles
- Access Profiles
- Entitlements

**Date Filters:**
- Created date
- Modified date
- Last login
- Custom date ranges

---

## Applying Filters

### **Single Filter**

1. Run search: `department:IT`
2. Apply filter: Status = Active
3. Results show: Active IT employees only

### **Multiple Filters**

1. Run search: `department:IT`
2. Apply filter: Status = Active
3. Apply filter: Source = Active Directory
4. Results show: Active IT employees with AD accounts

### **Combining Search and Filters**

**Search query:**
```
department:IT OR department:HR
```

**Then apply filters:**
- Status: Active
- Location: NYC

**Final result:** Active IT or HR employees in NYC

---

## Result Pagination

Large searches return results in pages.

### **Pagination Controls**

**Bottom of results:**
- Results per page (50, 100, 250)
- Page numbers (1, 2, 3...)
- Next/Previous buttons
- Total count

### **Changing Results Per Page**

**Default:** 50 results per page

**Options:**
- 50 results
- 100 results
- 250 results

**To change:**
1. Look for "Items per page" dropdown
2. Select desired number
3. Results reload

### **Navigating Pages**

**Next page:** Click "Next" or page number
**Previous page:** Click "Previous" or page number
**Jump to page:** Click specific page number

---

## Sorting Results

Change the order of search results.

### **Available Sort Options**

**Common sorts:**
- Name (A-Z or Z-A)
- Created date (Newest or Oldest)
- Modified date (Recent or Oldest)
- Status
- Department

### **How to Sort**

1. Look for column headers
2. Click header to sort
3. Click again to reverse order

**Example:**
- Click "Name" → Sorts A-Z
- Click "Name" again → Sorts Z-A

---

## Exporting Search Results

Save search results for reporting or analysis.

### **Export Options**

**Available formats:**
- CSV (Excel compatible)
- JSON (for developers)
- PDF (for reports)

### **How to Export**

1. Run your search
2. Apply filters if needed
3. Click **Export** button
4. Select format (CSV, JSON, PDF)
5. Download file

### **What Gets Exported**

**CSV Export includes:**
- All visible columns
- All pages (not just current page)
- Applied filters
- Current sort order

**Tip:** Export downloads ALL matching results, not just the current page

---

## Result Limits

Understanding search limitations.

### **Search Limits**

**Maximum results:** 10,000 per search

**What this means:**
- Searches returning >10,000 results may not show all
- Use filters to narrow results
- Create specific queries for large datasets

### **Best Practices for Large Searches**

❌ **Too broad:**
```
cloudLifecycleState:active
→ May return 50,000+ identities
```

✅ **More specific:**
```
cloudLifecycleState:active AND department:IT AND created:[now-30d TO now]
→ Returns manageable result set
```

---

## Understanding Search Performance

### **Fast Searches**

✅ These searches run quickly:
- Specific field queries: `name:john.smith`
- Small result sets: `email:specific@email.com`
- Trailing wildcards: `name:john*`

### **Slow Searches**

❌ These searches take longer:
- Leading wildcards: `name:*smith`
- Very broad queries: `cloudLifecycleState:active`
- Complex nested queries
- Searches across all fields

### **Performance Tips**

1. **Be specific** - Use field names
2. **Limit wildcards** - Avoid leading wildcards
3. **Use filters** - Narrow results incrementally
4. **Test queries** - Start small, expand as needed

---

## Working with Search Results

### **Bulk Actions**

Select multiple results to perform actions:

**Available actions:**
- Export selected
- Assign access
- Modify attributes
- Generate report

**How to use:**
1. Check boxes next to items
2. Click bulk action button
3. Confirm action

### **Saving Searches**

Save frequently-used searches for quick access.

**How to save:**
1. Create your search query
2. Apply desired filters
3. Click **Save Search**
4. Name your search
5. Access later from saved searches

---

## Common Result Scenarios

### **Scenario 1: No Results Found**

**Possible causes:**
- No matching data exists
- Search syntax error
- Wrong field name
- Value doesn't exist

**What to do:**
1. Check spelling
2. Verify field names (lowercase)
3. Remove quotes and try again
4. Broaden search criteria

### **Scenario 2: Too Many Results**

**Problem:** 10,000+ results

**Solutions:**
1. Add more specific filters
2. Narrow date range
3. Add department or location filter
4. Use AND to combine conditions

### **Scenario 3: Unexpected Results**

**Problem:** Results don't match expectations

**Check:**
1. Parentheses placement
2. AND vs OR logic
3. NOT operator placement
4. Quote marks for multi-word values

---

## Filter Best Practices

### **Start Broad, Then Narrow**

**Step 1:** Run broad search
```
department:IT
```

**Step 2:** Apply filters
- Status: Active
- Location: NYC

**Step 3:** Add more filters if needed
- Manager: Specific person
- Created: Last 30 days

### **Combine Query and Filters**

**Best approach:**
1. Use search query for primary criteria
2. Use filters for refinement
3. Export or save when satisfied

---

## Quick Reference

### **Understanding Results**

| Element | What It Shows |
|---------|---------------|
| **Total count** | Number of matches |
| **Object tabs** | Identities, Accounts, Access |
| **Result cards** | Individual items |
| **Pagination** | Page navigation |
| **Filters** | Refinement options |
| **Export** | Download results |

### **Common Filters**

| Filter Type | Options |
|-------------|---------|
| **Status** | Active, Inactive |
| **Source** | AD, Salesforce, etc. |
| **Type** | Role, Access Profile |
| **Date** | Created, Modified |
| **Location** | Office locations |

---

## Practice Exercises

**Exercise 6:** You searched for IT department and got 5,000 results. How do you narrow to active employees only?

<details>
<summary>Answer</summary>

Apply the Status filter and select "Active"

Or add to query:
```
department:IT AND cloudLifecycleState:active
```
</details>

---

**Exercise 7:** You want to export all search results to Excel. What format should you choose?

<details>
<summary>Answer</summary>

CSV format - opens directly in Excel
</details>

---

**Exercise 8:** Your search returns 50 results but says "Total: 500". How do you see the rest?

<details>
<summary>Answer</summary>

1. Use pagination controls at bottom
2. Click "Next" or page numbers
3. Or increase "Results per page" to 100 or 250
</details>

---

# 1.1.3 Creating Custom Search Reports

**Learning Objective:** Create and configure saved search reports for regular use.

---

## What are Search Reports?

Search reports are saved search queries that you can:
- Run repeatedly
- Schedule automatically
- Share with others
- Export to files

**Why create reports?**
- Regular compliance checks
- Daily operational tasks
- Audit requirements
- Team collaboration

---

## When to Create a Report

### **Good Candidates for Reports**

✅ **Create reports for:**
- Daily uncorrelated account checks
- Weekly active user counts
- Monthly access reviews
- Quarterly compliance audits
- Regular security checks

❌ **Don't create reports for:**
- One-time searches
- Frequently changing criteria
- Personal quick lookups

---

## Creating Your First Report

### **Step-by-Step Process**

**Step 1: Build Your Search Query**

Start with a working search:
```
uncorrelated:true AND source.name:"Active Directory"
```

Test it to ensure results are correct.

**Step 2: Apply Necessary Filters**

Add filters if needed:
- Source: Active Directory
- Status: Enabled
- Created: Last 7 days

**Step 3: Save as Report**

1. Click **"Save Search"** or **"Create Report"** button
2. Report creation dialog opens

**Step 4: Configure Report Settings**

Fill in report details:
```
Report Name: Daily_AD_Uncorrelated_Accounts
Description: Uncorrelated AD accounts for daily review
Object Type: Accounts
```

**Step 5: Select Columns**

Choose which fields to include:
- ☑ Account Name
- ☑ Source
- ☑ Created Date
- ☑ Native Identity
- ☐ Last Modified (uncheck if not needed)

**Step 6: Save Report**

Click **Save** - Your report is now available!

---

## Report Configuration Options

### **Required Fields**

| Field | Description | Example |
|-------|-------------|---------|
| **Report Name** | Unique identifier | `Daily_AD_Uncorrelated` |
| **Description** | Purpose of report | `Daily check for orphaned AD accounts` |
| **Object Type** | What to search | Identities, Accounts, Access |
| **Query** | Search criteria | `uncorrelated:true` |

### **Optional Fields**

| Field | Description |
|-------|-------------|
| **Columns** | Which fields to show |
| **Sort Order** | How to order results |
| **Format** | CSV, Excel, PDF |
| **Schedule** | When to run automatically |
| **Recipients** | Who receives results |

---

## Choosing Report Columns

Select which information appears in your report.

### **Identity Report Columns**

**Essential:**
- Name
- Email
- Department
- Status (cloudLifecycleState)
- Manager

**Optional:**
- Location
- Start Date
- Employee Type
- Job Title
- Phone Number

### **Account Report Columns**

**Essential:**
- Account Name (nativeIdentity)
- Source Name
- Identity Name
- Status (Enabled/Disabled)
- Uncorrelated Status

**Optional:**
- Created Date
- Last Modified
- Entitlement Count
- Description

### **Access Report Columns**

**Essential:**
- Access Name
- Type (Role, Access Profile, Entitlement)
- Source
- Owner
- Requestable

**Optional:**
- Description
- Risk Level
- Created Date
- Modification Date

---

## Report Naming Conventions

Use clear, consistent names for easy identification.

### **Recommended Format**
```
[Frequency]_[ObjectType]_[Description]
```

### **Good Examples**
```
Daily_Accounts_Uncorrelated_AD
Weekly_Identities_New_Hires
Monthly_Access_High_Risk_Review
Quarterly_Identities_Contractors
```

### **Poor Examples**

❌ Avoid vague names:
```
Report1
MyReport
Test
AccountCheck
```

---

## Common Report Templates

### **Template 1: Daily Uncorrelated Accounts**

**Purpose:** Find orphaned accounts for cleanup
```
Report Name: Daily_Accounts_Uncorrelated_All_Sources
Description: All uncorrelated accounts across all sources
Query: uncorrelated:true
Columns:
  - Account Name
  - Source
  - Created Date
  - Native Identity
Format: CSV
```

### **Template 2: Weekly Active User Count**

**Purpose:** Track active user population
```
Report Name: Weekly_Identities_Active_Count
Description: Count of active identities by department
Query: cloudLifecycleState:active
Columns:
  - Name
  - Email
  - Department
  - Manager
Format: Excel
```

### **Template 3: Monthly Terminated Users**

**Purpose:** Audit terminated user access
```
Report Name: Monthly_Identities_Inactive_Review
Description: Inactive users for access review
Query: cloudLifecycleState:inactive
Columns:
  - Name
  - Email
  - Department
  - Status
  - Last Modified
Format: Excel
```

### **Template 4: High-Risk Access Review**

**Purpose:** Compliance and security audit
```
Report Name: Monthly_Access_High_Risk
Description: High-risk access for monthly review
Query: type:ENTITLEMENT AND privileged:true
Columns:
  - Name
  - Type
  - Source
  - Owner
  - Risk Level
Format: PDF
```

---

## Managing Saved Reports

### **Viewing Your Reports**

**Location:** Admin → Reports or Search → Saved Searches

**What you'll see:**
- Report name
- Description
- Created date
- Last run date
- Schedule status

### **Running a Report**

**Manual execution:**
1. Navigate to saved reports
2. Find your report
3. Click **Run Now**
4. Wait for completion
5. Download results

### **Editing a Report**

**To modify:**
1. Find report in list
2. Click **Edit** or report name
3. Update query, columns, or settings
4. Click **Save**

### **Deleting a Report**

**To remove:**
1. Find report in list
2. Click **Delete** or trash icon
3. Confirm deletion

**Warning:** Deletion is permanent!

---

## Report Output Formats

### **CSV Format**

**Best for:**
- Excel analysis
- Data imports
- Quick reviews

**Pros:**
- Opens in Excel
- Easy to manipulate
- Small file size

**Cons:**
- No formatting
- No charts

### **Excel Format**

**Best for:**
- Formatted reports
- Executive summaries
- Charts and graphs

**Pros:**
- Professional appearance
- Can add formatting
- Supports formulas

**Cons:**
- Larger file size

### **PDF Format**

**Best for:**
- Audit documentation
- Official reports
- Read-only distribution

**Pros:**
- Cannot be edited
- Consistent formatting
- Professional

**Cons:**
- Cannot manipulate data
- Larger file size

---

## Report Best Practices

### **Organization**

✅ **Do:**
- Use consistent naming
- Add clear descriptions
- Group related reports
- Document report purpose
- Review and update regularly

❌ **Don't:**
- Create duplicate reports
- Use vague names
- Leave descriptions empty
- Create "test" reports in production

### **Performance**

✅ **Do:**
- Keep queries focused
- Limit columns to essentials
- Use specific date ranges
- Test before scheduling

❌ **Don't:**
- Create overly broad queries
- Include unnecessary columns
- Run during peak hours
- Schedule too frequently

### **Maintenance**

**Monthly tasks:**
- Review report usage
- Delete unused reports
- Update outdated queries
- Verify recipients list

**Quarterly tasks:**
- Audit all reports
- Consolidate duplicates
- Update documentation
- Review performance

---

## Common Report Scenarios

### **Scenario 1: Compliance Audit**

**Need:** Monthly report of all privileged access

**Solution:**
```
Report Name: Monthly_Compliance_Privileged_Access
Query: type:ENTITLEMENT AND privileged:true
Columns: Name, Source, Owner, Users_with_Access
Format: Excel
Schedule: 1st of each month
Recipients: compliance@company.com, auditors@company.com
```

### **Scenario 2: Daily Operations**

**Need:** Daily list of accounts needing attention

**Solution:**
```
Report Name: Daily_Operations_Uncorrelated_Accounts
Query: uncorrelated:true
Columns: Account_Name, Source, Created_Date
Format: CSV
Schedule: Daily at 6:00 AM
Recipients: iam-ops@company.com
```

### **Scenario 3: HR Coordination**

**Need:** Weekly new hire report

**Solution:**
```
Report Name: Weekly_NewHires_Report
Query: cloudLifecycleState:active AND created:[now-7d TO now]
Columns: Name, Email, Department, Start_Date, Manager
Format: Excel
Schedule: Every Monday 8:00 AM
Recipients: hr@company.com, it-helpdesk@company.com
```

---

## Sharing Reports

### **Share with Team**

**Option 1: Email Distribution**
- Add recipients to report
- Automatic email on run
- Includes download link

**Option 2: Shared Access**
- Grant view/edit permissions
- Others can run report
- Collaborative management

**Option 3: Export and Send**
- Run report manually
- Download file
- Email as attachment

---

## Troubleshooting Reports

### **Problem: No Results**

**Check:**
- Query syntax is correct
- Data exists for criteria
- Filters are appropriate
- Date ranges are valid

### **Problem: Too Many Results**

**Solution:**
- Add more specific filters
- Narrow date range
- Increase specificity
- Break into multiple reports

### **Problem: Wrong Data**

**Check:**
- Column selection
- Sort order
- Filter settings
- Query accuracy

### **Problem: Report Times Out**

**Solution:**
- Narrow query scope
- Reduce result set
- Schedule during off-hours
- Contact admin for assistance

---

## Quick Reference

### **Report Creation Steps**

1. Build and test query
2. Apply filters
3. Click "Save Search"
4. Name report
5. Add description
6. Select columns
7. Choose format
8. Save

### **Report Elements**

| Element | Required | Purpose |
|---------|----------|---------|
| Name | Yes | Identifies report |
| Description | No | Explains purpose |
| Query | Yes | Defines search |
| Columns | Yes | Data to show |
| Format | No | Output type |
| Schedule | No | Automation |

---

## Practice Exercises

**Exercise 9:** Create a report to find all inactive identities in the IT department

<details>
<summary>Answer</summary>
```
Report Name: IT_Inactive_Identities
Description: Inactive IT department employees
Query: cloudLifecycleState:inactive AND department:IT
Columns: Name, Email, Department, Last_Modified
Format: CSV
```
</details>

---

**Exercise 10:** What columns should you include in an uncorrelated accounts report?

<details>
<summary>Answer</summary>

Essential columns:
- Account Name (nativeIdentity)
- Source Name
- Created Date
- Status
- Native Identity

Optional:
- Last Modified
- Description
</details>

---

# 1.1.4 Scheduling and Managing Reports

**Learning Objective:** Automate reports and manage report distribution.

---

## What is Report Scheduling?

Scheduling runs reports automatically at specified times and delivers results to recipients.

**Benefits:**
- No manual execution needed
- Consistent timing
- Automatic distribution
- Reduced workload
- Timely information

---

## Scheduling Frequencies

### **Available Schedule Options**

| Frequency | When to Use | Example |
|-----------|-------------|---------|
| **On-Demand** | Manual only | Investigation reports |
| **Daily** | Daily operations | Uncorrelated account checks |
| **Weekly** | Regular reviews | Active user counts |
| **Monthly** | Compliance | Access certifications prep |
| **Custom** | Specific timing | First Monday of month |

---

## Creating a Schedule

### **Step-by-Step: Schedule a Daily Report**

**Step 1: Open Report for Editing**

1. Navigate to saved reports
2. Find your report
3. Click **Edit** or report name

**Step 2: Access Schedule Settings**

Look for **Schedule** section in report settings

**Step 3: Configure Schedule**
```
Schedule Type: Daily
Time: 06:00 AM
Time Zone: America/New_York
Start Date: 2024-02-13
End Date: (Leave blank for indefinite)
Enabled: Yes
```

**Step 4: Configure Delivery**
```
Delivery Method: Email
Recipients: iam-team@company.com
Subject: Daily Uncorrelated Accounts Report - {date}
Include Link: Yes
Attach File: Yes
Format: CSV
```

**Step 5: Save Schedule**

Click **Save** - Report is now scheduled!

---

## Schedule Configuration Options

### **Time Settings**

**Time:**
- Use 24-hour format (00:00 - 23:59)
- Example: `06:00` for 6:00 AM

**Time Zone:**
```
America/New_York
America/Los_Angeles
America/Chicago
Europe/London
Asia/Tokyo
```

**Important:** Choose correct time zone for your location

### **Date Range**

**Start Date:** When schedule begins
**End Date:** When schedule stops (optional)

**Examples:**
```
Start: 2024-02-13
End: (blank) → Runs indefinitely

Start: 2024-02-01
End: 2024-03-31 → Runs for 2 months only
```

---

## Delivery Methods

### **Email Delivery**

**Configuration:**
```
Method: Email
Recipients: team@company.com, manager@company.com
Subject: Daily Report - {date}
Body: Please review attached report for {date}
Attachment: Yes
Format: CSV
```

**Subject Line Variables:**
- `{date}` - Current date
- `{reportName}` - Report name
- `{time}` - Execution time

**Best Practices:**
- Use distribution lists
- Include date in subject
- Keep recipients list current
- Test email delivery

---

## Weekly Schedule Examples

### **Example 1: Every Monday**
```
Schedule Type: Weekly
Day: Monday
Time: 08:00
Time Zone: America/New_York
```

**Use case:** Weekly active user report for IT management

### **Example 2: Every Friday**
```
Schedule Type: Weekly
Day: Friday
Time: 17:00
Time Zone: America/New_York
```

**Use case:** End-of-week contractor access review

---

## Monthly Schedule Examples

### **Example 1: First of Month**
```
Schedule Type: Monthly
Day of Month: 1
Time: 07:00
Time Zone: America/New_York
```

**Use case:** Monthly compliance report

### **Example 2: Last Day of Month**
```
Schedule Type: Monthly
Day of Month: Last Day
Time: 18:00
Time Zone: America/New_York
```

**Use case:** Month-end access certification prep

---

## Managing Scheduled Reports

### **Viewing Schedule Status**

**Report dashboard shows:**
- Report name
- Schedule frequency
- Last run time
- Next run time
- Status (Active, Paused, Failed)
- Last result count

### **Execution Status Types**

| Status | Meaning | Action |
|--------|---------|--------|
| ✅ **Active** | Running normally | None needed |
| ⏸️ **Paused** | Schedule disabled | Resume if needed |
| ❌ **Failed** | Execution error | Check logs |
| ⏰ **Scheduled** | Waiting for next run | None needed |
| ⏳ **Running** | Currently executing | Wait |

---

## Monitoring Report Execution

### **Execution History**

**Access history:**
1. Navigate to report
2. Click **Execution History** or **History**
3. View past runs

**History shows:**
```
Date: 2024-02-12 06:00:00
Status: Success
Duration: 42 seconds
Results: 23 rows
File Size: 15 KB
Delivered To: iam-team@company.com
```

### **Reviewing Execution Logs**

**What logs contain:**
- Execution start time
- Query executed
- Results count
- Delivery status
- Any errors
- Completion time

---

## Pausing and Resuming Schedules

### **When to Pause**

**Common reasons:**
- Temporary project pause
- Holiday period
- Source system maintenance
- Report updates needed

### **How to Pause**

1. Navigate to scheduled report
2. Click **Pause** or toggle **Enabled** to OFF
3. Confirm pause

**Result:** Schedule stops, no emails sent

### **How to Resume**

1. Navigate to paused report
2. Click **Resume** or toggle **Enabled** to ON
3. Confirm resume

**Result:** Schedule resumes at next scheduled time

---

## Updating Scheduled Reports

### **What You Can Update**

**Schedule settings:**
- Frequency
- Time
- Time zone
- Recipients

**Report settings:**
- Query
- Columns
- Format
- Description

### **How to Update**

1. Navigate to report
2. Click **Edit**
3. Modify settings
4. Click **Save**

**Important:** Changes apply to next execution

---

## Report Retention

### **How Long Reports are Kept**

**Execution results:**
- Kept for 30-90 days (varies by tenant)
- Older results automatically deleted
- Download important results for archive

**Report definitions:**
- Kept indefinitely until manually deleted
- No automatic removal
- Regular cleanup recommended

---

## Best Practices for Scheduling

### **Timing Best Practices**

✅ **Do:**
- Schedule during off-peak hours (2 AM - 6 AM)
- Stagger multiple reports (avoid same time)
- Consider recipient time zones
- Allow time for data updates

❌ **Don't:**
- Schedule during business hours (9 AM - 5 PM)
- Run all reports at same time
- Schedule before data refresh completes

**Example good timing:**
```
Report A: 02:00 AM
Report B: 03:00 AM
Report C: 04:00 AM
```

### **Recipient Best Practices**

✅ **Do:**
- Use distribution lists
- Keep recipient lists current
- Test delivery before scheduling
- Include backup recipients

❌ **Don't:**
- Use individual emails only
- Forget to update when people leave
- Include unnecessary recipients
- Send to external emails (security risk)

### **Maintenance Best Practices**

**Weekly:**
- Check failed executions
- Verify deliveries

**Monthly:**
- Review report usage
- Update recipient lists
- Check execution times

**Quarterly:**
- Audit all scheduled reports
- Remove obsolete reports
- Optimize slow reports

---

## Common Scheduling Patterns

### **Pattern 1: Daily Operational Report**
```
Name: Daily_Uncorrelated_Accounts
Schedule: Daily at 06:00 AM
Recipients: iam-ops@company.com
Format: CSV
Retention: 30 days
```

**Purpose:** Daily operational checks

### **Pattern 2: Weekly Management Report**
```
Name: Weekly_Active_User_Summary
Schedule: Every Monday at 08:00 AM
Recipients: it-management@company.com
Format: Excel
Retention: 90 days
```

**Purpose:** Weekly status to management

### **Pattern 3: Monthly Compliance Report**
```
Name: Monthly_Privileged_Access_Review
Schedule: 1st day of month at 07:00 AM
Recipients: compliance@company.com, audit@company.com
Format: PDF
Retention: 7 years
```

**Purpose:** Regulatory compliance

---

## Troubleshooting Scheduled Reports

### **Problem: Report Not Running**

**Check:**
1. Schedule is enabled (not paused)
2. Start date has passed
3. End date hasn't passed
4. Time zone is correct

### **Problem: Not Receiving Emails**

**Check:**
1. Email address is correct
2. Check spam folder
3. Email server not blocking
4. Recipients list includes you
5. Delivery method is set to Email

### **Problem: Report Fails**

**Common causes:**
- Query syntax error
- Source unavailable
- Timeout (too many results)
- Permission issues

**Solution:**
1. Check execution logs
2. Test query manually
3. Narrow query scope
4. Contact support if needed

### **Problem: Wrong Results**

**Check:**
1. Query is correct
2. Filters are appropriate
3. Columns match expectations
4. Data has updated

---

## Report Performance Optimization

### **For Large Reports**

**Tips:**
- Schedule during off-hours (2-6 AM)
- Narrow query scope
- Limit columns
- Use specific date ranges

**Example - Optimized:**
```
Query: uncorrelated:true AND source.name:"Active Directory" AND created:[now-7d TO now]
```

Instead of:
```
Query: uncorrelated:true
```

### **For Multiple Reports**

**Stagger execution:**
```
Report 1: 02:00 AM
Report 2: 02:30 AM
Report 3: 03:00 AM
Report 4: 03:30 AM
```

**Benefits:**
- Reduces system load
- Prevents timeouts
- Ensures completion

---

## Advanced Scheduling

### **Custom Cron Schedules**

For very specific timing needs, some systems support cron expressions.

**Example: First Monday of Each Month**
```
Cron: 0 9 1-7 * 1
```

**Example: Every Weekday at 6 AM**
```
Cron: 0 6 * * 1-5
```

**Note:** Check if your ISC tenant supports custom cron schedules

---

## Report Lifecycle Management

### **Creation Phase**

1. Identify need
2. Build query
3. Test manually
4. Create report
5. Schedule delivery

### **Active Phase**

1. Monitor executions
2. Review usage
3. Update as needed
4. Respond to failures

### **Retirement Phase**

1. Identify unused reports
2. Confirm no longer needed
3. Archive final results
4. Delete report

---

## Quick Reference

### **Schedule Setup Steps**

1. Edit saved report
2. Enable scheduling
3. Set frequency
4. Set time and time zone
5. Add recipients
6. Choose format
7. Save

### **Common Schedules**

| Frequency | Example | Use Case |
|-----------|---------|----------|
| Daily | 6:00 AM | Operational checks |
| Weekly | Monday 8 AM | Status reports |
| Monthly | 1st at 7 AM | Compliance |
| Custom | Specific needs | Special requirements |

### **Troubleshooting Checklist**

- [ ] Schedule enabled?
- [ ] Correct time zone?
- [ ] Recipients correct?
- [ ] Query works manually?
- [ ] Email not in spam?
- [ ] Execution logs checked?

---

## Practice Exercises

**Exercise 11:** Schedule a report to run every Monday at 8 AM Eastern time

<details>
<summary>Answer</summary>
```
Schedule Type: Weekly
Day: Monday
Time: 08:00
Time Zone: America/New_York
Enabled: Yes
```
</details>

---

**Exercise 12:** Your daily report failed. What should you check first?

<details>
<summary>Answer</summary>

1. Check execution logs for error message
2. Test the query manually to see if it works
3. Verify source systems are available
4. Check if query returns too many results (timeout)
</details>

---

**Exercise 13:** You need a report on the 1st of every month. How do you schedule it?

<details>
<summary>Answer</summary>
```
Schedule Type: Monthly
Day of Month: 1
Time: 07:00 (choose appropriate time)
Time Zone: (your time zone)
Enabled: Yes
```
</details>

---

## Section Summary

You've learned how to:

✅ **Search effectively** - Use operators, wildcards, and filters  
✅ **Interpret results** - Read and understand search output  
✅ **Create reports** - Save searches for repeated use  
✅ **Schedule automation** - Set up automatic report delivery  

**Next Steps:**
- Practice searches in your test tenant
- Create your first saved report
- Schedule a daily operational report
- Review execution history regularly

---

## Additional Resources

📚 **Official Documentation:**
- [SailPoint Search Guide](https://documentation.sailpoint.com/saas/help/search/index.html)
- [Creating Reports](https://documentation.sailpoint.com/saas/help/search/saved-search.html)
- [Report Scheduling](https://documentation.sailpoint.com/saas/help/search/scheduled-search.html)

💡 **Pro Tips:**
- Start with simple searches and build complexity
- Test reports manually before scheduling
- Document report purposes
- Review and clean up reports quarterly

---

**End of Section 1.1: Search and Reporting**

**Next Section:** [1.2 API and Integration →](#12-api-and-integration)
