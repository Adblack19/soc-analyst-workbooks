# SOC Analyst Workbook: Windows & Linux Discovery & Data Collection Commands (Detailed Explanations)

This workbook explains **exactly what each command does**, how it works internally, **why attackers use it**, and **what it means to a SOC analyst**. Think of this as *command-level threat intelligence*.

---

## 1. Files and Folders Discovery

**Goal:** Understand what the system is used for and locate sensitive data.

### `type <file>` (Windows CMD)

**What it does:**

* Reads the contents of a text file and prints it directly to the terminal.
* Does not open a GUI application.

**Why attackers use it:**

* Fast, quiet way to inspect logs, configs, or credentials.
* Commonly used to check `.txt`, `.log`, `.csv`, `.ini` files.

**SOC meaning:**

* File read activity without opening an editor is suspicious.
* Often paired with `findstr` to extract secrets.

---

### `Get-Content <file>` (PowerShell)

**What it does:**

* PowerShell version of `type`.
* Can read files line-by-line and pipe output to other commands.

**Why attackers use it:**

* More powerful for automation and scripting.
* Works well with filters, regex, and exports.

**SOC meaning:**

* Script Block Logging (Event ID 4104) is critical here.

---

### `dir <folder>` / `Get-ChildItem <folder>`

**What it does:**

* Lists files and subfolders in a directory.

**Why attackers use it:**

* Identify valuable files (documents, backups, credentials).
* Understand directory structure.

**SOC meaning:**

* Directory listing of user folders shortly after login = recon.

---

### Linux equivalents: `ls`, `cat`, `less`, `find`

**What they do:**

* Perform the same file enumeration and reading tasks.

**SOC meaning:**

* Monitor shell history and auditd logs.

---

## 2. Users and Groups Discovery

**Goal:** Identify accounts and privileges.

### `whoami`

**What it does:**

* Displays the current logged-in user account.

**Why attackers use it:**

* Confirm access level (user vs admin).

**SOC meaning:**

* Usually the first command after access.

---

### `net user`

**What it does:**

* Lists all local user accounts on the system.

**Why attackers use it:**

* Identify potential targets for credential theft.

**SOC meaning:**

* Rare for normal users to run.

---

### `net localgroup`

**What it does:**

* Displays local groups and their members.

**Why attackers use it:**

* Identify admin group membership.

**SOC meaning:**

* High-value discovery activity.

---

### Linux equivalents: `id`, `cat /etc/passwd`, `getent group`

**What they do:**

* Enumerate users and group privileges.

---

## 3. System and Installed Applications

**Goal:** Identify vulnerabilities and data-rich applications.

### `tasklist /v`

**What it does:**

* Lists running processes with verbose details.

**Why attackers use it:**

* Identify security tools, browsers, messengers.

**SOC meaning:**

* Process enumeration = mid-stage recon.

---

### `systeminfo`

**What it does:**

* Displays OS version, patch level, domain info.

**Why attackers use it:**

* Find unpatched systems.

**SOC meaning:**

* Strong signal of environment profiling.

---

### `wmic product get name,version`

**What it does:**

* Lists installed software from Windows Installer.

**Why attackers use it:**

* Identify exploitable software.

**SOC meaning:**

* WMIC usage is uncommon and suspicious.

---

### Linux equivalents: `ps aux`, `uname -a`, `apt list --installed`

---

## 4. Network Settings Discovery

**Goal:** Understand network layout.

### `ipconfig /all`

**What it does:**

* Displays IP address, DNS, gateway, domain.

**Why attackers use it:**

* Identify corporate networks and internal ranges.

---

### `netstat -ano`

**What it does:**

* Lists active connections and listening ports.

**Why attackers use it:**

* Identify C2 channels and services.

---

### Linux equivalents: `ip a`, `ss -tulpn`

---

## 5. Antivirus Discovery

### PowerShell WMI AV Query

**What it does:**

* Queries registered antivirus products.

**Why attackers use it:**

* Decide whether to evade or disable AV.

**SOC meaning:**

* One of the strongest malicious indicators.

---

## 6. Manual File Inspection

### `notepad.exe <file>`

**What it does:**

* Opens a file in a GUI editor.

**Why attackers use it:**

* Manual inspection of financial or sensitive files.

---

## 7. Keyword Searching

### `findstr password`

**What it does:**

* Searches files for specific keywords.

**Why attackers use it:**

* Extract credentials from logs.

---

### Linux: `grep -i password`

---

## 8. Recursive File Searches

### `Get-ChildItem -Recurse`

**What it does:**

* Searches directories recursively.

**Why attackers use it:**

* Locate documents at scale.

---

## 9. Data Collection

### `copy` / `cp -r`

**What it does:**

* Copies files to a staging location.

**Why attackers use it:**

* Prepare for exfiltration.

---

## 10. Data Archiving

### `Compress-Archive`, `7za`, `tar`, `zip`

**What they do:**

* Compress stolen data.

**Why attackers use it:**

* Reduce size and evade detection.

---

