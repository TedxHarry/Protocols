# Section 7.4.3: Policy Violations and Alerts

**SailPoint Identity Security Cloud (ISC) Administrator Certification**
*Section 7: Supporting Governance | Module 7.4: Policies*

---

## Overview

This section covers the complete lifecycle of policy violations in SailPoint Identity Security Cloud — from detection through remediation and audit reporting. Building on the policy configuration knowledge from Section 7.4.2, we now focus on what happens **after** policies are active: how violations are detected, how alerts are routed, how violations are remediated or mitigated, and how the entire process feeds into compliance reporting.

**Key learning objectives:** Understand preventive vs. detective violation detection mechanisms, configure alert notifications and routing, execute violation remediation workflows, manage exception handling and approval processes, implement mitigation strategies with compensating controls, generate compliance reports and maintain audit trails, and troubleshoot common violation scenarios.

---

## Violation Detection Mechanisms

ISC detects policy violations through two fundamentally different mechanisms. Understanding **when** and **how** each mechanism fires is one of the most exam-relevant topics in this section.

### Preventive Detection (Pre-Access)

Preventive detection evaluates policy rules **at the time an access request is submitted** — before the access is actually provisioned to the target system.

**How it works:**

1. A user submits an access request through the ISC Access Request Center (e.g., requesting the "Approve Payments" role).
2. Before the request enters the approval workflow, ISC checks the request against all active policies that have preventive enforcement enabled.
3. ISC evaluates: "If this identity receives the requested access, will it create a policy violation?"
4. If a violation is detected, ISC takes the configured action — either blocking the request outright, flagging the request with a warning for the approver, or requiring additional approval from the policy owner.
5. The requester and/or approver receives notification of the conflict.

**What triggers preventive detection:**
- Access requests submitted through the ISC Access Request Center
- Access requests submitted via the ISC API
- Role assignment requests
- Access profile assignment requests

**What does NOT trigger preventive detection:**
- Entitlements granted directly on the target system (out-of-band provisioning)
- Access added through bulk import or CSV upload to the source
- Role or group membership changes made by a source system administrator
- Access inherited through organizational changes (e.g., department transfer that triggers role reassignment via lifecycle events)

> 🔴 **CRITICAL EXAM CONCEPT: The Preventive Detection Gap**
>
> Preventive detection has a fundamental blind spot: it **only** evaluates access changes that flow through ISC's request process. Any access granted outside of ISC is invisible to preventive controls until the next detective scan picks it up. This is why preventive enforcement alone is **never sufficient** — it must always be paired with detective detection.

### Detective Detection (Post-Access)

Detective detection evaluates policy rules **against existing access** — access that has already been provisioned. It scans the current state of all identities and their entitlements to find violations.

**How it works:**

1. A detective policy scan is triggered (by schedule, after aggregation, or on-demand).
2. ISC retrieves the current entitlements, roles, and access profiles for all in-scope identities.
3. Each identity's access is evaluated against every active policy rule.
4. If an identity's current access matches a violation condition, a violation record is created.
5. The policy owner and configured notification recipients are alerted.

**When detective scans run:**

| Trigger | Description | Typical Use |
|---------|-------------|-------------|
| **Post-aggregation** | Runs automatically after a source aggregation completes. | Best for catching changes imported from source systems. Ensures violations are detected as soon as new data enters ISC. |
| **Scheduled** | Runs on a configured schedule (daily, weekly, monthly). | Provides regular, predictable evaluation cycles. Useful for compliance reporting cadence alignment. |
| **On-demand** | Administrator manually triggers a policy evaluation. | Used after configuration changes, emergency audits, or ad-hoc compliance checks. |

> 📘 **EXAM TIP: Detection Timing**
>
> If an exam question asks "when will a violation be detected," consider the enforcement mode. Preventive = immediately during the access request. Detective = at the next scan (post-aggregation, scheduled, or manual). If the access was granted outside ISC, the answer is **always** detective — there is no preventive option for out-of-band access.

### Violation Detection Flow Diagram

