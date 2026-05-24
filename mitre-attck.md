# MITRE ATT&CK

**Platform:** TryHackMe · SOC Level 1 Path — Cyber Defence Frameworks  
**Room:** [MITRE](https://tryhackme.com/room/mitre)  
**Difficulty:** Easy  
**Status:** ✅ Completed

---

## Overview

MITRE ATT&CK (Adversarial Tactics, Techniques & Common Knowledge) is a globally accessible knowledge base of adversary behaviour based on real-world observations. It is the industry standard for describing, classifying, and communicating attacker TTPs.

---

## Core Structure

### Tactics (The "Why")
High-level adversary goals — 14 enterprise tactics including:

| Tactic | ID | Description |
|--------|----|-------------|
| Reconnaissance | TA0043 | Gathering info before attack |
| Initial Access | TA0001 | Getting into the environment |
| Execution | TA0002 | Running malicious code |
| Persistence | TA0003 | Maintaining foothold |
| Privilege Escalation | TA0004 | Gaining higher-level permissions |
| Defence Evasion | TA0005 | Avoiding detection |
| Credential Access | TA0006 | Stealing credentials |
| Discovery | TA0007 | Learning about the environment |
| Lateral Movement | TA0008 | Moving through the network |
| Collection | TA0009 | Gathering data of interest |
| Command & Control | TA0011 | Communicating with compromised systems |
| Exfiltration | TA0010 | Stealing data |
| Impact | TA0040 | Disrupting/destroying systems |

### Techniques (The "How")
Specific methods used to achieve a tactic. Each technique has a unique ID (e.g. T1566 — Phishing).

### Sub-techniques
Further granularity within techniques (e.g. T1566.001 — Spearphishing Attachment).

### Procedures
Specific real-world implementations of a technique by known threat actors.

---

## Key Techniques Relevant to SOC L1

### T1566 — Phishing
- **Sub-techniques:** .001 Attachment, .002 Link, .003 Spearphishing via service
- **Detection:** Email gateway logs, suspicious attachment types, header anomalies
- **Applied in:** Phishing Analysis labs — mapped each sample to T1566.001

### T1059 — Command and Scripting Interpreter
- **Sub-techniques:** .001 PowerShell, .003 Windows Command Shell, .007 JavaScript
- **Detection:** Script block logging, process creation events (Event ID 4688)
- **Applied in:** Malware triage scenarios — PowerShell spawned by Office process

### T1078 — Valid Accounts
- **Detection:** Authentication logs, impossible travel, off-hours logins
- **Applied in:** SIEM log correlation — anomalous authentication events

### T1071 — Application Layer Protocol (C2)
- **Sub-techniques:** .001 Web Protocols (HTTP/S), .004 DNS
- **Detection:** Proxy logs, DNS query volume anomalies, beaconing patterns

### T1110 — Brute Force
- **Detection:** Multiple failed auth attempts (Event ID 4625), account lockouts (4740)
- **Applied in:** Splunk lab — surfaced brute force pattern in authentication logs

---

## MITRE ATT&CK Navigator

The ATT&CK Navigator is a web-based tool for annotating and visualising the ATT&CK matrix.

**Use cases in a SOC:**
- Heat-mapping which techniques your detections cover
- Identifying coverage gaps in SIEM rules
- Mapping a threat actor's known TTPs to understand exposure
- Comparing your defensive coverage against a specific APT group

---

## Threat Actor Example — APT28 (Fancy Bear)

| Technique | ID | Description |
|-----------|----|-------------|
| Spearphishing Attachment | T1566.001 | Primary initial access method |
| PowerShell | T1059.001 | Execution and download |
| Credential Dumping | T1003 | Post-exploitation |
| Exfiltration over C2 | T1041 | Data theft |

---

## Key Takeaways

- **ATT&CK is the common language of threat intelligence** — use technique IDs in incident reports for precision and portability
- **Coverage mapping is a force multiplier** — knowing where your SIEM has gaps lets you prioritise detection engineering
- **Every phishing email and malware alert maps to at least one ATT&CK technique** — tagging alerts accelerates escalation and pattern recognition
- ATT&CK + Kill Chain + Pyramid of Pain together provide a complete operational picture of any threat

---

## Tools Referenced

- [ATT&CK Navigator](https://mitre-attack.github.io/attack-navigator/) — matrix visualisation and coverage mapping
- [attack.mitre.org](https://attack.mitre.org) — full technique reference database
- `Splunk` — ATT&CK technique tagging in alert rules
