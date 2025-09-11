---
Version: v0.1
Updated: 2025-09-11T08:13:40.198035Z
File: vulnerabilities/xss/payloads/split/non-executable_markers_good_for_diffing_safe_vs_unsafe.md
Vulnerability: XSS
Category: Payload
---
## Non-executable markers (good for diffing safe vs unsafe)
- Angle brackets only: `< >`
- Quote breakers: `"` / `'` / `\`
- Attribute splitters: space, `/>`
- Control chars that should be encoded: `&`, backticks ``` ` ```

---