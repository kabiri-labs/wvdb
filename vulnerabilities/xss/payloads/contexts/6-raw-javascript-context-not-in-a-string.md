---
Vulnerability: XSS
Category: Payload
---
# 6) **Raw JavaScript** context (not in a string)

If a value lands directly in code position (rare in safe templates):

- **Probe:** `1` → should behave as a constant.
- **PoC:** `alert(1)` (dangerous; avoid constructs that allow this).
- **Safe tell:** Untrusted values are **never** concatenated into raw code; use data attributes or JSON.

---
