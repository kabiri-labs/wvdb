---
Version: v0.1
Updated: 2025-09-11T08:13:40.198035Z
File: vulnerabilities/xss/payloads/split/14_postmessage_cross-document_messaging.md
Vulnerability: XSS
Category: Payload
---
## 14) **postMessage** / cross-document messaging
If the app consumes messages and injects into DOM:

- **Probe:** `{ "msg": "XSS_TEST" }`
- **PoC:** `<img src=x onerror=alert(1)>` passed as message data (only executes if injected into a dangerous sink).
- **Safe tell:** Messages are validated; rendering uses safe APIs.

---