# Measuring ISC Implementation Maturity

Two organizations both "have ISC deployed." One uses it to govern access for 3 systems with manual correlation and no certifications. The other governs 40 systems with automated lifecycle management, event-driven termination, quarterly certifications producing real revocations, and AI-assisted recommendations. Both "have ISC" — but they're at completely different points on the maturity curve.

The maturity model gives integration engineers and ISC platform teams a common language for assessing current state and a roadmap for meaningful improvement. It's also the framework for having the budget conversation: "to move from Level 2 to Level 3, we need X investment in Y capabilities."

---

## The Four Maturity Levels

### Level 1: Basic — Reactive and Manual

**Characteristics**:
- ISC deployed with 1–3 systems connected (usually AD as both source and target)
- Identity data in ISC is partially accurate; aggregation runs but with high uncorrelated account rates (>5%)
- Provisioning is manual or partially automated for a single system
- No certification campaigns configured or running
- No SoD policies
- Access requests managed outside ISC or via manual IT tickets
- The ISC platform team is 1–2 people, largely self-taught, using ISC primarily for reporting
- Termination processing is manual or nightly batch (24-hour window acceptable because there's no alternative)

**What it looks like in practice**: IT has ISC configured and does aggregations from Active Directory. They can answer "who has what AD groups" from ISC. But Salesforce, ServiceNow, GitHub, and 12 other systems are not connected. Access to those systems is tracked in spreadsheets. The quarterly access review is an email asking managers to review a spreadsheet. Response rate is 60%.

**Organizations at Level 1**: Typically recently deployed, small IT teams, limited budget, or ISC deployed as a point solution for a specific compliance requirement that's now past.

---

### Level 2: Managed — Documented Processes and Partial Automation

**Characteristics**:
- 5–15 systems connected, including the primary HR source and key business applications
- Correlation rate >97% for all connected systems; uncorrelated accounts documented
- Provisioning automated for at least 5 target systems via ISC roles and access profiles
- At least one certification campaign running (usually annual, manager campaign)
- SoD policies defined but in detective mode only (no preventive blocking)
- Role model covers 60% of job functions with birthright provisioning
- Lifecycle states (Active, Terminated) functioning; LOA may be manual
- ISC admin team of 2–4 people with formal training and documented runbooks
- Termination processing is 4–24 hours (nightly or frequent delta aggregation)

**What it looks like in practice**: GlobalTech in the first year after full Phase 3 deployment. Workday and AD are solid. Salesforce, GitHub, and ServiceNow are governed. The quarterly SOX certification runs in ISC; completion rate is 78% and the compliance team can produce audit evidence. The security team is starting to see value in the SoD violation reports. But 20 systems are still unconnected, LOA is handled via manual ticket, and the termination process still relies on overnight aggregation.

**Movement from Level 1 to Level 2**: Typically 6–12 months of focused implementation work. The defining characteristics of Level 2 are documented processes (not just configured settings) and the shift from "ISC as reporting tool" to "ISC as provisioning platform."

---

### Level 3: Optimized — Automated, Metrics-Driven, Governance-Active

**Characteristics**:
- 20+ systems connected, covering all high-risk and most medium-risk applications
- Correlation rate >99.5% across all connected systems; service account governance fully configured
- Full lifecycle automation: Pre-hire provisioning, active LOA management, event-driven terminations
- Certification program: quarterly for privileged/financial access, annual for standard; completion rate >90%; revocation rate 3–8%; remediation completed within SLA
- SoD policies in preventive mode for critical conflicts; detective mode for medium-risk
- Role model covers 85%+ of job functions; role governance board reviewing new requests
- Access request catalog fully populated with described, owned access profiles
- ISC admin team of 4–8 people; dedicated ISC security engineering capability
- Termination processing <15 minutes for privileged users (event-driven Workday webhook)
- Performance metrics tracked: aggregation duration, provisioning success rate, certification completion, violation age

**What it looks like in practice**: Meridian Capital at month 18 of their implementation. The CISO's dashboard shows real-time governance coverage across 28 systems. The quarterly SOX certification produces 94% completion in 10 business days. The SoD engine blocks 3–5 conflicting access requests per week. When an employee is terminated, the security team's Slack channel shows all accounts disabled in under 12 minutes. Internal Audit finds no material governance gaps in the annual review.

**What distinguishes Level 3 from Level 2**: Metrics. Level 3 organizations know their certification completion rate, their violation age distribution, their provisioning success rate, and their aggregation duration trends. They use those metrics to drive improvement decisions — not to report on ISC health, but to improve it.

**Movement from Level 2 to Level 3**: 12–18 months. The defining investment is operational maturity: monitoring, metrics, SLA definitions, and the governance board processes that prevent the configuration from drifting.

> 🎯 **KEY POINT:** Most ISC implementations plateau at Level 2. The configuration is done; systems are connected; certifications run. But without metrics-driven continuous improvement, the implementation doesn't become more effective over time — it becomes less effective as the organization grows and the configuration drifts.

---

### Level 4: Advanced — AI-Driven, Strategic, Real-Time

**Characteristics**:
- 30+ connected systems; near-complete application coverage
- AI access recommendations active and trusted by reviewers; recommendation acceptance rate tracked
- Real-time access intelligence: ISC dashboards show live governance posture, not day-old aggregation snapshots
- Risk-adaptive certifications: high-risk identities certified more frequently, automatically identified by AI
- Automated remediation for low-risk violations (auto-revoke after defined period without exception approval)
- ISC data feeds enterprise risk management systems (SIEM, GRC platforms)
- Zero-trust integration: ISC access decisions inform network access control and cloud security posture
- ISC platform team acts as strategic advisors, not just operators — influencing application procurement decisions, security architecture, and M&A integration planning

**What it looks like in practice**: A large financial institution or regulated enterprise where identity governance is a board-level concern. ISC is not the IT team's tool — it's the enterprise's access governance platform that risk management, compliance, and business leadership actively use.

**Organizations at Level 4**: Global financial services companies, large healthcare systems, defense contractors with strict access controls. Level 4 is achievable but requires sustained investment over 3+ years.

---

## Self-Assessment Framework

Rate your current ISC implementation on each dimension:

| Dimension | Level 1 | Level 2 | Level 3 | Level 4 |
|---|---|---|---|---|
| **System coverage** | 1–3 systems | 5–15 systems | 20+ systems | 30+ systems, near-complete |
| **Correlation quality** | >5% uncorrelated | <3% uncorrelated | <0.5% uncorrelated | <0.1%, AI-assisted |
| **Provisioning automation** | Manual or 1 system | 5+ systems automated | All high-risk automated | Fully automated, predictive |
| **Certification program** | None or ad hoc | Annual, <80% completion | Quarterly, >90% completion | Risk-adaptive, AI-assisted |
| **SoD enforcement** | None | Detective mode | Preventive + detective | Automated remediation |
| **Lifecycle automation** | Manual termination | Nightly batch | Event-driven (<15 min) | Real-time, AI-predictive |
| **Metrics and monitoring** | None | Basic alerts | Full dashboard, SLAs defined | Predictive analytics |

Score each dimension 1–4. Average score indicates overall maturity level. Dimension where score is lowest indicates the highest-value improvement area.

---

## Roadmap: Moving to the Next Level

**Level 1 → Level 2 Priority Actions**:
1. Connect 3–5 more high-risk systems (focus on financial and privileged access systems)
2. Run first certification campaign (even if small scope — prove the process)
3. Implement SoD policies in detective mode for top 5 conflicts
4. Achieve >97% correlation on all connected systems (data remediation investment)

**Level 2 → Level 3 Priority Actions**:
1. Implement event-driven termination (Workday webhook for privileged user terminations)
2. Expand certification program to quarterly frequency for high-risk access
3. Enable SoD preventive blocking for the highest-severity policies
4. Define and track governance metrics (certification completion, violation age, provisioning SLAs)
5. Implement Pre-hire lifecycle state for Day 1 readiness

**Level 3 → Level 4 Priority Actions**:
1. Enable and validate AI access recommendations across certification campaigns
2. Integrate ISC with SIEM (export governance events as security signals)
3. Implement risk-adaptive certification (certify higher-risk identities more frequently)
4. Build ISC into application procurement requirements (new applications must support SCIM or have a connector)
5. Develop ISC governance metrics dashboard for executive visibility

---

## Key Takeaways

- The four maturity levels (Basic, Managed, Optimized, Advanced) provide a shared language for assessing current state and planning improvements
- Most ISC implementations plateau at Level 2 — configuration is done but metrics-driven improvement doesn't happen
- Level 3 is distinguished by operational metrics: if you don't know your certification completion rate and violation age distribution, you're not at Level 3
- Self-assessment by dimension reveals the specific areas with highest improvement value (often lifecycle automation or certification quality)
- Movement between levels requires 12–18 months of focused investment, not just additional configuration
- The primary audience for maturity assessment is not technical — it's the executive conversation about "what have we gotten from our ISC investment and where should we invest next?"

> 🔗 **CONNECTS TO:** Maturity assessment identifies where to focus improvement. Expert troubleshooting addresses the hard problems that arise in mature implementations — the 20% of issues that consume 80% of investigation time. The next document is a deep dive into advanced problem-solving.
