---
Vulnerability: XSS
Category: Payload
---
# 2) HTML **Attribute** context (quoted)

Value lands inside quotes of an attribute, e.g., `<div title="<HERE>">`

- **Probe:** `"` (double-quote) → should be encoded to `&quot;`.
- **PoC:** `" autofocus onfocus=alert(1) x="`
- **Safe tell:** The quote is escaped; new attributes **do not** appear.

**Variant (single-quoted attributes):**
- **Probe:** `'` → should be encoded to `&#39;` or `&apos;` (HTML5 compatible).
- **PoC:** `' autofocus onfocus=alert(1) a='`

---
