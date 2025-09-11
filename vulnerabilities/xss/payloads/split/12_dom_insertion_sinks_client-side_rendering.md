---
Version: v0.1
Updated: 2025-09-11T08:13:40.198035Z
File: vulnerabilities/xss/payloads/split/12_dom_insertion_sinks_client-side_rendering.md
Vulnerability: XSS
Category: Payload
---
## 12) **DOM insertion sinks** (client-side rendering)
If code writes directly with dangerous sinks:

- **Sinks:** `innerHTML`, `outerHTML`, `document.write`, inline event attributes, `eval`, `Function`, `setTimeout('string')`.
- **PoC:** any of the **HTML Text / Attribute / JS String** payloads above, depending on the final context.
- **Safe tell:** Use `textContent`, element creation, and `setAttribute` with encoding; disallow string-eval.

---