```
Access Request via ISC ──► Preventive Check ──► Violation Detected?
                                                  │
                                    YES ──► Flag/Block Request
                                     │      Notify Approver
                                    NO ──► Continue Approval Flow
                                            ──► Provision Access

Direct Grant on Target System ──► No Preventive Check
                                     │
                                     ▼
                          Next Detective Scan ──► Violation Detected?
                                                    │
                                      YES ──► Create Violation Record
                                       │      Notify Policy Owner
                                      NO ──► No Action
```

---

## Alert Configuration and Notification Routing

When a violation is detected, ISC generates alerts and notifications to ensure the right people are informed. Proper alert configuration prevents both missed violations and alert fatigue.

### Notification Recipients

ISC supports multiple notification targets for policy violations. Each can be configured independently per policy.

| Recipient | Role | When They Should Be Notified |
|-----------|------|------------------------------|
| **Policy Owner** | Business stakeholder responsible for the policy. Reviews violations, approves exceptions. | Always — every violation should reach the policy owner. This is the primary notification target. |
| **Violation Identity's Manager** | The manager of the identity in violation. | When manager involvement is needed for remediation decisions (e.g., manager must decide which entitlement to revoke). |
| **Identity in Violation** | The person whose access is in conflict. | Optional — some organizations notify the user so they can self-remediate or provide business justification proactively. |
| **Governance Group/Distribution List** | A compliance team or governance committee. | When violations require committee review or when multiple stakeholders need visibility (e.g., SOX compliance team). |
| **ISC Administrator** | Technical admin who manages the ISC platform. | Only for systemic issues (e.g., a policy scan failure) — NOT for individual business violations. |

### Notification Configuration Options

Each policy's notification settings allow you to control the following:

- **Notification trigger:** Whether to send notifications for new violations only, or also for recurring violations that persist across scan cycles.
- **Notification frequency:** How often reminders are sent for unresolved violations (e.g., send a reminder every 7 days until the violation is resolved or mitigated).
- **Escalation notifications:** If a violation remains open for a defined period, escalate the notification to a higher-level stakeholder.
- **Email template:** ISC uses configurable email templates for violation notifications. Templates can include violation details, affected identity information, conflicting entitlements, and links to the violation record in the ISC console.

> ⚠️ **IMPORTANT: Alert Fatigue Is Real**
>
> One of the biggest implementation failures in policy management is alert fatigue. If a policy generates 500 violations on day one and the policy owner receives 500 emails, they will stop reading them. Prevention strategies include: (1) running simulation before activation to identify expected violations, (2) pre-mitigating known exceptions before going live, (3) using summary/digest notifications instead of individual emails, and (4) setting realistic violation thresholds that match the policy owner's capacity to review.

### Configuring Effective Alert Routing

**Decision framework for notification routing:**

| Violation Severity | Policy Owner | Manager | Identity | Governance Group |
|-------------------|:------------:|:-------:|:--------:|:----------------:|
| **Critical SoD (Financial)** | Always | Yes | Optional | Yes |
| **High-Risk Access Policy** | Always | Yes | No | Yes |
| **Account Policy (Hygiene)** | Always | Optional | No | No |
| **Informational/Monitoring** | Always | No | No | No |

> ✅ **BEST PRACTICE: Notification Routing**
>
> Route notifications based on who needs to **act**, not who needs to **know**. The policy owner always needs the notification because they are accountable. The manager gets it when they need to make a remediation decision. The governance group gets it when committee oversight is required. Don't CC everyone "just in case" — that's how alerts get ignored.

---

## Violation Lifecycle and Status Tracking

Every policy violation in ISC follows a defined lifecycle. Understanding the status transitions is critical for both exam questions and real-world management.

### Violation Statuses

