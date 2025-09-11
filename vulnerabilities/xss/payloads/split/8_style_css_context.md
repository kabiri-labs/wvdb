---
Version: v0.1
Updated: 2025-09-11T08:13:40.198035Z
File: vulnerabilities/xss/payloads/split/8_style_css_context.md
Vulnerability: XSS
Category: Payload
---
## 8) **Style** / CSS context
Modern browsers do **not** execute JS from CSS. However, CSS can still leak data or make network requests (e.g., `url()`), but not XSS in modern engines.

- **Probe:** `color:red` → should be treated as data.
- **Safe tell:** User input is not interpolated into raw `style` strings; instead, use sanitized style maps.

**Notes:** Historical `expression()` was IE-only and obsolete.

---