## 11. Shell Execution

### `start cmd.exe`

**What it does:**

* Launches a new command shell.

**Why attackers use it:**

* Maintain interactive control.

---

## Linux Equivalents – Detailed Explanations

Linux systems are heavily used for servers, cloud workloads, CI/CD, and developer machines. The same **post-compromise logic** applies, but the tooling is native Unix.

---

## 1. Linux File & Folder Discovery

### `ls <directory>`

**What it does:**

* Lists files and directories in the specified path.
* With flags like `-l`, `-a`, `-h`, it reveals permissions, owners, sizes, and hidden files.

**Why attackers use it:**

* Quickly understand directory contents.
* Identify interesting files (`.ssh`, `.env`, backups, configs).

**SOC meaning:**

* Frequent listing of home or `/etc` directories after login is suspicious.

---

### `cat <file>`

**What it does:**

* Outputs the entire content of a file to stdout.

**Why attackers use it:**

* Fast inspection of credentials, logs, config files.
* Often used on `.env`, `.bash_history`, `.ssh/config`.

**SOC meaning:**

* File read activity without editing = reconnaissance.

---

### `less <file>`

**What it does:**

* Opens a file in a pager, allowing scrolling.

**Why attackers use it:**

* Safer for large files without dumping everything to terminal.

**SOC meaning:**

* Indicates manual, interactive attacker activity.

---

### `find <path> -type f -name "*.ext"`

**What it does:**

* Recursively searches for files matching criteria.

**Why attackers use it:**

* Locate documents, keys, backups at scale.

**SOC meaning:**

* Recursive search across `/home` or `/var` is high-risk.

---

## 2. Linux Users & Privilege Discovery

### `whoami`

**What it does:**

* Displays current user identity.

**Why attackers use it:**

* Confirm access context.

---

### `id`

**What it does:**

* Shows user ID, group ID, and group memberships.

**Why attackers use it:**

* Check sudo/admin-equivalent access.

**SOC meaning:**

* Privilege checking immediately after access is classic attacker behavior.

---

### `cat /etc/passwd`

**What it does:**

* Lists all local user accounts.

**Why attackers use it:**

* Identify service and human users.

**SOC meaning:**

* Rarely needed in normal workflows.

---

### `getent group`

**What it does:**

* Displays all system groups and members.

**Why attackers use it:**

* Identify privileged groups (sudo, docker).

---

## 3. Linux Process & Application Discovery

### `ps aux`

**What it does:**

* Lists all running processes with user, PID, and command.

**Why attackers use it:**

* Detect security tools, browsers, databases.

**SOC meaning:**

* Full process enumeration is recon.

---

### `top` / `htop`

**What it does:**

* Real-time process monitoring.

**Why attackers use it:**

* Interactive analysis of system behavior.

---

### Package Enumeration

#### `apt list --installed`

#### `rpm -qa`

**What they do:**

* List installed software packages.

**Why attackers use them:**

* Identify vulnerable software.

**SOC meaning:**

* Package enumeration on servers is suspicious.

---

## 4. Linux System Information

### `uname -a`

**What it does:**

* Displays kernel version and architecture.

**Why attackers use it:**

* Identify kernel exploits.

---

### `lsb_release -a`

**What it does:**

* Displays OS distribution details.

---

## 5. Linux Network Discovery

### `ip a` / `ifconfig`

**What they do:**

* Display IP addresses and interfaces.

**Why attackers use them:**

* Identify internal networks.

---

### `ss -tulpn` / `netstat -tulpn`

**What they do:**

* List listening ports and connections.

**Why attackers use them:**

* Identify exposed services.

---

### Firewall Enumeration

#### `iptables -L`

#### `ufw status`

**What they do:**

* Display firewall rules.

**Why attackers use them:**

* Assess egress filtering.

---

## 6. Linux Security Tool Discovery

### `ps aux | grep <security_tool>`

**What it does:**

* Searches for EDR/AV processes.

**Why attackers use it:**

* Evaluate detection risk.

**SOC meaning:**

* Explicit AV hunting = high confidence threat.

---

## 7. Linux Keyword Searching

### `grep -i "password" <file>`

**What it does:**

* Searches text for matching patterns.

**Why attackers use it:**

* Extract credentials from logs and configs.

---

## 8. Linux Data Collection

### `cp -r <source> <dest>`

**What it does:**

* Recursively copies files.

**Why attackers use it:**

* Stage data for exfiltration.

---

## 9. Linux Data Archiving

### `tar -czvf`

### `zip -r`

**What they do:**

* Compress files into archives.

**Why attackers use them:**

* Reduce size and speed exfiltration.

---

## 10. Linux Shell Execution

### `/bin/bash`, `sh`

**What it does:**

* Spawns an interactive shell.

**Why attackers use it:**

* Maintain persistent control.

---

## Analyst Takeaway

These commands form a **classic post-compromise kill chain**:

1. Discovery
2. Privilege assessment
3. Data identification
4. Collection
5. Staging

Recognizing these early saves environments.