| Status | Meaning | How It Gets Here | What Happens Next |
|--------|---------|------------------|-------------------|
| **Open** | Violation has been detected and no action has been taken. The identity currently holds conflicting access. | Detected by a policy scan (detective) or at request time (preventive). Also: when a mitigated violation's exception expires. | Must be reviewed by policy owner. Options: Remediate, Mitigate, or escalate. |
| **Mitigated** | Violation is acknowledged. The conflicting access remains, but a compensating control is documented and an exception has been approved. | Policy owner or reviewer creates a mitigation with justification, mitigating control, expiration date, and approver sign-off. | Remains Mitigated until the exception expires (returns to Open) or access is removed (moves to Resolved). |
| **Resolved** | The conflicting access has been removed. The violation condition no longer exists for this identity. | The identity's access changes such that the policy rule no longer matches. Typically through entitlement revocation, role removal, or account deprovisioning. | No further action needed. Violation record is retained for audit history. |

### Violation Lifecycle Flow

```
Policy Scan Detects Conflict
         │
         ▼
   ┌──────────┐
   │   OPEN   │◄──────────────────────────────────┐
   └────┬─────┘                                    │
        │                                          │
        ├── Conflicting access removed ──► RESOLVED │
        │                                          │
        └── Exception approved ──► MITIGATED       │
                                      │            │
                                      │  Exception │
                                      │  Expires   │
                                      └────────────┘
```

> 📘 **EXAM TIP: Status Transitions**
>
> Memorize these key transitions: (1) Open → Mitigated: exception is approved with compensating control. (2) Open → Resolved: conflicting access is removed. (3) Mitigated → Open: exception expires without renewal. (4) Mitigated → Resolved: conflicting access is removed while the exception is active. A violation **never** goes directly from Mitigated to Resolved by the expiration of the exception — expiration always returns it to Open first.

### Violation Record Details

Each violation record in ISC contains the following information, which forms the audit trail:

- **Violation ID:** Unique identifier for the violation instance.
- **Policy Name and Rule Name:** Which policy and specific rule was violated.
- **Identity:** The identity in violation (name, identity ID, employment details).
- **Conflicting Access:** The specific entitlements, roles, or accounts that triggered the violation (for SoD, this shows which items from the Left Set and Right Set are held).
- **Source Systems:** Which target systems the conflicting entitlements reside on.
- **Detection Date:** When the violation was first detected.
- **Current Status:** Open, Mitigated, or Resolved.
- **Mitigation Details (if applicable):** Justification, mitigating control description, approver, and expiration date.
- **Resolution Details (if applicable):** What access was removed, when, and by whom.
- **History/Audit Log:** Complete record of status changes, comments, and actions taken.

---

## Violation Remediation Workflows

Remediation is the process of resolving a violation by removing the conflicting access. ISC provides multiple pathways for remediation, and the choice depends on the organization's governance model.

### Remediation Options

