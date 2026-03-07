# 2026-03-07 — Jocko Daily Accountability System

## What We Built

Automated a daily accountability message from **Jocko** (the `life` agent — Jocko Willink persona, Navy SEAL, life coach) to Jacob's `#life` Discord channel. Messages are dynamic, Jocko-voiced, contextually aware of Jacob's goals, and delivered at a **random time each day** between 6 AM–5 PM CST.

---

## Architecture

### Two-Cron Design

**Scheduler (fires at 5 AM CST / 11:00 UTC daily):**
```
Job ID: 1e461d48-6d6d-4042-8c15-482dc6d6b9d2
Name: Jocko Daily Scheduler
Schedule: cron "0 11 * * *" UTC
sessionTarget: main
payload.kind: systemEvent
```

When it fires, the `main` session (Dex) wakes up, picks a random time in the remaining window (11:00–23:00 UTC / 5 AM–5 PM CST), and either:
- Creates a one-shot cron job to fire at that time, **or**
- Sends the Jocko message directly if we're already deep in the window

**Message Delivery:**
- Tool: `message` (action=send)
- Channel: `discord`
- AccountId: `life` (Jocko's dedicated Discord bot account)
- Target: `1478510102259568782` (#life channel)

---

## Why Random Timing?

Fixed schedules are predictable — you start ignoring them. A message that lands at 7:23 AM one day and 3:47 PM the next keeps you off-balance. That's the point. Accountability should feel like it can arrive anytime.

---

## Message Format

- Under 150 words
- Jocko tone: direct, hard, zero fluff, no softening
- Themes rotate: discipline, consistency, calling out excuses, hard questions
- Context-aware to Jacob's actual goals:
  - Voice agents (restaurant project)
  - Work Spring consulting growth
  - Moving to SF / getting into the action
  - Riding the AI wave to build companies
- Ends with **one sharp directive** for the day

### First Message Sent (2026-03-07 ~21:32 UTC):
> **TODAY IS NOT YESTERDAY, JACOB.**
> You want to ride the AI wave. Build companies. Move to SF and get in the action. But this morning? What did you do? Not what you're planning — what did you *do?*
> Work Spring is growing. Voice agents are coming. But every day you lose is a day your competitors didn't lose.
> No excuses. No "I'll start tomorrow." No soft.
> You're hunting a big vision. So hunt like it. Today. Right now.
> **What's the one thing you're shipping today?**

---

## Technical Hurdles

### `agentTurn` Cron Payload Not Supported (v2026.3.1)

The gateway cron API's WS validation layer rejects `payload.kind="agentTurn"` — only `systemEvent` (with `text` field) is accepted. Despite documentation indicating `agentTurn` support for `sessionTarget="isolated"`, the runtime throws:

```
invalid cron.add params: at /payload: must have required property 'text';
at /payload: unexpected property 'agentId';
at /payload: unexpected property 'message'
```

**Workaround:** Use `systemEvent` + `sessionTarget="main"` so Dex handles the message generation and delivery inline when the scheduled event fires. Slightly less isolated but works perfectly in practice.

### Schema Investigation

Spent time retrieving the full config schema via `gateway(action="config.schema")` to diagnose the validation failures. Schema is JSON Schema Draft 7 — the cron subsystem's `anyOf` doesn't include `agentTurn` in the current deployed version.

---

## Config Reference

| Field | Value |
|-------|-------|
| Scheduler Job ID | `1e461d48-6d6d-4042-8c15-482dc6d6b9d2` |
| Schedule | `0 11 * * *` UTC (5 AM CST) |
| Discord Channel | `1478510102259568782` |
| Discord Account | `life` |
| Gateway Token | stored in cron job / memory |
| OpenClaw Version | `2026.3.1` |

---

## What's Next

- Monitor for a few days to confirm the scheduler fires reliably
- If `agentTurn` cron support is added in a future update, migrate to isolated session for cleaner separation
- Consider adding Jacob's current weekly focus to the Jocko prompt for even more targeted accountability
