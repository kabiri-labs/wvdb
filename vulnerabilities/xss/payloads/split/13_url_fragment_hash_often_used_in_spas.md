---
Version: v0.1
Updated: 2025-09-11T08:13:40.198035Z
File: vulnerabilities/xss/payloads/split/13_url_fragment_hash_often_used_in_spas.md
Vulnerability: XSS
Category: Payload
---
## 13) **URL fragment / hash** (often used in SPAs)
If the app reads `location.hash` and writes to DOM:

- **Probe:** `#XSS_TEST`
- **PoC:** `#x=';alert(1);//` → only fires if the app injects into JS string or event attrs.
- **Safe tell:** Hash is parsed safely and rendered via safe APIs with encoding.

---