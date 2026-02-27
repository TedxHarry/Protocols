# What Happens When Employee Status Changes

Identity governance would be simple if people stayed in the same job forever. They don't.

Sarah Chen gets promoted. Marcus Rivera transfers to a new department. Jennifer Smith becomes Jennifer Rodriguez after getting married. David Park leaves the company. Each of these events changes what access is appropriate — and without automation, each one requires manual intervention across every connected system.

This is the **JML framework** — Join, Move, Leave. It's the fundamental lifecycle model for identity management, and the events in each phase are where governance either works or fails.

## JOIN: The Starting Point

Sarah's onboarding was covered in the previous document. To recap the key points:

- Authoritative source (Workday) creates the identity record
- ISC detects the new identity via aggregation
- Lifecycle state transitions to Active on start date
- Access assignment rules evaluate → role assigned → birthright access provisions
- Accounts created on all target systems automatically

JOIN is usually the most polished lifecycle event because it's well-understood, frequently exercised, and executives notice when new employees can't access their tools on day one. Most organizations get JOIN right relatively early in their ISC implementation.

MOVE and LEAVE are where governance complexity lives.

## MOVE: Four Real Change Scenarios

### Scenario 1: Promotion

Sarah is promoted from Software Engineer to Senior Software Engineer.

HR updates Workday: `jobTitle` changes from "Software Engineer" to "Senior Software Engineer", `jobCode` changes from `ENG-SW-003` to `ENG-SW-005`.

ISC detects the change during the next aggregation. Access assignment rules re-evaluate:
- Old rule ("Software Engineer"): `jobTitle CONTAINS "Engineer" AND jobCode = ENG-SW-003` → no longer matches
- New rule ("Senior Software Engineer"): `jobTitle CONTAINS "Senior" AND jobCode = ENG-SW-005` → matches

ISC transitions Sarah from the "Software Engineer" role to the "Senior Software Engineer" role. The difference in access profiles:

| Access Profile | Software Engineer | Senior Software Engineer | Change |
|---|---|---|---|
| GitHub Developer Access | ✓ | ✓ | No change |
| Jira Developer Access | ✓ | ✓ | No change |
| Slack Engineering Access | ✓ | ✓ | No change |
| AWS Developer Access | ✓ | ✓ | No change |
| AD Engineering Base | ✓ | ✓ | No change |
| GitHub Admin View | - | ✓ | **ADD** |
| Production Environment Access | - | ✓ | **ADD** |
| Code Review Approver | - | ✓ | **ADD** |

ISC provisions the three additional access profiles. No revocations needed (Senior Engineer role is a superset). Sarah gets expanded access automatically, without any manual IT tickets.

> 💡 **REMEMBER:** ISC handles both the additive and subtractive sides of role transitions. A promotion might be purely additive (superset role). A lateral transfer might be a swap (different entitlements, neither a superset). A demotion might be purely subtractive. ISC's delta provisioning handles each case: provision what's new, deprovision what's no longer applicable.

### Scenario 2: Department Transfer

Marcus Rivera transfers from the Finance department to DevOps Engineering. This is where lifecycle changes get complicated.

Before transfer: Marcus has the "Finance Analyst" role:
- SAP Finance access (read/write)
- Excel financial modeling tools
- Finance data warehouse (read)
- Quarterly compliance certification tools

After transfer: Marcus needs the "DevOps Engineer" role:
- AWS (admin-level for DevOps)
- Terraform Cloud
- PagerDuty
- Kubernetes management tools
- AD DevOps groups

The challenge isn't adding the new access — that's straightforward. The challenge is removing the old access correctly and completely.

ISC's rule evaluation detects the department change. The "Finance Analyst" role no longer matches Marcus's attributes. The "DevOps Engineer" role now matches. ISC creates a provisioning plan: add all DevOps Engineer access profiles, remove all Finance Analyst access profiles.

**The transition risk**: If there's a delay between removing Finance access and completing the move to DevOps, Marcus has a gap window where he can't access either set of systems productively. ISC can be configured to sequence provisioning to avoid this: provision new role first, then deprovision old role.

> ⚠️ **COMMON MISTAKE:** Assuming ISC automatically removes ALL previous access when someone changes roles. ISC removes access that was assigned *through the role*. It does NOT automatically remove access that was individually requested (approved access requests). If Marcus requested access to the Finance Reporting Portal separately from his role, that access persists even after his department transfer — because it wasn't role-assigned, so ISC doesn't revoke it during the role transition. This is why certification campaigns after role changes are important: they catch the residual individually-granted access.

### Scenario 3: Name Change

Jennifer Smith marries and legally changes her name to Jennifer Rodriguez.

HR updates Workday: `lastName` from "Smith" to "Rodriguez", email potentially changes from `jennifer.smith@globaltech.com` to `jennifer.rodriguez@globaltech.com`.

This triggers a cascade of attribute updates across every connected system:

**Active Directory**: `displayName` updated to Jennifer Rodriguez. `mail` updated to jennifer.rodriguez@globaltech.com. `userPrincipalName` updated.

**Salesforce**: `Username` needs updating (it's email-based). If ISC manages Salesforce usernames, ISC provisioning updates it. If not, it's a manual step.

**GitHub**: Display name updated. GitHub login (schen — wait, wrong person, Jennifer is `jsmith-dev`) needs to be evaluated — GitHub doesn't allow username changes via API, so this might require a manual GitHub admin action.

**Email aliases**: The old email address needs an alias configured so email sent to `jennifer.smith@globaltech.com` still reaches Jennifer during the transition.

**Correlation risk**: If email correlation is being used for any system, the email change breaks correlation for that system. ISC will try to correlate Jennifer's old account (with old email) to the updated identity (with new email) and may fail, creating an uncorrelated account.

This is why employee ID correlation is more robust than email correlation: the employee ID doesn't change when someone's name changes.

> 🎯 **KEY POINT:** Name changes are often treated as trivial HR updates. For identity governance, they're one of the most operationally complex lifecycle events — because email is often used as both the identity and correlation anchor across multiple systems. Design your correlation strategy to be resilient to name changes from day one.

### Scenario 4: Manager Change

Sarah Chen's manager James Wilson leaves GlobalTech. His direct reports (including Sarah) are temporarily reassigned to the Engineering Director, Priya Patel.

In ISC, Sarah's identity record updates: `manager` changes from `james.wilson@globaltech.com` to `priya.patel@globaltech.com`.

The downstream effects:
- Future access request approvals route to Priya
- Future certification campaigns for Sarah's access go to Priya
- Any access policies conditioned on manager approval route to Priya
- Any access profiles explicitly limited to James's direct reports that Sarah shouldn't have get re-evaluated

Manager changes are often overlooked as governance events, but they affect approval routing and certification assignment. If certifications are running and the manager record is stale, certifications might go to the wrong person — or an ex-employee.

## LEAVE: The Highest-Stakes Event

David Park resigns. Effective Friday.

LEAVE is the most governance-critical lifecycle event. A missed deprovisioning step means an ex-employee has active credentials on a company system after their departure. For privileged access, this is a security incident waiting to happen.

**Immediate disable vs. deferred deprovisioning**:

For most terminations (voluntary, positive-circumstance departures), some organizations configure a brief transition window: accounts remain active for the last day to allow data handoff, then disable triggers automatically at business-day end.

For involuntary terminations (performance, misconduct), immediate disable is typically required. Some organizations configure ISC with two termination profiles: standard (end-of-day disable) and immediate (disable within minutes of HR action).

**The termination sequence**:
1. Workday marks David as Terminated
2. ISC detects via aggregation (or real-time event trigger — preferred for terminations)
3. ISC transitions David's lifecycle state to Terminated
4. Termination workflow executes:
   - AD account disabled immediately (blocks SSO)
   - All group memberships removed from AD
   - Salesforce account disabled
   - GitHub org membership suspended
   - AWS IAM access keys deactivated
   - Jira account disabled
   - Slack deactivated
   - SAP account locked
5. Deprovisioning report generated → sent to security team
6. Manager notified to review any shared resources David owned
7. Certification campaign triggered for David's manager to confirm all access removed

**The orphaned account problem at LEAVE**:

The hardest part of deprovisioning isn't the systems ISC knows about — it's the systems it doesn't. If ISC governs 40 of GlobalTech's 47 systems, the remaining 7 are ungoverned. David might have accounts on those 7 systems. ISC can't deprovision them automatically.

This is why offboarding checklists still exist even with ISC: for ungoverned systems, manual deprovisioning is documented as a required step. The ISC termination workflow can trigger a ServiceNow ticket for manual deprovisioning of ungoverned systems, keeping the obligation tracked even when automation isn't possible.

> ⚠️ **COMMON MISTAKE:** Treating "ISC handled the deprovisioning" as "deprovisioning is complete." ISC handles the systems it governs. Your offboarding process must account for systems that aren't connected. The security team should receive a post-termination report that explicitly lists which systems ISC deprovisioned and which require manual verification.

## Key Takeaways

- The JML (Join, Move, Leave) framework covers all identity lifecycle events — governance must handle all three phases
- JOIN is usually the most polished phase; MOVE and LEAVE are where governance gaps most often occur
- Role transitions (MOVE) remove role-assigned access but do NOT remove individually-requested access — certifications must catch residual access
- Name changes are more complex than they appear: email-based correlation breaks, usernames may need manual updates, and aliases need configuration
- LEAVE is the highest-stakes event — immediate deprovisioning of all connected systems, with a documented process for ungoverned systems

> 🔗 **CONNECTS TO:** You now understand the full identity lifecycle. The next document categorizes the different types of integrations you'll build as an integration engineer — sources, targets, bidirectional, and flat file approaches.
