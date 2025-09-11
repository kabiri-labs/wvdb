---
Version: v0.1
Updated: 2025-09-11T08:13:40.198035Z
File: vulnerabilities/xss/payloads/split/2_html_attribute_context_quoted.md
Vulnerability: XSS
Category: Payload
---
## 2) HTML **Attribute** context (quoted)
Value lands inside quotes of an attribute, e.g., `<div title="<HERE>">`

- **Probe:** `"` (double-quote) → should be encoded to `&quot;`.
- **PoC:** `" autofocus onfocus=alert(1) x="`
- **Safe tell:** The quote is escaped; new attributes **do not** appear.

**Variant (single-quoted attributes):**
- **Probe:** `'` → should be encoded to `&#39;` or `&apos;` (HTML5 compatible).
- **PoC:** `' autofocus onfocus=alert(1) a='`

---