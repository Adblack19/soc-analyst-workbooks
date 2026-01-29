# Sysmon Files, Network, and Registry – SOC Investigation Workflow

This document defines a complete SOC investigation workflow for monitoring
file creation, process access, registry modifications, network connections,
and DNS activity using Sysmon logs.

This workflow focuses on detecting:
- Malware execution
- Process injection
- Persistence mechanisms
- Command-and-Control (C2) communication

## Objective
- Detect malicious file creation and payload drops
- Identify suspicious registry-based persistence
- Monitor process-to-process access
- Detect outbound network and DNS activity
- Correlate Sysmon events into an attack timeline

## Log Source
- Sysmon (Microsoft Sysinternals)

## Sysmon Event IDs Covered
- **Event ID 3**  – Network Connection
- **Event ID 10** – Process Access
- **Event ID 11** – File Create
- **Event ID 13** – Registry Value Set
- **Event ID 22** – DNS Query

## Key Fields

Common:
- Image
- ProcessId
- User
- CommandLine

Event ID 3 (Network):
- DestinationIp
- DestinationPort
- Protocol

Event ID 10 (Process Access):
- SourceImage
- TargetImage
- GrantedAccess

Event ID 11 (File Create):
- TargetFilename
- Hashes

Event ID 13 (Registry Value Set):
- TargetObject
- Details

Event ID 22 (DNS Query):
- QueryName
- QueryResults

## SOC Workflow Overview

1. Detect suspicious file creation (Event ID 11)
2. Identify registry modifications for persistence (Event ID 13)
3. Detect suspicious process access (Event ID 10)
4. Monitor DNS queries (Event ID 22)
5. Inspect outbound network connections (Event ID 3)
6. Correlate events into a single attack chain
7. Decide, respond, and escalate

## Event ID 11 – File Create
Used to detect:
- Malware payload drops
- Script staging
- LOLBins writing executables
High-Risk Locations
- C:\Users\*\AppData\
- C:\Windows\Temp\
- C:\Temp\

🚩 Red Flag
Executable created in a user-writable directory

## Event ID 13 – Registry Value Set
Monitors registry value changes.
This event is critical for detecting persistence mechanisms.
High-Risk Registry Locations
- HKCU\Software\Microsoft\Windows\CurrentVersion\Run
- HKLM\Software\Microsoft\Windows\CurrentVersion\Run
- RunOnce keys
- Services registry paths

🚩 Critical Indicator

New executable added to Run or RunOnce registry key
➡ Indicates persistence established

## Event ID 10 – Process Access
Detects:
- Process injection
- Credential dumping
- Memory access abuse
High-Risk Targets
- lsass.exe
- winlogon.exe
- explorer.exe

🚩 Critical Indicator
Untrusted process accessing lsass.exe

## Event ID 22 – DNS Query
Used to detect:
- C2 beaconing
- DGA-based malware

- Suspicious domain lookups
🚩 Red Flag

Random-looking domains queried by newly created executables

## Event ID 3 – Network Connection
Monitors outbound and inbound connections initiated by processes.

🚩 High Confidence Indicator
File creation followed by outbound connection to external IP

## Correlation Logic
Common malicious chains:

- Event 11 → Event 13
  (Payload drop → persistence)

- Event 11 → Event 22 → Event 3
  (Execution → DNS → C2)

- Event 11 → Event 10 → Event 3
  (Execution → injection → outbound traffic)

- Event 13 → Event 3
  (Persistence → beaconing)
➡ Strong evidence of active compromise

## Decision Matrix

Benign:
- Known software installers
- Signed binaries
- Trusted domains

Malicious:
- Unknown hashes
- Persistence registry keys
- External IP connections
- Injection behavior

## Response Actions
- Isolate affected endpoint
- Remove persistence registry entries
- Quarantine malicious files
- Block IPs/domains
- Collect memory artifacts

## Escalation Criteria
Escalate if:
- Persistence confirmed
- C2 communication detected
- Credential access suspected
- Multiple endpoints affected

## Analyst Checklist
☐ Review file creation (ID 11)  
☐ Check registry persistence (ID 13)  
☐ Analyze process access (ID 10)  
☐ Review DNS queries (ID 22)  
☐ Inspect network connections (ID 3)  
☐ Correlate into attack timeline  
☐ Respond or escalate  

## Key Takeaways
- File creation shows delivery
- Registry changes reveal persistence
- Process access exposes injection
- DNS + network confirms attacker control
- Correlation = confidence
