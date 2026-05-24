# CyberDefenders-Reveal-Lab
Reveal | Endpoint Forensics | Easy | Volatility 3 

![Platform](https://img.shields.io/badge/Platform-CyberDefenders-blue)
![Category](https://img.shields.io/badge/Category-Endpoint%20Forensics-orange)
![Difficulty](https://img.shields.io/badge/Difficulty-Easy-green)
![Score](https://img.shields.io/badge/Score-100%25-brightgreen)
![Tool](https://img.shields.io/badge/Tool-Volatility%203-purple)

## Scenario
Reconstruct a multi-stage attack by analyzing Windows memory dumps using Volatility 3, identifying malicious processes, command lines, and correlating findings with threat intelligence.

## Tools Used
- Volatility 3
- MITRE ATT&CK Navigator
- VirusTotal

## Investigation

### Q1 — Malicious process name
**Answer:** `powershell.exe`

**Command:**
```bash
vol.py -f reveal.dmp windows.pslist
vol.py -f reveal.dmp windows.pstree
```
PowerShell is commonly abused as a living-off-the-land binary. Its presence in an unusual process tree is a strong indicator of compromise.

---

### Q2 — Parent PID of malicious process
**Answer:** `4120`

**Command:**
```bash
vol.py -f reveal.dmp windows.pstree
```
Tracing the PPID reveals what spawned the malicious PowerShell instance, helping reconstruct the infection chain.

---

### Q3 — Second-stage payload filename
**Answer:** `3435.dll`

**Command:**
```bash
vol.py -f reveal.dmp windows.cmdline
```
A DLL second-stage payload evades signature detection targeting .exe files.

---

### Q4 — Shared directory on remote server
**Answer:** `davwwwroot`

**Command:**
```bash
vol.py -f reveal.dmp windows.netstat
vol.py -f reveal.dmp windows.cmdline
```
`davwwwroot` is a WebDAV share — attackers use WebDAV to host and deliver payloads while bypassing network controls.

---

### Q5 — MITRE ATT&CK sub-technique
**Answer:** `T1218.011` — Signed Binary Proxy Execution: Rundll32

Rundll32.exe is a legitimate Windows binary abused to execute malicious DLLs, evading defenses under the Defense Evasion tactic.

---

### Q6 — Username of malicious process
**Answer:** `Elon`

**Command:**
```bash
vol.py -f reveal.dmp windows.getsids
```

---

### Q7 — Malware family
**Answer:** `StrelaStealer`



StrelaStealer is an infostealer targeting email client credentials (Outlook, Thunderbird), delivered via phishing and WebDAV payload staging.

---

## Attack Chain

Phishing email
→ PowerShell spawned (PPID: 4120)
→ Connects to WebDAV share (davwwwroot)
→ Downloads & executes 3435.dll via Rundll32 (T1218.011)
→ Steals email credentials from user: Elon
→ Exfiltrates to attacker C2

## Key Takeaways
- Memory forensics reveals activity invisible to live analysis
- LotL techniques blend malicious activity with legitimate Windows processes
- WebDAV is an effective delivery mechanism bypassing perimeter controls
- Always correlate IOCs with MITRE ATT&CK for full threat context
