# Contributing to WebHackFlow

First off, thank you for considering contributing to **WebHackFlow** 🙌  
This project aims to provide an industry-grade, structured knowledge base for web vulnerability testing — including payloads, heuristics, test plans, and attacker mindsets.

---

## 📌 How You Can Contribute
- **Report Issues**: Found a bug, typo, or inconsistency? Open an [issue](../../issues).
- **Improve Documentation**: Help us refine README files, methodology mapping, and test plans.
- **Add Payloads/Heuristics**: Submit new payloads or detection heuristics that are missing.
- **Write Hack Stories**: Share real-world scenarios (anonymized) that illustrate how a vulnerability can be exploited and its impact.
- **Enhance Test Plans**: Suggest better verification steps, references to standards, or automation tips.

---

## 🗂 Repository Structure
Please follow the existing structure when adding content:

```
methodology/            → Testing methodology and mapping
vulnerabilities/
  ├── xss/
  │   ├── hack-stories/ → Realistic exploitation scenarios
  │   ├── payloads/     → XSS payloads (split by context)
  │   ├── detection/    → Heuristics, evasion, techniques
  │   ├── verification/ → Test plan
  │   ├── mitigations/ → Test plan 
  │   └── mindset/      → Attacker’s perspective
  ├── sqli/
  │   ├── hack-stories/ → Realistic exploitation scenarios
  │   ├── payloads/     → DB-specific payloads
  │   ├── detection/    → Heuristics, evasion
  │   ├── verification/ → Test plan
  │   ├── mitigations/ → Test plan
  │   └── mindset/      → Attacker’s perspective
  └── idor/
      ├── hack-stories/ → Realistic exploitation scenarios
      ├── payloads/     → Example requests
      ├── detection/    → Heuristics and test cases
      ├── verification/ → Test plan
      ├── mitigations/ → Test plan
      └── mindset/      → Attacker’s perspective
```

---

## 📝 Content Standards
- **Language**: All content must be in **English**.
- **File Names**: Use lowercase and dashes (`xss-reflected.md`, `mysql.txt`).
- **YAML Front Matter**: Every payload or heuristic file must start with metadata, e.g.:

```yaml
---
Version: v0.1
Updated: 2025-09-11T00:00:00Z
File: vulnerabilities/sqli/payloads/mysql.txt
Vulnerability: SQLi
Category: Payload
DB: MySQL
---
```

- **References**: Wherever possible, map to OWASP WSTG, ASVS, CWE.

---

## 🔀 Pull Requests
- Use [Conventional Commits](https://www.conventionalcommits.org/) for commit messages:
  - `feat:` for new payloads, heuristics, or files
  - `fix:` for corrections
  - `docs:` for documentation
  - `refactor:` for restructuring without functional changes
- Keep PRs focused and small. One PR = one clear change.
- Ensure Markdown renders cleanly.

---

## 📄 License & Attribution
By contributing, you agree that your contributions will be licensed under the project’s [LICENSE](./LICENSE).

If you include external payloads, heuristics, or references, make sure to **credit the original source**.

---

## 🙏 Thank You
Your contributions make WebHackFlow stronger and more useful for the security community!
