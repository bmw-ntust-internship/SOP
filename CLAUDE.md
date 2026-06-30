# CLAUDE.md — Project Knowledge for AI Assistants

This file provides persistent context about this repository for AI assistants (GitHub Copilot, Claude, etc.) so they can assist effectively across sessions without re-reading all files every time.

---

## What This Repository Is

This is the **Standard Operating Procedure (SOP)** repository for the **BMW Lab** (Broadband Multimedia Wireless Networks) at **NTUST** (National Taiwan University of Science and Technology), supervised by **Prof. Ray**.

It is a collection of Markdown documents that define rules, workflows, and checklists for students and researchers in the lab. It is **not** a software project — there is no source code to build or test.

The primary audience is:

- New graduate students (Master's and Ph.D.) joining the lab
- Interns assigned to the lab
- Ph.D. students managing lab operations (e.g., Ian Joseph Chandra, Bimo)

---

## Repository Structure

```shell
SOP/
├── README.md                  # Main SOP overview — entry point for all lab rules
├── daily-log.md               # How to write daily plans and logs (format, workflow, automation)
├── leaving-procedure.md       # Full leaving procedure: graduation, professor change policy, handover checklists, NDA requirement
├── .github/
│   ├── copilot-instructions.md  # GitHub Copilot project instructions (mirrors CLAUDE.md purpose)
│   └── workflows/
│       └── sync-sop.yml         # GitHub Actions workflow: auto-syncs selected SOP files to the intern org repo
├── .gitignore                 # Git ignore rules (.DS_Store, .claude/settings.local.json, graphify-out/)
├── .vscode/
│   └── mcp.json                 # VS Code MCP server config (Microsoft 365 Agents Toolkit MCP)
├── integration-guide.md       # Guideline for writing integration guides (covers installation, end-to-end integration, and post-installation verification for O-RAN interfaces)
├── user-guide.md              # Guideline for writing user guides for projects
├── paper-writing.md           # Steps and tips for writing and revising IEEE papers
├── project-documentation.md   # Guidelines for project documentation (architecture, diagrams)
├── source-code-guide.md       # Source code standards: OOP, design patterns (Adapter/Factory/Strategy), Sphinx/Doxygen, folder structure
├── AGENTS.md                  # Tool-neutral AI-agent preference base; tool adapters (CLAUDE.md) defer to it (seeded by sync-to-all-repos.sh)
├── .claude/
│   └── settings.json          # Per-repo Claude model/effort pin (settings.local.json is gitignored)
├── CLAUDE.md                  # This file — LLM knowledge snapshot (static)
├── CONTEXT.md                 # Living architecture overview: key files map, external services, git convention
├── MEMORY.md                  # Append-only session decisions log (one dated entry per session)
├── TODO.md                    # Task backlog: Now / Next / Later
├── assets/                    # Images and static assets
├── NDA/
│   └── nda-template.md        # Lab Non-Disclosure Agreement template — signed by all departing members before the Professor approves their departure
├── lab-automation/
│   ├── credentials-vault.md   # Credential management practices
│   └── llm-memory.md          # PostgreSQL long-term memory setup (per-user; managed by llm-skill-ltm)
├── lab-internal/
│   └── stipend.md             # Stipend-related information
├── logistics/
│   ├── meeting.md             # Meeting preparation and protocol guidelines
│   └── teep-preparation.md    # TEEP program preparation guide
└── templates/
    ├── paper/                  # IEEE paper templates
    └── slide/                  # Presentation slide templates
```

---

## Key Concepts and Terminology

| Term | Meaning |
|---|---|
| **BMW Lab** | Broadband Multimedia Wireless Networks Lab at NTUST |
| **Prof. Ray** | Lab supervisor |
| **Ian / Bimo** | Ph.D. students who review and manage lab handovers |
| **Daily Log** | Time-stamped daily work record; tracked via GitHub Projects card |
| **Milestones** | Project roadmap (`milestones.md` in each student's repo) |
| **Handover** | Process of a 2nd-year student transferring their project to a 1st-year student |
| **Oral Exam** | Thesis defense; governed by NTUST academic calendar |
| **pCloud** | Cloud storage used for datasets, demo videos, and thesis materials |
| **rApp** | RAN application (O-RAN ecosystem); common project type in this lab |
| **Main Research Repo** | Student's private GitHub repo for thesis code + docs + simulation |
| **Sub-module Repo** | Supporting repo for auxiliary components (e.g., OAI gNB modifications) |

---

## Writing and Formatting Conventions

- All documents are written in **GitHub-flavored Markdown**.
- Use GitHub callout syntax for important notices:
  - `> [!NOTE]` — informational
  - `> [!WARNING]` — important rules to follow
  - `> [!CAUTION]` — actions to avoid
  - `> [!IMPORTANT]` — critical requirements
  - `> [!TIP]` — helpful suggestions
- Section headings use numbered format matching the TOC (e.g., `## 1. Daily Plan and Log`).
- File links use relative paths (e.g., `[Daily Log](./daily-log.md)`).
- Checklist items use `- [ ]` with fully numeric IDs (e.g., `- [ ] 5.1.1`, `- [ ] 5.1.1.1`). Do **not** use letter suffixes (e.g., `5.1.1.a`).
- Keep SOP documents **concise and purpose-driven** — templates and detailed how-to content belong in their dedicated files, not in `README.md`.
- In integration guides, place each command and its expected output in a **single code block**; prefix output lines with `#` so they read as comments. Long or noisy outputs should be redirected to a `./logs/` folder instead of printed inline.

---

## Git Commit Convention

### Commit authorship

All commits in this repo use:

- **Author:** Ray-Guang Cheng `crg@mail.ntust.edu.tw` (set via `git config user.*` — local repo config)
- **Co-author:** Ian Joseph Chandra `ianjoseph2204@gmail.com` — always include the trailer below

### Commit message format

When committing changes to this repo, use the following structure:

```
<Short imperative summary title>

work duration:
- yyyy/mm/dd_hh:mm - hh:mm (N.Nh): <what was done that day, from the LTM>
- yyyy/mm/dd_hh:mm - hh:mm (N.Nh): <what was done that day, from the LTM>

Summary:
<One-paragraph summary of what changed and why>

Details:
1. <Specific change 1>
2. <Specific change 2>

Co-authored-by: Ian Joseph Chandra <ianjoseph2204@gmail.com>
```

**Working hour field rules** (see `daily-log.md`, Auto Daily-log, "Determine the working hour"):
- A single push can cover work done across several days. Reconstruct the hours and activities **per day** from the LTM (`worklog`/`activity` rows after the last commit), not from the commit date.
- One `work duration` line per day: `yyyy/mm/dd_hh:mm - hh:mm (N.Nh): <activities that day>`. Hours are that day's summed session durations; activities are distilled from that day's LTM sessions.
- A single-day session collapses to one line: `work duration: yyyy/mm/dd_hh:mm - hh:mm`.
- All times are 24-hour Asia/Taipei. The whole-span start is the earliest user-message timestamp after the latest commit.

Push to `origin/master` (this repo uses a single `master` branch).

---

## How to Maintain This File

> [!IMPORTANT]
> **Only reconcile this file and run `git commit` / `git push` when the user explicitly issues the command.** Do not perform these steps automatically at the end of a session.

At the end of each working session, students use **Prompt B** (defined in `daily-log.md §Auto Daily-log`) to reconcile all 4 project memory files and commit the result:

- **CLAUDE.md** — static snapshot: file list, terminology, conventions, external links
- **CONTEXT.md** — living architecture overview: key files, external services, environment notes
- **MEMORY.md** — append-only decisions log: one `## yyyy-mm-dd` entry per session
- **TODO.md** — task backlog: mark done, add new, re-prioritize under Now / Next / Later

**What belongs here:**
- New files added to the repo (with descriptions)
- Renamed sections or files
- New terminology or lab-specific concepts
- New writing/formatting conventions
- New external links

**What does NOT belong here:**
- Prompting activity logs or changelogs
- Session-specific notes or temporary context
- Step-by-step instructions (those belong in their dedicated `.md` files)

---

## Important External Links

- **BMW Lab GitHub Org**: https://github.com/orgs/bmw-ece-ntust
- **BMW Lab YouTube Channel**: https://www.youtube.com/@ntust.bmw-lab
- **Daily-Log Automation Repo**: [bmw-ece-ntust/progress-plan](https://github.com/bmw-ece-ntust/progress-plan) (local clone: `~/Documents/GitHub/daily-logs/`, tool at `src/dailylog/`)
- **Lab Members GitHub Project board**: https://github.com/orgs/bmw-ece-ntust/projects/8
- **Interns GitHub Project board**: https://github.com/orgs/bmw-ntust-internship/projects/4
- **Project template repo**: https://github.com/bmw-ece-ntust/template
- **Thesis pCloud folder**: http://u.pc.cd/Dbd
- **NTUST academic calendar**: https://www.academic.ntust.edu.tw/p/412-1048-8756.php?Lang=en

---

## Long-Term Memory (PostgreSQL)

The lab long-term memory is a **per-user PostgreSQL** store on the BMW Lab box (one DB per member, role named after the GitHub username), reached over an SSH tunnel. It is managed by the [`llm-skill-ltm`](https://github.com/bmw-ece-ntust/llm-skill-ltm) repo and replaces the previous MySQL setup. No raw prompts/responses are stored — only distilled knowledge plus `activity` and `worklog` rows attributed by GitHub account. Each row's `metadata` keeps `owner` (the GitHub account, org or personal) and `repo` (name only) as separate fields, so activities group per project for the daily-log. Setup and usage: `lab-automation/llm-memory.md`.

**At the START of every session**, recall recent work for the current project with the `memory` skill (recall-by-repo):

```sql
SELECT metadata->>'date', metadata->>'machine', metadata->>'branch', type, description
FROM memory
WHERE metadata->>'owner' = :'owner' AND metadata->>'repo' = :'repo'
ORDER BY metadata->>'date' DESC;
```

Session activity (`activity` per day, `worklog` per session with exact start/end timestamps) is recorded automatically by the `llm-skill-ltm` SessionStart hook — no manual inserts.

> **Agent rule:** Only execute the reconcile-CLAUDE.md and git commit/push steps when the user explicitly inserts the command. Do not run these automatically.
