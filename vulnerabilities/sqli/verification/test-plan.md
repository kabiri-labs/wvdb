# SQL Injection Verification – Test Plan

## 1) Scope
Verify SQL injection classes: error-based, union-based, boolean/time-based blind, out-of-band (DNS/HTTP), stacked queries, second-order. DB targets: MySQL, PostgreSQL, MSSQL, Oracle (see `payloads/*.txt`).

## 2) References
- OWASP WSTG v4.2: WSTG-INPV-05
- OWASP ASVS v5: V5.2 (Validation & Sanitization), V5.3 (Output Encoding), V8.3 (Data Access)
- CWE-89

## 3) Pre-requisites
- Proxy + repeatable requests; identify parameters (query, body, JSON, headers, cookies, path segments), batch endpoints, search/sort filters.
- Baseline error behavior & response timing under low load.
- OOB listener available (Burp Collaborator / Interactsh) for DNS/HTTP callbacks.

## 4) Method
1. **Parameter discovery**: spider + passive diff; enumerate hidden/JSON/array params; try method/verb changes.
2. **Heuristic probes**: `'`, `")`, backslash, comment markers (`--+`, `#`, `/*`), arithmetic toggles.
3. **Boolean-based**: condition flip `AND 1=1` vs `AND 1=2`; compare HTML size, JSON count, HTTP code.
4. **Time-based**: `SLEEP/PG_SLEEP/WAITFOR DELAY/DBMS_LOCK.SLEEP`; measure deltas with retries & jitter control.
5. **Union-based**: discover column count, null padding, type alignment; extract banner/current user/db name if safe.
6. **Error-based**: induce DB-specific errors; capture stack/message evidence (sanitize in report).
7. **Stacked/Second-order**: semicolons/batch; seed inputs stored then trigger processing path.
8. **OOB**: `load_file()`, `xp_dirtree`, `UTL_HTTP`/`DBMS_LDAP` (if permitted) to confirm exfil channel.

## 5) Negative/False Positive Controls
- Introduce benign delays server-side baseline to avoid timing bias.
- Cache busters; randomize payload tokens.
- Compare multiple oracles (content-length, token presence, sort order) rather than a single metric.

## 6) Evidence & Repro
- Save paired requests (true/false), timing table (median of N), error messages, and any extracted sample data (masked).
- Include DB fingerprint inference (banner/version) only if passively observed.

## 7) Severity & Impact
- **Critical**: read/write filesystem, RCE via `xp_cmdshell`/UDFs, admin credential dump.
- **High**: data extraction/modification, privilege escalation.
- **Medium**: Boolean-only inference without sensitive impact demonstrated.

## 8) Remediation Validation
- Prepared statements/parameterized queries; ORM safe APIs.
- Input allowlists for identifiers; escaping for LIKE, ORDER BY, dynamic fragments.
- Least-privilege DB accounts; no dangerous procs; WAF rules as compensating control only.
