---
Vulnerability: XSS
Category: Payload
---
# 14) **postMessage** / cross-document messaging

If the app consumes messages and injects into DOM:

- **Probe:** `{ "msg": "XSS_TEST" }`
- **PoC:** `<img src=x onerror=alert(1)>` passed as message data (only executes if injected into a dangerous sink).
- **Safe tell:** Messages are validated; rendering uses safe APIs.

---
