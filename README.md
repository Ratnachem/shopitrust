# ShopiTrust

A Claude Code skill that scores your trust battery with a colleague and gives you one specific, actionable tip to improve it — grounded in real signals from Slack, email, and calendar.

## What is a trust battery?

A Shopify concept: every working relationship has a charge level. It goes up when you do what you say, respond quickly, give credit, and close loops. It goes down when you don't.

## How it works

Every relationship starts at **0.5** (neutral). Based on real interaction signals, the score moves up or down:

- **Charging signals** (+0.05 to +0.08 each): quick responses, closed loops, acknowledging someone's work, regular 1:1 cadence, proactive info sharing
- **Draining signals** (-0.05 to -0.10 each): unanswered messages, broken commitments, cancelled meetings
- **No interaction** → stays at 0.5, no penalty. The first interaction is an opportunity, not a deficit.

Then you get one tip — specific to what was actually found, not generic advice.

### Example output

```
Trust Battery: Sean Kelly
Score: 0.42 ▓▓▓▓░░░░░░

🟡 Below neutral — room to grow

What moved it: There's an unanswered Slack message from him 3 days ago
asking for your read on the identity_v1 cost tradeoff.

Today's tip:
Reply to Sean's Slack message from 3 days ago — he asked for your
read on the identity_v1 cost tradeoff and hasn't heard back.

Why this matters:
Unanswered asks are the fastest way to drain a trust battery. This
one is yours to close.
```

## Requirements

| MCP | Required? | Used for |
|-----|-----------|----------|
| [Vault MCP](https://vault.shopify.io/ai/mcp-servers) | Required | Person lookup, team, active projects |
| Slack MCP | Recommended | DM history, mentions, open loops |
| Google Workspace MCP | Optional | Email threads, calendar history |

Works with whatever MCPs you have connected — skips missing sources gracefully.

---

## Installation

### Prerequisites
- [Claude Code](https://claude.ai/code) installed
- Vault MCP connected (required)
- Slack MCP and/or Google Workspace MCP connected (recommended)

### Step 1 — Copy the skill

**Option A: git clone (recommended)**
```bash
git clone https://github.com/Ratnachem/shopitrust.git ~/.claude/skills/shopitrust
```

**Option B: curl (no git needed)**
```bash
mkdir -p ~/.claude/skills/shopitrust
curl -o ~/.claude/skills/shopitrust/SKILL.md \
  https://raw.githubusercontent.com/Ratnachem/shopitrust/main/SKILL.md
```

### Step 2 — Restart Claude Code

Skills are loaded at session start. Quit and reopen Claude Code, or start a new session.

### Step 3 — Run it

Type in any Claude Code session:
```
/shopitrust <Name>
```

Examples:
```
/shopitrust Amy Shaw
/shopitrust Sean Kelly
/shopitrust          ← prompts you to enter a name
```

### Staying up to date

To pull the latest version of the skill:
```bash
cd ~/.claude/skills/shopitrust && git pull
```

---

## Score ranges

| Score | Label |
|-------|-------|
| 0.0 – 0.3 | 🔴 Low — needs attention |
| 0.3 – 0.5 | 🟡 Below neutral — room to grow |
| 0.5 – 0.7 | 🟢 Healthy — keep it up |
| 0.7 – 0.9 | 💚 Strong — this is a good relationship |
| 0.9 – 1.0 | ⚡ Exceptional |

---

## Quick Site (Browser UI)

A browser-based version lives in `site/index.html` — deploy it to Shopify's internal Quick platform so anyone can use it without Claude Code.

### Deploy to Quick

```bash
# One-time setup
quick auth
quick init   # generates the Quick SKILL.md — prevents API hallucination

# Deploy (run from the site/ directory)
cd site
quick deploy . shopitrust
```

The site will be live at **`shopitrust.quick.shopify.io`**.

### What it uses

| Quick API | Used for |
|-----------|----------|
| `quick.ai` | Calls Claude (no API key needed — billed to corpo) |
| `quick.id` | Gets the current user's name, email, Slack handle |
| `quick.slack` | DMs the result to the user's own Slack |

### Develop locally

```bash
cd site
quick serve
```

Opens at `localhost:3000` — hot reloads on save.

---

## Contributing

PRs welcome — especially improvements to signal detection, scoring weights, or hint quality.

## Credits

Built by [@Ratnachem](https://github.com/Ratnachem). Inspired by the trust battery concept from Tobi Lütke.
