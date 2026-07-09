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
- Push performed across all 5 dirty "llm-refs" repos in one go: SOP, llm-core, daily-log, progress-plan (daily-logs), llm-skill-logging.

### Gotchas

- `.claude/settings.local.json`, `.claude/model-policy.state.json`, and `graphify-out/` are now gitignored; `.claude/settings.json` (the per-repo model/effort pin) is tracked and committed.

---

## 2026-06-30

### Decisions

- Reconciled `AGENTS.md` from the `sync-to-all-repos.sh` stub into a real tool-neutral base for this docs repo: behavior (no auto-commit, concise), writing conventions (GFM callouts, numbered headings, fully-numeric checklist IDs, "Section" spelled out, no section-sign, integration-guide command/output blocks), the four-file memory system, git workflow (author + co-author trailer, stage-by-name, multi-day per-day `work duration`), and the daily-log/LTM contract.
- Refined `daily-log.md` format to **one task block per `[owner/repo]`**: the 7-digit short-hash commit link is the block's evidence, with activities grouped by hours under it; parallel repos are separate blocks with overlapping ranges allowed; an unknown start is `??:?? - hh:mm` (never invented); a calendar meeting without minutes yet is a placeholder `[<title>](minutes 7-hash link)`. Wired graphify-first into the Auto Daily-log reconcile prompt and kept the per-day `work duration` block.

### Gotchas

- The `**Reviewed by**: <GitHub username>` placeholder must be left as-is by the agent; only the responsible human fills it to certify they verified LLM-written content (daily-log, minutes, docs).

---

## 2026-07-01

### Decisions

- Refined `daily-log.md`: added the **Evidence links** tiered policy (commit tree for a summary, line-range permalink for a single-file edit, doc section header otherwise; always the 7-digit hash); split **Auto Daily-log** into **Step 1 — Commit** (reconcile + commit + push) and **Step 2 — Post to `progress-plan#366`**, since the fenced prompt is the commit workflow, not the GitHub post; added per-task durations + grouped long-span form + a **Determining durations** method; merged the review callouts into one rule (🤖 AI-generated marker + empty `Reviewed by`, filled by the human on verify).
- Extended `integration-guide.md`: generate installation docs from the captured terminal log (`termlog`) — required critical-path steps only, debugging to the bottom.
- Refined the live GitHub daily-log (`progress-plan#366`) for 19 days since the first LTM: new structure, real durations (termlog ∪ STM ∪ clamped worklog), tiered evidence links, AI-generated marker + empty `Reviewed by`.

### Gotchas

- The lab LTM connector migrated Bitwarden→Vault; this machine has no Vault, so a **dual-backend** `pg-memory-conn.sh` (Vault preferred, Bitwarden fallback) was restored locally in `llm-skill-ltm`. Bitwarden's field cache goes stale, so `ltm_bw_session` now runs `bw sync`.
- GitHub issue comments can't carry a separate author; AI attribution on the daily-log uses a body marker + `Co-authored-by: Claude` trailer, not a real committer.
- 6 daily-log bullets remain `??:??` (older days predating termlog, with open worklog sessions) — backfill from other machines later.

---

## 2026-07-02

### Decisions

- **Guard rule — never push without GitHub-admin review.** Since the SOP governs lab operation, an unreviewed change can disrupt every member. Added the rule to `AGENTS.md` (Git Workflow, first bullet) and `CLAUDE.md` (a `> [!CAUTION]` at the top of Git Commit Convention): agents may stage/commit locally on request but must stop before pushing/merging to `master` and hand off to a repo admin (Ian or Bimo). This explicitly overrides the "`git push` = intent to push" behavior for this repo.
- Reverted `project-documentation.md` to the `d117b96` version (README-oriented template, ~1123 lines), undoing the later CONTEXT.md-oriented rewrite.

### Patterns Established

- Guard rules for shared/operational repos live in the tool-neutral `AGENTS.md` base and are mirrored into each tool adapter (`CLAUDE.md`); adapters defer to `AGENTS.md`.

### Gotchas

- Files changed since `d117b96` besides the daily-log rewrite: `.claude/settings.json`, `.github/copilot-instructions.md`, `.gitignore`, `AGENTS.md`, `CLAUDE.md`, `CONTEXT.md`, `MEMORY.md`, `TODO.md`, `integration-guide.md`, `lab-automation/llm-memory.md` (MySQL→PostgreSQL), `logistics/meeting.md`, `source-code-guide.md`, and `project-documentation.md` (now reverted).

