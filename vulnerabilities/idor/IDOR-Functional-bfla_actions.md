# BFLA – Unauthorized Actions on Others (IDOR-like)

## Summary
Endpoint authorizes the caller but not the **target object**, enabling actions (approve/cancel/reset/change password) on others.

## Discovery
- Look for `?userId=` in action endpoints or `targetId` in bodies

## Minimal Exploit
- Trigger an action against a victim's ID; observe state change without proper authorization

## Remediation
- Bind actions to server-resolved target from the authenticated subject and verify policy on both subject and object
