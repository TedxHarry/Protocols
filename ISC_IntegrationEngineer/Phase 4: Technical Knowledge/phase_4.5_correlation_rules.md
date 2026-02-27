# Correlation Rules — Linking Accounts to Identities

Aggregation brings accounts into ISC. Transforms clean and normalize their attributes. Correlation rules answer the one question that determines whether all of that work means anything: which identity does this account belong to?

Correlation is the step where ISC takes an account record from Active Directory, Salesforce, GitHub, or any other system and says: "This account belongs to Sarah Chen." Without successful correlation, the account is ungoverned — it exists in ISC's awareness, but ISC has no idea whose it is, can't include it in access certifications, can't apply policies to it, and can't deprovision it when Sarah leaves.

Getting correlation right is the difference between an integration that governs access and an integration that just imports data.

---

## What Correlation Actually Does

During aggregation, after transforms run, ISC has a normalized account record: it knows the account's attribute values. The correlation rule takes those attribute values and searches ISC's identity cube for a matching identity.

A match is found → account is **correlated**: associated with that identity. The account appears on the identity's profile. Entitlements become visible in certifications. Provisioning and deprovisioning apply to this identity. Governance is active.

No match found → account is **uncorrelated**: orphaned. ISC logs it as an unmatched account. It generates governance alerts. It doesn't appear on any identity's profile. It is outside governance scope until someone manually links it or the data is fixed.

> 🎯 **KEY POINT:** Correlation is a governance on/off switch. A correlated account is governed. An uncorrelated account is not. Every uncorrelated human account is a security gap — access that ISC can see but cannot govern.

---

## Attribute-Based Correlation (Simple)

The most common correlation type. ISC compares one attribute on the account to one attribute on the identity.

**The pattern**: "Account attribute X should equal Identity attribute Y."

**Configuration**: In the ISC source configuration, the Correlation Rules section. Add a correlation filter:

| Account Attribute | Operator | Identity Attribute |
|---|---|---|
| `employeeId` | equals | `employeeId` |

If a Workday account has `employeeId = 48291` and there's an ISC identity with `employeeId = 48291`, they correlate.

**Multiple attribute correlation**: ISC supports multiple conditions in a single rule, either as AND conditions (all must match) or with fallback rules (try rule 1; if no match, try rule 2).

**Common correlation strategies by system type**:

| System Type | Primary Correlation Attribute | Notes |
|---|---|---|
| HR system (authoritative) | `employeeId` | Persistent, unique, doesn't change when names change |
| Active Directory | `employeeId` (stored in AD extension attribute) | Or by `samAccountName` pattern match if employeeId not in AD |
| Email systems (O365, Google) | `email` | Reliable for primary accounts; breaks for shared/functional mailboxes |
| SaaS apps | `email` or `employeeId` | Varies by system; check what the app reliably stores |
| Contractor systems | Vendor-assigned ID | May differ from employee ID convention — verify before assuming |

---

## Correlation Rule Priority and Fallback

ISC evaluates correlation rules in priority order. You can define multiple rules — if the first rule finds no match, ISC tries the next one.

**Why fallback rules matter**: GlobalTech has employees who joined before the employee ID standardization. Their AD accounts predate the `employeeId` extension attribute. Rule 1 (correlate by employeeId) finds no match for them. Rule 2 (correlate by email) succeeds for most of them. Rule 3 (correlate by `samAccountName` matching a pattern) catches the remaining few.

**Setting up fallback rules**:

Rule 1 (priority 10):
- Account: `employeeId` equals Identity: `employeeId`

Rule 2 (priority 20):
- Account: `mail` equals Identity: `email`

Rule 3 (priority 30):
- Account: `samAccountName` equals Identity: `uid` (if your UID convention matches AD naming)

ISC processes these in priority order (lower number = higher priority). Only accounts that didn't match any earlier rule are evaluated by later rules.

> ⚠️ **COMMON MISTAKE:** Overly permissive fallback rules. Rule: "if name matches, correlate." Names are not unique. Two people named "John Smith" get their accounts correlated to the same identity — or worse, to the wrong one. Use persistent, unique identifiers (employee ID, unique account ID). Email is acceptable but breaks on name changes. Name-based correlation is almost always wrong.

---

## Rule-Based Correlation (BeanShell)

When attribute matching isn't enough — when the logic requires transformation, conditional comparisons, or multi-step matching — you write a BeanShell correlation rule.

**Common scenarios requiring rule-based correlation**:

1. **Prefix/suffix handling**: Workday sends `GT-48291`, AD stores `48291`. You need to strip the prefix before comparing.

2. **Multi-field composite matching**: The system doesn't store employee ID, but the combination of first name + last name + hire date (within 7 days) is reliable enough for initial correlation.

3. **Format normalization needed before comparison**: Email addresses in different cases. Phone numbers in different formats.

4. **Legacy ID migration**: Old accounts have a different ID format than new accounts. The rule handles both formats.

**BeanShell correlation rule structure**:

```java
import sailpoint.object.*;
import sailpoint.api.*;

// 'account' is the account object being correlated
// Return a Filter object that ISC uses to search identities
// Or return null if this rule can't determine a correlation

String workerId = account.getAttribute("Worker_ID");

if (workerId == null || workerId.length() == 0) {
    return null;  // can't correlate without this attribute
}

// Strip the "GT-" prefix
if (workerId.startsWith("GT-")) {
    workerId = workerId.substring(3);
}

// Return a filter: find the identity where employeeId equals workerId
return Filter.eq("employeeId", workerId);
```

**What the rule returns**:
- A `Filter` object: ISC uses it to query the identity cube — correlates to whatever identity matches the filter
- `null`: no correlation — the account remains uncorrelated from this rule
- Throwing an exception: rule error — logged, account marked with error state

