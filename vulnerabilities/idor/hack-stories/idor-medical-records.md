# Hack Story: Exposed Medical Records via Sequential Patient ID (IDOR)

## Context
A healthcare web application allowed patients to view their medical records by visiting:
```
https://example.com/patient/records?id=12345
```

## Exploit
The application only checked if the user was logged in, but did not validate ownership of the patient record. 
By changing the `id` parameter from `12345` to `12346`, an attacker could view another patient’s records.

## Impact
- Exposure of sensitive PII (name, date of birth, SSN)
- Disclosure of medical history (HIPAA violation)
- Potential legal and financial consequences for the organization

## Lessons
- Always enforce object-level authorization
- Never rely on obscurity or sequential IDs
