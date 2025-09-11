---
Version: v0.1
Updated: 2025-09-11T08:13:40.198035Z
File: vulnerabilities/xss/payloads/split/quick_cross-check_when_a_payload_doesn_t_fire.md
Vulnerability: XSS
Category: Payload
---
## Quick cross-check when a payload "doesn't fire"
- Check CSP (it may block inline execution while encoding is still wrong).
- Confirm the **exact** final context (HTML text vs attribute vs JS string).
- Ensure your PoC matches that context’s escaping rules.
- For blind scenarios, use an **OOB** beacon instead of dialogs.

---

*Pair this catalog with the XSS stories in `vulns/xss/` for step-by-step: probes → PoC → fix → verification → detection → attack chain.*