# The Transform Library — Data Manipulation Without Code

Every source system uses its own data formats, naming conventions, and attribute structures. ISC needs consistent, normalized data to make governance decisions. Transforms are the bridge between what source systems provide and what ISC needs.

The transform library is ISC's built-in collection of data manipulation operations. Before you write a line of BeanShell, you should know what transforms can do — because most of what integration engineers reach for custom rules to handle, the transform library handles natively with better performance, better error handling, and zero maintenance cost.

This document covers every transform category with real examples from common integration scenarios. Read it once, reference it every time you start a new attribute mapping.

---

## How Transforms Work in ISC

Transforms are configured as JSON in ISC and assigned to attribute mappings (source → ISC attribute). A transform takes an input (usually an account attribute value from the source system), applies one or more operations, and returns the transformed value to store in ISC.

Transforms chain: the output of one transform becomes the input of the next. A three-step transform is written by nesting transforms — the innermost runs first, its output goes to the next level out, and so on.

**Configuration location**: Identity Profile → Source Attributes section, or directly in the Account Schema attribute mapping.

---

## Category 1: String Operations

These handle the most common data manipulation tasks — modifying string values.

### Concat

Joins two or more values into a single string.

**Common use case**: Building a display name from first and last name.
```json
{
  "type": "concat",
  "attributes": {
    "values": [
      { "attributes": { "sourceName": "IdentityNow", "attributeName": "firstname" }, "type": "accountAttribute" },
      { "attributes": { "value": " " }, "type": "static" },
      { "attributes": { "sourceName": "IdentityNow", "attributeName": "lastname" }, "type": "accountAttribute" }
    ]
  }
}
```

**Other common uses**: Building an email from first/last name + domain. Building a username from employee ID + prefix. Combining department code with company prefix.

### Substring

Extracts a portion of a string.

**Common use case**: Workday provides employee IDs as `GT-48291`. ISC needs `48291`. Strip the first 3 characters.
```json
{
  "type": "substring",
  "attributes": {
    "begin": 3
  }
}
```

Specify `end` to extract from a fixed end position. Combine `begin` and `end` for a fixed window. Note: indices are 0-based and `end` is exclusive (like Python slicing).

### Split

Splits a string on a delimiter and returns one segment.

**Common use case**: Workday provides full org hierarchy as `Engineering.Software.Backend`. ISC needs only the top-level department (`Engineering`). Split on `.` and return index 0.
```json
{
  "type": "split",
  "attributes": {
    "delimiter": ".",
    "index": 0
  }
}
```

**Also useful for**: Extracting the domain from an email (`user@globaltech.com` → `globaltech.com` using `@` delimiter, index 1). Parsing compound codes into components.

### Trim

Removes leading and trailing whitespace from a string value.

**Common use case**: A legacy system stores names with trailing spaces (`John Smith   `). Trim before storing in ISC to prevent mismatches in searches and correlation.
```json
{
  "type": "trim"
}
```

Apply trim to any attribute from legacy systems — trailing space bugs are extremely common in data exported from old databases and flat files.

### Replace

Replaces occurrences of a pattern (literal string or regex) with another value.

**Common use case**: Source system uses spaces in department names but ISC needs underscores for downstream system compatibility (`Human Resources` → `Human_Resources`).
```json
{
  "type": "replace",
  "attributes": {
    "regex": " ",
    "replacement": "_"
  }
}
```

Use `literal` mode for simple string replacements, `regex` mode for pattern-based replacements. A common use: normalizing phone number formats by removing non-digit characters.

### Lower / Upper

Converts a string to all lowercase or all uppercase.

**Common use case**: Normalizing email addresses from a system that stores them in mixed case.
```json
{
  "type": "lower"
}
```

Apply `lower` to email and username attributes before storing. Many downstream systems (especially LDAP) treat attribute comparisons case-sensitively — normalized case prevents mismatches.

### LeftPad / RightPad

Pads a string to a fixed length by adding a character on the left or right.

**Common use case**: Source system exports employee IDs as variable-length numbers (`483`, `1293`, `48291`). A legacy target system needs 8-character zero-padded IDs (`00000483`, `00001293`, `00048291`).
```json
{
  "type": "leftPad",
  "attributes": {
    "length": 8,
    "padding": "0"
  }
}
```

---

## Category 2: Lookup and Mapping

These transforms map input values to output values — translating codes, normalizing terminology, or deriving values from a reference table.

### Lookup

Maps an input value to a mapped output value using a table you define.

**Common use case**: Workday uses `Worker_Type` values (`Regular`, `Contract`, `Temporary`) that don't match ISC's identity lifecycle states or the values expected by downstream systems.

