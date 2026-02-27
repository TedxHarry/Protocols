# Event-Driven Integration — Real-Time Governance Response

Scheduled aggregation runs every 4 hours. An employee was terminated 3 hours and 59 minutes ago. Their credentials are still active.

That gap is not a technical limitation — it's an architecture choice. Batch aggregation is simpler to implement, but it creates access windows between when something changes in the real world and when ISC responds. For most governance operations, those windows are acceptable. For terminations of privileged users, security incidents, and real-time access control decisions, they are not.

Event-driven integration eliminates the gap. Instead of ISC asking "what changed?" every few hours, the source system tells ISC "something just changed" the moment it happens. ISC responds immediately.

---

## What Event-Driven Integration Actually Is

Event-driven integration is a pattern where systems emit events when state changes occur, and ISC acts as either:

**An event consumer** (inbound events): External systems send events to ISC when changes happen. Workday sends a termination event → ISC receives it → deprovisioning fires within seconds.

**An event producer** (outbound events): ISC sends notifications or calls external systems when governance events occur. Access request approved → ISC calls ServiceNow to create an account → ServiceNow sends confirmation → ISC marks provisioning complete.

Both directions exist in enterprise implementations. Most event-driven ISC work focuses on inbound events from authoritative sources.

---

## Inbound Events: How External Systems Notify ISC

**Webhooks**: The most common pattern. When a specified event occurs in the source system, it makes an HTTP POST call to a URL you configure — ISC's event endpoint. ISC receives the payload, parses it, and triggers configured actions.

Workday Integration Studio can be configured to call ISC's REST API when specific Workday events occur:
- Employee status changes to "Terminated" → Workday calls ISC
- Employee hired (future start date set) → Workday calls ISC
- Employee transfers to new manager → Workday calls ISC

The ISC endpoint that receives these calls: `https://[tenant].api.identitynow.com/beta/event-triggers/subscriptions/[subscription-id]/trigger`

**Message queues**: For higher-volume event streams or more reliable delivery guarantees, organizations route events through a message queue (AWS SQS, Azure Service Bus, Apache Kafka) before ISC. The queue handles retry logic, ordering, and backpressure. ISC polls the queue or subscribes to it via connector. This pattern is common in financial services where event delivery guarantees are non-negotiable.

**ISC Trigger Subscriptions**: ISC's event trigger framework allows ISC itself to subscribe to external event sources. ISC polls or subscribes to the source, ISC processes events, ISC executes workflows. The configuration is entirely within ISC — no external system needs to be configured to call ISC.

**Push vs Pull**: Webhooks are push (source calls ISC). ISC trigger subscriptions can be pull (ISC reads from a queue). Push is simpler and has lower latency. Pull provides more reliable delivery in high-volume scenarios.

---

## Outbound Events: ISC Notifying External Systems

ISC can call external systems via workflow steps when governance events occur:

**Workflow HTTP Action**: A workflow step that makes an HTTP call to any REST endpoint. Use cases:
- Access request approved → POST to ServiceNow API to create the account
- Policy violation detected → POST to Slack channel to alert the security team
- Termination initiated → POST to Jira to create an offboarding checklist ticket

**Notification actions**: ISC workflow can send email, Slack messages (via Slack connector or webhook), or Teams messages when specific governance events occur.

**Outbound use case at GlobalTech**: When ISC detects a SOD policy violation, a workflow fires that:
1. Creates a ServiceNow incident ticket with violation details
2. Sends a Slack message to the `#identity-security` channel
3. Emails the violating identity's manager with remediation instructions

The security team doesn't monitor ISC — they monitor ServiceNow and Slack. ISC pushes violations into those channels automatically.

---

## Real-Time Termination: The Primary Use Case

The killer use case for event-driven integration is high-risk termination. Phase 3.2 covered the termination latency decision; this document shows the implementation.

**GlobalTech's real-time termination architecture**:

```
HR (Workday)
   │ Employee terminated in Workday
   │ Workday Integration Studio configured to fire on status = "Terminated"
   ▼
Workday sends HTTP POST to ISC event endpoint (< 60 seconds after Workday update)
   │
   ▼
ISC receives event, parses employee ID from payload
ISC finds matching identity, updates lifecycle state to "Terminated"
   │
   ▼
ISC lifecycle termination workflow fires:
   ├── Disable AD account (VA → AD domain controller)
   ├── Disable Salesforce account (direct API)
   ├── Remove GitHub access (direct API)
   ├── Disable AWS IAM access (direct API)
   ├── Revoke all CyberArk privileged sessions (CyberArk integration)
   └── Create ServiceNow offboarding ticket
   │
   ▼
All accounts disabled within 8–12 minutes of Workday update
```

**Total elapsed time**: Under 15 minutes from HR action to all access disabled. The previous batch-aggregation approach was up to 24 hours.

**What you configure to make this work**:

In Workday (configured by Workday admin):
- Create an Integration System (Workday Integration Studio)
- Configure it to monitor for `Worker_Status_Change` events where new status = `Terminated`
- Configure it to call ISC's event endpoint with the worker's ID and new status

In ISC:
- Event trigger subscription configured to receive the Workday payload
- Workflow mapped to the trigger: receive event → find identity by employee ID → execute termination lifecycle actions

