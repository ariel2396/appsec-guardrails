# Pattern: API Endpoints

Triggers when generating or editing: HTTP route handlers, controllers, REST or GraphQL resolvers, RPC handlers, server actions, webhooks, middleware, and any function reachable from the network.

**Primary OWASP categories:** A01 (Broken Access Control), A05 (Injection), A07 (Authentication Failures), A10 (Mishandling of Exceptional Conditions).

## The core principle

> Every handler is public until the handler itself proves otherwise. Authenticate, then authorize the specific object and the specific action, on the server, on every request.

Hiding a button, checking a role in the client, or relying on an unguessable ID is not access control.

## Checklist

### 🔴 Critical

- [ ] **Authentication is verified server-side inside the handler or the data layer.** Deny by default. Do not rely on middleware or a route matcher as the only gate: path-matching mistakes and framework bugs (for example CVE-2025-29927 in Next.js middleware) turn a single gate into no gate.
- [ ] **Object-level authorization on every identifier that comes from the client.** For each ID in the path, query, or body, confirm the authenticated principal owns it or is allowed to act on it (IDOR / BOLA).
- [ ] **Function-level authorization on every privileged operation.** Role and permission checks run on the server for each admin, billing, export, and destructive action.
- [ ] **Identity and privilege come from the verified session, never from the request.** Ignore `userId`, `tenantId`, `role`, `isAdmin`, or `price` sent in the body, query string, headers, or unsigned cookies.
- [ ] **Server actions and RPC functions are treated as public POST endpoints.** Each one authenticates and authorizes on its own, even if the page that calls it is protected.
- [ ] **No mass assignment.** Pick the allowed fields explicitly. Never spread the request body into a create or update call.
- [ ] **Server-side requests to user-supplied URLs are restricted (SSRF).** Allowlist schemes and hosts, block loopback, private, link-local, and cloud metadata ranges (`169.254.169.254`), validate the resolved IP, and re-validate after every redirect.

### 🟠 High

- [ ] **Rate limiting on authentication, password reset, OTP, signup, and expensive endpoints.** Limit per account and per IP.
- [ ] **State-changing operations never use GET.** With cookie-based sessions, add CSRF protection: `SameSite` cookies plus a token or an `Origin` check.
- [ ] **Responses are shaped explicitly.** Return a DTO or a selected set of fields. Never return the raw database row (password hashes, tokens, internal flags).
- [ ] **Sessions are handled safely.** Cookies are `HttpOnly`, `Secure`, and `SameSite`. Rotate the session ID on login and privilege change. Invalidate it server-side on logout.
- [ ] **Authentication responses do not reveal whether an account exists.** Same message and comparable timing for "unknown user" and "wrong password".
- [ ] **Webhooks verify the signature over the raw body** with a timing-safe comparison, and reject stale timestamps or replayed event IDs.
- [ ] **List endpoints enforce pagination with a maximum page size.**
- [ ] **A failure inside an authentication or authorization check denies the request.** See `errors-logging.md`.

### 🟡 Medium

- [ ] **CORS uses an explicit origin allowlist.** No wildcard and no reflected `Origin` together with credentials.
- [ ] **Request body size limits and timeouts are set.**
- [ ] **Sensitive multi-step flows are enforced server-side.** The server tracks the step; the client cannot skip to the end.
- [ ] **Idempotency keys on payments and other non-repeatable operations.**
- [ ] **Return 404 instead of 403 for resources the caller does not own** when the existence of the resource is itself sensitive.

### 🟢 Low

- [ ] **Framework and server version headers are removed** (`X-Powered-By`, `Server`).
- [ ] **API responses set `Cache-Control: no-store` when they carry personal or session data.**

## Example

Insecure: authenticated, but any user can read any invoice.

```ts
export async function GET(req: Request, { params }: { params: { id: string } }) {
  const session = await getSession(req);
  if (!session) return new Response("Unauthorized", { status: 401 });

  const invoice = await db.invoice.findUnique({ where: { id: params.id } });
  return Response.json(invoice);
}
```

Secure: ownership is part of the query, and the response is shaped.

```ts
export async function GET(req: Request, { params }: { params: { id: string } }) {
  const session = await getSession(req);
  if (!session) return new Response("Unauthorized", { status: 401 });

  const invoice = await db.invoice.findFirst({
    where: { id: params.id, userId: session.userId },
    select: { id: true, number: true, total: true, status: true, issuedAt: true },
  });
  if (!invoice) return new Response("Not found", { status: 404 });

  return Response.json(invoice);
}
```

## Stack notes

- **Next.js:** Server Actions and Route Handlers are network-reachable regardless of which page imports them. Check the session in each one. Keep privileged code in files marked `server-only`.
- **GraphQL:** authorize in resolvers or a shared data layer, not only at the gateway. Limit query depth and complexity. Disable introspection in production when the schema is private.
- **Supabase / PostgREST:** every table and RPC in an exposed schema is an endpoint. Authorization lives in RLS policies. See `data-access.md`.

## When to consult other patterns

- The handler reads or writes a database → `data-access.md`
- The handler parses a body, query, headers, or files → `input-handling.md`
- Login, signup, tokens, password reset → `crypto.md`
- Error responses and audit logging → `errors-logging.md`
- CORS, cookies, and security headers configuration → `secrets-config.md`
