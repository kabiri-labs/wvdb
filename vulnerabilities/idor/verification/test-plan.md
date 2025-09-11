# Verification Plan – IDOR Fixes

## Scope
List endpoints and objects covered by the fix.

## Tests
- Unit tests asserting 403/404 for non-owned objects
- Integration tests for bulk arrays with mixed ownership
- E2E tests for cross-tenant isolation
- GraphQL resolver tests

## Acceptance
- All tests pass; logs show attempted cross-object access blocked
- No leakage in list/search/export endpoints
