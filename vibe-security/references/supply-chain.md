# Supply Chain & Dependency Security

AI coding assistants introduce a unique supply chain risk: they hallucinate package names that don't exist, and attackers register those names with malicious code. This is called **slopsquatting**.

## Slopsquatting (Hallucinated Packages)

Research shows ~20% of AI-generated code samples recommend packages that don't exist on npm or PyPI. 43% of hallucinated names recur consistently across repeated prompts, making them predictable targets for attackers.

Real-world example: In January 2026, a hallucinated npm package `react-codeshift` (a conflation of real packages `jscodeshift` and `react-codemod`) was found spreading through AI-generated Agent Skills on GitHub.

### What to Check

For every dependency in `package.json` or `requirements.txt`:

1. **Verify the package exists** on the official registry (npmjs.com, pypi.org)
2. **Check download stats** — a legitimate utility package has thousands of weekly downloads, not 12
3. **Check the publisher** — legitimate packages have identifiable authors with multiple published packages
4. **Check creation date** — a package created last week that your AI just recommended is suspicious
5. **Check for name confusion** — does the name suspiciously combine two real package names?

### Red Flags

- Package has fewer than 100 weekly downloads
- Package was published within the last 30 days
- Package name looks like two well-known packages smashed together
- Package has no GitHub repo linked, or the repo has no stars
- Package has a `postinstall` script (check `package.json` of the dependency)

## Lock File Hygiene

Always commit lock files (`package-lock.json`, `yarn.lock`, `pnpm-lock.yaml`). Lock files pin exact versions and integrity hashes, preventing supply chain substitution.

```bash
# Verify lock file exists and is committed
git ls-files | grep -E "(package-lock|yarn\.lock|pnpm-lock)"
```

If no lock file is committed, any `npm install` can pull in different (potentially compromised) versions than what was tested.

## Dependency Auditing

Run these after every vibe-coding session before deploying:

```bash
# npm — built-in audit
npm audit

# More thorough — check for known vulnerabilities
npx audit-ci --critical

# Scan for leaked secrets in dependencies
npx lockfile-lint --path package-lock.json --type npm --allowed-hosts npm
```

For Python projects:
```bash
pip-audit
```

## Unpinned Dependencies

AI assistants often generate `package.json` with loose version ranges:

```json
// BAD: accepts any minor/patch version — a compromised update auto-installs
"dependencies": {
  "some-lib": "^2.0.0"
}

// BETTER: pin exact versions for production
"dependencies": {
  "some-lib": "2.0.3"
}
```

At minimum, use a lock file. For critical production apps, pin exact versions.

## Common AI-Generated Default Credentials

AI-generated apps frequently ship with hardcoded test credentials that work in production:

| Username | Password | Where it appears |
|----------|----------|-----------------|
| `user@example.com` | `password123` | Login/register endpoints |
| `admin@admin.com` | `admin` or `admin123` | Admin panels |
| `test@test.com` | `test` or `test123` | Seeded user data |
| `demo@demo.com` | `demo` | Demo accounts |

Search for these patterns:
```bash
grep -rn "password123\|admin123\|@example\.com\|@test\.com\|@admin\.com" --include="*.ts" --include="*.js" --include="*.tsx" --include="*.sql"
```

If found in seed data that runs in production, or in auth logic, flag as **High** severity.

## JWT Signing Secrets

AI-generated apps commonly use weak, predictable JWT secrets:

- `"secret"`, `"jwt-secret"`, `"your-secret-key"`
- `"supersecret"`, `"changeme"`, `"keyboard-cat"`

These are trivially guessable, allowing token forgery. Search for:
```bash
grep -rn "secret\|changeme\|keyboard.cat\|supersecret" --include="*.ts" --include="*.js" --include="*.env*"
```
