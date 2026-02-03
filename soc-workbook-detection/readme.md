# SOC Workbook – Post-Exploitation Windows Threat Detection

This SOC workbook focuses on what happens **after initial access** on a
Windows host.

After breaching a system, threat actors must decide whether to:
- Maintain stealthy long-term access (persistence), or
- Act immediately to achieve their objectives

This workbook focuses on the **second approach**, covering activity that
typically follows Initial Access — beginning with **Discovery** and
progressing through **Collection**, **Exfiltration**, and **Ingress Tool Transfer**.

---

## Learning Objectives

By working through this workbook, you will learn how to:

- Detect common **Discovery** techniques using Windows Event Logs
- Reconstruct **process trees** to trace attack origin
- Identify what data attackers typically collect
- Detect **collection and exfiltration** techniques
- Understand and detect **ingress tool transfer**
- Observe how malicious commands are logged by executing them yourself

---

## Workbook Structure

### 1. Discovery
- Commands attackers use to enumerate a system
- How discovery appears in Windows logs
- Reconstructing attacker activity via process trees

### 2. Collection & Exfiltration
- Data attackers target
- Collection techniques
- Exfiltration methods
- Detection strategies

### 3. Ingress Tool Transfer
- How attackers bring tools into the environment
- Detection techniques using Windows logs

---

## Intended Audience

- SOC Analysts (Level 1–2)
- Blue Team learners
- TryHackMe / lab-based defenders
- Anyone learning Windows threat detection

---

## Disclaimer

This workbook is for **defensive security and educational purposes only**.
