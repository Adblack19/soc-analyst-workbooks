# Windows Security Log – User Management Investigation Workflow

This document defines a complete SOC investigation workflow for monitoring
and analyzing Windows user and group management activities using the
Windows Security Event Log.

The workflow covers user account lifecycle events, password changes and
resets, and group membership changes to detect persistence, privilege
escalation, and account compromise.

## Objective

- Detect unauthorized user account creation or deletion
- Detect suspicious password changes or resets
- Identify unauthorized account enablement or modification
- Detect privilege escalation via group membership changes
- Standardize SOC investigations involving identity abuse

## Scope and Data Sources

Log Source:
- Windows Security Event Log

Threat Focus:
- Account compromise
- Persistence
- Privilege escalation
- Defense evasion

## Key Windows User Management Event IDs

### User Account Lifecycle
- **4720** – User account created
- **4722** – User account enabled
- **4725** – User account disabled
- **4726** – User account deleted
- **4738** – User account changed

### Password Management
- **4723** – User attempted to change password
- **4724** – Password reset attempt

### Group Membership Changes
- **4732** – User added to a local security group
- **4733** – User removed from a local security group
- **4728** – User added to a global security group
- **4729** – User removed from a global security group
Password reset (4724) performed by someone else is higher risk than a user changing their own password (4723).

## SOC Investigation Workflow

Step 1: Identify user management event
Step 2: Determine action type (create, modify, password, group change)
Step 3: Identify target account
Step 4: Identify actor account
Step 5: Assess privilege impact
Step 6: Correlate with authentication activity
Step 7: Determine legitimacy
Step 8: Respond and escalate if required

## Step 1: User Account Creation and Deletion

Account creation and deletion are strong indicators of persistence
or attacker cleanup activity.

Events Reviewed
- 4720 – User account created
- 4726 – User account deleted

Analyst Questions
- Who performed the action?
- Is the account name suspicious or generic?
- Was the action performed outside business hours?
- Does this align with approved change activity?

## Step 2: Password Change and Reset

Password-related events are critical indicators of account takeover,
persistence, or attacker remediation actions.

## Events Reviewed
- 4723 – User attempted to change their own password
- 4724 – Password reset attempt (performed by another account)

## Analyst Questions
- Who initiated the password change or reset?
- Was the target account compromised earlier?
- Was the password reset performed by an admin?
- Does the user confirm this action?

### 🚩High-Risk Indicators

Password reset (4724) without a helpdesk ticket
Password reset followed by privileged group addition
Password reset followed by RDP or VPN login

## Step 3: Account Enablement and Modification

Attackers may enable dormant accounts or modify account attributes
to maintain access.

## Events Reviewed
- 4722 – User account enabled
- 4725 – User account disabled
- 4738 – User account changed

## Analyst Questions
- Was a previously disabled account re-enabled?
- Were attributes like UPN or display name changed?
- Is this behavior normal for this account?

## Step 4: Group Membership Changes

Group membership changes are one of the strongest indicators of
privilege escalation.

## High-Risk Groups
- Administrators
- Domain Admins
- Enterprise Admins
- Remote Desktop Users
- Backup Operators

## Events Reviewed
- 4732 / 4728 – User added to group
- 4733 / 4729 – User removed from group

## Step 5: Correlation Logic

Suspicious patterns include:
- 4720 → 4724 → 4732 (new account + password reset + admin rights)
- 4724 followed by RDP logon (4624)
- 4738 followed by privileged group addition
- Password reset shortly after brute-force attempts (4625)
Strong indicators of account compromise or persistence

## Step 6: Risk Assessment

Assess:
- Privilege level of target account
- Privilege level of actor account
- Time of activity
- Host or domain scope
- Correlation with other alerts

## Decision Matrix

Benign Indicators:
- Known admin or helpdesk activity
- Approved change requests
- Business-hours activity
- User confirmation

Malicious Indicators:
- No approval or ticket
- Off-hours password reset
- Privileged group assignment
- Correlation with suspicious logons

## Response Actions

- Disable or lock affected account
- Reset credentials securely
- Remove unauthorized group memberships
- Review systems accessed by account
- Preserve logs for forensic investigation

## Escalation Criteria

Escalate immediately if:
- Privileged accounts are affected
- Password resets occur after suspicious logons
- Multiple accounts are impacted
- Domain-level changes are observed

## Investigation Checklist

☐ Identify user management event ID  
☐ Identify target account  
☐ Identify actor account  
☐ Review password-related activity  
☐ Assess privilege changes  
☐ Correlate with logon events  
☐ Validate legitimacy  
☐ Execute response actions  

## Key Takeaways

- Password resets are high-risk identity events
- 4724 is more suspicious than 4723
- Privilege escalation often follows password changes
- Correlation is essential for accurate detection