**Key BeanShell patterns for correlation**:

```java
// Null check before processing (always do this first)
String attrValue = account.getAttribute("attributeName");
if (attrValue == null) {
    return null;
}

// String manipulation before matching
String cleaned = attrValue.trim().toLowerCase();
return Filter.eq("email", cleaned);

// Multiple conditions (AND)
return Filter.and(
    Filter.eq("firstname", account.getAttribute("givenName")),
    Filter.eq("lastname", account.getAttribute("sn"))
);

// Ignore case comparison
return Filter.ignoreCase(Filter.eq("email", account.getAttribute("mail")));
```

> ⚠️ **COMMON MISTAKE:** No null check at the top of a correlation rule. If the attribute you're correlating on is null (missing from the account record), and the rule tries to call `.substring()` or `.toLowerCase()` on null, you get a NullPointerException. The rule errors out, the account gets an error state, and it looks like a connector problem rather than a rule bug. First line of every correlation rule: null check. Return null if the key attribute is missing.

---

## Manual Correlation Override

Some accounts legitimately can't be correlated by rule. Executive email accounts with custom formats. Shared service accounts with designated human owners. Accounts from acquired companies with incompatible ID schemes.

ISC supports manual correlation: in the Uncorrelated Accounts section of the admin console, an administrator can manually link an account to a specific identity. Manual correlations persist through aggregations — ISC remembers the manual link.

**When to use manual correlation**:
- Genuine exceptions where no rule can reliably match (less than 1% of accounts in a healthy implementation)
- Temporary bridges while data quality issues are fixed at the source
- Service accounts with designated owners (correlate to the owner's identity)

**When NOT to rely on manual correlation**:
- As a substitute for fixing data quality problems in the source
- For large numbers of accounts (more than 1% manual correlation is a signal that your rules need work)
- As a permanent solution for a systemic mismatch (fix the root cause)

---

## Testing Correlation Rules Before Production

Testing correlation rules against sample data before running them in production is not optional. Correlation errors in production generate incident-level problems — accounts correlated to the wrong identity, mass uncorrelation of existing accounts, governance blind spots.

**Testing approach**:

**Step 1: Preview mode**
ISC supports a correlation preview — run the aggregation in report-only mode. ISC shows you which accounts would correlate to which identities, without making any changes. Review the preview output for:
- Any account correlating to an identity that isn't its obvious owner
- Multiple accounts correlating to the same identity (possible duplicate or multi-account scenario — verify intentional)
- High-value or admin accounts unexpectedly uncorrelated

**Step 2: Test against a small dataset in sandbox**
Configure the source in your ISC sandbox tenant. Run aggregation against a representative but small dataset (100–200 accounts). Manually verify the correlation results for accounts you know — pick 10 accounts where you know who they should belong to and verify ISC matched them correctly.

**Step 3: Test edge cases explicitly**
For each type of exception in your data (null employee ID, prefix format, legacy format, contractor naming), create test accounts that represent those cases and verify the fallback rules handle them correctly.

**Step 4: Full production dataset dry run**
Before enabling live governance, run the aggregation against the full production dataset in a non-production ISC tenant (or ISC sandbox refreshed with production data). Review the uncorrelated account count against your expected threshold. Investigate every category of uncorrelated account before enabling governance operations.

---

## Diagnosing Correlation Failures

When the uncorrelated account count is higher than expected, diagnose systematically:

**Step 1: Export the uncorrelated account list**
ISC admin console → source → uncorrelated accounts. Export to CSV. You now have a complete list of accounts that didn't correlate.

**Step 2: Categorize by type**
Group the uncorrelated accounts:
- Service accounts (naming pattern: `svc-*`, `app-*`, `admin-*`)
- Test accounts (naming pattern: `test-*`, accounts in test OUs)
- Contractors vs employees (different ID formats?)
- Legacy accounts from before a certain date?
- Accounts with null/missing correlation attribute?

**Step 3: For each category, identify root cause**
- Service accounts: create exclusion rule to remove from correlation scope
- Missing correlation attribute: data quality issue — fix at source or add null-handling rule
- Format mismatch: update the correlation rule or add a preprocessing transform
- Genuinely unmatched: these are the exceptions that need manual investigation

**Step 4: Fix at the source when possible**
Correlation rule workarounds for data quality problems treat symptoms. If 200 AD accounts are missing `employeeId` because IT never populated that extension attribute, the right fix is populating it in AD — not a progressively more complex correlation rule.

**Step 5: Document the residual exceptions**
After fixing data quality issues and updating rules, document the remaining uncorrelated accounts and why they're uncorrelated. "22 service accounts excluded from correlation per policy" and "3 executive accounts manually correlated" is an acceptable audit-ready state. "467 uncorrelated accounts, cause unknown" is not.

---

## Key Takeaways

- Correlation is the governance on/off switch — correlated accounts are governed, uncorrelated are not
- Use persistent unique identifiers (employee ID) not mutable attributes (email, name) as primary correlation keys
- Configure fallback rules to handle edge cases — multiple rules in priority order catch what the primary rule misses
- Every BeanShell correlation rule must null-check the correlation attribute on the first line — NullPointerExceptions are the most common rule failure mode
- Test correlation rules in preview mode and against known accounts before any production run
- Diagnose uncorrelated accounts systematically: categorize, find root causes, fix at source, document residuals

> 🔗 **CONNECTS TO:** You now understand the full technical foundation: VA deployment, connector selection, aggregation mechanics, data transforms, and correlation rules. Phase 5 moves to the governance layer built on top of this foundation — access profiles, roles, and lifecycle management — where the data you've been importing becomes governed access decisions.
