# Security Policy

## Scope of this repository

This repository is a defensive reference. It documents vulnerability patterns so developers can find and fix them in their own codebases. The checklist and scan prompt are intended for use on applications you own or have explicit authorization to audit.

Do not use the techniques described here against systems you do not have permission to test.

## Reporting a security issue in this repo

If you find a security problem with the repository itself (for example, a GitHub Actions workflow that leaks secrets, or a CI dependency with a known CVE), please report it privately rather than opening a public issue.

Open a [GitHub Security Advisory](https://github.com/a-leks/genai-app-security-checklist/security/advisories/new) on this repository.

Please include:
- A description of the issue
- How it could be exploited
- A suggested fix if you have one

We will acknowledge reports within 5 business days and aim to resolve them within 14 days.

## Reporting errors in the checklist

If a checklist item is factually wrong, has an incorrect severity rating, or describes a pattern that is outdated, open a regular GitHub issue or pull request. These are content corrections, not security disclosures, so the advisory process is not needed.

## What this repo is not

This is not a CVE database and does not track vulnerabilities in specific libraries or packages. The checklist covers patterns in application code that AI assistants commonly produce. For dependency-level CVEs, use `npm audit`, `pip-audit`, GitHub Dependabot, or the advisory database relevant to your ecosystem.

## Responsible use

The `prompt.md` file is designed to help developers audit their own applications. Running it against a system without authorization is outside the intended use of this project. If you discover a real vulnerability using this checklist, follow the affected organization's responsible disclosure process before publishing your findings.
