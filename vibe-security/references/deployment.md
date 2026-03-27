# Deployment Security

## Production Configuration

- **Disable debug mode** in production. Debug pages often leak stack traces, environment variables, and internal paths.
- **Disable source maps** in production. Source maps expose your entire source code to anyone who opens DevTools.
- **Verify `.git` directory is not accessible** in production. If `https://yoursite.com/.git/HEAD` returns content, your entire source code and commit history (including any secrets ever committed) are exposed.

## Environment Separation

Use separate environment variables for each environment in Vercel (or equivalent):

| Environment | Purpose |
|-------------|---------|
| Production | Live users, real keys |
| Preview | PR previews, should use test/staging keys |
| Development | Local dev, uses local/test keys |

Preview deployments should **never** use production API keys, database credentials, or payment keys. A preview deployment is often accessible to anyone with the URL.

### Preview Deployments Are Public by Default

**This is a critical gap in vibe-coded projects on Vercel.** By default, every PR or branch push generates a preview URL that is publicly accessible. If preview deployments use production environment variables:

- Production database is exposed via the preview app
- Real Stripe keys process test payments against live accounts
- AI API keys get burned through preview usage

**Mitigations:**
1. In Vercel Dashboard → Settings → Environment Variables, scope production secrets to "Production" only — never "Preview"
2. Enable Vercel Deployment Protection (Settings → Deployment Protection) to require authentication for preview URLs
3. Use separate Supabase projects or branches for preview environments
4. Never share preview URLs publicly — treat them as semi-private

### Check for Leaked Preview URLs

Search Slack, email, and project management tools for Vercel preview URLs that may have been shared externally. Each one is a potential entry point if it uses production credentials.

## Security Headers

Set these headers on all responses:

```
Content-Security-Policy: default-src 'self'; script-src 'self'
Strict-Transport-Security: max-age=31536000; includeSubDomains
X-Frame-Options: DENY
X-Content-Type-Options: nosniff
Referrer-Policy: strict-origin-when-cross-origin
Permissions-Policy: camera=(), microphone=(), geolocation=()
```

Adjust `Content-Security-Policy` based on your app's needs (e.g., if you use inline styles or load scripts from CDNs), but start restrictive and loosen as needed — not the other way around.

## Pre-Ship Checks

Before deploying:
- Run `gitleaks detect` on your repo to scan for leaked secrets in git history
- Verify `.env` files are in `.gitignore`
- Confirm debug mode / verbose logging is disabled
- Check that error pages don't leak stack traces
- Verify CORS is configured to allow only your domains, not `*`

## CORS Configuration

- Never use `Access-Control-Allow-Origin: *` on authenticated endpoints
- Whitelist only your own domains
- Be careful with `Access-Control-Allow-Credentials: true` — it must be paired with specific origins, not wildcards
