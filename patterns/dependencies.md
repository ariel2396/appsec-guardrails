# Pattern: Dependencies and Build Pipeline

Triggers when generating or editing: `package.json`, `requirements.txt`, `pyproject.toml`, `go.mod`, `Gemfile`, `Cargo.toml`, `composer.json`, lockfiles, install commands, CI/CD workflows, build and release scripts, Docker base images, and `<script>` tags that load third-party code.

**Primary OWASP categories:** A03 (Software Supply Chain Failures), A08 (Software or Data Integrity Failures).

## The core principle

> Every dependency and every build step is code you run with your own privileges. Know what it is, pin what you verified, and give it as little access as possible.

## Checklist

### 🔴 Critical

- [ ] **Every package added is verified to exist and to be the intended one.** Language models can produce plausible package names that do not exist, and attackers register those names (slopsquatting) as well as misspellings of popular packages (typosquatting). Before adding a dependency, confirm the exact name in the official registry, its source repository, maintainers, and release history. Do not add a package from memory alone.
- [ ] **The lockfile is committed, and CI installs from it without modification** (`npm ci`, `pnpm install --frozen-lockfile`, `yarn install --immutable`, `pip install --require-hashes`, `cargo build --locked`).
- [ ] **No known critical or high vulnerabilities are introduced.** Run the ecosystem's audit tool (`npm audit`, `pip-audit`, `osv-scanner`, `cargo audit`) when adding or upgrading.
- [ ] **Build and deploy scripts do not pipe remote content into a shell** (`curl ... | bash`) without pinning a version and verifying a checksum or signature.
- [ ] **CI secrets are not exposed to untrusted code.** Workflows triggered by forks or by `pull_request_target` do not check out and run contributor code with access to secrets or write tokens.

### 🟠 High

- [ ] **Third-party CI actions are pinned to a full commit SHA,** not a mutable tag. Tags can be moved to malicious code, as happened with `tj-actions/changed-files` (CVE-2025-30066).
- [ ] **CI tokens have minimal permissions.** Declare `permissions:` explicitly and start from read-only.
- [ ] **Applications pin exact dependency versions.** Ranges are acceptable in libraries, not in deployed apps.
- [ ] **Install-time scripts are reviewed or disabled** (`--ignore-scripts`) for packages that do not need them.
- [ ] **Container base images are pinned by digest** and come from a trusted publisher.
- [ ] **Private package names are protected from dependency confusion.** Use scoped names and configure the registry per scope.
- [ ] **Downloaded binaries, models, and artifacts are verified** against a checksum or signature before use.
- [ ] **Third-party scripts loaded in the browser use Subresource Integrity** or are self-hosted.
- [ ] **Release artifacts are built in CI from a tagged commit,** not from a developer machine, and carry provenance where the ecosystem supports it.
- [ ] **Automated update tooling is enabled** (Dependabot, Renovate), with a short delay before adopting brand-new releases.

### 🟡 Medium

- [ ] **Unused dependencies are removed,** and unmaintained ones are replaced.
- [ ] **A software bill of materials is generated** (CycloneDX or SPDX) for each release.
- [ ] **Branch protection requires review** for changes to workflows, lockfiles, and build scripts.
- [ ] **Developer tooling is in scope:** editor extensions, MCP servers, and agent plugins run with the developer's credentials.

### 🟢 Low

- [ ] **Dependency licenses are compatible with the project's license.**
- [ ] **A documented process exists for responding to a compromised dependency.**

## Example

Insecure workflow: mutable tag, broad default token, secrets available to fork code.

```yaml
on: pull_request_target
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          ref: ${{ github.event.pull_request.head.sha }}
      - run: npm install && npm test
        env:
          DEPLOY_KEY: ${{ secrets.DEPLOY_KEY }}
```

Secure workflow: untrusted trigger without secrets, read-only token, pinned action, frozen install.

```yaml
on: pull_request
permissions:
  contents: read
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@<full-commit-sha> # v4.x.x
      - run: npm ci --ignore-scripts
      - run: npm test
```

Replace `<full-commit-sha>` with the commit of the release you reviewed.

## Before adding a dependency

1. Is it needed, or does the standard library or an existing dependency cover it?
2. Does the exact name exist in the official registry, and does it link to the expected repository?
3. Is it maintained, and how many transitive dependencies does it add?
4. Does it run install scripts or request network or filesystem access it should not need?

## When to consult other patterns

- Secrets used by pipelines and images → `secrets-config.md`
- Deserialization of untrusted data → `input-handling.md`
- Choosing cryptography libraries → `crypto.md`
- Dockerfile hardening → `secrets-config.md`
