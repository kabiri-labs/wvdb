---
id: XSS-R1
title: Reflected XSS — same-response reflection without contextual encoding
group: XSS
subtype: Reflected
severity_range: medium to critical
auth_scope: often unauthenticated
contexts: [HTML text, HTML attribute, JavaScript string, URL attribute]
mapping:
  wstg: "Testing for Reflected XSS"
  asvs_v5: ["V5 Output Encoding & Escaping"]
  cwe: ["CWE-79"]
tags: [xss, reflected, output-encoding, csp, asvs, wstg, cwe-79]
last_updated: 2025-09-10
---

# Scene
A user-controlled value (e.g., `GET /search?q=...`) is echoed back in the **same** HTTP response. The browser interprets it as **code** because the output is not encoded for the exact rendering context.

# Attack Surface / Context
**Entry points:** query string, path segments, POST form fields, JSON echoed into templates, occasionally request headers in debug banners.  
**Rendering contexts:** 
- **HTML text** (between tags),
- **HTML attributes** (`title=""`, `data-*`, event attributes),
- **JavaScript strings** (inline `<script>`),
- **URL attributes** (`href`, `src`, `action`).

# Preconditions
1) The value is reflected.  
2) The final **context** is known.  
3) No proper **contextual output encoding** at the render point (and Content Security Policy (CSP), if any, is permissive).

# Influencers (Probability Modifiers)
- **Lowering:** templating with auto-escape, encode-on-output, restrictive CSP (no inline, strict `script-src`).  
- **Raising:** ad-hoc blacklists, naive filtering, inline scripts, legacy templates.

# Hypotheses & Probes (Hands-on)
> Only in a controlled environment with explicit permission.

**1) HTML text**  
- Probe: `<svg onload=alert(1)>`  
- Safe tell: the string appears literally, not executed.

**2) HTML attribute**  
- Probe: `" autofocus onfocus=alert(1) x="`  
- Safe tell: quotes are escaped; the attribute remains intact.

**3) JavaScript string (inline)**  
- Probe: `';alert(1);//`  
- Safe tell: the string is escaped; no string-break happens.

**4) URL attribute**  
- Probe non-executable markers first; verify that executable protocols (e.g., `javascript:`) are not allowed.

# Minimal Exploit (Show real impact)
Demonstrate a harmless **state-changing request** using the victim’s session in a test tenant, for example toggling a profile flag via `fetch('/api/me', {method:'POST', body:'...'});`.  
This proves “can act as the victim” — stronger than just `alert(1)`.

# Fix & Hardening
- **Encode on output** for the **exact** context (HTML/Attribute/JavaScript/URL).  
- Remove inline scripts and dangerous concatenation.  
- Add a **restrictive CSP** (block inline; pin script sources).

# Verification
- Unit tests for the template/encoder per context.  
- Integration test: repeat the probes — they must render as harmless text.

# Detection & Observability
- Log unusual characters in key parameters; enable CSP violation reports; rate-limit repetitive failed probes.

# Quick Triage Checklist
- Where exactly does the value land (HTML/Attr/JS/URL)?  
- Is there **any** place where auto-escape is disabled?  
- Do templates or middleware normalize/strip characters (may hide issues)?

# Automation (Playwright sample)
```js
import { chromium } from 'playwright';
const url = 'https://app.test/search?q=</p><svg onload=alert(1)>';
(async () => {
  const browser = await chromium.launch();
  const page = await browser.newPage();
  let dialogFired = false;
  page.on('dialog', async d => { dialogFired = True; await d.dismiss(); });
  await page.goto(url, { waitUntil: 'domcontentloaded' });
  console.log('XSS dialog fired:', dialogFired);
  await browser.close();
})();
```

# False Positives & Edge Cases
- **CSP blocked inline script**: payload may not fire although encoding is missing — check CSP reports.  
- **HTML sanitizers**: confirm allow-list rules; some permit SVG/MathML by default.  
- **Legacy browsers/quirks**: behavior differs; test modern targets first.

# Attack Chain
- **Reflected XSS → Cross-Site Request Forgery (CSRF) bypass:** read DOM CSRF tokens or craft stateful requests directly.  
- **Reflected XSS → Session data exposure:** if tokens live in Web Storage or cookies lack `HttpOnly`.  
- **Reflected XSS → Privileged actions:** if the victim is staff/admin, change settings, extract data, or plant **Stored XSS** elsewhere.  
- **Reflected XSS → BOLA/IDOR discovery:** enumerate object IDs/filters via `fetch` to quickly identify authorization gaps in APIs.

# Standards Mapping
- **WSTG:** Testing for Reflected XSS.  
- **ASVS v5:** Output Encoding & Escaping.  
- **CWE:** CWE-79 (Cross-Site Scripting).
