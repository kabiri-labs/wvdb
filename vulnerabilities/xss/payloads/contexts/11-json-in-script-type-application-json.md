---
Vulnerability: XSS
Category: Payload
---
# 11) **JSON in `<script type="application/json">`**

Not executed by the browser by default, but front-end code may parse it.

- **Probe:** `"` → must be JSON-escaped (e.g., `\"`).
- **PoC idea:** Not direct execution; look for **DOM code** that injects parsed values into dangerous sinks.
- **Safe tell:** JSON is safely parsed and rendered with encode-on-output later.

---
