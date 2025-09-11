# IDOR – Basic Object-Level

## Summary
Access or modify another user's object by changing an identifier (path/query/body) without server-side ownership checks.

## Impact
- Read/modify/erase other users' data
- Account takeover of linked resources

## Discovery (Manual)
1. Identify object IDs in path (`/api/v1/users/123/profile`), query (`?id=123`), or JSON body.
2. Log in as User A; capture a baseline request.
3. Replay with another known/guessed ID belonging to User B; observe diffs.

## Discovery (Automation hints)
- Parameter mining for `id`, `userId`, `accountId`, `orderId`, `invoiceId`, `orgId`, `tenantId`, `nodeId` (GraphQL).
- Fuzz numeric ranges, timestamp-like IDs, and UUIDs using wordlists harvested from UI/JS.
- Two-account diffing: replay User A's requests while substituting User B's IDs.

## Minimal Exploit (HTTP)
```http
# A: detail view (owned)
GET /api/v1/orders/1029 HTTP/1.1
Authorization: Bearer {{A_token}}

# Swap to a victim ID (foreign)
GET /api/v1/orders/1030 HTTP/1.1
Authorization: Bearer {{A_token}}
```

## Minimal Exploit (Body-held ID)
```http
PATCH /api/v1/profile HTTP/1.1
Authorization: Bearer {{A_token}}
Content-Type: application/json

{ "targetUserId": 77, "bio": "test" }
```

## Verification
- Repeat with at least two distinct victims
- Ensure no allowlist exceptions; verify the backend truly enforces ownership per object

## Common Anti-Patterns
- Only checking that the caller is authenticated (no per-object check)
- Trusting client-supplied foreign keys on write paths
- Hiding buttons in UI but leaving unprotected endpoints
- Returning different errors for foreign vs. missing IDs (enumeration)

## Root Cause
- Business logic trusts client-supplied identifiers
- Authorization checks performed only at session level, not object level

## Remediation
- Resolve the object by owner on the server (e.g., `SELECT * FROM orders WHERE id = :id AND owner_id = :sub`)
- Centralize authorization checks; avoid sprinkling ad-hoc `if` guards

## Regression Tests
- Unit/integration: attempts to access non-owned IDs must 403/404 consistently
- Negative tests for mixed-tenant scenarios
