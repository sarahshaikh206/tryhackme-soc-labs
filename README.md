# 🛡️ TryHackMe — SOC Analyst Lab Portfolio

**Sarah Shaikh · Entry-Level SOC Analyst**  
[![TryHackMe](https://img.shields.io/badge/TryHackMe-sarahsiraj206-red?style=flat&logo=tryhackme)](https://tryhackme.com/p/sarahsiraj206)
[![Path](https://img.shields.io/badge/Path-SOC%20Level%201-blue?style=flat)](https://tryhackme.com/path/outline/soclevel1)
[![Labs](https://img.shields.io/badge/Labs%20Completed-15%2B-green?style=flat)]()

---

## About This Repository

This repository documents my hands-on blue team training completed on the **TryHackMe** platform across the **Pre Security** and **SOC Level 1** learning paths. Each folder contains structured write-ups, key takeaways, tool usage, and mapped adversary frameworks for every lab completed.

> **Focus Areas:** Alert triage · Phishing analysis · SIEM log correlation · Malware classification · Cyber Kill Chain · Pyramid of Pain · MITRE ATT&CK

---

## 📁 Repository Structure

```
tryhackme-soc-labs/
│
├── 01-cyber-defence-frameworks/
│   ├── pyramid-of-pain.md
│   ├── cyber-kill-chain.md
│   └── mitre-attck.md
│
├── 02-phishing-analysis/
│   ├── phishing-analysis-fundamentals.md
│   ├── phishing-emails-in-action.md
│   ├── phishing-analysis-tools.md
│   └── the-greenholt-phish.md
│
├── 03-siem-and-splunk/
│   ├── intro-to-siem.md
│   └── splunk-the-basics.md
│
├── 04-network-traffic-analysis/
│   ├── wireshark-the-basics.md
│   └── wireshark-traffic-analysis.md
│
├── 05-malware-triage/
│   ├── malware-classification.md
│   └── intro-to-malware-analysis.md
│
└── README.md
```

---

## 🧪 Labs Completed

### Module 1 — Cyber Defence Frameworks
| Room | Key Skills | THM Link |
|------|-----------|----------|
| Pyramid of Pain | IOC types, attacker adaptability, detection durability | [🔗](https://tryhackme.com/room/pyramidofpainax) |
| Cyber Kill Chain | 7-stage attack model, mapping TTPs to defences | [🔗](https://tryhackme.com/room/cyberkillchainzmt) |
| MITRE ATT&CK | Tactics, Techniques & Procedures, ATT&CK Navigator | [🔗](https://tryhackme.com/room/mitre) |

### Module 2 — Phishing Analysis
| Room | Key Skills | THM Link |
|------|-----------|----------|
| Phishing Analysis Fundamentals | Email header inspection, spoofed domains, X-headers | [🔗](https://tryhackme.com/room/phishingemails1tryoe) |
| Phishing Emails in Action | Live phishing sample dissection, IOC extraction | [🔗](https://tryhackme.com/room/phishingemails2rytmuv) |
| Phishing Analysis Tools | PhishTool, MXToolbox, VirusTotal, URLScan | [🔗](https://tryhackme.com/room/phishingemails3tryoe) |
| The Greenholt Phish | End-to-end SOC phishing triage challenge | [🔗](https://tryhackme.com/room/phishingemails5fgjlzxc) |

### Module 3 — SIEM & Splunk
| Room | Key Skills | THM Link |
|------|-----------|----------|
| Introduction to SIEM | Log ingestion, alert rules, SIEM architecture | [🔗](https://tryhackme.com/room/introtosiem) |
| Splunk: The Basics | SPL queries, log correlation, anomaly detection | [🔗](https://tryhackme.com/room/splunk101) |

### Module 4 — Network Traffic Analysis
| Room | Key Skills | THM Link |
|------|-----------|----------|
| Wireshark: The Basics | Packet capture, display filters, protocol inspection | [🔗](https://tryhackme.com/room/wiresharkthebasics) |
| Wireshark: Traffic Analysis | Anomalous traffic patterns, lateral movement indicators | [🔗](https://tryhackme.com/room/wiresharktrafficanalysis) |

### Module 5 — Malware Concepts for SOC
| Room | Key Skills | THM Link |
|------|-----------|----------|
| Malware Classification | Ransomware, trojans, RATs, severity classification | [🔗](https://tryhackme.com/room/malwareclassification) |
| Intro to Malware Analysis | Static vs dynamic analysis, sandbox tools, IOC extraction | [🔗](https://tryhackme.com/room/intromalwareanalysis) |

---

## 🔍 Frameworks Applied

| Framework | Application |
|-----------|-------------|
| **Cyber Kill Chain** | Mapped phishing and malware alerts to attack stages (Delivery → Exploitation) |
| **Pyramid of Pain** | Assessed defensive value of blocking hash vs domain vs TTP indicators |
| **MITRE ATT&CK** | Tagged attacker behaviours to technique IDs during alert triage |

---

## 🛠️ Tools Used

`Splunk` · `Wireshark` · `VirusTotal` · `PhishTool` · `MXToolbox` · `URLScan.io` · `Linux CLI` · `NetworkMiner`

---

## 📌 Related Projects

- 🔗 [Phishing Email Analysis — Triage Documentation](../phishing-analysis-project) — 4 real-world phishing samples dissected with structured SOC-style incident reports
- 🔗 [Security Event Dashboard — Power BI](../security-event-dashboard) — Multi-page dashboard tracking alert volume, severity distribution, and temporal attack patterns

---

*All write-ups are my own notes and takeaways. No flags or challenge answers are included in compliance with TryHackMe's terms of service.*
