# AI-Coded App Vulnerability Checklist

Comprehensive reference for static and runtime vulnerability scanning of applications built with AI assistance. Covers Claude/Anthropic, GPT/OpenAI, Gemini/Google, Grok/xAI, Mistral, Copilot/GitHub, and equivalents. Organized by category, with detection method and severity for every item.

[![License](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](LICENSE)
[![Contributions Welcome](https://img.shields.io/badge/contributions-welcome-brightgreen.svg)](CONTRIBUTING.md)

---

## Why this exists

AI coding assistants produce working code fast. They also produce the same security mistakes at scale: missing auth middleware on protected routes, hardcoded secrets, user input concatenated directly into LLM system prompts, vector databases with no tenant isolation, and a long list of patterns that show up repeatedly across AI-generated codebases.

This checklist captures those patterns. Each item has a detection method so you know how to find it, not just that it exists. The checklist covers both traditional web app vulnerabilities and a full category for LLM integration issues that do not appear in older references.

The companion `prompt.md` turns the entire checklist into a structured scan you can run against any codebase with a single command.

---

## What is in this repo

| File | Purpose |
|------|---------|
| [`vulnerabilities.md`](vulnerabilities.md) | 258 vulnerabilities across 17 categories, each with severity rating and detection method |
| [`prompt.md`](prompt.md) | Paste into Claude, ChatGPT, Gemini, Grok, or any capable model to run a structured security scan covering all 258 items |
| [`CONTRIBUTING.md`](CONTRIBUTING.md) | Item schema and guidelines for adding new entries |
| [`CHANGELOG.md`](CHANGELOG.md) | Version history |

---

## The checklist

258 items across 17 categories:

| # | Category | Items |
|---|----------|-------|
| 1 | [Authentication & Session Management](vulnerabilities.md#1-authentication--session-management) | 21 |
| 2 | [Authorization & Access Control](vulnerabilities.md#2-authorization--access-control) | 12 |
| 3 | [Input Validation & Injection](vulnerabilities.md#3-input-validation--injection) | 20 |
| 4 | [API Security](vulnerabilities.md#4-api-security) | 18 |
| 5 | [Secrets & Configuration](vulnerabilities.md#5-secrets--configuration) | 15 |
| 6 | [**AI-Specific Vulnerabilities**](vulnerabilities.md#6-ai-specific-vulnerabilities-llm-integration) | **33** |
| 7 | [Database & Data Layer](vulnerabilities.md#7-database--data-layer) | 18 |
| 8 | [Infrastructure & Deployment](vulnerabilities.md#8-infrastructure--deployment) | 28 |
| 9 | [Frontend & Client-Side](vulnerabilities.md#9-frontend--client-side) | 17 |
| 10 | [Email, Payments & Third-Party Integrations](vulnerabilities.md#10-email-payments--third-party-integrations) | 15 |
| 11 | [Password Reset & Account Recovery](vulnerabilities.md#11-password-reset--account-recovery) | 8 |
| 12 | [Logging, Monitoring & Incident Response](vulnerabilities.md#12-logging-monitoring--incident-response) | 10 |
| 13 | [Performance Vulnerabilities](vulnerabilities.md#13-performance-vulnerabilities-that-become-security-issues-at-scale) | 10 |
| 14 | [Cryptography](vulnerabilities.md#14-cryptography) | 10 |
| 15 | [Supply Chain & Dependency Security](vulnerabilities.md#15-supply-chain--dependency-security) | 9 |
| 16 | [Business Logic](vulnerabilities.md#16-business-logic) | 10 |
| 17 | [Emerging Patterns](vulnerabilities.md#17-emerging-patterns) | 4 |

### Severity legend

- 🔴 **Critical** — exploitable without auth, leads directly to data loss or account takeover
- 🟠 **High** — significant risk, requires some context or chaining to exploit
- 🟡 **Medium** — non-trivial to exploit but meaningful impact
- 🟢 **Low** — defense in depth, best practice

### Detection legend

- **[S]** Static analysis: grep, AST scan, or code review
- **[R]** Runtime: check against a running local or staging server
- **[C]** Config/infrastructure: environment variables, deployment settings, or provider dashboard

---

## How to use `prompt.md`

`prompt.md` instructs an AI model to scan a codebase for all 258 items and return a formatted report with file paths, line numbers, code snippets, and specific remediations.

**Option 1: Claude Code (recommended)**

```bash
# No clone needed - runs directly from GitHub
claude "$(curl -s https://raw.githubusercontent.com/a-leks/genai-app-security-checklist/main/prompt.md)"

# Or if you have cloned the repo
claude "$(cat /path/to/prompt.md)"
```

Run this against codebases you own or have authorization to audit. Claude Code reads your codebase, runs every check, and outputs a findings report.

**Option 2: Claude.ai, ChatGPT, or any web UI**

1. Open the model's file upload or code context feature
2. Paste the contents of `prompt.md` as your message
3. Upload or paste your source files into the same context window
4. The model produces a structured report

**Option 3: CI / pre-commit hook**

Wrap the prompt in a script that passes your changed files to the AI API. See [CONTRIBUTING.md](CONTRIBUTING.md) for a reference integration pattern.

### What the report looks like

```
# Vulnerability Scan Report
Generated: 2025-05-13T10:32:00Z
Project: Next.js + Prisma + OpenAI

## Summary
| Severity    | Count |
|-------------|-------|
| Critical    | 4     |
| High        | 11    |
| Medium      | 6     |
| Low         | 2     |
| Pass        | 198   |
| N/A         | 25    |

## Findings

### [6.1] Prompt injection - user input in system prompt
- **Severity:** Critical
- **File:** `app/api/chat/route.ts`
- **Line:** 34
- **Snippet:**
  ```ts
  const systemPrompt = `You are a helpful assistant. User context: ${req.body.userBio}`
  ```
- **Remediation:** Move user-supplied content to the `user` message role, never the `system` role.
  Sanitize for prompt control characters before passing to the model.
```

---

## AI-specific vulnerabilities (category 6 highlights)

These are patterns that show up most often in AI-generated LLM integration code:

| # | Vulnerability | Why AI coders get this wrong |
|---|---------------|------------------------------|
| 6.1 | Prompt injection | User input gets interpolated directly into system prompts to make demos work quickly |
| 6.2 | Indirect prompt injection | Agentic code reads emails, files, or web content without sanitizing what comes back |
| 6.4 | LLM used for auth decisions | AI assistants suggest asking the model whether a user has permission, with no deterministic fallback |
| 6.7 | API key in frontend | Scaffolding places keys in `NEXT_PUBLIC_` env vars so the frontend can call the API directly |
| 6.9 | User-controlled model parameter | Admin panels expose model selection without validating against an allowlist |
| 6.17 | Agentic shell or filesystem access without sandbox | Tool-use scaffolding hands agents `child_process` or `fs` access without restrictions |
| 6.20-6.21 | Vector DB with no tenant isolation | RAG scaffolding uses a single shared namespace for all users |

---

## Related references

This checklist covers how to detect the patterns that the following frameworks define at the conceptual level:

- [OWASP Top 10 for LLM Applications](https://owasp.org/www-project-top-10-for-large-language-model-applications/) - the most widely cited LLM security taxonomy
- [MITRE ATLAS](https://atlas.mitre.org/) - adversarial threat landscape for AI systems, grounded in observed attacks
- [NIST AI Risk Management Framework](https://airc.nist.gov/) - governance and risk framing for AI systems

The goal of this checklist is to give you the "how to actually check your code" layer that those references do not provide.

---

## Contributing

The checklist stays useful when the community keeps it current as AI coding patterns change. See [CONTRIBUTING.md](CONTRIBUTING.md) for the item schema, PR requirements, and how to add new categories.

---

## License

Apache 2.0. Free to use, adapt, and redistribute. See [LICENSE](LICENSE).
