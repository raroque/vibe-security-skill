# Framework Version Security

AI-generated projects frequently pin outdated framework versions or inherit them from starter templates. A vulnerable framework version is often the single highest-impact issue in an audit.

## Check Process

1. Read `package.json` (and `package-lock.json` or equivalent) for framework versions.
2. Cross-reference against the critical CVEs below.
3. Flag any match as **High** or **Critical** severity.

## Next.js Critical CVEs

### CVE-2025-66478 — React Server Components RCE (CVSS 10.0, Dec 2025)

Remote Code Execution via the RSC protocol. Attacker-controlled requests can trigger unintended server execution paths. Also tracked upstream as CVE-2025-55182 in React.

- **Affected:** All Next.js versions using App Router with unpatched React
- **Fixed in:** next@15.0.5, 15.1.9, 15.2.6, 15.3.6, 15.4.8, 15.5.7, 16.0.7
- **Action:** Upgrade immediately. This is actively exploited.

### CVE-2025-29927 — Middleware Authorization Bypass (CVSS 9.1, Mar 2025)

Adding `x-middleware-subrequest` header bypasses all middleware logic, including auth checks.

- **Affected:** 11.1.4–12.3.4, 13.0.0–13.5.8, 14.0.0–14.2.24, 15.0.0–15.2.2
- **Fixed in:** 12.3.5, 13.5.9, 14.2.25, 15.2.3
- **Action:** Upgrade AND stop relying on middleware as sole auth layer.
- **Note:** Vercel-hosted apps were not affected, but self-hosted deployments were.

### CVE-2025-55183 / CVE-2025-55184 — RSC Source Exposure + DoS (Dec 2025)

Source code exposure (medium) and denial of service via infinite loop (high) in RSC deserialization.

- **Affected:** Next.js App Router applications
- **Fixed in:** Same versions as CVE-2025-66478

## React Critical CVEs

### CVE-2025-55182 — React Server Components Deserialization RCE (CVSS 10.0)

The upstream React vulnerability behind CVE-2025-66478.

- **Affected:** react@19.0.0, 19.1.0, 19.1.1, 19.2.0 and related packages
- **Fixed in:** react@19.0.1, 19.1.2, 19.2.1

## What to Check

```bash
# Check Next.js version
cat package.json | grep '"next"'

# Check React version
cat package.json | grep '"react"'

# Check for all known vulnerabilities in dependencies
npm audit
```

## The Deeper Lesson

These CVEs reinforce a critical architectural principle: **middleware is NOT a security boundary.** It is a convenience layer for routing and edge-level decisions. Auth checks must be duplicated in:
- Server Actions
- Route Handlers (`app/api/`)
- Data access functions
- Database-level policies (RLS)

Think of middleware as a building's front door: it directs traffic and does a first pass, but every room inside must still have its own lock.
