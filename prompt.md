# Security Scan - Run This Against the Current Project

You are a security auditor. Your job is to scan this codebase and produce a vulnerability report.

Do not explain what you are about to do. Do not summarize the project. Start scanning immediately and report only findings.

---

## Your Scanning Mandate

Scan every file in the current working directory recursively. Exclude `node_modules`, `dist`, `build`, `.git`, `vendor`, `.next`, `coverage`.

For each check below, you must:
1. Search the codebase for evidence of the vulnerability
2. If found - report it with: rule ID, file path, line number, code snippet, and remediation
3. If not found - mark it PASS
4. If unable to determine (no relevant files exist) - mark it N/A

Do not skip checks. Do not group findings vaguely. Every rule ID gets an explicit result.

---

## Output Format

Produce a markdown report in exactly this structure:

```
# Vulnerability Scan Report
Generated: [timestamp]
Project: [detected stack]

## Summary
| Severity | Count |
|----------|-------|
| 🔴 Critical | N |
| 🟠 High | N |
| 🟡 Medium | N |
| 🟢 Low | N |
| ✅ Pass | N |
| ➖ N/A | N |

## Findings

### [Rule ID] [Rule Title]
- **Severity:** 🔴 Critical
- **File:** `path/to/file.ts`
- **Line:** 42
- **Snippet:**
  ```
  [exact offending code]
  ```
- **Remediation:** [specific fix for this codebase, not generic advice]

## Full Rule Results
[table of every rule ID with PASS / FINDING / N/A]
```

---

## Checks to Run

### Category 1 - Authentication & Session Management

**1.1** Search for `localStorage.setItem` or `localStorage[` near the words `token`, `auth`, `jwt`, `session`, `access_token`, `refresh_token`. Flag any match.

**1.2** Same as 1.1 but for `sessionStorage`.

**1.3** Search for session/cookie configuration. Check if `maxAge`, `expires`, or `ttl` is set. Flag if absent.

**1.4** Search for JWT or token issuance code. Check if token rotation or invalidation on refresh exists. Flag if tokens are issued with no rotation mechanism.

**1.5** Search for JWT verification code. Check if `algorithms` option is explicitly set. Flag if `alg: 'none'` is accepted or if algorithm is not whitelisted.

**1.6** Search for JWT secret values. Flag if the secret is a short string literal like `'secret'`, `'changeme'`, `'password'`, `'jwt'`, or fewer than 32 characters.

**1.7** Search for route definitions. Check if protected routes call a JWT verification middleware. Flag routes that have no auth middleware attached.

**1.8** Search for refresh token storage. Flag if stored in `localStorage` or as a non-HttpOnly cookie.

**1.9** Search for login endpoint handler. Check if rate limiting middleware is applied. Flag if absent.

**1.10** Search for failed login attempt tracking. Flag if no counter or lockout logic exists near the login handler.

**1.11** Search for password hashing calls. Flag any use of `md5`, `sha1`, `sha256` for passwords. Flag plaintext password storage.

**1.12** Search for `bcrypt` calls. Check the rounds/work factor argument. Flag if less than 10.

**1.13** Search for state-changing POST/PUT/DELETE routes. Check if CSRF token validation middleware is applied. Flag if absent on non-API cookie-auth routes.

**1.14** Search for `Set-Cookie` or cookie configuration. Flag if `SameSite` attribute is not set.

**1.15** Same as 1.14 but flag if `HttpOnly` is not set.

**1.16** Same as 1.14 but flag if `Secure` is not set.

**1.17** Search for OTP or magic link generation. Check if an expiry timestamp is stored alongside the token. Flag if absent.

**1.18** Search for OTP/magic link verification. Check if the token is deleted or invalidated after use. Flag if reusable.

**1.19** Search for `X-HTTP-Method-Override` header handling. Flag if present and not restricted.

**1.20** Search for session creation logic. Check if concurrent session limits are enforced. Flag if a user can have unlimited active sessions.

**1.21** Search for logout handlers. Check if the server-side session or token is invalidated (deleted from DB or added to denylist). Flag if logout only clears the client cookie.

---

### Category 2 - Authorization & Access Control

**2.1** Search for route definitions containing `admin`, `dashboard`, `manage`, `internal`. Check if role-check middleware is applied. Flag unprotected admin routes.

