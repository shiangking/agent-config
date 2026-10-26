# OpenHands Product Factory — Template-Driven Multi-Agent Orchestration

**Status:** Approved workplan — authoritative for pipeline, gates, and repair.
**Date:** 2026-10-23 · **Model policy amended:** 2026-10-26
**⚠ Model routing superseded:** §2 previously split work across GLM/Qwen/DeepSeek. As of 2026-10-26 `deepseekv4.1flash` is the ONLY active model and performs every role. See §2 below. Everything else in this document (pipeline, template conformance, verification gates, repair, approval gates) still applies unchanged.
**Applies to:** `agent-orchestration-poc/` (Phase E Product Factory). Extend and reuse — never rebuild.

## Mission

Upgrade our existing OpenHands Product Factory to autonomously build complete, polished, functional applications using approved frontend templates and our existing multi-agent orchestration infrastructure.

Input:
1. A product idea.
2. An existing frontend template, starter project, or GitHub repository.
3. Optional feature requirements.

The system then autonomously adapts the template, implements the required functionality, tests the product, repairs defects, and prepares a finished application for delivery.

**Do not build a new orchestration framework. Extend and reuse the existing Phase E Product Factory.**

The objective is to produce applications that preserve the professional appearance and usability of their source templates rather than generating generic AI-designed interfaces.

---

## 1. Existing Infrastructure

We already have a working OpenHands multi-agent system with:

- OpenHands Agent Server and Agent Canvas.
- Soclaas model provider connection.
- Model-specific agent execution.
- Git worktree isolation.
- Deterministic verification gates.
- Autonomous repair loops.
- Persistent SQLite state and recovery.

Inspect the actual files before making changes. Do not assume any module exposes functionality that has not been verified.

Preserve existing successful tests, run histories, SQLite state, and recovery mechanisms.

---

## 2. Model and Agent Responsibilities

### Single active model — DeepSeek V4.1 Flash (`deepseekv4.1flash`)

`deepseekv4.1flash` (`openai/x-test-2` via the Soclaas provider) is the **only** active
coding model. One agent performs every role in the pipeline:

- Interpret product requirements.
- Analyze the selected frontend template (real source, real license).
- Design the application architecture and identify reusable components.
- Define feature-to-component mappings.
- Break development into independent tasks and manage dependencies.
- Implement business logic, APIs, backend services, and databases where required.
- Implement authentication and application state / data flows.
- Connect frontend components to working functionality.
- Adapt the approved template: reuse components, preserve typography, spacing, colors,
  layouts, responsive behavior, and accessibility.
- Write and run unit and integration tests.
- Fix backend, integration, and frontend defects.
- Review the actual changes (not claims) and produce the acceptance report.

There is no orchestrator/worker model split. Architecture, implementation, frontend, and
review all run on DeepSeek, in the same conversation or in sequential/parallel worktrees.

### Independent review, single model

Review is a **stage**, not a separate model. Before declaring work complete:

- Re-read the actual diff and the actual test output — never accept a prior claim.
- Rely on deterministic checks (build, lint, typecheck, tests, browser gates,
  template-conformance) as the primary evidence.
- State plainly what was verified and what was not; never mark an unavailable check as passed.

### Dormant profiles

Retained on disk, never selected, never referenced in automation:

| Profile | Model | State |
|---|---|---|
| `glm5.3` | `openai/x-test-1` | dormant |
| `qwen-3.8flash` | `openai/x-test-3` | dormant |
| `MiniMax-M3.1Flash` | `openai/x-test-4` | dormant |
| `router` (agent profile) | → `glm5.3` | dormant — do not launch |

Re-activation requires the user's explicit approval. Do not create new profiles, routers,
meta-profiles, or `classify_and_switch_llm` routing.

---

## 3. Template Import and Registration

Templates come only from the approved library: https://github.com/shiangking/ui-templates (`registry.json` is the index).

Rules:

- Import statically via `tools/import_template.sh <id> [dest]` — never execute template code outside an isolated environment.
- Record provenance: template id, version, source path/upstream commit, and license in the product's manifest.
- Licenses marked "verify author terms" must be verified before adaptation; reference renders are guides, not page templates.
- If no approved template fits, request one from the user — do not invent a visual direction.

---

## 4. Template-First Product Planning

When a new product idea is submitted, the orchestrator must first identify the most appropriate approved template.

- If a template is specified, use it unless it is technically incompatible or its license prevents reuse.
- If no template is specified, select from the existing approved template library.
- If no suitable approved template exists, request one rather than inventing a visual direction.

Before implementation, a concrete plan must be produced:

