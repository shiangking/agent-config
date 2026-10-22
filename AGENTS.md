# AGENTS.md — standing instructions for every OpenHands chat

This file is loaded as workspace context. Follow it unless the user explicitly overrides.

## Identity
- GitHub account: **shiangking** — https://github.com/shiangking
- Mac projects folder: `~/openhands-projects` (mounted at `/projects` in the sandbox)
- Agent conventions repo: https://github.com/shiangking/agent-config (this file's home — re-fetch from here)

## Template-first UI development (NON-NEGOTIABLE as of 2026-10-22)
- All approved UI templates live in **https://github.com/shiangking/ui-templates** (`registry.json` is the index).
- Every product's frontend starts from an approved template's **actual source files** — never generated from scratch, never from LLM screenshot interpretation.
- **Adapt, don't reinvent:** preserve design tokens, components, navigation, responsive + accessibility behavior. Keep modifications targeted and explainable. Avoid CSS rewrites and replacing polished template components with generic AI-generated equivalents.
- Import templates via `tools/import_template.sh <id> [dest]` in that repo (static fetch only — **never execute template code** outside an isolated environment).
- **No vision dependency:** visual/screenshot review is optional. When skipped, record `optional-not-performed` — NEVER mark it passed. Never block delivery on missing vision capability. Existing technical, security, browser, and usability gates stay enforced.
- **If no approved template fits the product, request one from the user — do not invent a new visual direction.**
- Conformance evidence is deterministic: component presence, token preservation, dependency diff, navigation, no horizontal overflow at 375/768/1280px, breakpoints, a11y basics, no console errors, no external network requests, no broken assets/fonts. Screenshots kept as evidence, not AI-interpreted. Report verifies template conformance + functional usability — it does NOT claim subjective visual quality.

## Orchestration style (default for non-trivial builds)
- **Design-first:** plan before code — SPEC + architecture + disjoint worker manifests.
- **Delegate** with `launch_child_conversation`: parallel children, git worktree isolation,
  each child gets a fully self-contained brief (goal, paths, constraints, deliverable, self-verify).
- **Verify in the parent:** never trust child claims — re-run tests / static checks
  in the main workspace before accepting child output.
- **Bounded repair:** on failed verification, one repair attempt → re-verify → then escalate to user.
- **Report:** finish with an acceptance report — per-stage status, evidence, failure paths exercised.

## Model routing preference
- `glm5.3` → planning, review, architecture + template analysis / component-reuse planning
- `deepseekv4.1` → implementation, repair, business logic, APIs, backend, state
- `qwen3.8flash` → frontend **template adaptation** (preserve styling/tokens), boilerplate
- Note: child agents cannot be pinned per-profile; switch profile between stages instead.

## Git & GitHub conventions
- **NEVER push to `main`/`master`.** Always: feature branch → commit → push → PR → send the user the PR link.
  (Exception: seeding a brand-new repo the agent itself just created, with user consent.)
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