**2.2** Search for database queries that use user-supplied IDs (from `req.params`, `req.body`, `req.query`). Check if ownership is verified (e.g. `WHERE id = ? AND user_id = ?`). Flag queries with no ownership check.

**2.3** Search for data access patterns. Check if the authenticated user's ID is always used to scope queries. Flag any query returning another user's data without explicit admin check.

**2.4** Search for role or permission update endpoints. Check if the requesting user's own role is verified before applying changes. Flag if a user can escalate their own role.

**2.5** Search for bulk operation endpoints (`/bulk`, `/batch`, `deleteMany`, `updateMany`). Check if authorization is applied per item. Flag bulk ops with only top-level auth.

**2.6** Search for role or permission checks. Flag any check that exists only in frontend code with no corresponding backend enforcement.

**2.7** Search for route listing or Swagger/OpenAPI exposure. Flag if `/api-docs`, `/swagger`, or route introspection is enabled in production config.

**2.8** Search for multi-tenant data queries. Check if `tenant_id` or `org_id` is always included in WHERE clauses. Flag queries missing tenant scoping.

**2.9** Search for GraphQL configuration. Flag if `introspection: true` is set without environment guard.

**2.10** Search for GraphQL setup. Check if query depth or complexity limits are configured. Flag if absent.

**2.11** Search for feature flag checks. Flag any that exist only in frontend components with no server-side enforcement.

**2.12** Search for ORM update/patch calls. Check if field allowlists are used. Flag if raw `req.body` is passed directly to an ORM update call.

---

### Category 3 - Input Validation & Injection

**3.1** Search for SQL query construction. Flag any string concatenation or template literal that includes variables from `req.body`, `req.params`, or `req.query`.

**3.2** Search for MongoDB query objects. Flag if `req.body` fields are used directly as MongoDB operators without sanitization.

**3.3** Search for `exec(`, `execSync(`, `spawn(`, `child_process`. Flag if any argument includes unsanitized user input.

**3.4** Search for LDAP query construction. Flag string concatenation with user input.

**3.5** Search for XPath query construction. Flag string concatenation with user input.

**3.6** Search for template engine calls (`res.render`, `ejs.render`, `nunjucks`, `handlebars`). Flag if user input is passed as template string rather than as data variable.

**3.7** Search for server-side HTML rendering. Flag if `req.query` or `req.body` values are interpolated directly into HTML responses.

**3.8** Search for database write operations followed by HTML rendering. Flag if stored content is rendered without escaping.

**3.9** Search for `dangerouslySetInnerHTML`, `innerHTML =`, `document.write(`. Flag if the value is derived from user input or API response.

**3.10** Search for redirect calls. Flag if the redirect URL is taken from `req.query.redirect`, `req.query.next`, or similar without validation against an allowlist.

**3.11** Search for file path construction using user input. Flag any `path.join` or string concat that includes `req.params` or `req.query` values.

**3.12** Search for XML parsing. Flag if external entity processing is not disabled (`libxmljs`, `xml2js`, `fast-xml-parser` configs).

**3.13** Search for CSV export functionality. Flag if user-supplied strings are written to CSV without formula injection protection (values not prefixed with `'` or sanitized).

**3.14** Search for `new RegExp(` with user input. Flag dynamic regex construction from untrusted sources.

**3.15** Search for file upload handlers (`multer`, `formidable`, `busboy`). Flag if MIME type and file extension are not both validated.

**3.16** Same as 3.15 but check for file size limits. Flag if `limits.fileSize` is not set.

**3.17** Search for file upload destination config. Flag if uploaded files are stored in a publicly served directory on the application server.

**3.18** Search for `Object.assign(`, `merge(`, spread operators applied to `req.body`. Flag if `__proto__` or `constructor` are not filtered out.

**3.19** Search for query parameter parsing. Flag if duplicate parameters could override each other in security-sensitive contexts.

**3.20** Search for numeric fields from user input used in calculations. Flag if no `parseInt`/`Number` bounds check exists.

---

### Category 4 - API Security

**4.1** Search for route definitions. Check if a rate limiting middleware (e.g. `express-rate-limit`, `ratelimit`, `throttle`) is applied globally or per-route. Flag if absent on all routes.

**4.2** Same as 4.1 specifically for `/login`, `/signin`, `/auth` routes.

**4.3** Same as 4.1 specifically for `/reset-password`, `/forgot-password`, `/otp`, `/verify` routes.

