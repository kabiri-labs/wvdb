---
Version: v0.1
Updated: 2025-09-11T08:13:40.198035Z
File: vulnerabilities/xss/payloads/split/1)_html_**text**_context_(between_tags).md
Vulnerability: XSS
Category: Payload
---
## 1) HTML **Text** context (between tags)
Typical sinks: server template output, `innerHTML` of text nodes.

- **Probe:** `XSS_TEST_<123>` → should render literally.
- **PoC (simple tag):** `<b>X</b>` → if it becomes bold, HTML is interpreted.
- **PoC (evented SVG):** `<svg onload=alert(1)>`
- **Safe tell:** `<` and `>` are encoded (e.g., `&lt;svg onload=alert(1)&gt;`).

**Notes:** If CSP blocks inline execution, `alert(1)` might not fire even though encoding is missing—still a bug. Use “bold-text” probe to detect HTML interpretation.

---