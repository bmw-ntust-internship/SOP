# Lab Skills Catalog — What Your AI Assistant Already Knows How to Do

> [!NOTE]
> **Who this is for.** Lab members who have run `/register-member` ([auto-lab.md](../auto-lab.md)) and want to know what they can ask for. A **skill** is a workflow written in plain markdown, so any LLM on any platform (Claude Code, Cowork, Copilot, Codex) can follow it. Trigger a skill by its slash command or by simply describing the task. The machine-readable registry is [`llm-core/skills/MANIFEST.md`](https://github.com/bmw-ece-ntust/llm-core/blob/master/skills/MANIFEST.md).

## Onboarding

> Full registration procedure (member + admin steps, access levels, verification): [registration.md](./registration.md).

| Skill | Say / type | What it does | Lives in |
| --- | --- | --- | --- |
| `register-member` | `/register-member`, "set me up" | One-touch setup of the whole LLM system (preferences, skills, LTM) keyed on your GitHub username; invokes `creds-register` for your Vault identity | `llm-core` |
| `creds-register` | "register me for server access" | Drafts a credential-access request for admin approval (never self-grants); also the admin Google-Form batch path | `llm-skill-creds` |
| `project-init` | "onboard this repo" | Builds the graphify knowledge graph once, creates the four project files, right-sizes the model | `llm-core` |

## Daily-log

| Skill | Say / type | What it does | Lives in |
| --- | --- | --- | --- |
| `daily-plan` | `/daily-plan` (morning) | Turns your plain-words plan into a measurable checklist on `progress-plan#366` | `daily-log` |
| `daily-log-commit` | just type `git push` | Reconciles the four project files, commits in SOP format with working hours from the LTM, pushes, writes a memory record | `daily-log` |
| `daily-log` | `/daily-log` (evening) | Commits + pushes all pending lab repos, then fills the day's checklist with durations and evidence | `daily-log` |
| `daily-log-audit` | `/daily-log-audit` | Full-history audit against the current SOP format: rewrites non-conforming entries, fills missing days (`--since <date>` = quick gap-check). *Supersedes the retired `verify-daily-log`.* | `daily-log` |
| `bmw-lab-daily-log` | "format my daily log" (Cowork) | Knowledge-only formatter: turns rough notes into a correctly structured entry | `llm-core` (Cowork plugin) |

## Long-Term Memory

| Skill | Say / type | What it does | Lives in |
| --- | --- | --- | --- |
| `memory` | `/memory`, "recall past work on this repo" | Recall or store distilled long-term memory, per repo across machines | `llm-skill-ltm` |
| `memory-gui` | "browse the LTM" | Opens a local web UI (pgweb) over the SSH tunnel | `llm-skill-ltm` |
| `stm-backup` | "back up session worklogs" | One worklog row per session with exact start/end times | `llm-skill-ltm` |
| `memory-backfill` | "back up past sessions" | One activity row per repo per day from local transcripts | `llm-skill-ltm` |
| `ltm-flush` | "drain the cowork spool" | Flushes Cowork's offline activity/worklog spool into Postgres | `llm-skill-ltm` |

## Terminal Logging → Documentation

| Skill | Say / type | What it does | Lives in |
| --- | --- | --- | --- |
| `termlog` | "provision terminal logging" | Captures every command (output, exit code, timestamp) into the lab DB; Cowork spools offline | `llm-skill-logging` |
| `integration-guide` | "make a guide from my commands" | Turns your captured command history into a BMW Lab SOP-format integration guide | `llm-skill-logging` |

## Understand a Codebase Fast

| Skill | Say / type | What it does | Lives in |
| --- | --- | --- | --- |
| `graphify` | `/graphify` | Builds + queries a knowledge graph of a repo, keeping token use flat | external (`graphifyy`, pip) |
| `study-repo` | "study / analyze this repo" | Graphify-first analysis: answers architecture questions from `graphify-out/`, never raw-scans source | `llm-core` (Cowork plugin) |
| `adjust-model` | "right-size the model for this repo" | Judges and pins the best model + effort tier from the repo's scope | `llm-core` |

## Credentials (Vault)

| Skill | Say / type | What it does | Lives in |
| --- | --- | --- | --- |
| `creds-access` | "ssh into `<user>@<host>`", "I need the WiFi password" | Brokers a short-lived, audited credential — you never see the raw secret | `llm-skill-creds` |
| `creds-admin` | "grant/revoke `<member>` access" | **Admin-only**: grant, change, revoke levels + server scope; provision Vault identities | `llm-skill-creds` |
| `creds-audit` | "who accessed what?" | **Admin-only**: review the Vault audit log per member/secret | `llm-skill-creds` |

## Lab Operations (admin)

| Skill | Say / type | What it does | Lives in |
| --- | --- | --- | --- |
| `llm-deploy-prod` | "deploy prefs to prod" | Down-syncs the `bmw-ece-ntust` source repos into their `bmw-ntust-internship` forks | `llm-core` |

---

## Retired / Merged Skills

Kept here so old docs and habits resolve to the right place:

| Old | Now | Why |
| --- | --- | --- |
| `verify-daily-log` | `daily-log-audit` (use `--since <date>` for the old quick scope) | Audit is a superset: gap-fill **and** format conformance |
| `llm-creds` (in `llm-core`) | The four `creds-*` skills in `llm-skill-creds`; admin guide at [`docs/vault-admin-guide.md`](https://github.com/bmw-ece-ntust/llm-skill-creds/blob/master/docs/vault-admin-guide.md) | Credential layer extracted to its own repo, split by role (member / admin) |

## Adding a Skill

1. Write `skills/<name>/SKILL.md` in the owning repo as plain, tool-neutral instructions (see the [neutrality rules](https://github.com/bmw-ece-ntust/llm-core/blob/master/skills/MANIFEST.md#neutrality-rules-for-writing-a-skill)).
2. Register it in `llm-core/skills/MANIFEST.md` (and the owning repo's `catalog.md`).
3. Add it to this catalog under the right domain, then re-run the owning repo's `install.sh`.
