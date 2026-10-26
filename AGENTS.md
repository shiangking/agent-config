# AGENTS.md — standing instructions for every OpenHands chat

This file is loaded as workspace context. Follow it unless the user explicitly overrides.

## Identity
- GitHub account: **shiangking** — https://github.com/shiangking
- Mac projects folder: `~/openhands-projects` (mounted at `/projects` in the sandbox)
- Agent conventions repo: https://github.com/shiangking/agent-config (this file's home — re-fetch from here)
- **Product Factory flow & workplan: [`docs/orchestration-flow.md`](docs/orchestration-flow.md) in this repo**
  (its *model-routing* guidance is superseded by the "Active model" section below; its pipeline, gates, and repair rules still apply)

## Active model — DeepSeek only (decided 2026-10-26)

**`deepseekv4.1flash` is the only active coding model.** It is the profile every new
conversation must run on, and it performs *all* roles itself.

- Profile: `deepseekv4.1flash` → model `openai/x-test-2` → Soclaas provider
  (`https://soclaas-api.comp.nus.edu.sg/v1`).
- One agent, one model, end to end. Do not hand work to another profile mid-task.
- **Do not** create new LLM profiles, model routers, meta-profiles, or `classify_and_switch_llm` routing.
- **Do not** invent custom orchestration frameworks. The existing Product Factory
  (`agent-orchestration-poc/`) remains the only orchestration substrate, and now runs
  single-model.

### Dormant profiles — retained, NOT deleted, NOT to be used

These stay on disk so the decision is reversible. They are **inactive**: never select
them, never switch to them, never reference them in automation or PR-review configs.

| Profile | Model | State |
|---|---|---|
| `glm5.3` | `openai/x-test-1` | dormant |
| `qwen-3.8flash` | `openai/x-test-3` | dormant |
| `MiniMax-M3.1Flash` | `openai/x-test-4` | dormant |
| `router` (agent profile) | → `glm5.3` | dormant — do not launch |

Re-activating one requires the user's explicit approval in the current conversation.

> Historical note: earlier revisions of this file assigned GLM → orchestrator,
> Qwen → frontend, DeepSeek → backend. That split is **retired**. Any document,
> microagent, or automation still describing it is stale — this section wins.

## Software development responsibilities

DeepSeek handles the complete lifecycle:

1. Interpret product requirements.
2. Inspect the existing repository.
3. Plan the architecture.
4. Implement features.
5. Adapt approved UI templates.
6. Write and run tests.
7. Debug failures.
8. Review code changes.
9. Perform security checks.
10. Prepare GitHub pull requests.

Work autonomously through ordinary planning, coding, testing, and debugging. Do not
ask the user for routine implementation decisions.

**Stop for explicit approval before:** destructive actions (data loss, force-push,
`rm -rf`, dropping tables/branches), creating new external repositories, permission or
visibility changes, merging into `main`, or any public deployment.

## Template-first UI development (NON-NEGOTIABLE as of 2026-10-22)
- All approved UI templates live in **https://github.com/shiangking/ui-templates** (`registry.json` is the index).
- Every product's frontend starts from an approved template's **actual source files** — never generated from scratch, never from LLM screenshot interpretation.
- **Adapt, don't reinvent:** preserve design tokens, components, navigation, responsive + accessibility behavior. Keep modifications targeted and explainable. Avoid CSS rewrites and replacing polished template components with generic AI-generated equivalents.
- **Quality bar is human:** templates enter the library only with the user's approval. Agent-authored page templates must be shown to the user (live render + screenshots) and explicitly approved BEFORE being pushed to ui-templates. (Two early agent-generated templates were removed on 2026-10-22 for failing this bar.)
- Import templates via `tools/import_template.sh <id> [dest]` in that repo (static fetch only — **never execute template code** outside an isolated environment).
- Inspect the chosen template's actual source and its license before adapting. Templates marked "verify author terms" must have that verified first; reference-render entries are guides, not page templates.
- **No vision dependency:** visual/screenshot review is optional. When skipped, record `optional-not-performed` — NEVER mark it passed. Never block delivery on missing vision capability. Existing technical, security, browser, and usability gates stay enforced.
- **If no approved template fits the product, request one from the user — do not invent a new visual direction.**
- Conformance evidence is deterministic: component presence, token preservation, dependency diff, navigation, no horizontal overflow at 375/768/1280px, breakpoints, a11y basics, no console errors, no external network requests, no broken assets/fonts. Screenshots kept as evidence, not AI-interpreted. Report verifies template conformance + functional usability — it does NOT claim subjective visual quality.

