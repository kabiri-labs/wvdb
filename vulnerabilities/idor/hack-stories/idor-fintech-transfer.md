# Hack Story: Unauthorized Balance Transfers via Mobile API (BFLA/IDOR)

## Context
A fintech mobile app exposed an API endpoint:
```
POST /api/v1/transfer
{
   "fromAccount": "1234567890",
   "toAccount": "9876543210",
   "amount": "500"
}
```

## Exploit
The server validated the session but did not check if the `fromAccount` belonged to the logged-in user.
By intercepting the request and modifying `fromAccount`, an attacker could transfer money from **any account**.

## Impact
- Full compromise of financial integrity
- Direct financial theft
- Loss of trust and regulatory fines

## Lessons
- Enforce function-level authorization checks on every sensitive action
- Use server-side session binding to user accounts
