
# Part 4 – Normalization and Mapping (Best)

[⬅️ Back to Home](../README.md)

---

## Purpose
This part explains what happens after data is read but before it is stored.

Extraction only copies raw data.  
Normalization and mapping decide what that data **means** inside ISC.

If this step is wrong, everything downstream will look wrong in confusing ways, because the data is shaped incorrectly before it ever becomes an account or identity.

---

## Where This Fits in the Master Flow

Trigger → Extract → **Normalize** → Persist → Correlate → Evaluate → Recompute → Publish

Normalization is the translator between the source world and the ISC world.  
If normalization lies, persistence stores a lie.

---

## Visual Mental Map

Raw Source Data  
→ Normalize (convert types, clean values)  
→ Map (assign meaning and destination)  
→ Stored Account Attributes

This is where “data” becomes “truth.”

---

## Mini-Glossary (Plain English)

Normalization: converting raw source data into ISC’s data types.  
Mapping: deciding where each source field goes in ISC.  
Transform: changing a value with logic.  
Data type: text, number, date, boolean, list.  
Silent failure: data breaks without loud errors.

---

## Wrong vs Right Thinking

Wrong: Extraction already fixed the data.  
Right: Extraction only copied it. Normalization defines meaning.

Wrong: If job completes, mapping is fine.  
Right: Mapping can fail silently and still say Completed.

Wrong: Empty field means source was empty.  
Right: Empty field usually means mapping or type mismatch.

---

## Failure Story

HR had hire dates for everyone.  
ISC showed hireDate empty for all users.

Team blamed identity logic.

Root cause:  
hireDate mapped as text into a date field.  
Normalization dropped it silently.

---

## Running Example

Source: HR system  
People: Alice, Bob, Carol  
Accounts: HR_Alice, HR_Bob, HR_Carol  
Fields: employeeId, email, department, hireDate  

Extraction already read raw data.  
Now ISC must translate it.

---

## What Normalization Really Means

Systems don’t speak the same language.

One system stores dates as text.  
Another as numbers.  
ISC wants a date object.

Normalization is ISC saying:  
“I will convert whatever you give me into my own types.”

If wrong:
Dates become text.  
Numbers become strings.  
Lists collapse.  
Later logic breaks quietly.

---

## What Mapping Really Means

Mapping is you teaching ISC meaning.

“When you see this field in the source, put it here.”

Example:
employeeId → accountAttribute.employeeId  
email → accountAttribute.email  
department → accountAttribute.department  
hireDate → accountAttribute.hireDate  

You are not copying.  
You are assigning meaning.

---

## Data Types Matter More Than You Think

Every field has a type.

Single vs multi-value matters.  
Text vs date matters.  
Number vs string matters.

Common silent bugs:
Multi-value into single → only one kept  
Text into date → dropped  
Number into text → breaks comparisons  

Job still says Completed.

---

## Transforms and Logic

Sometimes raw value is not what you want.

Examples:
first + last → displayName  
“Y” / “N” → true / false  
trim spaces  
change case  

Transforms run during normalization.  
Wrong transform = wrong data forever.

---

## Silent Failures

This phase fails quietly.

You may see:
Fields always empty  
Values partially missing  
Weird formatting  

And the job still says Completed.

---

## Running Example Through Normalization

Raw HR record:
employeeId = "1001"  
email = "alice@company.com"  
department = "Engineering"  
hireDate = "2025-01-01"  

Normalization:
"1001" → number 1001  
"2025-01-01" → date  

Mapping:
1001 → accountAttribute.employeeId  
alice@company.com → accountAttribute.email  
Engineering → accountAttribute.department  
2025-01-01 → accountAttribute.hireDate  

Now data has meaning inside ISC.

---

## Learning Check

If source had many values but ISC shows only one,  
most likely mistake: single-value field used for multi-value data.

---

## Why This Matters

If normalization or mapping is wrong:
Persistence stores wrong data.  
Correlation uses wrong keys.  
Identity logic misfires.  
Access logic breaks.

Many bugs blamed on correlation start here.

---

## Debug Playbook

When values look wrong:

1) Check mapping preview  
2) Check target field type  
3) Check transform logic  
4) Compare source vs normalized value  
5) Only then blame later phases  

---

## Broken Flow Walkthrough

Symptom:
Alice has department in source but empty in ISC.

Check:
Source has department ✔  
Extraction logs show department ✔  
Mapping preview shows blank ❌  

Root cause:
Wrong source field mapped.

Not correlation.  
Not identity logic.  
Normalization error.

---

## If You See This → Do This

Field empty → check mapping  
Weird format → check transform  
Only one value → check multi vs single  
Wrong comparisons → check type  

---

## How People Fail Here

Trust defaults blindly  
Ignore data types  
Skip preview testing  
Copy transforms without understanding  
Assume errors are loud  

---

## Proof Paths

Mapping preview shows transformed values.  
API shows stored account attributes.  
Logs show transform and type warnings.

---

## What Must Not Happen

Do not change mapping mid-run  
Do not assume empty = source empty  
Do not ignore type warnings  

---

## Safe Fixes

Fix mapping → preview again  
Fix transform → test one record  
Fix type mismatch → rerun small scope  

---

## Quick Pre-Run Checklist

Before running big aggregation:
- Preview shows correct values
- Types match target fields
- Multi-value fields mapped correctly
- Transforms tested with sample record
- No warnings ignored

---

## Mindset

Never ask:  
Why is correlation wrong?

First ask:  
Was the data shaped correctly?

---

## Confidence Check

You understand this part if you can answer:

Difference between extraction and normalization?  
Why do data types matter?  
Why jobs can complete with wrong data?  
Why later bugs often start here?

---

### Navigation
⬅️ Previous: [Part 3 – Extraction Phase](../Part_3_Extraction_Phase_Best.md)  
🏠 Home: [README – Aggregation Master Series](../README.md)  
➡️ Next: [Part 5 – Write and Persistence](../Part_5_Write_and_Persistence.md)
