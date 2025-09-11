---
Vulnerability: XSS
Category: Payload
---
# XSS Payloads Index

This index lists XSS payload collections split by context for consistency with SQLi payload organization.

| Context | File |
|---|---|
| Legend | `vulnerabilities/xss/payloads/contexts/legend.md` |
| 1) HTML **Text** context (between tags) | `vulnerabilities/xss/payloads/contexts/1-html-text-context-between-tags.md` |
| 2) HTML **Attribute** context (quoted) | `vulnerabilities/xss/payloads/contexts/2-html-attribute-context-quoted.md` |
| 3) HTML **Attribute** context (unquoted) | `vulnerabilities/xss/payloads/contexts/3-html-attribute-context-unquoted.md` |
| 4) **Event handler attribute** (on\*) | `vulnerabilities/xss/payloads/contexts/4-event-handler-attribute-on.md` |
| 5) **JavaScript string** context (inside `<script>` as a string) | `vulnerabilities/xss/payloads/contexts/5-javascript-string-context-inside-script-as-a-string.md` |
| 6) **Raw JavaScript** context (not in a string) | `vulnerabilities/xss/payloads/contexts/6-raw-javascript-context-not-in-a-string.md` |
| 7) **URL attribute** context (`href`, `src`, `action`, `formaction`) | `vulnerabilities/xss/payloads/contexts/7-url-attribute-context-href-src-action-formaction.md` |
| 8) **Style** / CSS context | `vulnerabilities/xss/payloads/contexts/8-style-css-context.md` |
| 9) **SVG / MathML** context | `vulnerabilities/xss/payloads/contexts/9-svg-mathml-context.md` |
| 10) **HTML comment** context | `vulnerabilities/xss/payloads/contexts/10-html-comment-context.md` |
| 11) **JSON in `<script type="application/json">`** | `vulnerabilities/xss/payloads/contexts/11-json-in-script-type-application-json.md` |
| 12) **DOM insertion sinks** (client-side rendering) | `vulnerabilities/xss/payloads/contexts/12-dom-insertion-sinks-client-side-rendering.md` |
| 13) **URL fragment / hash** (often used in SPAs) | `vulnerabilities/xss/payloads/contexts/13-url-fragment-hash-often-used-in-spas.md` |
| 14) **postMessage** / cross-document messaging | `vulnerabilities/xss/payloads/contexts/14-postmessage-cross-document-messaging.md` |
| 15) **Email/Report renderers** (HTML emails, PDF/HTML exports) | `vulnerabilities/xss/payloads/contexts/15-email-report-renderers-html-emails-pdf-html-exports.md` |
| Non-executable markers (good for diffing safe vs unsafe) | `vulnerabilities/xss/payloads/contexts/non-executable-markers-good-for-diffing-safe-vs-unsafe.md` |
| What **not** to do in testing | `vulnerabilities/xss/payloads/contexts/what-not-to-do-in-testing.md` |
| Quick cross-check when a payload "doesn't fire" | `vulnerabilities/xss/payloads/contexts/quick-cross-check-when-a-payload-doesn-t-fire.md` |
