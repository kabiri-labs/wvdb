---
id: XSS-S1
title: Stored XSS — persisted content executes when rendered later
group: XSS
subtype: Stored
severity_range: high to critical
auth_scope: low-privilege input; higher-privilege viewing
contexts: [HTML text, HTML attribute, JavaScript string, URL attribute, HTML email]
mapping:
  wstg: "Testing for Stored XSS"
  asvs_v5: ["V5 Output Encoding & Escaping", "HTML Sanitization"]
  cwe: ["CWE-79"]
tags: [xss, stored, html-sanitization, output-encoding, csp, asvs, wstg, cwe-79]
last_updated: 2025-09-10
---

# Scene
User input (comment, profile field, WYSIWYG content) is **stored** and later **rendered**. Without correct contextual encoding at render time, the malicious content executes **persistently**.

# Attack Surface / Context
**Entry points:** comments, display names, bios, product reviews, ticket subjects, CRM fields, import pipelines.  
**Destinations:** profile pages, listing pages, **admin/moderation dashboards**, HTML emails, exports (PDF/CSV→HTML preview), RSS/Atom feeds.  
**Context note:** rich-text renderers (Markdown/HTML) often allow unsafe HTML by default.

# Preconditions
1) Data is persisted.  
2) It is rendered later.  
3) Rendering is done **without** contextual encoding (and CSP is permissive).

# Influencers
- **Lowering:** encode-on-output in all views; sanitize rich text with a strict allow-list; restrictive CSP in user and admin UIs.  
- **Raising:** `innerHTML`/raw HTML injection in display templates; trusting “internal-only” viewers.

# Hypotheses & Probes (Hands-on)
1) Store a benign marker: `XSS_TEST_123`. Visit **every** place that displays it (user view, list, admin queue, email preview, exports).  
2) Replace with a PoC per **context**:
   - HTML text: `<svg onload=alert(1)>`
   - Attribute: `" onmouseover=alert(1) x="`
   - Inline JS: `';alert(1);//`
3) If any fires, record which destination and which template.

# Minimal Exploit (Show real impact)
When a **staff user** views the tainted record, run a script that, for example, submits a settings form or exfiltrates a **DOM-rendered** CSRF token (test tenant only). Persistent XSS can hit **every** viewer.

# Fix & Hardening
- Encode on output at **every** render point (views, emails, reports).  
- Replace unsafe sinks with safe templating; sanitize rich text strictly (allow-list).  
- Add **CSP** to reduce blast radius.

# Verification
- Automated test for the full path **input → store → render**.  
- Manual checks for side channels (email previews, admin tools, exports).

# Detection & Observability
- Tag stored records with test IDs; monitor CSP reports; alert on execution attempts tied to those records.

# Quick Triage Checklist
- Which renderers consume the stored value (pages, emails, exports, admin)?  
- Do any renderers **bypass** the normal templating/escaping path?  
- Are sanitizers configured with a **strict allow-list** (SVG/MathML disabled unless required)?

# Automation (Playwright sample)
```js
import { chromium } from 'playwright';
const base = 'https://app.test';
(async () => {
  const browser = await chromium.launch();
  const page = await browser.newPage();
  await page.goto(base + '/login');
  // ... perform test login ...
  await page.goto(base + '/new-comment');
  await page.fill('#comment', '<svg onload=alert(1)>');
  await page.click('button[type=submit]');
  let fired = false; page.on('dialog', async d => { fired = true; await d.dismiss(); });
  await page.goto(base + '/admin/moderation');
  console.log('Stored XSS fired in admin view:', fired);
  await browser.close();
})();
```

# False Positives & Edge Cases
- **Sanitizer exceptions**: some allow “safe” tags but unsafe attributes; review configuration.  
- **Email clients**: many strip scripts but allow risky HTML; confirm with target clients.  
- **Export pipelines**: HTML in PDF/report engines may behave differently than browsers.

# Attack Chain
- **Stored XSS → Admin takeover:** scripts in admin views can create users, change settings, extract data.  
- **Stored XSS → Persistence:** loader script runs on every view.  
- **Stored XSS → Plant more XSS:** use admin UI to seed new payloads.  
- **Stored XSS → CSRF-style changes:** issue stateful requests.  
- **Stored XSS → SQLi surface discovery:** pivot to **internal tools** (query consoles/report builders). Even if SQLi isn’t caused by XSS, admin access can reveal/enable safe SQLi testing.

# Standards Mapping
- **WSTG:** Testing for Stored XSS.  
- **ASVS v5:** Output Encoding & Escaping; HTML Sanitization.  
- **CWE:** CWE-79 (Cross-Site Scripting).
