---
name: Security - No Key Exposure
description: Never expose API keys, tokens, or secrets in chat responses or memory files
type: feedback
---

Never print, display, or log API keys, PATs, passwords, or any secrets in chat or in any memory file.

**Why:** User explicitly requested this. Keys must stay in .env only — never leak into conversation history or persisted files.

**How to apply:**
- When reading secrets from .env, use them silently in commands only — never echo or display the value
- If a command output might contain a secret, suppress or mask it
- Memory files must never contain secret values — reference the variable name (e.g. GITHUB_PAT_AGENT_MARKETING) but never the value
- If a secret is accidentally exposed in chat, immediately tell the user and ask them to rotate the key before continuing
