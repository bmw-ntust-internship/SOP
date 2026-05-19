# LLM Long-Term Memory — MySQL Setup Guide

This document describes how to set up a MySQL database on your VM to persist LLM prompting activity (prompts, responses, session summaries) across commits and projects.

---

## Table of Contents

- [LLM Long-Term Memory — MySQL Setup Guide](#llm-long-term-memory--mysql-setup-guide)
  - [Table of Contents](#table-of-contents)
  - [Purpose](#purpose)
  - [1. Install MySQL on the VM](#1-install-mysql-on-the-vm)
  - [2. Create the Database and Schema](#2-create-the-database-and-schema)
  - [3. Schema Reference](#3-schema-reference)
  - [4. Inserting Records](#4-inserting-records)
  - [5. Querying for Daily Logs](#5-querying-for-daily-logs)
  - [6. Recommended Tooling](#6-recommended-tooling)
  - [7. Automatic Memory Integration — MCP Setup](#7-automatic-memory-integration--mcp-setup)
    - [7.1 Install the MySQL MCP Server](#71-install-the-mysql-mcp-server)
    - [7.2 Configure Claude Code (VS Code)](#72-configure-claude-code-vs-code)
    - [7.3 Configure GitHub Copilot (VS Code)](#73-configure-github-copilot-vs-code)
    - [7.4 Add Automatic-Behavior Instructions](#74-add-automatic-behavior-instructions)
  - [8. LLM Setup Prompt](#8-llm-setup-prompt)

---

## Purpose

The MySQL long-term memory stores:

1. **Prompting activity** — each significant prompt sent to Copilot or Claude, and a brief summary of the LLM's response.
2. **Session summaries** — a one-paragraph summary of the work done in each coding session, linked to the git commit hash produced at the end of that session.

This enables automatic daily log generation by querying the database instead of reconstructing history from git alone.

---

## 1. Install MySQL on the VM

```bash
# Ubuntu / Debian
sudo apt update
sudo apt install -y mysql-server

# Start and enable the service
sudo systemctl start mysql
sudo systemctl enable mysql

# Secure the installation (set root password, remove test DB)
sudo mysql_secure_installation
```

> [!NOTE]
> If your VM runs a different OS (CentOS/Rocky), replace `apt` with `dnf` and the package name with `mysql-community-server`.

---

## 2. Create the Database and Schema

Log in as root and run the setup script:

```bash
sudo mysql -u root -p
```

Then execute:

```sql
CREATE DATABASE IF NOT EXISTS llm_memory CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER IF NOT EXISTS 'llmuser'@'localhost' IDENTIFIED BY 'your_strong_password';
GRANT ALL PRIVILEGES ON llm_memory.* TO 'llmuser'@'localhost';
FLUSH PRIVILEGES;
USE llm_memory;
```

Create the tables:

```sql
CREATE TABLE sessions (
    id               INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    github_username  VARCHAR(100)     NOT NULL COMMENT 'git config user.name of the committer',
    github_email     VARCHAR(255)     NOT NULL COMMENT 'git config user.email of the committer',
    repo             VARCHAR(255)     NOT NULL COMMENT 'GitHub repo name, e.g. bmw-ece-ntust/nonrtric-rapp-test-automation',
    start_at         DATETIME         NOT NULL COMMENT 'Timestamp of the first user prompt after the latest commit',
    end_at           DATETIME         NOT NULL COMMENT 'Timestamp of the last user prompt in this session',
    base_commit      CHAR(40)         NOT NULL COMMENT 'Commit hash that was HEAD at session start',
    result_commit    CHAR(40)         NULL     COMMENT 'Commit hash produced at the end of this session (fill in after push)',
    commit_title     VARCHAR(255)     NULL     COMMENT 'Short imperative summary title used in the git commit message',
    created_at       DATETIME         NOT NULL DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE prompts (
    id              INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    session_id      INT UNSIGNED     NOT NULL,
    tool            ENUM('copilot','claude','other') NOT NULL COMMENT 'Which LLM tool was used',
    prompted_at     DATETIME         NOT NULL COMMENT 'Timestamp of the user message',
    prompt_summary  TEXT             NOT NULL COMMENT 'Brief summary of the user prompt (1-2 sentences)',
    response_summary TEXT            NOT NULL COMMENT 'Brief summary of the LLM response (1-2 sentences)',
    FOREIGN KEY (session_id) REFERENCES sessions(id) ON DELETE CASCADE
);

CREATE TABLE session_summaries (
    id          INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    session_id  INT UNSIGNED NOT NULL UNIQUE,
    summary     TEXT         NOT NULL COMMENT 'One-paragraph summary of what was accomplished in the session',
    FOREIGN KEY (session_id) REFERENCES sessions(id) ON DELETE CASCADE
);
```

---

## 3. Schema Reference

| Table              | Key Columns                                              | Purpose                                              |
|--------------------|----------------------------------------------------------|------------------------------------------------------|
| `sessions`         | `github_username`, `github_email`, `repo`, `start_at`, `end_at`, `base_commit`, `result_commit`, `commit_title` | One row per working session |
| `prompts`          | `session_id`, `tool`, `prompted_at`, `prompt_summary`, `response_summary` | One row per key prompt–response exchange |
| `session_summaries`| `session_id`, `summary`                                  | One-paragraph summary written at commit time; used as the "Summary" field in the git commit message |

---

## 4. Inserting Records

At the end of each session (step 5 of the auto daily-log prompt), insert records via the `mysql` CLI or a helper script.

**Step 1 — capture committer identity from git:**

```bash
GIT_USER=$(git config user.name)
GIT_EMAIL=$(git config user.email)
REPO=$(git remote get-url origin | sed 's|.*github.com[:/]\(.*\)\.git|\1|;s|.*github.com[:/]||')
BASE_COMMIT=$(git log -1 --format="%H")
```

**Step 2 — open session record:**

```sql
INSERT INTO sessions (github_username, github_email, repo, start_at, end_at, base_commit, commit_title)
VALUES (
    'Ian Joseph Chandra',
    'ian@example.com',
    'bmw-ece-ntust/your-repo',
    '2026-05-08 09:15:00',
    '2026-05-08 11:42:00',
    '60a45a1ce65dac36229945d9a8a3b58ef015accc',
    'Add GitHub identity and repo fields to LLM memory schema'
);
-- Note the new session id for use in subsequent inserts
SET @sid = LAST_INSERT_ID();
```

**Step 3 — log prompts:**

```sql
INSERT INTO prompts (session_id, tool, prompted_at, prompt_summary, response_summary)
VALUES
    (@sid, 'copilot', '2026-05-08 09:15:00',
     'Asked to add GitHub identity and repo fields to the LLM memory MySQL schema',
     'Updated llm-memory.md sessions table with github_username, github_email, commit_title columns'),
    (@sid, 'claude',  '2026-05-08 10:30:00',
     'Asked to update the auto daily-log prompt to capture committer identity before DB insert',
     'Updated daily-log.md step 4 to run git config and include identity in sessions INSERT');
```

**Step 4 — session summary (reused as git commit "Summary"):**

```sql
INSERT INTO session_summaries (session_id, summary)
VALUES (@sid, 'Extended the LLM long-term memory schema to capture the committer GitHub username and email, the working repository, and the git commit title per session, and updated the auto daily-log workflow to inject this identity context before inserting records.');
```

**Step 5 — update result commit after push:**

```sql
UPDATE sessions SET result_commit = 'abc1234...' WHERE id = @sid;
```

---

## 5. Querying for Daily Logs

Generate a daily-log entry by querying all sessions and prompts for a given date:

```sql
SELECT
    s.repo,
    DATE_FORMAT(s.start_at, '%Y/%m/%d') AS date,
    DATE_FORMAT(s.start_at, '%H:%i')    AS start_time,
    DATE_FORMAT(s.end_at,   '%H:%i')    AS end_time,
    ss.summary,
    p.tool,
    DATE_FORMAT(p.prompted_at, '%H:%i') AS prompt_time,
    p.prompt_summary,
    p.response_summary
FROM sessions s
JOIN session_summaries ss ON ss.session_id = s.id
JOIN prompts p             ON p.session_id  = s.id
WHERE DATE(s.start_at) = '2026-05-08'
ORDER BY p.prompted_at;
```

Feed the output to an LLM to format it into a GitHub Projects daily-log card entry.

---

## 6. Recommended Tooling

| Tool | Purpose |
|------|---------|
| [TablePlus](https://tableplus.com/) | GUI client for macOS — connect directly to the VM MySQL |
| [DBeaver](https://dbeaver.io/) | Cross-platform alternative GUI |
| `mysql` CLI | Quick inserts/queries from terminal |
| Python `mysql-connector-python` | Automate inserts via script if needed |

Connect TablePlus/DBeaver directly to `140.118.122.119:3306` with the `llmuser` credentials.

> [!NOTE]
> Install the MySQL CLI client on macOS with `brew install mysql-client`, then add it to your PATH:
> `echo 'export PATH="/usr/local/opt/mysql-client/bin:$PATH"' >> ~/.zshrc`

---

## 7. Automatic Memory Integration — MCP Setup

Both GitHub Copilot and Claude Code can read from and write to the `llm_memory` database using the **Model Context Protocol (MCP)**. Once configured, the LLM issues SQL queries as tool calls — no manual copy-paste required.

**Architecture:**

```
VS Code (Copilot / Claude Code)
        │
        │  MCP tool calls (SQL)
        ▼
MySQL MCP Server  ←  runs locally on your laptop
        │
        │  TCP direct  (140.118.122.119:3306)
        ▼
   MySQL on VM (bmw-ntust)
```

### 7.1 Install the MySQL MCP Server

```bash
npm install -g @benborla29/mcp-server-mysql
```

> [!TIP]
> You can skip the global install and use `npx -y @benborla29/mcp-server-mysql` in the config below — npx downloads and caches the package on first run.

---

### 7.2 Configure Claude Code (VS Code)

Claude Code reads MCP server definitions from `~/.claude.json` (global, applies to all projects).

Add the `mysql-memory` server entry:

```json
{
  "mcpServers": {
    "mysql-memory": {
      "command": "npx",
      "args": ["-y", "@benborla29/mcp-server-mysql"],
      "env": {
        "MYSQL_HOST": "140.118.122.119",
        "MYSQL_PORT": "3306",
        "MYSQL_USER": "llmuser",
        "MYSQL_PASS": "your_strong_password",
        "MYSQL_DB": "llm_memory"
      }
    }
  }
}
```

Restart Claude Code in VS Code after saving. The MCP server should appear as **Connected** in the Claude Code panel.

> [!IMPORTANT]
> `~/.claude.json` is in your home directory and is not committed to git. Never paste these credentials into a file tracked by git.

---

### 7.3 Configure GitHub Copilot (VS Code)

MCP servers for Copilot in VS Code are configured either at user level (`settings.json`) or per workspace (`.vscode/mcp.json`).

**Option A — User level** (applies to all workspaces):

Open VS Code `settings.json` (`Cmd+Shift+P` → *Preferences: Open User Settings (JSON)*) and add:

```json
{
  "mcp": {
    "servers": {
      "mysql-memory": {
        "command": "npx",
        "args": ["-y", "@benborla29/mcp-server-mysql"],
        "env": {
          "MYSQL_HOST": "140.118.122.119",
          "MYSQL_PORT": "3306",
          "MYSQL_USER": "llmuser",
          "MYSQL_PASS": "your_strong_password",
          "MYSQL_DB": "llm_memory"
        }
      }
    }
  }
}
```

**Option B — Workspace level** (per-repo, add `.vscode/mcp.json`):

```json
{
  "servers": {
    "mysql-memory": {
      "command": "npx",
      "args": ["-y", "@benborla29/mcp-server-mysql"],
      "env": {
        "MYSQL_HOST": "140.118.122.119",
        "MYSQL_PORT": "3306",
        "MYSQL_USER": "llmuser",
        "MYSQL_PASS": "your_strong_password",
        "MYSQL_DB": "llm_memory"
      }
    }
  }
}
```

> [!CAUTION]
> If you use Option B, add `.vscode/mcp.json` to `.gitignore` so credentials are never committed.

After saving, open the Copilot Chat panel. The `mysql-memory` tool will appear in the MCP tools list.

---

### 7.4 Add Automatic-Behavior Instructions

The MCP connection alone does not make the LLM act automatically — you must instruct it via context files.

**For Claude Code** — add to `~/.claude/settings.json` (global) as shown in Section 7.3, and add to `CLAUDE.md` (project root):

````markdown
## Long-Term Memory (MySQL)

A `mysql-memory` MCP server is configured and connects directly to `llm_memory` on the BMW Lab VM at `140.118.122.119:3306`.

**At the START of every session**, load recent context:

```sql
SELECT s.github_username, s.repo, s.start_at, s.end_at, s.commit_title, ss.summary
FROM sessions s
JOIN session_summaries ss ON ss.session_id = s.id
ORDER BY s.start_at DESC
LIMIT 5;
```

**At the END of every session** (before the git commit), insert the session record, prompts,
and summary into the database following Section 4 of `lab-automation/llm-memory.md`.
Then update `result_commit` after pushing.
````

**For GitHub Copilot** — add the same block to `.github/copilot-instructions.md`.

> [!NOTE]
> If VS Code reports the MCP server as unavailable, verify direct connectivity with `mysql -h 140.118.122.119 -P 3306 -u llmuser -pbmwlab llm_memory -e "SELECT 1;"` before starting a session.

---

## 8. LLM Setup Prompt

Use the prompt below to have an LLM (GitHub Copilot or Claude Code) set up the entire long-term memory ecosystem automatically in a new environment. Paste it at the start of a fresh session.

```
I want to set up a MySQL long-term memory system for LLM sessions on this machine.
The database will persist prompting activity, session summaries, and git commit metadata
across all projects. Follow the guide at `lab-automation/llm-memory.md` and complete
all steps below:

**1. VM — MySQL installation and schema**
SSH into wifi@140.118.122.119 and:
  a. Install MySQL Server if not already installed (Ubuntu 22.04: `sudo apt install -y mysql-server`).
  b. Create the `llm_memory` database, `llmuser` account, and all three tables
     (`sessions`, `prompts`, `session_summaries`) using the exact schema in Section 2.
  c. Verify with `SHOW TABLES;` and `DESCRIBE sessions;`.

**2. Mac — MySQL client**
  a. Install `mysql-client` via Homebrew if not present.
  b. Add `/usr/local/opt/mysql-client/bin` to `~/.zshrc` if not already there.
  c. Test direct connectivity: `mysql -h 140.118.122.119 -P 3306 -u llmuser -p llm_memory -e "SHOW TABLES;"`.

**3. Mac — MCP server**
  a. Confirm `npx -y @benborla29/mcp-server-mysql` runs without error.
  b. Write the `mysql-memory` server block into `~/.claude.json` (for Claude Code)
     using the config in Section 7.2 (host 140.118.122.119, port 3306, db llm_memory).
  c. Add the same `mcp.servers.mysql-memory` block to VS Code user `settings.json`
     using the config in Section 7.3.

**4. Instruction files**
  a. Add the Long-Term Memory section from Section 7.4 to `CLAUDE.md` in this repo.
  b. Add the same section to `.github/copilot-instructions.md`.

After each step, confirm it succeeded before moving to the next.
Ask me for the sudo password if needed — run interactive sudo commands directly
in the terminal so I can type the password myself.
```
