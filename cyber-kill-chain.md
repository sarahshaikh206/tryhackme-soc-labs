# Cyber Kill Chain

**Platform:** TryHackMe · SOC Level 1 Path — Cyber Defence Frameworks  
**Room:** [Cyber Kill Chain](https://tryhackme.com/room/cyberkillchainzmt)  
**Difficulty:** Easy  
**Status:** ✅ Completed

---

## Overview

The Cyber Kill Chain is a framework developed by Lockheed Martin that models a cyberattack as a sequence of seven stages. Understanding this model allows SOC analysts to identify *where in the attack lifecycle* an alert sits, enabling faster, more targeted response.

> **Core principle:** Disrupting any stage of the kill chain can break the entire attack chain.

---

## The Seven Stages

### 1. Reconnaissance
Attacker gathers intelligence on the target before any active intrusion.

- **Passive:** OSINT — LinkedIn, Shodan, DNS records, WHOIS, job postings
- **Active:** Port scanning, service enumeration
- **SOC Relevance:** Hard to detect — occurs outside your perimeter. Threat intel feeds and honeypots help.
- **Defender action:** Minimise public footprint, monitor for scanning activity on perimeter

---

### 2. Weaponisation
Attacker creates or acquires a malicious payload suited to the target.

- Malware is paired with an exploit (e.g. macro-enabled Word doc exploiting CVE)
- May include a dropper, RAT, or ransomware payload
- **SOC Relevance:** Occurs entirely off-network — not directly observable
- **Defender action:** Patch management, application whitelisting, disable macros by policy

---

### 3. Delivery
The weapon is transmitted to the target.

- **Vectors:** Phishing email, malicious USB, drive-by download, watering hole attack
- **SOC Relevance:** First stage visible to defenders — email gateway, web proxy logs
- **Key IOCs:** Sender IP, Reply-To mismatch, malicious attachment hash, malicious URL
- **Defender action:** Email filtering, SPF/DKIM/DMARC enforcement, user awareness training

---

### 4. Exploitation
The payload executes, exploiting a vulnerability or user action.

- May exploit unpatched software (CVE), misconfiguration, or user clicking a link
- **SOC Relevance:** EDR/AV alerts fire here — process creation, script execution, privilege escalation attempts
- **Key IOCs:** Unusual parent-child process relationships, PowerShell execution, suspicious file writes
- **Defender action:** Patch management, EDR with behavioural detection, script blocking

---

### 5. Installation
Attacker installs a persistent backdoor to maintain access.

- Techniques: Registry run keys, scheduled tasks, DLL hijacking, service installation
- **SOC Relevance:** Windows Event Logs (4697 — service install, 4698 — scheduled task)
- **Key IOCs:** New services, persistence registry keys, unusual startup entries
- **Defender action:** Monitor privileged operations, baseline normal behaviour

---

### 6. Command & Control (C2)
Malware beacons out to attacker-controlled infrastructure.

- Protocols: HTTP/S, DNS tunnelling, ICMP, social media APIs
- **SOC Relevance:** Network monitoring — unusual outbound connections, beaconing patterns
- **Key IOCs:** Regular interval outbound connections, connections to new/rare domains, large DNS responses
- **Defender action:** DNS filtering, proxy inspection, network anomaly detection in SIEM

---

### 7. Actions on Objectives
Attacker achieves their goal — data exfiltration, ransomware deployment, lateral movement.

- **SOC Relevance:** By this stage, detection has failed — focus shifts to containment and eradication
- **Key IOCs:** Large data transfers, encryption activity, credential dumping (Mimikatz), lateral movement
- **Defender action:** Data Loss Prevention (DLP), network segmentation, IR playbook activation

---

## Kill Chain Mapping Applied

From my phishing analysis labs, I mapped real phishing samples to kill chain stages:

| Sample | Stage | Observable |
|--------|-------|-----------|
| Spoofed HR email with malicious .docm | Delivery | Sender domain mismatch, attachment extension |
| Macro triggers PowerShell download | Exploitation | PowerShell child process under WINWORD.EXE |
| Dropped .exe writes to AppData | Installation | Unusual file write to non-standard path |
| Outbound HTTPS to rare domain every 60s | C2 | Beaconing pattern in proxy logs |

---

## Key Takeaways

- **Every alert has a kill chain stage** — knowing the stage shapes your triage and escalation decision
- **Earlier detection = less damage** — stopping at Delivery prevents Exploitation entirely
- **Delivery + Exploitation are the highest-value detection points** for a Tier 1 SOC analyst
- The Kill Chain complements the **Pyramid of Pain**: Kill Chain tells you *where*, Pyramid tells you *what to block* for maximum impact

---

## Tools Referenced

- `MITRE ATT&CK` — maps kill chain stages to specific techniques
- `Splunk` — log correlation across kill chain stages
- `VirusTotal` — payload analysis at weaponisation/delivery stages
- `Wireshark` — C2 traffic identification
