# IDOR – Tenant/Composite Identifiers

## Summary
Composite identifiers (`{org_id}/{user_id}`, `tenant_id`) fail to enforce tenant isolation; cross-tenant reads/writes become possible.

## Impact
- Data leakage across organizations/workspaces
- Privilege escalation if cross-tenant writes are accepted

## Discovery
- Observe patterns like `/orgs/10/users/77` or body fields `{ "tenantId": 10 }`.
- Swap only `org_id`/`tenantId` while keeping a valid `user_id` and session.

## Minimal Exploit
```http
GET /api/v1/orgs/20/invoices/555 HTTP/1.1
Authorization: Bearer {{A_token_of_org10}}
```

## Data-Layer Enforcement (RLS example)
- Preferred: database row-level security (RLS) to enforce tenant isolation.
- PostgreSQL policy sketch:
```sql
CREATE POLICY tenant_isolation ON invoices
USING (tenant_id = current_setting('app.tenant_id')::int);
```
- The app must set `app.tenant_id` from the authenticated session; **never** trust payload `tenantId`.

## Remediation
- Derive `tenant_id` from session/context (JWT claims), not from request payload
- Add RLS/filters at the data layer in addition to service-level checks
- Ensure all list/export endpoints apply tenant filters
