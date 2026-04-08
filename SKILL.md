---
name: shopitrust
description: Get a single, specific, actionable daily hint for improving your trust battery with a colleague. Pulls context from Slack DMs, email threads, calendar history, and Vault projects to surface real open loops and relationship signals — not generic advice. Use when you want to strengthen a working relationship or type /shopitrust <Name>.
---

# ShopiTrust

Get a daily trust battery hint for a specific colleague — grounded in real context from Slack, email, calendar, and their active work. One actionable thing you can do today.

The trust battery is a Shopify concept: every relationship has a charge level that goes up when you do what you say, acknowledge others' work, and close loops — and down when you don't. This skill surfaces where your battery might be draining and gives you one specific thing to act on today.

Uses Vault MCP, Slack MCP, and Google Workspace MCP if available. Degrades gracefully if any source is unavailable — skip it silently and work with what's connected.

The person to assess is in `$ARGUMENTS`. If no name is provided, ask the user.

---

## Step 1: Identify the Person

1. Search Vault: `vault_search_users` with the name from `$ARGUMENTS`.
   - If multiple matches, show top 3 and ask the user to confirm.
   - If one clear match, proceed.
2. Call `vault_get_user` to get their full profile:
   - Title, team, manager, tenure
3. Call `vault_get_projects` with their user ID to get active GSD projects.

Run the user lookup and project lookup in parallel.

---

## Step 2: Pull Slack Signals (if Slack MCP available)

Skip this step entirely if Slack MCP tools are not available.

1. **DM history** — search for direct message threads with this person. Pull the last 20 messages.
2. **Mentions** — search Slack for messages mentioning their name or handle in the last 30 days.
3. **Identify open loops**:
   - Did they ask you something you haven't replied to?
   - Did you ask them something still unanswered (note: this is on them, not you)?
   - Any commitments made in Slack that haven't been followed up on?
4. **Identify positive signals**:
   - Did you acknowledge their work or give them a shoutout recently?
   - Did you respond quickly to their asks?
   - Did you share credit or loop them into relevant conversations?

---

## Step 3: Pull Email Signals (if Google Workspace MCP available)

Skip if unavailable.

Query: `from:<their email> OR to:<their email>` with `max_results: 10`.

Look for:
- Threads you started but haven't followed up on
- Threads they started that you replied slowly or not at all
- Any pending asks or decisions sitting in email

---

## Step 4: Pull Calendar Signals (if Google Workspace MCP available)

Skip if unavailable.

1. Call `calendar_events` with `use_all_calendars: true`, `include_attendees: true`, past 60 days.
2. Find shared events — 1:1s, syncs, reviews they were on.
3. Note:
   - When you last met
   - Whether 1:1s are regular or sporadic
   - Any upcoming meeting coming up soon

---

## Step 5: Score the Trust Battery

Based on all signals collected, categorize what you found:

**Draining signals (lower trust):**
- Unanswered Slack messages or emails from them (open loops you own)
- Commitments made and not followed up
- Long gaps since last meaningful interaction (>3 weeks with no async either)
- Meetings cancelled or rescheduled repeatedly
- No acknowledgment of their recent work or wins

**Charging signals (higher trust):**
- Quick response times
- Followed through on past commitments
- Gave them credit or acknowledged their work
- Proactively shared information they'd care about
- Regular 1:1 cadence maintained

**Neutral / insufficient data:**
- No recent interaction found
- MCP sources not available

Produce a brief internal summary: `[DRAINING: <list>] [CHARGING: <list>]`

---

## Step 6: Generate the Daily Hint

Generate **exactly one hint**. It must be:
- **Specific** — grounded in actual context found, not generic advice
- **Actionable today** — something they can do in the next few hours
- **Proportionate** — if the battery looks healthy, the hint should reflect that (a small gesture, not an intervention)

Hint format:
> **Your trust hint for [Name] today:**
> [One sentence describing the specific action]
>
> **Why:** [One sentence grounding it in what you found — the open loop, the gap, the win to acknowledge]

Examples of good hints:
> **Your trust hint for Sean Kelly today:**
> Reply to his Slack message from 3 days ago about the identity_v1 model cost tradeoff — he asked for your read and hasn't heard back.
>
> **Why:** Open loops are trust battery drains, and this one has been sitting for 3 days.

> **Your trust hint for Sneha Shah today:**
> Send her a quick note acknowledging the Human Detection 2026 Scope milestone — she just passed week 10 of 46 and it's a meaty project.
>
> **Why:** Recognizing ongoing hard work charges the battery, especially mid-project when there's no visible finish line yet.

> **Your trust hint for [Name] today:**
> Your trust battery with them looks healthy — your last interaction was recent and responsive. Keep the cadence going: check if there's anything they need from you before your next 1:1.
>
> **Why:** Batteries drain passively over time. A small proactive check-in is worth more than a big gesture later.

---

## Step 7: Ask if They Want to Send It

After displaying the hint, ask:

> Would you like me to send this hint as a Slack DM to yourself so you have it in your inbox today?

If yes, use `send_message` to DM the hint to the user's own Slack handle.

---

## Notes

- Always generate exactly one hint — not a list, not multiple options.
- If data is scarce (no Slack, no email, limited calendar), say so and generate a conservative hint based on what little is known about their role and projects.
- The hint should never feel like a performance review — it's a nudge, not a diagnosis.
- Trust battery is bidirectional, but this skill only surfaces things **you** can act on. Don't list things they owe you.
- Today's date is available in system context as `currentDate`.
