---
Version: v0.1
Updated: 2025-09-11T08:13:40.198035Z
File: vulnerabilities/xss/payloads/split/10_html_comment_context.md
Vulnerability: XSS
Category: Payload
---
## 10) **HTML comment** context
If a value lands in `<!-- <HERE> -->`:

- **Probe:** `-->` → should be encoded or prevented; otherwise, can break out into HTML/JS contexts depending on template.
- **Safe tell:** Comments do not become execution vectors; breaking out of comments should not be possible.

---