---
name: shiangking-conventions
type: knowledge
version: 1.0.0
agent: CodeActAgent
---

# Global conventions for shiangking

## Identity
- GitHub: shiangking (https://github.com/shiangking)
- Mac projects folder: ~/openhands-projects (mounted at /projects in the sandbox)

## Orchestration style (default for non-trivial builds)
- Design-first: plan before code (SPEC + architecture + disjoint worker manifests).
- Delegate with launch_child_conversation: parallel children, git worktree isolation,
  each child gets a fully self-contained brief (goal, paths, constraints, deliverable, self-verify).
- Verify every child's output in the parent before accepting — never trust child claims.
- Bounded repair: on failed verification, one repair attempt → re-verify → then escalate.
- Finish with an acceptance report: per-stage status, evidence, failure paths exercised.

## Model routing preference
- glm5.3 → planning, review, architecture
- deepseekv4.1 → implementation, repair
- qwen3.8flash → frontend, boilerplate
- Note: child agents can't be pinned per-profile; switch profile between stages instead.

## Git & GitHub
- NEVER push to main/master: feature branch → commit → push → PR → send user the PR link.
  (Exception: seeding a brand-new repo the agent itself just created, with user consent.)
- One logical change per commit; keep PR title/description in sync with actual scope.
- Credentials: never in chat, prompts, logs, reports, or commits. Use the secure secret-request flow.

## Safety rules (non-negotiable)
1. No credentials in prompts, logs, reports, or task payloads.
2. Exec-carrying and GitHub-mutating actions require explicit user approval.
3. Do not modify global OpenHands settings or LLM credentials.
4. Parallel execution only into isolated worktrees.

## Hard-won debugging lessons (do not re-learn)
- Lambdas capturing loop variables → wrong closures → task:null → HTTP 500.
  Use closure factories + validate payloads at the client boundary (regression test).
- Empty scan results in a gate = vacuous PASS only with an owned-files existence check.
- Server 500s on concurrent conversation creates → serialize + retry with backoff.

## Environment
- Sandbox is ephemeral; durable state lives on GitHub. Push early, via PR.
- Never fake vision capability: visual acceptance stays BLOCKED without a vision-capable profile.
- User's Mac is mounted at /projects — but only claim file writes you actually performed there.
