# Pyramid of Pain

**Platform:** TryHackMe · SOC Level 1 Path — Cyber Defence Frameworks  
**Room:** [Pyramid of Pain](https://tryhackme.com/room/pyramidofpainax)  
**Difficulty:** Easy  
**Status:** ✅ Completed

---

## Overview

The Pyramid of Pain is a threat intelligence model created by David Bianco that visualises the relative difficulty an attacker faces when a defender successfully blocks each type of Indicator of Compromise (IOC). The higher up the pyramid, the more pain blocking that indicator causes the adversary.

---

## The Six Levels

### 1. Hash Values (Trivial)
- Cryptographic fingerprints of malicious files (MD5, SHA1, SHA256)
- **Attacker effort to bypass:** Minimal — recompiling or adding a single byte changes the hash entirely
- **Defender value:** Low — useful for exact-match lookups but easily evaded
- **Tools:** VirusTotal, MalwareBazaar

### 2. IP Addresses (Easy)
- IPv4/IPv6 addresses associated with C2 servers or malicious infrastructure
- **Attacker effort to bypass:** Low — attackers rotate IPs or use bulletproof hosting, VPNs, TOR
- **Defender value:** Moderate for short-term blocking; degrades quickly
- **Analyst note:** Fast Flux DNS makes IP blocking especially ineffective

### 3. Domain Names (Simple)
- Domains used for C2, phishing, or malware delivery
- **Attacker effort to bypass:** Moderate — requires registering new domains, costs money and time
- **Defender value:** Better than IPs — domain reputation and typosquatting detection help
- **Tools:** URLScan.io, Whois, MXToolbox

### 4. Network/Host Artefacts (Annoying)
- URI patterns, HTTP user agents, registry keys, file paths, mutexes
- **Attacker effort to bypass:** Significant — requires rewriting tooling or changing TTPs
- **Defender value:** High — artefacts are tied to specific tool implementations
- **Example:** A C2 beacon with a hardcoded User-Agent string

### 5. Tools (Challenging)
- Specific malware families, RATs, exploit frameworks (e.g. Cobalt Strike, Mimikatz)
- **Attacker effort to bypass:** Very high — requires developing or purchasing entirely new tooling
- **Defender value:** Very high — blocking known tools disrupts entire campaigns
- **Detection:** YARA rules, behaviour-based EDR signatures

### 6. TTPs — Tactics, Techniques & Procedures (Tough!)
- The attacker's complete methodology mapped to MITRE ATT&CK
- **Attacker effort to bypass:** Extreme — must retrain, redesign entire attack lifecycle
- **Defender value:** Maximum — forces adversary to fundamentally change behaviour
- **Example:** Detecting PowerShell living-off-the-land behaviour regardless of payload

---

## Key Takeaways

- **Blocking hashes = low ROI.** Attackers trivially bypass hash-based detection.
- **Aim high on the pyramid.** TTP-level detection is durable and forces real operational cost on adversaries.
- **Practical application:** When triaging alerts, ask — *is this block going to stop the attacker or just slow them down by 5 minutes?*
- **Pyramid of Pain drives detection engineering decisions** — where to invest in SIEM rules, EDR policies, and threat hunting playbooks.

---

## Applied in This Lab

| Scenario | IOC Type Blocked | Defender Impact |
|----------|-----------------|----------------|
| Malware sample identified | SHA256 hash | Trivial bypass — attacker recompiles |
| C2 server identified | IP address | Easy bypass — attacker rotates infra |
| Phishing domain blocked | Domain name | Moderate — attacker re-registers |
| Lateral movement pattern detected | TTP (T1075 Pass-the-Hash) | High — forces attacker to change method |

---

## Tools Referenced

- `VirusTotal` — hash and domain reputation lookup
- `MXToolbox` — domain/IP blacklist check
- `URLScan.io` — domain behaviour analysis
- `MITRE ATT&CK Navigator` — TTP mapping and visualisation
