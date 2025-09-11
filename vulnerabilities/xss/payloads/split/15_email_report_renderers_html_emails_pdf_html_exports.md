---
Version: v0.1
Updated: 2025-09-11T08:13:40.198035Z
File: vulnerabilities/xss/payloads/split/15_email_report_renderers_html_emails_pdf_html_exports.md
Vulnerability: XSS
Category: Payload
---
## 15) **Email/Report renderers** (HTML emails, PDF/HTML exports)
Payload executes in **another** renderer (Blind XSS scenarios).

- **Probe:** `XSS_TEST_EMAIL`
- **PoC:** `<svg onload=alert(1)>` (many email clients block scripts; still relevant in internal tools)
- **Safe tell:** Templates sanitize/encode; risky tags/attrs stripped.

---