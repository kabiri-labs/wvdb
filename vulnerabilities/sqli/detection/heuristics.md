---
Vulnerability: SQLi
Category: Heuristic
---
# SQLi Detection Heuristics

> Purpose: Practical signals and playbooks for **detecting SQL injection** (union, error-based, boolean/time-blind, OOB, stacked, second-order) across **web/API** apps. Aimed at DAST authors, blue teams, and observability engineers.

---

## 1) Signal Taxonomy
- **Content deltas**: HTML/JSON size change, element presence/absence, template switch.
- **Error banners**: DB/vendor messages, stack traces, ORM/driver errors.
- **Timing anomalies**: latency spikes clustered at precise intervals (e.g., ~3s, ~5s).
- **Out-of-band callbacks**: DNS/HTTP/SMB egress originating during request handling.
- **State/side-effects**: temp rows/files, changed settings; stacked/piggy‑backed execution.
- **DB log anomalies**: syntax/type/permission errors, multi‑statement batches.

---

## 2) Heuristics by Technique
### A) Union-based
- **A/B deltas** on paired inputs: `AND 1=1` (control TRUE) vs `AND 1=2` (control FALSE) produce **consistent length/status differences**.
- Injections containing `UNION SELECT` or **null padding** cause layout/length changes once **column count matches**.
- **Error signatures** while calibrating:
  - *MySQL*: `Unknown column`, `You have an error in your SQL syntax`, `Illegal mix of collations`.
  - *MSSQL*: `All queries combined using a UNION... must have an equal number of expressions`, `Incorrect syntax near 'UNION'`.
  - *PostgreSQL*: `UNION types ... are not compatible`, `syntax error at or near "UNION"`.
  - *Oracle*: `ORA-01789: query block has incorrect number of result columns`.
- **Printable column discovery**: presence of unique markers (e.g., `q9xZ...`) in response.

### B) Error-based
- Look for **engine‑specific** messages that include attacker data or clear DB tokens:
  - *MySQL/MariaDB*: `XPATH syntax error:` (from `EXTRACTVALUE/UPDATEXML`), `Truncated incorrect integer value`.
  - *MSSQL*: `Conversion failed when converting the varchar value '...' to data type int`, `Unclosed quotation mark`, `Incorrect syntax near`.
  - *PostgreSQL*: `invalid input syntax for type integer: "..."`, `division by zero`, `operator does not exist`.
  - *Oracle*: `ORA-01722: invalid number`, `ORA-00933`, `ORA-00936`, `ORA-01476`.
- **Differentiation**: Ensure the banner appears **only** with crafted inputs and disappears with benign ones.

### C) Boolean-based Blind
- **Two‑cluster behavior**: responses fall into two size/status buckets for TRUE vs FALSE.
- Stable **DOM diffs** (presence of strings like `No results`, pagination widgets) across repeated trials.
- Use **median of 3** response sizes to reduce jitter; flag when median distance > *k* bytes (tune per endpoint).

### D) Time-based Blind
- Establish **baseline latency** (median/95th) over ≥10 benign requests.
- Run **paired tests** with predicates that trigger delays on TRUE (e.g., `SLEEP/WAITFOR/pg_sleep/dbms_lock.sleep`).
- Classify TRUE when **median latency** exceeds baseline by `Δ >= threshold` (e.g., ≥2000 ms above 95th percentile). Use **n≥3** samples.
- Detect **quantized spikes** around specific intervals (2s/3s/5s) aligned with inputs.

### E) Out-of-Band (OOB)
- Monitor **resolver and proxy logs** for callbacks shortly after suspicious requests:
  - DNS labels with **hex/base32 chunks** or app tokens (`q9xZ`).
  - HTTP beacons to unusual hosts; SMB/LDAP attempts from DB servers.
- Correlate **request → callback** using unique **correlation IDs** embedded in payloads.

### F) Stacked/Piggy‑backed Queries
- Presence of `;` followed by a second statement (e.g., `; WAITFOR DELAY`, `; SELECT pg_sleep(3)`, `; BEGIN ... END;`).
- DB logs show **multi‑statement** batches in a single request execution.
- Short, reproducible **delays** without content change.

### G) Second‑order SQLi
- **Seed‑and‑recall**: plant inert markers in write paths, later observe admin/report/batch paths for activation (errors, delays, banner leaks).
- Temporal correlation between **ingestion time** and **activation window** (cron/ETL run).
- Logs show dynamic SQL construction using stored fields (search/report builders).

---

