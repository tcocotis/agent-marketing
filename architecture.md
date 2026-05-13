# Solution Architecture
## myNHD AI-Assisted Marketing Programs
**Version 1.0 | May 2026**

---

## Overview

This document describes the zero-code, Claude-agent based solution architecture for two independent AI-assisted marketing programs at myNHD. Both programs leverage myNHD's existing Claude Teams and Levitate licenses, with Zapier as the automation layer connecting all components.

**Core design principles:**
- Zero custom code — all configuration, no programming
- Claude agents handle all reasoning, adaptation, classification, and reporting
- Zapier handles all workflow orchestration and tool connectivity
- Prompts stored as `.md` files in GitHub for version control and easy updates
- Human review gates at all critical points
- All tooling configured on myNHD-licensed accounts for seamless production migration

---

## Tool Stack

| Role | Tool | License |
|---|---|---|
| AI Agent (reasoning, adaptation, classification) | Claude (Anthropic) | myNHD Claude Teams |
| Workflow automation & orchestration | Zapier | New — ~$20/mo |
| Outbound email execution | Levitate | myNHD existing license |
| Social media scheduling & publishing | Buffer | Free tier |
| Graphic adaptation | Canva | To confirm with Laura Kressig |
| Asset & data handoff | Google Drive / Google Sheets | Standard |
| Prompt & workflow version control | GitHub | tcocotis/agent-marketing |

**Estimated net new cost to myNHD: ~$25–35/mo**

---

## Program 1: AI-Assisted Social Media

### Objective
Execute myNHD's social media program (~20 posts/month) across Instagram, LinkedIn, and Facebook using AI-assisted content adaptation and automated scheduling. myNHD provides all content, messaging, and graphics.

### Workflow

```
1. Laura Kressig drops content brief + graphic assets
   → Google Drive (designated drop folder)

2. Zapier detects new file
   → Triggers Claude Agent (Content Adapter)

3. Claude Agent — Content Adapter
   → Reads content brief
   → Drafts platform-specific captions for Instagram, LinkedIn, Facebook
   → Notes any graphic adaptation required per platform
   → Outputs scheduling queue to Google Sheet

4. Graphic adaptation
   → Canva templates resize assets to platform specs
   → (Instagram: 1080x1080 / 1080x1350, LinkedIn: 1200x627, Facebook: 1200x630)

5. Human review gate
   → Laura reviews captions and adapted graphics in Google Sheet
   → Approves or edits before scheduling

6. Zapier pushes approved posts → Buffer
   → Posts scheduled per content calendar

7. Buffer publishes to Instagram, LinkedIn, Facebook
   → On scheduled date and time
```

### Claude Agent: Content Adapter
- **Prompt file:** `prompts/program1-content-adapter.md`
- **Input:** Content brief (topic, key message, tone notes, links) — one brief per invocation
- **Output:** Structured JSON with three platform-specific captions, hashtag arrays, character counts, and graphic adaptation flags
- **Model:** Claude Sonnet (via Zapier Anthropic module)
- **Verification step:** Agent confirms character limits and tone match before outputting
- **Status output:** Writes result status to Google Sheet ("Processed" or "Flagged: [reason]")

### Key Assumptions
- myNHD uses Canva for graphic creation (to confirm with Laura Kressig)
- Laura provides briefs via Google Drive drop folder
- Buffer free tier sufficient for POC (3 channels, scheduling queue)

---

## Program 2: AI-Assisted Commercial Outreach Email Campaign

### Objective
Establish an automated outbound email campaign targeting commercial real estate agents and brokers in California to drive awareness and orders for the myNHD Commercial NHD Report. Daily leads sourced from CoStar exports.

### Workflow

```
1. myNHD team member exports new commercial listings from CoStar daily
   → Manually compiles agent/broker email addresses
   → Uploads completed lead list to Google Sheet (daily intake tab)

2. Zapier detects new rows in Google Sheet
   → Triggers Claude Agent (Lead Processor)

3. Claude Agent — Lead Processor
   → Parses and validates each lead record
   → Personalizes outreach context using listing details (property type, location, listing date)
   → Flags any data quality issues (missing email, incomplete record)
   → Formats leads for Levitate import

4. Zapier pushes formatted leads → Levitate
   → Lead added to outbound sequence
   → myNHD-provided email content delivered per sequence cadence

5. Levitate executes email sequence
   → Day 1: Introduction + Commercial NHD Report overview
   → Day 3: Follow-up + ordering information
   → Day 7: Final follow-up + support/conversation offer
   → Tracks delivery, opens, clicks, replies

6. Zapier monitors Levitate for replies and engagement
   → Triggers Claude Agent (Reply Classifier)

7. Claude Agent — Reply Classifier
   → Reads incoming reply
   → Classifies: Hot Lead / Interested / Not Interested / Question / Out of Office
   → Hot Lead → Zapier sends alert to Steve Schattmaier (email or Slack)
   → Question → Claude drafts suggested response for human review
   → Not Interested / OOO → logged, sequence paused

8. Claude Agent — Weekly Report
   → Pulls campaign metrics from Levitate
   → Compiles: delivery rate, open rate, CTR, reply rate, orders, escalations
   → Outputs formatted report to Google Sheet and/or email to stakeholders
```

