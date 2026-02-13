# Section 1.1: Search and Reporting

**Mastering search capabilities and report generation in SailPoint Identity Security Cloud**

---

## Table of Contents

- [1.1.1 Valid Search Query Syntax and Operators](#111-valid-search-query-syntax-and-operators)
- [1.1.2 Understanding Search Results and Filters](#112-understanding-search-results-and-filters)
- [1.1.3 Creating Custom Search Reports](#113-creating-custom-search-reports)
- [1.1.4 Scheduling and Managing Reports](#114-scheduling-and-managing-reports)

---

## 1.1.1 Valid Search Query Syntax and Operators

### **What is Search in ISC?**

Search is the primary mechanism for finding identities, accounts, entitlements, access profiles, roles, and other objects within Identity Security Cloud. Understanding search syntax is critical for day-to-day administration tasks such as:
- Finding specific identities or accounts
- Troubleshooting provisioning issues
- Generating compliance reports
- Identifying orphaned or uncorrelated accounts
- Auditing access assignments

### **Search Syntax Components**

SailPoint ISC uses **Elasticsearch-based query syntax**. Understanding the operators and syntax is essential for effective searching.

#### **Basic Search Operators**

| Operator | Purpose | Example | Description |
|----------|---------|---------|-------------|
| `:` | Field-specific search | `name:john` | Search for "john" in the name field |
| `*` | Wildcard (matches any characters) | `name:john*` | Matches john, johnny, johnathan |
| `AND` | Both conditions must be true | `name:john AND department:IT` | Must match both conditions |
| `OR` | Either condition can be true | `department:IT OR department:HR` | Matches if either is true |
| `NOT` | Excludes results | `NOT department:IT` | Excludes IT department |
| `()` | Groups conditions | `(department:IT OR department:HR) AND location:NYC` | Groups OR conditions |
| `""` | Exact phrase match | `name:"John Smith"` | Exact match for full phrase |

#### **Common Searchable Fields**

Understanding which fields are searchable is crucial for building effective queries.

**For Identities:**

| Field | Description | Example |
|-------|-------------|---------|
| `name` | Identity name/username | `name:john.smith` |
| `email` | Email address | `email:*@company.com` |
| `displayName` | Display name | `displayName:"John Smith"` |
| `department` | Department attribute | `department:IT` |
| `manager` | Manager attribute | `manager:"Jane Doe"` |
| `cloudLifecycleState` | Lifecycle state | `cloudLifecycleState:active` |
| `source.name` | Authoritative source | `source.name:"HR System"` |
| `identityProfile.name` | Identity profile name | `identityProfile.name:Employees` |
| `attributes.*` | Any custom attribute | `attributes.employeeType:Contractor` |

**For Accounts:**

| Field | Description | Example |
|-------|-------------|---------|
| `name` | Account name | `name:jsmith` |
| `nativeIdentity` | Account ID in source | `nativeIdentity:"CN=John Smith,OU=Users"` |
| `sourceId` | Source system ID | `sourceId:2c9180887...` |
| `source.name` | Source system name | `source.name:"Active Directory"` |
| `uncorrelated` | Correlation status | `uncorrelated:true` |
| `disabled` | Account status | `disabled:false` |
| `locked` | Lock status | `locked:true` |
| `identity.name` | Associated identity | `identity.name:"John Smith"` |

**For Access (Roles, Access Profiles, Entitlements):**

| Field | Description | Example |
|-------|-------------|---------|
| `name` | Access name | `name:"VPN Access"` |
| `type` | Object type | `type:ACCESS_PROFILE` |
| `source.name` | Source system | `source.name:Salesforce` |
| `owner.name` | Access owner | `owner.name:"IT Manager"` |
| `requestable` | Can be requested | `requestable:true` |
| `privileged` | Privileged access flag | `privileged:true` |

### **Example Search Queries**

Let's explore practical examples organized by common use cases:

#### **Finding Identities**

**Example 1: Find all identities in IT department**
```
department:IT
```

**Example 2: Find identities with names starting with "John"**
```
name:john*
```

**Example 3: Find active identities in IT or HR departments**
```
cloudLifecycleState:active AND (department:IT OR department:HR)
```

**Example 4: Find identities without a manager assigned**
```
NOT _exists_:manager
```

**Example 5: Find contractors in New York**
```
attributes.employeeType:Contractor AND attributes.location:"New York"
```

**Example 6: Find identities from specific source**
```
source.name:"Workday"
```

#### **Finding Accounts**

**Example 7: Find uncorrelated accounts in Active Directory**
```
uncorrelated:true AND source.name:"Active Directory"
```

**Example 8: Find disabled accounts**
```
disabled:true
```

**Example 9: Find accounts for a specific identity**
```
identity.name:"John Smith"
```

**Example 10: Find locked accounts that are not disabled**
```
locked:true AND disabled:false
```

#### **Finding Access**

**Example 11: Find all entitlements from Salesforce**
```
type:ENTITLEMENT AND source.name:Salesforce
```

**Example 12: Find requestable access profiles**
```
type:ACCESS_PROFILE AND requestable:true
```

**Example 13: Find privileged entitlements**
```
type:ENTITLEMENT AND privileged:true
```

**Example 14: Find roles owned by IT Manager**
```
type:ROLE AND owner.name:"IT Manager"
```

#### **Complex Queries**

**Example 15: Active IT employees in New York without manager**
```
cloudLifecycleState:active AND department:IT AND attributes.location:"New York" AND NOT _exists_:manager
```

**Example 16: Contractors with expiring access**
```
attributes.employeeType:Contractor AND attributes.endDate:[now TO now+30d]
```

**Example 17: Accounts created in the last 7 days**
```
created:[now-7d TO now]
```

**Example 18: Identities with more than 5 accounts**
```
accountCount:>5
```

### **Special Operators**

#### **Existence Operators**

| Operator | Purpose | Example |
|----------|---------|---------|
| `_exists_:field` | Field has a value | `_exists_:manager` |
| `NOT _exists_:field` | Field is null/empty | `NOT _exists_:email` |

**Use Cases:**
```
# Find identities missing email address
NOT _exists_:email

# Find identities with a manager assigned
_exists_:manager

# Find accounts without an associated identity (uncorrelated)
NOT _exists_:identity
```

#### **Range Operators**

| Operator | Purpose | Example |
|----------|---------|---------|
| `field:[min TO max]` | Range query (inclusive) | `created:[2024-01-01 TO 2024-12-31]` |
| `field:>value` | Greater than | `accountCount:>5` |
| `field:>=value` | Greater than or equal | `accountCount:>=5` |
| `field:<value` | Less than | `accountCount:<2` |
| `field:<=value` | Less than or equal | `accountCount:<=2` |

**Date Range Examples:**
```
# Created between specific dates
created:[2024-01-01 TO 2024-12-31]

# Modified in the last 24 hours
modified:[now-1d TO now]

# Created in the last 7 days
created:[now-7d TO now]

# Future dates (e.g., contract end date)
attributes.endDate:[now TO now+30d]
```

**Numeric Range Examples:**
```
# Identities with more than 5 accounts
accountCount:>5

# Identities with 1-3 accounts
accountCount:[1 TO 3]

# Identities with no accounts
accountCount:0
```

#### **Wildcard Operators**

| Wildcard | Purpose | Example | Matches |
|----------|---------|---------|---------|
| `*` | Multiple characters | `name:john*` | john, johnny, johnathan |
| `?` | Single character | `name:sm?th` | smith, smyth |

**Wildcard Best Practices:**

✅ **Good - Trailing wildcard:**
```
name:john*        # Fast - uses index
email:*@company.com  # Fast - uses index
```

❌ **Poor - Leading wildcard:**
```
name:*smith       # Slow - cannot use index efficiently
```

⚠️ **Use with caution - Middle wildcard:**
```
name:jo*smith     # Moderate speed - partial index use
```

### **Advanced Query Techniques**

#### **Boolean Logic with Parentheses**

Group conditions to control evaluation order:

**Example: Find active employees in IT or HR located in NYC or Boston**
```
cloudLifecycleState:active AND (department:IT OR department:HR) AND (location:NYC OR location:Boston)
```

**Example: Complex contractor query**
```
attributes.employeeType:Contractor AND (attributes.vendor:"Acme Corp" OR attributes.vendor:"TechStaff Inc") AND NOT attributes.status:Terminated
```

#### **Nested Field Searches**

Search within nested objects using dot notation:
```
# Search within source object
source.name:"Active Directory"
source.id:2c9180887f4e4b52017f504f30d80456

# Search within identity profile
identityProfile.name:Employees
identityProfile.id:2c9180857f4e4b52017f504f30d80123

# Search within attributes
attributes.department:IT
attributes.location:"New York"
attributes.customField1:value
```

#### **Case Sensitivity**

⚠️ **Important:** Field names are case-sensitive, but field values may vary:
```
# Correct - field name lowercase
name:john

# Incorrect - field name capitalized
Name:john

# Value case sensitivity varies by field
# Some fields are case-sensitive:
cloudLifecycleState:Active   # Correct
cloudLifecycleState:active   # Incorrect

# Some fields are case-insensitive:
name:john                    # Finds John, JOHN, john
```

**Pro Tip:** Use the `.exact` suffix for exact case-sensitive matches:
```
attributes.department.exact:IT     # Case-sensitive
attributes.department:it           # Case-insensitive
```

### **Performance Optimization**

#### **Fast Searches**

✅ **These queries are optimized and fast:**

1. **Indexed field searches**
```
   name:john
   email:john.smith@company.com
   id:2c9180887f4e4b52017f504f30d80234
```

2. **Specific field queries**
```
   department:IT AND cloudLifecycleState:active
```

3. **Small result sets**
```
   name:"John Smith"  # Returns 1 result
```

4. **Trailing wildcards on indexed fields**
```
   name:john*
```

#### **Slow Searches**

❌ **These queries are slow and should be avoided:**

1. **Leading wildcards**
```
   name:*smith        # Very slow
```

2. **Global searches without field specification**
```
   john               # Searches all fields
```

3. **Searches returning >10,000 results**
```
   cloudLifecycleState:active  # May return 50,000+ results
```

4. **Complex nested queries with many OR conditions**
```
   (dept:IT OR dept:HR OR dept:Finance OR dept:Sales OR dept:Marketing) AND (loc:NYC OR loc:LA OR loc:Chicago OR loc:Boston)
```

#### **Optimization Tips**

💡 **Best Practices for Query Performance:**

1. **Be Specific**
```
   # Instead of:
   *smith
   
   # Use:
   name:smith* OR displayName:smith*
```

2. **Limit Result Sets**
```
   # Instead of:
   cloudLifecycleState:active
   
   # Use:
   cloudLifecycleState:active AND department:IT AND created:[now-30d TO now]
```

3. **Use Indexed Fields First**
```
   # Instead of:
   attributes.customField:value AND name:john
   
   # Use:
   name:john AND attributes.customField:value
```

4. **Combine Filters**
```
   # Instead of multiple separate queries, combine:
   department:IT AND cloudLifecycleState:active AND location:NYC
```

### **Common Pitfalls and How to Avoid Them**

#### **Pitfall 1: Reserved Characters**

❌ **Problem:**
```
name:john@company.com      # @ is interpreted as operator
```

✅ **Solution:**
```
name:"john@company.com"    # Use quotes for special characters
email:john\@company.com    # Or escape with backslash
```

**Reserved characters that need escaping or quotes:**
```
: * ( ) " \ + - = & | > < ! { } [ ] ^ ~ ? / @
```

#### **Pitfall 2: Spaces in Values**

❌ **Problem:**
```
name:John Smith            # Interpreted as name:John AND Smith (global)
```

✅ **Solution:**
```
name:"John Smith"          # Use quotes for multi-word values
```

#### **Pitfall 3: Case Sensitivity Confusion**

❌ **Problem:**
```
Name:john                  # Field name wrong case
```

✅ **Solution:**
```
name:john                  # Use correct field name case
```

#### **Pitfall 4: Incorrect Boolean Logic**

❌ **Problem:**
```
department:IT OR department:HR AND location:NYC
# This is interpreted as: department:IT OR (department:HR AND location:NYC)
```

✅ **Solution:**
```
(department:IT OR department:HR) AND location:NYC
# Explicit grouping with parentheses
```

#### **Pitfall 5: Date Format Issues**

❌ **Problem:**
```
created:02/12/2024         # Incorrect date format
```

✅ **Solution:**
```
created:2024-02-12         # Use ISO format YYYY-MM-DD
created:[2024-02-01 TO 2024-02-12]  # Date ranges
created:[now-7d TO now]    # Relative dates
```

### **Search Query Validation Checklist**

Before executing a search query, verify:

- [ ] Field names are lowercase and correctly spelled
- [ ] Multi-word values are wrapped in quotes
- [ ] Special characters are escaped or quoted
- [ ] Boolean logic uses proper parentheses
- [ ] Wildcards are positioned for performance (trailing preferred)
- [ ] Date formats use ISO standard (YYYY-MM-DD)
- [ ] Nested fields use dot notation correctly
- [ ] Query is specific enough to avoid timeout

### **Practical Exercise**

**Challenge:** Write queries for these scenarios:

1. Find all active identities in IT department hired in the last 30 days
2. Find uncorrelated accounts in Active Directory that were created more than 7 days ago
3. Find all contractors whose contracts end in the next 60 days
4. Find identities without an email address who are in active state
5. Find all privileged entitlements from Salesforce that are requestable

**Solutions:**
```
1. cloudLifecycleState:active AND department:IT AND created:[now-30d TO now]

2. uncorrelated:true AND source.name:"Active Directory" AND created:[* TO now-7d]

3. attributes.employeeType:Contractor AND attributes.endDate:[now TO now+60d]

4. cloudLifecycleState:active AND NOT _exists_:email

5. type:ENTITLEMENT AND source.name:Salesforce AND privileged:true AND requestable:true
```

### **Official Documentation**

📚 **SailPoint Resources:**
- [Search Documentation](https://documentation.sailpoint.com/saas/help/search/index.html)
- [Search Query Syntax](https://documentation.sailpoint.com/saas/help/search/searchable-fields.html)
- [Elasticsearch Query String Syntax](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-query-string-query.html)

### **Summary**

**Key Takeaways:**

✅ Search is fundamental to ISC administration
✅ Master basic operators: `:`, `*`, `AND`, `OR`, `NOT`, `()`
✅ Know commonly searchable fields for identities, accounts, and access
✅ Use specific field searches for better performance
✅ Avoid leading wildcards and overly broad queries
✅ Use quotes for multi-word values and special characters
✅ Understand case sensitivity rules
✅ Group conditions with parentheses for complex logic
✅ Test queries before using in automation or reports

---

**Next Topic:** [1.1.2 Understanding Search Results and Filters](#112-understanding-search-results-and-filters)
