# Learning from Real Implementation Stories

Theory describes what should happen. Case studies describe what actually happened. The gap between them is where the most useful learning occurs — the unexpected data quality problems, the organizational resistance that slowed adoption, the technical decisions that turned out to be wrong and had to be rebuilt.

The three case studies in this document are composites from real ISC implementation patterns. The companies are fictional; the challenges, solutions, and lessons are drawn from patterns that recur in actual enterprise deployments.

---

## Case Study 1: Rapid Scale — Startup Growing from 100 to 5,000 Employees

**Background**: Veritas AI, an enterprise AI software company, grew from 90 employees to 520 employees in 18 months following a Series B funding round. Manual access management had worked when the HR manager knew every employee. By employee 200, it was failing visibly: new hires waited days for access, terminated employees retained accounts for weeks, and an external audit found 40 orphaned accounts with no identified owners.

**The Challenge**: The head of IT had 6 weeks to implement basic identity governance before the next external audit. The team was 3 people. Budget was limited. The existing "tech stack" was Google Workspace, GitHub, Slack, Notion, AWS, and an HR system (Rippling) that had been added 6 months ago.

**What They Did**:

*Week 1–2*: Connected Rippling as the authoritative HR source. Rippling had a pre-built ISC connector. First aggregation brought in 520 employee records. Correlation by email worked for 508 of them. The 12 that didn't correlate were all employees who had used personal Gmail addresses during the early days and had since gotten corporate emails — Rippling had the corporate email, ISC had no identities yet. Created identities for all 520; correlation issue was an initialization artifact.

*Week 3–4*: Connected Google Workspace (direct connector, OAuth). Connected GitHub (direct connector, OAuth). Connected AWS (SSO configuration using AWS Identity Center). Three systems, 2 weeks.

*Week 5–6*: Configured lifecycle states. Termination processing: Rippling → ISC lifecycle change → Google Workspace disabled, GitHub access removed, AWS SSO session terminated. Tested with 3 test identities. End-to-end termination: 22 minutes from Rippling update to all systems cleared.

**The Results at audit time**:
- 520 identities in ISC with >99% correlation
- 3 connected systems with automated lifecycle management
- 0 orphaned human accounts (the 40 orphans were investigated — 22 were former contractors, 18 were test accounts; all deprovisioned before the audit)
- First access certification campaign: 85% completion in 10 days

**What Went Well**:
The Rippling connector was straightforward. Google Workspace and GitHub pre-built connectors were configured in 2 days each. The small team size meant communication was fast — no approval chains, no stakeholder alignment marathons.

**What Was Hard**:
AWS was harder than expected. Veritas used AWS Organizations with 8 separate accounts. ISC integrates with AWS Identity Center (SSO), not individual IAM accounts. Getting AWS Organizations and Identity Center set up correctly took 2 weeks of AWS admin work before ISC could be connected. Lesson: SaaS identity governance depends on the underlying SaaS being correctly configured — ISC can't govern an AWS environment where AWS IAM is the access model rather than Identity Center.

**The Decision Veritas Got Right**: Scope to 3 systems for the audit, not 6. Slack and Notion were deferred to a Phase 2 after the audit. A clean 3-system implementation that actually worked was worth more than a 6-system implementation with half-configured connections.

**Lessons**:
- For rapid growth, prioritize systems that hold the most sensitive access (cloud infrastructure > internal wikis)
- Pre-built connectors for modern SaaS (Google, GitHub, Slack) make initial implementations faster than expected
- AWS governance requires AWS IAM architecture decisions before ISC configuration — if AWS isn't set up for SSO-based governance, ISC can't add it
- Scope ruthlessly in time-constrained implementations; add systems in planned phases

---

## Case Study 2: Legacy Modernization — Manufacturing Company With 20-Year-Old Systems

**Background**: Continental Components, a precision manufacturing company with 4,200 employees, had run their IT environment on on-premises systems since 2003. Their AD environment was 4 domains, accumulated through acquisitions. Their primary HR system was a 2004-vintage PeopleSoft installation. Their manufacturing floor access control was a proprietary system from a vendor that went out of business in 2015. And somehow, all of it still ran production.

**The Challenge**: Continental's insurance provider required a SOC 2 Type 2 audit for a new customer contract. The audit would evaluate logical access controls across Continental's systems. The IT director had 9 months.

**What They Did**:

*Phase 1 — HR modernization (parallel track)*: Continental upgraded from PeopleSoft to Workday over 6 months while simultaneously starting ISC implementation. This was the single most consequential decision of the project. Trying to connect the 2004 PeopleSoft to ISC via flat file while simultaneously implementing Workday would have meant building the integration twice. The 6-month wait for Workday was worth it.

*Phase 2 — AD consolidation (before ISC connection)*: 4 AD domains from acquisitions had never been consolidated. Attempting to govern 4 independent AD domains with different user naming conventions, different group structures, and different employee ID formats would have required 4 separate correlation rule sets. Continental's IT team did a 3-month AD consolidation (migrated 3 domains into the primary domain) before connecting to ISC. ISC then connected to 1 AD domain with a clean, consistent structure.

*Phase 3 — ISC implementation (Months 7–9)*: With Workday live and AD consolidated, the ISC implementation was comparatively smooth. Workday as source, AD as the primary target, and 8 additional systems (ERP read-only, quality management, Salesforce, Microsoft 365, SharePoint, 3 engineering tools).

**The Proprietary Manufacturing Floor System**: The old physical access control system had no API, no database access, and a vendor that no longer existed. Decision: governance without connectivity. ISC governed the digital systems; physical access was governed through a separate quarterly spreadsheet review that fed into ISC's certification evidence as a manual process. The auditors accepted this — the compensating control was documented and the physical access review was performed and evidenced.

