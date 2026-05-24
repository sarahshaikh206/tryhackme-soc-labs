# Wireshark: The Basics

**Platform:** TryHackMe · SOC Level 1 Path — Network Traffic Analysis  
**Room:** [Wireshark: The Basics](https://tryhackme.com/room/wiresharkthebasics)  
**Difficulty:** Easy  
**Status:** ✅ Completed

---

## Overview

Wireshark is the industry-standard packet capture and network protocol analyser. For SOC analysts, Wireshark is used to investigate network anomalies, confirm C2 communication, analyse malware behaviour at the packet level, and validate firewall/proxy alert findings.

---

## Interface Layout

| Panel | Contents |
|-------|---------|
| **Packet List** | Chronological list of all captured packets |
| **Packet Details** | Protocol breakdown for selected packet |
| **Packet Bytes** | Raw hex/ASCII representation of packet data |
| **Status Bar** | Capture statistics — total packets, elapsed time, interface |

---

## Core Concepts

### Capture vs Display Filters
- **Capture filters** — applied at capture time, limits what is recorded (BPF syntax)
- **Display filters** — applied post-capture, filters what is shown (Wireshark syntax)

> For SOC analysis of a provided PCAP, display filters are used.

### Protocol Layers in Packet Details
```
Frame (physical layer metadata)
  └── Ethernet (MAC addresses)
       └── IP (source/destination IP)
            └── TCP/UDP (ports, flags)
                 └── Application protocol (HTTP, DNS, FTP, etc.)
```

---

## Essential Display Filters

### Filter by IP
```wireshark
ip.addr == 192.168.1.100          # Any traffic to/from this IP
ip.src == 10.0.0.5                # Traffic FROM this IP only
ip.dst == 8.8.8.8                 # Traffic TO this IP only
```

### Filter by Protocol
```wireshark
http                              # HTTP traffic only
dns                               # DNS queries and responses
ftp                               # FTP traffic
tcp                               # All TCP traffic
udp                               # All UDP traffic
icmp                              # ICMP (ping) traffic
tls                               # TLS/SSL encrypted traffic
```

### Filter by Port
```wireshark
tcp.port == 80                    # HTTP
tcp.port == 443                   # HTTPS
tcp.port == 22                    # SSH
tcp.port == 3389                  # RDP
tcp.dstport == 4444               # Common Metasploit listener port
```

### Filter by TCP Flags
```wireshark
tcp.flags.syn == 1                # SYN packets (new connections)
tcp.flags.rst == 1                # RST packets (connection resets — anomaly indicator)
tcp.flags == 0x002                # SYN only (port scanning indicator)
```

### Filter by Content
```wireshark
http.request.method == "POST"     # HTTP POST requests (credential submission)
http contains "password"          # HTTP traffic containing word 'password'
dns.qry.name contains "malware"   # DNS queries containing a keyword
```

### Combining Filters
```wireshark
ip.src == 10.0.0.5 && http        # HTTP traffic from specific IP
http || dns                        # HTTP or DNS traffic
!(ip.addr == 192.168.1.1)         # Exclude specific IP
```

---

## Key Analysis Techniques

### Following a Stream
Right-click any packet → **Follow → TCP Stream**  
Shows the complete conversation between client and server in readable form.  
Essential for: HTTP credential extraction, plaintext protocol analysis, malware C2 communication.

### Statistics Menu
- **Statistics → Protocol Hierarchy** — see breakdown of all protocols in the capture
- **Statistics → Conversations** — list all connections with byte counts (large transfers = exfiltration candidate)
- **Statistics → Endpoints** — all unique IPs/ports observed
- **Statistics → DNS** — all DNS queries in the capture

### Export Objects
**File → Export Objects → HTTP**  
Extracts files transferred over HTTP (malware samples, downloaded payloads, exfiltrated documents).

---

## IOC Identification in PCAPs

| Suspicious Indicator | Filter | Significance |
|---------------------|--------|-------------|
| DNS queries to newly registered domains | `dns` | C2, phishing infrastructure |
| Regular interval outbound connections | Manual review of timestamps | C2 beaconing |
| Large outbound data transfer | Statistics → Conversations | Data exfiltration |
| Cleartext credentials | `http contains "password"` | Credential theft |
| Connection to unusual ports | `tcp.dstport > 1024` | C2, reverse shells |
| RDP connections from external IPs | `tcp.port == 3389 && ip.src` | Remote access |
| Port scan pattern | `tcp.flags == 0x002` | Reconnaissance |

---

## Lab Exercises

### Exercise 1 — HTTP Traffic Analysis

Applied filter: `http.request.method == "POST"`

Found a POST request to `/login.php` containing cleartext credentials in the request body — visible by following the TCP stream.

**Finding:** Username and password transmitted in plaintext over HTTP (no TLS). Credential theft possible if traffic was intercepted on network.

---

### Exercise 2 — DNS Exfiltration Pattern

Applied filter: `dns`

Identified a host making DNS queries with unusually long subdomains:
```
a3f4c9d2b1e8.exfil-domain.com
b7e2f1c4a9d3.exfil-domain.com
```

**Finding:** Encoded data being exfiltrated via DNS subdomains — a DNS tunnelling technique (T1071.004). Data is encoded in the subdomain, responses from the C2 server carry commands.

---

### Exercise 3 — Identifying C2 Beaconing

Applied filter: `ip.dst == [suspicious_ip]` and examined packet timestamps.

Found packets sent to a single external IP at exactly 30-second intervals over 4 hours.

**Finding:** Regular-interval beaconing pattern consistent with C2 malware implant. Extracted destination IP as IOC.

---

## Key Takeaways

- **Display filters are the primary investigation tool** — practice until common filters are muscle memory
- **Follow Stream reveals context** — raw packet lists are meaningless without seeing the conversation
- **Statistics → Conversations surfaces exfiltration candidates** — look for abnormally large byte counts to external IPs
- **DNS tunnelling is subtle** — always look at query length and frequency, not just reputation
- **Wireshark confirms what SIEM/proxy alerts suspect** — it's a validation and deep-dive tool, not a primary detection mechanism

---

## Tools Referenced

- `Wireshark 4.x` — packet capture and protocol analysis
- `NetworkMiner` — complementary tool for file extraction and session reconstruction
- `Zeek` — network security monitoring via scripted log generation
