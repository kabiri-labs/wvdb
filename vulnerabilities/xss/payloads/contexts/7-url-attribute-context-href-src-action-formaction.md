---
Vulnerability: XSS
Category: Payload
---
# 7) **URL attribute** context (`href`, `src`, `action`, `formaction`)

Value is placed in a URL-bearing attribute.

- **Probe:** `test` → should become a safe path or be URL-encoded.
- **PoC (blocked in many apps):** `javascript:alert(1)`
- **PoC (data URL):** `data:text/html,<svg onload=alert(1)>` (often blocked by CSP or validators)
- **Safe tell:** Application validates **scheme** (http/https only), encodes characters, and rejects `javascript:` / `data:` for executable contexts.

**Notes:** Scheme allow-lists and URL-encoding are mandatory.

---
