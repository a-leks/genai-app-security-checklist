# AI-Coded App Vulnerability Checklist

Comprehensive reference for static and runtime vulnerability scanning of applications built with AI assistance (Claude/Anthropic, GPT/OpenAI, Gemini/Google, Grok/xAI, Mistral, Copilot/GitHub, and equivalents). Organized by category. Each item includes detection method and severity.

---

## Legend

- **[S]** - Static analysis (codebase scan)
- **[R]** - Runtime check (against local/staging server)
- **[C]** - Config/infrastructure check
- 🔴 Critical | 🟠 High | 🟡 Medium | 🟢 Low

---

## 1. Authentication & Session Management

| # | Vulnerability | Detection | Severity |
|---|---------------|-----------|----------|
| 1.1 | Auth tokens stored in `localStorage` | [S] grep/AST | 🔴 |
| 1.2 | Auth tokens stored in `sessionStorage` | [S] grep/AST | 🟠 |
| 1.3 | Sessions that never expire (no `maxAge` or TTL) | [S][R] | 🔴 |
| 1.4 | Stolen token = permanent access (no token rotation) | [S] | 🔴 |
| 1.5 | JWT `alg: none` accepted | [R] | 🔴 |
| 1.6 | JWT secret is weak or hardcoded default (`secret`, `changeme`) | [S] | 🔴 |
| 1.7 | JWT not verified on protected routes | [S] AST | 🔴 |
| 1.8 | Refresh tokens stored insecurely or never rotated | [S] | 🟠 |
| 1.9 | No brute-force protection on login endpoint | [R] | 🔴 |
| 1.10 | No account lockout after N failed attempts | [R] | 🟠 |
| 1.11 | Password stored in plaintext or with MD5/SHA1 | [S] | 🔴 |
| 1.12 | Bcrypt/argon2 work factor too low (< 10 rounds) | [S] | 🟠 |
| 1.13 | No CSRF protection on state-changing endpoints | [S][R] | 🔴 |
| 1.14 | `SameSite` cookie attribute missing | [R] headers | 🟠 |
| 1.15 | `HttpOnly` cookie flag missing | [R] headers | 🟠 |
| 1.16 | `Secure` cookie flag missing (cookies sent over HTTP) | [R] headers | 🟠 |
| 1.17 | Magic link / OTP codes that never expire | [S] | 🔴 |
| 1.18 | Magic link / OTP codes that are reusable | [S] | 🟠 |
| 1.19 | Auth bypass via HTTP method override (`X-HTTP-Method-Override`) | [R] | 🟠 |
| 1.20 | Concurrent session limit not enforced | [S] | 🟢 |
| 1.21 | Logout does not invalidate server-side session | [R] | 🔴 |

---

## 2. Authorization & Access Control

