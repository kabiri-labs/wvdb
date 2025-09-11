---
Vulnerability: XSS
Category: Payload
---
# Quick cross-check when a payload "doesn't fire"

- Check CSP (it may block inline execution while encoding is still wrong).
- Confirm the **exact** final context (HTML text vs attribute vs JS string).
- Ensure your PoC matches that context’s escaping rules.
- For blind scenarios, use an **OOB** beacon instead of dialogs.

---

*Pair this catalog with the XSS stories in `vulns/xss/` for step-by-step: probes → PoC → fix → verification → detection → attack chain.*
