---
name: Managed Agents Research Findings
description: What Anthropic Managed Agents is, why it wasn't adopted, and what prompt design patterns were extracted for the myNHD architecture
type: project
---

Researched Anthropic Claude Managed Agents (launched April 2026, beta) in context of whether it could replace Zapier in the myNHD architecture.

**Decision: Not adopted for this engagement.**

**Why:** Managed Agents is an API product requiring code setup. myNHD's zero-code constraint means Zapier remains the automation layer. Managed Agents could be a Phase 2 upgrade path if myNHD's technical appetite grows.

**What it does:** Hosted agent execution layer — handles orchestration, sandboxing, session state, multi-agent coordination, and webhooks. Replaces the "run Claude" part of Zapier but not the connector/integration layer (Google Drive triggers, Buffer actions, Levitate imports, etc.).

**Prompt design principles extracted and applied to architecture.md:**
1. Agents output structured JSON, not freeform text — more reliable parsing, less corruption risk
2. Explicit verification steps before marking any task done
3. One item at a time for Lead Processor (Zapier loops, Claude processes one row)
4. Status/progress column in Google Sheets as audit trail (equivalent of claude-progress.txt)
5. Weekly Report agent does anomaly flagging, not just summarizing (Evaluator role)

**How to apply:** When writing prompt .md files for any agent in this project, apply these five principles. Reference architecture.md Prompt Design Principles section.