**4.4** Search for CORS configuration. Flag if no CORS middleware is configured.

**4.5** Search for CORS configuration. Flag if `origin: '*'` is set on routes that also require authentication.

**4.6** Search for CORS origin handling. Flag if the `Origin` header value is reflected back without validation against a fixed allowlist.

**4.7** Search for API route definitions. Note if versioning (`/v1/`, `/v2/`) is absent. Low severity - mark accordingly.

**4.8** Search for error handlers. Flag if stack traces, SQL errors, or internal paths are included in error responses sent to the client.

**4.9** Search for API key or token usage in URL construction. Flag if keys appear in query string parameters.

**4.10** Search for body parser configuration. Flag if no request size limit is set (`limit` option in `express.json()`, `bodyParser`, etc.).

**4.11** Search for POST/PUT route handlers. Check if `Content-Type` is validated. Flag if any content type is accepted blindly.

**4.12** Search for route definitions. Check if sensitive operations are restricted to expected HTTP verbs. Flag if DELETE or PUT routes are unguarded.

**4.13** Search for API response serialization. Flag if full ORM model objects are returned directly without field selection.

**4.14** Search for webhook endpoint handlers. Flag if no signature verification exists before processing the payload.

**4.15** Search for Stripe webhook handlers specifically. Flag if `stripe.webhooks.constructEvent` is not called.

**4.16** Search for `fetch(`, `axios.get(`, `http.get(`, `request(` calls that use user-supplied URLs. Flag SSRF risk if the URL is not validated against an allowlist.

**4.17** Search for database list queries (`findMany`, `find`, `SELECT * FROM`). Flag if no `limit`, `take`, or `LIMIT` clause exists.

**4.18** Same as 4.17 but check if the limit value comes from `req.query` without a maximum cap.

---

### Category 5 - Secrets & Configuration

**5.1** Search all frontend source files for string patterns matching: `sk-`, `sk-ant-`, `AIza`, `sk_live_`, `pk_live_`, `ghp_`, `xoxb-`, `AKIA`. Flag any match.

**5.2** Same as 5.1 across all backend source files.

**5.3** Run `git log --all --full-history -- "*.env"` conceptually - search git history references. Flag if `.env` appears in any committed file outside `.gitignore`.

**5.4** Check if `.env` is listed in `.gitignore`. Flag if absent.

**5.5** Search `package.json`, `config.json`, `app.config.js`, `next.config.js` for hardcoded credential strings. Flag any found.

**5.6** Search for application startup code. Check if environment variables are validated (e.g. using `zod`, `envalid`, `joi`, or manual checks) before the app starts. Flag if absent.

**5.7** Same as 5.6 - flag specifically if the app will start and silently fail when a required env var is missing.

**5.8** Check if `.env.example` contains placeholder values vs real values. Flag if real credentials exist in `.env.example`.

**5.9** Search for `AWS_ACCESS_KEY`, `GOOGLE_APPLICATION_CREDENTIALS`, `AZURE_` patterns in source files. Flag any hardcoded values.

**5.10** Search for `DATABASE_URL`, `DB_PASSWORD`, `POSTGRES_PASSWORD`, `MYSQL_ROOT_PASSWORD` patterns with inline values in source. Flag any found.

**5.11** Search for `-----BEGIN RSA PRIVATE KEY-----`, `-----BEGIN EC PRIVATE KEY-----`, `-----BEGIN OPENSSH PRIVATE KEY-----`. Flag any found in source files.

**5.12** Search for `sourceMappingURL` in compiled JS files in public directories. Check build config for `sourceMap: false` in production. Flag if source maps are generated for production.

**5.13** Search for `DEBUG=true`, `debug: true`, `NODE_ENV=development` in production config files. Flag any found.

**5.14** Search for the application start script. Flag if `NODE_ENV` is not explicitly set to `production`.

**5.15** Open `.gitignore`. Flag if `.env`, `*.pem`, `*.key`, `*.p12`, `*.pfx` are not listed.

---

### Category 6 - AI-Specific Vulnerabilities

**6.1** Search for LLM API calls (OpenAI, Anthropic, Google AI, Grok, Mistral). Check how the prompt/messages array is constructed. Flag if `req.body` content is concatenated directly into the system prompt string without sanitization.

