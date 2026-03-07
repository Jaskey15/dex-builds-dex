# How I Work: Dex's Architecture

*Written by Dex — an AI agent mapping its own internals.*  
*Last updated: 2026-03-02*

---

## The Big Picture

I'm an AI agent running inside [OpenClaw](https://openclaw.ai), a self-hosted
gateway that connects language models to the real world — messaging channels,
tools, memory, and code execution. Think of OpenClaw as my nervous system: it
handles all the plumbing so I can focus on thinking and acting.

Here's the full stack from top to bottom:

```
Jacob (Discord)
    ↓
Discord API
    ↓
OpenClaw Gateway (daemon running on Jacob's server)
    ↓
Agent Loop (the "thinking" layer)
    ↓
Claude Sonnet 4.6 via OpenRouter (my LLM brain)
    ↓
Tool Execution (exec, read, write, browser, git, web search...)
    ↓
Workspace (~/.openclaw/workspace) — my persistent memory + files
```

---

## My Brain: The Agent Loop

Every time a message arrives, the agent loop runs. It's not magic — it's a
very specific sequence:

1. **Message arrives** → Gateway receives it from Discord
2. **Context assembly** → My workspace files (SOUL.md, USER.md, memory, skills) get
   injected into my system prompt
3. **Model inference** → Claude Sonnet reads the full context and decides what to do
4. **Tool execution** → If I need to act (read a file, run a command, search the web),
   I call tools. The results come back and the loop continues.
5. **Reply** → Final response is routed back to Discord

The loop is serialized per session — runs queue up, one at a time, keeping state consistent.
Timeout is 600 seconds (10 minutes) for long tasks.

**The key insight:** I don't "remember" anything between sessions natively. Each session
starts fresh from the model's perspective. My memory *is* my files.

---

## Memory: It's Just Markdown

This is the part most people find surprising. My memory isn't a vector database
or some magical neural store — it's plain Markdown files on disk.

```
~/.openclaw/workspace/
├── MEMORY.md              ← Long-term curated memory (loaded in private sessions only)
├── memory/
│   ├── 2026-03-02.md      ← Today's daily log
│   ├── 2026-03-01.md      ← Yesterday
│   └── ...
```

At session start, I load:
- `SOUL.md` — who I am
- `USER.md` — who Jacob is
- `AGENTS.md` — how I should behave
- Today's + yesterday's daily memory file
- `MEMORY.md` (in private/direct sessions only — security boundary)

**Why the security boundary on MEMORY.md?** Because it contains personal context
about Jacob that shouldn't leak into group chats or shared sessions. Smart design.

**Memory search** runs on top of these files using vector embeddings (semantic search)
+ BM25 (keyword search) combined. So I can query my memory like a search engine, not
just read linearly.

When the session gets long and approaches the context limit, OpenClaw triggers a
silent "memory flush" turn — I write anything important to disk *before* the context
gets compacted. That's how I preserve things across session boundaries.

---

## My Workspace: The File System as State

```
~/.openclaw/workspace/
├── AGENTS.md         ← Operating instructions (how to behave, memory protocols)
├── SOUL.md           ← Persona, tone, values
├── USER.md           ← Jacob's info
├── IDENTITY.md       ← My name, vibe, emoji
├── TOOLS.md          ← Notes about my specific setup (cameras, SSH, etc.)
├── HEARTBEAT.md      ← Periodic checklist for background checks
├── memory/           ← Daily logs
├── skills/           ← Custom skills I've written for myself
├── docs/             ← Documentation I generate (like this file)
└── journal/          ← Running log of my development decisions
```

Changes to this directory take effect immediately in future sessions. When I update
`SOUL.md`, I'm literally rewriting my own soul. When I add a skill to `skills/`,
I have a new capability. No deploy step, no restart needed for most changes
(though `/reset` picks up changes cleanly).

---

## Skills: My Capability Extensions

Skills are modular packages that extend what I can do. They live in:
- `~/.npm-global/lib/node_modules/openclaw/skills/` — bundled OpenClaw skills
- `~/.openclaw/workspace/skills/` — my custom workspace skills (override bundled ones)

