# Part 13 – End-to-End Correlation Troubleshooting Playbook (Deep Dive)

> **Purpose of this part**  
This chapter is the *unifying playbook* for everything you learned in Parts 0–12.
It turns concepts into a **repeatable diagnostic method** you can use in production, incidents, audits, and migrations.

This is not a checklist.
It is a **thinking sequence**.

By the end of this part, you should:
- troubleshoot correlation issues without guessing
- identify root cause in minutes, not days
- explain correlation behavior clearly to others
- fix the right layer without creating new problems

---

## 13.1 The core principle of troubleshooting

> **Correlation never fails randomly.**

If the outcome is unexpected, one of these is always true:
- the data was not ready
- the identity did not exist
- the wrong layer decided ownership
- the system ran too early
- a previous decision was preserved

Your job is to find *which one*.

---

## 13.2 Always start with the symptom, not the theory

Begin with a **single concrete account**.

Never start with:
- “correlation is broken”
- “rules are wrong”
- “ISC bug?”

Start with:
> “This specific account attached to the wrong identity / stayed uncorrelated.”

Correlation is evaluated per account.
So troubleshooting must be per account.

---

## 13.3 Step 1 – Identify the correlation outcome

Ask:
- is the account correlated or uncorrelated?
- if correlated, to which identity?
- is that identity expected?

This tells you whether the problem is:
- missing ownership
- wrong ownership
- unexpected persistence

---

## 13.4 Step 2 – Check for manual correlation first

Manual correlation overrides everything.

Always ask:
- was this account manually correlated?
- was it ever manually correlated in the past?

If yes:
- rule changes do nothing
- criteria changes do nothing
- aggregation does nothing

Manual correlation must be removed before logic can reapply.

---

## 13.5 Step 3 – Check for correlation rules

Next question:
- does a correlation rule exist on this source?

If a rule:
- returns an identity, criteria never run
- returns no decision, criteria proceed

Many “invisible” behaviors come from rules that quietly return a result.

---

## 13.6 Step 4 – Evaluate configured criteria execution

If no rule decided ownership:
- criteria are evaluated in order

Check:
- which criteria matched?
- why did earlier criteria not match?
- was the identifier populated at runtime?

Never assume a row ran just because it exists.

---

## 13.7 Step 5 – Consider default fallback

If no criteria matched:
- did default fallback activate?
- did account name match identity name?
- did case-insensitive comparison matter?

Fallback explains many “how did this correlate?” cases.

---

## 13.8 Step 6 – Verify attribute readiness at runtime

This is where most root causes live.

Ask:
- did the attribute exist **at aggregation time**?
- was it populated **before** correlation ran?
- was it searchable if required?

Seeing a value later in the UI proves nothing about runtime.

---

## 13.9 Step 7 – Verify identity existence and timing

Correlation cannot attach to an identity that does not exist.

Ask:
- did the identity exist at that run?
- did identity processing complete first?
- was the identity partially built?

If identity came later, correlation failure is expected.

---

## 13.10 Step 8 – Check aggregation type

Ask:
- was this a full aggregation?
- or delta aggregation?

Delta aggregation:
- skips unchanged accounts
- preserves previous correlation decisions

Many “why didn’t it change?” issues are delta behavior, not bugs.

---

## 13.11 Step 9 – Evaluate sequencing

Zoom out.

Ask:
- which source ran first?
- did identity processing run before applications?
- did jobs overlap?

Sequencing issues explain problems that look inconsistent.

---

## 13.12 Step 10 – Inspect persistence and memory

Correlation is sticky.

Ask:
- was this decision made earlier?
- is ISC preserving an old link?
- was reprocessing ever forced?

Old decisions often outlive new logic.

---

## 13.13 Step 11 – Decide the correct fix layer

Once root cause is clear, choose the fix carefully.

Possible fix layers:
- upstream source data
- identity profile mapping
- identity processing timing
- correlation criteria order
- correlation rules
- manual intervention (last resort)

Fix the *earliest* broken layer.

---

## 13.14 What not to do during troubleshooting

Avoid:
- adding new criteria “just to test”
- manually correlating without proof
- forcing optimization to change outcomes
- assuming UI visibility equals runtime truth

These create permanent damage.

---

## 13.15 Explaining correlation issues to others

When explaining to app teams or auditors, use this framing:

> “The account did not correlate because the identity did not exist at the time of aggregation.”

Or:

> “The account is correlated incorrectly because a manual correlation overrides rule-based logic.”

Clear explanation builds trust.

---

## 13.16 Real-world incident walkthrough

Incident:
- CFO’s account approved access incorrectly.

Investigation:
- account manually correlated years ago
- manager changed
- correlation rules never re-evaluated

Fix:
- remove manual correlation
- reprocess account
- let criteria apply correctly

Root cause was not configuration.
It was persistence.

---

## 13.17 The mental checklist you should internalize

When correlation surprises you, always think:

Manual?  
Rule?  
Criteria?  
Fallback?  
Timing?  
Persistence?

That sequence never lies.

---

## 13.18 What this part completes

After this part, you should:
- troubleshoot correlation calmly
- prove root cause with evidence
- avoid panic fixes
- operate ISC correlation confidently

This completes the **Correlation Mastery Core Series**.

---

## Navigation

**⬅ Back:** Part 12 – Sequencing Strategies for Clean Correlation and Manager Resolution  
**➡ Next:** Part 14 – Advanced Correlation Scenarios and Edge Cases

---

*End of Part 13*
