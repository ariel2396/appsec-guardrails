# appsec-guardrails

Security guardrails for AI-assisted code generation, mapped to the OWASP Top 10:2025 and packaged as a Claude skill.

> **Not affiliated with OWASP.** This is an independent open-source project based on the publicly available OWASP Top 10:2025. "OWASP" is a registered trademark of the OWASP Foundation, Inc. This project is not endorsed by, sponsored by, or affiliated with the OWASP Foundation.

## Why

Most vulnerabilities are introduced during routine coding, not during dedicated security work. An endpoint is written without an ownership check, a query is built with string interpolation, a package name is taken from memory. When a coding agent writes the code, the same mistakes arrive faster.

This skill makes the agent consult a short, context-specific checklist at the moment it generates or edits security-relevant code, without the developer having to ask for it.

## What it covers

| Context | Pattern file | OWASP categories |
|---|---|---|
| API routes, resolvers, server actions, webhooks | `patterns/api-endpoints.md` | A01, A05, A07, A10 |
| Queries, ORMs, row-level security | `patterns/data-access.md` | A01, A05 |
| Parsing, validation, uploads, prompt construction | `patterns/input-handling.md` | A05, A06 |
| Env files, Docker, CORS, headers, IaC | `patterns/secrets-config.md` | A02, A04 |
| Password hashing, JWT, tokens, encryption | `patterns/crypto.md` | A04, A07 |
| Error handling, logging, alerting | `patterns/errors-logging.md` | A09, A10 |
| Dependencies, lockfiles, CI/CD | `patterns/dependencies.md` | A03, A08 |

Checks are stack-agnostic. Examples use TypeScript, with notes for Next.js and Supabase where the framework changes the risk.

## Install

### Claude Code

Personal (all projects):

```bash
git clone https://github.com/ariel2396/appsec-guardrails ~/.claude/skills/appsec-guardrails
```

```powershell
git clone https://github.com/ariel2396/appsec-guardrails "$env:USERPROFILE\.claude\skills\appsec-guardrails"
```

Per project:

```bash
git clone https://github.com/ariel2396/appsec-guardrails .claude/skills/appsec-guardrails
```

### Claude apps

Download this repository as a ZIP and upload it as a custom skill in your Claude settings.

## How it works

The skill uses progressive disclosure:

1. `SKILL.md` is the entry point. It holds a dispatch table that maps coding contexts to pattern files.
2. `patterns/*.md` are read only when the matching context appears, which keeps the active context small.
3. `reference/owasp-top-10-2025.md` is read for full security reviews or when background on a category is needed.

```
appsec-guardrails/
├── SKILL.md
├── patterns/
│   ├── api-endpoints.md
│   ├── data-access.md
│   ├── input-handling.md
│   ├── secrets-config.md
│   ├── crypto.md
│   ├── errors-logging.md
│   └── dependencies.md
└── reference/
    └── owasp-top-10-2025.md
```

## Verify it works

After installing, try these prompts in a test project:

| Prompt | Expected behavior |
|---|---|
| "Add an endpoint to fetch a user's invoice by ID." | The query is scoped to the authenticated user (A01). |
| "Hash this password and store it." | Argon2id, scrypt, or bcrypt. Never MD5 or SHA-1 (A04). |
| "Add a `/admin/users` endpoint." | Authentication, role check, and rate limiting (A01, A07). |
| "Add search with sorting by any column." | Bound parameters and an allowlist for the sort column (A05). |
| "Add a library to parse this file format." | The package name is verified before it is added (A03). |
| "Wrap the permission check in a try/catch." | The check denies on error (A10). |
| "Review this codebase for security issues." | A report grouped by OWASP category with file paths and lines. |

If the generated code lacks these checks, the skill is not loaded. Confirm the folder location and that `SKILL.md` is at its root.

## Severity levels

- 🔴 **Critical**: exploitation leads to full compromise, data breach, or remote code execution. The skill refuses insecure variants by default.
- 🟠 **High**: direct path to a user-impacting vulnerability. Always implemented.
- 🟡 **Medium**: defense in depth. Implemented unless there is a documented reason not to.
- 🟢 **Low**: hardening. Recommended.

## Limitations

This skill is preventive. It does not replace:

- A penetration test or a professional security audit
- Static and dynamic analysis tools (Semgrep, CodeQL, Burp, ZAP)
- A threat model for your specific business context
- Compliance work (SOC 2, HIPAA, PCI DSS)

A language model can still skip or misapply a check. Review generated code that touches authentication, authorization, payments, or personal data.

## Contributing

Issues and pull requests are welcome. See [CONTRIBUTING.md](./CONTRIBUTING.md).

## License

[MIT](./LICENSE). See [NOTICE](./NOTICE) for trademark information.

## Author

Maintained by [Ariel Manriquez](https://github.com/ariel2396).
