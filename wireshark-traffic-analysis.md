# Wireshark: Traffic Analysis

**Platform:** TryHackMe · SOC Level 1 Path — Network Traffic Analysis  
**Room:** [Wireshark: Traffic Analysis](https://tryhackme.com/room/wiresharktrafficanalysis)  
**Difficulty:** Medium  
**Status:** ✅ Completed

---

## Overview

This room moves beyond syntax to applied threat hunting in packet captures. Given real-world PCAP files, the objective is to identify attack patterns, extract attacker infrastructure details, and classify the threat — simulating a network-level SOC investigation.

---

## Scenario 1 — ARP Poisoning / MITM Detection

### Background
Alert fired from network IDS — unusual ARP broadcast activity on the internal network.

### Investigation

Applied filter: `arp`

**Observed pattern:**
- Multiple ARP reply packets sent without corresponding ARP requests (Gratuitous ARP)
- Two different MAC addresses both claiming to be the default gateway IP (`192.168.1.1`)
- One legitimate MAC (router), one attacker MAC

**ARP Poisoning Mechanics:**
```
Normal: Host asks "Who has 192.168.1.1?" → Router replies with its MAC
Attack: Attacker constantly broadcasts "I am 192.168.1.1" with own MAC
Result: Victims update ARP cache → all traffic routes through attacker (MITM)
```

**Wireshark filter used:**
```wireshark
arp.opcode == 2                    # ARP Reply packets only
```

**Finding:** Identified attacker MAC address and originating switch port. All traffic from victim hosts was being intercepted.

**ATT&CK Mapping:** T1557.002 — ARP Cache Poisoning

**IOCs:**
- Attacker MAC address
- Attacker internal IP (identified from subsequent IP traffic)

---

## Scenario 2 — Port Scanning / Reconnaissance

### Background
Spike in RST packets observed on perimeter firewall — possible port scan in progress.

### Investigation

Applied filter: `tcp.flags == 0x002` (SYN-only — no ACK response expected)

**Observed pattern:**
- Single source IP sending SYN packets to sequential destination ports
- Massive volume in short timeframe (5,000+ packets in 30 seconds)
- Mix of SYN-RST (closed port) and SYN-SYN/ACK (open port) responses

**Port Scan Types Identified:**
| Scan Type | Pattern | Wireshark Filter |
|-----------|---------|----------------|
| SYN Scan (half-open) | SYN → SYN/ACK or RST | `tcp.flags == 0x002` |
| Connect Scan | Full 3-way handshake | `tcp.flags.syn == 1 && tcp.flags.ack == 0` |
| UDP Scan | UDP packet → ICMP unreachable | `icmp.type == 3` |

**Finding:** TCP SYN scan from external IP against a web server — scanning common service ports. Followed by targeted exploitation attempts against port 8080 (web app).

**ATT&CK Mapping:** T1046 — Network Service Scanning

**IOCs:**
- Scanning source IP
- Target ports enumerated (map of open/closed)
- Timestamp of scan (aids correlation with other log sources)

---

## Scenario 3 — Malware C2 Traffic Analysis

### Background
EDR alert fired on a workstation — suspicious outbound connection. PCAP extracted from network tap for analysis.

### Investigation

Applied filter: `ip.dst == [alert_IP] && tcp`

**Observed pattern:**
- Regular HTTPS connections to an IP with no associated domain name (IP-direct communication)
- Beaconing interval: exactly 45 seconds
- Small, uniform packet sizes (C2 check-in, not data transfer)
- Occasional larger packet burst (command delivery or data upload)

**Beacon Analysis:**
```
Time 00:00:00 → 45s → 01:30 → 02:15 → 03:00...
All packets: ~200 bytes outbound, ~50 bytes inbound (check-in/ack)
Burst at 15:32: 45KB upload (likely data exfiltration)
```

**TLS Certificate Investigation:**
- Navigated to TLS handshake packet → Certificate details
- Self-signed certificate — no trusted CA
- Common Name: randomly generated string (not a legitimate domain)
- Certificate first valid: 2 days prior to infection

**Finding:** Classic C2 beaconing over HTTPS to IP-direct infrastructure with self-signed certificate. Consistent with Cobalt Strike or similar commercial RAT.

**ATT&CK Mapping:** T1071.001 (Web Protocols) + T1573.002 (Asymmetric Cryptography)

**IOCs:**
- C2 IP address
- TLS certificate fingerprint (SHA1)
- Beaconing interval (45 seconds)
- First/Last seen timestamps

---

## Scenario 4 — Data Exfiltration via HTTP

### Background
DLP alert on unusual outbound data volume from a finance workstation.

### Investigation

Applied filter: `http.request.method == "POST" && ip.src == [workstation_IP]`

**Observed pattern:**
- Series of HTTP POST requests to external IP
- POST bodies contained base64-encoded data
- Destination is not a known business application

**Decoding the Payload:**
1. Followed TCP stream on POST request
2. Copied base64 string from POST body
3. Decoded: revealed compressed archive of what appeared to be document filenames

**Statistics → Conversations** showed 2.3GB uploaded to this destination over 6 hours.

**Finding:** Staged data exfiltration — malware collected documents, compressed and base64-encoded them, uploaded via HTTP POST to attacker infrastructure.

**ATT&CK Mapping:** T1041 — Exfiltration Over C2 Channel + T1560.001 — Archive via Utility

**IOCs:**
- Exfiltration destination IP
- Volume: 2.3GB over 6 hours
- Encoding method: base64
- Timeframe of exfiltration

---

## Investigation Workflow Summary

```
1. Open PCAP in Wireshark
2. Check Statistics → Protocol Hierarchy (understand traffic composition)
3. Check Statistics → Conversations (identify top talkers, large transfers)
4. Apply relevant display filters based on alert context
5. Follow TCP/UDP streams on suspicious connections
6. Extract IOCs: IPs, domains, certificates, file hashes
7. Cross-reference against VirusTotal / threat intel feeds
8. Map findings to ATT&CK techniques
9. Document timeline and produce investigation report
```

---

## Key Takeaways

- **Protocol Hierarchy and Conversations are always first** — they give immediate context before diving into individual packets
- **Beaconing is regular; legitimate traffic is irregular** — pattern recognition is a core Wireshark skill
- **Self-signed TLS certificates are a red flag** — legitimate services use trusted CA certificates
- **Base64 in HTTP POST bodies almost always warrants investigation** — it's a common encoding for exfiltration
- **ARP anomalies are invisible to SIEM without network monitoring** — Wireshark is essential for layer 2 attack detection
- **Every network anomaly links to an ATT&CK technique** — tagging findings accelerates escalation and threat intel sharing

---

## Tools Referenced

- `Wireshark` — packet capture analysis
- `NetworkMiner` — session reconstruction and file carving
- `CyberChef` — base64 decoding, payload analysis
- `VirusTotal` — C2 IP and certificate reputation
