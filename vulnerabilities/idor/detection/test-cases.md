---
Vulnerability: IDOR
Category: Heuristic
---
# Portable Test Cases

- **TC1 – Basic read:** Swap `id` in GET detail; expect 403/404 (fail if 200)
- **TC2 – Basic write:** PUT/PATCH foreign `id`; expect 403/404 (fail if 2xx)
- **TC3 – Tenant isolation:** Change `tenantId` only; expect isolation
- **TC4 – Bulk mix:** Send owned+unowned `ids`; ensure per-item enforcement
- **TC5 – Mass assignment:** Attempt to set `owner_id`/`account_id`
- **TC6 – File download:** Access adjacent file keys/paths
- **TC7 – GraphQL:** Swap global ID in query/mutation
