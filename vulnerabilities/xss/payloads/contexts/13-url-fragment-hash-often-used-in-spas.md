---
Vulnerability: XSS
Category: Payload
---
# 13) **URL fragment / hash** (often used in SPAs)

If the app reads `location.hash` and writes to DOM:

- **Probe:** `#XSS_TEST`
- **PoC:** `#x=';alert(1);//` → only fires if the app injects into JS string or event attrs.
- **Safe tell:** Hash is parsed safely and rendered via safe APIs with encoding.

---
