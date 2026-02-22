# STEP 09: Advanced Text Operations & Data Encoding
## Specialized Operations for Complex Text Handling

---

## What You'll Master
- ✅ split - Parse delimited strings
- ✅ normalizeNames - Handle special names
- ✅ decomposeDiacriticalMarks - Remove accents
- ✅ Base64 Encode/Decode - Encode sensitive data
- ✅ e164Phone - Format phone numbers internationally
- ✅ iso3166 - Convert country codes

---

## OPERATION 1: SPLIT - Parse Delimited Strings

### What It Does
Splits a string by a delimiter and extracts a specific part.

### Syntax
```json
{
  "type": "split",
  "attributes": {
    "input": { "type": "identityAttribute", "attributes": { "name": "email" } },
    "delimiter": "@",
    "index": 0
  }
}
```

### Parameters
- **input** - The string to split
- **delimiter** - What character separates the parts
- **index** - Which part to return (0-based indexing)

### Example 1: Extract Username from Email
```json
{
  "type": "split",
  "attributes": {
    "input": { "type": "identityAttribute", "attributes": { "name": "email" } },
    "delimiter": "@",
    "index": 0
  }
}
```

**Input:** email = "john.smith@company.com"  
**Result:** "john.smith"

**How it works:**
1. Split "john.smith@company.com" by "@"
2. Parts: ["john.smith", "company.com"]
3. Return index 0: "john.smith"

---

### Example 2: Extract Domain from Email
```json
{
  "type": "split",
  "attributes": {
    "input": { "type": "identityAttribute", "attributes": { "name": "email" } },
    "delimiter": "@",
    "index": 1
  }
}
```

**Input:** email = "john.smith@company.com"  
**Result:** "company.com"

**How it works:**
1. Split by "@"
2. Parts: ["john.smith", "company.com"]
3. Return index 1: "company.com"

---

### Example 3: Parse Comma-Separated Values
```json
{
  "type": "split",
  "attributes": {
    "input": { "type": "identityAttribute", "attributes": { "name": "roles" } },
    "delimiter": ",",
    "index": 0
  }
}
```

**Input:** roles = "Admin,User,Manager"  
**Result:** "Admin"

**How it works:**
1. Split by ","
2. Parts: ["Admin", "User", "Manager"]
3. Return index 0: "Admin"

---

### Important Notes
- **Index starts at 0** (first part is index 0)
- **If index doesn't exist** - returns NULL
- **Index out of bounds** - returns NULL

---

## OPERATION 2: NORMALIZEENAMES - Handle Special Names

### What It Does
Standardizes names with special handling for:
- "McDonald" vs "Mcdonald" (Mc prefix)
- "MacDonald" vs "Macdonald" (Mac prefix)
- Proper title casing
- International names

### Syntax
```json
{
  "type": "normalizeNames",
  "attributes": {
    "input": { "type": "identityAttribute", "attributes": { "name": "lastName" } }
  }
}
```

### Example 1: Handle Mc Prefix
```json
{
  "type": "normalizeNames",
  "attributes": {
    "input": { "type": "static", "attributes": { "value": "mcdonald" } }
  }
}
```

**Input:** "mcdonald"  
**Output:** "Mcdonald" (Proper Mc handling)

**Why it matters:**
- Input: "mcdonald" → Output: "Mcdonald" (not "MCDONALD")
- Input: "MCDONALD" → Output: "Mcdonald" (correct casing)
- Input: "McDonaLd" → Output: "Mcdonald" (standardized)

---

### Example 2: Handle Mac Prefix
```json
{
  "type": "normalizeNames",
  "attributes": {
    "input": { "type": "static", "attributes": { "value": "macdonald" } }
  }
}
```

**Input:** "macdonald"  
**Output:** "Macdonald" (Proper Mac handling)

---

### Example 3: International Name Normalization
```json
{
  "type": "normalizeNames",
  "attributes": {
    "input": { "type": "identityAttribute", "attributes": { "name": "lastName" } }
  }
}
```

**Input:** "O'BRIEN"  
**Output:** "O'brien" (Proper title case)

**Input:** "van der HEYDEN"  
**Output:** "Van der Heyden" (Proper spacing and case)

---

