# Windows RDP Logon Investigation Workflow (Event IDs 4625 & 4624)

This document defines a complete SOC investigation workflow for Windows
Remote Desktop Protocol (RDP) activity using Windows Security Event IDs
4625 (failed logon) and 4624 (successful logon).

The workflow is designed to always begin with failed logons (4625) to
quickly identify brute-force and unauthorized access attempts before
pivoting to successful logons (4624).

## Objective

- Detect RDP brute-force and password-spraying attacks
- Identify successful access following repeated failures
- Reduce analysis time by prioritizing failed logons
- Standardize RDP investigations for SOC operations

## Scope and Data Sources

Log Source:
- Windows Security Event Log

Event IDs:
- 4625 – Failed logon (primary detection signal)
- 4624 – Successful logon (confirmation signal)

## SOC Investigation Workflow

Step 1: Identify failed logons (4625)
Step 2: Validate RDP context (Logon Type 10)
Step 3: Analyze brute-force patterns
Step 4: Pivot to successful logons (4624)
Step 5: Correlate failures with success
Step 6: Assess impact and privilege level
Step 7: Respond and escalate if required

## Step 1: Analyze Failed Logons (Event ID 4625)

Event ID 4625 is generated when a logon attempt fails. This is the
primary signal for detecting brute-force, password spraying, and
unauthorized RDP access attempts.

- Account Name
- Logon Type
- Source Network Address
- Failure Reason
- Status / SubStatus
- Time of Attempt
If Logon Type = 10, the failed attempt is RDP-related.

## Brute-Force Detection Logic

Indicators:
- High volume of 4625 events
- Logon Type = 10
- Same source IP targeting one or multiple accounts
- Short time window

Analyst Questions:
- How many failures occurred?
- Are multiple usernames being targeted?
- Is the source IP external or unusual?
- Did activity occur outside business hours?

RDP Identification:
- Logon Type = 10 (RemoteInteractive)

## Step 2: Review Successful Logons (Event ID 4624)

Event ID 4624 confirms successful authentication. This step is performed
only after suspicious failed logons have been identified.
- Account Name
- Logon Type
- Source Network Address
- Authentication Package
- Logon Process
- Time of Logon
4624 + Logon Type = 10 confirms successful RDP access.

## Step 3: Correlate Failed and Successful Logons

Correlation Logic:
1. Identify multiple 4625 events (Logon Type 10)
2. Group by source IP and username
3. Identify a subsequent 4624 event
4. Confirm same source IP or account
5. Measure time gap between failure and success

## Step 4: Risk Assessment

Assess the following:
- Is the account privileged?
- Is the host a server or critical system?
- Is the source IP external or high-risk?
- Did login occur outside normal hours?
- Were multiple systems accessed?

## Decision Matrix

Benign Indicators:
- Low number of failures
- Known internal IPs
- Expected remote access patterns
- User confirms activity

Malicious Indicators:
- High-volume failures
- Multiple usernames targeted
- External IP addresses
- Successful logon after failures
- Privileged account involvement

## Response Actions

- Validate activity with user or system owner
- Reset or disable compromised account
- Block malicious IP address
- Enforce MFA for RDP access if available
- Isolate host if post-access activity is suspicious

## Escalation Criteria

Escalate immediately if:
- Privileged account accessed
- Multiple hosts compromised
- Evidence of lateral movement
- Post-RDP suspicious activity observed

## Investigation Checklist

☐ Review 4625 event volume  
☐ Confirm Logon Type = 10  
☐ Identify source IP  
☐ Identify targeted account(s)  
☐ Pivot to 4624 if needed  
☐ Correlate failure-to-success  
☐ Assess privilege and impact  
☐ Execute response actions  

## Key Takeaways

- Always start with 4625 for early attack detection
- Correlation reveals compromise, not isolated events
- RDP access is high-risk due to interactive control
- Structured workflows reduce SOC fatigue and errors