## 3) Robust A/B Testing Recipe (DAST)
1. **Stabilize**: fix headers (User‑Agent), disable caching via `Cache-Control: no-store` and **vary a nonce** to bypass CDN.
2. **Triplets** per probe: send **control**, **variant‑TRUE**, **variant‑FALSE**.
3. Compute **metrics** per triplet: status code, body length, normalized DOM hash (strip timestamps/CSRF), TTFB/latency.
4. **Decision**: flag if two of three metrics differ consistently TRUE vs FALSE across **k ≥ 3** triplets.
5. **Safety**: cap payload count & rate; rotate payloads to avoid WAF learning.

---

## 4) Log Field Checklist (collect & index)
- **Edge/App**: timestamp, src IP/user, method, path, query/body (tokenized), status, bytes, **time_taken**, route/controller, request ID/correlation ID.
- **DB/ORM**: SQL text (redacted), bind params (hashed), duration, error class & message, connection ID/session ID.
- **Network**: DNS queries from app/DB subnets, egress HTTP/SMB/LDAP attempts.

---

## 5) Example Queries (quick wins)
### Splunk (web edge)
```
index=web ("UNION SELECT" OR "WAITFOR DELAY" OR "pg_sleep" OR "dbms_lock.sleep" OR "UTL_HTTP" OR "xp_dirtree")
| stats count by src, uri, user
```
### Elastic/KQL (time spikes)
```
url.path : * AND http.response.status_code : 200 and event.duration >= 3000000000
and http.request.body.content : ("AND SLEEP" or "WAITFOR DELAY" or "pg_sleep")
```
### SQL‑error banners (generic)
```
message : ("invalid number" or "syntax error at or near" or "Conversion failed when converting" or "XPATH syntax error")
```
*(Tune field names to your schema.)*

---

## 6) Regex Snippets (signature layer)
- **Keyword splitting / mixed case** (UNION SELECT within 16 chars gap):
```
U\w{0,8}N\w{0,8}I\w{0,8}O\w{0,8}N\s+S\w{0,8}E\w{0,8}L\w{0,8}E\w{0,8}C\w{0,8}T
```
- **Sleep primitives**:
```
(?i)\bWAITFOR\s+DELAY\b|\bpg_sleep\s*\(|\bSLEEP\s*\(|dbms_\w*\.sleep\s*\(
```
- **OOB packages/procs**:
```
(?i)\bxp_dirtree\b|\bUTL_HTTP\b|\bUTL_INADDR\b|\bDBMS_LDAP\b|COPY\s*\(.*\)\s*TO\s*PROGRAM
```

---

## 7) False Positives / Negatives
**FP sources**: debug pages in dev, legitimate admin reports calling sleep (load tests), complex search pages with large result variance, CDN retries.
**FN sources**: WAF normalizing responses, cached pages masking deltas, asynchronous backends (queueing hides latency), aggressive rate limiting.

**Mitigations**:
- Normalize responses (strip volatile fields) before hashing.
- Use **paired** controls; re‑try near thresholds; require **consensus across metrics** (status+length+latency).
- Apply **per‑endpoint baselines**; do not reuse thresholds globally.

---

## 8) WAF/IDS Awareness
- Attackers may use: mixed case, inline comments (`/**/`), hex/CHR builders, Unicode homoglyphs, or **no‑quotes** numeric predicates.
- Detection should combine **behavioral** + **signature** layers; ban‑lists alone are brittle.
- Alert on **dense clusters** of suspicious tokens from a single source within a short window.

---

## 9) Second‑order Playbook (Blue Team)
- Build **seed→discover** jobs: plant benign markers in non‑critical fields in staging; run admin/report flows; confirm no dynamic concatenation.
- Instrument **admin/report** UIs and **batch jobs** with: SQL text sampling (redacted), execution duration, error capture.
- Add CI checks: static scans for string concatenation near SQL builders; unit tests that pass seeded values through.

---

## 10) CI/CD Hooks (DAST & Regression)
- Add **headless browser DAST** steps for stored/second‑order paths (login → navigate → verify marker/time).
- Maintain a **canary payload set** (`q9xZ` markers, short `sleep(2)` probes) gated by rate/timeout.
- Fail builds on **confirmed** behavioral SQLi (A/B/T timing consensus), not on single raw signatures.

---

## 11) Minimal Analyst Checklist
- [ ] Endpoint baseline captured (status/length/latency)
- [ ] A/B probes produce stable deltas or time spikes
- [ ] Engine hints present (banners or function tokens)
- [ ] OOB monitored and correlated (if used)
- [ ] Findings reproduced with **benign, minimal** payloads
- [ ] Engineering given a **parameterization** fix path and reproduction steps

---

## 12) References & Next
- See `/sqli/types/*` for exploitation specifics and `/sqli/db-notes/*` for engine behaviors.
- For prevention patterns, proceed to `/sqli/mitigations/prepared-statements.md`.
