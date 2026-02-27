# What Does an Integration Engineer Actually Do?

Job descriptions for ISC integration engineers tend to say things like "implement and configure SailPoint ISC connectors" and "develop identity governance workflows." That's accurate the way "writes code" is accurate for a software engineer — true, but not particularly illuminating about what the job actually involves.

Here's a more honest picture: about 40% of the work is technical configuration and development. The other 60% is information archaeology — figuring out what the systems actually do, what the business actually needs, why the current state is the way it is, and translating between technical realities and business requirements that are often stated in completely different vocabularies.

## What You're Actually Doing Day-to-Day

An integration engineer's work falls into three core activities: requirements gathering, technical design, and implementation with testing. Let's look at each honestly.

## Activity 1: Requirements Gathering

The project kickoff meeting. A room full of stakeholders — an HR Director, a CISO, an IT Director, and maybe an ISC Admin who's been running the existing configuration. Someone says: "We need to connect Workday to ISC."

What does that mean, exactly? It means nothing yet. Here's what you actually need to find out:

**Discovery questions to ask for any new integration:**

1. **What is this system, and what does it contain?** (Workday HR, contractor management, financial system — each has different governance implications)

2. **Is this a source or a target?** (Does this system tell ISC who people are, or does ISC tell this system who gets access?)

3. **Which attributes matter?** (For an HR source: employee ID, name, department, title, manager, status. NOT salary, SSN, or personal phone — governance doesn't need those and you shouldn't aggregate them.)

4. **How many records are we talking about?** (500 accounts vs. 50,000 accounts affects aggregation scheduling, VA sizing, and performance expectations.)

5. **What's the current state?** (Is this a new system, or are there existing accounts in ISC that need to be mapped to this source? Are there orphaned accounts we'll be inheriting?)

6. **What are the lifecycle events?** (How does this system signal "new hire"? "Termination"? "Leave"? Are events real-time or batch?)

7. **What's the latency tolerance for terminations?** (This is the most important latency question — it determines whether you can use nightly aggregation or need event-driven triggers.)

8. **Who approves access to this system?** (For target systems: what approval chain does the org need before ISC provisions an account?)

9. **Are there contractors, service accounts, or non-human identities in this system?** (These need special handling in correlation rules.)

10. **What does a "correct" integration look like?** (How will stakeholders verify the integration is working? What reports or evidence do they need for compliance?)

> 🎯 **KEY POINT:** Business stakeholders almost never give you answers to these questions unprompted. They say "connect Workday to ISC" and expect you to make it happen. Your job is to ask the right questions to surface the requirements they haven't articulated yet — because they'll definitely notice when something is missing after the fact.

The ambiguity problem is real. When the IT Director says "sync user data from Workday," they mean: update employee attributes when they change and deprovision accounts when people leave. When the HR Director hears "sync user data," they mean: ISC should show HR directors which systems their employees have access to. When the Security team hears it, they mean: ISC should pull Workday data to evaluate access risk against HR policies.

All three are reasonable interpretations. All three have different technical implications. Getting everyone in the same room to agree on the answer before you write a single line of configuration is not optional — it's the entire game.

> ⚠️ **COMMON MISTAKE:** Starting technical implementation before requirements are complete. The temptation is real: you know how to configure a Workday connector, you have the credentials, and the ISC admin tenant is sitting there ready. Starting now feels productive. But every assumption you bake into the configuration is a rework risk later. Spend the time on discovery first.

## Activity 2: Technical Design

Requirements complete (or good enough). Now you translate business requirements into technical specifications that ISC can implement.

**Attribute mapping**: For a Workday source, you decide which Workday attributes map to which ISC identity attributes. The output is a mapping table:

| Workday Attribute | ISC Identity Attribute | Transformation Needed? |
|---|---|---|
| `Worker_ID` | `employeeId` | None (direct map) |
| `Legal_Name/First_Name` | `firstname` | None |
| `Legal_Name/Last_Name` | `lastname` | None |
| `Primary_Work_Email` | `email` | Lowercase (normalize) |
| `Organization_Reference/Name` | `department` | Lookup table (dept codes → dept names) |
| `Job_Title` | `jobTitle` | None |
| `Worker_Type` | `employmentType` | Map: Employee→Full-Time, Contractor→Contractor |
| `Active_Status` | `lifecycleState` | Map: true→Active, false→Inactive |
| `Manager/Worker_ID` | `manager` | Lookup (manager ID → manager email) |

**Correlation design**: For each system being connected, you define the correlation rule. For Workday as a source, you need to ensure the `employeeId` attribute is populated and consistent with what target systems store.

**Access profile and role design**: Working with the business to define what "Software Engineer" should include — which access profiles from which systems. This involves system owners, HR, and security teams because it's a business decision encoded as technical configuration.

**Provisioning workflow design**: What approval chains exist for different types of access? Which access is birthright (automatic, no approval) versus requested (requires approval)?

