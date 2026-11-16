# Pattern: Errors and Logging

Triggers when generating or editing: `try/catch` blocks, error and exception classes, global error handlers, error responses, logging setup, audit trails, monitoring and alerting hooks, retries, timeouts, and cleanup code.

**Primary OWASP categories:** A09 (Security Logging & Alerting Failures), A10 (Mishandling of Exceptional Conditions).

## The core principle

> Fail closed, tell the client little, tell the logs enough, and make sure someone is alerted.

When something unexpected happens, the safe outcome is denial. The client receives a generic message. The server records enough context to investigate, without recording secrets.

## Checklist

### 🔴 Critical

- [ ] **Security checks fail closed.** If authentication, authorization, validation, signature verification, or a policy lookup throws or times out, the request is denied. An exception never falls through to the success path.
- [ ] **Clients never receive stack traces, SQL errors, file paths, or internal identifiers.** Return a generic message and a correlation ID.
- [ ] **Secrets and credentials are never logged.** No passwords, tokens, API keys, session IDs, authorization headers, full payment data, or complete request bodies of authentication endpoints.
- [ ] **No empty or swallow-all `catch` around security-relevant operations.** Handle the error, rethrow it, or deny.
- [ ] **A failed step rolls back the whole operation.** No partial state after an error in a multi-step write, payment, or permission change.

### 🟠 High

- [ ] **Security events are logged with context:** login success and failure, logout, password and MFA changes, access-control denials, validation failures on sensitive endpoints, privilege changes, and admin actions. Include user ID, timestamp, source IP, and request ID.
- [ ] **Logs produce alerts, not just records.** Thresholds exist for repeated login failures, spikes in denials or 5xx responses, and access to sensitive operations.
- [ ] **Log entries are structured, or user-controlled values are neutralized.** CR/LF in input cannot forge log lines (log injection).
- [ ] **A global handler catches uncaught exceptions and unhandled promise rejections,** logs them, and returns a safe response.
- [ ] **Return values and error codes of security-relevant calls are checked.**
- [ ] **Resources are released on every path** (`finally`, `defer`, context managers): connections, file handles, locks.
- [ ] **External calls have timeouts and a defined behavior on failure.** Retries are bounded and use backoff.
- [ ] **Authentication errors are uniform.** Same message for unknown account and wrong password.

### 🟡 Medium

- [ ] **Logs go to a central, append-only destination with a defined retention period,** and application credentials cannot delete them.
- [ ] **Personal data in logs is minimized or masked.**
- [ ] **Error handling paths are rate limited or cheap,** so that triggering errors repeatedly does not exhaust resources.
- [ ] **Custom error pages replace framework defaults.**
- [ ] **Missing, extra, or malformed parameters are handled explicitly** instead of falling back to a permissive default.

### 🟢 Low

- [ ] **Log levels are consistent,** and debug logging is off in production.
- [ ] **Error types distinguish expected failures from bugs,** so alerts stay meaningful.

## Example

Insecure: the check fails open and the error leaks internals.

```ts
let allowed = true;
try {
  allowed = await policy.canAccess(user, resource);
} catch (e) {
  console.log(e);
}
if (allowed) return serve(resource);

// elsewhere
return Response.json({ error: err.stack }, { status: 500 });
```

Secure: default deny, structured log without secrets, generic response.

```ts
let allowed = false;
try {
  allowed = await policy.canAccess(user, resource);
} catch (err) {
  logger.error({ event: "authz_check_failed", userId: user.id, resourceId: resource.id, requestId, err });
}

if (!allowed) {
  logger.warn({ event: "access_denied", userId: user.id, resourceId: resource.id, requestId });
  return Response.json({ error: "Forbidden", requestId }, { status: 403 });
}
return serve(resource);
```

## When to consult other patterns

- The handler producing the error response → `api-endpoints.md`
- Validation failures and what to return for them → `input-handling.md`
- Transactions and partial writes → `data-access.md`
- Logging around login, tokens, and resets → `crypto.md`
- Log destinations, retention, and access configuration → `secrets-config.md`