**The Workday payload** (simplified):
```json
{
  "event": "Worker_Terminated",
  "workerId": "GT-48291",
  "effectiveDate": "2025-11-15",
  "terminationType": "Voluntary"
}
```

ISC's event trigger maps `workerId` to the identity correlation attribute, finds Sarah Chen, and fires the termination workflow.

> ⚠️ **COMMON MISTAKE:** No idempotency handling. If Workday sends the same termination event twice (network retry, duplicate trigger), ISC should not process it twice. ISC workflow steps are generally idempotent (disabling an already-disabled account is a no-op), but custom rule logic triggered by events may not be. Verify that reprocessing the same event twice produces the same result, not doubled actions.

---

## New Hire Pre-Provisioning: The Second Major Use Case

Real-time new hire provisioning ensures Day 1 readiness. The event flow:

**Event trigger**: Workday creates a new employee record with a future start date. Integration Studio sends an event to ISC at the time of record creation (2–4 weeks before start date, typically).

**ISC action**: Receives the new hire event. Creates the ISC identity in `Pre-hire` lifecycle state. Provisions limited pre-hire access (email account, onboarding portal). Schedules a lifecycle state change to `Active` on the start date.

**Start date transition**: At midnight on the start date, ISC lifecycle evaluation transitions the identity from `Pre-hire` to `Active`. Full role-based access provisions overnight. The new employee arrives on their first day to fully provisioned accounts.

**Edge cases to design for**:
- Start date changes (employee pushes start by 2 weeks) — ISC receives an update event, adjusts the scheduled transition
- Onboarding cancellation (offer rescinded) — ISC receives cancellation event, removes pre-hire access
- Start date in the past (backdated record) — handle gracefully; don't provision pre-hire access retroactively

---

## ISC Event Triggers Framework

ISC's event trigger framework (available in the ISC developer portal) allows you to configure ISC to react to specific events it internally detects:

| Trigger Type | When It Fires |
|---|---|
| Identity Created | New identity appears in ISC |
| Identity Updated | Identity attributes change |
| Identity Deleted | Identity removed from ISC |
| Access Request Submitted | Access request created |
| Access Request Pre-Approved | Automated approval condition met |
| Provisioning Completed | Provisioning operation finishes |
| Certification Item Completed | Reviewer makes a decision |
| Policy Violation | SoD or policy violation detected |

Each trigger can invoke a workflow — ISC's visual workflow builder creates the action chain.

**Developer documentation**: The complete event trigger schema and API documentation is at `developer.sailpoint.com/isc/api/`. Each trigger type has a defined payload schema — build your workflows to match the documented payload structure.

> 💡 **REMEMBER:** ISC triggers are ISC-internal events that workflows can react to. External webhooks (Workday sending events to ISC) are configured differently — they use ISC's inbound REST API endpoint, not the trigger subscription framework. The two mechanisms work together: external webhook → ISC inbound event → triggers ISC identity update → ISC-internal trigger fires → workflow executes.

---

## Reliability and Error Handling

Event-driven integrations fail in ways batch integrations don't:

**Event not delivered**: Workday's webhook call to ISC times out or receives an error. Workday may retry (if configured), or the event is lost. Solution: configure retry on the sending side; add a monitoring check that verifies termination events arrived (compare Workday termination count to ISC lifecycle changes in the same window).

**Event delivered but not processed**: ISC received the event but the workflow failed. Check ISC workflow execution logs after known events. For high-stakes events (terminations), configure a dead-letter alert: if the workflow fails, alert the security team immediately.

**Duplicate events**: Source system sends the same event twice. Workflow runs twice. Solution: ISC should log a unique event ID for each event; if the same event ID is received again, skip processing.

**Order-of-events problems**: Update event arrives before create event for a new hire. ISC tries to update an identity that doesn't exist yet. Solution: add a retry with backoff in the workflow — if the identity isn't found, wait 30 seconds and try again (up to 3 times).

> 🎯 **KEY POINT:** Event-driven integrations need explicit failure monitoring. Batch aggregation failures are visible in the aggregation log every cycle. Event-driven failures can be silent — an event that wasn't delivered and wasn't retried leaves no trace. Build monitoring: alert when expected events don't arrive within a time window after known source system actions.

---

## Key Takeaways

- Event-driven integration eliminates the latency gap between real-world identity changes and ISC governance response
- Inbound webhooks (source → ISC) and ISC trigger framework (ISC-internal events → workflow) are the two primary mechanisms
- Real-time termination is the primary business case: Workday termination event → ISC → all systems disabled in under 15 minutes
- New hire pre-provisioning creates Day 1 readiness by provisioning accounts before the start date
- Event-driven integrations require explicit reliability design: retry logic, idempotency, dead-letter alerting, and delivery monitoring
- Use event-driven for time-sensitive governance (terminations, security incidents, real-time access control); use batch for routine attribute sync

> 🔗 **CONNECTS TO:** Event-driven integrations use pre-built connectors and ISC's webhook infrastructure. Some systems can't be connected with any pre-built mechanism. The next document covers when building a custom connector makes sense — and how to make that decision correctly.
