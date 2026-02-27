# When to Build Custom Connectors — And How to Decide

Every experienced ISC engineer has been in this situation: a stakeholder asks to connect a system that has no pre-built connector. The Web Services connector doesn't handle its authentication model. SCIM isn't supported. The JDBC approach works for reading but not writing. The system owner wants full provisioning.

The next words out of the stakeholder's mouth are usually: "Can you build a custom connector?"

The right answer is almost never the first thing that comes to mind. Building a custom connector is the correct choice for a small set of situations. For the majority, alternatives exist that are faster to implement and cheaper to maintain. This document gives you the decision framework.

---

## What a Custom Connector Actually Is

A custom connector is a Java application that implements SailPoint's Connector SDK interface. It exposes specific operations (aggregate accounts, aggregate entitlements, provision, deprovision) through a defined contract, and ISC calls those operations exactly as it would call a pre-built connector.

From ISC's perspective, a custom connector looks identical to a pre-built one. The difference is entirely in who built and maintains it — SailPoint for pre-built connectors, your team for custom ones.

**The maintenance obligation**: SailPoint updates the Connector SDK periodically. Major ISC version upgrades occasionally introduce breaking changes in the SDK. Your custom connector needs to be tested against every ISC upgrade and potentially updated. This is the cost that's almost always underestimated in the "should we build a custom connector?" conversation.

---

## The Decision Framework: Build or Alternative?

Work through this decision tree before committing to custom development:

### Step 1: Is There a Pre-Built Connector?

Search the SailPoint connector catalog (Compass, SailPoint's partner portal, and your ISC tenant's Sources catalog). As of 2025, the catalog has 200+ connectors and grows regularly. Systems added to the catalog in the last 2 years include many mid-market SaaS applications that previously required custom work.

If yes → use it. No further evaluation needed.

### Step 2: Does the System Support SCIM 2.0?

Check the system's integration documentation. Many SaaS vendors added SCIM 2.0 support in the last 3–5 years because it's the industry standard. SCIM covers account provisioning and group management well.

If yes → use the SCIM connector. It's maintained by SailPoint; you configure it, you don't build it.

### Step 3: Does the System Have a REST or SOAP API?

Nearly every modern system has one. Evaluate whether the Web Services connector can handle it:
- Is the authentication model standard? (OAuth 2.0, API key, Basic Auth, SAML)
- Are the API responses in JSON or XML with predictable structure?
- Are account and entitlement operations available as discrete API calls?
- Is error handling straightforward (standard HTTP status codes)?

If yes → the Web Services connector likely works. Spend a day configuring a proof of concept before escalating to custom development.

> ⚠️ **COMMON MISTAKE:** Escalating to custom connector too quickly when the Web Services connector would work. Web Services connector configuration is non-trivial — it requires understanding the API, constructing request/response templates, and handling edge cases. Engineers who haven't used it deeply sometimes incorrectly conclude it's insufficient. Always attempt Web Services connector first.

### Step 4: Is the Data in a Database ISC Can Reach?

If the system's identity data lives in a relational database and read/write access to the database is permitted by the vendor, the JDBC connector may work for both source and limited target operations.

### Step 5: Is Flat File the Right Fit?

