# Pattern: Secrets and Configuration

Triggers when generating or editing: `.env` files, framework config (`next.config`, `vite.config`, `settings.py`), Dockerfiles, `docker-compose`, deployment manifests, infrastructure-as-code templates, CORS settings, cookie settings, security headers, and cloud storage or IAM configuration.

**Primary OWASP categories:** A02 (Security Misconfiguration), A04 (Cryptographic Failures).

## The core principle

> Secure by default, explicit when relaxed, and no secret ever lives in something that gets copied.

Source code, container images, client bundles, logs, and CI output are all copied widely. A secret placed in any of them is already disclosed.

## Checklist

### 🔴 Critical

- [ ] **No secrets in source code, commits, images, or example files.** Use environment variables or a secret manager. `.env` files are listed in `.gitignore`. Placeholders in `.env.example` are obviously fake.
- [ ] **Client-exposed variables never hold secrets.** Anything prefixed `NEXT_PUBLIC_`, `VITE_`, `EXPO_PUBLIC_`, or `REACT_APP_` ships to every browser or device.
- [ ] **No default, shared, or blank credentials** for databases, admin users, dashboards, or message brokers.
- [ ] **Debug mode, verbose errors, and development endpoints are off in production.**
- [ ] **TLS certificate verification is never disabled** (`rejectUnauthorized: false`, `verify=False`, `NODE_TLS_REJECT_UNAUTHORIZED=0`, `InsecureSkipVerify: true`).
- [ ] **CORS does not combine credentials with a wildcard or a reflected origin.**
- [ ] **Storage buckets, databases, and admin interfaces are private by default.** No public ACLs, no `0.0.0.0/0` on database or SSH ports.

### 🟠 High

- [ ] **Security headers are set:** `Content-Security-Policy`, `Strict-Transport-Security`, `X-Content-Type-Options: nosniff`, `frame-ancestors` (or `X-Frame-Options`), and `Referrer-Policy`.
- [ ] **Session cookies set `HttpOnly`, `Secure`, and `SameSite`**, with the narrowest workable `Path` and `Domain`.
- [ ] **Configuration is validated at startup and the process fails fast** when a required value is missing or malformed.
- [ ] **Containers run as a non-root user**, use a minimal base image pinned by digest, and use multi-stage builds so build tools and secrets do not reach the final image.
- [ ] **Secrets are not passed through `ARG` or `ENV` in a Dockerfile.** They persist in image layers and history. Use build secrets or runtime injection.
- [ ] **`.dockerignore` excludes `.env`, `.git`, and local credentials.**
- [ ] **Cloud and CI identities follow least privilege.** No wildcard actions or resources in IAM policies.
- [ ] **Each environment has its own secrets.** Production credentials are not reused in staging, preview, or local setups.

### 🟡 Medium

- [ ] **Unused features, sample apps, default pages, and directory listings are removed or disabled.**
- [ ] **Production source maps are not publicly served** unless that is a deliberate decision.
- [ ] **Secrets have an owner and a rotation procedure.**
- [ ] **Preview and staging deployments require authentication** when they use real data.

### 🟢 Low

- [ ] **A `security.txt` file and a documented disclosure contact exist.**
- [ ] **Configuration differences between environments are documented.**

## If a secret was committed

Deleting the line is not enough. The value remains in history, forks, clones, and CI caches. Revoke and rotate it first, then clean up the history if needed.

## Example

Fail-fast configuration with a clear split between server and client values:

```ts
import { z } from "zod";

const ServerEnv = z.object({
  DATABASE_URL: z.string().url(),
  SESSION_SECRET: z.string().min(32),
  STRIPE_SECRET_KEY: z.string().startsWith("sk_"),
});

// Throws at startup instead of failing later in a request
export const env = ServerEnv.parse(process.env);
```

Container baseline:

```dockerfile
FROM node:22-slim@sha256:<digest> AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:22-slim@sha256:<digest>
WORKDIR /app
ENV NODE_ENV=production
COPY --from=build /app/dist ./dist
COPY --from=build /app/node_modules ./node_modules
USER node
CMD ["node", "dist/server.js"]
```

Replace `<digest>` with the digest of the image you have verified.

## When to consult other patterns

- Key generation, hashing, and encryption choices → `crypto.md`
- CI/CD pipelines, base images, and lockfiles → `dependencies.md`
- Per-route CORS, cookies, and session behavior → `api-endpoints.md`
- Database roles and row-level security → `data-access.md`
- What may be written to logs → `errors-logging.md`
