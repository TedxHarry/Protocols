# How to Diagnose Any Integration Problem

Random configuration changes are not a troubleshooting methodology. They're how engineers spend three hours "trying things" and end up further from the answer than when they started — often having introduced new problems in the process.

Every integration problem, regardless of how novel it seems, follows a diagnostic path. Source data wrong? Provisioning failing? Correlation producing unexpected results? Lifecycle state not triggering? Each of these starts with a symptom and has a traceable chain back to a root cause. The skill is knowing which chain to follow.

This document gives you that methodology — a repeatable process that applies to any ISC integration problem.

---

## The Six-Step Diagnostic Process

### Step 1: Understand the Symptom Precisely

"The integration is broken" is not a symptom. "James Rivera submitted an access request for Salesforce on Monday and it's still in Pending state three days later" is a symptom.

Before investigating anything, answer these questions:

**What exactly is wrong?** (Be specific)
- Wrong: "AD isn't working"
- Right: "AD aggregation completed but 47 accounts that exist in ISC no longer correlate to identities after today's aggregation"

**When did it start?** (Narrow the change window)
- First occurrence: today's 3 AM aggregation
- What changed in that window: ISC was updated on Sunday; a new AD OU was added on Sunday

**Who is affected?** (Quantify scope)
- Is it every user or specific ones?
- Is it all sources/targets or one specific connector?

**What is the expected behavior?** (Define success)
- Access request should complete in under 30 minutes
- Aggregation should complete without errors
- 9,600 accounts should correlate; only 9,553 did

Spending 5 minutes answering these questions precisely saves hours of mis-directed investigation.

> 🎯 **KEY POINT:** The more specific the symptom description, the faster the diagnosis. "AD isn't working" requires you to check every possible thing. "AD aggregation completes but `memberOf` returns only the first group for each account" tells you exactly where to look: multi-value attribute configuration in the schema.

---

### Step 2: Isolate the Component

Integration problems occur in one of four components. Identify which one before investigating further:

```
[Source System] → [Connector/VA] → [ISC] → [Target System]
       A                B             C           D
```

**Component A (Source System)**: Data quality problems, API changes in the source, authentication issues when ISC reads the source, schema changes.

**Component B (Connector/VA)**: Network connectivity failures, VA health issues, connector version bugs, timeout problems.

**Component C (ISC)**: Correlation rule errors, transform failures, workflow configuration issues, lifecycle state rule problems, access profile misconfigurations.

**Component D (Target System)**: Provisioning permission issues, target API changes, target system downtime, target data validation rejecting ISC's writes.

**Isolation test**: Can you reproduce the problem in isolation?
- Can you authenticate to the source manually? (Tests A connectivity)
- Does the VA health show Active? (Tests B)
- Does a manual ISC aggregation complete or fail? (Tests B + C)
- Can the ISC service account authenticate to the target directly? (Tests D connectivity)

Each of these tests either confirms or eliminates a component.

---

### Step 3: Check the Logs

Once you've isolated the component, go to the right log.

**For source/aggregation problems** → ISC Aggregation Activity log:
- Admin → Sources → [Source Name] → Aggregation History
- Look for the specific aggregation run that exhibited the problem
- Check: `totalAccounts` vs expected, `errors` count, `uncorrelated` count
- Download the error file if `errors > 0`

**For VA/connector problems** → VA log:
- `/home/sailpoint/log/ccg.log` on the VA host
- Filter by timestamp of the failed operation
- Look for: `ERROR`, `WARN`, `Exception`, `timeout`, `ConnectorException`

**For provisioning problems** → ISC Provisioning Activity:
- Admin → Provisioning Activity
- Find the specific failed operation
- View the provisioning plan (what ISC attempted) and the error details

**For workflow/lifecycle problems** → ISC Workflow Execution log:
- Admin → Workflows → [Workflow Name] → Execution History
- Find the execution that failed or didn't trigger
- Check: which step failed, what the step inputs were, what error was returned

**For ISC rule errors** → Connector Debug log:
- Admin → Sources → [Source] → Accounts → Connector Debug
- Shows rule execution details and BeanShell exceptions

> 💡 **REMEMBER:** Logs have timestamps in UTC. When comparing ISC logs to target system logs, convert to the same timezone. More than one troubleshooting session has been derailed by a 5-hour timestamp offset that made cause and effect appear reversed.

---

### Step 4: Form a Hypothesis

After reviewing logs, you should be able to state a specific hypothesis: "I believe X is happening because the log shows Y."

Examples of well-formed hypotheses:
- "The correlation rule is returning null for contractor accounts because the BeanShell rule doesn't handle the `GT-` prefix format that contractor Worker_IDs use in this Workday tenant"
- "The provisioning failure is because the Salesforce service account's OAuth token expired on Sunday when ISC was updated, and the token refresh failed due to a scope mismatch"
- "The 47 uncorrelated accounts stopped correlating because a new AD aggregation filter was added on Sunday that excludes the `OU=Contractors` OU where those accounts live"

A specific hypothesis tells you exactly what to test to confirm or deny it.

