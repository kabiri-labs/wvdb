# Remediation Checklist – IDOR/BOLA

- [ ] Ownership/tenant checks on **every** read/write
- [ ] Derive tenant/account from session, not client payload
- [ ] Per-item authorization in bulk APIs
- [ ] DTO/whitelist – ignore unsafe fields (owner/account/tenant)
- [ ] Authorization in each GraphQL resolver/mutation
- [ ] File gateways enforce authorization before serving data
- [ ] Negative tests included in CI
