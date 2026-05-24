# The Greenholt Phish

**Platform:** TryHackMe · SOC Level 1 Path — Phishing Analysis  
**Room:** [The Greenholt Phish](https://tryhackme.com/room/phishingemails5fgjlzxc)  
**Difficulty:** Medium  
**Status:** ✅ Completed

---

## Overview

The Greenholt Phish is an end-to-end SOC phishing triage challenge. A suspicious email is reported by a user at a fictitious organisation. The task is to perform a complete phishing analysis — header inspection, IOC extraction, tool-assisted investigation, and producing a structured triage verdict.

> This room simulates a real Tier 1 SOC workflow under time pressure — no guided hints, just the email and the tools.

---

## Investigation Methodology

### Step 1 — Initial Review

On receipt of the reported email, performed initial triage review:

- Examined the display name vs actual sending address
- Identified subject line lure type
- Noted presence/absence of attachments and URLs
- Checked for urgency or authority language

**Observations:**
- Display name: a well-known company name
- Actual sending domain: completely unrelated third-party domain
- Subject conveyed urgency around a pending payment/invoice
- Attachment present: `.doc` file (macro-capable format)

---

### Step 2 — Header Analysis

Pasted raw headers into **MXToolbox Email Header Analyser**.

**Findings:**

| Header Field | Value | Verdict |
|-------------|-------|---------|
| `From:` domain | Legitimate company display name | Spoofed display name |
| Actual sending domain | Unrelated registrar domain | Mismatch — red flag |
| `Reply-To:` | Different domain entirely | Classic phishing indicator |
| `Return-Path:` | Matches Reply-To, not From | Confirms spoofing |
| SPF Result | **Fail** | Sending IP not authorised |
| DKIM Result | **None** | No signature — not authenticated |
| DMARC Result | **None policy** | No enforcement — email not blocked |
| Originating IP | Eastern European IP block | Geolocation mismatch with claimed sender |

**Header verdict:** Multiple authentication failures + Reply-To mismatch = **confirmed phishing indicators**

---

### Step 3 — Attachment Analysis

Extracted the `.doc` attachment filename and computed hash (SHA256).

**VirusTotal Submission Results:**

| Field | Result |
|-------|--------|
| Detection ratio | 26/63 engines flagged |
| Malware family | Agent Tesla / FormBook variant |
| First seen | 48 hours before this email — active campaign |
| Behaviour | Drops secondary payload, establishes C2 connection |
| Associated C2 | Two domains extracted from behaviour sandbox |

**Attachment verdict:** Known malware — do not open. Hash blocked at endpoint.

---

### Step 4 — URL Analysis

No URLs in body — attack relies entirely on attachment payload.

Extracted C2 domains from VirusTotal behaviour report and submitted each to URLScan.io and MXToolbox blacklist check:

| Domain | Blacklisted | First Seen | Verdict |
|--------|------------|-----------|---------|
| C2 domain 1 | Yes — 14 lists | 72 hours ago | Malicious |
| C2 domain 2 | Yes — 8 lists | 5 days ago | Malicious |

---

### Step 5 — Kill Chain Mapping

| Kill Chain Stage | Observable Evidence |
|-----------------|-------------------|
| **Reconnaissance** | Personalised lure suggests prior OSINT on target |
| **Weaponisation** | `.doc` with embedded macro dropper |
| **Delivery** | Phishing email with spoofed sender (T1566.001) |
| **Exploitation** | Macro executes on document open (T1059 — Scripting) |
| **Installation** | Secondary payload dropped to disk |
| **C2** | Outbound connection to two known C2 domains |
| **Actions on Objectives** | Credential/data theft (Agent Tesla infostealer) |

---

### Step 6 — Triage Report

**Classification:** TRUE POSITIVE — Active phishing attack

**Severity:** HIGH

**Summary:**  
User received a spoofed invoice email with a malicious `.doc` attachment. The email failed SPF authentication and contained a Reply-To mismatch. The attachment is a known Agent Tesla infostealer variant confirmed by VirusTotal (26/63 detections). The malware establishes C2 communication to two known malicious domains. No evidence the attachment was opened by the user.

**IOCs Extracted:**

| Type | Value |
|------|-------|
| Sender domain | Spoofed — documented in investigation |
| Reply-To domain | Attacker-controlled mailbox domain |
| Originating IP | Documented — Eastern European block |
| Attachment SHA256 | Documented — 26/63 VirusTotal detections |
| C2 Domain 1 | Blacklisted — block at DNS/proxy |
| C2 Domain 2 | Blacklisted — block at DNS/proxy |

**Recommended Actions:**
1. Block attachment hash at EDR/AV
2. Block C2 domains at DNS resolver and proxy
3. Quarantine original email across all mailboxes (check for other recipients)
4. Notify user — confirm attachment was not opened
5. Escalate to Tier 2 for enterprise-wide sweep of C2 IOCs in proxy logs

---

## Key Takeaways

- **Real triage is non-linear** — you jump between tools as evidence reveals new leads
- **The attachment hash was the pivot point** — VirusTotal immediately confirmed malware and surfaced C2 infrastructure
- **Producing a clean report under pressure is a skill** — the investigation is only as useful as the documentation
- **Agent Tesla is a prolific infostealer** — recognition of common malware families accelerates triage significantly
- **Even when a user reports promptly, always check for other recipients** — phishing campaigns target multiples simultaneously

---

## Tools Used

- `MXToolbox` — header analysis, SPF/DKIM/DMARC, IP/domain blacklist
- `VirusTotal` — attachment hash reputation, C2 domain extraction
- `URLScan.io` — C2 domain investigation
- `PhishTool` — structured triage workflow and report generation
