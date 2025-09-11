# XSS Verification – Test Plan

## 1) Scope
Verify reflected, stored, DOM-based, and blind XSS across common contexts (HTML text, attribute, JS string/raw, URL, CSS, SVG/MathML).

## 2) References
- OWASP WSTG v4.2: WSTG-CLNT-01/02/03 (DOM/Reflected/Stored XSS)
- OWASP ASVS v5: V5.3 (Output Encoding), V5.4 (Template/JS), V4.3 (Content Security)
- CWE-79, CWE-116

## 3) Pre-requisites
- Authenticated user (if app requires) with test roles.
- Proxy set up (Burp/ZAP) + headless browser (Puppeteer/Playwright) for auto-detection.
- Payload set: see `vulnerabilities/xss/payloads/contexts/*.md`.
- CSP/Headers baseline captured (CSP, X-Content-Type-Options, X-Frame-Options, Referrer-Policy).

## 4) Method
1. **Mapping sinks/sources** (grep/DevTools): identify DOM APIs (innerHTML, document.write, location, postMessage).
2. **Context-driven payloads**: select payload per context; avoid noise with minimal proof payloads first.
3. **Delivery channels**: query params, body (JSON/Form), headers, WebSocket, stored inputs (comments/profile), second-order triggers (notifications, admin panels).
4. **Execution validation**: observe via headless browser listeners (alert/console/error/network beacon). See `tools/xbrowser/` if available.
5. **Bypass attempts**: encoding, event attributes, protocol changes (`javascript:`, `data:`), HTML5 quirks, SVG, CSS `expression()` (legacy), template injection to JS.

## 5) Test Cases (High-level)
- **Reflected**: echo in HTML text/attr/JS. Expect JS execution.
- **Stored**: persist payload; revisit display contexts; test delayed/notification paths.
- **DOM**: sink tainted by URL hash/search, postMessage, localStorage. No server reflection needed.
- **Blind**: use OOB canary (unique token) → receive DNS/HTTP callback or logged beacon.
- **Framework-specific**: templating (Angular/React/Vue) unsafe bindings, `v-html`, `dangerouslySetInnerHTML`, legacy jQuery `html()`.
- **Rich inputs**: Markdown/WYSIWYG uploaders; test sanitizer rules/bypass (nested tags, broken attributes, mathml/svg).

## 6) Negative/False Positive Controls
- Mirror tests with aggressive encoding server-side; verify no execution.
- Ensure payload reflections inside attributes are properly quoted & encoded for the exact context.
- Confirm CSP blocks inline/script URLs (report-only first).

## 7) Evidence & Repro
- Record request/response pairs, DOM snippet (before/after), console logs, network beacon ID, and screenshot/GIF from headless run.
- Minimal PoC HTML link or cURL + repro steps.

## 8) Severity & Impact
- **Critical**: admin scope, session hijack, arbitrary requests on sensitive endpoints, supply chain widgets.
- **High**: authenticated user data theft/privilege actions.
- **Medium**: limited scope or strong CSP blocks most vectors.
- Map to business impact (PII/PCI/PHI).

## 9) Remediation Validation
- Verify context-appropriate output encoding (HTML, attr, JS, URL, CSS), input validation for structural tokens, strict CSP (no `unsafe-inline`), templating safe APIs, sanitizer unit tests.
