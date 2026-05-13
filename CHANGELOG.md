# Changelog

All notable changes to this checklist will be documented here.

Format follows [Keep a Changelog](https://keepachangelog.com/en/1.0.0/).

---

## [Unreleased]

Nothing yet.

---

## [1.0.0] - 2025-05-13

### Added

- `vulnerabilities.md` with 258 items across 17 categories
- `prompt.md` with 1:1 rule coverage of all 258 items
- Categories: Authentication & Session Management, Authorization & Access Control, Input Validation & Injection, API Security, Secrets & Configuration, AI-Specific Vulnerabilities (LLM Integration), Database & Data Layer, Infrastructure & Deployment, Frontend & Client-Side, Email / Payments / Third-Party Integrations, Password Reset & Account Recovery, Logging / Monitoring / Incident Response, Performance Vulnerabilities, Cryptography, Supply Chain & Dependency Security, Business Logic, Emerging Patterns (provisional)
- Category 6 includes 8 agentic security items: MCP tool poisoning (6.26), agent memory poisoning (6.27), encoding-based jailbreaks (6.28), model extraction via probing (6.29), cross-agent prompt injection (6.30), insecure agent handoff (6.31), unvalidated LLM JSON output (6.32), conversation history isolation (6.33)
- Category 17 covers 4 provisional patterns: computer use agent screen capture (17.1), AI code review manipulation (17.2), stateful agent session bleed (17.3), multimodal prompt injection via image metadata (17.4)
- GitHub issue templates for new vulnerability proposals and severity corrections
- Detection method tagging for every item: `[S]` static, `[R]` runtime, `[C]` config
- Severity rating for every item: Critical, High, Medium, Low
- Summary table with per-category severity breakdowns
- Detection tooling reference table (ripgrep, semgrep, trufflehog, gitleaks, ts-morph, npm audit)
- `README.md`, `CONTRIBUTING.md`, `SECURITY.md`
- Apache 2.0 license

---

## How to read this file

Each release notes what was added, changed, or removed. When contributing, add your changes under `[Unreleased]` before they are merged into a tagged release.

Version numbers follow semantic versioning adapted for a checklist:
- **Major** (1.x.x to 2.x.x): new category added, or schema changed in a breaking way
- **Minor** (1.0.x to 1.1.x): new items added to existing categories
- **Patch** (1.0.0 to 1.0.1): corrections, severity updates, wording fixes
