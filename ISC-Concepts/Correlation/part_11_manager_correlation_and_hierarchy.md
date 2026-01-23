# Part 11 – Manager Correlation and Hierarchical Identity Links (Deep Dive)

> **Purpose of this part**  
Account correlation answers: “Who owns this account?”  
Manager correlation answers: “Who is this identity’s manager?”

They sound similar, but they behave very differently in ISC.

By the end of this part, you should:
- understand what manager correlation is and what it is not
- know the real runtime sequence that makes manager links succeed or fail
- predict the most common failure patterns (null managers, wrong managers, loops)
- design manager correlation that survives real-life data problems

---

## 11.1 Start with the mental model

### Account correlation
Account correlation links:
- **Account → Identity**

This is ownership.

### Manager correlation
Manager correlation links:
- **Identity → Identity**

This is hierarchy.

That one difference changes everything.

Because now the “target” of correlation is not an account.
It is another identity, which must already exist and be correct.

---

## 11.2 Why manager correlation matters more than it looks

Even if you never use manager-based correlation for accounts, manager links drive:

- manager approvals in access requests
- who certifies whom in certifications
- escalation paths
- identity governance reporting and org structure views

If managers are wrong, approvals look correct but are actually wrong.
That becomes an audit and security problem.

---

## 11.3 What manager correlation uses as input

Manager correlation typically starts from a manager reference found in an authoritative source, like HR.

Common patterns:
- employee record contains `managerEmployeeId`
- employee record contains `managerEmail`
- employee record contains `managerUsername`

Important idea:
- **manager correlation does not invent managers**
- it resolves a reference to an actual identity

So the input is almost always:
> “Here is the manager identifier, find the matching identity.”

---

## 11.4 The strict prerequisite most teams miss

For manager correlation to succeed, the manager’s identity must exist **before** the employee’s manager link is resolved.

If ISC cannot find the manager identity at runtime:
- the employee’s manager stays null
- or manager correlation waits until a later run depending on processing order

This is why org charts often look broken right after initial onboarding.

---

## 11.5 The runtime sequence that decides everything

This is the practical runtime reality:

1. Source aggregates accounts
2. Identity attributes are populated (identity profile mappings)
3. Manager reference attribute is populated (example: managerEmployeeId)
4. ISC attempts to resolve that reference into a manager identity
5. ISC writes the identity-to-identity manager link

Where it breaks most often:
- step 3 happens, but step 4 cannot find the manager identity yet
- step 4 succeeds, but manager identity values are stale (identity processing timing)
- step 4 finds multiple candidates because the identifier is not unique

---

## 11.6 Manager correlation is a search problem

Just like account correlation, manager correlation needs to answer:

> “Which identity matches this manager identifier?”

So the manager identifier must map to an identity attribute that is:
- populated
- stable
- unique
- searchable where required

If it is not searchable or not present at the time:
- manager correlation cannot reliably resolve

---

## 11.7 Choosing the right manager identifier

This is a design decision, not a checkbox.

### Strong choices
- managerEmployeeId (best)
- managerWorkerId
- other immutable HR IDs

### Risky choices
- managerEmail (can change, aliases, rehires)
- managerUsername (can change, reuse)
- managerDisplayName (collisions guaranteed)

Rule of thumb:
If you would not use it as a primary account correlation identifier, do not use it for manager correlation either.

---

## 11.8 Why “manager correlation” fails even when data looks correct

The most common explanation is timing.

Example:
- Employee record references managerEmployeeId = 12345
- The manager identity exists in HR, but hasn’t been processed into ISC identities yet
- The employee is processed first

Result:
- employee’s manager is null
- later, after processing, it may resolve automatically or require reprocessing depending on how the tenant is set up

This is not a bad identifier problem.
It is a sequencing problem.

---

## 11.9 The three classic failure patterns

### Pattern 1: Null manager
Causes:
- manager identity not present yet
- manager identifier missing
- manager identifier not searchable / not usable at runtime

### Pattern 2: Wrong manager
Causes:
- identifier not unique (example: email alias collision)
- stale identity values (identity processing lag)
- rehire scenarios where identifier reused

Wrong manager is worse than null manager.

### Pattern 3: Manager loops
Example:
- A reports to B
- B reports to A (bad HR data)
Or:
- identity points to itself

Loops break org charts and can confuse approval routing.

---

## 11.10 Defensive design to prevent manager loops

Even if HR is authoritative, treat manager data as “trusted but verified”.

Defensive rules:
- do not allow self-manager
- detect A↔B loops where possible
- keep an exception workflow for HR corrections

If you allow loops to persist, governance becomes unreliable.

---

## 11.11 Manager correlation and lifecycle changes

Manager relationships change often:
- promotions
- reorganizations
- temporary reporting lines
- contractors moving teams

So you should expect:
- manager links to change over time
- reprocessing to be needed
- “correct today, wrong tomorrow” is normal if the upstream source changes

This is why stable identifiers are critical.

---

## 11.12 Practical troubleshooting flow

When a user’s manager is wrong or null, debug in this order:

1. What is the **manager identifier value** coming from the source (raw)?
2. After mapping, what is the **normalized identity attribute** value?
3. Is the target identity attribute used for lookup **searchable**?
4. Does the manager identity exist **at the time of processing**?
5. Are there multiple identities matching the manager identifier?

If you answer these five, manager correlation stops being mysterious.

---

## 11.13 Real-world example (the sequencing trap)

HR aggregates and processes in alphabetical order by name.

- Employee: Priyanka (managerEmployeeId = 900)
- Manager: Ted (employeeId = 900)

If Priyanka processes before Ted’s identity exists in ISC:
- Priyanka’s manager stays null

On the next processing run, once Ted exists:
- Priyanka’s manager resolves

Fix is not changing mapping.
Fix is controlling processing timing and ensuring manager identities exist early.

---

## 11.14 What this part prepares you for

After this part, you should:
- treat manager correlation as identity-to-identity resolution
- design stable identifiers for manager lookup
- debug manager issues with sequencing and searchability, not guesses
- avoid dangerous “email-based manager” designs unless you truly must

Next, we go deeper into **sequencing strategies** and how to design aggregation and processing so hierarchies resolve cleanly.

---

## Navigation

**⬅ Back:**- [Part 10 – Correlation Rules and Precedence](part_10_correlation_rules_and_precedence.md)

**⬅ Next:**- [Part 12 – Sequencing Strategies for Clean Correlation](part_12_sequencing_strategies_for_clean_correlation.md)

---

*End of Part 11*