| Remediation Method | Description | When to Use |
|-------------------|-------------|-------------|
| **Manual Remediation (Policy Owner)** | The policy owner reviews the violation and decides which entitlement to revoke. They initiate the revocation through ISC. | When the remediation decision requires business judgment (e.g., which of the two conflicting entitlements is less critical to the user's job). |
| **Manager-Initiated Remediation** | The violation is routed to the identity's manager, who decides the remediation action. | When the manager has the best context about the identity's daily responsibilities and which access is essential. |
| **Self-Service Remediation** | The identity in violation is notified and can voluntarily release access through the ISC Access Request Center. | Low-risk violations where the user is trusted to make the right decision. Reduces administrative overhead. |
| **Certification-Based Remediation** | The violation is flagged during the next access certification campaign. The reviewer revokes access as part of the certification decision. | When violations are not urgent and can wait for the next scheduled review cycle. Integrates violation remediation with the broader access review process. |
| **Automated Remediation** | ISC automatically revokes the conflicting access based on predefined rules (e.g., always remove the most recently granted entitlement). | High-volume, low-complexity violations where the remediation decision is deterministic. Use with extreme caution — automated revocation can disrupt business operations. |

### Remediation Decision Framework

When a violation is detected, the policy owner or reviewer must decide **which** access to remove. For SoD violations, the identity holds conflicting items from both the Left Set and Right Set — but removing access from either side resolves the violation. The decision should be based on:

1. **Job role alignment:** Which entitlement is core to the identity's current job function? Keep that one.
2. **Least privilege:** Which entitlement grants broader or more sensitive access? Remove the more privileged one.
3. **Temporary vs. permanent need:** Was one entitlement recently added for a temporary project? Remove the temporary one.
4. **Business impact:** Which revocation would cause less operational disruption?

> ⚠️ **IMPORTANT: Remediation Requires Provisioning**
>
> When a reviewer decides to revoke an entitlement to resolve a violation, ISC must actually provision the change to the target system. This means the source must have a configured connector with write/provisioning capability. If the source is read-only (aggregation only, no provisioning), ISC can create a manual work item for an administrator to remove the access directly on the target system. The exam may test whether you understand that remediation depends on provisioning configuration.

### Remediation via Certifications Integration

One of the most powerful features of ISC's policy engine is the integration between policy violations and certification campaigns. When you configure this integration:

- Violations are surfaced directly to the certification reviewer alongside the access being reviewed.
- The reviewer sees a visual indicator (typically a flag or warning icon) on items that are in violation.
- The reviewer can revoke the violating access as part of their normal certification workflow.
- If the reviewer approves (retains) the violating access, they are prompted to provide a justification, which ISC records as a mitigation.

> 📘 **EXAM TIP: Certification-Policy Integration**
>
> The exam frequently tests the interaction between policies and certifications. Key points: (1) Policy violations can be surfaced during certifications. (2) A reviewer who approves violating access must provide justification. (3) This justification is treated as a mitigation, not a resolution — the violation still exists but is acknowledged. (4) If the reviewer revokes the access, the violation moves to Resolved status after the revocation is provisioned.

---

## Exception Handling and Approval

Not every violation can be remediated. This section details the exception management process within ISC, expanding on the mitigation concepts introduced in Section 7.4.2.

### Exception Request Workflow

```
Violation Detected (Open)
         │
         ▼
Policy Owner Reviews Violation
         │
         ├── Can access be removed? ──► YES ──► Initiate Remediation ──► Resolved
         │
         └── Business justification exists? ──► YES ──► Create Exception Request
                                                              │
                                                              ▼
                                                    Document Justification
                                                    Define Mitigating Control
                                                    Set Expiration Date
                                                              │
                                                              ▼
                                                    Route to Approver
                                                              │
                                                    ┌─────────┴─────────┐
                                                    │                   │
                                                Approved           Rejected
                                                    │                   │
                                                    ▼                   ▼
                                              Status: Mitigated    Remains Open
                                                                   (must remediate)
```

### Exception Approval Levels

Organizations typically define approval hierarchies for exceptions based on risk level:

| Risk Level | Example Policy | Exception Approver | Renewal Requirements |
|------------|---------------|-------------------|---------------------|
| **Critical** | Financial SoD (SOX controls) | CFO or Audit Committee | Quarterly review. Must re-justify each renewal. Cannot exceed 1 year total. |
| **High** | Privileged access + contractor status | CISO or VP of Security | Semi-annual review. Requires updated risk assessment at each renewal. |
| **Medium** | Cross-departmental data access | Department Head + Compliance Officer | Annual review. Standard justification renewal process. |
| **Low** | Account proliferation (multiple accounts) | IT Manager | Annual review. Can be batch-renewed if conditions are unchanged. |

### Mitigating Control Standards

The quality of mitigating controls directly determines whether an exception will survive an audit. ISC stores the mitigating control description as free text, but organizations should enforce standards.

**A strong mitigating control must be:**

- **Specific:** Clearly describes what control is in place (not "manager is aware").
- **Measurable:** Can be verified by an auditor (e.g., "weekly report is generated and signed").
- **Active:** Requires ongoing action, not a one-time acknowledgment.
- **Documented:** Evidence of the control's execution is retained.
- **Proportional:** The control's rigor matches the risk level of the violation.

**Examples of strong vs. weak mitigating controls:**

| Weak (Will Fail Audit) | Strong (Audit-Ready) |
|------------------------|---------------------|
| "Manager is aware of the conflict." | "Manager reviews and signs off on all purchase orders created by this user weekly. Sign-off logs are retained in the compliance SharePoint site." |
| "Low risk, no action needed." | "Transactions by this user are limited to $5,000 by system configuration. All transactions over $2,500 require secondary approval from the Controller." |
| "Temporary access." | "Access expires on March 31, 2026 via ISC scheduled revocation. During this period, the user's activity is monitored through daily audit log review by the SOX compliance analyst." |
| "Approved by VP." | "VP approved with condition: user cannot process payments for vendors they created. This is enforced by a workflow restriction in SAP (transaction SU01 configuration documented in change ticket CHG-4521)." |

> 🔴 **CRITICAL: Auditor Perspective on Exceptions**
>
> External auditors (SOX, HIPAA, ISO 27001) will review your policy exceptions during an audit. They look for: (1) Is the exception justified? (2) Is the mitigating control real and verifiable? (3) Is there an expiration date? (4) Was the exception approved by someone with appropriate authority? (5) Has the exception been reviewed at regular intervals? If any of these are missing, the auditor may issue a finding — even if the underlying access is perfectly legitimate.

---

## Violation Reporting and Metrics

ISC provides reporting capabilities that transform policy violation data into actionable compliance intelligence. This section covers what reports are available, what metrics matter, and how to use them.

### Built-In Policy Reports

| Report | What It Shows | Primary Audience |
|--------|--------------|-----------------|
| **Policy Violation Summary** | Total violations by policy, status (Open/Mitigated/Resolved), and trend over time. | Compliance leadership, audit committee. High-level dashboard view. |
| **Violation Detail Report** | Individual violations with identity details, conflicting entitlements, detection date, and current status. | Policy owners. Used for day-to-day violation management. |
| **Exception/Mitigation Report** | All active exceptions with justifications, mitigating controls, approvers, and expiration dates. | Auditors and compliance officers. Used during audit reviews. |
| **Policy Effectiveness Report** | Metrics on detection rate, remediation time, exception rates, and recurring violations. | Governance program managers. Used to evaluate and improve the policy program. |
| **Violation Aging Report** | Violations grouped by how long they have been open (0-30 days, 31-60 days, 60+ days). | Compliance leadership. Identifies where remediation is stalling. |

### Key Compliance Metrics

These metrics are what compliance teams and auditors care about most. Understanding them helps you both pass the exam and run an effective governance program.

| Metric | What It Measures | Target/Benchmark | Why It Matters |
|--------|-----------------|------------------|----------------|
| **Open Violation Count** | Total number of unresolved violations across all policies. | Trending downward over time. | Rising open violations indicate remediation is not keeping pace with detection. |
| **Mean Time to Remediate (MTTR)** | Average number of days from violation detection to resolution. | Under 30 days for critical policies. | Long MTTR exposes the organization to prolonged risk. Auditors will flag slow remediation. |
| **Exception Rate** | Percentage of violations that are mitigated (exceptions) vs. resolved (access removed). | Below 20% for most policies. | High exception rates suggest the policy may be poorly calibrated, or the business process inherently requires the conflicting access. |
| **Recurring Violation Rate** | Percentage of violations that recur after being resolved (i.e., the same identity re-acquires the conflicting access). | Below 10%. | High recurrence indicates a root cause that is not being addressed — likely a role model issue or a broken joiner/mover process. |
| **Exception Expiration Compliance** | Percentage of exceptions that are reviewed and renewed (or remediated) before expiration. | 100%. | Expired, unreviewed exceptions are audit findings. They indicate governance process failures. |
| **Policy Coverage** | Percentage of high-risk business processes that have corresponding policies configured in ISC. | 100% of SOX-relevant processes. | Gaps in policy coverage mean risks are not being monitored. |

> 📘 **EXAM TIP: Metrics Interpretation**
>
> The exam may present a scenario with metrics data and ask you to identify the problem. For example: "A policy has a 45% exception rate. What does this indicate?" The answer is that the policy is likely too broad — nearly half of all violations are being excepted, which suggests the rule needs to be refined, or the business process requires conflicting access and the policy should be restructured to target only the truly risky combinations.

---

## Compliance Audit Trail

ISC maintains a comprehensive audit trail for all policy-related activities. This trail is the backbone of demonstrating compliance to internal and external auditors.

### What the Audit Trail Captures

| Event | Data Recorded |
|-------|--------------|
| **Policy Creation/Modification** | Who created or modified the policy, what changed, and when. Includes changes to rules, enforcement mode, and notification settings. |
| **Violation Detection** | Violation details (identity, conflicting access, source systems), detection method (preventive or detective), and timestamp. |
| **Violation Status Changes** | Every transition between Open, Mitigated, and Resolved — with the actor who initiated the change and timestamp. |
| **Exception Approval** | Justification text, mitigating control description, approver identity, approval timestamp, and expiration date. |
| **Exception Renewal/Expiration** | Whether the exception was renewed or expired, who renewed it, and the new expiration date. |
| **Remediation Actions** | What access was revoked, when, how (manual vs. automated), and the resulting provisioning event on the target system. |
| **Certification Integration Events** | When a violation was surfaced during certification, the reviewer's decision (revoke or mitigate), and justification if retained. |

### Audit Trail Retention

ISC retains violation and audit trail data according to the tenant's data retention configuration. For compliance-sensitive environments:

- Violation records should be retained for a minimum of **7 years** for SOX compliance.
- HIPAA environments may require **6 years** of retention.
- GDPR environments must balance retention needs with data minimization requirements.

> ✅ **BEST PRACTICE: Audit Preparation**
>
> Before an audit, export your policy violation summary, exception report, and remediation history. Auditors will ask: "Show me all SoD violations for the past year, how many were remediated, how many were excepted, and what controls are in place for the exceptions." Having this data readily available in ISC's reporting demonstrates a mature governance program.

---

## Common Violation Scenarios and Troubleshooting

This section covers the most frequently encountered violation scenarios and how to diagnose and resolve them. These are also common exam question patterns.

### Scenario 1: Policy Is Active But No Violations Are Detected

**Symptoms:** You created and activated an SoD policy, but no violations appear even though you know identities have conflicting access.

**Troubleshooting checklist:**

1. **Verify the policy state is Active**, not Draft. Draft policies do not generate violations.
2. **Check entitlement values in the rule.** Are you using the correct entitlement values (the technical values from aggregation), or display names? The rule must match the exact entitlement value as it appears in the aggregated data.
3. **Confirm aggregation has run** since the policy was activated. Detective detection requires current aggregated data.
4. **Verify the entitlements are on the correct source.** If your rule references "SAP" entitlements but the source is named "SAP_Production" in ISC, the rule may not match.
5. **Check identity scope.** Are the identities you expect to be in violation within the policy's identity scope? Some policies can be scoped to specific identity populations.
6. **Validate that the identities actually hold entitlements from BOTH sets** (for SoD). The identity must have at least one from the Left Set AND at least one from the Right Set.

> 🔴 **CRITICAL: Entitlement Value Mismatch**
>
> This is the #1 cause of "silent" policies that don't generate expected violations. The rule configuration uses the entitlement's **value** (the technical string aggregated from the source), not the display name. For example, the display name might be "Create Purchase Orders" but the value is `ZMM_PO_CREATE`. If you put "Create Purchase Orders" in the rule, it will never match. Always verify values against the aggregated entitlement data in the ISC entitlement catalog.

### Scenario 2: Too Many Violations After Policy Activation

**Symptoms:** You activated a new SoD policy and it immediately generated 2,000 violations. The policy owner is overwhelmed.

**Root causes and resolution:**

| Possible Cause | Diagnosis | Resolution |
|---------------|-----------|------------|
| Entitlement sets are too broad. | Review the Left and Right sets. Do they include entitlements that are commonly co-assigned and don't represent a genuine risk? | Narrow the entitlement sets to only the truly conflicting items. Remove generic or low-risk entitlements. |
| The business process inherently requires this access combination. | Talk to process owners. If 80% of violations are for a single team that legitimately needs both functions, the policy may not match reality. | Consider restructuring the rule to exclude the legitimate combination, or pre-create mitigations for the known population. |
| Policy was not simulated before activation. | Check if simulation was run. | Deactivate the policy (return to Draft), run simulation, review with stakeholders, then re-activate after adjustments. |
| Role model grants conflicting access by default. | Check if a role includes entitlements from both the Left and Right sets. | Fix the role model — remove the conflicting entitlement from the role. This resolves the root cause rather than treating symptoms. |

### Scenario 3: Violation Detected But Cannot Be Remediated

**Symptoms:** A violation is detected and the policy owner decides to revoke one of the conflicting entitlements, but the revocation fails or does not take effect.

**Troubleshooting checklist:**

1. **Source provisioning capability.** Does the source connector support write operations (provisioning)? If the source is configured for aggregation only (read-only), ISC cannot provision the revocation.
2. **Provisioning plan errors.** Check the provisioning activity log for errors. Common issues include: target system connectivity failures, insufficient permissions on the service account used by the connector, or attribute validation errors.
3. **Manual work item.** If automated provisioning is not possible, ISC should generate a manual work item for an IT administrator to perform the revocation directly on the target system. Verify that manual work items are configured and being routed.
4. **Entitlement is part of a role.** If the entitlement was granted through a role assignment (not direct), you cannot revoke the individual entitlement — you must remove the role. This may have broader access implications.

### Scenario 4: Violations Reappear After Remediation

**Symptoms:** You resolved a violation by revoking an entitlement, but the same violation reappears after the next aggregation or policy scan.

**Root causes and resolution:**

| Possible Cause | Diagnosis | Resolution |
|---------------|-----------|------------|
| Access was re-granted on the target system. | Check the identity's access history. Was the entitlement re-added directly on the source after ISC revoked it? | Identify who re-granted the access and address the process gap. Enable preventive enforcement to catch future re-grants through ISC. |
| Role assignment re-adds the entitlement. | Check if the identity has a role that includes the revoked entitlement. The role may be re-provisioning the entitlement. | Remove the identity from the role, or fix the role model to remove the conflicting entitlement. |
| Lifecycle event re-grants access. | Check if a lifecycle event (joiner, mover) triggered re-provisioning. | Review the lifecycle/birthright access rules to ensure they don't automatically assign conflicting access. |
| Aggregation re-discovers the entitlement. | If the revocation didn't actually take effect on the target system, the next aggregation will re-import the entitlement. | Verify the revocation was successful on the target system. Check provisioning logs. |

> 📘 **EXAM TIP: Recurring Violations Root Cause**
>
> When the exam describes recurring violations, the most common correct answer involves **role model issues** or **lifecycle automation conflicts**. The question tests whether you understand that removing an individual entitlement doesn't help if a role or automated process keeps re-granting it. You must address the root cause — the role assignment or the lifecycle rule — not just the symptom.

### Scenario 5: Preventive Check Blocks a Legitimate Access Request

**Symptoms:** A user submits an access request through ISC, but it is blocked by a preventive SoD policy check. The user insists the access is needed for their job.

**Resolution workflow:**

1. **Verify the violation is real.** Confirm the user's existing access and the requested access genuinely conflict under the policy rule.
2. **Assess business justification.** Does the user have a legitimate need for both? If yes, this is an exception scenario, not a policy error.
3. **Policy owner decision:** The policy owner can either approve an exception (creating a mitigation with compensating controls) or deny the request.
4. **If approved:** The exception is documented in ISC, the preventive block is overridden, and the access is provisioned with the mitigation on record.
5. **If denied:** The user must identify alternative access that does not create a conflict, or the business process must be restructured.

---

## Violation Status Dashboard and Operational Management

### Daily Governance Operations

Policy violation management is an ongoing operational activity, not a one-time configuration task. The ISC governance dashboard provides the operational view.

**Daily review checklist for policy owners:**

- Review new Open violations from the last 24 hours.
- Prioritize violations by policy severity/risk level.
- Assign remediation actions or initiate exception workflows.
- Check for exceptions approaching expiration (within 30 days).
- Verify that previously initiated remediations have been successfully provisioned.

**Weekly review checklist for governance program managers:**

- Review violation trend data (are open violations increasing or decreasing?).
- Evaluate MTTR metrics — are violations being resolved within SLA?
- Review exception rates by policy — do any policies have abnormally high exception rates?
- Check for recurring violations — are the same identities repeatedly appearing in violations?
- Validate that notification routing is working (confirm policy owners are receiving and acting on alerts).

**Monthly/Quarterly review for compliance leadership:**

- Generate compliance reports for audit committee review.
- Assess policy coverage against the risk register — are all high-risk processes covered?
- Review and update policy rules based on changes to business processes, organizational structure, or regulatory requirements.
- Conduct exception portfolio review — are all active exceptions still justified?

---

## Section Summary and Key Exam Concepts

### Quick Reference: Violation Lifecycle

| Phase | Key Action | Key Actor |
|-------|-----------|-----------|
| **Detection** | Preventive (request-time) or Detective (scan-based) | ISC system (automated) |
| **Notification** | Alert sent to configured recipients | ISC notification engine |
| **Review** | Assess violation, determine remediation or exception | Policy owner |
| **Remediation** | Revoke conflicting access | Policy owner, manager, or automated |
| **Mitigation** | Document exception with compensating control | Policy owner + approver |
| **Audit** | Report on violations, exceptions, and trends | Compliance team |

### Concepts Most Likely to Appear on the Exam

| Concept | What to Know |
|---------|-------------|
| **Preventive vs. Detective Detection** | Preventive = at request time, through ISC only. Detective = post-access, catches all channels. Both are needed. |
| **Preventive Detection Gap** | Does NOT catch out-of-band access grants. Only evaluates requests flowing through ISC. |
| **Violation Statuses** | Open → Mitigated (exception approved) → Open (exception expires) or Resolved (access removed). |
| **Mitigated vs. Resolved** | Mitigated = conflict still exists with compensating control. Resolved = conflicting access removed. |
| **Exception Expiration Behavior** | Expired exception returns violation to Open status. Does not auto-resolve. |
| **Certification Integration** | Violations are surfaced during certs. Reviewer approving violating access creates a mitigation, not a resolution. |
| **Entitlement Value Mismatch** | #1 cause of silent policies. Rules must use technical values, not display names. |
| **Recurring Violation Root Cause** | Usually a role model issue or lifecycle automation conflict. Must fix root cause, not just revoke the entitlement. |
| **Remediation and Provisioning** | Remediation requires the source to support provisioning. Read-only sources need manual work items. |
| **Exception Quality Standards** | Mitigating controls must be specific, measurable, active, documented, and proportional. "Manager is aware" fails audits. |

### Policy Troubleshooting Master Checklist

When the exam presents a policy violation issue, work through this checklist:

1. **No violations detected:**
   - Is the policy Active (not Draft)?
   - Do entitlement values in the rule match aggregated values?
   - Has aggregation run recently?
   - Are identities in scope for the policy?
   - For SoD: does the identity hold items from BOTH sets?

2. **Too many violations:**
   - Were entitlement sets defined too broadly?
   - Was simulation run before activation?
   - Does a role grant conflicting access by default?

3. **Violations recurring after remediation:**
   - Is a role re-provisioning the entitlement?
   - Is a lifecycle rule re-granting access?
   - Did the revocation actually succeed on the target system?

4. **Remediation failing:**
   - Does the source support provisioning (not read-only)?
   - Is the connector's service account properly configured?
   - Is the entitlement granted through a role (requiring role removal instead)?

5. **Preventive check not working:**
   - Is the access being granted through ISC or directly on the target system?
   - Is preventive enforcement enabled on the policy?
   - Is the policy Active?

> 📘 **FINAL EXAM TIP: The Compliance Story**
>
> Many exam questions on this topic can be answered by thinking about the **audit narrative**. Ask yourself: "If an auditor looked at this situation, what would they want to see?" They want to see that violations are detected promptly (detection), the right people are notified (alerts), violations are resolved or properly excepted (remediation/mitigation), exceptions are justified and time-bound (exception management), and there's a clear record of everything that happened (audit trail). If an answer choice supports this narrative, it's likely correct.

---

*End of Section 7.4.3 — Section 7.4: Policies is now complete.*
*Proceed to the next section in your study plan.*