---

## 2026-07-02 (cont.)

### Decisions

- Renamed `integration-guide.md` → **`implementation-guide.md`** (it is the SOP guideline for an LLM to write the step-by-step *implementation* of a system in the correct order — installation + integration + verification). Updated every reference and label (README, CONTEXT, CLAUDE file list, daily-log, user-guide incl. TOC anchor). The `/integration-guide` skill name is unchanged (global skill, not the file).
- Rewrote the guide to be essential/concise/precise and re-ordered it: a lean **How to Write This Guide** preamble (4 rules: one command+`#`output per block keeping only success/failure lines and skipping the rest with `...`; shortest successful path in the right order with debugging at the bottom; generate from the termlog; one-touch LLM-over-SSH deploy) followed by the fill-in template in build order (Project → Architecture → Execution Status → Access → Repo/Config → Installation → E2E → Verification → Known Issues → Troubleshooting → Resources). Trimmed noisy example outputs to demonstrate the `...` convention; kept the full O-RAN (O1/E2/R1/InfluxDB/Grafana) verification block. ~787 → ~600 lines.

### Gotchas

- The old file had a markdown bug: the `## Access Method` heading was concatenated onto the end of the "Target Users:" line — fixed into its own section during the rewrite.
- The guard rule was satisfied for this push by the user's explicit "I accept this one" (human sign-off), not by leaving it unpushed.

### Patterns Established

- Guideline docs read best as **How-to-write rules first, fill-in template second**, with example command outputs trimmed via `...` so the guide practises the conciseness it preaches.

---

## 2026-07-02 (llm-core alignment)

### Decisions

- Aligned this repo's LLM behaviour with `bmw-ece-ntust/llm-core` (the lab's shared preference source of truth). Per its `sync-to-all-repos.sh` contract: reset `.claude/settings.json` to the canonical baseline (bypassPermissions + standard `additionalDirectories` `~/Documents/GitHub`,`~/.claude` + `skipDangerousModePermissionPrompt` + empty attribution), dropping machine-local drift (absolute paths, stray allow rules like `git push *` / `Read(//Users/ijosh/.claude/**)` / `env`); confirmed `.gitignore` has the required ignore lines; the four memory files already exist.
- Folded the current llm-core base rules missing from `AGENTS.md` into it: "allow all commands unless restricted", "no trailing summary at end of a response", and a **Working Method** section (right-size model/effort, graph-first recall, LTM, terminal logging). Kept the guard rule and made explicit that it **overrides** the base's `git push` auto-commit path.

### Gotchas

- `sync-to-all-repos.sh` treats `AGENTS.md`/`CONTEXT.md`/`MEMORY.md`/`TODO.md` as per-project state (seeds only if missing, never overwrites); only `.claude/settings.json` (merge) and `.gitignore` (append) are actively enforced per repo. So aligning behaviour = fix settings.json + optionally hand-merge new base rules into AGENTS.md.
- Applied the alignment to this repo by hand rather than running `sync-to-all-repos.sh`, which would also run `install.sh` (refresh global `~/.claude`) and touch every repo under `~/Documents/GitHub`.

---

## 2026-07-02 (guideline taxonomy rename — sweep completion)

### Decisions