```json
{
  "type": "lookup",
  "attributes": {
    "table": {
      "Regular": "Full-Time",
      "Contract": "Contractor",
      "Temporary": "Temp",
      "default": "Unknown"
    }
  }
}
```

The `default` key handles values not explicitly in the table. Without a default, unmatched values return null — which can cause provisioning rules to fail silently. Always define a default.

**Other uses**: Department code to department name. Job code to access tier. Country code to country name. Status codes to lifecycle state labels.

### Lookup from Source

A variant of Lookup that reads the mapping table from an external source (another account attribute, an identity attribute, or an ISC configuration object) rather than hardcoding values in the transform.

**When to use**: The mapping table has hundreds of entries that would make the JSON unwieldy, or the table changes frequently and you want to update it without modifying the transform configuration.

> 💡 **REMEMBER:** Hardcoded lookup tables in transform JSON must be updated whenever the source system adds new valid values. When your lookup table returns "Unknown" for new department codes, it's usually because new departments were added after the transform was last updated. Build a process for table maintenance — either schedule quarterly reviews or migrate to lookup-from-source for high-churn mappings.

---

## Category 3: Date and Time Operations

Date handling is consistently one of the messier parts of integration work. Systems store dates in dozens of formats. ISC needs consistent date values. These transforms handle the conversion.

### DateFormat

Converts a date from one format to another.

**Common use case**: SAP returns hire dates as `20240315` (YYYYMMDD). ISC needs `2024-03-15` (ISO 8601).

```json
{
  "type": "dateFormat",
  "attributes": {
    "inputFormat": "yyyyMMdd",
    "outputFormat": "yyyy-MM-dd"
  }
}
```

Format tokens follow Java's `SimpleDateFormat` patterns. Common patterns:
- `yyyy-MM-dd` → ISO 8601 date
- `MM/dd/yyyy` → US date format
- `dd/MM/yyyy` → EU date format
- `yyyy-MM-dd'T'HH:mm:ssZ` → ISO 8601 with time
- `yyyyMMdd` → Compact date (no separators)

**Time zone matters**: If the input date includes a time component, specify the source time zone in `inputTZ` and target time zone in `outputTZ`. Ignoring time zones in date transforms causes midnight-boundary bugs where dates shift by one day depending on where the server is running.

### DateMath

Performs date arithmetic — adds or subtracts a duration from a date.

**Common use case**: Access should expire 90 days after a contractor's hire date.

```json
{
  "type": "dateMath",
  "attributes": {
    "expression": "+90d",
    "roundUp": false
  }
}
```

Duration syntax: `+Nd` (add N days), `-Nh` (subtract N hours), `+Nw` (add N weeks), `/d` (round to start of day), `+1M` (add 1 month).

**Other uses**: Computing end dates from start dates + duration. Determining probationary period end. Setting password expiration windows.

---

## Category 4: Attribute Sources

These transforms retrieve values from ISC data — other attributes on the same identity, attributes from other accounts, or configuration values.

### AccountAttribute

Reads an attribute value from an account on a specific source.

**Common use case**: Your attribute transform needs the employee's `department` from Active Directory (not from the account being aggregated). AccountAttribute lets you reference any account attribute on any connected source.

```json
{
  "type": "accountAttribute",
  "attributes": {
    "sourceName": "Active Directory",
    "attributeName": "department"
  }
}
```

Use AccountAttribute when you need to derive a value from one source based on data from another. Common pattern: using the AD account's `distinguishedName` to derive the organizational unit, even during a Workday aggregation.

### IdentityAttribute

Reads an attribute from the identity itself (the ISC identity cube).

**Common use case**: During provisioning, you need to derive the new account username from the identity's existing `firstname` and `lastname` attributes already stored in ISC.

```json
{
  "type": "identityAttribute",
  "attributes": {
    "name": "lastname"
  }
}
```

### Static

Returns a fixed, hardcoded value — no input, always returns the same thing.

**Common use case**: Every user provisioned to a specific system should get a default profile type. Or every identity from a particular source should have `company` set to `GlobalTech`.

```json
{
  "type": "static",
  "attributes": {
    "value": "GlobalTech"
  }
}
```

Use Static for: default values, company identifiers, standard email domains, fixed profile assignments.

### Reference

References another named transform object in ISC (transforms can be saved as reusable objects in the transform catalog).

**Common use case**: Your organization has a standard "username generation" transform used across 15 different sources. Instead of duplicating the transform JSON everywhere, you save it as a named transform and reference it.

```json
{
  "type": "reference",
  "attributes": {
    "id": "username-generation-standard"
  }
}
```

> 🎯 **KEY POINT:** Use Reference transforms to avoid duplicating complex transform logic across sources. When the username format changes (it will), you update one named transform instead of 15 source configurations. This is the transform equivalent of writing a function instead of copying code.