### Claude Agents

**Lead Processor**
- **Prompt file:** `prompts/program2-lead-processor.md`
- **Input:** One CoStar lead row per invocation (property address, type, listing date, agent name, email)
- **Output:** Structured JSON — validated, formatted lead record with personalization context
- **Model:** Claude Haiku (cost-efficient for high-volume processing)
- **Verification step:** Agent confirms all required fields are present and non-empty before flagging as processed
- **Status output:** Writes to Google Sheet ("Processed," "Flagged: missing email," "Flagged: incomplete record")

**Reply Classifier**
- **Prompt file:** `prompts/program2-reply-classifier.md`
- **Input:** One incoming email reply per invocation
- **Output:** Structured JSON — classification label, routing instruction, optional draft response
- **Model:** Claude Sonnet
- **Verification step:** Agent re-reads classification against original reply text before outputting

**Weekly Report**
- **Prompt file:** `prompts/program2-weekly-report.md`
- **Input:** Campaign metrics data from Levitate (full week)
- **Output:** Anomaly flags section (top) + formatted performance summary with commentary
- **Model:** Claude Sonnet
- **Evaluator role:** Before summarizing, agent checks for: missing classifications, reply rate drops >20% WoW, leads with no status update, delivery rate below threshold

### Key Assumptions
- Levitate integrates with Zapier (confirmed — Zapier has native Levitate connector)
- myNHD-provided email content loaded into Levitate sequences before POC launch
- Steve Schattmaier receives hot lead alerts via email (Slack optional)
- Daily CoStar export volume: TBD (to confirm with Steve Schattmaier)

---

## Shared Infrastructure

### Prompt Design Principles

These principles apply to all Claude agent prompt files in this architecture, derived from Anthropic's Managed Agents research (April 2026) on reliable long-running agent behavior.

**1. Structured JSON outputs**
All agents output structured JSON rather than freeform text. JSON is less likely to be corrupted or misformatted by Claude across repeated runs, and is more reliably parsed by Zapier's data mapping layer. Prompt files should define an explicit output schema.

**2. Explicit verification before completion**
Every agent prompt includes a verification step before marking a task done. Examples:
- Lead Processor: *"Confirm all required fields are present and non-empty before flagging the record as processed."*
- Reply Classifier: *"Re-read your classification against the original reply text before outputting."*
- Content Adapter: *"Review each caption against the platform's character limit and tone guidelines before outputting."*

**3. One item at a time**
Agents process one item per invocation — one lead record, one reply, one content brief. Zapier's loop steps handle batching. This prevents context overflow on large batches and isolates failures to individual records rather than the whole run.

**4. Status and progress tracking in Google Sheets**
Every agent updates a dedicated status column in its Google Sheet on completion. Statuses should be explicit and human-readable (e.g., *"Processed," "Flagged: missing email," "Classified: Hot Lead," "Error: no reply text"*). This provides a live audit trail, allows Zapier filters to skip already-processed rows, and makes debugging straightforward.

**5. Anomaly flagging in the Weekly Report agent**
The Weekly Report agent is not just a summarizer — it acts as a quality evaluator. Before compiling metrics, it checks for anomalies: missing classifications, reply rate drops >20% week-over-week, leads with no status update, or delivery rate below threshold. Anomalies are flagged at the top of the report before the standard summary section.

---

### GitHub Repository
`tcocotis/agent-marketing`

All prompt `.md` files, workflow documentation, and handoff instructions are version-controlled here. Updating agent behavior requires only editing a prompt file — no code changes.

**Repository structure:**
```
/prompts
  program1-content-adapter.md
  program2-lead-processor.md
  program2-reply-classifier.md
  program2-weekly-report.md
/docs
  program1-workflow.md       ← Laura Kressig handoff guide
  program2-workflow.md       ← Steve Schattmaier handoff guide
  levitate-evaluation.md     ← Vendor assessment
architecture.md
```

### Human Review Gates
| Gate | Program | Who Reviews |
|---|---|---|
| Caption and graphic review before scheduling | Program 1 | Laura Kressig |
| Data quality flags before lead import | Program 2 | myNHD team member |
| Question reply drafts before sending | Program 2 | Steve Schattmaier |
| Weekly performance report | Both | Respective POC |

---

## Cost Summary

| Item | Monthly Cost |
|---|---|
| Claude Teams | Already licensed by myNHD |
| Levitate | Already licensed by myNHD |
| Zapier Starter | ~$20/mo |
| Claude API calls via Zapier (POC volume) | ~$5–15/mo |
| Buffer | Free |
| Canva | Free (or existing license) |
| Google Drive / Sheets | Free |
| **Total net new cost** | **~$25–35/mo** |

---

## Open Questions

1. Does myNHD use Canva for graphic creation? *(Confirm with Laura Kressig)*
2. What is the typical daily CoStar export volume (leads per day)? *(Confirm with Steve Schattmaier)*
3. Does Steve Schattmaier prefer email or Slack for hot lead escalation alerts?
4. What email sequences has myNHD already prepared for Levitate?

---

*Document maintained by consultant. Last updated May 2026.*