### Use Cases
- Handle Scottish/Irish names (Mc, Mac)
- International name variations
- Standardize casing across organizations
- Prepare names for official documents

---

## OPERATION 3: DECOMPOSEDIACRITICALMARKS - Remove Accents

### What It Does
Removes accents and special characters from text:
- José → Jose
- Müller → Muller
- François → Francois
- Łódź → Lodz

### Syntax
```json
{
  "type": "decomposeDiacriticalMarks",
  "attributes": {
    "input": { "type": "identityAttribute", "attributes": { "name": "firstName" } }
  }
}
```

### Example 1: Convert Accented Names
```json
{
  "type": "decomposeDiacriticalMarks",
  "attributes": {
    "input": { "type": "static", "attributes": { "value": "José" } }
  }
}
```

**Input:** "José"  
**Output:** "Jose"

---

### Example 2: Handle Multiple Accents
```json
{
  "type": "decomposeDiacriticalMarks",
  "attributes": {
    "input": { "type": "static", "attributes": { "value": "Müller" } }
  }
}
```

**Input:** "Müller"  
**Output:** "Muller"

---

### Example 3: International Names
```json
{
  "type": "decomposeDiacriticalMarks",
  "attributes": {
    "input": { "type": "identityAttribute", "attributes": { "name": "firstName" } }
  }
}
```

**Input:** "François"  
**Output:** "Francois"

**Input:** "Åsa"  
**Output:** "Asa"

---

### Use Cases
- Support international organizations
- Systems that don't support Unicode
- Create ASCII-safe usernames
- Email address standardization
- Filename generation

---

### Often Combined with normalizeNames
```json
{
  "type": "normalizeNames",
  "attributes": {
    "input": {
      "type": "decomposeDiacriticalMarks",
      "attributes": {
        "input": { "type": "identityAttribute", "attributes": { "name": "lastName" } }
      }
    }
  }
}
```

**Input:** "josé"  
**Step 1 (decompose):** "jose"  
**Step 2 (normalize):** "Jose"  
**Final Output:** "Jose"

---

## OPERATION 4: BASE64 ENCODE/DECODE

### Base64 Encode - What It Does
Converts text to Base64 (encoded format):
- "MySecret" → "TXlTZWNyZXQ="
- Used for passwords, sensitive data
- Makes text safe for certain systems

### Syntax - Encode
```json
{
  "type": "base64Encode",
  "attributes": {
    "input": { "type": "identityAttribute", "attributes": { "name": "password" } }
  }
}
```

### Example: Encode a Password
```json
{
  "type": "base64Encode",
  "attributes": {
    "input": { "type": "static", "attributes": { "value": "MyPassword123" } }
  }
}
```

**Input:** "MyPassword123"  
**Output:** "TXlQYXNzd29yZDEyMw=="

---

### Base64 Decode - What It Does
Converts Base64 back to text:
- "TXlQYXNzd29yZDEyMw==" → "MyPassword123"

### Syntax - Decode
```json
{
  "type": "base64Decode",
  "attributes": {
    "input": { "type": "identityAttribute", "attributes": { "name": "encodedValue" } }
  }
}
```

### Example: Decode a Password
```json
{
  "type": "base64Decode",
  "attributes": {
    "input": { "type": "static", "attributes": { "value": "TXlQYXNzd29yZDEyMw==" } }
  }
}
```

**Input:** "TXlQYXNzd29yZDEyMw=="  
**Output:** "MyPassword123"

---

### Use Cases
- Encode passwords for transmission
- Prepare data for API requests requiring Base64
- Handle encoded data from systems
- Security compliance (encoded vs plaintext)
- OAuth/API authentication tokens

---

## OPERATION 5: E164PHONE - Format Phone Numbers

### What It Does
Formats phone numbers to E.164 international standard:
- "(555) 123-4567" → "+15551234567"
- "555-123-4567" → "+15551234567"
- "5551234567" → "+15551234567"

All become: +[country code][phone number]

### Syntax
```json
{
  "type": "e164Phone",
  "attributes": {
    "input": { "type": "identityAttribute", "attributes": { "name": "phone" } }
  }
}
```

### Example 1: US Phone Number
```json
{
  "type": "e164Phone",
  "attributes": {
    "input": { "type": "static", "attributes": { "value": "(555) 123-4567" } }
  }
}
```

