# Introduction to SIEM

**Platform:** TryHackMe · SOC Level 1 Path — Core SOC Solutions  
**Room:** [Introduction to SIEM](https://tryhackme.com/room/introtosiem)  
**Difficulty:** Easy  
**Status:** ✅ Completed

---

## Overview

A Security Information and Event Management (SIEM) system is the central nervous system of a SOC. It aggregates, normalises, and correlates log data from across an organisation's infrastructure to surface security alerts that would be impossible to detect from any single source.

---

## What is a SIEM?

A SIEM combines two capabilities:

| Component | Function |
|-----------|---------|
| **SIM** (Security Information Management) | Long-term log storage, compliance reporting, historical analysis |
| **SEM** (Security Event Management) | Real-time event correlation, alerting, dashboard monitoring |

Together they enable: **detect → alert → investigate → respond**

---

## Core SIEM Functions

### 1. Log Collection & Ingestion
SIEM ingests logs from across the environment via:
- **Agents** — lightweight software installed on endpoints that forward logs
- **Agentless** — syslog, WMI, API-based log forwarding
- **Network taps / SPAN ports** — network traffic mirroring

**Common log sources:**
- Windows Event Logs (Security, System, Application)
- Linux syslog / auth.log
- Firewall and proxy logs
- DNS query logs
- Active Directory / LDAP logs
- Cloud platform logs (AWS CloudTrail, Azure Monitor)
- Email gateway logs
- EDR/AV alerts

### 2. Normalisation
Raw logs from different systems have different formats. The SIEM normalises them into a common schema so they can be correlated.

Example: A Windows login event and a Linux SSH login event both get normalised to a common `authentication` event type with fields like `user`, `source_ip`, `timestamp`, `outcome`.

### 3. Correlation Rules
Pre-built logic that identifies suspicious patterns across normalised data.

**Example rules:**
- `5 failed logins from same IP within 60 seconds` → Brute force alert
- `Login from country never seen for this account` → Impossible travel alert
- `New admin account created outside business hours` → Privilege escalation alert
- `Outbound connection to known C2 IP` → Malware C2 alert

### 4. Alerting & Dashboards
- Alerts fire when correlation rules trigger
- Dashboards provide real-time visibility of alert volume, severity, and trends
- Analysts work a queue of alerts, triaging each as True Positive, False Positive, or Benign

### 5. Search & Investigation
Analysts can search across all ingested logs using the SIEM's query language (e.g. SPL in Splunk) to:
- Investigate specific events in detail
- Build timelines of attacker activity
- Hunt for indicators not yet in correlation rules

---

## SIEM Architecture

```
[Log Sources] → [Collection Layer] → [SIEM Engine]
                                          ↓
                              [Normalisation & Parsing]
                                          ↓
                              [Correlation & Rules Engine]
                                          ↓
                              [Alerting → Analyst Queue]
                                          ↓
                              [Dashboards & Search]
```

---

## Common SIEM Platforms

| Platform | Notes |
|---------|-------|
| **Splunk** | Industry leader — powerful SPL query language, extensive marketplace |
| **Elastic (ELK Stack)** | Open-source — Elasticsearch, Logstash, Kibana |
| **Microsoft Sentinel** | Cloud-native — integrates natively with Azure/M365 |
| **IBM QRadar** | Enterprise-grade, strong compliance features |
| **ArcSight** | Mature platform, common in large enterprises |

---

## Alert Triage Workflow

```
Alert fires in SIEM
    ↓
Analyst reviews alert details
    ↓
Pivot to raw logs for context
    ↓
Classify: True Positive / False Positive / Benign
    ↓
True Positive → Document IOCs → Escalate to Tier 2
False Positive → Document reason → Update rule tuning notes
```

---

## SIEM Use Cases Practised in Labs

| Use Case | Log Source | Detection |
|----------|-----------|----------|
| Brute force detection | Windows Security Log (Event ID 4625) | 5+ failed logins → 1 success |
| Lateral movement | Authentication logs + network logs | Internal login to previously untouched host |
| C2 communication | Proxy/firewall logs | Regular interval outbound to new domain |
| Privilege escalation | Windows Security Log (Event ID 4672) | Special privilege assigned to new account |
| Anomalous authentication | All auth logs | Login from new country or unusual hours |

---

## Key Takeaways

- **SIEM is only as good as its log sources** — gaps in coverage = blind spots for attackers
- **Correlation rules require tuning** — poorly tuned rules create alert fatigue (too many false positives)
- **Context is everything** — a single failed login is noise; 500 failed logins followed by a success is a critical alert
- **Log normalisation enables cross-source correlation** — the real power of a SIEM is connecting dots across different systems
- **Tier 1 analysts live in the alert queue** — speed, accuracy, and documentation discipline are the core job skills

---

## Tools Referenced

- `Splunk` — practised in adjacent Splunk: The Basics room
- `Elastic Stack` — practised in Elastic: The Basics room
- `Windows Event Viewer` — understanding raw log format before SIEM ingestion
