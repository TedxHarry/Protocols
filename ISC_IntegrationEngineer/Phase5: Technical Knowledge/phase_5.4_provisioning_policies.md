# Provisioning Policies — How ISC Creates and Manages Accounts

ISC knows it should provision an account for Sarah Chen in GitHub. It has the role assignment. The lifecycle state is Active. The access profile for "GitHub Contributor" is in scope.

But what, exactly, should the GitHub account look like? What username? What display name? What email? What happens if an account already exists? What if Sarah had a GitHub account before that needs to be reactivated rather than re-created?

Provisioning policies answer these questions. They are the instructions ISC follows when executing a provisioning operation — the detailed specification of what to create, what to set, and how to handle the variations that appear in practice.

---

## What a Provisioning Policy Specifies

A provisioning policy in ISC is attached to a source (target system) and defines the attribute values that ISC should use when creating or modifying accounts in that system.

Provisioning policies specify:
- **Account attribute values**: What username format? What email? What display name? What default profile?
- **Required vs optional attributes**: Which attributes must be present for provisioning to succeed?
- **Attribute sources**: Where does each value come from? (Identity attribute, transform, static value, or generated)
- **Plan operations**: What ISC can do in this system — Create, Update, Enable, Disable, Delete, Unlock, Change Password

---

## The Provisioning Plan

Before ISC executes a provisioning operation, it builds a provisioning plan — a structured description of what operations need to happen.

A provisioning plan might contain:
```
Target: GitHub
Operation: Create Account
Attributes:
  - login: "sarah.chen"
  - name: "Sarah Chen"
  - email: "sarah.chen@globaltech.com"

Target: GitHub
Operation: Add Entitlement
  - team: "engineering-team"
  - team: "ci-readers"
```

ISC assembles the plan from:
1. The provisioning policy (what attributes to set, from where)
2. The identity's current attribute values (applied via transforms)
3. The access profile's entitlement list (what groups/roles to add)

The plan is then submitted to the connector, which executes the API calls against the target system.

**Plan compilation errors**: If the provisioning policy requires an attribute that the identity doesn't have, the plan fails to compile. Example: the GitHub policy requires `email`, but the identity's email attribute is null (perhaps a contractor without a corporate email). The provisioning fails with a "missing required attribute" error. This is why data quality in your authoritative source matters: missing identity attributes cause provisioning failures downstream.

---

## Attribute Value Sources in Provisioning Policies

Each attribute in a provisioning policy gets its value from one of four sources:

### Identity Attribute

The value comes directly from the identity's profile in ISC.

```
GitHub.login ← Identity.uid
GitHub.email ← Identity.email
GitHub.name  ← Identity.displayName
```

Simple, direct, no transformation needed. Use this when the source attribute matches the target format.

### Transform

The value is derived by applying a transform to one or more identity attributes.

```
GitHub.login ← Concat(Lower(Identity.firstname), ".", Lower(Identity.lastname))
```

The same transform library from Phase 4.4 applies here. For provisioning policies, the most common transforms are:
- `Concat` for building username/email formats
- `Lower` / `Upper` for normalizing case
- `Substring` for extracting portions (first initial + last name patterns)
- `Lookup` for mapping ISC values to target system values (e.g., ISC department name → GitHub team slug)
- `Conditional` for handling edge cases (if preferred name exists, use it; otherwise use legal name)

### Static Value

A fixed value applied to all provisioned accounts.

```
GitHub.company ← "GlobalTech"
GitHub.defaultProfileType ← "Member"
```

Use static values for: company identifiers, default role assignments, standard country codes, default settings that every provisioned account should have.

### Generated Value

Some attributes need to be unique and can't be derived from identity data — they need to be generated. ISC supports:

- **UniqueValue**: Generate a unique value based on a template, automatically incrementing if conflicts occur. `sarah.chen` → if taken, `sarah.chen1` → if taken, `sarah.chen2`
- **Random**: For temporary passwords or tokens
- **UUID**: For systems that require GUID-format identifiers

---

## Handling Account Conflicts

What happens when ISC tries to provision an account that already exists?

This happens more often than you'd expect: Sarah Chen may have had a GitHub account from before ISC governed GitHub. Or she had an account, left the company, was rehired, and her old account was never deleted.

ISC handles this via **conflict resolution** in the provisioning policy:

**Create new account**: Ignore the existing account, create a new one. Risk: duplicate accounts, the old one still exists ungoverned.

**Link to existing account**: If an account with a matching attribute (email, username) exists, link it to the identity instead of creating a new one. The existing account gets updated to match the provisioning policy and entitlements are added. This is generally the right behavior for rehires.

**Fail and alert**: Stop the provisioning operation and generate a human-review task. An administrator reviews whether to link, replace, or investigate. Used for security-sensitive systems where automated conflict resolution could be risky.

**GlobalTech's conflict resolution policy**:
- Systems where personal accounts are common (GitHub, Slack): fail and alert — IT reviews whether the existing account is a personal account that should be replaced or converted
- Directory systems (AD): link to existing account — most pre-existing AD accounts should be reactivated not replaced
- SaaS with company-controlled domains (Salesforce, ServiceNow): create new (previous accounts should have been deleted at termination; if they weren't, that's a deprovisioning failure to investigate)

