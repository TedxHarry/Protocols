# Part 15 – Correlation Design Review Checklist and Governance Readiness

> **Purpose of this part**  
This chapter is the practical close to the Correlation Mastery Series.
It gives you a **defensible review framework** you can apply before audits, new app onboarding, mergers, or production changes.

This is not a generic checklist.
It is a **decision framework** that reflects how ISC actually behaves.

By the end of this part, you should:
- review any tenant’s correlation design with confidence
- spot hidden risk before it becomes an incident
- assess governance readiness realistically
- explain correlation posture to auditors and leadership

---

## 15.1 The mindset shift

Stop asking:
> “Does correlation work?”

Start asking:
> “Is correlation **defensible**?”

Defensible means:
- ownership decisions are explainable
- behavior is predictable
- exceptions are intentional
- risk is visible

---

## 15.2 Layer 1 review – Identity foundation

Ask these questions first:

- Is there exactly **one** authoritative source that creates identities?
- Are identity creation rules deterministic?
- Are core identifiers immutable or near-immutable?
- Do identities exist **before** application aggregation?

If this layer is weak, correlation above it cannot be trusted.

---

## 15.3 Layer 2 review – Attribute readiness

Review the attributes used for correlation:

- Are they populated early in the lifecycle?
- Are they stable over time?
- Are they unique in practice, not theory?
- Are custom attributes marked searchable where required?
- Has identity processing completed before aggregation?

If attributes are late or unstable, correlation results will drift.

---

## 15.4 Layer 3 review – Correlation criteria design

Evaluate criteria design intentionally:

- Are criteria ordered by **risk**, not convenience?
- Is there a strong primary identifier?
- Are fallback identifiers documented and justified?
- Is default fallback relied on or merely tolerated?

Good correlation usually needs fewer rows, not more.

---

## 15.5 Layer 4 review – Correlation rules

If correlation rules exist:

- Why does each rule exist?
- What specific cases does it handle?
- Does it return “no decision” for normal accounts?
- Is its behavior documented and testable?

Rules without documentation are future outages waiting to happen.

---

## 15.6 Layer 5 review – Manual correlation usage

Manual correlation should be rare.

Review:
- how many manually correlated accounts exist?
- why they were manually correlated
- whether ownership proof exists
- whether they are periodically revalidated

Manual correlation without governance is hidden debt.

---

## 15.7 Layer 6 review – Manager correlation health

Manager links drive approvals.

Check:
- which identifier is used for manager lookup
- whether manager identities exist early
- whether null managers are expected or accidental
- whether loops or self-managers exist

Wrong managers create silent governance failures.

---

## 15.8 Layer 7 review – Sequencing and scheduling

Review job timing holistically:

- authoritative aggregation first
- identity processing immediately after
- applications only after identities settle
- no unsafe overlaps

Most “random” issues are sequencing issues.

---

## 15.9 Layer 8 review – Optimization and metrics

Metrics should inform, not drive.

Ask:
- are optimization suggestions reviewed or blindly applied?
- is uncorrelated count explained and accepted?
- are percentages used as signals, not targets?

Healthy tenants tolerate some ambiguity.

---

## 15.10 Layer 9 review – Persistence awareness

Correlation decisions persist.

Confirm:
- teams understand delta vs full aggregation behavior
- reprocessing strategy exists
- old manual links are not assumed to self-correct

Memory is a feature, not a bug.

---

## 15.11 Governance readiness questions

You should be able to answer “yes” to these:

- Can we explain why any given account is correlated?
- Can we explain why some accounts are uncorrelated?
- Can we prove ownership decisions to an auditor?
- Can we predict behavior after a rule or schedule change?

If not, readiness is incomplete.

---

## 15.12 Pre-onboarding correlation readiness (new apps)

Before onboarding a new application, ask:

- which identity identifier will it use?
- does that identifier exist early?
- what happens if correlation fails?
- is fallback acceptable here?
- who owns uncorrelated accounts?

If these answers are unclear, pause onboarding.

---

## 15.13 M&A and migration readiness

During migrations:

- reduce automation temporarily
- accept higher uncorrelated counts
- avoid aggressive correlation
- prioritize traceability over coverage

Governance integrity beats speed.

---

## 15.14 Executive explanation (non-technical)

A simple way to explain correlation posture:

> “We only automate ownership when we can prove it.  
When we can’t, we make the risk visible instead of guessing.”

That statement signals maturity.

---

## 15.15 Final takeaway

Strong correlation design is not about:
- eliminating uncorrelated accounts
- maximizing percentages
- adding more logic

It is about:
- **trust**
- **predictability**
- **defensibility**

If you achieve those, governance works.

---

## Navigation

**⬅ Back:** Part 14 – Advanced Correlation Scenarios and Edge Cases  
**🏁 End:** Correlation Mastery Series Complete

---

*End of Part 15*