1. Product specification.
2. Feature list and acceptance criteria.
3. Technical architecture.
4. Template component inventory.
5. Feature-to-component mapping.
6. Required component modifications.
7. New components that genuinely need to be created.
8. Task breakdown with dependencies.
9. Verification plan mapped to deterministic gates.

---

## 5. Multi-Agent Execution Workflow

### Parallelism, not model splitting

Work may be split across workers, but **every worker runs DeepSeek**. Parallelism is a
speed optimization only.

- Separate Git worktrees per worker.
- Explicit file ownership — no two workers write the same file concurrently.
- Independent tasks run concurrently; dependent tasks are sequenced.
- For frontend-only products, do not create an unnecessary backend.

Canonical split (adapt per product):

**Worker A — Frontend**: template component adaptation, UI interactions, responsive behavior, frontend state, frontend tests.

**Worker B — Backend**: business logic, APIs, database, data validation, backend tests.

### Integration

- Merge worker output into the integration worktree.
- Resolve conflicts explicitly; record decisions.
- Rerun the full gate set after integration.

### Review

Review the actual diff and actual command output. Never approve on a worker's claim alone.

---

## 6. Preserve Template Quality

The template is the design baseline.

- Preserve typography, tokens, spacing, components, navigation, breakpoints, and accessibility.
- Every significant visual change needs a documented reason.
- Avoid CSS rewrites; apply motion via the template's own data-attributes.
- Produce a template-conformance manifest: preserved / adapted / new / removed / approved token changes / deviations needing repair.

---

## 7. Deterministic Product Verification

Verification must be deterministic and machine-readable:

- Technical gates: build, lint, typecheck, unit and integration tests.
- Browser gates: Playwright user journeys — navigation, forms, buttons, persistence, loading/empty/error states, mobile + desktop, no horizontal overflow, no console errors, no broken assets or fonts, no unexpected external network requests.
- Template-conformance gates: component presence, token preservation, dependency diff, navigation, breakpoints (375/768/1280px), a11y basics.
- Security gates: dependency audit and secrets scan. If a scanner is unavailable, record that honestly — never mark it passed.

Unavailable checks are never marked passed.

---

## 8. Visual Verification Policy

Visual/screenshot review is optional and is not a delivery blocker.

- When skipped, record `optional-not-performed`.
- Never claim visual quality was independently reviewed when it was not.
- Screenshots are kept as evidence, not AI-interpreted.
- The report verifies template conformance + functional usability. It does not assert subjective visual quality.

---

## 9. Autonomous Repair

- Classify the defect (backend/integration, frontend/conformance, architecture, security).
- Fix it directly — there is no defect hand-off between models.
- Rerun the failed gates **plus** regressions.
- Record evidence for each repair.
- Never weaken tests or acceptance criteria.
- Bounded retries + loop detection; stop and report when progress stalls.

---

## 10. GitHub Integration

- Feature branch → commit → push → PR. Never push to `main`/`master`.
- One logical change per commit; meaningful messages.
- Run checks before pushing.
- PR body includes a summary and test evidence.
- Never merge into `main` or deploy publicly without explicit user approval.
- Never commit credentials or sensitive files.

---

## 11. Persistent Unattended Execution

- Preserve SQLite state and recovery mechanisms.
- Server 500s on concurrent conversation creates → serialize + retry with backoff.
- Empty scan results in a gate are a vacuous PASS **only** with an owned-files existence check.

---

## 12. Product Factory Input

A product request consists of:

1. Product idea / requirements.
2. Optional specified template.
3. Optional feature list and constraints.

---

## 13. Acceptance Tests

Create a disposable demonstration project using an approved, appropriately licensed frontend template.

Verify:

1. Template imports successfully.
2. Template provenance and license are recorded.
3. A real component-mapping plan is produced.
4. Implementation tasks run on DeepSeek (optionally in parallel worktrees).
5. Parallel worktrees remain isolated.
6. Template components are reused.
7. Required product functionality works.
8. Technical gates pass.
9. Playwright user journeys pass.
10. Template-conformance gates pass.
11. Security gates pass.
12. Autonomous repair handles planted defects.
13. No credentials appear in commits, logs, or reports.

---

## 14. Implementation Instructions

- Inspect the actual `agent-orchestration-poc/` modules before changing them.
- Extend existing modules (`controller.py`, `factory.py`, `design.py`, `parallel.py`, `repair.py`, `github.py`, `verify.py`, `gates/`, `report.py`).
- No new orchestration frameworks.
- Keep existing tests green; add tests for new behaviour.
- Approval gates: external repo creation, permission changes, merges into main, public deployment — always require explicit user approval.
