# TODO.md — Task Backlog

## Now

- [ ] Fill in the Handover Candidates table in leaving-procedure.md Section 6 (actual student names and project topics; README Section 5 now only links there)
- [ ] Add the real Recognition Form link in leaving-procedure.md item 4.0 (currently the `<form link>` placeholder)
- [ ] Add `progress-plan` tool usage instructions to `daily-log.md §Generating the Daily-Log Entry`
- [ ] Add a Contacts table to README Getting Started (GitHub admin = Ian/Bimo, meeting admin, infra admin — names needed)
- [ ] Pick the lab reference project for the first-week "reproduce one result" fallback (greenfield projects have no predecessor)
- [x] Mirror the "never push without GitHub-admin review" guard rule into `.github/copilot-instructions.md` (added a "BMW Lab SOP — base instructions & guard rule" section that defers to `AGENTS.md`)
- [x] De-duplicate README into a students' checklist + links: moved Code Quality Standards → `programming.md` Section 13, Simulation Video/InfluxDB/Grafana/AIML → `simulation.md` Section 7, Graduation Procedure → `leaving-procedure.md` Section 8
- [x] Add the "record the working directory / `# run from:`" rule + a top-of-guide **Project Root** note to `implementation.md` (undocumented CWD was causing failed reproductions)
- [x] Rewrite `programming.md` as an all-in-one study guide (why each OOP principle & design pattern is essential, with examples) + Markdown All-in-One nested ToC

## Next

- [ ] Adopt `templates/student-card.md` as an issue template in `bmw-ece-ntust/progress-plan` (`.github/ISSUE_TEMPLATE/`)
- [ ] Propagate the `README.md` / `research.md` / `implementation.md` taxonomy to the `bmw-ece-ntust/template` repo (rename files, seed the three skeletons)
- [x] Create the simulation guideline (`simulation.md`: scenarios, immutable raw files, dataset registry, one-command figure regeneration)
- [x] Review `leaving-procedure.md` for consistency — README Section 5 de-duplicated to a summary; leaving-procedure.md is the single source (D-180 conformance item 4.1.1 added, Continuity & Access group 5.12–5.15 added, review renumbered to 5.16)
- [ ] Add new features to `progress-plan` daily-log automation tool
- [ ] Verify PostgreSQL LTM (llm-skill-ltm) recall + SessionStart hook end-to-end when the BMW Lab box SSH tunnel is available

## Later

- [ ] Add `students.md` referenced in the "Student Progress Stages" section of README (file does not exist yet)
- [ ] Sync `programming.md` to intern org via `sync-sop.yml` workflow
- [ ] Decide if CONTEXT.md / MEMORY.md / TODO.md belong in repo root or a subdirectory (e.g. `.claude/`)
