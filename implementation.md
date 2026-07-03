<h1 align="center">Implementation Guide — Guideline</h1>
<hr>

> [!CAUTION]
> Keep this document **private** by default. Make it public only after the project's paper is published. Request access from the group's GitHub admin.

---

> [!NOTE]
> **`implementation.md`** (this file) covers per-component **installation** (setup, configuration, dependencies) and **end-to-end integration** (connecting components, verifying interfaces) — the single document for bringing the full system, the testbed, up from a clean state. The project `README.md` links here from its Setup section; `research.md`'s System Architecture links here per component. Full documentation map: [README Section 3](./README.md#3-documentation-requirements); hierarchy diagram: [research.md](./research.md#purpose).

## Table of Contents

> [!TIP]
> Generate the Table of Contents automatically with the [Markdown All in One extension for VS Code](https://marketplace.visualstudio.com/items?itemName=yzhang.markdown-all-in-one#table-of-contents).

- [Table of Contents](#table-of-contents)
- [How to Write This Guide](#how-to-write-this-guide)
  - [1. One command, one block — with concise output](#1-one-command-one-block--with-concise-output)
  - [2. Record the working directory — every `cd`](#2-record-the-working-directory--every-cd)
  - [3. Only the required steps, in the right order](#3-only-the-required-steps-in-the-right-order)
  - [4. Generate from the captured terminal log](#4-generate-from-the-captured-terminal-log)
  - [5. One-touch deployment (LLM over SSH)](#5-one-touch-deployment-llm-over-ssh)
- [Project Root](#project-root)
- [Project Description](#project-description)
- [System Architecture](#system-architecture)
- [Execution Status](#execution-status)
- [Access Method](#access-method)
- [Repository Structure](#repository-structure)
  - [Configuration](#configuration)
  - [Installation Steps](#installation-steps)
- [End-to-End Walkthrough](#end-to-end-walkthrough)
- [Post-Installation Verification](#post-installation-verification)
- [Known Issues](#known-issues)
- [Troubleshooting](#troubleshooting)
- [Additional Resources](#additional-resources)

## How to Write This Guide

Five rules produce a guide a successor — human or LLM — can follow top-to-bottom to a working system.

### 1. One command, one block — with concise output

Put the command in a fenced block tagged with its shell, and comment the terminal output in the **same** block with `#`. Keep only the lines that prove the step succeeded or failed; skip the rest with `...`. Redirect long or noisy logs to a `./logs/` folder and link them.

```shell
git clone https://github.com/<org>/<repo>.git && cd <repo>

# Cloning into '<repo>'...
# ...
# Resolving deltas: 100% (42/42), done.
```

### 2. Record the working directory — every `cd`

> [!IMPORTANT]
> Most failed reproductions come from running a command in the **wrong directory**. The working directory is state that a code block does not show, so make it explicit — never rely on an undocumented "current" directory.

- **Anchor at the project root.** The root is the directory you land in after you SSH into the server and `git clone` the repo (the `git clone` PATH). Declare it once at the top of the guide — see the [Project Root](#project-root) note — and treat every path as relative to it.
- **Put a `# run from:` comment on top of each command block**, naming the directory the command runs in — e.g. `# run from: ~/<repo>` (the root) or `# run from: ~/<repo>/src/component-a`. It is the first thing a reader checks before running the block.
- **Show every `cd` as its own documented step**, with a short note on *why* — which component or step runs there. Do not let a directory change happen silently between blocks.
- **Prefer absolute paths** (or paths relative to the stated root) for config files, scripts, and outputs, so a step does not depend on where the previous one left the shell.

```shell
# run from: ~/oran-sc/ric-plt   (Near-RT RIC platform)
./install.sh
# ...
# [INFO] RIC platform installed
```

### 3. Only the required steps, in the right order

Document the **shortest successful path**: the commands that actually worked, on the critical path, runnable top-to-bottom from a clean state to a working system. Drop exploratory, repeated, and dead-end commands from the main flow. Put failures, wrong turns, and their fixes in [Known Issues](#known-issues) / [Troubleshooting](#troubleshooting) at the bottom, each as *symptom → cause → fix*.

### 4. Generate from the captured terminal log

Terminal activity is captured automatically by the lab terminal-logging system (`termlog` schema, managed by `bmw-ece-ntust/llm-skill-logging`; see [daily-log.md](./daily-log.md) and [lab-automation/llm-memory.md](./lab-automation/llm-memory.md)). Generate the guide **from that stored data**, not from memory — run the `/integration-guide` skill, which pulls the session's commands ordered on the shared UTC timestamp and renders each as a `shell` block with `#`-commented output per the rules above.

### 5. One-touch deployment (LLM over SSH)

> [!IMPORTANT]
> Structure the guide so an LLM can take a clean server to a working system over SSH: the human fills in the Prerequisites, the LLM runs one idempotent script, and it pauses only for the human-only steps you flag.

**Prerequisites — the human fills these in first.** List everything the LLM cannot invent. Keep secret *values* out of Git; list only their names and where they live.

| Input | What it is | Example |
| --- | --- | --- |
| SSH access | account with `sudo`, reachable over VPN | `user@<ip>` |
| Config values | ports, FQDN/IP, paths | `PORT=8200` |
| Secret pointers | secret name + location (Vault path, Keychain item) — never the value | `secret/lab/<proj>/db` |
| Certs / keys | TLS or SSH material the human provisions | `/etc/<svc>/tls/*` |
| Human-custody items | unseal keys, root tokens, OIDC secrets stored offline | held by key holders |

**One-touch script.** Provide a single idempotent script (for example `deploy/<component>-deploy.sh`) that performs the whole server-side bring-up, plus the one command that runs it:

```bash
scp -r deploy/<component>-deploy.sh config "$SSH_USER@$HOST:/tmp/dep/"
ssh "$SSH_USER@$HOST" "sudo VAR1=... VAR2=... bash /tmp/dep/<component>-deploy.sh"

# == step 1: install packages ... ==
# ...
# == done ==
```

- **Idempotent** — safe to re-run; each step checks state before acting (installed? enabled? configured?), so reruns converge instead of duplicating.
- **Reads inputs from the environment**, never hardcoded; **fails closed** with a clear message naming the missing input.
- **Human-only steps stay manual** — the script prints what the human must do (store unseal keys offline, approve access) and stops or continues safely.
- **Verifiable output** — each stage prints a success/failure marker so the LLM can confirm before proceeding.
- Keep a **Manual reference** (the same steps one at a time) below the one-touch path so the deployment can be audited or run by hand.

Worked example: `bmw-ece-ntust/llm-skill-creds/installation-guide.md` (Prerequisites → `deploy/deploy-vault.sh` → Manual reference).

## Project Root

> [!IMPORTANT]
> **Declare the project root path here, before any step.** This is the directory you land in after you SSH into the server and `git clone` the repo — the anchor for every `# run from:` comment and relative path in this guide. If a reader is not in this directory, the steps will not reproduce.

| Item | Value |
| --- | --- |
| SSH entry | `ssh <user>@<server-ip>` (see [Access Method](#access-method)) |
| Clone command | `git clone https://github.com/<org>/<repo>.git` |
| **Project root (PATH)** | **`~/<repo>`** — every step runs here unless its `# run from:` comment says otherwise |

## Project Description

**Project Name:** [Replace with actual project name]

**Description:** A solution for [specific use case] that provides [key functionality] and enables users to [main benefit].

**Key Features:**

- Feature 1: [Brief description]
- Feature 2: [Brief description]
- Feature 3: [Brief description]

**Target Users:** [Developers / Researchers / System Administrators / etc.]

## System Architecture

> [!NOTE]
> **draw.io files:** store raw `.drawio` files in `./docs/drawio/`, export PNG/SVG for embedding, version the raw files, and name them `<project-name>.drawio`.

Include in the architecture diagram:

1. **IP addresses** — per module/component
2. **Connection types & protocols** — WiFi, RJ-45, HTTP, TCP, UDP, WebSocket, etc.
3. **Sub-module structure** — internal components and their relationships
4. **Data-flow direction** — request/response patterns
5. **Port numbers** — communication ports
6. **Network boundaries** — segments (DMZ, internal, external)

## Execution Status

> [!NOTE]
> The project keeps **one** Execution Status table (✅⏳❌ per step, with dates), and it lives in [`research.md`](./research.md#execution-status) — do not keep a second copy here. Link each of its installation/integration steps to the matching section anchor in this guide (e.g. `[Install Component A](implementation.md#installation-steps)`).

## Access Method

> [!NOTE]
> Servers live in the server room. Contact the admin for VPN access.

```shell
Host: <IP address>
User: <username>
```

```shell
ssh user@<IP address>

# The authenticity of host '192.168.1.100' can't be established.
# Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
# ...
# Welcome to Ubuntu 22.04.3 LTS (GNU/Linux 5.15.0-87-generic x86_64)
# user@hostname:~$
```

## Repository Structure

The folder structure is defined **once**, in [programming.md Section 5](./programming.md#5-folder-structure) — do not restate it here. This guide relies on three locations:

- `config/.env.example` — every environment variable, names only (see [Configuration](#configuration))
- `deploy/` — the one-touch deployment script(s) (see [rule 4](#4-one-touch-deployment-llm-over-ssh))
- `./logs/` — redirected long or noisy command outputs (see [rule 1](#1-one-command-one-block--with-concise-output))

### Configuration

Create a `.env` file in the root directory (see `config/.env.example` for all variables):

```bash
# Database
DB_HOST=localhost
DB_PORT=5432
DB_NAME=your_database_name
DB_USER=your_username
DB_PASSWORD=your_password

# Application
APP_PORT=3000
APP_DEBUG=false
```

### Installation Steps

Run these from a clean checkout, top-to-bottom, to reach a running system.

1. **Clone the repository:**

    ```sh
    # run from: ~   (your SSH home; this creates the project root ~/<repo>)
    git clone https://github.com/<org>/<repo>.git && cd <repo>

    # Cloning into '<repo>'...
    # ...
    # Resolving deltas: 100% (42/42), done.
    ```

2. **Install dependencies:**

    ```sh
    # run from: ~/<repo>   (project root)
    pip install -r requirements.txt

    # Collecting package-name==1.2.3
    # ...
    # Successfully installed package-name-1.2.3 dependency-1-2.0.1 dependency-2-3.1.0
    ```

3. **Set up environment variables:** create `.env` in the root directory (see [Configuration](#configuration)).

4. **Run the application:**

    ```sh
    # run from: ~/<repo>   (project root)
    python3 app.py

    # * Serving Flask app 'app'
    # ...
    # * Running on http://127.0.0.1:3000
    ```

## End-to-End Walkthrough

> [!IMPORTANT]
> Every guide must include one end-to-end walkthrough that chains all components in the correct startup order. A successor must be able to follow this section alone to bring the full system from a clean state to a working state.

Required structure:

1. **Startup order** — which service must be running before the next one starts
2. **Each step** — command(s) + expected output confirming the component is ready
3. **Inter-component verification** — confirm each interface before proceeding (e.g. SCTP link up before testing E2)
4. **Estimated time** — per step, so successors can plan
5. **Single E2E test** — one command or API call that exercises the full chain

**Example template:**

```markdown
## End-to-End Walkthrough

Starting from a clean deployment, bring up the system in this order.
Total estimated time: ~45 minutes.

### 1. Start Component A (5 min)
```bash
# run from: ~/<repo>   (project root)
cd src/component-a && python3 app.py --config config/.env
# [INFO] Component A listening on port 8080
```

### 2. Start Component B — depends on A (10 min)
```bash
curl http://localhost:8080/health   # verify A is up first
cd src/component-b && ./start.sh
# [INFO] Connected to Component A at localhost:8080
# [INFO] Component B ready
```

### 3. Run E2E test — verifies the full chain
```bash
curl -X POST http://localhost:8090/api/run-test \
  -H "Content-Type: application/json" \
  -d '{"scenario": "basic"}'
# {"status": "PASSED", "duration_ms": 1234}
```
```

## Post-Installation Verification

Follow these steps to verify your installation was successful:

1. **Check Application Status:**

   ```bash
   # Check if the application is running
   ps aux | grep app.py

   # user     12345  0.5  2.1 345678 123456 ?      Ssl  10:30   0:15 python3 app.py
   ```

2. **Test Basic Functionality:**

   ```bash
   # Test API endpoint (if applicable)
   curl http://localhost:3000/health

   # HTTP/1.1 200 OK
   # ...
   # {"status": "OK", "timestamp": "2024-10-21T10:45:23.456Z", "uptime": 900}
   ```

3. **Verify Database Connection:**

   ```bash
   # Run database connectivity test
   python3 -c "from src.main import test_db_connection; test_db_connection()"

   # Connecting to database at localhost:5432...
   # Database connection successful!
   # Server version: PostgreSQL 14.9
   ```

4. **Verify the O-RAN interfaces and PM pipeline (O1 / E2 / R1 / InfluxDB / Grafana):**

   Follow [oran-verification.md](./oran-verification.md) — the lab-common conformance checks, also required for the **D-180 code & project review** before the oral exam (leaving-procedure item 4.1.1).

## Known Issues

> [!IMPORTANT]
> This section is **required**. Document every known failure mode, expired credential, hardware quirk, or unresolved integration issue. A successor who hits an undocumented failure will waste days debugging what took you hours the first time.

| Issue | Severity | Status | Workaround / Steps to Resolve |
|-------|----------|--------|-------------------------------|
| [Short description of the failure] | ❌ BLOCK / ⚠️ WARN / ℹ️ INFO | Open / Resolved / Workaround | Step-by-step to unblock |

**Severity guide:**

- `❌ BLOCK` — prevents the system from running; must be fixed or documented as "build from source" before handover
- `⚠️ WARN` — system runs but a component is degraded; must have a documented workaround
- `ℹ️ INFO` — cosmetic or edge-case issue; note for awareness

**What to document:** container image pull failures (expired tokens, private registry access); empty git submodules after a plain clone; API endpoints or config parameters that changed from the thesis diagrams; hardware-specific timing/driver quirks; known-flaky integrations (with retry instructions); experimental configs that always fail (mark out of scope); dataset anomalies.

**Example:**

| Issue | Severity | Status | Workaround |
|-------|----------|--------|------------|
| NFO image `registry.example.com/nfo:latest` returns 401 | ❌ BLOCK | Token expired | Run `docker login registry.example.com` with credentials from the lab secrets vault; token rotates every 90 days |
| `src/sideloader/` empty after plain `git clone` | ❌ BLOCK | Submodule | Run `git submodule update --init --recursive` or re-clone with `--recursive` |
| TEIV adapter logs "JSON is empty" | ⚠️ WARN | Open | Restart FOCOM pod first (`kubectl rollout restart deploy/focom`), then restart TEIV |
| Pegatron/OKD/8-core/700Mbps always FAILED | ℹ️ INFO | Hardware limit | Known incompatibility; exclude this config from benchmarks |

## Troubleshooting

1. **Port already in use** (`Address already in use: 3000`)

   ```bash
   sudo lsof -i :3000          # find the process
   # python3 12345  user  3u  IPv4 ...  TCP *:3000 (LISTEN)
   kill -9 12345               # free the port
   ```

2. **Python dependency not found** (`ModuleNotFoundError: No module named 'module_name'`)

   ```bash
   pip install -r requirements.txt
   # ...
   # Successfully installed module_name-2.1.0
   ```

3. **Permission denied** (`Permission denied: '/path/to/file'`)

   ```bash
   chmod 755 /path/to/file     # fix permissions (no output on success)
   # or run with the appropriate user/privileges
   ```

## Additional Resources

- **Documentation:** [Official docs](https://your-project-docs.com) · [API reference](https://your-project-api.com) · [Configuration reference](https://your-project-config.com)
- **Support:** [GitHub Issues](https://github.com/<org>/<repo>/issues)
- **Maintainer:** Your Name (your.email@example.com)

---

> [!NOTE]
> This guide is updated regularly. For the latest version, check the [GitHub repository](https://github.com/<org>/<repo>).
