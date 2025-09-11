---
id: XSS-B1
title: Blind (OOB) XSS — executes elsewhere, proven by an out-of-band callback
group: XSS
subtype: Blind / OOB
severity_range: high to critical
auth_scope: low-privilege entry; high-privilege execution
contexts: [HTML email renderer, admin dashboards, reporting engines, PDF/HTML exports]
mapping:
  wstg: "Stored/Indirect XSS patterns + OOB confirmation"
  asvs_v5: ["V5 Output Encoding & Escaping", "HTML handling"]
  cwe: ["CWE-79"]
tags: [xss, blind-xss, oob, admin-ui, html-email, csp, asvs, wstg, cwe-79]
last_updated: 2025-09-10
---

# Scene
Your payload **does not** execute in the current page. It executes **later** in a different renderer (admin UI, HTML email, report viewer). You confirm success via an **out-of-band (OOB)** network interaction to a domain you control.

# Attack Surface / Context
**Entry points:** support/contact forms, ticket titles, CRM note fields, file metadata, error messages, logs re-rendered in consoles, BI imports.  
**Destinations where it fires:** admin/moderation dashboards, HTML email clients, reporting/PDF engines, monitoring tools that render HTML.

# Preconditions
1) The stored content flows to a different renderer.  
2) That renderer treats it unsafely.  
3) You can observe an **OOB** callback when it executes.

# Influencers
- **Lowering:** encode-on-output in destination templates; sanitize rich text; restrictive CSP in **staff/admin** surfaces.  
- **Raising:** raw HTML rendering in operational tools; HTML emails built from user-provided fields.

# Hypotheses & Probes (Hands-on)
- Seed likely fields (e.g., “ticket subject”) with a payload that, on execution, performs a **GET** to `https://YOUR-OOB.example/ID123`.  
- Track interactions by unique IDs per record.  
- If an interaction appears, your payload executed **elsewhere**.

# Minimal Exploit (Show real impact)
A one-line script making a beacon request to your OOB endpoint is enough to prove code execution in the target renderer (no dependency on its UI). Do this only in a sanctioned test environment.

# Fix & Hardening
- Fix the **destination** renderer: encode contextually, sanitize rich text strictly.  
- Add **CSP** to admin/staff surfaces; disable raw HTML in email/report templates unless absolutely required.

# Verification
- Repeat the OOB test; no callback should be received.  
- Review destination code for unsafe sinks; test email/report rendering against tainted samples.

# Detection & Observability
- Monitor OOB interactions during testing; log renderers that switch to “raw HTML” mode; add CSP report endpoints.

# Quick Triage Checklist
- Which **destination renderers** will consume this data?  
- Is there a **sanitization** step before rendering?  
- Can you instrument an **OOB** endpoint for confirmation?

# Automation (OOB workflow)
- Use a collaborator endpoint (unique subdomain per test).  
- Store payload in likely fields; watch for DNS/HTTP interactions tagged with the test ID.  
- Correlate the first interaction time with staff activity logs to identify the firing view.

# Attack Chain
- **Blind XSS → Admin compromise:** run code in staff context, change settings, create users, extract data.  
- **Blind XSS → Token/session theft:** read Web Storage or non-`HttpOnly` cookies (if present).  
- **Blind XSS → Lateral movement:** leverage admin tools to trigger SSRF-like features, export sensitive data, or plant **Stored XSS** across records.  
- **Blind XSS → Supply-chain style impact:** if back-office tools forward your HTML to downstream systems (mail gateways, BI dashboards), the effect can propagate.  
- **Blind XSS → BOLA/IDOR discovery:** enumerate privileged API endpoints from the admin origin to find authorization gaps.

# Standards Mapping
- **WSTG:** Stored/Indirect XSS patterns; OOB confirmation workflows.  
- **ASVS v5:** Output Encoding & Escaping; HTML handling.  
- **CWE:** CWE-79 (Cross-Site Scripting).