For source-only integrations (ISC reads data, doesn't write back), flat file may be perfectly adequate — especially for legacy systems, physical systems, or systems where the vendor won't permit API access.

### Step 6: Can a Hybrid Approach Work?

Some integrations use multiple connectors together. A system that supports SCIM for provisioning but only exports flat files for reporting can be connected with a SCIM connector for write operations and a flat file connector for read operations. Not elegant, but effective.

### Step 7: Custom Connector

You've reached custom territory only if:
- No pre-built connector exists
- SCIM is not supported
- The Web Services connector can't handle the authentication model or API structure
- JDBC isn't applicable
- Flat file doesn't meet the governance requirements

At this point, assess:
- **What's the governance value of this system?** High-risk privileged system with 500 accounts = worth custom development. Low-risk wiki with 50 accounts = probably not.
- **What's the expected lifespan of the system?** If the system is being decommissioned in 18 months, investing in a custom connector is hard to justify.
- **Does the vendor have a roadmap to add SCIM or a pre-built connector?** Some vendors add ISC connector support if asked directly (especially partners or vendors with a SailPoint technology partnership).

---

## When Custom Connector Development Is Justified

**Justified case 1: Proprietary mainframe or legacy system**

A banking institution has a mainframe-based core banking system from the 1990s with a proprietary RACF security model and no REST API. The data is available via VSAM files or direct DB2 access, but the access model requires custom logic to translate ISC operations into RACF commands. This is a genuine custom connector case — no generic connector framework handles RACF provisioning.

**Justified case 2: Hardware or physical system with vendor SDK**

Manufacturing floor systems, badge access control systems, and industrial control systems often provide vendor SDKs (C++ libraries, Java APIs) rather than REST APIs. The Connector SDK can wrap these SDKs, translating ISC connector operations into vendor SDK calls.

**Justified case 3: High-value SaaS with non-standard authentication**

A critical business application that doesn't support standard OAuth or API key authentication (uses a proprietary token scheme, certificate-based mutual TLS, or a vendor-specific auth protocol). Web Services connector can't handle non-standard auth. If this system holds sensitive data and has 1,000+ accounts under governance, the build cost is justified.

**Justified case 4: Complex provisioning logic that no framework can express**

The system's account creation requires a multi-step process: call API A to create the user shell, call API B to set the password, call API C to enroll in MFA, call API D to assign roles — with error compensation if any step fails. This transaction pattern exceeds what Web Services connector can model.

---

## The Connector SDK: What Building Looks Like

SailPoint's Connector SDK is available at `developer.sailpoint.com`. Custom connectors are built in Java and implement the `AbstractConnector` interface.

**Operations to implement**:

| Operation | Required | What It Does |
|---|---|---|
| `testConnection` | Yes | Verify credentials and connectivity |
| `getSchema` | Yes | Return account and entitlement attribute schema |
| `iterate(AccountObject)` | Yes for source | Return all accounts (full aggregation) |
| `iterate(EntitlementObject)` | For entitlements | Return all entitlements (groups, roles, etc.) |
| `searchAccounts` | For delta | Return accounts changed since a timestamp |
| `create` | For provisioning | Create a new account |
| `update` | For provisioning | Modify an existing account |
| `delete` | For provisioning | Delete an account |
| `enable` / `disable` | For lifecycle | Enable/disable an account |
| `changePassword` | Optional | Set account password |

**Minimum viable connector** (source only, no provisioning):
- `testConnection`, `getSchema`, `iterate(AccountObject)`, `iterate(EntitlementObject)` = 4 operations
- Approximately 500–1,000 lines of Java
- Build time: 1–2 weeks for an experienced Java developer

**Full provisioning connector**:
- All operations
- Approximately 1,500–3,000 lines of Java
- Build time: 3–6 weeks + testing

**Testing requirements**: The SailPoint Connector Tester tool exercises each operation and validates the schema. Run it against every operation before submitting the connector to ISC.

**Deployment**: Package as a JAR file, upload to ISC via the admin console (Admin → Connections → Custom Connectors), configure the source using the custom connector.

---

## Maintenance Planning

When you decide to build a custom connector, plan for maintenance from day one:

**ISC upgrade testing**: Each ISC major version upgrade should be tested against the custom connector within 30 days. Build this into the ISC upgrade process.

**SDK version management**: The connector's pom.xml (Maven build file) references a specific Connector SDK version. When SailPoint releases a new SDK version, evaluate whether upgrading is required for ISC compatibility.

**Owner assignment**: The custom connector needs a named owner responsible for testing, updating, and support. If the engineer who built it leaves, the owner designation ensures someone else picks it up.

**Documentation**: Document the connector's design decisions, the system's API quirks it handles, and any known limitations. Custom connectors are only as maintainable as their documentation.

> 💡 **REMEMBER:** The build cost of a custom connector is a one-time investment. The maintenance cost is ongoing — 2–5% of the build cost per year in ISC upgrade testing alone, more if the target system's API changes. Factor this into the build-vs-alternative decision.

---

## Alternatives to Custom Connectors That Are Often Overlooked

**Governance without provisioning (service desk integration)**: If the system can't be provisioned directly, ISC can still govern it. Access requests go through ISC; ISC creates a ServiceNow ticket for manual provisioning; the IT team provisions manually and closes the ticket; ISC marks the request complete. Certification and visibility work; only provisioning is manual. This serves 80% of the governance value at 10% of the cost.

**Vendor partnership**: SailPoint has a technology partner program. Some vendors will build and certify a connector if asked, especially if you can coordinate with other customers of the same vendor who also want ISC connectivity. This is worth exploring before spending 6 weeks on custom development.

**Staged approach**: Connect the system via flat file now (immediate governance coverage). Plan custom connector development for the next quarter (full provisioning coverage). Governance coverage is the priority; full automation can come in the next phase.

---

## Key Takeaways

- Always work through the decision framework before committing to custom development: pre-built → SCIM → Web Services → JDBC → flat file → custom
- Custom connectors are justified for mainframe/legacy systems with no API, hardware systems with vendor SDKs, and complex provisioning flows that no generic framework can model
- Maintenance obligation is the hidden cost: test against every ISC upgrade, maintain documentation, assign a named owner
- Governance without direct provisioning (service desk ticketing) is a legitimate alternative for many systems — 80% of governance value, minimal development cost
- If building: implement `testConnection`, `getSchema`, account iteration, and entitlement iteration for a viable source connector; add create/update/enable/disable for a provisioning connector
- Vendor partnership or customer advocacy for a pre-built connector sometimes eliminates the need for custom development entirely

> 🔗 **CONNECTS TO:** Custom connectors handle the connectivity challenge. Performance optimization handles the operational challenge — making aggregations and provisioning operations fast enough at enterprise scale. The next document covers diagnosing and fixing ISC performance problems.
