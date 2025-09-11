---
Vulnerability: XSS
Category: Payload
---
# 3) HTML **Attribute** context (unquoted)

Value lands after `=` without surrounding quotes, e.g., `<img alt=<HERE>>`

- **Probe:** ` test` (space) → should be encoded; space must **not** create a new attribute.
- **PoC:** ` onmouseover=alert(1)`
- **Safe tell:** No new attributes; the value is quoted by the template or safely encoded.

**Notes:** Avoid unquoted attributes in templates; they are error-prone.

---
