# Pattern: Data Access

Triggers when generating or editing: SQL queries, ORM calls, query builders, raw SQL helpers, NoSQL filters, search queries, migrations, row-level security policies, database functions, and any code that scopes data to a user or tenant.

**Primary OWASP categories:** A01 (Broken Access Control), A05 (Injection).

## The core principle

> Data never becomes query structure, and every query carries its own authorization.

User input is passed as bound parameters. The ownership or tenant condition is part of the query itself, not a filter applied to the results afterwards.

## Checklist

### 🔴 Critical

- [ ] **Queries are parameterized.** No string concatenation or template interpolation of user input into SQL. This includes ORM escape hatches such as `$queryRawUnsafe`, `sequelize.query` with interpolated strings, `knex.raw`, and `cursor.execute(f"...")`.
- [ ] **Dynamic identifiers are resolved through an allowlist.** Table names, column names, and sort direction cannot be bound as parameters. Map the user's choice to a fixed set of known values.
- [ ] **Every query is scoped to the authenticated principal.** The `user_id` or `tenant_id` comes from the verified session and appears in the `WHERE` clause of reads, updates, and deletes.
- [ ] **NoSQL filters reject operator injection.** Cast or validate types before building the filter so that `{ "$ne": null }` cannot arrive where a string was expected.
- [ ] **Privileged database credentials never reach the client and are never used on behalf of a user without an explicit check.** Service-role keys and admin SDKs bypass row-level security.
- [ ] **Row-level security is enabled on every table reachable from a client-accessible API** (Supabase, PostgREST, Hasura, Firebase rules). Each operation has a policy. Inserts and updates have a `WITH CHECK` clause. No `USING (true)` on private data.

### 🟠 High

- [ ] **Updates and deletes include the ownership predicate in the same statement.** Avoid check-then-act sequences that can race.
- [ ] **The application connects with a least-privilege database role.** No superuser, no schema-owner role for request handling.
- [ ] **Sensitive columns are excluded by default.** Select explicit columns instead of `SELECT *`.
- [ ] **Concurrent updates to balances, stock, quotas, and counters are atomic.** Use conditional updates, row locks, or serializable transactions.
- [ ] **Multi-step writes run in a transaction that rolls back completely on failure.**
- [ ] **Queries have a result limit**, and `LIKE` patterns escape `%` and `_` when the input is meant to be literal.
- [ ] **Views and functions do not silently bypass row-level security.** In Postgres, create views with `security_invoker = true`, and set a fixed `search_path` on `SECURITY DEFINER` functions.
- [ ] **Authorization data is stored where the user cannot edit it.** In Supabase, use `app_metadata` or a server-controlled table, not `user_metadata`.

### 🟡 Medium

- [ ] **Soft-deleted and archived rows are filtered consistently** through a shared scope or view.
- [ ] **Highly sensitive fields are encrypted at the application level.** See `crypto.md`.
- [ ] **Migrations are reviewed for new tables without policies and for widened permissions.**
- [ ] **Audit columns record who changed what and when on sensitive tables.**

### 🟢 Low

- [ ] **Database error details are logged server-side and never returned to the client.**
- [ ] **Connection strings enforce TLS.**

## Example

Insecure: injection through both the value and the sort column, and no ownership scope.

```ts
const rows = await db.$queryRawUnsafe(
  `SELECT * FROM orders WHERE status = '${status}' ORDER BY ${sort}`
);
```

Secure: bound parameters, allowlisted identifier, scoped to the session user.

```ts
const SORTABLE = { created: "created_at", total: "total" } as const;
const column = SORTABLE[sort as keyof typeof SORTABLE] ?? "created_at";

const rows = await db.$queryRaw`
  SELECT id, status, total, created_at
  FROM orders
  WHERE user_id = ${session.userId} AND status = ${status}
  ORDER BY ${Prisma.raw(column)} DESC
  LIMIT 50
`;
```

`Prisma.raw` is acceptable here only because `column` can take one of two hardcoded values.

Row-level security as defense in depth:

```sql
ALTER TABLE orders ENABLE ROW LEVEL SECURITY;

CREATE POLICY orders_select_own ON orders
  FOR SELECT USING (user_id = auth.uid());

CREATE POLICY orders_insert_own ON orders
  FOR INSERT WITH CHECK (user_id = auth.uid());
```

RLS does not replace application-layer scoping. Use both.

## When to consult other patterns

- The query result is returned over HTTP → `api-endpoints.md`
- Filter or search values come from user input → `input-handling.md`
- Storing passwords, tokens, or encrypted fields → `crypto.md`
- Database credentials and connection strings → `secrets-config.md`
- Adding or upgrading an ORM or driver → `dependencies.md`
