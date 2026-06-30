# MEMORY.md — Session Decisions Log

Append-only. Do not edit past entries. Add one `## yyyy-mm-dd` block per session at the bottom.

---

## 2026-05-25

### Decisions

- `installation-guide.md` merged into `integration-guide.md`: a single doc now covers per-component installation and end-to-end system integration, removing the need for a separate installation guide.
- Created `source-code-guide.md`: OOP principles, Adapter/Abstract Factory/Strategy design patterns (mandatory for rApp/xApp), Sphinx/Doxygen docstring formats, and standard folder structure.
- README sections 2–5 restructured: Section 4 renamed to "Simulation & Data Pipeline" (adds InfluxDB/Grafana/YouTube/pCloud requirements); Section 5 split into 5.1 Oral-Exam (personal graduation, ordered by priority), 5.2 Project Handover (grouped A→D: GitHub → Docs → Simulation → Review), 5.3 Handover Candidates table.
- `ea0fab6` also introduced "Student Progress Stages (by Year)" section in README: a rubric table distinguishing First-year (Proposal Stage) from Second-year (Execution Stage) deliverables.
- Prompt B added to `daily-log.md §Auto Daily-log`: offline mode that reconciles 4 files (CLAUDE.md, CONTEXT.md, MEMORY.md, TODO.md) instead of relying on MySQL. Prompt A (MySQL) drafted but commented out pending VM access verification.
- Introduced 3 new project memory files (CONTEXT.md, MEMORY.md, TODO.md) committed to the SOP repo root.

### Patterns Established

- Integration guide = single doc for "how to get the full system running" (installation + integration in one place).
- Checklist items in Section 5.2 use grouped headings A→B→C→D to establish explicit priority order.
- 4-file Prompt B pattern: CLAUDE.md (static snapshot) + CONTEXT.md (living architecture) + MEMORY.md (decisions) + TODO.md (backlog).
- Daily-log automation tool lives at `bmw-ece-ntust/progress-plan` → local clone `~/Documents/GitHub/daily-logs/`.

### Gotchas

- `raycg` had no prior commits in this repo; author email must be supplied explicitly as `--author="Ray-Guang Cheng <crg@mail.ntust.edu.tw>"`.
- `git mv` fails on untracked (new) files — use plain `mv` + `git add` instead.
- Force-push was needed to fix author on an already-pushed commit; remote allowed it with a "Bypassed rule violations" warning despite a branch protection rule.
- Apparent git state confusion mid-session: `ea0fab6` showed as HEAD despite earlier commits. Cause: the `ea0fab6` commit was already in the remote and incorporated all session changes; local HEAD matched remote after sync.

---

## 2026-06-26

### Decisions

- LTM migrated from MySQL to **per-user PostgreSQL**, managed by the `bmw-ece-ntust/llm-skill-ltm` repo. The earlier MySQL `llm_memory` setup (Prompt A, `mysql-memory` MCP) is retired and removed from all active docs. `lab-automation/llm-memory.md` rewritten as a PostgreSQL overview pointing to `llm-skill-ltm`; the SessionStart hook there records activity automatically in both Claude Code and Cowork.
- LTM `metadata` now keeps `owner` (the GitHub account that owns the repo, org or personal) and `repo` (name only) as **separate fields**, so daily-log activities group per project.
- `daily-log.md` format updated: each daily-log bullet is one session on one project, tagged `[owner/repo]`, with required `hh:mm - hh:mm` start/end times. Tasks may overlap because agentic AI runs tasks concurrently. The auto-commit `Work Start` field now carries a `yyyy/mm/dd: hh.mm - hh.mm` range.

### Patterns Established

- Project identity in the LTM and in daily-log bullets is always `owner` + `repo` as separate fields/tags, never a single `owner/repo` string embedded in the repo field.
- Overlapping daily-log time ranges across different `[owner/repo]` projects are expected and allowed.

### Gotchas

- `MEMORY.md` is append-only: the prior MySQL mention in the 2026-05-25 entry stays as a historical record; this entry supersedes it rather than editing it.

---

## 2026-06-29

### Decisions

- Committed the accumulated PostgreSQL-LTM migration work (docs rewrite across `daily-log.md`, `lab-automation/llm-memory.md`, `project-documentation.md`, `source-code-guide.md`, copilot-instructions, and the 4 project files) together as one daily-log push.
- Adopted `AGENTS.md` as the tool-neutral preference base (seeded by `sync-to-all-repos.sh`); `CLAUDE.md` and other tool adapters defer to it. Added it and `.claude/settings.json` to the CLAUDE.md / CONTEXT.md file maps.
- Push performed across all 5 dirty "llm-refs" repos in one go: SOP, llm-prefs, daily-log, progress-plan (daily-logs), llm-skill-logging.

### Gotchas

- `.claude/settings.local.json`, `.claude/model-policy.state.json`, and `graphify-out/` are now gitignored; `.claude/settings.json` (the per-repo model/effort pin) is tracked and committed.
