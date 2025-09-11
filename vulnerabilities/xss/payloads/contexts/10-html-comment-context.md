---
Vulnerability: XSS
Category: Payload
---
# 10) **HTML comment** context

If a value lands in `<!-- <HERE> -->`:

- **Probe:** `-->` → should be encoded or prevented; otherwise, can break out into HTML/JS contexts depending on template.
- **Safe tell:** Comments do not become execution vectors; breaking out of comments should not be possible.

---
