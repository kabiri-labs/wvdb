# IDOR – Mass Assignment / Overposting → Ownership Hijack

## Summary
Writable models expose sensitive fields (e.g., `owner_id`, `account_id`, `tenant_id`), letting attackers change record ownership or scope.

## Discovery
- Fuzz JSON/body for hidden or unadvertised fields; compare server diffs
- Inspect client-side models/JS bundles and API docs for candidate fields

## Minimal Exploit
```http
PATCH /api/v1/tickets/901 HTTP/1.1
Authorization: Bearer {{A_token}}
Content-Type: application/json

{ "owner_id": 77, "priority": "low" }
```

## Framework Guardrails
- **Rails**: use `strong_parameters` (`permit`) and ignore unsafe fields by default.
- **Laravel**: define `$fillable`/`$guarded` to allowlist bindable fields.
- **Spring**: map to DTOs; annotate sensitive fields (`@JsonIgnore`) and enforce invariants in service layer.
- **Node/Express**: explicit allowlists; schema validators (e.g., Zod/Joi) with strip-unknown.

## Remediation
- Use explicit allowlists/DTOs; ignore unsafe fields server-side
- Post-validate ownership/tenant invariants before commit
- Add negative tests that attempt to alter `owner_id`/`tenant_id`
