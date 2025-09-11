# WebHackFlow‑WPM Mappings to Industry Standards (v1.0)

This document maps the **WebHackFlow Web Penetration Testing Method (WebHackFlow‑WPM)** to commonly used frameworks and guides to ensure consistency and auditability.

---

## 1) PTES ↔ WebHackFlow‑WPM

| PTES Phase | WebHackFlow‑WPM Phase(s) | Notes |
|---|---|---|
| Pre‑engagement Interactions | 0 | SoW/RoE, scope, constraints, evidence handling |
| Intelligence Gathering | 1–3 & 5 | Passive OSINT; Active discovery; Fingerprinting; Info exposure |
| Threat Modeling | Cross‑cutting | Applied lens throughout execution |
| Vulnerability Analysis | 4, 6–10 | Known‑vuln sweep, auth/session, authorization, injection, fuzzing, business logic |
| Exploitation | 11 | Safe, minimal‑impact PoCs and chained scenarios |
| Post‑Exploitation | 11 (web‑scope) | Impact confirmation within web/app constraints |
| Reporting | 12 | Risk rating, remediation, retest |

---

## 2) OWASP WSTG 4.2 ↔ WebHackFlow‑WPM (Category Level)

| WSTG Category | WebHackFlow‑WPM Phase(s) | Examples |
|---|---|---|
| Information Gathering | 1–3 & 5 | OSINT, spidering, tech fingerprinting, content review |
| Configuration and Deployment Management Testing | 3–5 | TLS/headers, reverse proxy/WAF/CDN, backups and debug settings |
| Identity Management Testing | 6 | Registration, login, MFA, password reset, account recovery |
| Authentication Testing | 6 | Sessions, tokens, logout/expiry, fixation |
| Authorization Testing | 7 | BOLA/IDOR/BFLA, vertical/horizontal access control |
| Session Management Testing | 6 | Cookie flags, rotation, invalidation |
| Input Validation Testing | 8–9 | XSS/SQLi/SSRF/XXE/Deserialization/Prototype pollution; coverage‑aware fuzzing |
| Error Handling and Logging | 5 & 12 | Leak minimization; evidence capture and reporting |
| Data Protection | 6 & 12 | JWT/crypto handling, transport, key mgmt references |
| Business Logic Testing | 10 | Workflow abuse, concurrency, price/quantity manipulation |
| Client‑Side Testing | 5, 8 | JS route mining, DOM sinks, CSP/TT review |
| API Testing | 2, 6–10 | OpenAPI/GraphQL mining, authZ, injection, fuzzing |

> Granular WSTG test‑ID ↔ WebHackFlow mappings can be extended per project in `/methodology/mappings/`.

---

## 3) OWASP ASVS 5.0 ↔ WebHackFlow‑WPM (Selected)

| ASVS Area (v5) | WebHackFlow‑WPM Phase(s) | Notes |
|---|---|---|
| V1 – Architecture, Design & Threat Modeling | Cross‑cutting, 0, 3 | Lens across phases; document assumptions |
| V2 – Authentication | 6 | All auth flows and controls |
| V3 – Session Management | 6 | Token/cookie lifecycle, rotation, invalidation |
| V4 – Access Control | 7 | Object & function‑level rules; tenancy boundaries |
| V5 – Validation, Sanitization & Encoding | 8–9 | Context‑aware payloads; server/client‑side |
| V7 – Stored Cryptography | 6 & 12 | JWT/keys/secrets handling; at‑rest guidance |
| V10 – Malicious Code & Supply Chain | 3–4 & 12 | Dependency fingerprinting; known‑vuln sweep |
| V11 – Business Logic | 10 | Workflow‑level invariants |
| V13 – API & Web Service Security | 2, 6–10 | Discovery to exploitation lifecycle |

---

## 4) NIST SP 800‑115 ↔ WebHackFlow‑WPM (Condensed)

| NIST 800‑115 Activity | WebHackFlow‑WPM Phase(s) |
|---|---|
| Planning | 0 |
| Discovery | 1–3 & 5 |
| Vulnerability Identification | 4, 6–10 |
| Penetration (Exploitation) | 11 |
| Analysis & Reporting | 12 |

---

## 5) Risk Rating Alignment

- **OWASP Risk Rating** used for business‑readable risk in Phase 12 (impact/likelihood/context controls).
- **CVSS v3.1/4.0** optionally used for CVE‑tied technical severity in Phase 4; mapped into report’s overall risk for consistent triage.

---

## 6) Evidence & Auditability Crosswalk

| Evidence Item | Standard Expectation |
|---|---|
| Repro steps with exact roles/URLs/params | WSTG/ASVS reproducibility; NIST analysis |
| Raw HTTP transcripts & screenshots | WSTG evidence; NIST documentation |
| Risk score with rationale | OWASP Risk Rating, CVSS where applicable |
| Remediation with code/config samples | WSTG/ASVS guidance |
| Retest results & closure notes | PTES/NIST reporting & verification |

---

## 7) File Placement in Repo

- Place this file at `/methodology/WebHackFlow-Mapping.md` (or root as `WebHackFlow-Mapping.md`). 
- Keep mappings updated when standards or app architecture changes.

*End of v1.0*
