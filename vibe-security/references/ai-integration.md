# AI / LLM Integration Security

## API Keys Are Server-Side Only

AI API keys (OpenAI, Anthropic, Google, etc.) must never appear in client-side code. They allow unlimited API usage at your expense. A leaked key can drain thousands of dollars in minutes.

- No `NEXT_PUBLIC_OPENAI_API_KEY`
- No API keys in React Native / Expo bundles
- No API keys in client-side JavaScript

All AI API calls go through your backend. The client sends the user's message to your server; your server calls the AI API.

## Spending Caps

Set hard spending caps on every AI API provider:
- OpenAI: Usage limits in dashboard
- Anthropic: Spending limits in console
- Google: Budget alerts in Cloud Console

Also implement **per-user usage limits** in your application:
- Track token usage per user in your database
- Set daily/monthly caps per user or per tier
- Return a clear error when limits are exceeded
- Don't rely on the AI provider's caps alone — they may have lag

## Prompt Injection

User input must be sanitized before inclusion in prompts. Never concatenate raw user input into system prompts:

```typescript
// BAD: user can override system instructions
const prompt = `You are a helpful assistant. User says: ${userInput}`;

// BETTER: separate system and user messages
const messages = [
  { role: 'system', content: 'You are a helpful assistant.' },
  { role: 'user', content: userInput },
];
```

Even with separate messages, be aware that sophisticated prompt injection can still occur. For high-stakes applications, consider:
- Input validation and filtering
- Output validation before acting on LLM responses
- Limiting the LLM's capabilities (no tool access for user-facing chat)

## LLM Output Is Untrusted

LLM responses should be treated as untrusted user input:

- **Sanitize before rendering as HTML** — LLM output can contain script tags or event handlers
- **Never execute LLM output as code** without sandboxing
- **Validate tool/function call parameters** — if using function calling, validate all returned parameters against an allowlist and schema before executing

## Tool / Function Calling

If your application gives an LLM access to tools (database queries, API calls, file operations):
- Restrict operations to a safe allowlist
- Validate all parameters from the LLM against a schema
- Use least-privilege access (read-only where possible)
- Log all tool invocations for audit
- Never let the LLM construct raw SQL or shell commands from user input

## MCP (Model Context Protocol) Security

MCP connectors give AI agents access to external services (Supabase, GitHub, Slack, etc.). This is powerful but creates new attack surfaces:

### MCP + service_role = Bypassed RLS

If your MCP connector uses the Supabase `service_role` key, the AI agent bypasses ALL Row-Level Security. A prompt injection — hidden in a code comment, README, or package description that the agent reads — can instruct the agent to exfiltrate data.

**Mitigations:**
- Never give MCP connectors `service_role` access to production databases
- Use read-only database credentials for MCP connections where possible
- Review every MCP operation before approving it (don't auto-approve)
- Keep MCP connectors to the minimum set you actually need

### Prompt Injection via MCP Inputs

Content returned by MCP tools (e.g., file contents from GitHub, messages from Slack) can contain adversarial instructions. The AI agent may follow these instructions because it treats MCP tool results as trusted context.

**Mitigations:**
- Be aware that any content the agent reads could contain injections
- Don't auto-execute code or commands suggested by content retrieved via MCP
- Use separate MCP configurations for development and production

## AI Billing Protection

Beyond per-provider spending caps, implement application-level controls:

- **Per-user token budgets** stored server-side (not in client-accessible tables)
- **Request-level cost estimation** before making expensive API calls
- **Circuit breakers** — if total API spend exceeds a threshold in a short window, halt all AI calls and alert
- **Separate API keys** for development and production AI usage — a dev key leak shouldn't drain your production budget