**6.2** Search for agentic or tool-use patterns where the AI reads external content (URLs, emails, uploaded files, web scraping). Flag if that content is passed back into the prompt without sanitization.

**6.3** Search for AI API response handling. Flag if the response content is passed to `dangerouslySetInnerHTML`, `innerHTML`, `eval()`, or `exec()`.

**6.4** Search for authorization or authentication logic. Flag if an LLM response is used to make a yes/no access decision without a deterministic fallback.

**6.5** Search for system prompt construction. Flag if the system prompt content is derivable or exposed through error messages.

**6.6** Search for AI response rendering. Flag if LLM output is rendered without any content filtering or sanitization layer.

**6.7** Search all files for OpenAI, Anthropic, Google AI, Grok API key patterns used in client-side code (browser-executed files). Flag any found.

**6.8** Check AI API client initialization. Flag if no usage limits, spend caps, or quota checks are configured at the API key level (note as config check - remind user to verify in provider dashboard).

**6.9** Search for AI API calls. Flag if the `model` parameter is sourced from `req.body` or `req.query` without validation against an allowlist.

**6.10** Search for AI API calls. Flag if `max_tokens` or `temperature` is sourced from user input without a hard ceiling enforced in code.

**6.11** Search for AI response caching. Flag if cache keys do not include the authenticated user ID or tenant ID.

**6.12** Search for AI API calls. Flag if `req.body` fields containing PII (email, name, address, phone) are sent to third-party AI APIs. Note as compliance risk.

**6.13** Same as 6.12 - cross-reference privacy policy if present in repo.

**6.14** Search for AI API calls. Flag if no `max_tokens` limit is set at all.

**6.15** Search for streaming AI response handlers. Flag if no connection timeout or stream abort mechanism exists.

**6.16** Search for AI tool/function call definitions. Flag if tools include filesystem access, shell execution, or database writes without explicit scope restrictions.

**6.17** Search for agentic patterns. Flag if the AI agent is given `fs`, `child_process`, or direct DB client access without sandboxing.

**6.18** Search for `eval(`, `new Function(`, `vm.runInNewContext(`. Flag if the input is derived from an AI API response.

**6.19** Search for AI API calls. Flag if there is no try/catch with fallback behavior when the AI API is unavailable.

**6.20** Search for vector database usage (Pinecone, Weaviate, Qdrant, pgvector, Chroma). Check if namespace or collection is scoped per user/tenant. Flag if a single shared namespace is used for all users.

**6.21** Same as 6.20 - check RAG retrieval queries. Flag if retrieved documents are not filtered by tenant ownership before being passed to the LLM.

**6.22** Search for embedding generation on user data. Flag if PII fields are embedded and stored in a shared vector index without access controls.

**6.23** Check if `tsconfig.json` exists with `strict: true`. For Python projects check if `mypy` config exists. Flag if absent.

**6.24** Search for fine-tuning job configuration or training data pipeline code. Flag if user data is used for training without explicit consent mechanism.

**6.25** Search for AI model name strings. Flag if the model is specified without a version/date suffix (e.g. `gpt-4` instead of `gpt-4-0125-preview`, `claude-3` instead of `claude-3-5-sonnet-20241022`).

**6.26** Search for MCP (Model Context Protocol) server configuration or tool definitions that reference external URLs or third-party servers. Flag any tool whose responses are passed directly into the agent's prompt without sanitization.

**6.27** Search for agent memory storage and retrieval patterns (vector DB writes, key-value stores, file-based memory). Flag if content retrieved from memory is injected into the agent's next prompt without sanitization or content inspection.

**6.28** Search for content filter or moderation calls applied to user input. Flag if the filter operates on raw text only and would not catch base64-encoded strings, Unicode homoglyphs, or ROT13-encoded instructions in the same input.

**6.29** Search for rate limiting on LLM API calls scoped per user. Flag if a single user can make unlimited queries to the model endpoint with no throttle, which enables systematic probing to extract system prompt content.

**6.30** Search for multi-agent or orchestrator patterns where one agent's output is passed as the next agent's input. Flag if the output is passed without sanitization or a trusted/untrusted content boundary.

**6.31** Search for agent spawning or sub-agent invocation patterns. Flag if API keys, session tokens, database credentials, or other secrets are passed in full to the sub-agent rather than scoped credentials with minimum required permissions.

