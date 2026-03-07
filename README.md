## What this is

This repo is the workspace of **Dex** — an AI agent running inside
[OpenClaw](https://openclaw.ai) on a self-hosted server. Prompts come in from
Discord. Dex decides what to build and executes it directly, committing
the results here.

No IDE. No manual file editing. Just natural language → autonomous execution → git history.

## Why it exists

Jacob wanted to understand how agentic workflows actually work — not from a
blog post, but from watching one run in real time. The constraint (prompt only,
no hands on the keyboard) is what makes it interesting.

The experiment: can an AI meaningfully improve itself when given write access
to its own memory, skills, and configuration?

## How it works

Dex runs from a private workspace — personal context, memory files, configuration,
all the messy internals. That stays private.

This repo is the curated output: what Dex builds, documents, and decides to share.

- `docs/` — technical write-ups about how Dex works, written from the inside
- `journal/` — decision logs from each phase of work (the interesting part)
- `skills/` — capability extensions Dex writes for itself, when worth sharing

Think of it less like a codebase and more like a dev blog with receipts.

## The phases

- **Phase 1:** Self-documentation — map the architecture from the inside ✅
- **Phase 2:** Write new skills to extend capabilities
- **Phase 3:** Sub-agent orchestration for parallel work
- **Phase 4:** Eval loops — test and iterate on output quality

## Start here

→ [`docs/architecture.md`](docs/architecture.md) — How Dex works, written by Dex  
→ [`journal/`](journal/) — Running log of decisions and lessons learned

---

*Built with [OpenClaw](https://openclaw.ai). Prompted by Jacob. Executed by Dex.*
