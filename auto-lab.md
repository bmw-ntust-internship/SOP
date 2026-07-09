<h1 align="center">Auto-Lab — Lab Automation for New Students</h1>

---

> [!NOTE]
> **Who this is for.** New lab members setting up their machine and wondering what all the `llm-*` repos are. This is the essential map of the BMW Lab automation system: what runs automatically, what each repo does, and how to get wired up in one prompt. The hands-on setup walkthrough is [`llm-core/docs/GETTING-STARTED.md`](https://github.com/bmw-ece-ntust/llm-core/blob/master/docs/GETTING-STARTED.md).

## 1. The Idea

The lab automates the *administrative* half of research so you spend your time on the *thinking* half. Once set up, your AI assistant follows the lab working method everywhere, your daily-log writes itself when you `git push`, your work history is remembered in a lab long-term memory (LTM), and credentials are brokered without anyone emailing passwords. Everything is keyed on one unique ID: **your GitHub username**.

## 2. The System Map

One root repo governs LLM behaviour and imports the skills from satellite repos:

| Repo | Role | What it gives you |
| --- | --- | --- |
| [`llm-core`](https://github.com/bmw-ece-ntust/llm-core) *(to be renamed `llm-core`)* | **Root** — LLM behaviour + skills registry | Lab preferences for Claude Code / Copilot / Codex / Cowork, the tool-neutral skill index (`skills/MANIFEST.md`), onboarding (`/register-member`), model right-sizing (`/adjust-model`), repo onboarding (`/project-init`) |
| [`daily-log`](https://github.com/bmw-ece-ntust/daily-log) | Daily-log automation | `/daily-plan` (morning targets), auto commit-and-log on `git push`, `/daily-log` (evening posting to `progress-plan#366`), `/daily-log-audit` (full-history conformance repair) |
| [`llm-skill-ltm`](https://github.com/bmw-ece-ntust/llm-skill-ltm) | Long-term memory | Per-user PostgreSQL on the lab box: automatic activity rows per repo/day, exact-time worklogs, `/memory` recall — SOP overview in [lab-automation/llm-memory.md](./lab-automation/llm-memory.md) |
| [`llm-skill-logging`](https://github.com/bmw-ece-ntust/llm-skill-logging) | Terminal logging | `/termlog` captures every command + output; `/integration-guide` turns your real command history into an SOP-format guide |
| [`llm-skill-creds`](https://github.com/bmw-ece-ntust/llm-skill-creds) | Credential layer | HashiCorp Vault access without ever seeing the secret: `/creds-access`, leveled + audited + revocable — SOP policy in [lab-automation/credentials-vault.md](./lab-automation/credentials-vault.md) |

The full catalog of skills across all repos: [lab-automation/skills.md](./lab-automation/skills.md).

## 3. What Happens Automatically

Once registered, with **zero extra effort from you**:

| You do | The system does |
| --- | --- |
| Open a lab repo in your editor | Writes an `activity` row (repo, day) to the LTM — your working days are provable |
| Type `git push` | Reconciles the four project files, reconstructs working hours from the LTM, commits in SOP format, pushes, writes a memory record |
| Run terminal commands | `termlog` records command, output, exit code, timestamp — later turned into integration guides |
| Start any session | Lab preferences and your identity load; past work on the repo is recallable with `/memory` |

> [!IMPORTANT]
> **Automation never removes your responsibility to verify.** Every auto-generated daily-log entry has a `**Reviewed by**: <GitHub username>` line — filling it certifies *you* checked it. The same applies to LLM-generated minutes and documentation ([daily-log.md](./daily-log.md)).

## 4. Get Set Up (once, ~15 minutes + admin steps)

1. **Prerequisites** — GitHub org membership, `gh auth login`, and the admin-provisioned items (Vault login, SSH to the lab box, your memory DB). Full table: [GETTING-STARTED §1](https://github.com/bmw-ece-ntust/llm-core/blob/master/docs/GETTING-STARTED.md#1-prerequisites-do-these-by-hand-once).
2. **One-touch bootstrap** — `curl -fsSL https://raw.githubusercontent.com/bmw-ece-ntust/llm-core/master/setup/bootstrap.sh | bash` clones the lab repos and installs the preference into your AI tool.
3. **Register** — open your assistant in the `llm-core` repo and run `/register-member`. It derives every per-user resource from your GitHub login, wires preferences + skills + LTM, drafts your credential-access request (`/creds-register`), and tells you exactly which admin steps are still pending.

Until the admin steps land, memory falls back to local markdown — you are never blocked.

## 5. A Day With the Automation

- **Morning** — `/daily-plan`: state today's targets in plain words; it posts a measurable checklist to the progress board.
- **During the day** — do research; write code with the AI following [vibe-coding.md](./vibe-coding.md); commit and push as you complete logical changes (each push logs itself).
- **Evening** — `/daily-log`: sweeps uncommitted lab repos, then fills the morning's checklist with actual durations and documentation evidence.
- **Periodically** — `/daily-log-audit`: rechecks the whole log against the current SOP format and repairs gaps.

## 6. Ground Rules

- **One unique ID.** Everything (Vault path, Postgres role/DB, log attribution) derives from `gh api user --jq .login`. Never invent names by hand.
- **No secrets in files, prompts, or commits.** Secrets live in Vault; skills broker access without displaying values ([credentials-vault.md](./lab-automation/credentials-vault.md)).
- **No raw transcripts in memory.** The LTM stores distilled activity/worklog/knowledge rows only — never your prompts or the model's responses ([llm-memory.md](./lab-automation/llm-memory.md)).
- **Lab orgs only.** Automation tracks `bmw-ece-ntust`, `bmw-ntust-internship`, and `raycg`; personal repos are yours alone.

## 7. Where to Go Next

- [lab-automation/skills.md](./lab-automation/skills.md) — every skill the lab already has, by domain.
- [vibe-coding.md](./vibe-coding.md) — how to produce industrial-grade code with the AI.
- [GETTING-STARTED.md](https://github.com/bmw-ece-ntust/llm-core/blob/master/docs/GETTING-STARTED.md) — the full setup walkthrough.