**Red flags that mean you need more information**:
- "I'm not sure what the log is telling me" → re-read with fresh context or seek a second opinion
- "Multiple things could explain this" → rank them by likelihood and test in order
- "I've never seen this before" → search SailPoint Community and documentation before testing

---

### Step 5: Test the Hypothesis

Test hypotheses in ways that are reversible and don't affect production users.

**For correlation rule hypotheses**:
- Use ISC's correlation preview mode — run correlation without committing changes
- Test specific account records manually against the rule logic
- Verify the attribute values on the account match what the rule expects

**For provisioning hypotheses**:
- Use ISC's provisioning plan preview — build the plan without executing
- Review what attributes and entitlements would be included
- Test with a non-production identity or a sandbox ISC tenant

**For connectivity hypotheses**:
- Test the source connection in ISC (Source → Test Connection)
- For AD/LDAP: use `ldapsearch` from the VA host to verify connectivity independently of ISC
- For REST APIs: use curl or Postman with the same credentials ISC uses

**For workflow/rule hypotheses**:
- Test in ISC sandbox if available
- Trigger with a test identity rather than a production one
- Review the rule logic manually for the specific edge case

**Document what you tested and the result**. Even negative results (hypothesis disproved) are useful — they eliminate possibilities and narrow the remaining options.

---

### Step 6: Implement the Fix and Verify

Once the root cause is confirmed, implement the fix in the right order:

1. **Fix in sandbox first** (if available and the fix is configuration-level)
2. **Get peer review** for any rule change or significant configuration change
3. **Implement in production with a change management record** (for changes that affect live users)
4. **Verify the fix resolved the problem** — don't assume; confirm:
   - Run the aggregation and check the previously failing metric
   - Trigger the previously failing provisioning and confirm success
   - Check that the fix didn't introduce new problems in adjacent areas
5. **Monitor for 24–48 hours** — some fixes expose secondary issues that weren't visible before

---

## Decision Tree: Common Problem Patterns

```
Problem reported
      │
      ▼
Is the VA showing Disconnected?
  ├─ YES → Fix VA connectivity first (network, restart VA service)
  │
  └─ NO ▼
      │
Did aggregation run/complete?
  ├─ NO → Check VA health, connector debug log, authentication
  │
  └─ YES ▼
      │
Is the account count wrong (fewer than expected)?
  ├─ YES → Check aggregation scope (search base, object filter, OU changes)
  │
  └─ NO ▼
      │
Are accounts uncorrelated (more than expected)?
  ├─ YES → Export uncorrelated list → categorize → fix rules or data
  │
  └─ NO ▼
      │
Is provisioning failing?
  ├─ YES → Check provisioning activity → identify error type → use failure patterns
  │
  └─ NO ▼
      │
Are attributes wrong on identities?
  ├─ YES → Check transform chain → check source attribute value → check attribute authority
  │
  └─ NO ▼
      │
Is a lifecycle action not firing?
  ├─ YES → Check lifecycle state rule → check identity attribute values → check action config
  │
  └─ NO → Escalate with detailed symptom description and log evidence
```

---

## A Complete Troubleshooting Example

**Symptom reported** (Monday 9 AM): "New hires who started today still don't have Salesforce access. We checked — their managers requested it last week."

**Step 1 — Precise symptom**: Access requests for Salesforce submitted 5+ business days ago are still in Pending state. Normal completion time is under 4 hours. Affects all new hires this week (8 people). Longer-tenured employees' access requests are completing normally.

**Step 2 — Isolate**: Problem is in provisioning (D) or workflow (C). VA health shows Active. Aggregation ran successfully this morning. Problem is specific to the Salesforce target.

**Step 3 — Check logs**: ISC Provisioning Activity → filter by Salesforce → filter by Pending. Eight items, all pending since last week. Click one: the provisioning plan was built correctly, but the execution shows "Pending Manual Review." The item is sitting in the Provisioning Review queue, not failed — waiting for an administrator to action it.

**Step 4 — Hypothesis**: A change to the Salesforce provisioning policy or workflow last week set new Salesforce provisions to require manual approval before execution.

**Step 5 — Test**: Review the Salesforce source configuration → Provisioning Policy → check plan settings. Confirm: "Require Admin Approval" was checked during a configuration change on the previous Thursday (noted in ISC audit log).

**Step 6 — Fix**: Uncheck "Require Admin Approval" (it was set accidentally during a different change). Action all eight pending items in the Provisioning Review queue — they execute successfully within 10 minutes. Verify all eight new hires now have Salesforce access. Add a configuration change review step in the change management process.

Total diagnosis time with this methodology: 25 minutes.

---

## Key Takeaways

- Precise symptom description (who, what, when, expected vs actual) halves diagnostic time
- Isolate the component (Source → Connector/VA → ISC → Target) before going to logs
- The right log for the right component: aggregation activity, VA ccg.log, provisioning activity, workflow execution
- Form a specific hypothesis from log evidence before testing anything
- Test hypotheses in reversible, non-production ways first
- Verify the fix resolved the problem and monitor for 24–48 hours afterward

> 🔗 **CONNECTS TO:** Systematic troubleshooting finds problems. Systematic testing prevents them. The next document covers how to build a testing framework that catches integration problems before they reach production.
