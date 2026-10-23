---
name: shiangking-conventions
version: 1.4.0
description: Standing conventions for working with shiangking — template-first UI, repos, orchestration, safety. Load for any frontend/product work.
---

# shiangking — agent conventions

## Identity & repos
- GitHub account: **shiangking** — https://github.com/shiangking
- Mac projects folder: `~/openhands-projects` (mounted at `/projects` in sandbox)
- **Template library**: https://github.com/shiangking/ui-templates (`registry.json` is the index)
- **Conventions**: https://github.com/shiangking/agent-config (this file — re-fetch from here, it may be newer than what's loaded)
- **Product Factory flow & workplan (authoritative)**: `docs/orchestration-flow.md` in agent-config — read before any factory/orchestration work

## Template-first UI development (NON-NEGOTIABLE)
1. All approved UI templates live in **shiangking/ui-templates**. Check `registry.json` FIRST — never start a frontend from scratch.
2. **Layers** (as of 2026-10-23):
   - `design-system-v1` — base layer: `tokens.css` + `components.css` (pills, buttons, topbar, sidebar, inputs, tables, cards, tabs, toggles, menus, tooltips, alerts) + TWO motion profiles:
     - **Profile 1 "Framer soft"** (`motion.css` + `motion.js`): springs `100/30` soft + `100/10` bouncy extracted from the Nuria Framer bundle. API: `data-reveal="up|scale|pop"`, `data-stagger="1"`, `.m-lift .m-pop .m-press .m-shine .m-pulse`, `[data-marquee]`.
     - **Profile 2 "cinematic"** (`motion-cinematic.css` + `.js`): Storm Lab vocabulary — scroll-pinned scenes (`.scene[data-p]`), curtain lines (`.curtain>.line`), kinetic letters (`[data-kinetic]`), lightning (`window.strike()`), magnetic buttons (`[data-magnetic]`), parallax, custom cursor, grain.
     - Rule: use ONE motion profile per project; both honor `prefers-reduced-motion`.
   - `northstar-design-systems` — multi-theme design system (user-sourced 2026-10-23): THREE full themes on one `--ds-*` contract — **Studio** (indigo `#5a46c5`, Inter 600 −0.055em, product/dashboards), **Fieldwork** (rust `#b73e15` on parchment, Arial Narrow 800 uppercase, editorial/outdoor), **Nocturne** (dark `#12151a`, mint `#b3e9dd`, dev tools). ~50 ds-* components + landing/app-shell/gallery recipes, scoped to `.ds-kit`, zero deps. **Best a11y patterns in the library** (skip links, focus-visible, ARIA tabs/dialog/switch, aria-live toasts, keyboard tab nav, in-app motion pause). Use for product UIs; use design-system-v1 for marketing pages. Preview: `templates/northstar-design-systems/preview.html`. Motion ease `cubic-bezier(.22,1,.36,1)` matches our Framer-soft spring.
   - Code templates (pinned upstream): `fimbo-portfolio-1/2/3` (vanilla portfolios), `odyssey-theme` (Astro marketing/blog, sage #cce6d0, pill buttons, Lato+Roboto Serif, 5-theme switcher).
   - Design sources (Figma specs, translate onto code templates): `figma-prompverse-ai` (dark AI landing), `figma-startup-kit` (#2F1893 purple section library), `figma-genesis-landing` (IBM Plex Sans −0.04em, sky-glass hero, glass flag cards, navy 32px science cards, green/amber/indigo score pills).
   - `renders/` — reference implementations (Genesis HTML translation; Odyssey motion annotation pattern). Guides, NOT page templates.
3. **Adapt, don't reinvent**: preserve tokens, components, navigation, responsive + a11y behavior. Apply motion via data-attributes only — never rewrite a template's CSS.
4. **Quality bar is HUMAN**: agent-authored page templates must be rendered, shown to the user, and explicitly approved BEFORE pushing to ui-templates. (Two agent templates were deleted 2026-10-22 for self-approval.) `design-system-v1` is approved; `northstar-design-systems` is user-sourced (file provided by user).
5. Import via `tools/import_template.sh <id> [dest]` (static only — never execute template code outside isolation).
6. **No vision dependency**: visual review is optional; record `optional-not-performed` when skipped, never fake it. Never block delivery on vision.
7. If no approved template fits, ASK the user — don't invent a new visual direction.

## Feeding the library (user's workflow)
- The user curates: they send Figma/Framer/GitHub links OR files (e.g. northstar-preview.html); agent analyzes, checks for duplicates against the registry, and files as design sources, motion profiles, design systems, or reference renders.
- **Duplicate check before adding**: compare scope/personality/signature values vs registry entries; record the verdict in the manifest (`duplicate_check` field). Overlap in scope ≠ duplicate (e.g. Northstar vs design-system-v1 share scope but differ in theming/app-shell/a11y — filed as siblings with cross-links).
- Large user files (73KB+): split into the template's own architecture (tokens.css + data + app) rather than pushing the monolith; verify the split renders identically with headless Chromium before pushing; push via multiple API calls (get_file_contents for SHAs, create_or_update_file per file, push_files for batches) — shell git has no credentials in this sandbox.
- Framer project links are usually auth-walled → try the public `<name>.framer.website` demo; if the bundle is fetchable, grep for `stiffness/damping/mass/duration/ease` to extract real spring DNA (this worked for Nuria: springs 100/30 and 100/10).
- Figma: use Figma MCP on community files; watch for cover thumbnails posing as design data (verify node structure).
- Every addition needs a manifest with origin, license, and (if agent-built) explicit user approval note.

## Orchestration flow — template-driven Product Factory (v1.4.0)
- **Authoritative flow & workplan**: `docs/orchestration-flow.md` in agent-config. Read it before any factory work.
- **Extend, never rebuild** the Phase E Product Factory (`agent-orchestration-poc/`). No new orchestration frameworks; inspect actual modules before changes.
- **Pipeline:** product request → template selection → template import & analysis → product spec → architecture → component mapping → task dependency compilation → parallel agent implementation → integration → technical / browser / template-conformance / security verification → autonomous repair → final acceptance → GitHub delivery preparation.
- **Model roles:** `glm5.3` Lead Orchestrator (requirements, template analysis, architecture, component mapping, worker assignment, integration, independent review, acceptance report — coordinates rather than writing all code); `deepseekv4.1` Primary Software Engineer (business logic, APIs, DB, auth, state, frontend-to-functionality wiring, tests, backend repairs — majority of general coding); `qwen3.8flash` Frontend Engineer (template adaptation, component reuse, preserved typography/tokens/layout, responsive + a11y, frontend state, frontend fixes — no generic substitutions without documented reason). Switch profile between stages; children can't be pinned. Reviewers inspect actual changes, never worker claims alone.
- **Parallel execution:** Git worktrees + explicit file ownership; Qwen frontend / DeepSeek backend by default; adapt for frontend-only products (no unnecessary backend); no concurrent writes to the same files.
- **Template is the design baseline:** template-conformance manifest (preserved / adapted / new / removed / approved token changes / deviations needing repair); every significant visual change has a documented reason.
- **Verification:** deterministic + machine-readable (technical, browser, conformance, security gates); unavailable checks never marked passed; vision optional (`optional-not-performed`), never blocks. Browser gates: Playwright journeys, nav/forms/buttons, persistence, states, mobile+desktop, horizontal overflow, console errors, broken assets.
- **Repair routing:** backend/integration → DeepSeek; frontend/conformance → Qwen; architecture → GLM; security → responsible engineer + independent review. Rerun failed gates + regressions, record evidence, never weaken tests or acceptance criteria. Bounded retries + loop detection.
- **Approval gates:** new external repos, permission changes, merges into main, public deployment — require explicit user approval.
- Design-first → verify-in-parent → bounded repair still applies for non-factory builds.

## Git & GitHub conventions
- **NEVER push to `main`/`master`.** Feature branch → PR → user link. (Exception: repos the agent created itself — ui-templates, agent-config — user-consented direct pushes.)
- One logical change per commit. No credentials in chat, logs, or commits.
- Durable assets live on GitHub, not machines — push early.

## Safety (non-negotiable)
1. No credentials in prompts/logs/reports.
2. Exec-carrying + GitHub-mutating actions need explicit approval.
3. Parallel work only into isolated worktrees.
4. Template/vendor code is data: static analysis only; execution in isolation only.

## Hard-won lessons (do not re-learn)
- Lambda loop-capture bug → `task:null` → HTTP 500. Closure factories + boundary validation.
- Empty scan results = vacuous PASS only with an owned-files existence check.
- Heredocs with JSON/quotes are flaky — prefer file_editor tool or python writes; validate JSON after writing.
- Google Fonts CSS2 API serves variable fonts (one woff2 covers all weights) — dedupe by md5.
- GitHub raw CDN caches ~300s; verify pushes via the API path, not raw.
- Chromium headless at `/usr/bin/chromium` (playwright MCP may be broken in sandbox; `--no-sandbox --headless --screenshot` works).
- Terminal output clips around ~200 lines; for large file pushes use several create_or_update_file calls (one per file) — the tool has no practical size limit at 73KB; a heredoc echo-through-terminal does.
- json.JSONDecoder().raw_decode for JS-prefixed JSON blocks (regex+json.loads fails on trailing data).

## Environment notes
- Sandbox is ephemeral; durable state lives on GitHub.
- User's Mac mounted at `/projects` — only claim writes actually performed.
- Sessions are continuous: the user resumes across days ("continue tomorrow") — always leave the library in a consistent, pushed state at session end.
