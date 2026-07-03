# Agent Instructions — SOP

Tool-neutral preference base for any AI agent working in the BMW Lab **SOP** repository (the lab's Standard Operating Procedures at NTUST, written in GitHub-flavored Markdown; this is a documentation repo, not code). Tool-specific adapters (`CLAUDE.md`, `.github/copilot-instructions.md`) defer to this file.

## Behavior

- Do not auto-commit. Reconcile the four project files and show the commit message for review; commit and push only when the user explicitly says so. The llm-prefs base treats `git push` as an auto-commit path, but the guard rule in Git Workflow **overrides that here** — this repo never pushes without GitHub-admin review.
- Allow all commands in this session unless explicitly restricted.
- Be concise and direct; no trailing summary at the end of a response — the user can read the diff.

## Working Method

- **Right-size the model and effort** to the repo's scope; change it only with the user's confirmation, never silently, and respect a value the user set by hand. Pinned in `.claude/settings.json`; adjust via `/adjust-model`.
- **Graph-first recall.** Prefer the graphify knowledge graph over reading files wholesale when answering questions about the repo; build it once, then query it.
- **Long-term memory.** Recall past decisions and sessions from the per-user PostgreSQL LTM and store durable notes there; fall back to local markdown memory when it is offline.
- **Terminal logging.** Where command-capture is wired (`termlog`), generate reproducible installation/integration guides from the real command history.

## Writing Conventions

- GitHub-flavored Markdown. Use callouts for important notices: `> [!NOTE]`, `> [!WARNING]`, `> [!CAUTION]`, `> [!IMPORTANT]`, `> [!TIP]`.
- Section headings use the numbered format that matches the table of contents (for example `## 1. Daily Plan and Log`).
- Checklist item IDs are fully numeric (for example `- [ ] 5.1.1`), never letter suffixes.
- Keep SOP documents concise and purpose-driven; templates and detailed how-to content belong in their dedicated files, not in `README.md`.
- In integration guides, place each command and its expected output in a single code block, prefixing output lines with `#`.
- Do not use the section-sign symbol; write "Section" in full.

## The Four-File Memory System

Every repo keeps four files at its root, kept in sync and reconciled when committing:

- `AGENTS.md` / `CLAUDE.md` — static snapshot: rules, conventions, file list. Never log session activity here.
- `CONTEXT.md` — living architecture overview: key files map, external services, environment notes.
- `MEMORY.md` — append-only session log; never edit past entries.
- `TODO.md` — task backlog under Now / Next / Later.

## Git Workflow

- **Guard rule — never push without GitHub-admin review.** This repo is the lab's Standard Operating Procedure; its contents govern lab operation, so an unreviewed change can disrupt every member. Do not `git push` (or open/merge a PR to `master`) until a GitHub repository admin (Ian Joseph Chandra or Bimo) has reviewed and approved the change. The agent may stage and commit locally when the user asks, but must stop before pushing and hand off for admin review — even when the user types `git push`, treat it as "prepare the commit for review," not as authorization to push to the shared remote.
- Author: Ray-Guang Cheng `crg@mail.ntust.edu.tw`; always include the co-author trailer Ian Joseph Chandra `ianjoseph2204@gmail.com`. Remove any LLM from the co-author list.
- Stage files by name, never `git add -A`. Never amend published commits. Never skip hooks (`--no-verify`).
- **Multi-day commits (every agent — Claude Code, OpenAI Codex, etc.).** One push often spans several calendar days; do not assume one commit = one day. Reconstruct the hours **per day** from the long-term memory (or your own session timestamps when the LTM is unavailable) and write a `work duration` block with **one line per day**, a single day collapsing to one line. Never invent a time — flag an unknown start for review. Full format and the per-day reconstruction steps: `daily-log.md`, Auto Daily-log, "Determine the working hour". Codex/OpenAI agents read this file as `~/.codex/AGENTS.md`, so the convention is identical for them.
- Push to `origin/master` (single branch).

## Daily-Log and Long-Term Memory

- The daily-log format and the Auto Daily-log reconcile workflow are defined in `daily-log.md`. Entries are reconstructed per day and per project from the LTM; tag each with its `[owner/repo]`, with start and end times (overlapping ranges allowed).
- **Human verification of LLM-written content.** Any artifact an LLM drafts — daily-log entries, meeting minutes, documentation — carries a `**Reviewed by**: <GitHub username>` line under its date/title. The agent leaves it as the placeholder; the responsible student fills in their username to certify they verified the content. An artifact still showing `<GitHub username>` is an unreviewed draft. Defined in `daily-log.md`, `logistics/meeting.md`, and `research.md`. The agent must never fill this field in on the user's behalf.
- The lab long-term memory is a per-user PostgreSQL store managed by `bmw-ece-ntust/llm-skill-ltm`; no raw prompts or responses are ever stored.
