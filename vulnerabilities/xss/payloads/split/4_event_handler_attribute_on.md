---
Version: v0.1
Updated: 2025-09-11T08:13:40.198035Z
File: vulnerabilities/xss/payloads/split/4_event_handler_attribute_on.md
Vulnerability: XSS
Category: Payload
---
## 4) **Event handler attribute** (on\*)
If the value is written into an event handler attribute (e.g., `<div onclick="<HERE>">`), the interpreter treats it as JS.

- **Probe:** `1` → should be treated as a literal string by safe rendering.
- **PoC:** `alert(1)`
- **Safe tell:** The handler is not created from untrusted input; values are not interpolated into `on*` attributes at all.

**Notes:** The fix is architectural—do not bind user input into event attributes. Use addEventListener and data attributes.

---