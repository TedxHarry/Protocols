# Decision Guide — Choosing Your Integration Approach

The previous documents covered how to connect individual systems to ISC. This document steps back to the architecture level: when you're designing how ISC fits into your organization's overall identity ecosystem, which integration pattern makes sense — and why?

Three fundamental integration patterns appear in enterprise ISC deployments. Most organizations end up using all three simultaneously, each for the systems it fits best. Understanding the patterns, their strengths, and their failure modes lets you make these architectural decisions confidently rather than defaulting to whatever you did last time.

---

## Pattern 1: Hub-and-Spoke (ISC as Central Authority)

**How it works**: ISC is the hub. Every system in scope connects directly to ISC. ISC makes all identity and access governance decisions and communicates them outward to connected systems. Nothing talks to anything else — all paths go through ISC.

```
HR System ──────────────────────────────────────────────┐
                                                         ▼
AD ──────────────────────── ISC (Hub) ─────────────── Salesforce
                              ▲   │
GitHub ─────────────────────┘    └──────────────── AWS
```

**When to use hub-and-spoke**:
- Organization has many heterogeneous systems (10+) that all need governed access
- Governance priority is high: centralized visibility, unified audit trail, consistent policy enforcement
- Slight access provisioning latency is acceptable (minutes to hours, not seconds)
- SailPoint connectors exist for most target systems

**GlobalTech example**: 35 of GlobalTech's 47 systems are connected in a hub-and-spoke pattern. Workday feeds employee data in. ISC governs all 35 target systems outward. Every provisioning event, every certification, every policy violation goes through ISC. One place to look for governance evidence.

**Strengths**:
- Single source of governance truth: all access decisions made in one place
- Unified policy enforcement: SoD rules apply across all 35 systems simultaneously because ISC sees all of them
- Centralized audit trail: one report answers "who has access to what" across all systems
- Simplified compliance: auditors query one system rather than 35
- ISC's AI capabilities (peer analysis, anomaly detection) require seeing the full access picture — hub-and-spoke gives ISC the data it needs

**Weaknesses**:
- ISC becomes a critical single point of failure. If ISC is unavailable, provisioning stops. If the Virtual Appliance goes down, on-premises systems lose their governance connection. High-availability ISC configurations mitigate this but require careful design.
- Complex to maintain at very large scale (100+ connected systems, millions of identities). Not impossible — SailPoint's multi-VA architecture handles it — but operational complexity grows with the number of connections.
- Every new system requires an ISC connector. If a new system doesn't have a pre-built connector, you need to build one or accept a flat file fallback.

> 🎯 **KEY POINT:** Hub-and-spoke is the default pattern for ISC implementations. It's what ISC is designed for. If a system can connect to ISC via a supported connector, it should. The other patterns exist for specific cases that hub-and-spoke doesn't handle well.

---

## Pattern 2: Event-Driven Integration

**How it works**: Instead of ISC polling systems on a schedule, systems push events to ISC when changes occur. ISC responds to events in real time (or near-real-time).

```
Workday ──[termination event]──► ISC ──[immediate deprovisioning]──► All systems
Okta ────[failed login alert]──► ISC ──[access review trigger]─────► Security team
HR ──────[new hire event]──────► ISC ──[day-1 provisioning]────────► AD + SaaS apps
```

Events can flow in both directions: systems notify ISC of changes (inbound events), and ISC notifies systems of governance decisions (outbound events/webhooks).

**When to use event-driven**:
- Time-sensitive operations: terminations for privileged users, security incidents, real-time access control
- Modern SaaS systems that support webhooks or message queues (Workday, Okta, Slack, many others)
- Compliance requirements that mandate specific response times to access events
- When scheduled aggregation latency (hours) creates unacceptable security risk windows

