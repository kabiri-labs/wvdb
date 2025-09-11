# IDOR – Relationship/Child Object

## Summary
Child references (e.g., `payment.invoice_id`) allow bypassing parent ownership checks by manipulating the child or link.

## Discovery
- Identify foreign keys in bodies: `... "invoice_id": 1024 ...`
- Try linking a child to a victim's parent resource

## Minimal Exploit
- POST a child with `parent_id` of another user's object; verify side effects (e.g., charges, comments, attachments on victim's record).

## Remediation
- Validate parent ownership from server-side context before linking child records
- Use server-calculated parent IDs (not client-provided) for state-changing operations
