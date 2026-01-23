# Part 9 – Default Correlation Fallback and Case Sensitivity

> **Purpose of this part**  
This chapter explains correlation behavior that surprises even experienced admins:
- accounts that correlate even though no rule seems to match
- matches that succeed despite case differences
- correlation outcomes no one explicitly configured

These are not bugs.
They are **default behaviors** built into ISC.

By the end of this part, you should:
- understand default correlation fallback logic
- know when and why it activates
- understand case sensitivity behavior
- diagnose “mystery correlations” confidently

---

## 9.1 The misconception to clear first

A common belief:

> “If no correlation rule matches, the account stays uncorrelated.”

That is **not always true**.

ISC includes a **default correlation fallback** designed to prevent obvious misses.

---

## 9.2 What default correlation fallback is

Default correlation fallback is ISC saying:

> “If no configured criteria matched, try a basic identity name comparison.”

This is not configurable.
It exists to catch simple ownership matches.

---

## 9.3 How default fallback works conceptually

When all configured correlation criteria fail:

1. ISC takes the account’s **account name**
2. ISC compares it with the identity’s **name**
3. If values match, correlation succeeds

This comparison happens automatically.
No UI configuration is required.

---

## 9.4 Why this fallback exists

Without fallback:
- simple environments would require extra configuration
- obvious matches would remain uncorrelated
- onboarding friction would increase

Fallback improves usability.
But it introduces surprises if you do not expect it.

---

## 9.5 What makes fallback dangerous

Fallback correlation:
- ignores business intent
- ignores lifecycle design
- ignores uniqueness guarantees

It assumes:
> “Same name means same person.”

This assumption is sometimes wrong.

---

## 9.6 When fallback usually triggers

Fallback correlation commonly appears when:
- no criteria are configured
- criteria are misordered
- primary identifiers are missing
- attributes were not searchable
- correlation rules failed earlier

Admins then ask:
> “Why did this correlate?”

The answer is fallback.

---

## 9.7 Case sensitivity: another hidden behavior

By default, correlation comparisons are **case-insensitive**.

This means:
- USER123 equals user123
- Ted equals ted
- Email comparisons ignore case

This applies to:
- configured criteria
- default fallback

---

## 9.8 Why case-insensitive matching exists

Most systems treat identifiers case-insensitively.

ISC aligns with that reality to:
- reduce false negatives
- improve match rates

However, some legacy systems treat case as significant.

---

## 9.9 When case-insensitivity causes problems

Case-insensitive matching is risky when:
- identifiers differ only by case
- systems reuse usernames with case variation
- external systems enforce case sensitivity

In these scenarios:
- correlation may attach incorrectly
- fallback becomes more dangerous

---

## 9.10 Changing case sensitivity behavior

Case sensitivity is not adjusted in the UI.

It requires API-level configuration.

This is intentional.
Changing case behavior impacts performance and correctness.

Such changes should be rare and tested.

---

## 9.11 How fallback and case-insensitivity combine

This is the most confusing scenario.

Example:
- no criteria match
- account name = Ted
- identity name = ted
- case-insensitive fallback triggers
- correlation succeeds

Admins see correlation and assume a rule matched.

It did not.

---

## 9.12 Diagnosing mystery correlations

When correlation surprises you, ask:

1. was there a manual correlation?
2. did a correlation rule run?
3. did configured criteria match?
4. did default fallback activate?
5. did case-insensitive comparison matter?

Answering these removes guesswork.

---

## 9.13 How to design defensively around fallback

Best practices:
- configure explicit criteria for intended ownership
- do not rely on fallback for production correctness
- design identity names carefully
- avoid identity name reuse

Fallback should be safety net, not strategy.

---

## 9.14 Real-world example

An environment has no correlation configured.

Account name: jsmith  
Identity name: jsmith  

Fallback correlates the account.

Later, a contractor named John Smith joins.
Identity name reused.

Result:
- fallback attaches the wrong account
- audit finds incorrect ownership

Fallback did its job.
Design did not.

---

## 9.15 When fallback is acceptable

Fallback is acceptable when:
- identity names are guaranteed unique
- environment is small and controlled
- lifecycle is simple

As complexity grows, reliance on fallback should shrink.

---

## 9.16 What this part prepares you for

After this part, you should:
- recognize default behaviors immediately
- stop blaming “random correlation”
- design correlation that does not depend on fallback

Next, we move into **correlation rules and precedence**, where logic becomes programmable.

---

## Navigation

**⬅ Back:** Part 8 – Manual Correlation, Permanence, and Risk  
**➡ Next:** Part 10 – Correlation Rules and Precedence

---

*End of Part 9*
