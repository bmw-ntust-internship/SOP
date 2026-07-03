# CONTEXT.md — Architecture Snapshot

> Living document. Updated each session via Prompt B. Reflects the current repo state.

---

## What This Repo Is

SOP repository for the **BMW Lab** at NTUST, supervised by Prof. Ray. Pure documentation — no code to build or test. Entry point for all lab rules, workflows, and checklists for graduate students and interns.

## Key Files Map

| File | Purpose |
| --- | --- |
| `README.md` | Main SOP — the new-student guideline: Research Philosophy, Getting Started (first week), stage navigation, year rubric, Sections 1–6 |
| `daily-log.md` | Daily tracking format + Auto Daily-Log workflow (PostgreSQL LTM + 4-file reconcile) |
| `implementation.md` | Guideline for a project's `implementation.md`: per-component installation + end-to-end integration, ends with the O-RAN verification checks (renamed from `implementation-guide.md`) |
| `readme-guide.md` | Guideline for a project's `README.md`: Prerequisites → `implementation.md` link → User Guide (renamed from `user-guide.md`) |
| `research.md` | Guideline for a project's `research.md`: complete research description in IEEE transactions structure, reused for paper writing (renamed from `project-documentation.md`) |
| `programming.md` | Coding standards + study guide: WHY each OOP principle & design pattern (Adapter/Abstract Factory/Strategy) matters with examples, Sphinx/Doxygen, folder structure, O-RAN/3GPP reference (renamed from `source-code-guide.md`) |
| `simulation.md` | Guideline for a project's `simulation/` folder: scenarios, immutable raw files, dataset registry, one-command figure regeneration |
| `oran-verification.md` | Lab-common O-RAN conformance checks (O1 / E2 / R1 + PM pipeline), required at the end of every `implementation.md` and for the D-180 review |
| `paper-writing.md` | IEEE paper writing and revision steps |
| `leaving-procedure.md` | Full leaving/graduation process: handover, professor change policy, NDA requirement |
| `AGENTS.md` | Tool-neutral AI-agent preference base; CLAUDE.md (and other tool adapters) defer to it |
| `CLAUDE.md` | LLM knowledge snapshot — static (file list, terminology, conventions, links) |
| `CONTEXT.md` | This file — living architecture overview |
| `MEMORY.md` | Append-only session decisions log |
| `TODO.md` | Task backlog: Now / Next / Later |

## Subdirectories

| Path | Purpose |
| --- | --- |
| `.github/copilot-instructions.md` | GitHub Copilot instructions (mirrors CLAUDE.md) |
| `.github/workflows/sync-sop.yml` | Auto-syncs selected SOP files to intern org repo on push |
| `lab-automation/credentials-vault.md` | Credential management practices |
| `lab-automation/llm-memory.md` | PostgreSQL long-term memory setup (per-user; managed by llm-skill-ltm) |
| `lab-internal/stipend.md` | Stipend information |
| `logistics/meeting.md` | Meeting preparation and protocol |
| `logistics/teep-preparation.md` | TEEP program preparation guide |
| `NDA/nda-template.md` | NDA signed by all departing members |
| `templates/paper/` | IEEE paper templates (PDF) |
| `templates/slide/` | Presentation slide templates (PDF) |
| `templates/student-card.md` | Student card template: first bubble = profile + hyperlinks + deadline checklists; comments = daily-log posts |

## External Services

| Service | Address | Purpose |
| --- | --- | --- |
| BMW Lab GitHub Org | <https://github.com/bmw-ece-ntust> | All project repositories |
| Lab Members Board | <https://github.com/orgs/bmw-ece-ntust/projects/8> | Daily-log tracking for lab members |
| Interns Board | <https://github.com/orgs/bmw-ntust-internship/projects/4> | Daily-log tracking for interns |
| BMW Lab YouTube | <https://www.youtube.com/@ntust.bmw-lab> | Simulation and demo videos |
| Thesis pCloud | <http://u.pc.cd/Dbd> | Datasets, slides, thesis materials |
| NTUST Academic Calendar | <https://www.academic.ntust.edu.tw/p/412-1048-8756.php?Lang=en> | Graduation deadlines |
| BMW Lab box | `140.118.122.119` (SSH tunnel) | Per-user PostgreSQL `*-memory` DBs for long-term LLM memory (managed by llm-skill-ltm) |
| Daily-Log Automation | [bmw-ece-ntust/progress-plan](https://github.com/bmw-ece-ntust/progress-plan) | Auto daily-log tool (`src/dailylog/`); local clone at `~/Documents/GitHub/daily-logs/` |

## Auto Daily-Log Workflow

Defined in `daily-log.md` (Auto Daily-log). Two parts:

| Part | When | What it does |
| --- | --- | --- |
| **LTM (PostgreSQL)** | SessionStart hook from `llm-skill-ltm` | Records `activity` (per repo/day) and `worklog` (per session, exact start/end) rows in the per-user Postgres DB, attributed by GitHub account, with `owner`/`repo` as separate fields |
| **4-file reconcile** | At commit (`git push`) | Reconciles CLAUDE.md, CONTEXT.md, MEMORY.md, TODO.md; commits all 4 |

## Git Convention

- Branch: `master` (single branch)
- **Guard rule:** never push (or merge to `master`) without review from a GitHub repo admin (Ian or Bimo) — the SOP governs lab operation. Agents commit locally on request but hand off for admin review before pushing, even when the user types `git push`. Defined in `AGENTS.md` (Git Workflow) and `CLAUDE.md` (Git Commit Convention).
- Author: Ray-Guang Cheng `crg@mail.ntust.edu.tw`
- Co-author: Ian Joseph Chandra `ianjoseph2204@gmail.com`
- Commit format: Title / work duration (`yyyy/mm/dd_hh:mm - yyyy/mm/dd_hh:mm`) / Summary / Details / Co-authored-by
