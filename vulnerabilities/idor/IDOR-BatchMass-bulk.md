# IDOR – Bulk/Batch Operations

## Summary
Batch endpoints (`ids=[...]`, CSV uploads, multi-select) process non-owned records due to missing per-item checks.

## Discovery
- Provide mixed arrays: owned + unowned IDs; observe which are processed

## Minimal Exploit
- Submit `{"ids":[1,2,3]}` where `2` and `3` are foreign; verify partial/complete success

## Remediation
- Enforce per-item authorization with atomic rejection or precise failure reporting
- Log and block when any unowned ID is present
