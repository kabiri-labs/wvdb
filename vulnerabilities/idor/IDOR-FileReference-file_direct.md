# IDOR – Direct File/Resource Reference

## Summary
Files are served by guessable/iterable identifiers or paths without per-request authorization.

## Discovery
- Test endpoints like `/files/12345.pdf`, `/download?fileId=abc`
- Try directory/ID stepping, alternate extensions, and `HEAD`/`GET`

## Minimal Exploit
- Access another user's file using a predictable ID or adjacent path.

## Secure Patterns
### NGINX X-Accel-Redirect (gateway authorized)
1. Backend authenticates & authorizes the request for a specific file.
2. If allowed, it responds with `X-Accel-Redirect: /protected/receipts/2025-000123.pdf`.
3. NGINX serves the file from an internal location that is otherwise unreachable.

_NGINX sketch_
```nginx
location /protected/ {
  internal;
  alias /var/app/private/;
}
```

### Pre-signed URLs (Object Storage)
- Generate short-lived, scoped URLs (method-limited, content-type restricted).
- Avoid long TTLs for sensitive data; prefer claim-bound server-side checks when feasible.

## Remediation
- Always authorize at the gateway before issuing file responses (even with X-Accel/X-Sendfile)
- Store files with non-enumerable keys and enforce ownership checks per request
- Consider watermarking/auditing on sensitive exports
