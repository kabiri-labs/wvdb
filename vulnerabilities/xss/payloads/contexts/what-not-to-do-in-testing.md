---
Vulnerability: XSS
Category: Payload
---
# What **not** to do in testing

- Do not use obfuscated/polyglot payloads against production systems.
- Do not brute-force payload lists; focus on **context** first.
- Do not rely on `alert(1)` alone; always show a **minimal real impact** in a safe environment.

---
