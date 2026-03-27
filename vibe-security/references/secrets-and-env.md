# Secrets & Environment Variables

## Hardcoded Credentials

Never hardcode API keys, tokens, passwords, or credentials in source code. This includes:
- Strings that look like API keys in source files
- Connection strings with embedded passwords
- Private keys or certificates in the repo

If a secret was ever committed to Git history, consider it compromised — deleting the file doesn't remove it from history. The key must be rotated immediately. Run `gitleaks detect` to scan for leaked secrets.

## Client-Side Environment Variable Prefixes

These prefixes cause env vars to be inlined into the client bundle at build time. Everything in the bundle is visible to anyone:

| Framework | Client Prefix | Danger |
|-----------|--------------|--------|
| Next.js | `NEXT_PUBLIC_` | Inlined into browser JS at build time |
| Vite | `VITE_` | Inlined into browser JS at build time |
| Expo / React Native | `EXPO_PUBLIC_` | Baked into the app bundle |
| Create React App | `REACT_APP_` | Inlined into browser JS at build time |

**What belongs client-side:**
- Stripe publishable key (`pk_live_*`, `pk_test_*`)
- Supabase anon key
- Firebase client config (apiKey, authDomain, projectId)
- Public analytics IDs

**What must NEVER be client-side:**
- Supabase `service_role` key (bypasses all RLS)
- Stripe secret key (`sk_live_*`, `sk_test_*`)
- Any database connection string
- Any third-party API secret key
- JWT signing secrets
- OAuth client secrets

## .gitignore

Ensure `.env`, `.env.local`, `.env.*.local`, and any file containing secrets is in `.gitignore` **before the first commit**. Check that `.env.example` or `.env.sample` files contain only placeholder values, not real keys.

## Detection Tips

When auditing, search for:
- Files named `.env` that are tracked by git (`git ls-files | grep .env`)
- Strings matching common key patterns: `sk_live_`, `sk_test_`, `AKIA`, `ghp_`, `glpat-`, `xoxb-`, `Bearer `
- `process.env.NEXT_PUBLIC_` or `import.meta.env.VITE_` referencing anything with "secret", "private", "service", or "key" in the name
- Hardcoded URLs containing credentials (e.g., `postgresql://user:password@host`)

## Common AI-Generated Default Credentials

AI code generation tools consistently produce applications with hardcoded default credentials. These are functional in production and enable trivial account takeover.

Search for these patterns in your codebase:
```bash
grep -rn "password123\|admin123\|@example\.com\|@test\.com\|@admin\.com\|changeme\|supersecret\|keyboard.cat\|your-secret-key" --include="*.ts" --include="*.js" --include="*.tsx" --include="*.sql" --include="*.env*"
```

Common offenders:
- `user@example.com` / `password123` — login and seed data
- `admin@admin.com` / `admin` — admin panels
- JWT secrets: `"secret"`, `"jwt-secret"`, `"your-secret-key"`, `"supersecret"`, `"changeme"`

If any of these are present in production code or seed data that runs in production, flag as **High** severity. See also `references/supply-chain.md` for the full list.

## Supabase New Key Model (2025+)

Supabase projects created after mid-2025 use a new API key model:
- **Publishable keys** replace the old `anon` key — safe for client-side
- **Secret keys** replace `service_role` — revocable and auto-detected by GitHub Secret Scanning

If auditing a newer Supabase project, check for the new key format. The same principle applies: publishable keys go client-side, secret keys stay server-side only.
