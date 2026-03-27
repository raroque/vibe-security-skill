# Vibe Security v2.0 — Updated Agent Skill for AI Coding Assistants

> Fork of [raroque/vibe-security-skill](https://github.com/raroque/vibe-security-skill) by Chris Raroque, updated with CVEs, attack patterns, and tooling from 2025–2026.

An agent skill that audits vibe-coded apps for security vulnerabilities. Works with Claude Code, OpenAI Codex, and other agents that support the [Agent Skills](https://agentskills.io) standard.

## Why this fork?

The original skill (v1.0, March 2025) covers the fundamentals well. But the security landscape for vibe-coded apps has shifted significantly since then:

- **React2Shell (CVE-2025-66478, CVSS 10.0)** — Critical RCE in React Server Components, December 2025
- **Slopsquatting** — LLMs hallucinate package names in ~20% of generated code; attackers register them with malware
- **2,000+ vulnerabilities found** across 5,600 vibe-coded apps ([Escape.tech research](https://escape.tech/blog/methodology-how-we-discovered-vulnerabilities-apps-built-with-vibe-coding/))
- **Supabase new API key model** — publishable keys + revocable secrets replaced anon/service_role in 2025
- **MCP attack surface** — AI agents with service_role access bypass all RLS via prompt injection

This fork adds coverage for these gaps while keeping the original structure and philosophy intact.

## What's new in v2.0

### 3 new reference files

| File | What it covers |
|------|---------------|
| `framework-versions.md` | Known critical CVEs for Next.js and React — version checking as first audit step |
| `supply-chain.md` | Slopsquatting, hallucinated packages, default credentials (`user@example.com:password123`), weak JWT secrets |
| `tooling.md` | Automated security tools by stack (gitleaks, npm audit, Supabase Security Advisor, etc.) |

### 5 updated reference files

| File | What was added |
|------|---------------|
| `database-security.md` | Supabase Edge Functions security, MCP + service_role vulnerability, new API key model, GoTrue auth issues |
| `authentication.md` | Middleware explicitly marked as NOT a security boundary (CVE-2025-29927 + CVE-2025-66478), passkeys/WebAuthn, `import 'server-only'` |
| `ai-integration.md` | MCP security (prompt injection via tool results), AI billing circuit breakers |
| `deployment.md` | Vercel preview deployments as attack surface (public by default, often with production keys) |
| `secrets-and-env.md` | AI-generated default credentials patterns, new Supabase key model |

### Audit process changes

- **Step 1 is now Framework Version Check** — catches vulnerable Next.js/React before anything else
- **New Step 7: Supply Chain & Dependencies** — slopsquatting, lock files, dependency auditing
- **Output format updated** — every audit ends with "Next Steps" listing specific tools to run
- **11 audit steps** (was 9)

Total content: 9 → 12 reference files, 772 → 1,176 lines (+52%).

## Installing

### Claude Code

```bash
npx skills add https://github.com/fartiacht/vibe-security-skill --skill vibe-security
```

### Claude.ai

Download the `vibe-security/` folder as a ZIP and upload via **Settings → Features**.

### Manual (any agent)

```bash
git clone https://github.com/fartiacht/vibe-security-skill.git
cp -r vibe-security-skill/vibe-security/ ~/.claude/skills/vibe-security/
```

## Using it

**Claude Code:** `/vibe-security` or ask naturally — "audit my security", "is this safe?", "check my Supabase RLS".

The skill loads only the reference files relevant to your stack. If you don't use Stripe, it skips payment checks. If you don't use Firebase, it skips Firebase rules.

## Context

I'm a non-developer founder building SaaS products with AI (Next.js + Supabase + Vercel). I rely entirely on AI coding assistants — which means I'm exactly the type of person this skill is built for.

I improved this skill after researching the latest vibe-coding security incidents and CVEs. The updates are based on published research, official security advisories, and documented real-world breaches — not on my own security expertise.

**If you spot gaps, outdated patterns, or things I got wrong, please open an issue or PR.** I'd rather have someone tell me I missed something than ship with a false sense of security.

## Credits

- Original skill by [Chris Raroque](https://github.com/raroque) ([@raroque](https://twitter.com/raroque)) and the team at [Aloa](https://aloa.co)
- Research sources: [Escape.tech](https://escape.tech), [Supabase Security Retro 2025](https://supabase.com/blog/supabase-security-2025-retro), [Next.js Security Advisories](https://nextjs.org/blog), [Aikido Security](https://www.aikido.dev/blog/slopsquatting-ai-package-hallucination-attacks), [Unit 42](https://unit42.paloaltonetworks.com/securing-vibe-coding-tools/)

## License

MIT — same as the original.
