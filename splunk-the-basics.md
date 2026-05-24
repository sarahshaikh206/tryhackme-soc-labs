# Splunk: The Basics

**Platform:** TryHackMe · SOC Level 1 Path — Core SOC Solutions  
**Room:** [Splunk: The Basics](https://tryhackme.com/room/splunk101)  
**Difficulty:** Easy  
**Status:** ✅ Completed

---

## Overview

Splunk is the most widely deployed SIEM platform in enterprise SOC environments. This room builds hands-on competency in Splunk's Search Processing Language (SPL) — the core skill for log correlation, anomaly detection, and alert investigation.

---

## Splunk Interface — Key Components

| Component | Purpose |
|-----------|---------|
| **Search & Reporting** | Primary workspace — SPL queries, results, visualisations |
| **Dashboards** | Pre-built and custom views for real-time monitoring |
| **Alerts** | Scheduled searches that fire notifications on threshold breach |
| **Reports** | Saved searches for recurring analysis |
| **Data Inputs** | Configuration of log source ingestion |

---

## SPL (Search Processing Language) — Core Syntax

### Basic Search Structure
```spl
index=<index_name> sourcetype=<log_type> <keyword> <field>=<value>
```

### Essential Commands

**Filtering**
```spl
index=windows_logs EventCode=4625
```
*Search Windows logs for failed login events (Event ID 4625)*

**Time Range**
```spl
index=proxy_logs earliest=-24h latest=now
```
*Limit results to the last 24 hours*

**Field-specific search**
```spl
index=auth_logs src_ip="192.168.1.100" action=failure
```

**Pipe operator — chain commands**
```spl
index=windows_logs EventCode=4625 | stats count by src_ip | sort -count
```
*Count failed logins grouped by source IP, sorted descending*

---

## Key SPL Commands

### `stats` — Aggregate and count
```spl
index=firewall_logs | stats count by dest_ip | sort -count | head 10
```
*Top 10 destination IPs by connection count*

### `table` — Format results as a table
```spl
index=auth_logs | table _time, user, src_ip, action
```

### `where` — Filter based on conditions
```spl
index=auth_logs | stats count(action=failure) as fails by user | where fails > 10
```
*Users with more than 10 failed logins*

### `eval` — Create calculated fields
```spl
index=web_logs | eval response_kb = response_bytes / 1024
```

### `rex` — Extract fields using regex
```spl
index=email_logs | rex field=raw_message "From: (?<sender_email>[^\s]+)"
```

### `timechart` — Time-series visualisation
```spl
index=windows_logs EventCode=4625 | timechart count span=1h
```
*Failed logins per hour — useful for spotting brute force timing patterns*

### `transaction` — Group related events
```spl
index=auth_logs | transaction user maxspan=5m | where eventcount > 5
```
*Group auth events per user within 5-minute windows*

---

## Lab Exercises Completed

### Exercise 1 — Failed Login Analysis (Brute Force Detection)

**Objective:** Identify accounts under brute force attack in Windows security logs

```spl
index=windows_logs EventCode=4625 
| stats count by Account_Name, src_ip 
| where count > 20 
| sort -count
```

**Finding:** One account had 847 failed login attempts from a single external IP within a 2-hour window, followed by a successful login (EventCode=4624). Classic brute force → successful compromise pattern.

**Kill Chain Stage:** Exploitation (Credential Access — T1110 Brute Force)

---

### Exercise 2 — Anomalous Authentication Detection

**Objective:** Surface logins outside normal business hours (off-hours access)

```spl
index=auth_logs action=success 
| eval hour=strftime(_time, "%H") 
| where hour < 7 OR hour > 19 
| table _time, user, src_ip, hour
```

**Finding:** Three service accounts logged in between 02:00 and 04:00 — outside normal business hours. Flagged for review — potential attacker using compromised service account credentials.

---

### Exercise 3 — Lateral Movement Indicators

**Objective:** Detect a user authenticating to hosts they have never accessed before

```spl
index=auth_logs action=success 
| stats dc(dest_host) as unique_hosts by user 
| where unique_hosts > 10
```

**Finding:** One user account accessed 23 unique internal hosts within 4 hours — far outside normal behaviour. Indicator of lateral movement post-compromise.

**Kill Chain Stage:** Lateral Movement (T1021 — Remote Services)

---

### Exercise 4 — C2 Beaconing Detection

**Objective:** Identify regular-interval outbound connections to new/rare domains

```spl
index=proxy_logs 
| stats count, dc(dest_domain) as domains, earliest(_time) as first_seen by src_ip 
| where domains > 1 AND count > 100
```

**Finding:** One internal host making HTTPS connections to a domain first registered 3 days prior, exactly every 60 seconds. Classic C2 beaconing pattern.

**Kill Chain Stage:** Command & Control (T1071.001 — Web Protocols)

---

## Splunk Alert Configuration

In the lab, configured a saved alert for brute force detection:

- **Search:** `index=auth_logs EventCode=4625 | stats count by src_ip | where count > 50`
- **Schedule:** Run every 15 minutes
- **Condition:** Number of results > 0
- **Alert action:** Send email to SOC queue + create notable event in Incident Review

---

## Key Takeaways

- **SPL is the most valuable Splunk skill** — complex investigations reduce to well-constructed queries
- **`stats count by` is the workhorse command** — almost every detection starts with aggregation
- **Time-series analysis (`timechart`) reveals patterns** invisible in raw event lists
- **Brute force detection requires a time window** — raw failed login counts without time context miss the pattern
- **Pivot fast** — start broad, filter down. Never start with an overly specific query.
- **Saved searches and alerts turn one-time investigations into continuous detection coverage**

---

## Common Event IDs (Windows) for SOC

| Event ID | Event | SOC Relevance |
|---------|-------|--------------|
| 4624 | Successful logon | Baseline + anomaly detection |
| 4625 | Failed logon | Brute force detection |
| 4648 | Explicit credential logon | Pass-the-hash / lateral movement |
| 4672 | Special privileges assigned | Privilege escalation |
| 4688 | Process creation | Execution detection |
| 4698 | Scheduled task created | Persistence |
| 4720 | User account created | Account creation monitoring |
| 4740 | Account lockout | Brute force side effect |

---

## Tools Referenced

- `Splunk Enterprise` — primary SIEM platform
- `Splunk Search & Reporting` — SPL query interface
- `Windows Event Logs` — primary data source for authentication and execution events