**Deprovisioning design** (often skipped by beginners — don't): What happens at each lifecycle state transition? What's the sequence? What happens to data owned by the terminated user?

> 💡 **REMEMBER:** The design document is your contract with stakeholders. It specifies exactly what you'll build and what "done" looks like. Getting sign-off on the design before implementation prevents the "that's not what we meant" conversation after a two-week build sprint.

## Activity 3: Implementation and Testing

Configuration in ISC for most integrations is less "writing code" and more "assembling the right configuration pieces in the right order." But it requires precision — one wrong attribute mapping, one incorrect correlation rule, one misspelled field name, and the integration produces wrong data at scale.

**The typical implementation sequence**:

1. Configure the connector in ISC (authentication credentials, endpoint URL, attribute schema)
2. Run a test aggregation on a small data subset (not production, or a filtered subset)
3. Verify account records look correct in ISC
4. Configure correlation rules and verify against known test identities
5. Verify entitlement aggregation (groups, profiles, roles) looks correct
6. Test access assignment rules on test identities
7. Test provisioning in sandbox/non-prod target systems
8. Run full aggregation and verify results at scale
9. Test lifecycle events (simulate hire, role change, termination) in sandbox
10. Load test if scale requires it
11. Document the configuration thoroughly
12. Plan the production cutover and run it

**Dealing with real data problems**: In a perfect world, Workday has clean, complete data. In reality, you'll find:
- Employees with two records (name change created a duplicate)
- Missing employee IDs on contractor records
- Department codes that don't exist in any policy definition
- 40 accounts that don't match any correlation rule for reasons nobody can explain
- A service account with the same naming pattern as real user accounts

Each of these is a debugging conversation: is this a data quality issue (fix in the source system), a correlation rule issue (fix the rule), or an exception case (document and handle separately)?

## A Real Day

Sprint planning Monday morning. Your current sprint has three items:
1. Debug a Workday correlation rule that's generating 47 uncorrelated accounts
2. Complete the configuration document for the GitHub Enterprise connector
3. Review the access certification results from last quarter with the CISO to determine if the certification campaign design needs adjustment

**9 AM**: Sprint planning with the team. Discuss priorities, dependencies, blockers. The GitHub connector needs a service account credential that IT hasn't provided yet — add blocker, ping IT.

**10 AM**: Dig into the 47 uncorrelated Workday accounts. Export the uncorrelated account list from ISC. Cross-reference against Workday. Discover: 15 are contractors with non-standard employee IDs (missing the "GT-" prefix), 22 are service accounts that should be excluded, and 10 are legitimate employees who joined after a system migration and have a different ID format.

**12 PM**: Three fixes needed: update contractor correlation rule to handle missing prefix, add service account exclusion pattern, update migration-legacy account handling. Document root cause for each.

**2 PM**: Meeting with CISO to review certification results. 89% completion rate — good. But 23% of certifications were "bulk approved" (manager opened campaign, clicked approve all, closed in under 2 minutes). Those are rubber-stamp certifications. Recommend: add entitlement descriptions so managers understand what they're reviewing, and add a minimum review time prompt for campaigns with high-risk entitlements.

**4 PM**: Write up GitHub connector configuration notes. IT still hasn't provided credentials — document that GitHub connector is blocked pending credential delivery.

This is the actual job. Some configuration work, some debugging, some stakeholder communication, some process design. The technical skills matter. So do the communication skills. Neither alone is sufficient.

## Skills That Actually Matter

**Data thinking**: You're constantly looking at attribute values, correlation mismatches, and entitlement sets. SQL-level thinking (filter, join, aggregate, compare) applies constantly even though you're rarely writing SQL.

**Reading API documentation**: Every connector you work with has an API behind it. When something doesn't work, you need to read the system's API docs to understand what it actually returns versus what you expected.

**BeanShell/rule basics**: ISC uses BeanShell (Java-like scripting) for rule-based logic. You don't need to be a Java developer, but you need to read and modify rules without breaking them.

**Business translation**: Converting what a business stakeholder says they need into what ISC can actually do — and communicating back what ISC is doing in terms the stakeholder understands. This is the skill most engineers underestimate.

**Systematic debugging**: When something is wrong (correlation fails, provisioning errors, wrong access assigned), methodical root cause analysis beats random configuration changes every time.

## Key Takeaways

- Integration engineering is 40% technical configuration, 60% requirements gathering, design, communication, and debugging
- Discovery requires asking 10+ specific questions before touching ISC configuration — business stakeholders rarely volunteer everything you need
- Technical design produces artifacts (attribute mapping, correlation rules, access profile design, deprovisioning sequence) that need stakeholder sign-off before implementation
- Real data problems are inevitable and are part of the job — each one needs a root cause, not just a workaround
- The job requires both technical depth (data thinking, rule writing) and business communication (translating requirements, explaining governance outcomes)

> 🔗 **CONNECTS TO:** Now that you understand the role, the next document gives you the decision-making framework that governs every integration design choice — three questions that apply to any integration you'll ever build.