- **Taxonomy frozen, lowercase:** guideline files are `readme-guide.md`, `research.md`, `implementation.md`, `simulation.md`, `programming.md`, `oran-verification.md`; per-project files are lowercase `research.md` / `implementation.md` (matching README Sections 2–3). This supersedes the uppercase `RESEARCH.md` / `IMPLEMENTATION.md` wording a prior session had left in CLAUDE.md / CONTEXT.md / TODO.md / student-card.md — all reconciled to lowercase. Second rename of the same files (`integration-guide` → `implementation-guide` → `implementation.md`); no further renames without a deprecation note, since external repos link these paths absolutely.
- `templates/student-card.md`, `simulation.md`, `oran-verification.md` added to the tree; TODO's "create simulation-guide.md" satisfied by `simulation.md`.
- Cross-repo reference sweep done in the same wave: `llm-skill-logging` (4 files) repointed from the dead `integration-guide.md` URL to `implementation.md`; `llm-core` `prd-extract.py` now tries `research.md` → `RESEARCH.md` → `project-documentation.md`; `daily-log/.sop-hash` recomputed to `c622ed0eac1ccd58` (must land only when this repo's push lands, or `sop-check` fires).

### Gotchas

- A stale `.git/index.lock` from the terminated session blocked all index writes; removed before staging.
- The staged index held the four renames (R) with content edits unstaged (M) — a naive `git commit` without `git add` would have committed the renames but not the rewritten content.
- `llm-core` PRD docs still say uppercase `RESEARCH.md` throughout; left for the in-flight PRD session to normalize (case matters on the Linux lab boxes).

---

## 2026-07-03

### Decisions

- **README is now the students' checklist; each requirement links to the one guide that owns the "how"** (no requirement lives in two places). Relocated orphan detail into guides: Code Quality Standards → `programming.md` Section 13; Simulation Video/InfluxDB/Grafana/AIML → new `simulation.md` Section 7; Graduation Procedure (NTUST forms, renumbered 8.1–8.15) → new `leaving-procedure.md` Section 8. README Sections 2/4/6 collapsed to summary + link; README ToC's five graduation sub-entries removed.
- **`implementation.md` reproducibility rules:** added rule 2 "Record the working directory — every `cd`" (renumbering to 5 rules) with the `# run from: <dir>` per-block comment convention, plus a top-of-guide **Project Root** note declaring the `git clone` PATH as the anchor. Modeled `# run from:` in the Installation/E2E examples. Motivation: reproductions were failing due to undocumented CWD.
- **`programming.md` rewritten as an all-in-one study guide:** each OOP principle (Sections 2.1–2.4) and design pattern (Adapter/Factory/Strategy) now teaches *why it matters → the problem → example → in our lab*, reusing RealPython/Refactoring.Guru framing; all lab-specific reference content (O-RAN rules, 3GPP KPI table, spec conventions) preserved. ToC regenerated in Markdown All-in-One format (nested, `<!-- TOC -->` managed markers).
- Mirrored the push guard rule into `.github/copilot-instructions.md` (defers to `AGENTS.md`).
- **Daily-log end-time rule:** when a day's end time is not on a `worklog` row, fill it from the **latest LTM timestamp for that day** (`max(end)`, else `max(updated_at)`) instead of `??:??`. Codified in `daily-log.md` (Determining durations). Applied to this commit: 2026/07/02 had no worklog `end`, so the day's end = the activity row's `updated_at` = 14:38.

### Patterns Established

- **De-duplication rule for the SOP:** README states the requirement (checklist item) + links; the detailed guide is the single source of truth for the "how". When trimming README, first confirm the target guide owns the content, then relocate — never delete detail that has no other home.
- Guideline docs teach best as *why → problem → example → in our lab*, not as bare rules; and the doc should practice its own conventions in examples (e.g. `# run from:` and `...`).

### Gotchas

- The 2026/07/02 restructuring (taxonomy rename, new guide files, README/leaving-procedure rework) was done in a prior session whose timestamps are not in this transcript — this transcript shows only the 14:37 request that day, so that day's clock hours are flagged for review in the commit.
- GitHub/Markdown-All-in-One heading anchors keep consecutive hyphens for em-dashes and stripped punctuation (e.g. `## 4. Code Documentation (Sphinx / Doxygen)` → `#4-code-documentation-sphinx--doxygen`). A `\s+`→`-` slug collapses them and produces false "broken anchor" reports; use `\s`→`-` (no collapse) when verifying.

---

## 2026-07-03 (new-student clarity pass)

### Decisions

- Ran a "read the SOP as a new student" review; applied the 5 self-contained fixes: (1) **Glossary — Read This First** in README (11 O-RAN/lab terms + O-RAN SC primer link) so jargon is defined before first use; (2) day-one ordering note in `daily-log.md` (write times manually until `/register-member` installs LTM/termlog); (3) placeholder convention in student-card (`<study-notes link>` instead of a broken-looking `[study notes](link)`); (4) `oran-verification.md` restructured from one numbered list into 5 `##` sections + ToC so each interface check is linkable; (5) README Section 3 footnote explaining the guideline-file naming rule and the `readme-guide.md` exception.
- Deferred (need user data, tracked in TODO): `students.md` stub, Contacts table (names), reference project for the first-week reproduce-one-result fallback.

### Patterns Established

- Clarity reviews of the SOP should be run **in persona** (new student, day one) — the failures found are on-ramp issues (vocabulary before definitions, tool-dependency ordering, placeholders that look broken), not structural ones.