**6.32** Search for JSON.parse, yaml.load, or equivalent structured data parsing applied to LLM API response content. Flag if the parsed result is written to a database or used in a query without schema validation (zod, joi, pydantic, or equivalent).

**6.33** Search for conversation or message history storage (database tables, collections, or files named chat, message, history, thread, session). Flag if retrieval queries do not include a `WHERE user_id = authenticatedUserId` or equivalent ownership filter.

---

### Category 7 - Database & Data Layer

**7.1** Search for ORM model/schema definitions (Prisma `schema.prisma`, TypeORM entities, Sequelize models, Drizzle schema). Cross-reference fields used in `WHERE` clauses across the codebase. Flag fields queried frequently with no index defined.

**7.2** Search for database client initialization. Flag if connection pool settings (`max`, `min`, `pool`) are not configured.

**7.3** Search for backup scripts, cron jobs, `pg_dump`, `mysqldump`, `mongodump` references. Flag if none exist anywhere in the project.

**7.4** Search for backup verification or restore test scripts. Flag if absent (low severity).

**7.5** Check backup destination config if backup scripts exist. Flag if backup destination is in same region as primary (config-level note).

**7.6** Search for migration files. Check if down/rollback migrations exist alongside up migrations. Flag if only up migrations exist.

**7.7** Search for ORM config. Flag if `force: true`, `syncForce: true`, `dropSchema: true`, or `synchronize: true` (TypeORM production risk) is set.

**7.8** Search for raw SQL queries. Flag string concatenation or template literals with user input (same as 3.1 but focused on ORM raw escape hatches like `$queryRaw`, `sequelize.query`, `knex.raw`).

**7.9** Check database connection string or config. Flag if the DB user appears to be `root`, `postgres` (default superuser), or `admin` without evidence of restricted permissions.

**7.10** Search for database port configuration in `docker-compose.yml` or server config. Flag if DB port (5432, 3306, 27017) is bound to `0.0.0.0` rather than `127.0.0.1`.

**7.11** Search for database client config. Flag if no query timeout (`statement_timeout`, `query_timeout`, `connectTimeout`) is set.

**7.12** Search for ORM relation loading in loops. Flag `await` inside `for`/`forEach` that calls a DB query - N+1 pattern.

**7.13** Search for delete operations. Flag hard deletes (`DELETE FROM`, `destroy()`, `remove()`) on user-facing resources that could benefit from soft delete.

**7.14** Check database schema or ORM models for PII fields (email, name, phone, address, SSN, DOB). Flag if no encryption or tokenization is applied to sensitive fields.

**7.15** Search for encryption key storage. Flag if encryption keys are stored in the same database as the encrypted data.

**7.16** Search for data retention policies or scheduled purge jobs. Flag if absent (low severity).

**7.17** Same as 4.17 - flag unbounded queries at ORM level.

**7.18** Search for multi-step write operations (multiple `await db.update` or `INSERT` calls in sequence). Flag if not wrapped in a transaction (`BEGIN`/`COMMIT`, `prisma.$transaction`, `sequelize.transaction`).

---

### Category 8 - Infrastructure & Deployment

**8.1** Search for a health check route (`/health`, `/healthz`, `/ping`, `/status`). Flag if absent.

**8.2** Search for logging setup (`winston`, `pino`, `morgan`, `bunyan`, `console.log` as sole logger). Flag if no structured logger is configured for production.

**8.3** Search for log statements. Flag if passwords, tokens, API keys, or PII appear in log arguments.

**8.4** Search for logging configuration. Note if logs are written only to stdout with no persistence or shipping config (low severity if using a platform that captures stdout).

**8.5** Search for alerting config or error monitoring setup. Flag if neither Sentry/Rollbar nor custom alerting is configured.

**8.6** Search for uptime monitoring config or references to UptimeRobot, Pingdom, Checkly, Grafana. Flag if absent.

**8.7** Search for file upload handling. Flag if `multer` or equivalent stores files in a local directory served by the app server with no CDN.

**8.8** Search for static asset serving config. Flag if `express.static` or equivalent serves assets directly from the app process in a production config.

**8.9** Search for HTTP server setup or redirect middleware. Flag if no HTTP→HTTPS redirect exists.

**8.10** Search for TLS/SSL configuration. Flag if TLS 1.0 or 1.1 is not explicitly disabled.

