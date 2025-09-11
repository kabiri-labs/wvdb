---
Version: v0.1
Updated: 2025-09-11T08:13:40.198035Z
File: vulnerabilities/xss/payloads/split/9_svg_mathml_context.md
Vulnerability: XSS
Category: Payload
---
## 9) **SVG / MathML** context
When user input is allowed to form SVG/MathML nodes:

- **PoC:** `<svg onload=alert(1)>`
- **Safe tell:** Either SVG is sanitized (allow-list) or plain text encoded; no scripts/events allowed.

**Notes:** Many sanitizers accidentally allow risky SVG/MathML attributes. Prefer strict allow-lists or strip entirely unless necessary.

---