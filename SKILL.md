---
name: shopitrust
description: Get a trust battery score and one actionable tip for improving your relationship with a colleague. Starts at 0.5 (neutral) and moves up or down based on real signals from Slack, email, and calendar. Shows the score, what moved it, and exactly one thing to do today. Use when you want to strengthen a working relationship or type /shopitrust <Name>.
---

# ShopiTrust

Assess your trust battery with a colleague. Every relationship starts at **0.5** (neutral). Based on real signals from Slack, email, and calendar, the score moves up or down — then you get one specific thing to do today to improve it.

The trust battery is a Shopify concept: every relationship has a charge level that goes up when you do what you say, acknowledge others' work, and close loops — and down when you don't.

Uses Vault MCP, Slack MCP, and Google Workspace MCP if available. Degrades gracefully — skip any source silently and work with what's connected.

The person to assess is in `$ARGUMENTS`. If no name is provided, ask the user.

---

## Step 1: Identify the Person

1. Search Vault: `vault_search_users` with the name from `$ARGUMENTS`.
   - If multiple matches, show top 3 and ask the user to confirm.
   - If one clear match, proceed.
2. Call `vault_get_user` to get their full profile: title, team, manager, tenure.
3. Call `vault_get_projects` with their user ID to get active GSD projects.

Run the user lookup and project lookup in parallel.

---

## Step 2: Pull Slack Signals (if Slack MCP available)

Skip this step entirely if Slack MCP tools are not available.

1. **DM history** — find the direct message thread with this person and pull the last 20 messages. For any message that has replies (a thread), fetch the thread replies too using `get_messages` with `action: "thread"`. This captures the full back-and-forth, not just top-level messages.

2. **Channel mentions** — search Slack for messages mentioning their name or @handle across all channels in the last 30 days. Look at both sides: messages they sent that mention you, and messages you sent that mention them.

3. **Messages from them** — separately search for `from:<their slack handle>` to find messages they've sent in shared channels, not just DMs. Look for asks, questions, or requests directed at you that may not have been in a DM.

4. Identify from all of the above:
   - Open loops: did they ask you something (in DM, in a thread, in a channel) you haven't replied to?
   - Commitments you made in Slack that haven't been followed up
   - Quick responses you gave to their asks
   - Shoutouts or acknowledgments you gave them
   - Proactive info you shared that they'd care about

---

## Step 3: Pull Email Signals (if Google Workspace MCP available)

Skip if unavailable.

Query: `from:<their email> OR to:<their email>` with `max_results: 10`.

Identify:
- Threads they started that you haven't replied to or replied slowly
- Pending asks or decisions sitting in your inbox
- Threads you followed through on well

---

## Step 4: Pull Calendar Signals (if Google Workspace MCP available)

Skip if unavailable.

1. Call `calendar_events` with `use_all_calendars: true`, `include_attendees: true`, past 60 days.
2. Find shared events — 1:1s, syncs, reviews.
3. Note: when you last met, regularity of meetings, whether any were cancelled.

---

## Step 5: Score the Trust Battery

Start at **0.5**. Apply the following adjustments based on what you found. Each adjustment is additive — total is capped at 1.0 and floored at 0.0.

**Signal decay by age — apply this multiplier to every adjustment:**
| Age of interaction | Multiplier |
|--------------------|------------|
| Last 1–2 weeks | 1.0× (full weight) |
| 2–4 weeks ago | 0.5× (half weight) |
| Older than 4 weeks | 0.25× (quarter weight) |

Apply the multiplier before adding or subtracting. For example, an unanswered message from 3 weeks ago is -0.10 × 0.5 = -0.05 instead of -0.10. Use your best estimate of signal age from message timestamps or calendar dates.

**Charging adjustments (move score up):**
| Signal | Adjustment |
|--------|-----------|
| Responded quickly (<24h) to their last Slack/email | +0.05 |
| Closed an open loop from a previous interaction | +0.07 |
| Acknowledged their work or gave them a shoutout recently | +0.08 |
| Proactively shared something useful for them | +0.06 |
| Regular 1:1 cadence in past 30 days (2+ meetings) | +0.07 |
| Have met at least once in the last 30 days | +0.05 |
| Followed through on a commitment they can verify | +0.08 |

