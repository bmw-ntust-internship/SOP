# New-Member Registration — Skills, Vault, LTM, Termlog

> [!NOTE]
> **Who this is for.** A new lab member registering for the automation system, and the **admin** processing that registration. Overview of the system itself: [auto-lab.md](../auto-lab.md). Hands-on source of truth: [`llm-core` README (Getting Started)](https://github.com/bmw-ece-ntust/llm-core/blob/master/README.md#getting-started-new-members).

## How Registration Works

Everything is keyed on **one unique ID — your GitHub username** (`gh api user --jq .login`). From that single login the system derives your Vault KV path (`secret/lab/ltm/<login>/db`), your Postgres role (`<login>-llm`) and database (`<login>-memory`), and your daily-log attribution. You never invent a name by hand, and the admin can revoke everything by that same ID when you leave.

The flow has three phases:

```text
MEMBER requests            ADMIN provisions                MEMBER verifies
/register-member  ──────►  Vault identity + policies  ──►  store secret, re-run
(drafts everything,        Postgres role + DB              /register-member,
 works locally already)    SSH access                      confirm all tiers green
```

Nothing blocks you while you wait: until the admin steps land, memory falls back to local markdown and every other skill already works.

## Part A — Member Steps

### A1. Prerequisites (by hand, once)

| # | Item | How |
| --- | --- | --- |
| 1 | GitHub account in the lab org | Ask the GitHub admin to add you to `bmw-ece-ntust` (interns also `bmw-ntust-internship`) |
| 2 | `gh` CLI authenticated | Install [`gh`](https://cli.github.com/), run `gh auth login` — this is where your unique ID comes from |
| 3 | An AI coding assistant | Claude Code (recommended), Copilot, or Codex — the bootstrap installs an adapter for each |

### A2. Bootstrap (one line)

```bash
curl -fsSL https://raw.githubusercontent.com/bmw-ece-ntust/llm-core/master/setup/bootstrap.sh | bash
```

Clones the lab repos into `~/Documents/GitHub/` and installs the lab preference into your AI tool.

### A3. Register (one prompt)

Open your assistant in the `llm-core` repo and run **`/register-member`** (or paste the getting-started prompt from [README Getting Started, Section 2](https://github.com/bmw-ece-ntust/llm-core/blob/master/README.md#2-one-touch-deploy-let-the-llm-do-it)). It will:

1. Read your login from `gh api user` and confirm it with you.
2. Generate your gitignored `identity.sh` + LTM `.env` and run the installers — **skills** are now live (daily-log on `git push`, `/daily-plan`, `/project-init`, `/termlog`, ...).
3. Draft your **Vault access request** via the `creds-register` workflow (baseline **L1** = project KV read; ask for more only if your project needs a server — see [credentials-vault.md](./credentials-vault.md)).
4. Report exactly which admin steps (Part B) are still pending for you.

### A4. After the Admin Provisions You

1. **SSH key to the lab box** (needed for the LTM tunnel): `ssh-copy-id wifi@140.118.122.119`.
2. **Store your LTM connection string** (printed by the admin's provisioning step) in your Vault path:

   ```bash
   bash ~/Documents/GitHub/llm-core/home/prompts/vault-create-memory-secret.sh
   ```

3. **Re-run `/register-member`** — it re-verifies every tier and should now report all green.

### A5. Verify Your Registration

| Check | How | Expect |
| --- | --- | --- |
| Skills installed | Type `git push` in a lab repo | Daily-log workflow triggers automatically |
| Vault identity | `vault login` then `/creds-access` a permitted secret | Access brokered, secret never displayed |
| LTM reachable | `/memory` in a lab repo | Recall of your recorded activity (not the markdown fallback) |
| Termlog capturing | `/termlog` → "check terminal logging" | Recent commands present in the `termlog` schema |

## Part B — Admin Steps (once per member)

Triggered by the member's `creds-register` request. Full command detail: [`llm-skill-creds/docs/vault-admin-guide.md`](https://github.com/bmw-ece-ntust/llm-skill-creds/blob/master/docs/vault-admin-guide.md) and `/creds-admin`.

| # | Step | Tool / command |
| --- | --- | --- |
| 1 | Add member to the GitHub org (+ Project board) | GitHub admin UI |
| 2 | Create Vault identity named after the GitHub login: userpass login, entity + alias, `lab-members` group, `ltm-read` policy scoped to `secret/lab/ltm/<login>/*` (grant the **lowest level that fits**, L1 baseline) | `/creds-admin` |
| 3 | Grant SSH access to the lab box (shared key/password handover) | admin |
| 4 | Provision the member's memory DB — creates `<login>-llm` role + `<login>-memory` database + schema, prints the connection string | `bash llm-skill-ltm/deploy/provision-member.sh <login>` |
| 5 | Hand the connection string to the member (they store it themselves, step A4.2) — or write it directly: `vault kv put secret/lab/ltm/<login>/db connection_string=...` | `vault` |

> [!IMPORTANT]
> Registration **never self-grants**: `creds-register` only drafts the request; every grant goes through `/creds-admin` and is audited. Convention: the Postgres role, Vault login, and userpass name all equal the GitHub login — never deviate, it is what makes offboarding one command.

## Access Levels (what to request)

| Level | Grants | Typical member |
| --- | --- | --- |
| **L1** (default) | Project KV read — enough for LTM + shared project secrets | Everyone |
| **L2** | + dev-server SSH (short-lived certs, ≤ 8 h) | Members running on lab dev servers |
| **L3** | + prod SSH (short TTL, audit-reviewed) | Testbed maintainers |
| **admin** | Manage identities, policies, audit | Lab managers only |

## Offboarding

Leaving the lab reverses registration by the same unique ID: `/creds-admin` removes the member from all groups, deletes the login/entity, revokes tokens, and **rotates every shared secret they could read**; the Postgres role/DB is archived per [leaving-procedure.md](../leaving-procedure.md). See [vault-admin-guide.md, Section 5](https://github.com/bmw-ece-ntust/llm-skill-creds/blob/master/docs/vault-admin-guide.md).

## Troubleshooting

| Symptom | Likely cause | Fix |
| --- | --- | --- |
| `/memory` answers from markdown, not the DB | Admin steps B4/B5 pending, or SSH tunnel closed | Check with `/register-member`; re-run `ssh-copy-id`; store the secret (A4.2) |
| Daily-log hook doesn't fire on `git push` | Installer skipped or settings overwritten | Re-run `llm-core/install.sh` (idempotent) |
| Vault: permission denied | Level too low for that path | Request the specific grant via `/creds-register`; admin applies with `/creds-admin` |
| Wrong username used somewhere | A name was typed by hand | Everything must derive from `gh api user --jq .login` — re-run `/register-member` |
