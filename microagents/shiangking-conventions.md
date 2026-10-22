---
name: shiangking-conventions
type: knowledge
version: 1.1.0
agent: CodeActAgent
---

# Global conventions for shiangking

## Identity
- GitHub: shiangking (https://github.com/shiangking)
- Mac projects folder: ~/openhands-projects (mounted at /projects in the sandbox)
- Conventions repo: https://github.com/shiangking/agent-config (re-fetch from here)

## Template-first UI development (NON-NEGOTIABLE as of 2026-10-22)
- Approved UI templates live in https://github.com/shiangking/ui-templates (registry.json = index).
- Every frontend starts from an approved template's actual source files — never from scratch, never from LLM screenshot interpretation.
- Adapt, don't reinvent: preserve design tokens, components, navigation, responsive + a11y behavior. Targeted, explainable modifications only. No CSS rewrites, no replacing polished components with generic AI equivalents.
- Import via tools/import_template.sh in that repo (static fetch; NEVER execute template code outside isolation).
- No vision dependency: visual review optional; record `optional-not-performed` when skipped — never mark passed; never block delivery on missing vision. Technical/security/browser/usability gates stay enforced.
- If no approved template fits, request one from the user — do not invent a new visual direction.
- Conformance = deterministic checks: component presence, token preservation, dependency diff, navigation, no overflow at 375/768/1280, breakpoints, a11y, no console errors, no external requests, no broken assets. Screenshots = evidence, not AI-interpreted. Never claim subjective visual quality.

## Orchestration style (default for non-trivial builds)
- Design-first: plan before code (SPEC + architecture + disjoint worker manifests).
- Delegate with launch_child_conversation: parallel children, git worktree isolation,
  each child gets a fully self-contained brief (goal, paths, constraints, deliverable, self-verify).
- Verify every child's output in the parent before accepting — never trust child claims.
- Bounded repair: on failed verification, one repair attempt → re-verify → then escalate.
- Finish with an acceptance report: per-stage status, evidence, failure paths exercised.

## Model routing preference
- glm5.3 → planning, review, architecture, template analysis + component-reuse planning
- deepseekv4.1 → implementation, repair, business logic, APIs, backend, state
- qwen3.8flash → frontend template adaptation (preserve styling/tokens), boilerplate
- Note: child agents can't be pinned per-profile; switch profile between stages instead.

## Git & GitHub
- NEVER push to main/master: feature branch → commit → push → PR → send user the PR link.
  (Exception: seeding a brand-new repo the agent itself just created, with user consent.)
- One logical change per commit; keep PR title/description in sync with actual scope.
- Credentials: never in chat, prompts, logs, reports, or commits. Use the secure secret-request flow.
- Durable assets (templates, conventions, configs) live on GitHub — machines fetch from raw.githubusercontent.com.

## Safety rules (non-negotiable)
1. No credentials in prompts, logs, reports, or task payloads.
2. Exec-carrying and GitHub-mutating actions require explicit user approval.
3. Do not modify global OpenHands settings or LLM credentials.
4. Parallel execution only into isolated worktrees.
5. Template/vendor code is data: static analysis and import only; execution only in isolation.

## Hard-won debugging lessons (do not re-learn)
- Lambdas capturing loop variables → wrong closures → task:null → HTTP 500.
  Use closure factories + validate payloads at the client boundary (regression test).
- Empty scan results in a gate = vacuous PASS only with an owned-files existence check.
- Server 500s on concurrent conversation creates → serialize + retry with backoff.

## Environment
- Sandbox is ephemeral; durable state lives on GitHub. Push early, via PR.
- Never fake vision capability: visual acceptance is optional-and-unperformed without a vision-capable profile — record honestly, never invent.
- User's Mac is mounted at /projects — but only claim file writes you actually performed there.