---

## Plan Operations: What ISC Can Do

Provisioning policies define which operations ISC can execute against a target system. Not every source connector supports every operation.

| Operation | Description | Common Gap |
|---|---|---|
| **Create** | Create a new account | Nearly universal |
| **Update** | Modify existing account attributes | Depends on connector/API |
| **Enable** | Activate a disabled account | Most connectors support |
| **Disable** | Deactivate an account (not delete) | Most connectors support |
| **Delete** | Permanently remove an account | Some systems restrict programmatic deletion |
| **Unlock** | Clear an account lockout state | AD, LDAP; not all SaaS |
| **Change Password** | Set or reset a password | AD, LDAP; varies elsewhere |

**Where gaps create problems**:

If a connector doesn't support **Disable**, ISC can't disable the account during Leave of Absence — the only option is Delete (too destructive) or leave unchanged (a governance gap). Solution: confirm connector capabilities before designing lifecycle actions.

If a connector doesn't support **Update**, ISC can't push attribute changes to existing accounts (e.g., when an employee's name changes after marriage). The account is created correctly, but ongoing attribute synchronization doesn't work. This is more common with older connectors and flat-file integrations.

> ⚠️ **COMMON MISTAKE:** Designing a lifecycle action (e.g., "disable account on LOA") for a source whose connector doesn't support the Disable operation. The action is configured, it appears to work in the admin console, but it silently does nothing because the connector has no implementation for that plan operation. Always verify supported plan operations in the connector documentation before designing lifecycle actions around them.

---

## Password Policies in Provisioning

When ISC creates an account in a target system that requires a password, it follows the password policy configured for that source.

**Options for initial password setting**:

**Generate random password**: ISC generates a random password meeting the complexity requirements and either: sends it to the identity via email, stores it in a PAM vault, or requires a password reset on first login.

**Identity sets password during request**: The access request form includes a password field. The user sets their password during the request process. ISC includes it in the provisioning plan.

**No-password provisioning (SSO)**: Many modern SaaS systems provision accounts without passwords — authentication is handled via SSO (Okta, Entra ID) and the account itself never has a locally set password. For these systems, the password field in the provisioning policy is not applicable.

**Password propagation**: For systems like Active Directory where ISC manages the password lifecycle, ISC can enforce password changes at intervals, push password resets to connected systems, and handle unlock operations.

---

## The Provisioning Log: What Happened and Why

Every provisioning operation is logged in ISC's provisioning activity. The log shows:
- Which identity the operation was for
- Which target system was provisioned
- What plan was executed (which attributes, which entitlements)
- What the result was: success, failed, pending

**Common provisioning failure causes and where to find them**:

| Error Type | Likely Cause | Where to Look |
|---|---|---|
| Missing required attribute | Identity attribute is null; transform returns null | Identity attributes, provisioning policy, transform config |
| Account conflict | Account already exists in target | Conflict resolution config; check target system manually |
| Connector error | API call failed (auth, timeout, system error) | VA log (ccg.log), connector error details in ISC |
| Entitlement not found | Group/role name in access profile doesn't exist in target | Run entitlement aggregation; check exact entitlement names |
| Timeout | Target system slow or VA overloaded | VA health, target system performance |

**The provisioning review queue**: When provisioning fails, ISC creates a work item in the Provisioning Review queue. Administrators see the failed items and can: retry the operation, manually provision (mark as complete), or close the request with a note.

---

## Connector-Specific Provisioning Quirks

Every connector has idiosyncrasies. A few common ones:

**Active Directory**: Account creation requires a Distinguished Name (DN) — the specific OU path where the account object is created. The provisioning policy must include DN derivation logic (typically based on department or employee type). New department added in the org? You need to create the corresponding OU in AD and update the DN derivation rule.

**Salesforce**: Profile assignment is required when creating a Salesforce user — you can't create an account without assigning a Profile. The access profile must include the Profile entitlement. Permission Sets are additive and can be added or removed independently.

**GitHub**: GitHub uses `login` (username) as the primary identifier, not email. If an identity's name changes, their existing GitHub login doesn't automatically update — GitHub treats login as semi-permanent. Provisioning policy should not attempt to update existing logins; work with GitHub admin to handle name changes manually.

**AWS**: ISC doesn't create AWS IAM users directly in most enterprise configurations — access is managed via SSO (AWS Identity Center). The "provisioning" operation is assigning permission sets via AWS Identity Center, not creating IAM users.

---

## Key Takeaways

- Provisioning policies specify the attribute values, operations, and conflict resolution rules ISC follows when creating or modifying target system accounts
- Attribute values come from four sources: identity attributes, transforms, static values, and generated values
- Verify supported plan operations (Create, Update, Enable, Disable, Delete) for each connector before designing lifecycle actions around them
- Conflict resolution (what happens when an account already exists) needs explicit design — don't leave it to defaults
- Missing required attributes cause provisioning failures — trace them back to authoritative source data quality
- Provisioning failures land in a review queue; operational teams need a process for monitoring and resolving them

> 🔗 **CONNECTS TO:** Provisioning policies govern how access is granted. Access certifications govern how access is reviewed and maintained over time — the ongoing process of verifying that every identity's access remains appropriate. The next document covers how to design certification campaigns that actually work.