**Input:** "(555) 123-4567"  
**Output:** "+15551234567"

---

### Example 2: International Format
```json
{
  "type": "e164Phone",
  "attributes": {
    "input": { "type": "static", "attributes": { "value": "555-123-4567" } }
  }
}
```

**Input:** "555-123-4567"  
**Output:** "+15551234567"

---

### Example 3: Raw Numbers
```json
{
  "type": "e164Phone",
  "attributes": {
    "input": { "type": "identityAttribute", "attributes": { "name": "phone" } }
  }
}
```

**Input:** "5551234567"  
**Output:** "+15551234567"

---

### Use Cases
- Standardize phone numbers globally
- Prepare for VoIP/telecom systems
- International calling support
- SMS gateway integration
- Contact directory standardization

---

## OPERATION 6: ISO3166 - Convert Country Codes

### What It Does
Converts between country formats:
- "United States" → "US" or "USA"
- "Canada" → "CA" or "CAN"
- Standardizes country representations

### Syntax
```json
{
  "type": "iso3166",
  "attributes": {
    "input": { "type": "identityAttribute", "attributes": { "name": "country" } }
  }
}
```

### Example 1: Country Name to Code
```json
{
  "type": "iso3166",
  "attributes": {
    "input": { "type": "static", "attributes": { "value": "United States" } }
  }
}
```

**Input:** "United States"  
**Output:** "US" (or "USA" depending on format)

---

### Example 2: Country Code Standardization
```json
{
  "type": "iso3166",
  "attributes": {
    "input": { "type": "identityAttribute", "attributes": { "name": "country" } }
  }
}
```

**Input:** "Germany"  
**Output:** "DE" (or "DEU")

---

### Use Cases
- Standardize country codes
- International address validation
- Global organization support
- Compliance reporting
- Data consistency across systems

---

## COMBINING MULTIPLE OPERATIONS

### Real Example: Standardize International Name
```json
{
  "type": "normalizeNames",
  "attributes": {
    "input": {
      "type": "decomposeDiacriticalMarks",
      "attributes": {
        "input": {
          "type": "lower",
          "attributes": {
            "input": { "type": "identityAttribute", "attributes": { "name": "lastName" } }
          }
        }
      }
    }
  }
}
```

**Input:** "JOSÉ"  
**Step 1 (lower):** "josé"  
**Step 2 (decompose):** "jose"  
**Step 3 (normalize):** "Jose"  
**Output:** "Jose"

---

### Real Example: Parse and Process Email
```json
{
  "type": "lower",
  "attributes": {
    "input": {
      "type": "split",
      "attributes": {
        "input": { "type": "identityAttribute", "attributes": { "name": "email" } },
        "delimiter": "@",
        "index": 0
      }
    }
  }
}
```

**Input:** email = "John.Smith@Company.com"  
**Step 1 (split):** "John.Smith"  
**Step 2 (lower):** "john.smith"  
**Output:** "john.smith"

---

## KEY TAKEAWAYS

1. **split** - Parse delimited strings
   - Extract username from email
   - Extract domain
   - Parse comma-separated values

2. **normalizeNames** - Handle special names
   - McDonald/Mcdonald
   - Proper title casing
   - International variations

3. **decomposeDiacriticalMarks** - Remove accents
   - José → Jose
   - Müller → Muller
   - Make names ASCII-safe

4. **Base64 Encode/Decode** - Encode data
   - Encode passwords
   - Prepare for API transmission
   - Security compliance

5. **e164Phone** - Format phones internationally
   - Standardize formats
   - Add country codes
   - Support telecom systems

6. **iso3166** - Country code standardization
   - Convert to ISO format
   - Global compliance
   - Data consistency

---

## WHAT'S NEXT

Now you've learned:
- ✅ STEP 01-08: Foundation, JSON, and basic operations
- ✅ STEP 09: Advanced text and encoding operations

Next:
- **STEP 10:** Advanced Patterns and Expert Thinking (already created)
- **STEP 11:** Error Prevention & Debugging
- **STEP 12:** Testing & Validation
- **STEP 13:** Practice Module (10 exercises)

Ready? Move to STEP 10.
