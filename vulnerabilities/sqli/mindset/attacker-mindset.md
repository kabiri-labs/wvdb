# Attacker Mindset — SQL Injection (SQLi)

> This file captures the **attacker’s mental model** when probing for SQL Injection.
> Goal: help defenders and testers anticipate attacker **decision-making, priorities, and creativity**.

---

## 1) Core Attitude
- Treat every **input** as a potential **query fragment**.
- Assume developers are in a hurry; shortcuts (string concatenation, ORM `.raw()`, dynamic ORDER BY) are **everywhere**.
- Look for **patterns, errors, and timing**. Even one shaky signal can open the door.

---

## 2) Recon Phase
- **Fingerprint the stack** indirectly:
  - Error messages, response times, HTML templates, default pages.
  - Parameters in URLs that *smell* like primary keys, filters, search fields, sort columns.
- **Check context**:
  - Numeric vs string → payload shapes differ.
  - GET vs POST vs JSON vs headers (hidden channels often overlooked).
- **Baseline control**: send `q9xZ` marker, toggle between `1=1` / `1=2`.

---

## 3) Exploitation Decision Tree
1. **Direct output?**  
   → Try `UNION SELECT`, aim for version/user/database.
2. **Error banners visible?**  
   → Try type casts / invalid functions → harvest error text.
3. **No output, no errors?**  
   → Try **boolean flips** (`1=1` vs `1=2`).
4. **Still blind?**  
   → Try **time-based** (`SLEEP`, `WAITFOR`, `pg_sleep`, `dbms_lock.sleep`).
5. **All blocked?**  
   → Test **OOB** vectors (DNS/HTTP/SMB/LDAP callbacks).
6. **Context special?**  
   → Stored templates / batch jobs → **second-order SQLi**.

---

## 4) Lateral Thinking
- Don’t need full dump — a **single proof** (boolean/time) is enough to demand a fix.
- Pivot through **different clauses**: `WHERE`, `ORDER BY`, `LIMIT`, `JOIN`, `HAVING`.
- When column count fails: use **NULL padding** or type normalization (`CAST(... AS CHAR/NVARCHAR/text)`).
- If UNION blocked: try **stacked queries** or **procedural blocks**.
- Always test **multi-param flows** (two fields combined into one query).

---

## 5) Patience & Stealth
- Attackers use **short sleeps** (2–3s) and few iterations to avoid detection.
- Prefer **markers** (`q9xZ`) over obvious strings (`test`, `sql`).
- Blend in with normal traffic; replay legitimate requests with tiny mutations.
- Use **error thresholds** (e.g., 3 consecutive hits) before deciding injection is real.

---

## 6) Escalation Goals
- First → Prove injection exists.  
- Next → Confirm engine/version.  
- Then → Check what’s possible (read vs write vs OOB).  
- Only if authorized → escalate to **data extraction** or **privilege abuse**.

---

## 7) Defensive Takeaways
- If you know how attackers think:
  - **Generic error banners** remove early wins.
  - **Prepared statements** break most paths at root.
  - **Identifier allowlists** close ORDER BY / LIMIT tricks.
  - **Least privilege** stops file/OOB escalation.
  - **Monitoring** (time spikes, error clusters, DNS callbacks) can reveal probes early.

---

## 8) Minimal Checklist (attacker lens)
- [ ] Identified context & baseline marker
- [ ] Boolean flip works? (YES → injection confirmed)
- [ ] Time probe works? (YES → blind path open)
- [ ] Error shows attacker data? (YES → error-based path)
- [ ] UNION alignment possible? (YES → in-band extraction)
- [ ] OOB triggered? (YES → high-risk, cross-boundary)
- [ ] Second-order sink? (YES → delayed exploitation possible)
