# Lifecycle States — Modeling the Identity Journey Through ISC

Every identity has a lifecycle. People join organizations, change roles, go on leave, return, transfer between divisions, and eventually leave. Access should change at each transition — and it should change automatically, based on data from authoritative sources, not based on a ticket that someone might forget to file.

Lifecycle states in ISC are the mechanism that makes automatic, policy-driven access transitions possible. Getting the lifecycle model right is the difference between ISC that actively governs access and ISC that's a read-only reporting tool that people file tickets around.

---

## What a Lifecycle State Is

A lifecycle state is a named status that an identity holds at a given point in time. ISC evaluates lifecycle state rules continuously and changes an identity's state when the conditions change. Each state transition can trigger provisioning and deprovisioning actions.

ISC ships with a standard set of lifecycle states that cover the common cases. Organizations can extend these with custom states. The standard states:

| State | Meaning | Typical Access |
|---|---|---|
| `Pre-hire` | Hired but not yet started | Limited: maybe email, onboarding portal |
| `Active` | Currently employed, working | Full role-based access |
| `Leave of Absence` | Approved leave (medical, parental, etc.) | Suspended: accounts disabled but preserved |
| `Inactive` | No longer active, not formally terminated | Minimal or none |
| `Terminated` | Employment ended | Revoked: accounts disabled/deleted per policy |

---

## How ISC Determines Lifecycle State

ISC calculates lifecycle state from identity attributes — primarily data from your authoritative HR source (Workday, SAP SuccessFactors, etc.). The lifecycle state rules are configured in the Identity Profile.

**Common state rules for GlobalTech**:

```
Pre-hire:
  startDate > today AND terminationDate is null

Active:
  startDate <= today AND terminationDate is null AND leaveStatus = "Not on Leave"

Leave of Absence:
  leaveStatus = "On Leave" AND terminationDate is null

Terminated:
  terminationDate is not null AND terminationDate <= today
```

ISC evaluates these rules every time aggregation updates an identity's attributes. Attribute changes from Workday → aggregation runs → ISC re-evaluates lifecycle state → if state changes, configured actions execute.

**The termination transition** is the most governance-critical state change:
1. Workday marks employee as terminated (sets `terminationDate` to today or a past date)
2. ISC aggregation runs (scheduled delta or event-driven webhook)
3. ISC reads new `terminationDate` attribute
4. Lifecycle state rule evaluates: terminationDate ≤ today → state changes to `Terminated`
5. Terminated lifecycle state action fires → deprovisioning workflow executes → all accounts disabled

The latency of step 2 (aggregation) is what determines how quickly deprovisioning happens. This is why event-driven webhook integration for terminations is so significant — it collapses the latency from hours to minutes.

> 🎯 **KEY POINT:** The lifecycle state model is only as good as the data driving it. If Workday has a termination date set incorrectly, ISC will make access decisions based on incorrect data. If the "leave of absence" field uses inconsistent values across HR regions, the LOA state won't fire reliably. Clean authoritative source data is a prerequisite for reliable lifecycle management.

---

## Configuring State Actions

Each lifecycle state transition can trigger one or more actions. Actions are configured per identity profile and per target source.

**Common action types**:

**Enable account**: When transitioning from Pre-hire to Active, enable the AD account. When returning from LOA to Active, re-enable.

**Disable account**: When entering Leave of Absence or Terminated state, disable accounts in all managed systems. Disabled means: the account still exists but cannot be used to authenticate or access the system.

**Delete account**: After a retention period post-termination, delete accounts from systems where retention policy allows. Many organizations disable on termination and delete 90 days later after any data transfer/handoff is complete.

**Remove entitlements**: Revoke specific entitlements without disabling the account. Used for situations where a role change (not a termination) means specific elevated access should be removed.

**Trigger a workflow**: Launch an ISC workflow to handle more complex actions — notify a manager, create a ticket in ServiceNow, send access transfer instructions.

**GlobalTech's lifecycle action matrix**:

| Transition | AD | Workday | Salesforce | GitHub | AWS |
|---|---|---|---|---|---|
| Pre-hire → Active | Enable, provision groups | Source only | Create account | Create account | Create account |
| Active → LOA | Disable | Source only | Disable | Disable | Disable |
| LOA → Active | Enable | Source only | Enable | Enable | Enable |
| Active → Terminated | Disable | Source only | Disable | Remove, disable | Disable |
| 90 days post-termination | Delete | Source only | Delete | Delete | Delete |

---

## Pre-Hire State: Day-One Readiness

Pre-hire is a lifecycle state that many ISC implementations get wrong or skip entirely. The common failure: a new employee's first day involves 3 hours waiting for IT to create accounts that could have been provisioned the previous week.

