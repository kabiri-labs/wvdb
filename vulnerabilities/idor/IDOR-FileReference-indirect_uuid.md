# IDOR – Indirect References (UUID/Slug/Token)

## Summary
Opaque GUIDs/slugs are treated as security; server still skips ownership checks.

## Discovery
- Swap known GUIDs between two accounts; check read/write behavior
- Attempt timing/brute on poorly generated tokens

## Remediation
- Treat GUID/slug as identifiers only; always enforce server-side authorization
