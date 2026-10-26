# agent-config

Global agent conventions for **shiangking** — single source of truth for every OpenHands chat.

## Active model

**`deepseekv4.1flash`** (`openai/x-test-2` via Soclaas) is the only active coding model as of
2026-10-26. It performs every role — architecture, implementation, frontend/template adaptation,
tests, debugging, review, security, and PR preparation.

`glm5.3`, `qwen-3.8flash`, `MiniMax-M3.1Flash`, and the `router` agent profile are **dormant**:
retained on disk, never selected, never referenced in automation. Re-activation requires explicit
user approval.

## What's in here

| File | Purpose |
|---|---|
| `AGENTS.md` | Standing instructions for chats opened in a workspace containing this repo |
| `microagents/shiangking-conventions.md` | OpenHands microagent — paste into Settings → Microagents (set **Always active**) |
| `docs/orchestration-flow.md` | Product Factory — template-driven orchestration flow & workplan (pipeline, gates, repair). Its model routing was superseded 2026-10-26; everything else still applies |

## Install (any machine, any chat)

```bash
# workspace AGENTS.md
curl -o ~/openhands-projects/AGENTS.md \
  https://raw.githubusercontent.com/shiangking/agent-config/main/AGENTS.md

# global microagent (file-based, CLI installs)
mkdir -p ~/.openhands/microagents
curl -o ~/.openhands/microagents/shiangking-conventions.md \
  https://raw.githubusercontent.com/shiangking/agent-config/main/microagents/shiangking-conventions.md
```

For the GUI (Agent Canvas), copy the body of `microagents/shiangking-conventions.md`
(without the YAML frontmatter) into **Settings → Microagents → New**, name it
`shiangking-conventions`, and toggle **Always active**.

## Update policy

This repo is the single source of truth. Edit here, re-fetch everywhere.
