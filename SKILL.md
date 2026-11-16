---
name: appsec-guardrails
description: Apply OWASP Top 10:2025 security guardrails when generating, editing, or reviewing code. ALWAYS use when writing or modifying API routes, endpoints, controllers, GraphQL resolvers, server actions, auth flows (login, signup, logout, session, JWT), password hashing, encryption, database queries, ORM operations, raw SQL, form processing, file uploads, env vars, config files (Dockerfile, docker-compose, CI/CD, security headers, CORS), error handlers, logging setup, and any change to package.json, requirements.txt, go.mod, Gemfile, Cargo.toml, or composer.json. Trigger even when the user does not say "security" — most vulnerabilities are introduced during routine codegen, not during dedicated security work. Also use when the user asks for a security review, vulnerability check, audit, OWASP compliance, threat model, or "is this code safe". Stack-agnostic across major languages and frameworks.
license: MIT
---

# OWASP Guardrails for Code Generation

Preventive security guardrails for code generation, mapped to the [OWASP Top 10:2025](https://owasp.org/Top10/2025/). This skill exists because most vulnerabilities are introduced during routine coding — not during dedicated "security work". Applying the right checklist at the moment of generation prevents most issues before they reach review.

## How to use this skill

When you (Claude) are about to generate or modify code, identify which **codegen context** applies and consult the matching pattern file. Patterns are short — read the relevant ones, apply the checks, then proceed with generation.

### Dispatch table

| If you are generating or editing... | Read this pattern | Primary OWASP categories |
|---|---|---|
| API routes, endpoints, controllers, route handlers, GraphQL resolvers, server actions, RPC handlers | `patterns/api-endpoints.md` | A01, A05, A07, A10 |
| Database queries, ORM calls, raw SQL, NoSQL filters, search queries, anything that scopes data to a user | `patterns/data-access.md` | A01, A05 |
| Input parsing, form handlers, request body validation, file uploads, URL parameters, headers parsing | `patterns/input-handling.md` | A05, A06 |
| `.env` files, config files, Dockerfiles, deployment manifests, CORS settings, security headers, IaC templates | `patterns/secrets-config.md` | A02, A04 |
| Password hashing, JWT signing/verification, encryption, random token generation, TLS configuration | `patterns/crypto.md` | A04, A07 |
| Error handlers, try/catch blocks, exception classes, logging setup, audit trails, monitoring | `patterns/errors-logging.md` | A09, A10 |
| `package.json`, `requirements.txt`, `go.mod`, `Gemfile`, `Cargo.toml`, lockfiles, CI/CD pipelines, build scripts | `patterns/dependencies.md` | A03, A08 |

### When the user asks for a security review

Read **all** pattern files plus `reference/owasp-top-10-2025.md`, then walk the codebase systematically. Produce a report grouped by OWASP category, with file paths and line numbers for each finding.

### Decision rules

1. **Multiple patterns can apply at once.** A login endpoint touches `api-endpoints.md` (it's a route), `crypto.md` (password hashing), `errors-logging.md` (failed-attempt logging), and `data-access.md` (user lookup). Read all that apply.

2. **Trigger silently.** Do not announce "I'm consulting the OWASP skill". Apply the checks and produce secure code by default. Only surface security reasoning when explaining a non-obvious choice or when the user explicitly asks.

3. **Default to refusing insecure patterns.** If the user asks for code that violates a Critical-level check (e.g. `eval()` on user input, storing plaintext passwords, disabling TLS verification), explain the risk and propose the secure alternative. Do not just generate the insecure version with a warning comment — Claude users frequently miss those.

4. **Stack-agnostic by default.** When examples are needed, prefer the language and framework already present in the project. If unknown, default to TypeScript/Node.js examples and note that the principle is universal.

## Quick reference: OWASP Top 10:2025

| # | Category | Common manifestation in code |
|---|---|---|
| A01 | Broken Access Control | Missing or scoped-incorrectly authorization checks; IDOR; SSRF (now consolidated here) |
| A02 | Security Misconfiguration | Default credentials, exposed admin panels, missing security headers, permissive CORS, debug mode in prod |
| A03 | Software Supply Chain Failures | Unpinned deps, unverified packages, compromised build pipelines, missing SBOMs |
| A04 | Cryptographic Failures | Weak hashes (MD5/SHA1), hardcoded keys, plaintext secrets at rest or in transit, weak randomness |
| A05 | Injection | SQLi, XSS, command injection, NoSQL injection, LDAP injection, template injection, prompt injection |
| A06 | Insecure Design | Missing rate limits, no threat model, business logic flaws, insufficient defense-in-depth |
| A07 | Authentication Failures | Weak password policies, no MFA, session fixation, predictable tokens, credential stuffing exposure |
| A08 | Software or Data Integrity Failures | Unsigned updates, insecure deserialization, untrusted CI/CD, missing checksum verification |
| A09 | Security Logging & Alerting Failures | No audit trail for auth events, logs without alerts, sensitive data logged, no log integrity |
| A10 | Mishandling of Exceptional Conditions (NEW) | Failing open, swallowed exceptions, error messages leaking internals, unhandled edge cases |

For full details, CWE mappings, and remediation guidance, see `reference/owasp-top-10-2025.md`.

## What changed from 2021

- **NEW: A03 Software Supply Chain Failures** — expanded from "Vulnerable & Outdated Components"
- **NEW: A10 Mishandling of Exceptional Conditions** — error handling as a first-class category
- **CONSOLIDATED:** SSRF (was A10 in 2021) is now part of A01 Broken Access Control
- **A02 Security Misconfiguration** moved up from #5 to #2 — config-driven systems made this more prevalent
- **A07** renamed from "Identification and Authentication Failures" to "Authentication Failures"
- **A09** renamed from "Logging and Monitoring Failures" to "Logging & Alerting Failures" — emphasizing actionable alerts

## Severity levels used in patterns

Each check in the pattern files is labeled:

- **🔴 Critical** — exploitation leads to full compromise, data breach, or RCE. Refuse insecure variants by default.
- **🟠 High** — direct path to user-impacting vulnerability. Always implement.
- **🟡 Medium** — defense-in-depth. Implement unless there's a documented reason not to.
- **🟢 Low** — hardening. Recommend but don't block on it.

## Limitations

This skill is **preventive during codegen**, not a substitute for:
- A full penetration test or security audit
- Static analysis tools (Semgrep, CodeQL, Snyk)
- Dynamic analysis tools (Burp, ZAP, DAST scanners)
- A threat model for your specific business context

If the user is preparing for production launch, recommend running automated SAST/DAST in addition to applying this skill during development.