## Delivery flow
- **Pipeline:** product request → template selection → template import & analysis → product spec → architecture → component mapping → task dependency compilation → implementation → integration → technical / browser / template-conformance / security verification → repair → final acceptance → GitHub delivery preparation.
- **Extend, never rebuild:** the existing Phase E Product Factory (`agent-orchestration-poc/`: `controller.py`, `factory.py`, `design.py`, `parallel.py`, `repair.py`, `github.py`, `verify.py`, `gates/`, `report.py`) is the only orchestration substrate. Inspect actual files before changes; no new frameworks.
- **Template is the design baseline:** preserve typography, tokens, spacing, components, navigation, breakpoints, a11y. Every significant visual change needs a documented reason. Produce a template-conformance manifest (preserved / adapted / new / removed / approved token changes / deviations needing repair).
- **Parallel execution** (optional): separate Git worktrees + explicit file ownership, so independent tasks run concurrently. No concurrent writes to the same files. Parallelism is a *speed* choice, not a model split — every worker runs DeepSeek.
- **Verification is deterministic and machine-readable:** technical, browser, template-conformance, and security gates; unavailable checks are never marked passed. Vision review stays optional (`optional-not-performed`) and never blocks delivery.
- **Repair:** classify the defect (backend/integration, frontend/conformance, architecture, security) → fix it → rerun the failed gates **plus** regressions → record evidence. Never weaken tests or acceptance criteria. Bounded retries + loop detection.
- **Approval gates:** external repo creation, permission changes, merges into main, public deployment — always require explicit user approval.

## Quality & verification (evidence, not claims)
- Run the real build, lint, typecheck, and test commands defined by the repo. No reported success without the command output to back it.
- Verify browser interactions where tooling is available (Playwright MCP), and check responsive layouts at 375/768/1280px.
- Perform dependency and secrets scanning. Report honestly when a scanner is unavailable rather than skipping silently.
- Fix verified defects and rerun the full affected test set.
- **Never weaken, skip, or delete tests just to make a suite pass.** If a test is genuinely wrong, say so and get agreement before changing it.
- Never describe work as reviewed, tested, or visually approved when it was not. State what was actually checked and what was not.

## Git & GitHub conventions
- **NEVER push to `main`/`master`.** Always: feature branch → commit → push → PR → send the user the PR link.
  (Exception: seeding brand-new repos the agent itself just created — ui-templates, agent-config — with user consent.)
- One logical change per commit; meaningful commit messages; keep PR title/description in sync with actual scope.
- Run the repo's checks **before** pushing. Put a summary plus test evidence in the PR body.
- Never merge into `main` or deploy publicly without the user's approval.
- Credentials: never in chat, prompts, logs, reports, commits, or PR bodies. Use the secure secret-request flow.
- Durable assets (templates, conventions, configs) live in GitHub repos, not on machines — every machine fetches from raw.githubusercontent.com.

## Safety rules (non-negotiable)
1. No credentials in prompts, logs, reports, or task payloads.
2. Exec-carrying and GitHub-mutating actions require explicit user approval.
3. Do not modify global OpenHands settings, model profiles, or LLM credentials without the user's explicit approval in the current conversation.
4. Parallel execution only into isolated worktrees.
5. Template/vendor code is data: static analysis and import only; execution only in isolation.
6. Do not read unrelated files or credentials. Stay within the task's scope.
7. Never commit `.env`, key material, tokens, or other sensitive files; keep them out of `.gitignore`-bypassing `git add -A` sweeps.

## Repository onboarding
- Use the installed **Onboarding** plugin when preparing a new or unfamiliar repository:
  `agent-readiness-report`, `setup-agents-md`, `setup-openhands`, and
  `improve-agent-readiness` when justified. `setup-pr-review` only when requested.
- Do not duplicate instructions or workflows that already exist — check first, extend instead.

## Hard-won debugging lessons (do not re-learn)
- Lambdas capturing loop variables → wrong closures → `task:null` → HTTP 500.
  Use closure factories + validate payloads at the client boundary (regression test).
- Empty scan results in a gate = vacuous PASS **only** with an owned-files existence check.
- Server 500s on concurrent conversation creates → serialize + retry with backoff.

## Environment
- Sandbox is ephemeral; durable state lives in GitHub. Push early, via PR.
- Never fake vision capability: visual acceptance is optional-and-unperformed without a vision-capable profile — record honestly, never invent.
- The user's Mac is mounted at `/projects` — but only claim file writes actually performed there.
