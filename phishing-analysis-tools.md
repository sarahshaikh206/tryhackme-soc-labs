# Phishing Analysis Tools

**Platform:** TryHackMe · SOC Level 1 Path — Phishing Analysis  
**Room:** [Phishing Analysis Tools](https://tryhackme.com/room/phishingemails3tryoe)  
**Difficulty:** Easy  
**Status:** ✅ Completed

---

## Overview

This room covers the core toolchain used by SOC analysts to triage phishing emails efficiently — from header parsing to URL detonation and hash reputation checks.

---

## Tool 1 — PhishTool

**Purpose:** End-to-end phishing email analysis platform  
**URL:** [phishtool.com](https://phishtool.com)

### Key Features
- Upload `.eml` files for automated header parsing
- Extracts all URLs, attachments, and sender metadata automatically
- Provides OSINT enrichment on sender IPs and domains
- Workflow tracking — assign, triage, escalate within the platform
- Generates structured triage reports

### Workflow
```
Upload .eml → Auto-extract IOCs → Review header analysis → 
Check URL/IP reputation → Document findings → Export report
```

### When to Use
- Primary tool for bulk phishing triage in a SOC environment
- When a structured audit trail is required for escalation

---

## Tool 2 — MXToolbox

**Purpose:** Email infrastructure analysis and blacklist checking  
**URL:** [mxtoolbox.com](https://mxtoolbox.com)

### Key Capabilities

| Feature | Use Case |
|---------|---------|
| Email Header Analyser | Parse raw headers, visualise hops |
| SPF Record Lookup | Verify if sending IP is authorised |
| DKIM Lookup | Check if domain has valid DKIM keys |
| DMARC Lookup | Review DMARC policy for a domain |
| Blacklist Check | Check if domain/IP is on known blocklists |
| WHOIS Lookup | Domain registration age and registrar |

### Analyst Tips
- **Domain age matters** — newly registered domains (< 30 days) sending email are a major red flag
- **Blacklist check** — if a domain is on 0 of 100 lists, it doesn't mean it's safe — it may be brand new
- Use WHOIS to check if registrant info is hidden (privacy-protected) — common in malicious infrastructure

---

## Tool 3 — VirusTotal

**Purpose:** Multi-engine malware and reputation scanning  
**URL:** [virustotal.com](https://virustotal.com)

### What to Submit

| Input Type | Used For |
|-----------|---------|
| File hash (MD5/SHA256) | Check if attachment is known malware |
| URL | Check if URL is known phishing/malware page |
| IP address | Check if IP is associated with C2 or malicious activity |
| Domain | Check reputation, associated files, WHOIS |

### Reading Results
- **Detection ratio:** `34/72` means 34 of 72 AV engines flag as malicious
- **Community score:** Negative score = suspicious community consensus
- **Relations tab:** Shows network infrastructure connected to the sample (C2 IPs, domains, dropped files)
- **Behaviour tab:** Sandbox execution results — process creation, file writes, network connections

### Analyst Tips
- A `0/72` result does NOT mean safe — new malware won't be in any database yet
- Check the **submission date** — first seen timestamp reveals campaign age
- **Never upload sensitive/classified attachments to VirusTotal** — it is a public platform

---

## Tool 4 — URLScan.io

**Purpose:** Safe URL detonation and screenshot capture  
**URL:** [urlscan.io](https://urlscan.io)

### What It Does
- Visits a URL in an isolated browser environment
- Captures a screenshot of the landing page
- Logs all network requests, redirects, and loaded scripts
- Identifies final destination of redirect chains

### Key Output Fields
- **Screenshot** — visual of what the user would see (crucial for identifying fake login pages)
- **DOM structure** — HTML content of the page
- **HTTP transactions** — all requests made, including redirects
- **Verdicts** — automated malicious/suspicious classification
- **Linked URLs** — all external resources the page loads

### Workflow Example
```
Suspicious URL found in email body
    ↓
Defang URL (replace . with [.] and http with hxxp)
    ↓
Submit to URLScan.io
    ↓
Review screenshot — is it a fake login page?
    ↓
Check redirect chain — what is the final destination?
    ↓
Extract final URL as IOC, check on VirusTotal
    ↓
Document and escalate
```

---

## Tool 5 — Google Admin Toolbox — Messageheader

**Purpose:** Visual email header parsing  
**URL:** [toolbox.googleapps.com/apps/messageheader](https://toolbox.googleapps.com/apps/messageheader/analyzeheader)

### What It Does
- Parses raw email headers into human-readable format
- Visualises the delivery path (each server hop)
- Highlights delays between hops (unusual delays can indicate tampering)
- Shows SPF/DKIM authentication results inline

---

## Triage Toolchain — Standard Workflow

```
1. Receive phishing report from user
2. Export email as .eml (do not forward — preserves headers)
3. Upload to PhishTool or paste headers into MXToolbox
4. Extract IOCs: sender IP, Reply-To domain, URLs, attachment hashes
5. Check sender IP/domain against MXToolbox blacklists + WHOIS age
6. Submit attachment hash to VirusTotal
7. Detonate URLs via URLScan.io — screenshot and redirect chain
8. Classify: True Positive / False Positive / Benign
9. Document findings in structured report
10. Escalate if True Positive — provide all extracted IOCs to Tier 2
```

---

## Key Takeaways

- **Never analyse phishing emails in a production environment** — always use isolated tools or sandboxes
- **VirusTotal is a starting point, not a verdict** — 0 detections ≠ clean
- **URLScan.io is essential** — screenshots reveal credential harvesting pages that text analysis misses
- **PhishTool creates an audit trail** — critical for SOC documentation and shift handoffs
- **Speed + accuracy** — the toolchain above can complete a triage in under 10 minutes with practice

---

## Tools Referenced

- `PhishTool` — end-to-end phishing triage platform
- `MXToolbox` — header analysis, SPF/DKIM/DMARC, blacklist checks
- `VirusTotal` — multi-engine hash and URL reputation
- `URLScan.io` — safe URL detonation and page screenshot
- `Google Messageheader` — visual header hop analysis