| # | Vulnerability | Detection | Severity |
|---|---------------|-----------|----------|
| 2.1 | Admin routes with no role checks | [S][R] | 🔴 |
| 2.2 | IDOR - object IDs predictable and not ownership-checked | [R] | 🔴 |
| 2.3 | Horizontal privilege escalation (user A accesses user B's data) | [R] | 🔴 |
| 2.4 | Vertical privilege escalation (user promotes themselves to admin) | [R] | 🔴 |
| 2.5 | Missing authorization on bulk operations / batch endpoints | [S][R] | 🔴 |
| 2.6 | Role checks only in frontend, not backend | [S] | 🔴 |
| 2.7 | API endpoint enumeration reveals undocumented admin routes | [R] | 🟠 |
| 2.8 | Misconfigured multi-tenant isolation (tenant A reads tenant B's data) | [R] | 🔴 |
| 2.9 | GraphQL introspection enabled in production | [R] | 🟠 |
| 2.10 | GraphQL depth/complexity limits absent (DoS via nested queries) | [R] | 🟠 |
| 2.11 | Feature flags enforced only client-side | [S] | 🟠 |
| 2.12 | Mass assignment - ORM allows patching any model field via API | [S] | 🔴 |

---

## 3. Input Validation & Injection

| # | Vulnerability | Detection | Severity |
|---|---------------|-----------|----------|
| 3.1 | SQL injection - raw query string concatenation | [S] AST | 🔴 |
| 3.2 | NoSQL injection (MongoDB `$where`, `$gt` operator abuse) | [S] | 🔴 |
| 3.3 | Command injection - `exec()`, `spawn()` with unsanitized input | [S] AST | 🔴 |
| 3.4 | LDAP injection | [S] | 🔴 |
| 3.5 | XPath injection | [S] | 🟠 |
| 3.6 | Template injection (SSTI) - user input in server-side templates | [S] | 🔴 |
| 3.7 | Reflected XSS - unsanitized user input rendered in HTML | [S][R] | 🔴 |
| 3.8 | Stored XSS - persisted user content rendered without escaping | [S][R] | 🔴 |
| 3.9 | DOM XSS - `innerHTML`, `dangerouslySetInnerHTML` with user data | [S] AST | 🔴 |
| 3.10 | Open redirect - unvalidated `redirect` / `next` URL params | [S][R] | 🟠 |
| 3.11 | Path traversal - `../` in file paths from user input | [S] | 🔴 |
| 3.12 | XML external entity (XXE) injection | [S] | 🔴 |
| 3.13 | CSV injection - formula injection in exported spreadsheets | [S] | 🟠 |
| 3.14 | Regex DoS (ReDoS) - catastrophic backtracking in user-controlled regex | [S] | 🟠 |
| 3.15 | Missing file upload type validation (MIME + extension) | [S] | 🔴 |
| 3.16 | Missing file upload size limit | [S] | 🟠 |
| 3.17 | Uploaded files executed as code (no storage isolation) | [C] | 🔴 |
| 3.18 | JSON prototype pollution | [S] | 🟠 |
| 3.19 | HTTP parameter pollution | [R] | 🟡 |
| 3.20 | Integer overflow on user-supplied numeric fields | [S] | 🟠 |

---

## 4. API Security

| # | Vulnerability | Detection | Severity |
|---|---------------|-----------|----------|
| 4.1 | No rate limiting on any API routes | [R] | 🔴 |
| 4.2 | No rate limiting specifically on auth endpoints | [R] | 🔴 |
| 4.3 | No rate limiting on password reset / OTP endpoints | [R] | 🔴 |
| 4.4 | No CORS policy configured | [R] headers | 🔴 |
| 4.5 | CORS wildcard (`*`) on authenticated routes | [R] headers | 🔴 |
| 4.6 | CORS origin reflected without validation | [R] | 🔴 |
| 4.7 | API versioning absent - breaking changes with no migration path | [S] | 🟢 |
| 4.8 | Verbose error messages leak stack traces / DB schema | [R] | 🟠 |
| 4.9 | API keys accepted in URL query params (logged in server logs) | [S] | 🟠 |
| 4.10 | No request body size limit (memory exhaustion DoS) | [S][R] | 🟠 |
| 4.11 | Missing `Content-Type` validation on POST/PUT endpoints | [R] | 🟡 |
| 4.12 | HTTP verb confusion - PUT/DELETE not restricted | [R] | 🟠 |
| 4.13 | REST endpoints return full object when only subset needed (data over-exposure) | [S] | 🟠 |
| 4.14 | Webhook endpoints with no signature verification | [S] | 🔴 |
| 4.15 | Stripe webhook `event` trusted without `stripe.webhooks.constructEvent` | [S] grep | 🔴 |
| 4.16 | Server-Side Request Forgery (SSRF) - user-supplied URLs fetched server-side | [S] | 🔴 |
| 4.17 | No pagination on list endpoints - full table scanned per request | [S] | 🟠 |
| 4.18 | Pagination `limit` parameter unbounded (user sets limit=999999) | [S][R] | 🟠 |

---

## 5. Secrets & Configuration

| # | Vulnerability | Detection | Severity |
|---|---------------|-----------|----------|
| 5.1 | Hardcoded API keys in frontend source (OpenAI, Anthropic, Google, Stripe, etc.) | [S] entropy scan | 🔴 |
| 5.2 | Hardcoded API keys in backend source | [S] entropy scan | 🔴 |
| 5.3 | `.env` file committed to git history | [S] git log scan | 🔴 |
| 5.4 | `.env` file publicly accessible via web server | [R] | 🔴 |
| 5.5 | Secrets in `package.json`, `config.json`, or other committed config files | [S] | 🔴 |
| 5.6 | No environment variable validation at startup | [S] | 🟠 |
| 5.7 | App silently starts with missing/malformed env vars | [S] | 🟠 |
| 5.8 | Production secrets same as development secrets | [C] | 🟠 |
| 5.9 | Cloud provider credentials (AWS, GCP, Azure) in source | [S] | 🔴 |
| 5.10 | Database credentials in source or public config | [S] | 🔴 |
| 5.11 | Private keys / certificates committed to repo | [S] file pattern | 🔴 |
| 5.12 | Source maps exposed in production (reveals full source + potential secrets) | [R] | 🟠 |
| 5.13 | Debug mode / verbose logging enabled in production | [C][R] | 🟠 |
| 5.14 | `NODE_ENV` not set to `production` | [C] | 🟡 |
| 5.15 | `.gitignore` missing entries for `.env`, `*.pem`, `*.key` | [S] | 🟠 |

---

## 6. AI-Specific Vulnerabilities (LLM Integration)

| # | Vulnerability | Detection | Severity |
|---|---------------|-----------|----------|
| 6.1 | Prompt injection - user input passed directly into system prompt without sanitization | [S] AST | 🔴 |
| 6.2 | Indirect prompt injection - AI reads attacker-controlled content (email, web page, file) | [S] | 🔴 |
| 6.3 | AI output rendered as raw HTML / executed as code without validation | [S] | 🔴 |
| 6.4 | LLM used to make security decisions (auth, RBAC) without deterministic fallback | [S] | 🔴 |
| 6.5 | System prompt leaked via prompt injection (reveal instructions) | [R] | 🟠 |
| 6.6 | No output filtering on LLM responses before rendering to user | [S] | 🟠 |
| 6.7 | AI API key exposed in frontend (OpenAI, Anthropic, Google AI, Grok) | [S] entropy | 🔴 |
| 6.8 | No spend limits or usage caps on AI API keys | [C] | 🔴 |
| 6.9 | User-controlled `model` parameter - user selects expensive model to run up bill | [S][R] | 🟠 |
| 6.10 | User-controlled `max_tokens` or `temperature` with no ceiling | [S][R] | 🟠 |
| 6.11 | AI responses cached without tenant isolation (user A sees user B's cached AI output) | [S] | 🔴 |
| 6.12 | Sensitive user data sent to AI API without data processing agreement review | [C] | 🟠 |
| 6.13 | PII sent to third-party AI API in violation of privacy policy | [S] | 🟠 |
| 6.14 | No token budget enforcement - single request burns unbounded tokens | [S] | 🟠 |
| 6.15 | Streaming AI responses not rate-limited or connection-capped | [R] | 🟠 |
| 6.16 | AI tool/function calls executed without confirmation or scope restriction | [S] | 🔴 |
| 6.17 | Agentic AI given filesystem, shell, or DB access without sandboxing | [S] | 🔴 |
| 6.18 | LLM hallucination of code executed at runtime (eval of AI output) | [S] | 🔴 |
| 6.19 | No fallback when AI API is down - entire app fails | [S] | 🟠 |
| 6.20 | Vector database embeddings not scoped per tenant | [S] | 🔴 |
| 6.21 | RAG retrieval returns documents from other tenants' namespaces | [S][R] | 🔴 |
| 6.22 | Embedding model leaks PII through vector similarity search | [S] | 🟠 |
| 6.23 | AI-generated code shipped without type checking (no TypeScript, no mypy) | [S] | 🟠 |
| 6.24 | Fine-tuned model trained on user data without consent | [C] | 🟠 |
| 6.25 | Model version not pinned - silent behavior change on provider update | [S] | 🟡 |
| 6.26 | Tool poisoning via MCP server - attacker-controlled MCP server injects instructions into tool results read by the agent | [S][C] | 🔴 |
| 6.27 | Agent long-term memory poisoning - malicious content stored in agent memory is retrieved and executed in future sessions | [S] | 🔴 |
| 6.28 | Encoding-based jailbreak - user submits base64, Unicode homoglyphs, or obfuscated text that bypasses content filters | [R] | 🟠 |
| 6.29 | Model extraction via inference probing - user systematically queries the model to reconstruct the system prompt or training data | [R] | 🟠 |
| 6.30 | Cross-agent prompt injection - one agent's output containing an attacker payload is passed as input to a second agent without sanitization | [S] | 🔴 |
| 6.31 | Insecure agent handoff - parent agent passes API keys, session tokens, or credentials to sub-agents without reducing scope | [S] | 🔴 |
| 6.32 | LLM output parsed as structured data without schema validation before database write | [S] | 🟠 |
| 6.33 | Conversation history stored without per-user isolation - authenticated users can retrieve other users' chat history | [S][R] | 🟠 |

---

## 7. Database & Data Layer

| # | Vulnerability | Detection | Severity |
|---|---------------|-----------|----------|
| 7.1 | No database indexing on queried fields | [S] ORM scan | 🟠 |
| 7.2 | No database connection pooling | [S] | 🟠 |
| 7.3 | No backup strategy | [C] | 🔴 |
| 7.4 | Backups not tested / never restored | [C] | 🟠 |
| 7.5 | Backups stored in same region as primary DB | [C] | 🟠 |
| 7.6 | No database migration rollback strategy | [S] | 🟠 |
| 7.7 | ORM `syncForce: true` or `dropSchema: true` in production config | [S] | 🔴 |
| 7.8 | Raw SQL with string concatenation instead of parameterized queries | [S] AST | 🔴 |
| 7.9 | Database user has superuser privileges in production | [C] | 🟠 |
| 7.10 | Database port exposed publicly (no firewall rule) | [C][R] | 🔴 |
| 7.11 | No query timeout configured | [S] | 🟠 |
| 7.12 | N+1 query problem - ORM fetching relations in loops | [S] AST | 🟡 |
| 7.13 | Soft deletes not implemented - hard delete exposes IDOR window | [S] | 🟡 |
| 7.14 | PII stored unencrypted at rest | [C] | 🟠 |
| 7.15 | Encryption keys stored alongside encrypted data | [C] | 🔴 |
| 7.16 | No data retention / purge policy | [C] | 🟡 |
| 7.17 | Full table scans on large tables with no pagination | [S] | 🟠 |
| 7.18 | Transactions not used for multi-step writes (partial failure = corrupt state) | [S] | 🟠 |

---

## 8. Infrastructure & Deployment

| # | Vulnerability | Detection | Severity |
|---|---------------|-----------|----------|
| 8.1 | No health check endpoint | [R] | 🟠 |
| 8.2 | No logging in production | [C] | 🟠 |
| 8.3 | Logs contain plaintext passwords or tokens | [S] | 🔴 |
| 8.4 | Logs not centralized - no access to logs after pod/container restart | [C] | 🟠 |
| 8.5 | No alerting on error rate spikes | [C] | 🟠 |
| 8.6 | No uptime monitoring | [C] | 🟠 |
| 8.7 | Images uploaded directly to application server (no CDN) | [S] | 🟡 |
| 8.8 | Static assets served from app server (not CDN) | [C] | 🟡 |
| 8.9 | No HTTPS enforced - HTTP traffic not redirected | [R] | 🔴 |
| 8.10 | TLS version < 1.2 accepted | [R] | 🔴 |
| 8.11 | `HSTS` header missing | [R] headers | 🟠 |
| 8.12 | `X-Content-Type-Options: nosniff` missing | [R] headers | 🟠 |
| 8.13 | `X-Frame-Options` missing (clickjacking) | [R] headers | 🟠 |
| 8.14 | `Content-Security-Policy` absent | [R] headers | 🟠 |
| 8.15 | `Referrer-Policy` not set | [R] headers | 🟢 |
| 8.16 | `Permissions-Policy` not set | [R] headers | 🟢 |
| 8.17 | Server version disclosed in response headers (`X-Powered-By`, `Server`) | [R] headers | 🟡 |
| 8.18 | Default server error pages expose framework/version | [R] | 🟡 |
| 8.19 | No WAF in front of production | [C] | 🟠 |
| 8.20 | SSH root login enabled on production server | [C] | 🔴 |
| 8.21 | SSH password authentication enabled (no key-only) | [C] | 🟠 |
| 8.22 | All ports open on server - no firewall ruleset | [C] | 🔴 |
| 8.23 | No DDoS protection | [C] | 🟠 |
| 8.24 | Docker container running as root | [C] | 🟠 |
| 8.25 | Docker image using `latest` tag (non-deterministic builds) | [S] Dockerfile | 🟡 |
| 8.26 | Secrets passed as Docker env vars visible in `docker inspect` | [C] | 🟠 |
| 8.27 | No process manager / auto-restart on crash (PM2, systemd) | [C] | 🟠 |
| 8.28 | No memory/CPU limits on containers or processes | [C] | 🟠 |

---

## 9. Frontend & Client-Side

| # | Vulnerability | Detection | Severity |
|---|---------------|-----------|----------|
| 9.1 | No error boundaries in React UI | [S] AST | 🟠 |
| 9.2 | `dangerouslySetInnerHTML` used with user data | [S] AST | 🔴 |
| 9.3 | `eval()` used anywhere in codebase | [S] grep | 🔴 |
| 9.4 | `document.write()` used | [S] grep | 🟠 |
| 9.5 | Sensitive data in browser URL params (tokens, IDs) | [S] | 🟠 |
| 9.6 | Sensitive data in `window` global scope | [S] | 🟠 |
| 9.7 | Dependency with known CVE in `package.json` | [S] `npm audit` | 🔴 |
| 9.8 | Outdated dependencies not audited | [S] | 🟠 |
| 9.9 | No Subresource Integrity (SRI) on external scripts | [S] | 🟠 |
| 9.10 | Third-party scripts loaded without `async`/`defer` (performance + XSS risk) | [S] | 🟡 |
| 9.11 | Client-side feature flag / role state stored in `localStorage` or cookie without server validation | [S] | 🔴 |
| 9.12 | Business logic enforced only client-side | [S] | 🔴 |
| 9.13 | No TypeScript - untyped AI-generated code shipped to production | [S] | 🟠 |
| 9.14 | `console.log` with sensitive data left in production build | [S] grep | 🟠 |
| 9.15 | Source maps exposed publicly in production | [R] | 🟠 |
| 9.16 | No Content Security Policy - inline scripts allowed | [R] headers | 🟠 |
| 9.17 | Clickjacking - no frame-ancestors policy | [R] headers | 🟠 |

---

## 10. Email, Payments & Third-Party Integrations

| # | Vulnerability | Detection | Severity |
|---|---------------|-----------|----------|
| 10.1 | Emails sent synchronously in request handlers | [S] AST | 🟠 |
| 10.2 | No email queue / retry mechanism | [S] | 🟡 |
| 10.3 | Password reset links that never expire | [S] | 🔴 |
| 10.4 | Password reset links that are reusable | [S] | 🔴 |
| 10.5 | Password reset link not invalidated after use | [S] | 🔴 |
| 10.6 | Account enumeration via password reset (different error for existing vs non-existing email) | [R] | 🟠 |
| 10.7 | Stripe webhook not verified with signature | [S] grep | 🔴 |
| 10.8 | Payment amount trusted from client-side request | [S] | 🔴 |
| 10.9 | No idempotency on payment processing (double-charge on retry) | [S] | 🔴 |
| 10.10 | Subscription status trusted from client-side state | [S] | 🔴 |
| 10.11 | OAuth `state` parameter not validated (CSRF on OAuth flow) | [S] | 🔴 |
| 10.12 | OAuth `redirect_uri` not strictly validated | [S] | 🔴 |
| 10.13 | Third-party SDK loaded over HTTP (not HTTPS) | [S] | 🔴 |
| 10.14 | Webhook retries not deduplicated (event processed multiple times) | [S] | 🟠 |
| 10.15 | No timeout on outbound HTTP requests to third parties | [S] | 🟠 |

---

## 11. Password Reset & Account Recovery

| # | Vulnerability | Detection | Severity |
|---|---------------|-----------|----------|
| 11.1 | Password reset tokens not cryptographically random | [S] | 🔴 |
| 11.2 | Password reset token stored in plaintext in DB | [S] | 🔴 |
| 11.3 | Password reset token in URL logged in server logs | [S] | 🟠 |
| 11.4 | Security questions used as sole recovery method | [S] | 🟠 |
| 11.5 | No rate limit on password reset requests | [R] | 🟠 |
| 11.6 | New password not checked against previous passwords | [S] | 🟢 |
| 11.7 | No minimum password strength requirement | [S] | 🟠 |
| 11.8 | No check against breached password databases (HaveIBeenPwned API) | [S] | 🟡 |

---

## 12. Logging, Monitoring & Incident Response

| # | Vulnerability | Detection | Severity |
|---|---------------|-----------|----------|
| 12.1 | No structured logging (no JSON logs, no log levels) | [S] | 🟠 |
| 12.2 | Logs not persisted outside of application process | [C] | 🟠 |
| 12.3 | Auth events not logged (login, logout, failed attempts) | [S] | 🟠 |
| 12.4 | Admin actions not audited | [S] | 🟠 |
| 12.5 | No anomaly detection or alerting | [C] | 🟠 |
| 12.6 | No error tracking (Sentry, Rollbar, or equivalent) | [C] | 🟠 |
| 12.7 | Error tracking SDK leaking PII in payloads | [S] | 🟠 |
| 12.8 | No on-call rotation or incident response plan | [C] | 🟡 |
| 12.9 | Production logs accessible without authentication | [C] | 🔴 |
| 12.10 | Log injection - user input written directly into log lines | [S] | 🟡 |

---

## 13. Performance Vulnerabilities (that become security issues at scale)

| # | Vulnerability | Detection | Severity |
|---|---------------|-----------|----------|
| 13.1 | No database indexing on high-cardinality queried fields | [S] | 🟠 |
| 13.2 | Single request loads entire database table into memory | [S] | 🟠 |
| 13.3 | No caching layer - every request hits DB | [S] | 🟡 |
| 13.4 | Cache not invalidated on data mutation | [S] | 🟠 |
| 13.5 | File processing (image resize, PDF parse) done synchronously in request handler | [S] | 🟠 |
| 13.6 | No job queue for background tasks | [S] | 🟡 |
| 13.7 | Unbounded recursion in data processing code | [S] | 🟠 |
| 13.8 | Memory leak in long-running process (event listeners not removed) | [S] | 🟡 |
| 13.9 | Synchronous file I/O blocking event loop (Node.js) | [S] AST | 🟠 |
| 13.10 | No circuit breaker on downstream service calls | [S] | 🟡 |

---

## 14. Cryptography

| # | Vulnerability | Detection | Severity |
|---|---------------|-----------|----------|
| 14.1 | MD5 or SHA1 used for password hashing | [S] grep | 🔴 |
| 14.2 | MD5 or SHA1 used for security tokens | [S] grep | 🔴 |
| 14.3 | Custom cryptography implementation | [S] | 🔴 |
| 14.4 | Weak random number generator for tokens (`Math.random()`) | [S] grep | 🔴 |
| 14.5 | ECB mode used for symmetric encryption | [S] | 🔴 |
| 14.6 | Hardcoded IV/nonce in AES encryption | [S] | 🔴 |
| 14.7 | RSA key size < 2048 bits | [S] | 🟠 |
| 14.8 | TLS certificate expired or self-signed in production | [R] | 🟠 |
| 14.9 | Symmetric encryption key derived from password without PBKDF2/scrypt/argon2 | [S] | 🟠 |
| 14.10 | Tokens not signed - integrity not verified before use | [S] | 🔴 |

---

## 15. Supply Chain & Dependency Security

| # | Vulnerability | Detection | Severity |
|---|---------------|-----------|----------|
| 15.1 | No lockfile (`package-lock.json`, `yarn.lock`, `go.sum`) committed | [S] | 🟠 |
| 15.2 | Packages installed from unverified or typosquatted names | [S] | 🔴 |
| 15.3 | `postinstall` scripts in dependencies not audited | [S] | 🟠 |
| 15.4 | Dependencies not pinned to exact version | [S] | 🟡 |
| 15.5 | No automated dependency update scanning (Dependabot, Renovate) | [C] | 🟠 |
| 15.6 | CI/CD pipeline secrets exposed in build logs | [C] | 🔴 |
| 15.7 | CI/CD pipeline runs with excessive permissions | [C] | 🟠 |
| 15.8 | No code signing on published packages | [C] | 🟡 |
| 15.9 | Third-party code copied into repo without license or audit | [S] | 🟡 |

---

## 16. Business Logic

| # | Vulnerability | Detection | Severity |
|---|---------------|-----------|----------|
| 16.1 | Price or quantity manipulable via API request tampering | [R] | 🔴 |
| 16.2 | Coupon / promo codes applicable unlimited times per user | [R] | 🟠 |
| 16.3 | Free trial restartable by re-registering with same email alias | [R] | 🟠 |
| 16.4 | Race condition on credit/balance deduction (double-spend) | [R] | 🔴 |
| 16.5 | Negative quantity accepted in purchase flow | [R] | 🟠 |
| 16.6 | No idempotency key on critical write operations | [S] | 🟠 |
| 16.7 | Workflow state machine bypassable (skip steps via direct API call) | [R] | 🟠 |
| 16.8 | Export/download endpoint allows exfiltrating all user data without scope check | [R] | 🔴 |
| 16.9 | Account deletion does not purge all associated data | [S][R] | 🟠 |
| 16.10 | Referral/affiliate system gameable (self-referral) | [R] | 🟡 |

---

## 17. Emerging Patterns

> Items in this category are provisional. They describe patterns observed in the wild but not yet confirmed across enough codebases to assign a stable severity rating. Treat these as areas to investigate rather than definitive findings.

| # | Vulnerability | Detection | Severity |
|---|---------------|-----------|----------|
| 17.1 | Computer use agent captures sensitive screen content (passwords, PII visible on screen) in screenshots sent to LLM | [C] | 🔴 |
| 17.2 | AI-assisted code review manipulated via adversarial comments in PR that alter the review model's output | [R] | 🟠 |
| 17.3 | Stateful AI agent shares context across user sessions - prior user's data bleeds into the next user's session | [S][R] | 🔴 |
| 17.4 | Prompt injection via image metadata - EXIF or embedded text in uploaded images carries instructions read by a multimodal model | [S] | 🔴 |

---

## Summary Table

| Category | Items | Critical 🔴 | High 🟠 | Medium 🟡 | Low 🟢 |
|----------|-------|------------|---------|----------|--------|
| 1. Auth & Sessions | 21 | 9 | 9 | 1 | 2 |
| 2. Authorization | 12 | 9 | 3 | 0 | 0 |
| 3. Input & Injection | 20 | 11 | 7 | 2 | 0 |
| 4. API Security | 18 | 8 | 8 | 2 | 0 |
| 5. Secrets & Config | 15 | 7 | 6 | 2 | 0 |
| 6. AI-Specific | 33 | 14 | 15 | 2 | 2 |
| 7. Database | 18 | 4 | 10 | 4 | 0 |
| 8. Infrastructure | 28 | 7 | 16 | 4 | 1 |
| 9. Frontend | 17 | 6 | 8 | 3 | 0 |
| 10. Email & Payments | 15 | 9 | 5 | 1 | 0 |
| 11. Password Reset | 8 | 4 | 3 | 0 | 1 |
| 12. Logging & Monitoring | 10 | 1 | 7 | 2 | 0 |
| 13. Performance | 10 | 0 | 5 | 5 | 0 |
| 14. Cryptography | 10 | 6 | 4 | 0 | 0 |
| 15. Supply Chain | 9 | 2 | 4 | 2 | 1 |
| 16. Business Logic | 10 | 3 | 6 | 0 | 1 |
| 17. Emerging Patterns | 4 | 3 | 1 | 0 | 0 |
| **Total** | **258** | **117** | **118** | **30** | **8** |

---

## Detection Method Reference

| Method | Tooling Suggestions |
|--------|---------------------|
| Static grep | `ripgrep`, `semgrep` rules |
| AST analysis | `ts-morph` (TS/JS), `ast-grep`, `tree-sitter` |
| Entropy scan | `trufflehog`, `gitleaks`, custom Shannon entropy scanner |
| Git history scan | `gitleaks`, `git log -p` piped through grep |
| Runtime HTTP | Custom test runner hitting `localhost` or staging URL |
| Headers check | `curl -I` + header parser |
| npm audit | `npm audit --json` |
| Config inspection | Parse `.env.example`, `docker-compose.yml`, ORM config files |
| ORM scan | Parse migration files and model definitions for index declarations |

---

*Version 1.0 - contributions via PR following the schema above.*