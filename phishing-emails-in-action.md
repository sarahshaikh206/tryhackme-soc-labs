# Phishing Emails in Action

**Platform:** TryHackMe · SOC Level 1 Path — Phishing Analysis  
**Room:** [Phishing Emails in Action](https://tryhackme.com/room/phishingemails2rytmuv)  
**Difficulty:** Easy  
**Status:** ✅ Completed

---

## Overview

This room moves from theory to practice — analysing real phishing email samples end-to-end, identifying attack types, extracting IOCs, and mapping each attack to the Cyber Kill Chain. Four distinct phishing campaigns are examined.

---

## Sample 1 — Business Email Compromise (BEC)

**Lure type:** Authority / Urgency — impersonating a CEO requesting wire transfer  
**Kill Chain Stage:** Delivery (T1566.002 — Spearphishing Link)

### Indicators Found

| IOC Type | Value | Notes |
|----------|-------|-------|
| Sender domain | `ceo-alerts.net` | Registered 3 days prior to send |
| Reply-To | `cfo@external-payments.co` | Completely different domain — classic BEC |
| Urgency language | "Urgent — transfer needed today" | Social engineering pressure tactic |
| No attachment | N/A | BEC often link-free to avoid filters |

### Analysis Notes
- SPF: **Fail** — sending IP not in domain's SPF record
- DMARC: **None** policy — email not rejected despite SPF failure
- No malicious URL — attacker relies purely on social engineering to prompt offline action
- **Escalation recommendation:** High priority — financial fraud vector, notify finance department immediately

---

## Sample 2 — Credential Harvesting via Fake Login Page

**Lure type:** Account security alert — impersonating Microsoft  
**Kill Chain Stage:** Delivery → Exploitation (T1566.002 + T1056.003 Web Portal Capture)

### Indicators Found

| IOC Type | Value | Notes |
|----------|-------|-------|
| Sender address | `noreply@microsft-security.com` | Typosquatting — missing 'o' in Microsoft |
| URL in body | `hxxp://microsft-security[.]com/signin` | Defanged — leads to credential harvester |
| Landing page | Pixel-perfect Microsoft login clone | Captured via URLScan.io screenshot |
| Subject | "Your account has been compromised" | Fear-based urgency lure |

### Analysis Notes
- SPF: **Pass** — attacker registered the typosquat domain with valid SPF
- DKIM: **Pass** — signed by `microsft-security.com`
- DMARC: **Pass** — all checks pass because it's a legitimately registered lookalike
- **Key learning:** Passing all email auth checks does NOT mean the email is safe — domain reputation and content analysis are essential
- **Escalation recommendation:** Block domain at proxy/DNS level, submit URL to threat intel platform

---

## Sample 3 — Malware Delivery via Attachment

**Lure type:** Invoice / Finance — malicious Excel attachment  
**Kill Chain Stage:** Delivery → Exploitation (T1566.001 — Spearphishing Attachment)

### Indicators Found

| IOC Type | Value | Notes |
|----------|-------|-------|
| Attachment name | `Invoice_Q4_2024.xlsm` | .xlsm = macro-enabled Excel |
| Attachment SHA256 | `a3f4...c9d2` | VirusTotal: 34/72 engines detect as malicious |
| Sender domain | `accounts-billing.online` | Newly registered, no prior reputation |
| Macro behaviour | Downloads secondary payload from C2 | Identified via any.run sandbox analysis |

### Analysis Notes
- **Do not open attachments in a standard environment** — always use an isolated sandbox
- `.xlsm` extension is an immediate triage escalation trigger — macros are execution vectors
- VirusTotal hash lookup confirmed as Emotet dropper variant
- C2 domain extracted from macro: `hxxps://update-cdn[.]live/payload`
- **Escalation recommendation:** Critical — block hash at AV/EDR, block C2 domain at DNS/proxy, notify IR team

---

## Sample 4 — Spear Phishing with OSINT Personalisation

**Lure type:** Personalised HR communication — uses target's real name and manager  
**Kill Chain Stage:** Reconnaissance → Delivery (T1598 + T1566.001)

### Indicators Found

| IOC Type | Value | Notes |
|----------|-------|-------|
| Personalisation | Target's full name, department, manager name | Gathered from LinkedIn OSINT |
| Attachment | Fake onboarding form (PDF with embedded link) | Link leads to credential harvester |
| Sender | Spoofed internal HR address | Reply-To routes to external attacker mailbox |
| Urgency | "Complete before end of day to maintain system access" | Classic pressure tactic |

### Analysis Notes
- High sophistication — OSINT-driven personalisation increases click rate significantly
- PDF embedding is used to bypass email URL scanners (URL inside document, not email body)
- **Key learning:** Spear phishing targeting is driven by public OSINT — LinkedIn profiles, company websites, press releases
- **Escalation recommendation:** High — notify targeted employee and manager, review for further targets in the same department

---

## Cross-Sample Patterns

| Pattern | Frequency | Significance |
|---------|-----------|-------------|
| Reply-To mismatch | 3/4 samples | Most reliable single indicator |
| Urgency language | 4/4 samples | Universal social engineering constant |
| Newly registered sender domain | 3/4 samples | Reputation-based detection trigger |
| SPF/DKIM/DMARC failure | 2/4 samples | Not reliable alone — passes for sophisticated attacks |

---

## Key Takeaways

- **Social engineering lures are consistent** — urgency, authority, and fear appear in every sample regardless of technical sophistication
- **Email auth checks (SPF/DKIM/DMARC) are necessary but not sufficient** — typosquatted domains pass all checks
- **Every sample maps cleanly to ATT&CK T1566** — subtype depends on delivery vector (attachment vs link)
- **IOC extraction must be systematic** — use a checklist to ensure nothing is missed under triage pressure

---

## Tools Used

- `MXToolbox` — header analysis and SPF/DKIM/DMARC validation
- `VirusTotal` — attachment hash reputation
- `URLScan.io` — safe URL detonation
- `any.run` — interactive sandbox for macro analysis
- `PhishTool` — structured phishing triage workflow
