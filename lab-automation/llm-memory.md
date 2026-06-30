# LLM Long-Term Memory — PostgreSQL Setup Guide

This document describes the lab's long-term memory (LTM) for LLM sessions: a **per-user PostgreSQL** store on the BMW Lab box that persists distilled work activity across commits, projects, and machines.

> [!IMPORTANT]
> The LTM is implemented and maintained in the **[`bmw-ece-ntust/llm-skill-ltm`](https://github.com/bmw-ece-ntust/llm-skill-ltm)** repo. This guide is the SOP overview; the repo is the source of truth for scripts, schema, and setup. The previous MySQL `llm_memory` setup is retired.

## Table of Contents

- [LLM Long-Term Memory — PostgreSQL Setup Guide](#llm-long-term-memory--postgresql-setup-guide)
  - [Table of Contents](#table-of-contents)
  - [1. What It Stores](#1-what-it-stores)
  - [2. Topology](#2-topology)
  - [3. Setup](#3-setup)
  - [4. Recording Activity](#4-recording-activity)
  - [5. Recall During a Session](#5-recall-during-a-session)
  - [6. Privacy and Attribution](#6-privacy-and-attribution)

## 1. What It Stores

The LTM never stores raw prompts or responses. It stores three kinds of rows in one `memory` table:

| Layer | `type` | Written by | Granularity |
| --- | --- | --- | --- |
| Activity | `activity` | SessionStart hook (`record-session.sh`) + `memory-backfill.sh` | One row per repo per day |
| Worklog | `worklog` | `stm-backup.sh` | One row per session, with **exact start/end timestamps** |
| Knowledge | `session` / `reference` | the daily-log commit workflow + the `memory` skill | Distilled summary plus an evidence link |

Each row's `metadata` (JSONB) keeps the project identity in **separate fields**: `owner` (the GitHub account that owns the repo, an organization such as `bmw-ece-ntust` or a personal account) and `repo` (the repository name only). It also carries `user`, `machine`, `branch`, `date`, and, for worklogs, `start`/`end`. This lets the daily-log group activities per project and recover accurate start/end times.

## 2. Topology

Each member has their own PostgreSQL database on the lab box (role named after the GitHub username). PostgreSQL listens only on the server's localhost, so every machine reaches its database over an SSH tunnel. Because all machines write to the same per-user database, sessions collect per repo across machines. The database password is never stored in a file; `scripts/pg-memory-conn.sh` fetches it from the self-hosted Vaultwarden at runtime and opens the tunnel.

## 3. Setup

Setup is automated by the `llm-skill-ltm` repo. On a new machine:

```bash
# Clone the LTM repo and run its one-shot setup (idempotent).
git clone https://github.com/bmw-ece-ntust/llm-skill-ltm.git
cd llm-skill-ltm
cp .env.example .env   # fill in your Vaultwarden item + SSH user (no secrets in git)
bash setup.sh          # installs deps, pins the lab CA, runs install.sh, applies schema.sql, verifies the tunnel
```

`install.sh` copies the `memory`, `memory-backfill`, and `stm-backup` skills into `~/.claude/skills`, sets `LTM_HOME`, and wires the SessionStart activity hook. Because the skills live in `~/.claude/skills`, the same LTM works in both Claude Code and Cowork.

## 4. Recording Activity

Recording is automatic. The SessionStart hook writes an `activity` row when you open a lab repo. To back up exact session start/end times as `worklog` rows, or to backfill past sessions, run the helper scripts from the LTM repo:

```bash
bash "$LTM_HOME/scripts/stm-backup.sh"        # one worklog row per session (exact timestamps)
bash "$LTM_HOME/scripts/memory-backfill.sh"   # one activity row per repo per day, from past transcripts
```

No manual SQL inserts are required for normal use.

## 5. Recall During a Session

At the start of a session, recall recent work for the current project with the `memory` skill (recall-by-repo):

```sql
SELECT metadata->>'date', metadata->>'machine', metadata->>'branch', type, description
FROM memory
WHERE metadata->>'owner' = :'owner' AND metadata->>'repo' = :'repo'
ORDER BY metadata->>'date' DESC;
```

To feed the daily-log, combine commit history with the matching `worklog` rows to recover each session's start and end time, then emit one bullet per session tagged with its `[owner/repo]` project. Overlapping time ranges are allowed because agentic AI runs tasks concurrently.

## 6. Privacy and Attribution

No raw prompts or responses are ever stored. Every row is attributed by GitHub account (`metadata.user` = `gh api user --jq .login`) and machine (`metadata.machine` = `hostname -s`); lab performance is measured by GitHub account. Register your PostgreSQL role with the same name as your GitHub username so `current_user` corroborates the attribution. Full design: [`llm-skill-ltm/docs/ARCHITECTURE.md`](https://github.com/bmw-ece-ntust/llm-skill-ltm/blob/master/docs/ARCHITECTURE.md).