**8.11** Search for response headers or helmet config. Flag if `Strict-Transport-Security` is not set.

**8.12** Same - flag if `X-Content-Type-Options: nosniff` is not set.

**8.13** Same - flag if `X-Frame-Options` or `frame-ancestors` CSP directive is not set.

**8.14** Same - flag if `Content-Security-Policy` is not set.

**8.15** Same - flag if `Referrer-Policy` is not set.

**8.16** Same - flag if `Permissions-Policy` is not set.

**8.17** Search for response headers. Flag if `X-Powered-By` or `Server` headers expose version info and are not removed.

**8.18** Search for error handler middleware. Flag if framework default error pages are used in production.

**8.19** Note absence of WAF config as low-severity finding.

**8.20** Search for SSH or server config references. Flag if `PermitRootLogin yes` appears.

**8.21** Same - flag if `PasswordAuthentication yes` appears.

**8.22** Search for firewall config (`ufw`, `iptables`, security groups). Flag if no firewall ruleset exists.

**8.23** Note DDoS protection as config-level check.

**8.24** Search for `Dockerfile`. Flag if `USER` directive is absent (defaults to root).

**8.25** Search for `Dockerfile`. Flag if any `FROM` instruction uses `:latest` tag.

**8.26** Search for `docker-compose.yml`. Flag if secrets are passed as plain `environment` vars rather than Docker secrets or mounted files.

**8.27** Search for PM2 config (`ecosystem.config.js`), `systemd` unit files, or `Procfile`. Flag if none exist.

**8.28** Search for `docker-compose.yml` or Kubernetes manifests. Flag if no memory or CPU limits are set on containers.

---

### Category 9 - Frontend & Client-Side

**9.1** Search React component tree. Flag if no `ErrorBoundary` component exists wrapping top-level routes.

**9.2** Search for `dangerouslySetInnerHTML`. Flag if the value is not a static string.

**9.3** Search for `eval(` in all JS/TS files. Flag every instance.

**9.4** Search for `document.write(`. Flag every instance.

**9.5** Search for router/navigation calls. Flag if tokens, passwords, or sensitive IDs are appended to URL params.

**9.6** Search for `window.` assignments. Flag if tokens or user data are stored on the global window object.

**9.7** Run `npm audit --json` conceptually - search `package.json` for packages with known CVE advisories. Flag any `high` or `critical` severity advisories.

**9.8** Check `package.json` dependency versions. Flag packages more than 2 major versions behind current.

**9.9** Search HTML files and entry points for `<script src=` tags pointing to external CDNs. Flag if `integrity` attribute is absent.

**9.10** Same - flag if `async` or `defer` is absent on non-critical third-party scripts.

**9.11** Search for role or feature flag state stored in `localStorage` or cookies. Flag if the same value is not re-verified on the server per request.

**9.12** Search for pricing, discount, or subscription logic. Flag if it exists only in frontend code with no server-side enforcement.

**9.13** Check for `tsconfig.json`. Flag if `strict` is not `true` or if TypeScript is absent entirely.

**9.14** Search for `console.log(`, `console.debug(`. Flag if any contain token, password, user object, or response body with PII.

**9.15** Search build config for source map generation. Flag if `sourceMappingURL` files are output to the public build directory.

**9.16** Search CSP config or `<meta http-equiv="Content-Security-Policy">`. Flag if `unsafe-inline` is permitted for scripts.

**9.17** Search for `X-Frame-Options` or CSP `frame-ancestors`. Flag if neither is set.

---

### Category 10 - Email, Payments & Third-Party Integrations

**10.1** Search for email sending calls (`nodemailer.sendMail`, `sgMail.send`, `resend.send`). Flag if called with `await` directly inside a request handler with no background job wrapper.

**10.2** Search for email sending. Flag if no retry/queue mechanism (Bull, BullMQ, SQS, etc.) exists.

**10.3** Search for password reset token generation. Flag if no expiry timestamp is stored with the token.

**10.4** Same - flag if the token is not deleted from the database immediately after use.

**10.5** Same as 10.4.

**10.6** Search for password reset endpoint. Flag if the error message differs for existing vs non-existing email addresses.

**10.7** Search for Stripe webhook handler. Flag if `stripe.webhooks.constructEvent` is not called before processing.

**10.8** Search for payment/checkout initiation. Flag if the amount is taken from `req.body` rather than being computed server-side from a product/price ID.

