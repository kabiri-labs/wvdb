# WebHackFlow Web Penetration Testing Method (WebHackFlow‑WPM) v1.0

> A pragmatic, standards‑aligned methodology for web & API penetration tests. Designed for repeatability, evidence‑driven decisions, and easy repository integration.

## Scope & References
- **Scope:** Web applications, APIs (REST/GraphQL/gRPC), SPAs, supporting services (SSO/IdP, email, payment, queues), and web‑exposed infrastructure (reverse proxies/CDNs).
- **Primary references:** PTES, OWASP WSTG 4.2, OWASP ASVS 5.0, NIST SP 800‑115, OWASP Risk Rating (and CVSS when needed).

---

## 0) Pre‑Engagement & Governance
**Goals:** Define scope, rules, and success criteria; minimize legal/operational risk; set evidence, reporting, and retest expectations.

**Inputs:** SoW, RoE, asset list, test windows, escalation contacts, test accounts/roles.

**Activities:**
- Scope & in/out: apps, APIs, subdomains, third‑party integrations, SSO/IdP, mobile/web clients.
- Constraints: DoS, brute‑force rate limits, data handling (PII/PHI), prod vs staging, safe wordlists, synthetic data.
- Deliverables: report format, risk rating scheme, retest SLA, evidence storage path.
- Tooling plan: intercept proxy, scanners, fuzzers, headless browser, scripts.


---

## 1) Passive Discovery (OSINT)
**Goals:** Build context with zero touch to targets.

**Activities:** Search engines, cached copies, code & secret leaks, breach corpuses, CT logs, ASN/RDNS, third‑party docs, app store listings, social tech clues.

**Outputs:** OSINT notes; initial asset candidates, keywords, technology hints.

---

## 2) Active Discovery & Asset Inventory
**Goals:** Enumerate reachable assets/services and carve the attack surface.

**Activities:**
- **Network/host:** Port/service scans for web‑exposed services (HTTP(S), WS, gRPC, SSH for deploy, DB exposures, message queues).
- **Names:** Subdomain discovery, vhost/alt hostnames, environment prefixes (dev/test/stage/uat).
- **Endpoints:** Crawl/spider, sitemap/robots parsing, API specs (OpenAPI/GraphQL), JavaScript route mining, parameter discovery.


---

## 3) Fingerprinting & Technology Mapping
**Goals:** Identify platforms and moving parts to inform test depth.

**Activities:**
- App frameworks/CMS; language/runtime; reverse proxy/CDN/WAF; TLS posture; headers; third‑party SDKs; payment/email/queue/search; object storage (S3‑like), container/orchestrator hints.
- Version inference (safe); dependency manifests; build hashes.


---

## 4) Known‑Vulnerability Sweep
**Goals:** Quickly catch known issues matching identified stacks and versions.

**Activities:**
- Cross‑reference fingerprints against reputable advisories; targeted proof‑of‑concept checks where safe; avoid blind exploitation on prod.
- Use CVE/CVSS for technical severity; convert to business risk in reporting.


---

## 5) Content & Information Exposure Review
**Goals:** Uncover sensitive data through content, config, and metadata.

**Activities:**
- Web page content review; JS/WS/API traffic review; header/body leaks; error handling behavior.
- Unreferenced/backup files: `.bak`, `.old`, `~`, `.inc`, `.orig`, `.dev`, `.src`, `.log`, temp/export/db‑dump paths.


---

## 6) Authentication & Session Management
**Goals:** Validate identity, session, and token security across states.

**Activities:**
- Flows: register/login/MFA/reset/lockout/SSO.
- States: pre/post auth, logout/expiry/refresh, remember‑me, cross‑client (web ↔ mobile).
- Tokens/Cookies: predictability, entropy, scope, rotation, replay resistance, CSRF flags, `SameSite`, `HttpOnly`, `Secure`.
- Role sets: anon → user → privileged; impersonation; session fixation; credential stuffing defenses.


---

## 7) Authorization (Access Control)
**Goals:** Enforce vertical/horizontal boundaries at object & function levels.

**Activities:**
- **IDOR/BOLA/BFLA:** enumerate IDs, indirect references, filters; test mass assignment and forced browsing.
- **Multi‑tenant:** tenant isolation, scoping in APIs, search and export leakage.
- **Administrative paths:** privilege boundaries, support tools, feature flags.


---

## 8) Input Handling & Injection
**Goals:** Find injection classes and context escapes.

**Activities:**
- XSS (reflected/stored/DOM), template/server‑side injection, SQL/NoSQL/ORM, OS command, SSRF, XXE, deserialization, path traversal, prototype pollution.
- Context‑aware payloads; sink identification; out‑of‑band channels; CSP/Trusted Types review for modern SPAs.


---

## 9) Adaptive Fuzzing (Coverage‑Aware)
**Goals:** Move beyond static wordlists; target code regions likely to fault.

**Activities:**
- Feedback‑driven fuzzing on safely scoped endpoints (staging preferred); mutation strategies; corpus curation; crash triage.
- Directed fuzzing (e.g., AFLGo‑style) for patch verification and hot paths.


**Safety:** Respect rate limits, data constraints; gate on non‑destructive endpoints; coordinate with DevOps.

---

## 10) Business Logic & Workflow Abuse
**Goals:** Break assumptions and violate invariants that scanners miss.

**Activities:**
- Data integrity; multi‑step workflows; transaction ordering; concurrency/race conditions; file upload/download controls; email flows (reset, notifications); temporal issues; third‑party integrations.


---

## 11) Exploitation & Impact Demonstration
**Goals:** Safely demonstrate exploitability and realistic impact.

**Activities:**
- Minimal‑impact PoCs; constrained data access; proof of exfiltration on test data; abuse‑of‑function scenarios; chained exploits.


---

## 12) Reporting, Risk Rating, Remediation & Retest
**Goals:** Decision‑ready report; prioritized fixes; verification loop.

**Activities:**
- Normalize findings; risk rating (OWASP Risk Rating; CVSS for CVE‑tied items); business impact mapping; actionable remediation with code/config examples.
- Deliverables: executive summary, technical appendix, evidence pack; retest plan & results; regression advice.


---

## Cross‑Cutting Practices
- **Threat modeling as a lens:** keep STRIDE/kill‑chain thinking active across phases.
- **Privacy & data minimization:** scrub PII; use synthetic datasets; redact tokens in screenshots.
- **Traceability:** Every finding links: evidence → request/response → affected asset → remediation.
- **Coverage tracking:** List modules, roles, workflows, API methods tested and those deferred.


*End of v1.0*