Pre-hire state, configured correctly:
- Workday creates the employee record when hire is confirmed (typically 2+ weeks before start date)
- ISC aggregation picks up the new record; `startDate` is in the future → Pre-hire state
- Pre-hire state action provisions a limited set of accounts: email account (so the employee can be communicated with pre-start), HRIS self-service portal access, onboarding documentation portal
- On the morning of `startDate`, lifecycle state re-evaluates → transitions to Active → full provisioning fires overnight or at the scheduled window

**Result**: The employee's first morning, everything is ready. Their laptop has a working login. Their email is active. GitHub, Jira, and Slack are all provisioned.

> 💡 **REMEMBER:** Pre-hire provisioning requires careful scoping. The pre-hire employee is not yet an employee — they haven't signed NDAs, completed background checks, or started orientation. Pre-hire access should be limited to: what they need to complete pre-start paperwork and nothing more. Don't provision full system access before day one.

---

## Leave of Absence: The Underdesigned State

Leave of absence is the lifecycle state that most implementations under-design. The two failure modes:

**Failure mode 1: No LOA state**. Employee goes on parental leave. Nothing changes in ISC. Their accounts remain active for 16 weeks. If they decide not to return, deprovisioning happens eventually — but they had 16 weeks of active credentials they weren't using. This is a governance gap and, for privileged users, a security risk.

**Failure mode 2: LOA treated as Termination**. Employee goes on parental leave. ISC disables all accounts immediately. When they return, their access has been revoked and role assignments deleted. Reprovisioning takes a week. Terrible employee experience, HR escalation, and completely unnecessary.

The correct design:
- LOA state: all accounts disabled (can't authenticate), but not deleted and role assignments preserved
- Return from LOA: accounts re-enabled, role assignments remain, no reprovisioning needed
- If LOA converts to termination: transition directly from LOA → Terminated, deprovisioning fires

**What Workday provides for LOA**: Most modern HRMS systems have a leave status attribute that ISC can read. `leaveStatus = "On Leave"` triggers the LOA lifecycle state. `leaveStatus = "Returned from Leave"` triggers the return to Active. The challenge: HRMS systems often have multiple leave statuses (medical, parental, personal, military) with different values. Build your lifecycle rule to handle all valid leave values, not just one.

---

## Custom Lifecycle States

Standard states don't cover every scenario. ISC allows custom lifecycle states for organization-specific needs.

**Common custom states at GlobalTech**:

`Contractor-Active`: Contractors are Active in the technical sense, but they have a different access scope than employees. A separate lifecycle state lets GlobalTech configure different provisioning actions (no VPN, no access to HR systems, time-limited account expiration).

`Pending Offboard`: For organizations with a multi-day offboarding process (data transfer, exit interview, equipment return), a Pending Offboard state can reduce access to read-only before full termination.

`Internal Transfer`: When an employee transfers between divisions (e.g., from Engineering to Product), a brief Transfer state allows old access to be revoked and new access to be provisioned in a controlled sequence rather than simultaneously.

> ⚠️ **COMMON MISTAKE:** Too many lifecycle states. Each custom state needs maintained configuration, tested transitions, and documented actions. More than 8–10 states in a single identity profile creates a state machine that's difficult to understand and error-prone to maintain. Add custom states for genuine business distinctions; don't add them for edge cases that can be handled with attribute-based exceptions.

---

## Testing Lifecycle Transitions

Lifecycle state transitions must be tested before go-live. The test for each transition:

1. Create a test identity with attributes that put it in State A
2. Verify ISC assigns the correct lifecycle state
3. Verify the correct accounts are provisioned (or in the correct state)
4. Change the test identity's attributes to trigger the transition to State B
5. Run aggregation (or trigger manually)
6. Verify ISC detects the state change
7. Verify the configured actions execute correctly (accounts disabled, etc.)
8. Verify the target systems reflect the change

Test all transitions in order. The sequence Pre-hire → Active → LOA → Active → Terminated should be exercised with a test identity before any production identity goes through it.

---

## Key Takeaways

- Lifecycle states are named identity statuses that drive automatic provisioning and deprovisioning — when attributes change, states change, and actions execute
- State rules derive from HR source attributes (startDate, terminationDate, leaveStatus); data quality in the source is foundational
- Pre-hire state enables day-one readiness — provision limited access before start date so employees are productive from hour one
- Leave of Absence should disable accounts (not delete them) and preserve role assignments — return from LOA should restore access without reprovisioning
- Custom lifecycle states serve genuine business distinctions; resist adding states for edge cases that attributes can handle
- Test every state transition with a test identity before production — lifecycle failures affect real people's access in real time

> 🔗 **CONNECTS TO:** Lifecycle states determine when access changes. Provisioning policies determine how ISC creates accounts when it decides to provision — what attribute values to set, what templates to follow, and how to handle the edge cases that always come up. The next document covers provisioning policies in depth.
