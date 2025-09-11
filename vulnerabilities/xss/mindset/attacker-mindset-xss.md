
# Attacker Mindset — XSS

Understanding how an attacker thinks is critical for defending against Cross-Site Scripting (XSS). This file is not about payload collections, but about the **mental process** behind exploiting XSS.

---

## 1. Spotting the opportunity
An attacker does not start with payloads. They first ask:
- *Where do my inputs come back in the response?*
- *Are these values rendered in HTML, attributes, JavaScript, or URLs?*
- *Are there multiple render contexts for the same value?*

The attacker pays attention to tiny reflections, debug messages, hidden fields, even error pages.

---

## 2. Mapping the attack surface
- Inputs: query strings, POST bodies, headers, WebSocket messages, stored fields (comments, profiles, metadata).
- Outputs: HTML pages, dashboards, admin consoles, email templates, export files (CSV/PDF/HTML).
- Render contexts: HTML text, attribute, JavaScript string, URL attribute, DOM sinks.

The attacker thinks: *Can I reach the victim’s browser with my controlled data in a dangerous context?*

---

## 3. Testing hypotheses with minimal probes
Instead of jumping to full `<script>` tags, the attacker tries:
- Simple markers like `XSS_TEST` to track reflection.
- Harmless probes per context (closing quotes, angle brackets).
- Incremental escalation: `><` → `<b>` → `<svg onload=alert(1)>`.

This builds confidence about the context and defenses before attempting impact.

---

## 4. Escalating impact
When probes confirm code execution, the attacker escalates:
- **Session theft:** read cookies if not `HttpOnly`, or tokens from Web Storage.
- **CSRF bypass:** issue stateful requests as the victim.
- **Privilege pivot:** if the victim is an admin, create new users or change settings.
- **Persistence:** plant stored payloads or backdoors for recurring execution.

The attacker is not interested in `alert(1)` but in *real leverage*.

---

## 5. Combining with other weaknesses (attack chains)
XSS rarely lives alone. Attacker mindset links it with:
- **IDOR/BOLA:** use injected scripts to iterate API IDs and detect broken access controls.
- **SQLi discovery:** pivot through admin consoles exposed by XSS to reach SQL interfaces or query builders.
- **Stored → Blind XSS:** use low-privilege injection to target admin dashboards.
- **Second-order paths:** inject into logs, reports, or emails consumed later.

---

## 6. Evading defenses
Attackers experiment with bypasses:
- Context-specific encoding gaps (e.g., JSON inside `<script>`).
- Misconfigured sanitizers (allowed tags/attributes like `<math>`, `<svg>`).
- CSP gaps (inline scripts, wildcard sources).
- Browser quirks or legacy support modes.

---

## 7. Persistence and stealth
Advanced attackers minimize noise:
- Use beacons instead of dialogs.
- Trigger only for targeted victims (e.g., admin accounts).
- Obfuscate payloads to avoid detection.

---

## Defender’s takeaway
To defend effectively, adopt the same questions:
- *Where could my inputs flow back unencoded?*
- *What if an attacker chained this with other flaws (IDOR, CSRF, misconfigurations)?*
- *How do I verify fixes with both probes and impact tests?*

Thinking like an attacker reveals blind spots that static scanners often miss.