**GlobalTech example**: GlobalTech uses event-driven integration for three specific flows:
1. **Workday termination webhook** → ISC receives within 60 seconds → deprovisioning triggers immediately → all 40 connected systems disabled within 15 minutes
2. **Okta MFA failure events** → ISC receives repeated failure alerts → ISC triggers access review for the identity → security team notified
3. **New hire start date trigger** → Workday sends event day before start date → ISC provisions all accounts overnight → employee has everything on day 1

**Implementation in ISC**:
- **Inbound events** (system → ISC): ISC exposes API endpoints that external systems can call. Systems send event payloads to ISC's REST API. ISC's workflow engine processes events and triggers appropriate actions.
- **Outbound events** (ISC → systems): ISC workflow sends webhooks or API calls to external systems when governance events occur (provisioning complete notification, certification reminder, violation detected).
- **Virtual Appliance event listeners**: For on-premises systems that generate events, the VA can listen and forward to ISC cloud.

**Strengths**:
- Real-time governance response — seconds or minutes rather than hours
- Eliminates access windows that scheduled aggregation creates (the 24-hour termination window)
- Modern architecture: event-driven systems are designed for this pattern (Workday Integration Studio, Okta Events API, Slack Events API)
- ISC workflows can be highly responsive — trigger actions in real time based on specific conditions