---

## Category 5: Conditional and Null Handling

These handle the reality that data is messy — values are sometimes null, sometimes empty, sometimes in the wrong format.

### FirstValid

Returns the first non-null, non-empty value from a list of inputs.

**Common use case**: Some employees have a `preferredName` in the source system; others don't. Use preferred name if it exists; fall back to legal first name.

```json
{
  "type": "firstValid",
  "attributes": {
    "values": [
      { "type": "accountAttribute", "attributes": { "attributeName": "preferredFirstName" } },
      { "type": "accountAttribute", "attributes": { "attributeName": "legalFirstName" } }
    ]
  }
}
```

FirstValid is your null-handling workaround for most cases. Use it whenever an attribute might be null and you have a fallback value.

### Conditional

Evaluates a condition and returns one value if true, another if false.

**Common use case**: If `Worker_Type` is `Contract`, set `lifecycleState` to `Contractor`. Otherwise, set it to `Employee`.

```json
{
  "type": "conditional",
  "attributes": {
    "expression": "workerType.equals(\"Contract\")",
    "positiveCondition": "Contractor",
    "negativeCondition": "Employee"
  }
}
```

Conditional supports basic string comparison expressions. For complex conditional logic (multiple conditions, nested if-else), you've exceeded what transforms can do — that's a custom rule case.

---

## Category 6: Utility Transforms

### E164phone

Normalizes phone numbers to E.164 international format (`+14155551234`).

**Common use case**: Source systems store phone numbers in a dozen different formats (`(415) 555-1234`, `415.555.1234`, `4155551234`). ISC needs consistent format for downstream systems.

```json
{
  "type": "e164phone",
  "attributes": {
    "defaultRegion": "US"
  }
}
```

### UUID

Generates a random UUID value. Used when a system requires a unique identifier that doesn't come from the source.

### Base64Encode / Base64Decode

Encodes/decodes Base64 strings. Occasionally needed for specific system integrations that transmit credentials or binary data in Base64.

---

## Chaining Transforms: A Real Example

GlobalTech's Workday integration needs to generate a standard email address for provisioning to Office 365. The format is `firstInitial.lastname@globaltech.com`. The source provides `Legal_Name/First_Name` and `Legal_Name/Last_Name`.

Transform chain:
1. Get first name from Workday → Substring first character → lower case → store as `firstInitial`
2. Get last name from Workday → lower case → trim → store as `normalizedLast`
3. Concat: `firstInitial` + `.` + `normalizedLast` + `@globaltech.com`

This is achievable entirely with built-in transforms — no custom rule needed.

What the JSON looks like (simplified for readability):
```json
{
  "type": "concat",
  "values": [
    { "type": "lower", "input": { "type": "substring", "begin": 0, "end": 1, "input": { "type": "accountAttribute", "attributeName": "First_Name" } } },
    { "type": "static", "value": "." },
    { "type": "trim", "input": { "type": "lower", "input": { "type": "accountAttribute", "attributeName": "Last_Name" } } },
    { "type": "static", "value": "@globaltech.com" }
  ]
}
```

Four transforms, no code. The result is maintainable by any engineer who understands the transform library — not just whoever wrote the original BeanShell rule.

---

## When Transforms Are Not Enough

Write a custom BeanShell rule when:

1. **Conditional logic with multiple branches**: More than two conditions, or conditions that depend on multiple attributes simultaneously
2. **External system lookup**: The transform needs to query another system's API or database to derive a value (e.g., looking up a manager's system account ID by employee ID)
3. **Complex string parsing**: Regex with capture groups, multi-step parsing logic, format validation
4. **Arithmetic beyond dates**: Calculating values from numeric attributes (tenure = today - hire_date in years as a number, not a formatted date)
5. **Error handling with custom fallback logic**: More than FirstValid can handle — complex fallback chains with logging

Everything else: transform library first. Custom rule second.

---

## Key Takeaways

- The transform library covers 80% of real-world data manipulation — learn it before reaching for BeanShell
- Always define a `default` in Lookup transforms — missing defaults cause null values that silently break provisioning rules
- Use FirstValid for null handling — any attribute that might be null should have a fallback
- Chain transforms to build complex logic from simple pieces — the email example above uses four simple transforms to produce a result most engineers would write a rule for
- Use Reference transforms for reusable logic shared across multiple sources — one update propagates everywhere
- Custom rules are justified for multi-branch conditions, external lookups, and complex parsing — not for things the transform library already handles

> 🔗 **CONNECTS TO:** Transforms shape your data. Correlation rules determine what that data belongs to — which account matches which identity. The next document covers correlation rules in depth: how to write them, how to test them, and how to debug the cases where they fail.
