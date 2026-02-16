# Linux SOC Workbook – Command Reference & Log Analysis Notes

This README contains commonly used Linux commands and log locations for
SOC analysis, threat hunting, and incident response.

Focus areas:
- Log analysis
- Authentication monitoring
- User management
- Command execution tracking
- Package installation
- Kernel and system events
- Auditd investigations


## Important Log Files

- /var/log/kern.log  
  Kernel messages and errors (useful for driver issues, crashes, exploits)

- /var/log/syslog or /var/log/messages  
  Consolidated system events (general troubleshooting & threat hunting)

- /var/log/auth.log  
  Authentication, SSH, sudo, and session activity

- /var/log/dpkg.log or /var/log/apt  
  Package installation logs (Debian/Ubuntu)

- /var/log/dnf.log or /var/log/yum.log  
  Package installation logs (RHEL/CentOS/Fedora)

- /var/log/audit/audit.log  
  auditd logs for detailed command execution and security events

## Log Discovery

List available logs:
ls -l /var/log

Search for cron activity:
cat /var/log/syslog | grep cron

Search recursively for auth/login/session logs:
grep -R -E "auth|login|session" /var/log

## Authentication and SSH Monitoring

View SSH login success/failure:
cat /var/log/auth.log | grep "sshd" | grep -E "Accepted|Failed"

Track session creation and termination:
cat /var/log/auth.log | grep -E 'session opened|session closed'

## User & Account Changes

Detect password or user changes:
cat /var/log/auth.log | grep -E '(passwd|useradd|usermod|userdel)\['

List all system users:
cat /etc/passwd

Extract usernames only:
cut -d: -f1 /etc/passwd

Switch user (potential privilege escalation check):
sudo -i -u anotheruser

## Command Execution

Find sudo command execution:
cat /var/log/auth.log | grep -E 'COMMAND='

Review bash command history:
cat /home/ubuntu/.bash_history

## Package Installation Logs

Debian/Ubuntu:
cat /var/log/dpkg.log
cat /var/log/apt/history.log

RHEL/Fedora/CentOS:
cat /var/log/dnf.log
cat /var/log/yum.log

## Auditd Logs

View raw audit logs:
cat /var/log/audit/audit.log

Search audit events:
ausearch

Search for specific key (example: proc_wget):
ausearch -i -k proc_wget

## SOC Analyst Notes

- Authentication logs reveal brute force, lateral movement, and persistence
- Package logs reveal tool installation
- Bash history reveals attacker intent (when not wiped)
- auditd provides high-fidelity command execution telemetry
- Correlate timestamps across logs to reconstruct attacker timelines

- /var/log/auth.log is high-value for intrusion detection
- SSH logs reveal entry and lateral movement
- Package logs reveal tooling and persistence attempts
- auditd is essential for deep forensics
- Correlation across logs builds the full attack story
