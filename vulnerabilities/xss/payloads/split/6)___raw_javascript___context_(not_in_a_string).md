---
Version: v0.1
Updated: 2025-09-11T08:13:40.198035Z
File: vulnerabilities/xss/payloads/split/6)_**raw_javascript**_context_(not_in_a_string).md
Vulnerability: XSS
Category: Payload
---
## 6) **Raw JavaScript** context (not in a string)
If a value lands directly in code position (rare in safe templates):

- **Probe:** `1` → should behave as a constant.
- **PoC:** `alert(1)` (dangerous; avoid constructs that allow this).
- **Safe tell:** Untrusted values are **never** concatenated into raw code; use data attributes or JSON.

---