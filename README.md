<p align="center">
    <img src="https://img.shields.io/badge/security-vibe--coded%20apps-DC2626.svg" alt="Security for vibe-coded apps" />
    <img src="https://img.shields.io/badge/license-MIT-blue.svg" alt="MIT License" />
    <a href="https://twitter.com/raroque">
        <img src="https://img.shields.io/badge/Contact-@raroque-95a5a6.svg?style=flat" alt="Twitter: @raroque" />
    </a>
</p>

<h1 align="center">Vibe Security - Agent Skill for AI Coding Assistants</h1>

An agent skill that helps secure vibe-coded apps - or honestly any app - from common security vulnerability patterns. Built by [Chris Raroque](https://www.youtube.com/@raroque) ([@raroque](https://twitter.com/raroque)) in collaboration with my colleagues at [Aloa](https://aloa.co).

AI assistants are great at building features fast but consistently get security wrong: hardcoding secrets, skipping row-level security, trusting client-submitted prices, storing tokens in localStorage. This skill catches those patterns before they ship.

**Need help building AI apps, custom agents, or implementing AI at your company?** Work with Chris and the team at [Aloa](https://aloa.co).

## Background

This skill was built specifically to address the security issues that keep showing up in vibe-coded applications. When you're building fast with AI, security fundamentals get skipped - and the AI assistants themselves are often the ones introducing the vulnerabilities. This skill gives your agent the knowledge to catch and prevent those patterns.

It uses the [Agent Skills](https://agentskills.io/home) format, so it works with Claude Code, OpenAI Codex, and other compatible agents.

The security rules are organized as reference files that the agent loads based on what technologies your project uses. If you're using Supabase, it checks RLS policies. If you're using Stripe, it checks payment flows. If you're using React Native, it checks for secrets in the JS bundle. No wasted context on irrelevant checks.

## Installing Vibe Security

### Claude Code

```bash
npx skills add https://github.com/raroque/vibe-security-skill --skill vibe-security
```

If `npx` isn't available, install Node.js first: `brew install node` (macOS) or download from [nodejs.org](https://nodejs.org).

### OpenAI Codex

```bash
npx skills add https://github.com/raroque/vibe-security-skill --skill vibe-security
```

Select "Codex" when prompted for the agent platform.

### Manual Installation (Claude Code)

Clone this repo and copy the `vibe-security/` folder to your project or global skills directory:

```bash
# Project-level (applies to one project)
cp -r vibe-security/ .claude/skills/vibe-security/

# Global (applies to all projects)
cp -r vibe-security/ ~/.claude/skills/vibe-security/
```

## Using Vibe Security

**Claude Code:** Use `/vibe-security` to trigger a full security audit, or just ask naturally - "check my code for security issues", "is this safe?", "audit this project".

**Codex:** Use `$vibe-security` or describe what you need - "review this for vulnerabilities", "check my Supabase RLS".

The skill also activates automatically when you're writing or reviewing code that handles authentication, payments, database access, API keys, or user data.

## What It Checks

| Category | What It Catches |
|----------|----------------|
| **Secrets & Env Vars** | Hardcoded API keys, secrets in `NEXT_PUBLIC_`/`VITE_`/`EXPO_PUBLIC_` vars, missing `.gitignore` |
| **Database Security** | Disabled Supabase RLS, `USING (true)` policies, missing `WITH CHECK`, exposed sensitive fields, Firebase `allow: if true` rules, Convex missing auth |
| **Auth & Authorization** | `jwt.decode()` without verify, middleware-only auth, unprotected Server Actions, tokens in localStorage |
| **Rate Limiting** | Missing limits on auth/AI/email endpoints, client-tamperable rate counters, no billing caps |
| **Payments** | Client-submitted prices, missing webhook signature verification, stale subscription checks |
| **Mobile** | API keys in JS bundle, `AsyncStorage` for tokens, unsafe deep links, weak biometric auth |
| **AI / LLM** | Exposed AI API keys, no usage caps, prompt injection, unsafe output rendering |
| **Deployment** | Debug mode in production, exposed source maps, missing security headers, `.git` accessible |
| **Data Access** | SQL injection, Prisma operator injection, `$queryRawUnsafe`, mass assignment |

## Contributing

Contributions, corrections, and improvements are very welcome! This is meant to be a community resource. If you've found a security anti-pattern that AI assistants keep introducing, please add it.

See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

## License

Vibe Security is available under the MIT License. See [LICENSE](LICENSE) for details.

Created by [Chris Raroque](https://www.youtube.com/@raroque) ([@raroque](https://twitter.com/raroque)) and the team at [Aloa](https://aloa.co).
