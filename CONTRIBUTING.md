# Contributing to Vibe Security

Contributions, corrections, and improvements are welcome! This skill is meant to be a community resource for making AI-generated code safer.

## How to Contribute

1. **Fork** this repository
2. **Create a branch** for your changes
3. **Make your changes** following the guidelines below
4. **Submit a pull request** with a clear description of what you changed and why

## What to Contribute

- **New vulnerability patterns** — Found a security anti-pattern that AI assistants commonly introduce? Add it to the relevant reference file or create a new one.
- **Better examples** — Clearer before/after code samples help the AI understand what to flag and how to fix it.
- **Corrections** — If something is wrong or outdated, please fix it.
- **New platform coverage** — The skill currently covers Supabase, Firebase, Convex, Stripe, Next.js, and React Native. If you have security patterns for other platforms commonly used in vibe-coded apps, add them.
- **Real-world cases** — If you've seen a specific vulnerability in the wild that AI assistants keep introducing, that's exactly what this skill is for.

## Guidelines

- **Keep it concise.** Markdown should be scannable. Use short sentences and code examples over long explanations.
- **Explain the "why."** Don't just say "don't do X" — explain what an attacker can do if you do X. This helps the AI understand the severity and make better judgments.
- **Focus on AI-generated patterns.** This isn't a general security checklist. Focus on mistakes that AI coding assistants specifically and repeatedly make.
- **Test your changes.** If possible, verify that an AI assistant actually catches the issue with your updated instructions.
- **One topic per reference file.** If you're adding a new category, create a new file in `references/` and add a step to the audit process in `SKILL.md`.

## Code of Conduct

Be kind, be constructive, be helpful. We're all here to make vibe-coded apps safer.

## License

By contributing, you agree that your contributions will be licensed under the MIT License.