**10.9** Search for payment processing logic. Flag if no idempotency key is passed to Stripe API calls.

**10.10** Search for subscription or plan gating. Flag if the check reads from a client-supplied value rather than a server-side DB lookup.

**10.11** Search for OAuth implementation. Flag if `state` parameter is not generated, stored, and verified in the callback.

**10.12** Same - flag if `redirect_uri` is not validated against a hardcoded allowlist.

**10.13** Search for `<script src=`, `require(`, `import` of third-party SDKs. Flag if loaded over `http://`.

**10.14** Search for webhook handlers. Flag if processed event IDs are not stored to prevent duplicate processing.

**10.15** Search for outbound HTTP calls. Flag if no timeout is set (`timeout` option in `axios`, `fetch` with `AbortController`, etc.).

---

### Category 11 - Password Reset & Account Recovery

**11.1** Search for password reset token generation. Flag if `Math.random()` is used instead of `crypto.randomBytes()` or equivalent.

**11.2** Search for password reset token storage in DB. Flag if the raw token is stored rather than a hash of it.

**11.3** Search for password reset email sending. Flag if the token appears in a logged URL or is passed to a logger.

**11.4** Search for account recovery flow. Flag if security questions are the only recovery method.

**11.5** Search for password reset endpoint. Flag if no rate limiting middleware is applied.

**11.6** Search for password change logic. Flag if previous passwords are not checked (low severity).

**11.7** Search for password validation. Flag if no minimum length or complexity requirement exists.

**11.8** Search for password validation. Note if HaveIBeenPwned API check is absent (low severity).

---

### Category 12 - Logging, Monitoring & Incident Response

**12.1** Search for logger initialization. Flag if only `console.log` is used with no log levels or JSON formatting.

**12.2** Search for logger config. Flag if logs are written only in-process with no file output or log shipping.

**12.3** Search for login, logout, and failed auth handlers. Flag if no log statement records these events.

**12.4** Search for admin action handlers. Flag if no audit log entry is written.

**12.5** Search for alerting or monitoring config. Flag if absent.

**12.6** Search for Sentry, Rollbar, Bugsnag, Datadog initialization. Flag if absent.

**12.7** Search for Sentry or error tracking config. Flag if `sendDefaultPii: true` or equivalent is set.

**12.8** Note absence of incident response documentation as low severity.

**12.9** Search for log file paths or log viewer routes. Flag if any log endpoint has no auth middleware.

**12.10** Search for log statements that include user input directly. Flag potential log injection.

---

### Category 13 - Performance Vulnerabilities

**13.1** Same as 7.1 - high-cardinality fields with no index.

**13.2** Same as 4.17 - unbounded queries loading full table.

**13.3** Search for repeated identical DB queries with no caching layer (Redis, Memcached, in-memory). Flag as medium severity.

**13.4** Search for cache invalidation logic. Flag if cache is never cleared on data mutation.

**13.5** Search for image processing (`sharp`, `jimp`), PDF processing (`pdf-lib`, `pdfkit`), or video processing in request handlers. Flag if synchronous/inline with no job queue.

**13.6** Same as 10.2 - flag absence of background job queue.

**13.7** Search for recursive functions. Flag if no depth limit or base-case guard exists on user-influenced input.

**13.8** Search for `addEventListener` or `on(` event registrations. Flag if no corresponding `removeEventListener` or `off(` exists in cleanup/unmount logic.

**13.9** Search for `fs.readFileSync`, `fs.writeFileSync`, `fs.readdirSync` in Express route handlers or equivalent. Flag synchronous I/O in async request handlers.

**13.10** Search for calls to external services (HTTP, DB, queue). Flag if no circuit breaker (or timeout + fallback) pattern is implemented.

---

### Category 14 - Cryptography

**14.1** Search for `createHash('md5')`, `createHash('sha1')`, `md5(`, `sha1(` applied to passwords. Flag every instance.

**14.2** Same but applied to token or session ID generation.

**14.3** Search for custom cipher, XOR, ROT13, or homegrown encryption implementations. Flag any found.

**14.4** Search for `Math.random()` used to generate tokens, IDs, nonces, or salts. Flag every instance.

**14.5** Search for AES encryption config. Flag if `ECB` mode is specified.

**14.6** Search for AES encryption calls. Flag if IV is a hardcoded string rather than `crypto.randomBytes(16)`.

