---
Vulnerability: XSS
Category: Payload
---
# 8) **Style** / CSS context

Modern browsers do **not** execute JS from CSS. However, CSS can still leak data or make network requests (e.g., `url()`), but not XSS in modern engines.

- **Probe:** `color:red` → should be treated as data.
- **Safe tell:** User input is not interpolated into raw `style` strings; instead, use sanitized style maps.

**Notes:** Historical `expression()` was IE-only and obsolete.

---
