# Contributing

Contributions that make the checks more accurate, more specific, or easier for a coding agent to apply are welcome.

## What helps most

- **Corrections.** A check that is wrong, outdated, or misleading. Include a source.
- **Missed cases.** A prompt where the skill was loaded and the generated code was still insecure. Include the prompt, the stack, and the output.
- **Stack notes.** Framework-specific behavior that changes the risk (for example, how a framework exposes server functions).
- **False positives.** A check that blocks a legitimate pattern.

## Guidelines for pattern files

1. **One check, one line of intent.** Start with the rule in bold, then the detail needed to apply it.
2. **Checks must be verifiable in code.** "Be careful with input" is not a check. "Reject unknown fields in the request schema" is.
3. **Assign a severity** using the definitions in `SKILL.md`. Critical is reserved for issues that lead to compromise, data breach, or code execution.
4. **Keep files short.** Pattern files are loaded into an agent's context. Prefer a precise sentence over a paragraph.
5. **Examples show the insecure and the secure version**, in TypeScript unless the issue is specific to another language.
6. **Cite sources for parameters and version-specific claims** (hash parameters, CVE identifiers, framework behavior).
7. **Do not copy text from OWASP or other sources.** Summarize in your own words and link to the original.

## Changing the dispatch table

If you add a pattern file or change which contexts trigger it, update:

- The dispatch table in `SKILL.md`
- The coverage table and directory tree in `README.md`
- The summary table in `reference/owasp-top-10-2025.md`

The `description` field in `SKILL.md` must stay under 1,024 characters.

## Pull requests

- Keep each pull request focused on one change.
- Describe how you tested it: the prompt used and the difference in output with and without the change.

## Reporting a security issue in this repository

Open an issue. This repository contains documentation only, so there is no private disclosure process.
