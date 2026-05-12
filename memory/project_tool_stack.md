---
name: myNHD Tool Stack Decisions
description: Confirmed tool stack and cost structure for the myNHD AI program engagement
type: project
---

Agreed tool stack for both programs, zero-code, Claude-agent based.

**Confirmed decisions:**
- myNHD has Claude Teams (already licensed) — covers all direct Claude usage
- myNHD has Levitate license — kept for Program 2, integrated via Zapier
- Zapier is the automation layer (has native Levitate + Anthropic connectors)
- Buffer free tier for social scheduling (Program 1)
- Canva for graphic adaptation — need to confirm myNHD uses it
- Google Drive/Sheets for human-to-agent handoff
- GitHub repo for prompt .md files and workflow docs
- Zero custom code — all configuration, .md prompt files only

**Net new cost to myNHD for POC: ~$25–35/mo**
- Zapier Starter: ~$20/mo
- Claude API calls from Zapier: ~$5–15/mo
- Everything else already licensed or free

**Why:** myNHD's existing Claude Teams + Levitate licenses cover the heavy costs. Scope doc requires all config on myNHD-licensed accounts for seamless production migration.

**How to apply:** All tool recommendations in proposal and architecture doc should reflect this stack. Flag Canva as an assumption to confirm with Laura Kressig.
