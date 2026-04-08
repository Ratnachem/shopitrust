# ShopiTrust

A Claude Code skill that gives you one specific, actionable daily hint for improving your trust battery with a colleague — grounded in real context from Slack, email, and calendar. Not generic advice. Actual open loops and relationship signals from your working history with them.

## What is a trust battery?

A Shopify concept: every working relationship has a charge level. It goes up when you do what you say, respond quickly, give credit, and close loops. It goes down when you don't. This skill surfaces where yours might be draining — and gives you one thing to act on today.

## What it does

1. Looks up the person in Vault (name, team, active projects)
2. Pulls your Slack DM history and mentions of them (last 30 days)
3. Pulls recent email threads to/from them
4. Checks calendar for when you last met and what's coming up
5. Identifies open loops, long gaps, unacknowledged wins, and positive signals
6. Generates **exactly one hint** — specific and actionable for today
7. Optionally sends the hint as a Slack DM to yourself

## Requirements

| MCP | Required? | Used for |
|-----|-----------|----------|
| [Vault MCP](https://vault.shopify.io/ai/mcp-servers) | Required | Person lookup, team context, active projects |
| Slack MCP | Recommended | DM history, channel mentions, open loops |
| Google Workspace MCP | Optional | Email threads, calendar history |

The skill degrades gracefully — if Slack or Google Workspace MCPs aren't connected, it skips those sources and works with what's available.

## Installation

### 1. Copy the skill to your Claude skills folder

```bash
git clone https://github.com/Ratnachem/shopitrust.git ~/.claude/skills/shopitrust
```

Or manually:
```bash
mkdir -p ~/.claude/skills/shopitrust
curl -o ~/.claude/skills/shopitrust/SKILL.md \
  https://raw.githubusercontent.com/Ratnachem/shopitrust/main/SKILL.md
```

### 2. Restart Claude Code

The skill is picked up automatically on next session start.

### 3. Use it

```
/shopitrust Sean Kelly
/shopitrust Sneha Shah
/shopitrust          ← will ask you for a name
```

## Example output

```
Your trust hint for Sean Kelly today:
Reply to his Slack message from 3 days ago about the identity_v1 model 
cost tradeoff — he asked for your read and hasn't heard back.

Why: Open loops are trust battery drains, and this one has been sitting 
for 3 days.
```

## Contributing

This is a personal skill built for Shopify's internal tooling stack. PRs welcome — especially improvements to the trust signal scoring logic or hint generation quality.

## Credits

Built by [@Ratnachem](https://github.com/Ratnachem). Inspired by the trust battery concept from Tobi Lütke.
