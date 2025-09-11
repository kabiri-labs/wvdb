# Preventing SQL Injection with Prepared Statements & Safe Query Construction

> Audience: Developers/architects. Goal: Practical, language‑specific patterns to eliminate SQL injection by using **parameterized queries**, safe identifier mapping, and minimal dynamic SQL. Includes copy‑paste examples and anti‑patterns to **avoid**.

---

## 1) Core Principles
- **Never** concatenate untrusted data into SQL. Use **bound parameters** (`?`, `$1`, `@p1`, named params) for **values**.
- For **identifiers** (table/column names, ORDER BY), use **allowlists** and server‑side mapping – *never* pass raw identifiers from clients.
- Separate **query shape** from **data**: the SQL text is constant; only parameter values change.
- Apply **least privilege** on the DB: read‑only for reads, no DDL/DCL for app users.
- Centralize DB access helpers so the entire codebase follows the same safe patterns.

---

## 2) Language / Framework Cheat‑Sheet

### Java (JDBC)
**✅ Do**
```java
String sql = "SELECT id, name FROM users WHERE email = ?";
try (PreparedStatement ps = conn.prepareStatement(sql)) {
  ps.setString(1, email);
  try (ResultSet rs = ps.executeQuery()) { /* ... */ }
}
```
**❌ Don’t**
```java
String sql = "SELECT * FROM users WHERE email = '" + email + "'"; // vulnerable
```

### Spring Data / JPA
**✅ Do**
```java
@Query("SELECT u FROM User u WHERE u.email = :email")
User findByEmail(@Param("email") String email);
```
**Note:** Use parameter binding even in native queries.

### Node.js (pg / node‑postgres)
**✅ Do**
```js
const text = 'SELECT id, name FROM users WHERE email = $1';
const values = [email];
const { rows } = await client.query(text, values);
```
**❌ Don’t**
```js
const res = await client.query(`SELECT * FROM users WHERE email = '${email}'`);
```

### Python (psycopg / PostgreSQL)
**✅ Do**
```python
cur.execute("SELECT id, name FROM users WHERE email = %s", (email,))
```
**❌ Don’t**
```python
cur.execute(f"SELECT * FROM users WHERE email = '{email}'")
```

### Python (mysql‑connector / pymysql)
**✅ Do**
```python
cur.execute("SELECT id, name FROM users WHERE email = %s", (email,))
```

### .NET (C# / SqlClient / Dapper)
**✅ Do**
```csharp
using var cmd = new SqlCommand("SELECT Id, Name FROM Users WHERE Email = @email", conn);
cmd.Parameters.AddWithValue("@email", email);
using var r = cmd.ExecuteReader();
```
**Dapper**
```csharp
var users = conn.Query<User>("SELECT Id, Name FROM Users WHERE Email = @email", new { email });
```
**❌ Don’t**
```csharp
var sql = $"SELECT * FROM Users WHERE Email = '{email}'"; // vulnerable
```

### PHP (PDO)
**✅ Do**
```php
$stmt = $pdo->prepare('SELECT id, name FROM users WHERE email = :email');
$stmt->execute(['email' => $email]);
```
**❌ Don’t**
```php
$sql = "SELECT * FROM users WHERE email = '".$_GET['email']."'"; // vulnerable
```

### Go (database/sql)
**✅ Do**
```go
row := db.QueryRowContext(ctx, "SELECT id, name FROM users WHERE email = ?", email)
```

### Ruby (ActiveRecord)
**✅ Do**
```rb
User.where("email = ?", email)
```
**❌ Don’t**
```rb
User.where("email = '#{params[:email]}'")
```

---

## 3) Safe ORDER BY / Identifiers (Allowlist Mapping)
**Problem:** Parameters do **not** bind identifiers (e.g., column names in ORDER BY).  
**Solution:** Map client tokens to known safe identifiers.
```python
# Python example
ALLOWED_SORTS = { 'name': 'name', 'created': 'created_at', 'price': 'price' }
sort = ALLOWED_SORTS.get(request.args.get('sort'), 'created_at')
order = 'ASC' if request.args.get('dir') == 'asc' else 'DESC'
sql = f"SELECT id, name, price FROM items ORDER BY {sort} {order} LIMIT %s OFFSET %s"
cur.execute(sql, (limit, offset))
```
**Tip:** Validate direction separately, never interpolate raw, and consider **prepared statements with dynamic SQL** in stored procedures if you must.

---

## 4) Escaping vs Parameterization
- **Escaping** is brittle and DB‑specific; do **not** rely on manual quoting/escaping to prevent injection.
- **Parameterization** defers literal handling to the driver, which encodes safely and avoids changing query structure.

---

## 5) ORMs & Query Builders
- Prefer ORM methods that **bind parameters** (e.g., `where`, `findBy...`).
- Avoid raw SQL strings; if unavoidable, ensure placeholders and bindings are used.
- Be cautious with **string interpolation** features (template literals, f‑strings) around SQL.

---

## 6) Stored Procedures — Only with Parameters
- Stored procedures **do not** prevent SQLi by themselves. Ensure **all** dynamic SQL inside procs uses bind variables.
- Example (PostgreSQL):
```sql
CREATE OR REPLACE FUNCTION get_user(p_email text)
RETURNS TABLE(id int, name text) AS $$
BEGIN
  RETURN QUERY
  SELECT id, name FROM users WHERE email = p_email;  -- bound param
END;
$$ LANGUAGE plpgsql;
```

---

## 7) Defense‑in‑Depth Controls
- **Least privilege** DB users; separate read vs write; avoid owner/DBA contexts for apps.
- **Centralized error handling**: no DB error banners to clients.
- **Input validation**: type/format checks; reject control chars; enforce length limits.
- **Egress filtering** for DB subnets; block unnecessary DNS/HTTP/SMB to reduce OOB risks.
- **Secrets hygiene**: rotate creds; avoid shared accounts; use parameterized logging (no SQL text with user data).

---

## 8) Tests & Tooling
- Unit/integration tests covering all SQL‑building code paths; include **negative tests** with attacker‑style inputs.
- Add a **DAST canary suite** with benign toggles (`1=1/1=2`) and short sleeps to catch regressions.
- Static analysis rules: flag concatenation near SQL strings, dynamic ORDER BY, and string‑built identifiers.

---

## 9) Anti‑Patterns to Eliminate
- Building SQL with `+` or template strings from request params.
- Passing client‑provided column/table names directly into ORDER BY / JOIN / INSERT.
- Relying on WAF alone to stop injections.
- Swallowing DB errors silently (hides exploitable paths in logs and tests).

---

## 10) Minimal Engineering Checklist
- [ ] All data literals bound via parameters
- [ ] Identifier allowlists & mapping in place (ORDER BY, GROUP BY)
- [ ] Centralized DB helper enforcing prepared statements
- [ ] App DB users have least privilege (no DDL/DCL)
- [ ] Errors centralized & generic to clients
- [ ] Tests & DAST canaries added; static rules enabled

---

## 11) References
- OWASP Cheat Sheet: SQL Injection Prevention
- Vendor docs: JDBC PreparedStatement, Npgsql, SqlClient, PDO, psycopg, node‑postgres
- Query builders/ORMs: JPA/Hibernate, Sequelize, Knex, TypeORM, ActiveRecord