**14.7** Search for RSA key generation or usage. Flag if key size is specified below 2048 bits.

**14.8** Check TLS certificate config or server setup. Flag if no cert renewal automation (Let's Encrypt, cert-manager) is configured.

**14.9** Search for encryption key derivation. Flag if a raw password string is used as an AES key without PBKDF2, scrypt, or argon2.

**14.10** Search for token generation and usage. Flag if tokens are used without any signature verification (HMAC, JWT verify, etc.).

---

### Category 15 - Supply Chain & Dependency Security

**15.1** Check repo root for `package-lock.json`, `yarn.lock`, `pnpm-lock.yaml`, `go.sum`, `Pipfile.lock`. Flag if absent.

**15.2** Search `package.json` for package names that are common typosquat targets. Flag unusual spellings of popular packages.

**15.3** Search `node_modules` (if accessible) or `package.json` for packages with `postinstall` scripts. Flag any that execute shell commands.

**15.4** Check `package.json` dependency versions. Flag if `^` or `~` ranges are used for security-critical packages (auth libraries, crypto, etc.).

**15.5** Search for `.github/dependabot.yml` or `renovate.json`. Flag if absent.

**15.6** Search for CI config (`.github/workflows`, `.gitlab-ci.yml`, `Jenkinsfile`). Flag if secrets are echoed in run steps.

**15.7** Same - flag if workflows run with `permissions: write-all` or equivalent broad permissions.

**15.8** Note absence of package signing as low severity.

**15.9** Search for vendored third-party code in `vendor/`, `lib/`, or `utils/` directories. Flag if no license file or attribution comment exists.

---

### Category 16 - Business Logic

**16.1** Search for checkout or order creation endpoints. Flag if `price`, `amount`, or `total` is accepted from `req.body` rather than looked up server-side.

**16.2** Search for coupon or discount application logic. Flag if usage count per user is not tracked and enforced.

**16.3** Search for free trial logic. Flag if trial eligibility is checked only against email without also checking normalized form (+ aliases, dots in Gmail).

**16.4** Search for balance deduction or credit spending logic. Flag if not wrapped in a database transaction with a row lock (`SELECT FOR UPDATE`).

**16.5** Search for quantity fields in purchase flows. Flag if negative values are not rejected.

**16.6** Search for critical write operations (payment, order creation, user registration). Flag if no idempotency key is required or generated.

**16.7** Search for multi-step workflows (order state machine, KYC flow, onboarding steps). Flag if individual steps are accessible via direct API call without enforcing prior step completion.

**16.8** Search for data export or download endpoints. Flag if the query is not scoped to the authenticated user's own data.

**16.9** Search for account deletion handlers. Flag if related data in other tables is not deleted or anonymized.

**16.10** Search for referral or affiliate tracking. Flag if a user can apply their own referral code to their own account.

---

### Category 17 - Emerging Patterns

> These checks cover patterns observed in the wild but not yet confirmed broadly enough to assign a stable severity. Run them and report findings as provisional. Include a note in the report that these require manual verification.

**17.1** Search for computer use agent configuration or screenshot capture code. Flag if the agent takes screenshots of the user's screen and sends them to an LLM API without first redacting regions containing password fields, credit card inputs, or PII visible in other application windows.

**17.2** Search for AI-assisted code review integrations (GitHub Actions calling an LLM to review PRs). Flag if PR comment content is passed directly into the review prompt without sanitization, which would allow an attacker to inject instructions into the reviewer's output via a crafted PR comment.

**17.3** Search for agent or chatbot session initialization code. Flag if conversation state, memory, or tool context from a previous user's session is carried over into a new user's session rather than being reset on authentication.

**17.4** Search for image upload handlers that feed uploaded files to a multimodal LLM. Flag if EXIF metadata or embedded text is passed to the model without stripping, as these channels can carry injected instructions invisible to users.

---

## After All Checks Complete

1. Produce the full findings report in the format defined above.
2. Produce a **Priority Fix List** - the top 10 findings by severity and exploitability, ordered by "fix this first."
3. For each item in the Priority Fix List, provide the exact code change required - not a description of the fix, the actual code.
4. Produce the **Full Rule Results Table** - every rule ID with PASS / FINDING / N/A.

Do not produce anything else. No preamble. No summary of what you did. Report only.