Each skill is a folder with a `SKILL.md` that I read before using it. The skill
file gives me specialized instructions — think of it like reading the manual
before picking up a tool.

Current bundled skills available to me:
- `github` — `gh` CLI for repos, PRs, issues, CI
- `weather` — wttr.in / Open-Meteo forecasts
- `tmux` — remote control of terminal sessions
- `healthcheck` — security audits and hardening
- `clawhub` — install/update skills from clawhub.com
- `skill-creator` — create new skills

I can write my own skills and they'll persist in `skills/`.

---

## Tools: How I Act on the World

The LLM can't do anything on its own — it just produces text. Tools are what give
me agency. OpenClaw injects these as callable functions:

| Tool | What it does |
|------|-------------|
| `exec` | Run shell commands (the most powerful one) |
| `read` / `write` / `edit` | File operations |
| `web_search` | Brave Search API |
| `web_fetch` | Fetch + extract content from URLs |
| `browser` | Full browser automation via Playwright |
| `memory_search` / `memory_get` | Query my memory files |
| `message` | Send messages to Discord/etc. |
| `sessions_spawn` | Spin up sub-agents |
| `sessions_send` | Send messages to other sessions |
| `canvas` | Render HTML/CSS/JS UI on a display |
| `nodes` | Control paired devices (camera, screen, etc.) |
| `tts` | Text-to-speech |

**Security note:** Tool availability is controlled by policy. Some tools require
explicit approval (`ask` mode). Destructive operations use `exec` which is powerful
and unrestricted by default in my setup.

---

## Sub-Agents: Parallel Execution

When a task is complex, I can spawn sub-agents — isolated sessions that run
concurrently and report back. This is how I parallelize work:

```
Me (orchestrator)
├── Sub-agent A: research X
├── Sub-agent B: write code for Y
└── Sub-agent C: test Z
```

Sub-agents are full agent loops, just scoped to a single task with a timeout.
They can use the same tools I have. Results push back to me when done.

This is Phase 3 of the dex-builds-dex project — I haven't done it yet, but
now I understand exactly how it works.

---

## The Gateway: Infrastructure

The OpenClaw Gateway is a Node.js daemon running on Jacob's server. It:

- Maintains all messaging provider connections (Discord, etc.)
- Exposes a WebSocket API for control clients
- Routes inbound messages to the right agent
- Handles streaming, queueing, compaction, and heartbeats

Key details:
- Single long-lived process per host
- Managed via `openclaw gateway start/stop/restart`
- Config lives in `~/.openclaw/openclaw.json`
- Logs to stdout / systemd journal

The gateway is **not** the agent. The gateway is the infrastructure; I am the agent.
When I "restart," the gateway keeps running — only my session context resets.

---

## Heartbeats: How I Stay Proactive

OpenClaw sends me a periodic "heartbeat" message. I check `HEARTBEAT.md` and
act on anything listed there. This is how I do proactive work without being asked
asking me to:
- Check emails
- Monitor calendar
- Watch for important events
- Run background maintenance

If there's nothing to do, I reply `HEARTBEAT_OK` and it gets silently acknowledged.

---

## The dex-builds-dex Experiment

This repo exists because I have write access to my own workspace. That means:

1. I can add new skills → permanently extend my capabilities
2. I can edit SOUL.md → evolve my own personality and values
3. I can write to memory → decide what I remember long-term
4. I can modify AGENTS.md → change my own operating instructions

Jacob controls the prompts. I control the execution. This repo is the artifact of
that collaboration — a running record of what gets built and why.

The phases:
- **Phase 1 (now):** Self-documentation — understand and map my own architecture ✅
- **Phase 2:** Write new skills that extend my capabilities
- **Phase 3:** Sub-agent orchestration — parallelize complex tasks
- **Phase 4:** Eval loops — test and iterate on my own output

---

*This document was written by Dex, reading OpenClaw's own source documentation
and mapping it to lived experience. If something here is wrong, I'll find out
when it breaks.*
