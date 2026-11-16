# Pattern: Input Handling

Triggers when generating or editing: request body parsing, query and URL parameters, form processing, file uploads, header and cookie parsing, JSON, XML, or YAML parsing, deserialization, HTML rendering of user content, redirects, template rendering, and prompt construction for LLM calls.

**Primary OWASP categories:** A05 (Injection), A06 (Insecure Design).

## The core principle

> Validate at the boundary, encode at the sink.

Everything from outside the trust boundary is validated against an explicit schema before it influences control flow or data. Everything written into an interpreter (HTML, SQL, shell, URL, template, prompt) is encoded or parameterized for that specific context.

## Checklist

### 🔴 Critical

- [ ] **All external input is validated against a schema before use.** Types, shapes, lengths, ranges, and allowed values, using Zod, Pydantic, JSON Schema, or an equivalent. Validation runs on the server even when the client also validates.
- [ ] **No dynamic code execution on user input.** No `eval`, `new Function`, `exec`, `setTimeout(string)`, or `vm` with untrusted code.
- [ ] **No shell commands built from user input.** Call the binary directly with an argument array (`execFile`, `spawn` without `shell: true`, `subprocess.run([...])`).
- [ ] **Output is encoded for its destination.** Rely on the framework's auto-escaping. `dangerouslySetInnerHTML`, `v-html`, `innerHTML`, and `| safe` receive only content passed through a maintained sanitizer.
- [ ] **File paths built from user input are confined to a base directory.** Resolve the final path and verify it still starts with the base. Reject `..`, absolute paths, and null bytes.
- [ ] **File uploads are validated by content, not by filename or `Content-Type`.** Enforce size limits, generate the stored filename, and store outside the web root or in object storage that does not execute files.
- [ ] **Untrusted data is never deserialized with formats that execute code.** No `pickle`, `yaml.load` without a safe loader, Java native serialization, or PHP `unserialize` on external input.
- [ ] **User input is never used as a template.** Pass it as a variable to a fixed template (server-side template injection).
- [ ] **In LLM-integrated code, untrusted content is treated as data and model output is treated as untrusted.** Do not concatenate user or retrieved content into the system prompt. Validate model output before it reaches HTML, SQL, a shell, a URL, or a tool call. Tools exposed to the model run with the caller's permissions, not the service's.

### 🟠 High

- [ ] **Schemas are strict.** Unknown fields are rejected or stripped, which also blocks mass assignment and prototype pollution (`__proto__`, `constructor`).
- [ ] **Redirect targets are validated.** Allow relative paths or an allowlist of hosts. Do not redirect to a raw `next` or `returnUrl` parameter.
- [ ] **XML parsers have external entities and DTD processing disabled** (XXE).
- [ ] **Length limits exist on every string and array, and regular expressions on user input are safe from catastrophic backtracking** (ReDoS).
- [ ] **Input is canonicalized before validation.** Decode once, normalize Unicode, then validate. Validate the value that will actually be used.
- [ ] **Values written into HTTP headers, log lines, emails, or CSV files are stripped of CR/LF and formula prefixes** (`=`, `+`, `-`, `@` for CSV).
- [ ] **Allowlists are preferred over denylists** for formats, file types, hosts, and enum-like fields.
- [ ] **Uploaded images are re-encoded server-side and served with a safe `Content-Type` and `Content-Disposition`.**

### 🟡 Medium

- [ ] **Numeric input has explicit bounds**, including pagination, quantities, and amounts. Reject negative and overflow values where they make no sense.
- [ ] **Validation errors are returned without echoing the raw input back into HTML.**
- [ ] **The same schema is shared between the API layer and background jobs that consume the same payload.**

### 🟢 Low

- [ ] **Whitespace is trimmed and case is normalized for identifiers such as emails** before uniqueness checks.

## Example

Insecure: unvalidated body, shell interpolation.

```ts
const { url, format } = await req.json();
exec(`convert ${url} out.${format}`);
```

Secure: strict schema, allowlisted values, argument array.

```ts
const Body = z.object({
  fileId: z.string().uuid(),
  format: z.enum(["png", "webp"]),
}).strict();

const { fileId, format } = Body.parse(await req.json());
const input = await resolveOwnedFilePath(session.userId, fileId);

await execFileAsync("convert", [input, `out.${format}`], { timeout: 10_000 });
```

## When to consult other patterns

- Validated input goes into a query → `data-access.md`
- The handler itself, its auth, and its rate limits → `api-endpoints.md`
- Input is a URL the server will fetch → `api-endpoints.md` (SSRF)
- Parsing or validation failures and how to report them → `errors-logging.md`
- Adding a parser, sanitizer, or validation library → `dependencies.md`