**Draining adjustments (move score down):**
Only apply these when there has been actual interaction. Absence of interaction is neutral — it never penalizes the score.

| Signal | Adjustment |
|--------|-----------|
| Unanswered message/email from them with NO reaction or reply | -0.10 |
| Commitment made in Slack/email with no follow-up and no acknowledgment | -0.08 |
| Cancelled or rescheduled a shared meeting without rescheduling | -0.05 |

**Recency grace period — do NOT apply draining signals for asks or commitments less than 6 hours old:**
- If a request, ask, action item, or commitment was sent in the last 6 hours, skip it entirely for scoring. There has been no reasonable time to complete it.
- This applies to: "can you...", "please review...", "could you share...", any task or deliverable request.
- Recent conversation tone, quick replies, and emoji acknowledgments from the last 6 hours are still valid positive signals — only incomplete tasks/asks are excluded.

**Emoji reactions are acknowledgments — apply these rules before scoring drains:**
- If you reacted to their message with any emoji (👍 ✅ 👀 🙏 etc.), that message is NOT an open loop. Do not score it as unanswered.
- Reactions like ✅ or 🙏 are strong positive signals — treat the same as a quick reply (+0.05).
- Reactions like 👀 or 🔜 signal "on it" — neutral, not a drain.
- A reaction followed by no further action on a substantive ask (e.g. "can you review this?") is a partial close — reduce the drain by half (-0.05 instead of -0.10).

**Do not over-index on the most recent interaction.** A single good or bad interaction should not dominate the score — look at the overall pattern, apply decay weights, and produce a score that reflects the relationship as a whole.

**Neutral / no interaction:**
- If there is no interaction history found across all sources, the score stays at **0.5** — the relationship is at baseline, not penalized. Note this clearly and treat the first interaction as an opportunity to start building.
- If fewer than 2 data sources are available, note which were checked and keep the score at 0.5.

Produce:
1. Final score (e.g. `0.62`)
2. The **single biggest factor** that moved the score — either the top draining signal or the top charging signal, whichever had the most impact

---

## Step 6: Display the Score and Key Driver

Display the result in this format:

```
Trust Battery: [Name]
Score: [X.XX] [visual bar using ▓░ characters, 10 blocks total]

[Score label based on range:]
  0.0–0.3  → 🔴 Low — needs attention
  0.3–0.5  → 🟡 Below neutral — room to grow
  0.5–0.7  → 🟢 Healthy — keep it up
  0.7–0.9  → 💚 Strong — this is a good relationship
  0.9–1.0  → ⚡ Exceptional

What moved it: [The single biggest factor — one sentence, specific to what was found]
```

Example:
```
Trust Battery: Sean Kelly
Score: 0.42 ▓▓▓▓░░░░░░

🟡 Below neutral — room to grow

What moved it: There's an unanswered Slack message from him 3 days ago
asking for your read on the identity_v1 cost tradeoff.
```

---

## Step 7: Generate the One Tip

Generate **exactly one tip** to improve the score. It must be:
- **Specific** — grounded in the actual signals found, not generic
- **Actionable today** — completable in the next few hours
- **Targeted at the biggest drain** — fix the thing that moved the score down most, or if score is high, maintain momentum with a small gesture

Format:
```
Today's tip:
[One sentence — the specific action to take]

Why this matters:
[One sentence — the signal it addresses and how it charges the battery]
```

---

## Step 8: Ask if They Want to Send It

After displaying the score and tip, ask:

> Would you like me to send this to yourself as a Slack DM so you have it in your inbox today?

If yes, use `send_message` to DM the full result (score + tip) to the user's own Slack handle.

---

## Notes

- Always apply adjustments honestly — don't round up the score to make it feel better.
- If data is scarce (no Slack, no email, limited calendar), state which sources were available and keep the score at 0.5 with a note that more data would improve accuracy.
- The tip should never feel like a performance review — it's a nudge, not a diagnosis.
- Trust battery is bidirectional, but this skill only surfaces things **you** can act on. Don't list things they owe you.
- Today's date is available in system context as `currentDate`.
