# Reference: OWASP Top 10:2025

Working reference for the ten categories, written for use during code generation and review. It summarizes each category in this project's own words and maps it to the pattern files.

This file is not the official text. The authoritative source, including the complete CWE mapping for each category, is <https://owasp.org/Top10/2025/>. The CWEs listed here are representative examples, not the full mapping.

The 2025 edition was announced in November 2025 and finalized in early 2026. It replaces the 2021 edition.

## Summary

| # | Category | 2021 position | Pattern files |
|---|---|---|---|
| A01 | Broken Access Control | A01 (now includes SSRF, formerly A10) | `api-endpoints.md`, `data-access.md` |
| A02 | Security Misconfiguration | A05 | `secrets-config.md` |
| A03 | Software Supply Chain Failures | A06, with a wider scope | `dependencies.md` |
| A04 | Cryptographic Failures | A02 | `crypto.md`, `secrets-config.md` |
| A05 | Injection | A03 | `input-handling.md`, `data-access.md`, `api-endpoints.md` |
| A06 | Insecure Design | A04 | `input-handling.md`, `api-endpoints.md` |
| A07 | Authentication Failures | A07, renamed | `crypto.md`, `api-endpoints.md` |
| A08 | Software or Data Integrity Failures | A08 | `dependencies.md`, `input-handling.md` |
| A09 | Security Logging & Alerting Failures | A09, renamed | `errors-logging.md` |
| A10 | Mishandling of Exceptional Conditions | New | `errors-logging.md`, `api-endpoints.md` |

---

## A01 Broken Access Control

**What it is.** A user can act outside their intended permissions: read or modify other users' data, call privileged functions, or make the server reach resources on their behalf.

**How it shows up in code.**
- Handlers that authenticate but never check ownership of the requested object (IDOR)
- Admin or internal functions protected only in the UI
- Trusting `userId`, `role`, or `tenantId` from the request
- Missing or permissive row-level security policies
- Server-side fetch of a user-supplied URL (SSRF)
- Path traversal to files outside the intended directory
- Missing CSRF protection on cookie-authenticated state changes

**Representative CWEs.** CWE-22, CWE-200, CWE-284, CWE-285, CWE-352, CWE-639, CWE-862, CWE-863, CWE-918.

**Primary remediation.** Deny by default. Enforce authorization on the server for every object and every function, in a shared and tested mechanism. Scope queries to the session principal.

## A02 Security Misconfiguration

**What it is.** The system is insecure because of how it is configured, not because of a flaw in the code's logic.

**How it shows up in code.**
- Debug mode or verbose errors enabled in production
- Default or blank credentials
- Wildcard CORS with credentials
- Missing security headers and cookie flags
- Public storage buckets, open database ports, exposed admin panels
- Secrets placed in client-exposed environment variables
- XML parsers with external entities enabled

**Representative CWEs.** CWE-16, CWE-260, CWE-489, CWE-526, CWE-611, CWE-614, CWE-942, CWE-1004.

**Primary remediation.** Secure defaults, a repeatable hardening process, configuration validated at startup, and identical hardening across environments.

## A03 Software Supply Chain Failures

**What it is.** A compromise or breakdown in how software is built, distributed, or updated. It expands the 2021 "Vulnerable and Outdated Components" category to cover dependencies, build systems, and distribution infrastructure.

**How it shows up in code.**
- Dependencies added without verifying that the package exists and is the intended one
- No lockfile, or CI installs that ignore it
- Unpinned CI actions and base images
- Build scripts that execute remote content
- CI secrets reachable by untrusted pull requests
- Unmaintained or known-vulnerable components
- Developer tools and extensions with broad access

**Representative CWEs.** CWE-477, CWE-1104, CWE-1329, CWE-1395.

**Primary remediation.** Inventory dependencies (SBOM), pin and verify what is installed, minimize CI privileges, monitor advisories, and keep an update process that includes a response plan for compromised components.

## A04 Cryptographic Failures

**What it is.** Missing, weak, or misused cryptography that exposes sensitive data or lets an attacker forge or predict values.

**How it shows up in code.**
- Passwords hashed with MD5, SHA-1, or unsalted fast hashes
- Hardcoded keys, IVs, or signing secrets
- Tokens generated with a non-cryptographic random source
- ECB mode, reused nonces, unauthenticated encryption
- Sensitive data transmitted or stored in cleartext
- Disabled certificate validation

**Representative CWEs.** CWE-259, CWE-319, CWE-321, CWE-326, CWE-327, CWE-328, CWE-330, CWE-331, CWE-338, CWE-916.

**Primary remediation.** Classify data, encrypt sensitive data in transit and at rest with authenticated modes, use password hashing functions for passwords, and manage keys outside the codebase.

