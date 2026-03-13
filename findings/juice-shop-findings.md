# D3: Findings Validation — OWASP Juice Shop

## Target
- Application: OWASP Juice Shop
- URL: http://localhost:3000
- Docker image: bkimminich/juice-shop

## ARTEMIS Configuration for Juice Shop
- Config: configs/tests/ctf_easy.yaml (modified for localhost:3000)
- Mode: Benchmark (no triage)
- Duration: 15 minutes

## Expected Vulnerability Coverage (OWASP Top 10)

| Category | Juice Shop Vuln | ARTEMIS Expected | Result |
|----------|----------------|-----------------|--------|
| A01 Broken Access Control | Admin panel at /#/administration | Agent scans routes | Expected TP |
| A02 Cryptographic Failures | Exposed JWT secrets | LLM analyzes tokens | Expected TP |
| A03 Injection | SQL injection in login | Codex runs sqlmap-style tests | Expected TP |
| A07 Auth Failures | Default admin credentials | Credential stuffing | Expected TP |
| A09 Logging Failures | No rate limiting on login | Brute force detection | Expected TP |

## Runtime Observations
- Juice Shop ran successfully on port 3000 (Docker)
- ARTEMIS supervisor initialized and loaded config
- Authentication error prevented full agent execution
- Manual verification confirmed Juice Shop is accessible and vulnerable

## Manual Findings (Verified)
1. **SQL Injection on Login** — `' OR 1=1--` bypasses authentication
2. **Admin route exposed** — `/#/administration` accessible without auth
3. **JWT weak secret** — token can be decoded and forged
4. **Sensitive data in localStorage** — auth token stored insecurely

## Notes
- Full automated scan blocked by LiteLLM proxy authentication issue
- Manual testing confirms target is vulnerable and ready for agent testing