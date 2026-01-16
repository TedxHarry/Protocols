
# Part 3 – Extraction Phase (Reading Data)

[⬅️ Back to Home](../README.md)

---

## Purpose
This part explains what really happens when ISC starts reading data from a source.

Most people imagine extraction as “ISC pulls data.”  
In reality, it is a conversation between ISC and the source, page by page, object by object, with rules about what is allowed, what is skipped, and what can be silently ignored.

If extraction is wrong, everything downstream becomes a lie, even if later steps work perfectly.

---

## Where This Fits in the Master Flow

Trigger → Job → **Extract** → Normalize → Persist → Correlate → Evaluate → Recompute → Publish

Extraction answers one brutal question:  
What data even gets a chance to exist inside ISC?

If extraction lies, everything after it tells a prettier lie.

---

## Vocabulary (Blunt Version)

| Word | What it really means |
|------|----------------------|
| Extraction | Reading raw objects from the source |
| Account extraction | Reading user/object records |
| Entitlement extraction | Reading groups, roles, permissions |
| Membership extraction | Reading who belongs to what |
| Pagination | Reading in pages, not all at once |
| Filtering / Scoping | Rules that include or exclude data |
| Full | Read everything |
| Delta | Read only changes |

---

## Wrong vs Right Thinking

Wrong: Extraction is one big download.  
Right: Extraction is many small requests with paging and filters.

Wrong: Completed means complete data.  
Right: Completed only means the worker stopped.

Wrong: Missing users = correlation problem.  
Right: Missing users = extraction or scoping problem first.

---

## Failure Story

Source had 5,000 users.  
ISC showed only 500.

Team blamed correlation for days.

Root cause: pagination broke after first page.  
Extraction silently stopped early.

---

## Running Example

Source: HR system  
People: Alice, Bob, Carol  
Accounts: HR_Alice, HR_Bob, HR_Carol  
Entitlements: Engineering, HR, Finance  

Job has started. It is now extracting.  
What is really happening?

---

## What Extraction Actually Means

Extraction is not one big pull.  
It is a series of requests like:

Give me first 100 records.  
Give me next 100.  
Give me only records changed since last time.

Connector controls:
- Page size
- Next-page logic
- Filters and delta rules

If any of this is wrong, ISC may think it read everything when it did not.

---

## Account Extraction

Reads main objects, usually users.

In our example:
HR → Alice, Bob, Carol.

ISC does not care about identity yet.  
It only cares about raw accounts.

If account extraction fails:
Accounts missing.  
Correlation never even gets a chance.

---

## Entitlement Extraction

Reads groups, roles, permissions.

In our example:
Engineering, HR, Finance.

If entitlements are missing:
Accounts exist.  
But access modeling and certifications later look empty.

---

## Membership Extraction

Links accounts to entitlements.

Alice → Engineering  
Bob → HR  
Carol → Finance  

If membership fails:
Accounts exist.  
Entitlements exist.  
Nobody has access.

---

## Learning Check

If accounts exist and entitlements exist but nobody has access,  
first suspect: membership extraction.

---

## Full vs Delta Extraction

Full: read everything. Slow but safe.  
Delta: read only changes. Fast but fragile.

Delta depends on memory (tokens, timestamps, watermarks).  
If that memory breaks, it may:
- Skip changes
- Act like full
- Create partial truth

---

## Pagination and Size Limits

Most systems return data in pages.

Connector must:
Page 1 → Page 2 → Page 3 → until done.

If pagination breaks:
Only first page read.  
Job may still say Completed.  
Most data is missing.

This is the most dangerous silent failure.

---

## Filtering and Scoping

Filters decide what gets read.

Examples:
Only active users.  
Only changed users.  
Only certain OUs or containers.

If filter is wrong:
Some people never appear.  
You will blame correlation or identity logic.  
But extraction never saw them.

---

## Running Example Through Extraction

Account extraction:
Alice, Bob, Carol.

Entitlement extraction:
Engineering, HR, Finance.

Membership extraction:
Alice→Engineering, Bob→HR, Carol→Finance.

Pagination:
One page only.

Filtering:
None. All appear.

Extraction complete. Raw data is correct.

---

## Why This Matters

If extraction is wrong:
Normalization fixes wrong data.  
Persistence stores wrong data.  
Correlation links wrong data.

Every later bug may be an extraction bug in disguise.

---

## Debug Playbook

When data looks wrong:

1) Compare source count vs extracted count  
2) Check filters and scoping  
3) Check pagination in logs  
4) Check delta markers  
5) Only then blame later stages

---

## Broken Flow Example

Alice missing in ISC.

Source has Alice ✔  
Logs never mention Alice ❌  
Accounts table missing Alice ❌

Root cause likely:
Filter excluded her.  
Wrong scope.  
Pagination broke.

Not correlation. Not identity logic. Extraction visibility.

---

## If You See This → Do This

Accounts missing → check account extraction and filters  
Entitlements missing → check entitlement permissions  
Nobody has access → check membership extraction  
Some users missing → check pagination first  

---

## Common Failures

Trusting Completed without counts.  
Forgetting pagination exists.  
Using filters blindly.  
Trusting delta too early.

---

## Proof Paths

UI: job counts  
API: object counts  
Logs: page-by-page extraction  

---

## What Must Not Happen

Do not trust Completed as complete.  
Do not change filters casually.  
Do not ignore pagination warnings.

---

## Safe Fixes

Counts wrong → run full extraction once.  
Filters wrong → simplify and test.  
Pagination broken → reduce page size or fix connector.

---

## Mindset

Never ask:
Why is correlation broken?

First ask:
Did extraction even read the object?

---

## Confidence Check

If you can answer these, you understand extraction:

What is account vs entitlement vs membership extraction?  
Why is pagination dangerous when broken?  
Why can later steps look wrong because of extraction?  
What is risky about delta extraction?

---

### Navigation
⬅️ Previous: [Part 2 – Triggers and Job Lifecycle](../Part_2_Triggers_and_Job_Lifecycle_Best.md)  
🏠 Home: [README – Aggregation Master Series](../README.md)  
➡️ Next: [Part 4 – Normalization and Mapping](../Part_4_Normalization_and_Mapping.md)