**The Results at audit time**:
- 9 systems connected to ISC with automated lifecycle management
- Manufacturing floor access governed via documented manual quarterly review
- Termination processing: Workday → ISC → 9 systems disabled within 4 hours (frequent delta aggregation, event-driven not implemented yet)
- SOC 2 audit passed; one finding for the manufacturing floor system (documented exception accepted)

**What Was Hard**:
Change management. Continental's 4,200 employees were accustomed to requesting access via email to the IT manager. The introduction of an access request portal produced genuine resistance — "why do I have to click all these screens when I used to just email Tom?" The HR team had to actively champion ISC as the official process and decline email access requests to drive adoption.

**What Was Smart in Hindsight**:
Continental's decision to invest 6 months in HR and AD modernization before ISC implementation. The ISC project timeline was longer but the actual ISC implementation was cleaner, cheaper, and more reliable than it would have been with legacy infrastructure. Don't build identity governance on a crumbling foundation.

**Lessons**:
- Modernize the authoritative source and directory infrastructure before ISC implementation when both are needed — the upfront investment pays during implementation
- Legacy physical or proprietary systems without connectivity are a governance gap, but a documented manual process is an auditor-acceptable compensating control
- Change management (user adoption of the access request portal) is a project workstream, not an afterthought
- SOC 2 doesn't require perfection — it requires demonstrated controls and documented exceptions

---

## Case Study 3: Financial Services Compliance — Complex SoD and Certification Requirements

**Background**: Meridian Capital (introduced in Phase 7.1) at the end of their 18-month implementation. $85B AUM, 3,200 employees, 28 connected systems. The CISO wanted a specific outcome: prove to the SEC that Meridian had implemented effective access controls over trading and financial systems.

**The Challenge**: SEC examination requires evidence of:
1. Formal access provisioning process with documented approvals
2. Periodic access review with demonstrated remediation of inappropriate access
3. Separation of duties controls over trading activity
4. Timely deprovisioning of departed employee access

**What Made This Hard**:

*SoD across trading systems*: Meridian's SoD requirements involved 6 different trading and settlement systems (Bloomberg, Advent, Charles River, Linedata, DTC settlement, Refinitiv). A person with trade execution access in any system should not have trade approval access in any other. This is a cross-system SoD requirement that required ISC to see all 6 systems — which it did, after Phase 3 connections.

*Evidence quality for SEC*: The SEC examiner didn't just want "we ran certifications." They wanted evidence that certifications produced real outcomes — that reviewers made informed decisions, that revocations happened, that the remediation was completed. Meridian had to demonstrate that their 94% completion rate reflected genuine review, not rubber-stamp approval.

*What they did to demonstrate genuine review*: Implemented required-justification fields for all certification approve decisions on financial system access. Reviewers had to type a reason for approving, not just click. This slowed the review process and produced initial resistance, but the justifications became the evidence that reviews were substantive. The SEC examiner reviewed a sample of justifications and found them credible.

*Termination latency for traders*: The SEC examiner asked: "If a trader is terminated, how quickly are their access rights removed?" The previous answer (24-hour nightly batch) was unacceptable. The implementation of Workday event-driven webhook (Phase 5 of the implementation) reduced termination processing to under 15 minutes. This became a selling point in the examination: "We have technical controls that disable access faster than the SEC requires."

**The Results**:
- SEC examination passed without material findings
- SoD policy coverage across all 6 trading systems documented
- Certification evidence package: 14 campaign reports, remediation completion records, written justifications
- Termination evidence: Workday termination timestamp → ISC lifecycle change timestamp → all-systems-disabled timestamp, documented for 23 terminations in the examination period

**Lesson: Governance is the product**:
The most useful insight from the Meridian implementation: the SEC examiner was not impressed by ISC features. They were impressed by governance outcomes. "We have ISC" was irrelevant. "Here is evidence that inappropriate access was identified and removed on a documented schedule" was exactly what they needed.

The technology is the mechanism. Governance outcomes — reduced access risk, timely terminations, remediated SoD violations — are the product. Build ISC to produce those outcomes, not to demonstrate that ISC is configured.

**Lessons**:
- SoD across trading systems requires connecting all relevant systems before SoD policies can be correctly evaluated
- Certification evidence quality matters as much as completion rates — written justifications, remediation records, and timestamps are the substance
- Event-driven termination becomes a competitive differentiator in regulatory examination — "under 15 minutes" vs "up to 24 hours" is a meaningful difference
- The governance outcome (access risk reduced, violations remediated) is what regulators evaluate — not the technology platform

---

## Key Takeaways

- Startup rapid implementations: scope aggressively to the highest-risk systems; pre-built connectors for modern SaaS are fast to configure; AWS requires IAM architecture decisions before ISC
- Legacy modernization: invest in source and directory infrastructure before ISC configuration; document compensating controls for systems that can't be connected; change management is a project workstream
- Financial compliance: cross-system SoD requires all relevant systems connected; certification evidence quality includes written justifications and remediation records; termination latency is auditable
- The common thread: governance outcomes (reduced risk, timely terminations, remediated violations) are what stakeholders evaluate — ISC configuration is the means, not the end
- Each implementation faces data quality problems, organizational resistance, and scope constraints — the successful ones address all three explicitly

> 🔗 **CONNECTS TO:** Case studies show what's already been done. Continuous learning shows how to stay current as ISC evolves — the platform, the community, and the career path for integration engineers who want to grow their expertise.
