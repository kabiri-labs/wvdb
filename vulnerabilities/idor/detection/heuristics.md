---
Vulnerability: IDOR
Category: Heuristic
---
# Detection Heuristics (Manual & Assisted)

## Identifier hotspots
- Path segments: `/users/{id}`, `/orders/{id}`
- Query: `id, userId, accountId, orderId, invoiceId, orgId, tenantId`
- Body/JSON: nested foreign keys, arrays (`ids`), relationship fields
- Headers: custom `X-Account-Id`, `X-Tenant-Id`

## Signals
- 200/OK (or state change) with foreign data after ID swap
- Differential error/latency when probing foreign vs invalid IDs
- Bulk endpoints partially succeed on mixed-owned IDs
- GraphQL returns nodes outside caller's scope

## Tactics
- Record two sessions (User A/B) and diff requests/responses
- Wordlist from app content for realistic IDs (orders, invoices, file names)
- For UUIDs, leverage predictable sequences, leaked IDs, or logs/URLs

## Tool Recipes
- **Burp**: use Intruder on `id`/`tenantId` positions; `Pitchfork` with dual tokens (A/B). Macros to refresh tokens.
- **ZAP**: Contexts for A and B; use Forced User mode to replay with swapped IDs; Access Control add-on to verify per-role access.
- **GraphQL**: capture known `nodeId`s; if introspection is disabled, rely on app leaks (links/UI) and prior exports.

## SPA/GraphQL
- Monitor WebSocket/subscription channels for leakage
- For batching clients (apollo-link-batch-http), ensure per-request auth still enforces object-level checks
