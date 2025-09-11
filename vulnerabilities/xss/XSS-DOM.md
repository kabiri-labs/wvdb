---
id: XSS-D1
title: DOM-based XSS — source→sink in the browser
group: XSS
subtype: DOM
severity_range: medium to critical
auth_scope: varies
contexts: [DOM HTML injection, inline events, string-eval]
mapping:
  wstg: "Testing for DOM-based XSS"
  asvs_v5: ["V5 Output Encoding & Escaping", "Client-side rendering safety"]
  cwe: ["CWE-79"]
tags: [xss, dom-xss, dom-sinks, innerHTML, eval, csp, asvs, wstg, cwe-79]
last_updated: 2025-09-10
---

# Scene
Client-side code reads attacker-controlled data from a **source** (e.g., `location`, `document.referrer`, `postMessage`, Web Storage) and writes it to a dangerous **sink** (`innerHTML`, `document.write`, inline event handlers, `eval`), causing code execution **without** server involvement.

# Attack Surface / Context
**Sources:** `location.search`/`hash`, `document.referrer`, `postMessage`, Web Storage, query params parsed by front-end routers.  
**Sinks (avoid):** `innerHTML`/`outerHTML`, template concatenation, `eval`/`Function`, `setTimeout('string')`, inline event attributes.  
**Safer APIs:** `textContent`, `createElement`, `setAttribute` (with encoding), framework auto-escape templating.

# Preconditions
A controllable **source** flows to a dangerous **sink** without safe handling or encoding.

# Influencers
- **Lowering:** safe DOM APIs, auto-escaping templates, removing string-eval, restrictive CSP (no inline, strict sources).  
- **Raising:** direct `innerHTML`, concatenation of HTML strings, plugin code that injects raw HTML.

# Hypotheses & Probes (Hands-on)
1) Map `source → sink`: search code for `innerHTML`, `document.write`, `on*=` attributes, `eval`.  
2) Control the source and observe the DOM:
   - `?msg=<svg onload=alert(1)>`
   - `#x=';alert(1);//`
   - `window.postMessage('<img src=x onerror=alert(1)>','*')` (if handlers exist)
3) Confirm the exact rendering context and adjust the payload accordingly.

# Minimal Exploit (Show real impact)
Use a harmless PoC via the URL or message channel to trigger execution; document the flow (which component reads, transforms, and writes where). This shows **browser-only** exploitability.

# Fix & Hardening
- Replace `innerHTML` patterns with `textContent`/element creation; encode attributes/URLs.  
- Remove `eval`/string timeouts.  
- Add **CSP** (disallow inline; pin script sources).

# Verification
- Lint rules forbidding dangerous sinks; unit tests that render components with attacker-like inputs and assert **no HTML interpretation**; manual DOM-XSS checks.

# Detection & Observability
- Dev build warnings for dangerous sinks; CSP violation reporting; security code review gates on PRs.

# Quick Triage Checklist
- Which **sources** are attacker-controllable?  
- Do any components write to **dangerous sinks**?  
- Are there places using `dangerouslySetInnerHTML` or equivalent escape hatches?

# Automation (Playwright sample)
```js
import { chromium } from 'playwright';
const url = 'https://app.test/#msg=%3Csvg%20onload%3Dalert(1)%3E';
(async () => {
  const browser = await chromium.launch();
  const page = await browser.newPage();
  let dialogFired = false; page.on('dialog', async d => { dialogFired = true; await d.dismiss(); });
  await page.goto(url);
  console.log('DOM XSS dialog fired:', dialogFired);
  await browser.close();
})();
```

# False Positives & Edge Cases
- Frameworks that **auto-escape** (e.g., virtual DOM templates) will render text safely unless escape hatches are used.
- Event delegation and third-party widgets can introduce unseen sinks; audit external scripts.

# Attack Chain
- **DOM XSS → Token theft:** any script can read **Web Storage** (avoid storing session/long-lived tokens there).  
- **DOM XSS → Cookie readout** if cookies lack `HttpOnly`. Prefer `HttpOnly` for session cookies.  
- **DOM XSS → CSRF-style actions** and UI automation.  
- **DOM XSS → Plant stored XSS:** use authenticated UI features to save malicious content server-side.  
- **DOM XSS → BOLA/IDOR discovery:** iterate over object IDs in front-end API calls to find authorization gaps.

# Standards Mapping
- **WSTG:** Testing for DOM-based XSS.  
- **ASVS v5:** Output Encoding & Escaping; Client-side rendering safety.  
- **CWE:** CWE-79 (Cross-Site Scripting).
