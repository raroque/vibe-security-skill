# Authentication & Authorization

## JWT Handling

- **Use `jwt.verify()`, never `jwt.decode()` alone.** `decode` reads the payload without checking the signature — an attacker can forge any payload.
- **Explicitly reject `"alg": "none"`.** Some JWT libraries accept unsigned tokens if the algorithm is set to `"none"`. Your verification must reject this.
- **Validate issuer, audience, and expiration** — not just the signature.

```typescript
// BAD: reads token without verifying signature
const payload = jwt.decode(token);

// GOOD: verifies signature, rejects tampered tokens
const payload = jwt.verify(token, secret, {
  algorithms: ['HS256'],
  issuer: 'your-app',
});
```

## Next.js Middleware Is NOT a Security Boundary

**This is the #1 auth mistake in vibe-coded Next.js apps.**

Next.js middleware runs at the edge and is convenient for auth checks, but it is **architecturally incapable of being a reliable sole auth layer**:

- **CVE-2025-29927 (CVSS 9.1):** Middleware completely bypassed via a spoofed `x-middleware-subrequest` header. Affected versions 11.1.4 through 15.2.2.
- **CVE-2025-66478 (CVSS 10.0):** React Server Components RCE — unrelated to middleware, but reinforces that framework-level protections have limits.

Always verify auth again in **every** server-side entry point:
- Server Actions (they compile to public POST endpoints)
- Route Handlers (`app/api/`)
- Data access functions / database queries
- Database-level policies (RLS)

Middleware should direct traffic and do a first pass — like a building's front door. But every room inside must have its own lock.

## Server Actions Are Public Endpoints

Server Actions compile into public POST endpoints. Anyone can call them with `curl`. AI assistants frequently generate Server Actions that assume they're only called by the UI:

```typescript
// BAD: no auth check, no input validation
'use server';
export async function deleteItem(id: string) {
  await db.items.delete({ where: { id } });
}

// GOOD: validates input, authenticates, and authorizes
'use server';
export async function deleteItem(input: unknown) {
  const parsed = schema.safeParse(input);
  if (!parsed.success) return { error: 'Invalid input' };

  const session = await auth();
  if (!session?.user) redirect('/login');

  // Authorize: verify ownership, not just login
  await db.items.deleteMany({
    where: { id: parsed.data.id, userId: session.user.id }
  });
}
```

Every Server Action needs three things at the top:
1. **Input validation** (Zod or similar runtime schema)
2. **Authentication** (verify the user is logged in)
3. **Authorization** (verify the user owns the resource)

## API Route Handlers

Same rules apply to `app/api/` route handlers. Every route handler is a public endpoint. Authenticate and authorize at the top of every handler.

## Data Leakage to Client Components

Never pass entire database objects to Client Components. They may contain sensitive fields (hashed passwords, internal IDs, admin flags). Select only the fields the client needs:

```typescript
// BAD: leaks all fields to the client
const user = await db.users.findUnique({ where: { id } });
return <UserProfile user={user} />;

// GOOD: select only needed fields
const user = await db.users.findUnique({
  where: { id },
  select: { name: true, avatarUrl: true }
});
return <UserProfile user={user} />;
```

Use `import 'server-only'` at the top of data access modules to prevent them from being accidentally imported into Client Components.

## Session & Token Storage

- Store tokens in `HttpOnly + Secure + SameSite=Lax` cookies, **not localStorage**.
- localStorage is accessible to any JavaScript on the page — a single XSS vulnerability exposes all tokens.
- `HttpOnly` cookies are invisible to JavaScript and sent automatically by the browser.

## Passkeys / WebAuthn (2026 Best Practice)

As of 2026, all major platforms (iOS, Android, Windows, macOS) support passkeys natively. For new applications:

- **Default to passkeys** for sign-in where possible. They are phishing-resistant because each credential is cryptographically bound to your domain.
- **Treat passwords as a legacy fallback.** If you must support passwords, enforce strong minimum requirements and rate-limit login attempts.
- Passkeys eliminate entire vulnerability classes: credential stuffing, brute force, phishing, and password reuse.

If the project uses a managed auth provider (Clerk, Auth0, Supabase Auth), check whether passkey support is enabled. If it's a new project, recommend enabling it.

## `import 'server-only'`

Use `import 'server-only'` at the top of any module that contains auth logic, database queries, or secret keys. This causes a build error if the module is accidentally imported into a Client Component, preventing sensitive code from leaking to the browser bundle.

```typescript
// lib/auth.ts
import 'server-only';

export async function getCurrentUser() {
  // This function can never accidentally end up in client code
  const session = await auth();
  // ...
}
```
