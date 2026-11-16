# Pattern: Cryptography and Credentials

Triggers when generating or editing: password hashing, login and signup flows, JWT signing or verification, session and API token generation, password reset and email verification tokens, encryption of data at rest, signatures and HMACs, and TLS configuration.

**Primary OWASP categories:** A04 (Cryptographic Failures), A07 (Authentication Failures).

## The core principle

> Use vetted primitives through vetted libraries, with keys that live outside the code.

Do not design algorithms, modes, or token formats. Choose the high-level API of a maintained library and keep its defaults unless there is a documented reason to change them.

## Checklist

### 🔴 Critical

- [ ] **Passwords are hashed with a password hashing function: Argon2id, scrypt, or bcrypt.** Never MD5, SHA-1, plain SHA-256, or reversible encryption. Never stored or logged in plaintext.
- [ ] **Security tokens come from a cryptographically secure generator** (`crypto.randomBytes`, `crypto.randomUUID`, `secrets.token_urlsafe`, `crypto/rand`). Never `Math.random`, timestamps, or sequential IDs. Use at least 128 bits of entropy.
- [ ] **JWTs are verified, not just decoded.** Pin the accepted algorithms, reject `none`, and validate `exp`, `iss`, and `aud`. `decode()` is not verification.
- [ ] **No hardcoded keys, salts, IVs, or signing secrets.** Keys come from the environment or a key management service.
- [ ] **Symmetric encryption uses an authenticated mode** (AES-256-GCM or ChaCha20-Poly1305) with a unique nonce per message. No ECB. No unauthenticated CBC.
- [ ] **TLS verification is not disabled, and secrets are not sent over plaintext channels.**
- [ ] **No custom cryptographic constructions.** No hand-rolled ciphers, signature schemes, or "encode then XOR" obfuscation.

### 🟠 High

- [ ] **Secrets, tokens, and signatures are compared in constant time** (`crypto.timingSafeEqual`, `hmac.compare_digest`).
- [ ] **Reset, verification, and API tokens are stored as hashes.** A fast hash such as SHA-256 is acceptable for high-entropy random tokens. They are single-use and expire.
- [ ] **Access tokens are short-lived.** Refresh tokens rotate on use, and reuse of a rotated token revokes the session family.
- [ ] **The password policy favors length.** Allow at least 64 characters, set a sensible minimum, check against known-breached passwords, and avoid forced composition rules.
- [ ] **Multi-factor authentication is supported for privileged accounts,** and recovery flows are not weaker than the login flow.
- [ ] **Login, reset, and OTP endpoints are rate limited** with lockout or backoff. See `api-endpoints.md`.
- [ ] **TLS 1.2 or later is enforced** with HSTS on public sites.
- [ ] **bcrypt's 72-byte input limit is handled.** Reject longer passwords or use Argon2id.

### 🟡 Medium

- [ ] **Each key has one purpose.** Signing keys, encryption keys, and API secrets are separate.
- [ ] **Keys can be rotated.** Tokens and ciphertexts carry a key identifier.
- [ ] **JWT payloads hold no sensitive data.** They are encoded, not encrypted.
- [ ] **Sensitive data at rest uses envelope encryption** with the master key in a KMS.

### 🟢 Low

- [ ] **Hash and KDF parameters are stored alongside the hash** so they can be upgraded on next login.

## Baseline parameters

These follow the OWASP Password Storage Cheat Sheet at the time of writing. Check the current version before relying on them.

| Algorithm | Minimum configuration |
|---|---|
| Argon2id | 19 MiB memory, 2 iterations, parallelism 1 |
| scrypt | N = 2^17, r = 8, p = 1 |
| bcrypt | Work factor 10 or higher |
| PBKDF2-HMAC-SHA256 | 600,000 iterations (use only when FIPS compliance requires it) |

## Example

Insecure: decoding without verification, and a guessable reset token.

```ts
const payload = jwt.decode(token);
const resetToken = Math.random().toString(36).slice(2);
```

Secure: pinned algorithm with validated claims, and a random token stored as a hash.

```ts
const payload = jwt.verify(token, env.JWT_PUBLIC_KEY, {
  algorithms: ["RS256"],
  issuer: "https://auth.example.com",
  audience: "api",
});

const resetToken = crypto.randomBytes(32).toString("base64url");
const tokenHash = crypto.createHash("sha256").update(resetToken).digest("hex");
await db.passwordReset.create({
  data: { userId, tokenHash, expiresAt: new Date(Date.now() + 30 * 60_000) },
});
// Send resetToken to the user. Store only tokenHash.
```

## When to consult other patterns

- Where keys and secrets are stored and loaded → `secrets-config.md`
- Login and reset endpoints, sessions, and cookies → `api-endpoints.md`
- Logging authentication events without leaking credentials → `errors-logging.md`
- Choosing or upgrading a cryptography library → `dependencies.md`
