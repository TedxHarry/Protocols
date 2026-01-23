# Part 10 – Correlation Rules and Precedence (Deep Dive)

> **Purpose of this part**  
This chapter explains how **correlation rules** fit into the overall correlation engine and how **precedence** truly works.
This is where correlation stops being declarative and becomes programmable.

By the end of this part, you should:
- understand what correlation rules really are
- know how rules interact with criteria and defaults
- understand execution order and precedence
- design rule logic without creating unpredictable behavior

---

## 10.1 First, clear a dangerous misunderstanding

Correlation rules do **not** “add another row” to criteria.

Correlation rules:
- sit **above** criteria
- can override normal behavior
- short-circuit inference entirely

Think of rules as *custom decision engines*, not configuration helpers.

---

## 10.2 Where correlation rules sit in the precedence stack

Correlation decisions follow a strict confidence hierarchy:

1. **Manual correlation** – explicit human directive  
2. **Correlation rules** – programmable logic  
3. **Configured criteria list** – ordered inference  
4. **Default fallback** – identity name matching  

Higher layers override lower ones.

---

## 10.3 What a correlation rule actually does

A correlation rule:
- receives the account object
- can inspect account attributes
- can query identities
- returns a decision

Possible outcomes:
- assign the account to a specific identity
- return no decision and allow normal criteria to run

This flexibility is powerful and dangerous.

---

## 10.4 Rules are evaluated per account, per aggregation

Just like criteria, rules:
- run one account at a time
- run only during aggregation
- do not re-run automatically later

Rules are not global batch logic.
They are **runtime decision points**.

---

## 10.5 Why rules exist at all

Rules exist to handle scenarios that criteria cannot express, such as:
- conditional logic
- compound matching
- external context checks
- special account classes

If criteria can solve the problem safely, prefer criteria.
Rules are for edge cases, not defaults.

---

## 10.6 The biggest rule design mistake

The most common mistake:

> “Let’s put all correlation logic into a rule.”

This leads to:
- opaque behavior
- debugging nightmares
- audit resistance
- brittle systems

Rules hide intent.
Criteria document intent.

---

## 10.7 How rules interact with criteria

Two safe patterns exist:

### Pattern 1: Rule as gatekeeper
- rule handles special cases
- returns no decision for normal accounts
- criteria handle the majority

### Pattern 2: Rule as override
- rule explicitly assigns ownership for known exceptions
- criteria never see those accounts

Avoid rules that try to replace criteria entirely.

---

## 10.8 Returning “no decision” is a feature

A well-designed rule often does **nothing**.

Returning no decision:
- hands control back to criteria
- keeps logic layered
- preserves explainability

A rule that always returns something is usually wrong.

---

## 10.9 Rule precedence surprises

If a rule assigns ownership:
- criteria are skipped
- fallback is skipped
- rule result persists like correlation

Admins then wonder:
> “Why didn’t my criteria changes apply?”

The answer is precedence.

---

## 10.10 Debugging rule-driven correlation

When behavior is unexpected, ask:

1. is there a manual correlation?
2. is there a correlation rule?
3. did the rule return an identity?
4. did criteria even execute?
5. did fallback activate?

Rules often explain “invisible” behavior.

---

## 10.11 Performance considerations

Rules run during aggregation.

Poorly written rules:
- slow aggregation
- increase timeouts
- affect all accounts

Rules should be:
- narrow in scope
- efficient
- predictable

Never perform expensive searches per account without limits.

---

## 10.12 Auditing and explainability

Auditors care about:
- how ownership was decided
- whether it is repeatable
- whether it is justified

Criteria are self-documenting.
Rules require documentation.

Always document:
- why the rule exists
- what cases it handles
- how it should behave over time

---

## 10.13 Real-world example

Scenario:
- service accounts follow a naming pattern
- criteria cannot distinguish them safely

Rule logic:
- if accountName starts with svc_
- assign to service owner identity
- otherwise return no decision

Result:
- service accounts handled cleanly
- user accounts still follow criteria
- logic remains understandable

---

## 10.14 When rules should be avoided

Avoid correlation rules when:
- identifiers are clean
- criteria ordering is sufficient
- the logic is simple equality

Rules add complexity.
Only use them when necessary.

---

## 10.15 What this part prepares you for

After this part, you should:
- understand why rules override criteria
- design layered correlation safely
- debug precedence issues confidently

Next, we move into **manager correlation**, where identity relationships introduce another layer of complexity.

---

## Navigation

**⬅ Back:**- [Part 9 – Default Correlation Fallback and Case Sensitivity](part_9_default_correlation_fallback_and_case_sensitivity.md)

**⬅ Next:**- [Part 11 – Manager Correlation and Hierarchical Identity Links](part_11_manager_correlation_and_hierarchy.md)

---

*End of Part 10*
