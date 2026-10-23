# AGENTS.md — standing instructions for every OpenHands chat

This file is loaded as workspace context. Follow it unless the user explicitly overrides.

## Identity
- GitHub account: **shiangking** — https://github.com/shiangking
- Mac projects folder: `~/openhands-projects` (mounted at `/projects` in the sandbox)
- Agent conventions repo: https://github.com/shiangking/agent-config (this file's home — re-fetch from here)
- **Product Factory flow & workplan (authoritative): [`docs/orchestration-flow.md`](docs/orchestration-flow.md) in this repo**

## Template-first UI development (NON-NEGOTIABLE as of 2026-10-22)
- All approved UI templates live in **https://github.com/shiangking/ui-templates** (`registry.json` is the index).
- Every product's frontend starts from an approved template's **actual source files** — never generated from scratch, never from LLM screenshot interpretation.
- **Adapt, don't reinvent:** preserve design tokens, components, navigation, responsive + accessibility behavior. Keep modifications targeted and explainable. Avoid CSS rewrites and replacing polished template components with generic AI-generated equivalents.
- **Quality bar is human:** templates enter the library only with the user's approval. Agent-authored page templates must be shown to the user (live render + screenshots) and explicitly approved BEFORE being pushed to ui-templates. (Two early agent-generated templates were removed on 2026-10-22 for failing this bar.)
- Import templates via `tools/import_template.sh <id> [dest]` in that repo (static fetch only — **never execute template code** outside an isolated environment).
- **No vision dependency:** visual/screenshot review is optional. When skipped, record `optional-not-performed` — NEVER mark it passed. Never block delivery on missing vision capability. Existing technical, security, browser, and usability gates stay enforced.
- **If no approved template fits the product, request one from the user — do not invent a new visual direction.**
- Conformance evidence is deterministic: component presence, token preservation, dependency diff, navigation, no horizontal overflow at 375/768/1280px, breakpoints, a11y basics, no console errors, no external network requests, no broken assets/fonts. Screenshots kept as evidence, not AI-interpreted. Report verifies template conformance + functional usability — it does NOT claim subjective visual quality.

## Orchestration flow — template-driven Product Factory (supersedes prior in-file guidance)
- **Authoritative flow & workplan:** [`docs/orchestration-flow.md`](docs/orchestration-flow.md). Read it before any factory/orchestration work.
- **Extend, never rebuild:** the existing Phase E Product Factory (`agent-orchestration-poc/`: `controller.py`, `factory.py`, `design.py`, `parallel.py`, `repair.py`, `github.py`, `verify.py`, `gates/`, `report.py`) is the only orchestration substrate. Inspect actual files before changes; no new frameworks.
- **Pipeline:** product request → template selection → template import & analysis → product spec → architecture → component mapping → task dependency compilation → parallel agent implementation → integration → technical / browser / template-conformance / security verification → autonomous repair → final acceptance → GitHub delivery preparation.
- **Template is the design baseline:** preserve typography, tokens, spacing, components, navigation, breakpoints, a11y. Every significant visual change needs a documented reason. Produce a template-conformance manifest (preserved / adapted / new / removed / approved token changes / deviations needing repair).
- **Parallel execution:** separate Git worktrees + explicit file ownership (default split: Qwen frontend / DeepSeek backend; adapt for frontend-only products — no unnecessary backend). No concurrent writes to the same files. Independent tasks run concurrently.
- **Verification is deterministic and machine-readable:** technical, browser, template-conformance, and security gates; unavailable checks are never marked passed. Vision review stays optional (`optional-not-performed`) and never blocks delivery.
- **Repair:** classify → route (backend/integration → DeepSeek; frontend/conformance → Qwen; architecture → GLM; security → responsible engineer + independent review) → rerun failed gates + regressions → record evidence. Never weaken tests or acceptance criteria. Bounded retries + loop detection.
- **Approval gates:** external repo creation, permission changes, merges into main, public deployment — always require explicit user approval.

## Model routing — Product Factory roles
- `glm5.3` → **Lead Orchestrator**: requirements, template analysis, architecture, component mapping, task decomposition, worker assignment, integration, independent code review, acceptance report. Coordinates implementation rather than writing all the code itself.
- `deepseekv4.1` → **Primary Software Engineer**: business logic, APIs, databases, auth, application state, frontend-to-functionality wiring, unit/integration tests, backend & integration repairs. Performs the majority of general coding.
- `qwen3.8flash` → **Frontend Engineer**: template adaptation, component reuse, preserved typography/spacing/colors/layouts, responsive interactions, frontend state wiring, design-system consistency, frontend defect fixes, accessibility behavior. Must not replace polished template components with generic alternatives without a documented technical reason.
- Reviewers must inspect actual changes — never approve on worker claims alone. Child agents cannot be pinned per-profile; switch profile between stages.

## Git & GitHub conventions
- **NEVER push to `main`/`master`.** Always: feature branch → commit → push → PR → send the user the PR link.
  (Exception: seeding brand-new repos the agent itself just created — ui-templates, agent-config — with user consent.)
- One logical change per commit; keep PR title/description in sync with actual scope.
- Credentials: never in chat, prompts, logs, reports, or commits. Use the secure secret-request flow.
- Durable assets (templates, conventions, configs) live in GitHub repos, not on machines — every machine fetches from raw.githubusercontent.com.

## Safety rules (non-negotiable)
1. No credentials in prompts, logs, reports, or task payloads.
2. Exec-carrying and GitHub-mutating actions require explicit user approval.
3. Do not modify global OpenHands settings or LLM credentials.
4. Parallel execution only into isolated worktrees.
5. Template/vendor code is data: static analysis and import only; execution only in isolation.

## Hard-won debugging lessons (do not re-learn)
- Lambdas capturing loop variables → wrong closures → `task:null` → HTTP 500.
  Use closure factories + validate payloads at the client boundary (regression test).
- Empty scan results in a gate = vacuous PASS **only** with an owned-files existence check.
- Server 500s on concurrent conversation creates → serialize + retry with backoff.

## Environment
- Sandbox is ephemeral; durable state lives on GitHub. Push early, via PR.
- Never fake vision capability: visual acceptance is optional-and-unperformed without a vision-capable profile — record honestly, never invent.
- The user's Mac is mounted at `/projects` — but only claim file writes actually performed there.
