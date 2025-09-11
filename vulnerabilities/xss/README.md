---
title: XSS — Group Overview
last_updated: 2025-09-10
---

# XSS — Group Overview

This folder contains **four XSS stories** (Reflected, Stored, DOM-based, Blind) written in **plain English for non-native readers**.
Each story is practical: probes per rendering context, a minimal exploit that proves real impact, precise fixes, verification steps, and a comprehensive **attack chain**.

## How to read a story
1) Start with **Scene** to recognize the bug in the wild.  
2) Check **Attack Surface / Context** to list entry points and render contexts.  
3) Use **Hypotheses & Probes** to confirm safely.  
4) Show impact with **Minimal Exploit** (in a sandbox).  
5) Apply **Fix & Hardening** and then **Verification**.  
6) Study **Attack Chain** to anticipate follow-on attacks (**including BOLA/IDOR and second-order flows**).

## Quick triage checklist (copy/paste)
- Where does the value render? **Exact context** (HTML text/attr/JS/URL)?
- Are we using **encode-on-output** at the render point?
- Do any **dangerous sinks** exist (`innerHTML`, inline events, `eval`)?
- Is there a **CSP** blocking inline/script injection?
- If stored/rendered elsewhere (email/report/admin), plan a **blind/OOB** check.

## References
OWASP Cheat Sheet Series (XSS prevention, DOM XSS) · OWASP WSTG · OWASP ASVS v5 · CWE-79 · PortSwigger XSS Cheat Sheet