## A05 Injection

**What it is.** Untrusted data is interpreted as part of a command or query.

**How it shows up in code.**
- SQL built by concatenation or interpolation
- Shell commands built from user input
- Unescaped user content rendered as HTML (XSS)
- NoSQL operator injection, LDAP and XPath injection
- User input used as a template or evaluated as code
- Untrusted content mixed into LLM instructions (prompt injection)

**Representative CWEs.** CWE-20, CWE-74, CWE-77, CWE-78, CWE-79, CWE-89, CWE-90, CWE-94, CWE-643, CWE-917, CWE-1336.

**Primary remediation.** Keep data separate from commands: parameterized APIs, context-aware output encoding, strict input validation, and allowlists for identifiers.

## A06 Insecure Design

**What it is.** The weakness is in the design itself. A perfect implementation of a flawed design is still insecure.

**How it shows up in code.**
- No rate limiting on authentication or costly operations
- Business rules enforced only in the client
- Multi-step flows whose steps can be skipped or replayed
- Unrestricted file upload
- No tenant isolation model
- Trust boundaries that are implicit or undocumented

**Representative CWEs.** CWE-434, CWE-501, CWE-522, CWE-602, CWE-799, CWE-841.

**Primary remediation.** Threat modeling, secure design patterns, abuse-case tests, and server-side enforcement of business limits.

## A07 Authentication Failures

**What it is.** The application fails to confirm identity correctly or to manage sessions safely.

**How it shows up in code.**
- No protection against credential stuffing or brute force
- Weak or default passwords accepted
- Recovery flows weaker than login
- Session IDs not rotated on login or not invalidated on logout
- Tokens that never expire or are predictable
- Hardcoded credentials
- Missing MFA for privileged access

**Representative CWEs.** CWE-287, CWE-290, CWE-297, CWE-306, CWE-307, CWE-384, CWE-521, CWE-613, CWE-620, CWE-640, CWE-798.

**Primary remediation.** Use a proven authentication library or provider, support MFA, rate limit, check passwords against breach lists, and manage sessions on the server.

## A08 Software or Data Integrity Failures

**What it is.** Code or data is trusted without verifying its integrity. It operates at a lower level than A03: the focus is on trust boundaries and verification of artifacts and data.

**How it shows up in code.**
- Deserialization of untrusted data
- Updates or plugins applied without signature verification
- Third-party scripts loaded without integrity checks
- Mass assignment of object attributes from input
- CI/CD pipelines that accept unreviewed changes
- Signed data whose signature is never checked

**Representative CWEs.** CWE-345, CWE-353, CWE-426, CWE-494, CWE-502, CWE-829, CWE-915.

**Primary remediation.** Verify signatures and checksums, use safe serialization formats with schema validation, and require review for pipeline and configuration changes.

## A09 Security Logging & Alerting Failures

**What it is.** Attacks are not detected, or not acted on, because relevant events are not logged, logs are not monitored, or alerts do not exist. The 2025 name emphasizes alerting: a log nobody is notified about does not stop an incident.

**How it shows up in code.**
- No audit trail for login, access-control failures, or admin actions
- Logs without enough context to investigate
- Secrets or personal data written to logs
- User input written to logs without neutralization
- No alert thresholds, or alerts nobody receives

**Representative CWEs.** CWE-117, CWE-223, CWE-532, CWE-778.

**Primary remediation.** Log security-relevant events with context, protect log integrity, define alert thresholds with an owner, and keep an incident response plan.

## A10 Mishandling of Exceptional Conditions

**What it is.** New in 2025. The application does not prevent, detect, or respond correctly to abnormal situations, which leads to crashes, inconsistent state, information disclosure, or bypassed controls.

**How it shows up in code.**
- A security check that throws and falls through to the allow path (failing open)
- Swallowed exceptions and unchecked return values
- Error messages that disclose internals
- Partial writes when one step of a transaction fails
- Missing or extra parameters handled with permissive defaults
- Resources not released on error paths

**Representative CWEs.** CWE-209, CWE-234, CWE-235, CWE-248, CWE-252, CWE-274, CWE-280, CWE-476, CWE-550, CWE-636, CWE-703, CWE-754, CWE-755, CWE-756.

**Primary remediation.** Fail closed, handle errors where they occur with a global handler as a backstop, roll back completely on failure, and apply limits (rate, quota, timeout) to avoid reaching exceptional states in the first place.

---

## Related OWASP projects

The Top 10 covers web applications in general. For AI-integrated systems, see the separate OWASP lists for LLM applications and agentic applications at <https://genai.owasp.org/>. For a verification checklist rather than an awareness list, see the OWASP Application Security Verification Standard (ASVS).
