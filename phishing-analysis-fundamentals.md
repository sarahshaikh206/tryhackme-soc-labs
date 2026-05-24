# Phishing Analysis Fundamentals

**Platform:** TryHackMe · SOC Level 1 Path — Phishing Analysis  
**Room:** [Phishing Analysis Fundamentals](https://tryhackme.com/room/phishingemails1tryoe)  
**Difficulty:** Easy  
**Status:** ✅ Completed

---

## Overview

Phishing is the most common initial access vector in modern cyberattacks. This room builds the foundational skills to dissect phishing emails — examining headers, identifying spoofing techniques, and extracting indicators of compromise (IOCs) for triage and escalation.

---

## Email Header Anatomy

Every email carries headers that reveal its true origin and routing path. Analysts must read headers from **bottom to top** — the bottom reflects where the email originated, the top where it arrived.

### Critical Header Fields

| Header | Purpose | What to Look For |
|--------|---------|-----------------|
| `From:` | Display name and address shown to user | Spoofed display name, mismatched domain |
| `Reply-To:` | Where replies are sent | Different domain than From — major red flag |
| `Return-Path:` | Where bounces go | Mismatch with From domain = spoofing indicator |
| `Received:` | Mail server hops (read bottom-up) | Unexpected originating IP or server |
| `X-Originating-IP:` | IP of sending client | Cross-reference against known malicious IPs |
| `Message-ID:` | Unique message identifier | Domain in ID should match sending domain |
| `Date:` | Timestamp of sending | Timezone inconsistencies, unusual hours |
| `X-Spam-Status:` | Spam filter result | Check score and triggered rules |

---

## Spoofing Techniques

### Display Name Spoofing
The attacker sets a legitimate-looking display name (e.g. "Microsoft Support") while the actual sending address is completely unrelated.

```
From: Microsoft Support <attacker@malicious-domain.xyz>
```

### Domain Spoofing
Attacker registers a lookalike domain:
- `microsofft.com` (double letter)
- `microsoft-support.net` (hyphen + different TLD)
- `rnicrosoff.com` (substituting `rn` for `m`)

### Reply-To Hijacking
Email appears to come from a legitimate address but replies route to attacker-controlled inbox:
```
From: hr@legitimate-company.com
Reply-To: hr@attacker-controlled.com
```

### Subdomain Abuse
```
From: noreply@legit.legitimate-company.com.attacker.xyz
```
The real domain here is `attacker.xyz` — everything before is a subdomain.

---

## Authentication Mechanisms (SPF / DKIM / DMARC)

### SPF (Sender Policy Framework)
DNS record listing which IP addresses are authorised to send email on behalf of a domain.

- **Pass:** Sending IP is in the SPF record
- **Fail / SoftFail:** Sending IP not authorised — likely spoofed

### DKIM (DomainKeys Identified Mail)
Cryptographic signature attached to email. Receiving server verifies signature against public key in DNS.

- **Pass:** Email content has not been tampered with and matches signing domain
- **Fail:** Signature invalid — email may be forged or modified in transit

### DMARC (Domain-based Message Authentication, Reporting & Conformance)
Policy that instructs receiving mail servers what to do when SPF/DKIM fail.

- **p=none** — Monitor only, no action
- **p=quarantine** — Send to spam
- **p=reject** — Block entirely

> **SOC insight:** A phishing email that passes SPF/DKIM/DMARC has come from a *legitimately registered domain* — the attacker registered a lookalike. This is more sophisticated and harder to catch automatically.

---

## IOC Extraction Checklist

When triaging a phishing email, extract:

- [ ] Sender email address (full, including domain)
- [ ] Reply-To address (if different from From)
- [ ] Return-Path domain
- [ ] Originating IP address (`X-Originating-IP` or bottom `Received:` hop)
- [ ] All URLs in the email body (expand shortened URLs)
- [ ] Attachment filename and hash (MD5/SHA256)
- [ ] Subject line
- [ ] Any embedded images (may contain tracking pixels)

---

## Triage Decision Framework

```
Is the From / Reply-To / Return-Path consistent?
    NO → Spoofing indicator — escalate

Does SPF / DKIM / DMARC pass?
    FAIL → Authentication failure — escalate

Are URLs present? Do they match the claimed sender's domain?
    MISMATCH → Malicious redirect — extract URL as IOC

Is there an attachment?
    YES → Hash it, submit to VirusTotal — do NOT open locally
```

---

## Key Takeaways

- **Headers never lie** — even a well-crafted phishing email leaves traces in routing headers
- **Reply-To mismatch is the single most reliable phishing signal** in header analysis
- **SPF/DKIM/DMARC failures are immediate escalation triggers** in a Tier 1 SOC workflow
- Always extract IOCs *before* escalating — gives Tier 2 a head start on investigation
- Social engineering lures (urgency, authority, fear) are consistent across phishing types — recognising the lure helps classify the campaign

---

## Tools Used

- `MXToolbox Email Header Analyser` — automated header parsing
- `Google Admin Toolbox — Messageheader` — visualise header hops
- `VirusTotal` — URL and attachment reputation
- `URLScan.io` — safe URL detonation and screenshot
