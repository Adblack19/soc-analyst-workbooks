# Sysmon Process Monitoring – SOC Investigation Workflow

This document defines a complete SOC investigation workflow for monitoring
and analyzing process creation activity using Sysmon logs.

The workflow focuses on detecting malicious execution, living-off-the-land
abuse, and post-compromise activity through process telemetry.

## Objective

- Detect suspicious or malicious process execution
- Identify attacker tradecraft using legitimate binaries (LOLBins)
- Correlate process execution with user and logon activity
- Standardize SOC investigation of endpoint execution events

## Scope and Data Sources

Log Source:
- Sysmon (Microsoft Sysinternals)

Primary Event:
- **Event ID 1** – Process Creation

Threat Focus:
- Initial execution
- Post-exploitation activity
- Persistence preparation
- Lateral movement preparation

## Key Sysmon Fields (Event ID 1)

- Image
- CommandLine
- ParentImage
- ParentCommandLine
- User
- IntegrityLevel
- CurrentDirectory
- Hashes (MD5, SHA256)

📌 SOC Rule:
Command line + parent process = context

## SOC Investigation Workflow

Step 1: Identify process creation event
Step 2: Analyze process and command line
Step 3: Review parent-child relationship
Step 4: Assess execution context
Step 5: Correlate with authentication activity
Step 6: Determine malicious or benign
Step 7: Respond and escalate if required

## Step 1: Process Execution Analysis

Process creation events provide visibility into what is executed
on a system and how it was launched.
High-Risk Process Categories
- PowerShell (powershell.exe)
- Command Prompt (cmd.exe)
- Windows Script Host (wscript.exe, cscript.exe)
- Rundll32
- Mshta
- Certutil

🚩 Red Flag:
Execution of admin tools by standard users

## Step 2: Command Line Analysis

Attackers often hide malicious intent in command-line arguments.
Suspicious Indicators
- Encoded commands
- Download from external URLs
- Execution from user-writable directories
- Obfuscated or overly long command lines

🚩 Critical Indicator
- PowerShell with -EncodedCommand

## Step 3: Parent-Child Analysis

Understanding how a process was spawned provides context
on execution method.
Suspicious Relationships
- Office application → PowerShell
- Browser → Command shell
- Explorer → Script engine
➡ Common in phishing-based attacks

## Step 4: Execution Context

Evaluate:
- User account
- Integrity level
- Current directory
- Execution timing

🚩 Red Flags
- High integrity execution by non-admin user
- Execution during off-hours
- Execution shortly after RDP logon

## Correlation Logic

Suspicious sequences include:
- RDP logon (4624) → Sysmon Event ID 1
- Password reset → process execution
- Office macro → PowerShell execution
➡ Indicates post-compromise activity

## Decision Matrix

Benign Indicators:
- Known application paths
- Expected parent processes
- Approved scripts
- Admin activity during maintenance

Malicious Indicators:
- LOLBins with suspicious arguments
- Obfuscated command lines
- Unusual parent-child chains
- Execution from temp or user directories

## Response Actions

- Terminate malicious process
- Isolate affected endpoint
- Block malicious hashes
- Review child processes
- Collect memory or disk artifacts

## Escalation Criteria

Escalate if:
- Malicious execution confirmed
- Privileged context involved
- Multiple endpoints affected
- Correlation with other attack stages
## Investigation Checklist

☐ Identify process and command line  
☐ Review parent process  
☐ Validate execution context  
☐ Check hash reputation  
☐ Correlate with logon activity  
☐ Determine intent  
☐ Execute response actions  

## Key Takeaways

- Process creation is a core endpoint detection signal
- Command-line visibility is critical for detection
- Parent-child relationships expose execution methods
- Correlation confirms attacker intent
