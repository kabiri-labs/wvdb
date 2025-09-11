# IDOR (Broken Object-Level Authorization)

**Scope:** Practical, tester-first guidance for finding and exploiting authorization bypass via user-controlled identifiers across REST, file handlers, batch operations, and GraphQL. This module aligns with our standard flow: Problem → Impact → Detection → Minimal Exploit → Verification → Remediation.

**Sub-groups**
- `object-level/`: Classic BOLA patterns (basic, tenant-bound, child relationships)
- `file-reference/`: Direct/indirect resource references (file paths, keys, slugs)
- `batch-mass/`: Bulk operations & mass assignment leading to ownership hijack
- `functional/`: Action-on-others (BFLA) that behaves IDOR-like
- `alt-protocol/`: GraphQL (and similar) nuances
- `detection/`: Heuristics & portable test cases
- `payloads/`: Example HTTP requests
- `checklists/`: Attacker mindset & remediation checklists
- `verification/`: Test plan templates

> **CWE:** CWE-639 (Authorization Bypass Through User-Controlled Key), CWE-862 (Missing Authorization)

**Minimal threat model**
- *Asset:* Object owned by a user/tenant (records, files, child resources)
- *Entry:* User-controlled identifiers (path, query, body, headers, GraphQL IDs)
- *Control*: Server-side ownership & authorization checks
- *Failure:* Missing or weak checks permit cross-object access or actions


# References (curate as you go)

- CWE-639, CWE-862
- OWASP API Top 10 – Broken Object Level Authorization (BOLA)
- OWASP ASVS – Access Control
- OWASP Testing Guide – Authorization testing
