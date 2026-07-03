# Daily Plans and Logs

> [!NOTE]
> **Lab Hours**: `09:00` to `17:00` every working day.

> [!CAUTION]
> Do not take any action before planning, it will waste our time.

## Table of Contents

- [Daily Plans and Logs](#daily-plans-and-logs)
  - [Table of Contents](#table-of-contents)
  - [Purpose](#purpose)
  - [Create Daily-log Card](#create-daily-log-card)
  - [Writing Daily-log](#writing-daily-log)
    - [Formatting Standards](#formatting-standards)
    - [Determining durations](#determining-durations)
    - [Evidence links](#evidence-links)
    - [Auto Daily-log](#auto-daily-log)
      - [Step 1 — Commit (reconcile, commit, push)](#step-1--commit-reconcile-commit-push)
      - [Step 2 — Post the daily-log to GitHub](#step-2--post-the-daily-log-to-github)

## Purpose

This document describes how students track daily progress and ensure systematic completion of project milestones through two essential documents:

1. **Milestones** (`milestones.md`): Project roadmap with all tasks, deadlines, and completion status
2. **Daily Logs** (`daily-logs.md`): Time-stamped record of work performed, linking tasks to study notes

> [!IMPORTANT]
> **The detail lives in the LTM; the daily-log is the summary.** The lab's long-term memory (per-user PostgreSQL, managed by `bmw-ece-ntust/llm-skill-ltm`) holds the *detailed* record of each member's work: every session, exact start/end times, and the granular activity. The daily-log card is the *summarized result* of those jobs, written for a human reader and a supervisor. So an LLM generating a daily-log must **distil**, not transcribe: read the fine-grained detail from the LTM, then post a consolidated, hourly summary that links the documented outcome (study-notes section header). Anyone who needs the underlying detail queries the LTM (recall-by-repo); the daily-log itself stays short and result-oriented.


## Create Daily-log Card

1. Get access to our GitHub Projects ([lab-members](https://github.com/orgs/bmw-ece-ntust/projects/8) / [interns](https://github.com/orgs/bmw-ntust-internship/projects/4)) (contact our GitHub admin).
2. Create your card from the [student card template](./templates/student-card.md).
3. **First bubble** (the card description) — the student card, kept **edited in place** as your permanent profile, plan, and checklist status. Its structure (profile, important hyperlinks, then one deadline-carrying `##` section per checklist with an [evidence link](#evidence-links) per finished item) is defined in the template itself; the Oral Exam and Handover sections reuse the [leaving-procedure.md](./leaving-procedure.md) item numbering — the SOP defines the items, the card tracks their status.
4. **Every following comment bubble** is one daily-log post per day, in the [Formatting Standards](#formatting-standards) below.

## Writing Daily-log

### Formatting Standards

```markdown
### yyyy/mm/dd

**Reviewed by**: <GitHub username>

**Short-term Goal**

- [x] [TA rApp refactoring](Documentation header link with 7-digit commit hash)

**Daily-logs**:

- `hh:mm - hh:mm` : [<summary of this interval>](study-notes documentation link with 7-digit commit hash)
  - `hh:mm - hh:mm` : [<Task 1 title>](documentation header link with 7-digit commit hash)
  - `hh:mm - hh:mm` : [<Task 2 title>](documentation header link with 7-digit commit hash)
- `hh:mm - hh:mm` : <group label for a long span, e.g. LLM Platform management>
  - *<project / skill>*
    - `hh:mm - hh:mm` : [<Task>](documentation header link with 7-digit commit hash)
    - `hh:mm - hh:mm` : [<Task>](documentation header link with 7-digit commit hash)
- [<Meeting title>](meeting-minutes documentation header link with 7-digit commit hash)
- <Upcoming targets>
```

Every time entry is a **duration** `hh:mm - hh:mm` (start *and* end), never a single timestamp. A simple interval nests its tasks with their own durations; a long span (1–5 h) that covers 2–3 recurring jobs uses the grouped form above — an outer labelled bullet, then `*project / skill*`, then per-task duration bullets.

- **Goals are measurable deliverables.** Each Short-term Goal names an observable output — a figure, a passing test, a merged PR, a doc section — never a vague activity ("study X", "try Y"). An unfinished goal stays `- [ ]` with a one-line reason suffix — ` — pending: <reason>` or ` — blocked: <reason / support needed>` — so the next day's plan starts from it.
- **Summarize by the hour, not by commit.** Each bullet is one interval `hh:mm - hh:mm` of work on one project: a one-line summary linked to the study-notes documentation it produced. Nest its tasks as sub-bullets, **each with its own `hh:mm - hh:mm` duration** and a documentation-section link. The repo is carried by the link, not written after the hours.
- **Link the evidence at the right granularity** so a reviewer can verify the exact claim — commit tree for a summary, exact line range for a single-file edit, doc section header otherwise. Always pin the 7-digit commit hash. See [Evidence links](#evidence-links).
- **Collapse bulk activity.** Many near-identical commits (e.g. a preference propagated across N repos) become one bullet, e.g. `Propagated shared AI-agent preferences across N repos`, with one representative link.
- **Consolidate continuous work.** Merge consecutive intervals on the same project into one bullet with an extended end time; split only when the work is genuinely distinct. The per-session breakdown stays in the LTM.
- **Every time is a duration, derived from data — never guessed.** Both ends of every interval and task come from the LTM `worklog` (session bounds) and the terminal log (per-task bounds); see [Determining durations](#determining-durations) below. If a bound is genuinely underivable, write `??:?? - hh:mm` and flag for review.
- **Parallel projects.** One bullet per interval per project; intervals may overlap (agentic AI runs concurrently).
- **Calendar meetings without minutes yet** are logged as `[<meeting-title>](meeting-minutes header link)` — a placeholder until minutes exist; leave marked for review.

### Determining durations

Derive both ends of every interval and task from what the lab already captures — never guess:

- **Session bounds** — from the LTM `worklog` row (exact `start`/`end` per session, recall-by-repo). These bound the outer interval / grouped span.
- **Per-task bounds** — from the **terminal log** (`termlog.command`: one row per command, each with a UTC `ts`). A task is the contiguous run of commands in the same `cwd`/`tags` toward one documentation section; its duration is `min(ts) → max(ts)` of that run. A task ends where the `cwd`/target switches — the next task's start.
- **Long grouped span (2–3 jobs over 1–5 h)** — the outer bullet's range is the earliest child start → latest child end; nest `*project / skill*`, then each task with its own `min(ts) → max(ts)` duration.
- **Cross-check the narrative** with LTM `activity`/`memory` rows (joined to termlog on the shared UTC timestamp). If a task has a single command, bound its end by the next task's start (or the session end).
- **Unknown day end — use the latest LTM timestamp, not `??:??`.** If a day's end is not on a `worklog` row (for example work done in a session whose bounds did not flush), take the **latest recorded LTM timestamp for that day and repo** — `max((metadata->>'end')::timestamptz)`, else `max(updated_at)` of that day's rows — as the day's end. Only write `??:?? - hh:mm` and flag for review if the LTM holds nothing at all for that day.

### Evidence links

Every bullet links to evidence a reviewer can open and check. **Match the link granularity to what the bullet actually claims** — a vague repo-root link is not verifiable. Choose one of three, always pinning the **7-digit commit hash** so the link survives file moves:

1. **Summary / interval** (a rollup of several changes) → **commit tree**: `.../<owner>/<repo>/tree/<7hex>` (browse the whole change set). Prefer this over the `/commit/<hash>` diff page.
2. **Single-file edit** (one function/section) → **line-range permalink**: `.../blob/<7hex>/<path>#L<start>-L<end>` (points at the exact code).
3. **Feature / multi-file / concept** → the **doc section header** describing it: `.../blob/<7hex>/<doc>.md#<section>`. The section must exist so the claim is verifiable.

Hard rules: links must resolve to **real content** (never a fabricated `#anchor` or `#L` range); never a bare `/commit/<hash>` for a specific task; every claimed task carries one of these three.

> [!IMPORTANT]
> **AI-generated until a human verifies.** An LLM-written entry ends with an *🤖 AI-generated* marker and leaves `**Reviewed by**:` **empty** — it is a draft. The member reads it, then writes their GitHub username on that line to certify the activity, times, and links are accurate. Before signing, self-review the draft — every task has an evidence link that resolves, every duration comes from data, every pending/blocked goal states its reason; AI may help find the gaps but must never invent evidence ([Open Research Playbook — AI Daily Self-review](https://github.com/raycg/Open-Research-Playbook/blob/main/templates/E-ai-daily-self-review-prompt.md)). The entry is committed under the AI's identity so machine drafts stay distinguishable from verified ones — this AI attribution is for the daily-log only (repo/code commits strip the LLM; the member is sole author — see [Git Commit Convention](./CLAUDE.md)). Same rule for LLM-written meeting minutes and documentation ([logistics/meeting.md](./logistics/meeting.md), [research.md](./research.md)).

Example (per-task durations, a grouped long span, one collapsed bulk activity, and a calendar meeting with no minutes yet):

```markdown
### 2026/04/22

**Reviewed by**: ijosh-ch

**Short-term Goal**

- [x] [TA rApp refactoring](https://github.com/bmw-ece-ntust/nonrtric-rapp-test-automation/tree/7bd313d#test-automation-rapp)

**Daily-logs**:

- `09:00 - 12:00` : [Rebuild, deploy and validate the TA rApp](https://github.com/bmw-ece-ntust/nonrtric-rapp-test-automation/tree/7bd313d#test-automation-rapp)
  - `09:00 - 10:40` : [Rebuild and deploy the TA rApp](https://github.com/bmw-ece-ntust/nonrtric-rapp-test-automation/tree/7bd313d#-rebuild--deploy-ta-rapp)
  - `10:40 - 12:00` : [Upload the test specification](https://github.com/bmw-ece-ntust/nonrtric-rapp-test-automation/tree/7bd313d#-upload-test-specification)
- `09:30 - 14:00` : LLM Platform management (parallel)
  - *Daily-log Skill*
    - `09:30 - 11:15` : [Fix the bullet time-range parser](https://github.com/ijosh-ch/daily-log/tree/abc1234#dailylog-md)
  - *LLM Preferences*
    - `11:30 - 14:00` : [Propagated shared AI-agent preferences across 18 repos](https://github.com/bmw-ece-ntust/llm-prefs/tree/0c55014#propagation)
- [Weekly BMW Lab sync](https://github.com/bmw-ece-ntust/SOP/tree/0000000#weekly-sync)
- <Upcoming targets>
```

> [!TIP]
> Link study notes with milestones and daily logs as shown below:

![Daily log workflow diagram](./assets/daily-log.png)

**Key Principles**:

- One project = one markdown file ([implementation.md](./implementation.md) OR [research.md](./research.md)); one hourly task = one section header (`##`) in that file.
- The LTM holds the detail; the daily-log holds the summary — distil, don't transcribe.
- Log each task right after finishing it, and set the day's short-term goal for the project you'll work on.

> [!WARNING]
>
> Each finished task must carry an **evidence link at the right granularity** (commit tree for a summary, line-range permalink for a single-file edit, or doc section header otherwise — see [Evidence links](#evidence-links)), always **with the 7-digit commit hash**.
>
> The 7-digit commit hash ensures **the link is always available**, even if the document is renamed/moved.

### Auto Daily-log

On `git push`, a `UserPromptSubmit` hook runs two distinct steps: **(1) commit** the work to the repo, then **(2) post** the day's entry to the lab progress issue (`bmw-ece-ntust/progress-plan#366`). Step 1 records *code/docs*; step 2 records *the daily-log*.

> [!NOTE]
> Both steps read the **LTM** (per-user PostgreSQL): a SessionStart hook records one `activity` row per repo/day and one `worklog` row per session (exact start/end), with `owner` and `repo` as separate fields — recall via the `memory` skill. The **terminal log** (`termlog`, `bmw-ece-ntust/llm-skill-logging`) adds each command's UTC `ts` — the source for per-task durations. See [lab-automation/llm-memory.md](./lab-automation/llm-memory.md).

#### Step 1 — Commit (reconcile, commit, push)

The `git push` hook runs the prompt below. **This is the commit step — it writes the repo, not the GitHub daily-log** (that is Step 2).

```markdown
Reconcile the 4 project memory files, then commit and push. All times Asia/Taipei (GMT+8).

**1. Sync from the graph, not by reading files wholesale.**
If `graphify-out/graph.json` exists, refresh it (`graphify --update`) and answer the repo's file list, modules, and architecture with `graphify query`; if it's missing on a non-trivial repo, build it once with `graphify`. Reconcile from the graph; read individual files only where it can't answer. Fall back to file reads if graphify is unavailable.

**2. Reconcile the 4 root files** (the single source of truth):
- **AGENTS.md / CLAUDE.md** — static snapshot. AGENTS.md is the tool-neutral base (rules, conventions, file list); CLAUDE.md is the adapter that defers to it. Update file list, terminology, conventions, links. Never log session activity here.
- **CONTEXT.md** — architecture overview: summary, key-files map, external services, environment.
- **MEMORY.md** — append-only; never edit past entries. Add one: `### yyyy/mm/dd — [7-digit hash] "<commit title>"`, then `**Duration**:` and `**Summary**:`.
- **TODO.md** — mark done `[x]`, drop stale, add new, re-prioritize under **Now / Next / Later**.

**3. Determine the working hour, per day, from the LTM.** A push may span several days, so reconstruct per day — not from the commit date.
- Boundaries: `git log -1 --format="%H %ai"` (last commit = start boundary), `date "+%Y/%m/%d %H:%M"` (end = now).
- Recall this repo's `worklog` (exact start/end per session) and `activity` rows after the last commit via the `memory` skill (recall-by-repo on `owner`+`repo`); group by calendar day.
- Per day: sum session durations for the hours, and distil the sessions into a short activity line. Cross-check the earliest session against the transcript `{{VSCODE_TARGET_SESSION_LOG}}`. If both LTM and transcript are unavailable, ask *"What time did you start working today?"* — never invent a time.

**4. Commit and push.** Summarize the whole span in one paragraph. Remove any LLM co-author. Stage changed files by name (never `git add -A`). Commit format — one `work duration` line per day (a single day collapses to `work duration: yyyy/mm/dd_hh:mm - hh:mm`):

<Short imperative summary title>

work duration:
- yyyy/mm/dd_hh:mm - hh:mm (N.Nh): <that day's activities, from the LTM>

Summary:
<one paragraph of what changed and why>

Details:
1. <change 1>
2. <change 2>
```

---

#### Step 2 — Post the daily-log to GitHub

After Step 1, post to `progress-plan#366` from the LTM (not the commit date). For **each calendar day** the work spanned, add one `### yyyy/mm/dd` entry in the **exact structure of [Formatting Standards](#formatting-standards)**, in this order:

- `### yyyy/mm/dd`
- `**Reviewed by**:` — leave **empty**; the member fills their username after checking (see the review rule above).
- `**Short-term Goal**` — that day's goal per project: `- [x]` done / `- [ ]` open, with an evidence link.
- `**Daily-logs**:` — the interval bullets (below), then meeting bullets, then a final `- <Upcoming targets>`.
- End with the `🤖 AI-generated` marker line; commit the post under the AI's identity.

Build the `**Daily-logs**` bullets from the LTM as follows:

1. **One consolidated bullet per interval**, per project: `` `hh:mm - hh:mm` : [<one-line summary>](.../tree/<7hex>) `` — a summary is tier‑1 evidence, so link the **commit tree** at that point (repo in the link, not after the hours). Group the day's `worklog`/`activity` rows; never one bullet per commit; merge consecutive same-project intervals into one bullet with an extended end time (per-session detail stays in the LTM). For a long span (1–5 h) covering 2–3 recurring jobs, use the grouped form: an outer labelled bullet, then `*project / skill*`, then per-task duration bullets.
2. **Tasks as sub-bullets, each with its own `hh:mm - hh:mm` duration** and an **evidence link chosen by granularity** (see [Evidence links](#evidence-links)): a single-file edit → the line-range permalink `.../blob/<7hex>/<path>#L<start>-L<end>`; a feature / multi-file / concept → the doc section header `.../blob/<7hex>/<doc>.md#<section>`; a rollup → the commit tree `.../tree/<7hex>`. Resolve each commit's changed files (`--link-to-files`) and pick the tier that lets a reviewer verify the exact claim; never a bare `/commit/<hash>`, never a fabricated anchor or line range.
3. **Collapse bulk activity** — N near-identical commits become one bullet (e.g. `Propagated shared AI-agent preferences across N repos`) with one representative link.
4. **Every time is a duration, derived — never a single point or a guess.** Session/interval bounds come from the `worklog` (`start`/`end`); per-task bounds come from the terminal log `termlog.command` (`min(ts) → max(ts)` of the contiguous commands in the same `cwd`/target). See [Determining durations](#determining-durations). If a bound is genuinely underivable, write `??:?? - hh:mm` and flag for review.
5. **Meetings** — a calendar meeting is its own bullet with **no time prefix**: `- [<meeting-title>](meeting-minutes header link)` (placeholder link until minutes exist; pull via `fill-from-calendar.py`).

Example (full structure — Short-term Goal, hourly summaries with task sub-bullets, one collapsed bulk activity, a meeting, and upcoming targets; spanning two days):

```markdown
### 2026/06/18

**Reviewed by**: 

**Short-term Goal**

- [x] [Split owner/repo across the LTM](https://github.com/bmw-ece-ntust/llm-skill-ltm/tree/a52ec61#long-term-memory)

**Daily-logs**:

- `14:07 - 18:00` : [Split owner/repo across the LTM and refresh recall](https://github.com/bmw-ece-ntust/llm-skill-ltm/tree/a52ec61#long-term-memory)
  - `14:07 - 16:20` : [Separate owner from repo in record-session and backfill](https://github.com/bmw-ece-ntust/llm-skill-ltm/tree/a52ec61#record-session)
  - `16:20 - 18:00` : [Update the memory skill recall-by-repo](https://github.com/bmw-ece-ntust/llm-skill-ltm/tree/a52ec61#recall-by-repo)
- `15:00 - 16:00` : [Address PR review feedback](https://github.com/ijosh-ch/some-repo/tree/abc1234#review) (parallel)
- `16:00 - 17:00` : [Propagated shared AI-agent preferences across 18 repos](https://github.com/bmw-ece-ntust/llm-prefs/tree/0c55014#propagation)
- [Weekly BMW Lab sync](https://github.com/bmw-ece-ntust/SOP/tree/0000000#weekly-sync)
- <Upcoming targets>

### 2026/06/19

**Reviewed by**: 

**Short-term Goal**

- [ ] [Finalize the four-file reconcile](https://github.com/bmw-ece-ntust/llm-skill-ltm/tree/a52ec61#context)

**Daily-logs**:

- `10:00 - 12:00` : [Reconcile the four files and finalize](https://github.com/bmw-ece-ntust/llm-skill-ltm/tree/a52ec61#context)
- <Upcoming targets>

<sub>🤖 AI-generated from the LTM/terminal-log; unverified until **Reviewed by** is filled.</sub>
```