**Weaknesses**:
- More complex to build than scheduled aggregation — requires event infrastructure, error handling, retry logic, and idempotency (same event shouldn't process twice)
- Event delivery is not always reliable — networks drop packets, systems have outages. Event-driven integrations need dead-letter queues and monitoring for missed events.
- Debugging is harder — scheduled aggregation runs are discrete and auditable; event-driven failures can be subtle (event sent but not received, event received but not processed, processed incorrectly)
- Not all systems support event emission — legacy systems rarely do

> ⚠️ **COMMON MISTAKE:** Assuming event-driven is always better because it's faster. For most governance operations (standard attribute updates, role changes, access requests), scheduled aggregation is perfectly adequate and much simpler. Use event-driven specifically where latency matters. Terminations for privileged users: event-driven. Weekly role reports: scheduled batch. Match the complexity to the actual requirement.

---

## Pattern 3: Flat File / Scheduled Batch

**How it works**: Source systems generate files (CSV, delimited text, fixed-width) on a schedule. Files land on an agreed location (SFTP, file share). ISC's flat file connector picks them up and processes them as aggregation sources.

```
Legacy ERP ──[nightly CSV at 2 AM]──► SFTP Server ──[3 AM pickup]──► ISC aggregation
Badge System ─[weekly extract]──────► File Share ──[Sunday pickup]──► ISC audit records
Payroll ──────[weekly payroll file]──► SFTP ────────[Monday pickup]──► ISC attribute update
```

**When to use flat file**:
- Systems that have no REST API, LDAP interface, or supported ISC connector
- Legacy systems behind strict network perimeters where direct connectivity isn't possible
- Systems in regulated environments where controlled batch processing is required for compliance
- Acquired company systems during transition periods before migration to supported platforms
- Physical systems (badge access, facilities management) that output files as their only integration method

**GlobalTech example**: Three systems use flat file:
1. **Legacy payroll (2009 vintage)**: generates a 12,000-row CSV every night at 2 AM with employee ID, cost center, pay grade. ISC reads it at 3 AM. Provides cost center and pay grade attributes for access policies.
2. **Physical badge system**: exports weekly access log CSV. ISC reads it for physical access governance reporting.
3. **Benefits administration system**: quarterly export of benefit election data used to determine insurance-related system access.

**Making flat file integrations robust**:

File-based integrations are simple in concept but break in specific ways:
- File not delivered (upstream system failure, SFTP credential expiration) — ISC should alert when expected file doesn't arrive
- File format change (source system updated their extract without telling anyone) — ISC connector errors on column count or data type mismatch
- Partial file (extract cut off mid-run) — row count validation catches this
- Encoding issues (source system switched to UTF-16 from UTF-8) — character encoding mismatch in field values

Production flat file integrations need:
1. **Arrival monitoring**: alert if expected file doesn't arrive within X minutes of expected time
2. **Row count validation**: compare row count against previous file and against expected employee count. Flag if >5% variance.
3. **Checksum verification**: if source system provides a checksum file, validate before processing
4. **Error file review**: ISC generates an error file for rows that couldn't be processed. Review after every run.
5. **Runbook documentation**: what to do when the file doesn't arrive, when row counts are off, when ISC can't parse the file

> 💡 **REMEMBER:** Flat file integrations break silently if not monitored. If the payroll system stops generating the file and nobody set up arrival monitoring, ISC simply doesn't update those attributes — no error, no alert, stale data. Design monitoring into flat file integrations from day one.

---

## Decision Framework: Which Pattern for Which System?

| Factor | Hub-and-Spoke | Event-Driven | Flat File |
|---|---|---|---|
| System has ISC connector | Required | Optional | Not needed |
| System supports API/webhooks | Needed | Required | Not needed |
| Acceptable latency | Minutes–hours | Seconds–minutes | Hours–days |
| Complexity to implement | Medium | High | Low |
| Operational complexity | Medium | High | Low |
| Good for source of truth data | Yes | Yes | With caveats |
| Good for provisioning targets | Yes | Yes | No (read-only) |
| Good for legacy systems | Sometimes | Rarely | Yes |

**Three questions to determine pattern**:

1. **Does this system have an ISC-supported connector?**
   - Yes → Hub-and-spoke is your starting point
   - No → Flat file or custom connector (consider build cost vs. value)

2. **Does the timing of changes from this system matter for security?**
   - Terminations from this system → Must be event-driven (or very frequent delta)
   - Standard employee data → Scheduled aggregation is fine
   - Real-time access control decisions → Event-driven

3. **Is this system a source or a target? And if a target, does it support provisioning?**
   - Source with API → Hub-and-spoke (scheduled aggregation)
   - Source with events → Event-driven
   - Source without API → Flat file (read-only)
   - Target with connector → Hub-and-spoke (provisioning)
   - Target without connector → Flat file or service desk ticketing (no direct provisioning)

---

## The Reality: All Three Patterns Coexist

GlobalTech's ISC architecture uses all three patterns simultaneously:

| Pattern | Systems | Count |
|---|---|---|
| Hub-and-spoke | AD, Salesforce, GitHub, AWS, Jira, Slack, ServiceNow, + 28 more | 35 |
| Event-driven | Workday (terminations + new hire), Okta (MFA failures) | 3 flows |
| Flat file | Legacy payroll, badge system, benefits admin | 3 |

This is representative. Enterprise environments are heterogeneous — different systems have different capabilities, different risk profiles, and different latency requirements. The integration architect's job is matching each system to the right pattern based on what that system can do and what governance requires from it.

Forcing one pattern on all systems creates problems: applying flat file batch to terminations creates security risk; applying event-driven architecture to a legacy payroll system that can't support events is impossible; skipping hub-and-spoke for modern SaaS apps misses governance coverage.

The answer is never "one pattern for everything." The answer is "the right pattern for each system, assembled into a coherent overall architecture."

---

## Key Takeaways

- Three integration patterns: Hub-and-spoke (ISC governs all connected systems centrally), Event-driven (systems push events to ISC for real-time response), Flat file (file-based for legacy systems without API support)
- Hub-and-spoke is the default for most systems — it's what ISC is designed for and what the connector library enables
- Event-driven is specifically for latency-sensitive operations: terminations of privileged users, security incident response, real-time access control
- Flat file is for legacy systems that can't support API connectivity — add monitoring and validation to make it reliable
- Real implementations use all three patterns simultaneously — match the pattern to the system's capabilities and the governance requirement's latency needs

> 🔗 **CONNECTS TO:** Phase 3 is complete. You understand the role, the decision framework, the failure modes, and the architectural patterns. Phase 4 begins the technical deep dive into ISC connector development — how to build what you've been designing.
