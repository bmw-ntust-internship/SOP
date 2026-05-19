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
    - [Auto Daily-log](#auto-daily-log)
      - [Generating the Daily-Log Entry](#generating-the-daily-log-entry)

## Purpose

This document describes how students track daily progress and ensure systematic completion of project milestones through two essential documents:

1. **Milestones** (`milestones.md`): Project roadmap with all tasks, deadlines, and completion status
2. **Daily Logs** (`daily-logs.md`): Time-stamped record of work performed, linking tasks to study notes


## Create Daily-log Card

1. Get access to our GitHub Projects ([lab-members](https://github.com/orgs/bmw-ece-ntust/projects/8) / [interns](https://github.com/orgs/bmw-ntust-internship/projects/4)) (contact our GitHub admin).
2. Create your card based on the daily-log card template.
3. Write the information below as the **GitHub Project card description** in the first comment of the card. The card description serves as the student's permanent profile and milestone reference on the project board.

   **Example:**

    ```markdown
    **Profile Information**
    - Name: <Full Name>
    - Student ID: <Student ID>
    - Email: <ntust.edu.tw email>
    - Supervisor: Prof. Ray

    **Projects**
    1. Thesis: [<Thesis Title>](<GitHub repo link>)
    2. Industrial Project: [<Project Name>](<GitHub repo link>)
    3. Logistic Jobs: <e.g., Meeting administrator>

    **Milestones**
    - [ ] [<Milestone 1 Title>](<GitHub link>)
    - [ ] [<Milestone 2 Title>](<GitHub link>)
    ```

## Writing Daily-log

### Formatting Standards

```markdown
### yyyy/mm/dd

**Short-term Goal**

- [x] [TA rApp refactoring](Documentation link with 7-digit commit hash)

**Daily-logs**:

- `hh:mm - hh:mm`: [Task 1 Title](Documentation header link with 7-digit commit hash)
- `hh:mm - hh:mm`: [Task 2 Title](Documentation header link with 7-digit commit hash)
- <Upcoming targets>
```

Example:

```markdown
### 2026/04/22

**Short-term Goal**

- [x] [TA rApp refactoring](https://github.com/bmw-ece-ntust/nonrtric-rapp-test-automation/tree/7bd313d#test-automation-rapp)

**Daily-logs**:

- `09:00–10:30`: [Rebuild and Deploy TA rApp](https://github.com/bmw-ece-ntust/nonrtric-rapp-test-automation/tree/7bd313d#-rebuild--deploy-ta-rapp)
- `10:30–12:00`: [Upload Test Spec](https://github.com/bmw-ece-ntust/nonrtric-rapp-test-automation/tree/7bd313d#-upload-test-specification-to-ta-rapp---test-spec-info)
- <Upcoming targets>
```

> [!TIP]
> Link study notes with milestones and daily logs as shown below:

![Daily log workflow diagram](./assets/daily-log.png)

**Key Principles**:

- One project = One markdown file ([integration/user guide](./integration-guide.md) OR [project documentation](./project-documentation.md))
- Hourly task = One section (markdown header) in the study note
  - Task title = section header (## in the documentation)
- Students log each task immediately after completion.
- Students include accurate start and end times for each task.
- Students specify which project to work on as their daily short-term goal.

> [!WARNING]
>
> Each finished task must be linked to a documentation header link (**with 7-digit commit hash**) in the markdown file.
>
> The 7-digit commit hash ensures **the link is always available**, even if the document is renamed/moved.

**How to get the commit hash URL:**

1. Open the documentation in GitHub from the commit history (the 7-digit hash appears in the browser URL).
2. Click the designated section header link.
3. Copy the URL. Use either the full URL or shorten it to keep only the 7-digit commit hash.

### Auto Daily-log

Students commit all activity to the lab's GitHub Organizations or Prof. Ray's repositories using one of the two prompts below. Use **Prompt A** when the BMW Lab VM is reachable; use **Prompt B** when offline or the tunnel is unavailable.

<!-- ---

#### Prompt A — With Long-Term Memory (MySQL)

> [!IMPORTANT]
> Requires the SSH tunnel to be active (`localhost:3307 → wifi@140.118.122.119:3306`) and the `mysql-memory` MCP server connected in Claude Code. See [LLM Long-Term Memory Setup](./lab-automation/llm-memory.md).

```markdown
Reconcile CLAUDE.md with the current state of the repository. Check: file list (add new files, update renamed/deleted entries), terminology, conventions, and external links. CLAUDE.md is a static knowledge snapshot for LLMs — do not log session activity or prompting history.

Then:

0. Check MySQL connectivity: run `nc -z 140.118.122.119 3306 && echo OK || echo UNREACHABLE`.
   - If **OK** → proceed with steps 1–5.
   - If **UNREACHABLE** → stop and warn the user:
     > ⚠️ Cannot reach the BMW Lab VM MySQL database (140.118.122.119:3306).
     > The VM may be down or the network is unavailable. Please either:
     > - Verify the VM is reachable and retry; or
     > - Switch to Prompt B (no long-term memory) and confirm to proceed with commit only.

1. Log this session to the long-term memory MySQL database on the VM (see [LLM Long-Term Memory Setup](./lab-automation/llm-memory.md)):
   a. Run `git config user.name` and `git config user.email` to capture the committer identity.
   b. Insert a row into `sessions` with: GitHub username, email, repo name (from `git remote get-url origin`), session start/end timestamps, latest commit hash, and the short commit title you plan to use.
   c. Insert one row per key prompt–response exchange into `prompts` (tool name, timestamp, prompt summary, response summary).
   d. Insert a row into `session_summaries` with a one-paragraph summary of the session's work — this becomes the **"Summary"** field in the git commit message below.
   e. After pushing, run `UPDATE sessions SET result_commit = '<new hash>' WHERE id = <sid>;`.
2. Run `git log -1 --format="%H %ai"` to get the latest commit hash and its timestamp.
3. Collect all prompting activity from this session across both tools:
   - **GitHub Copilot**: Read the session log at `{{VSCODE_TARGET_SESSION_LOG}}`.
   - **Claude** (claude.ai / Claude Desktop): Check the Claude conversation history for the same session manually.
   - Use the **first user message timestamp across either tool** that falls after the latest commit (= session start).
   - Use the **last user message timestamp across either tool** (= session end).
4. Format the working session(s):
   - Single calendar day → one line: `yyyy/mm/dd: hh.mm - hh.mm`
   - Multiple calendar days → one line per day: `yyyy/mm/dd: hh.mm - hh.mm`
5. Commit and push all changes using the git message format below, deriving fields from the MySQL records:
   - **Title**: the `commit_title` stored in `sessions`
   - **Session**: formatted from `sessions.start_at` / `sessions.end_at`
   - **Summary**: copied from `session_summaries.summary`

---

<Short imperative summary title>

Session:
yyyy/mm/dd: hh.mm - hh.mm

Summary:
<One-paragraph summary of what changed and why>

Details:
1. <Specific change 1>
2. <Specific change 2>
```

---

#### Prompt B — Without Long-Term Memory -->

> [!NOTE]
> Use this prompt when offline, the SSH tunnel is unavailable, or the `mysql-memory` MCP server is not connected. No database insert is performed — session history is derived from git log and the conversation history only.

```markdown
Reconcile CLAUDE.md with the current state of the repository. Check: file list (add new files, update renamed/deleted entries), terminology, conventions, and external links. CLAUDE.md is a static knowledge snapshot for LLMs — do not log session activity or prompting history.

Then:

1. Run `git log -1 --format="%H %ai"` to get the latest commit hash and its timestamp.
2. Collect session timestamps from this conversation:
   - **GitHub Copilot**: Read the session log at `{{VSCODE_TARGET_SESSION_LOG}}`.
   - **Claude** (claude.ai / Claude Desktop): Check the Claude conversation history for the same session manually.
   - Use the **first user message timestamp across either tool** that falls after the latest commit (= session start).
   - Use the **last user message timestamp across either tool** (= session end).
3. Format the working session(s):
   - Single calendar day → one line: `yyyy/mm/dd: hh.mm - hh.mm`
   - Multiple calendar days → one line per day: `yyyy/mm/dd: hh.mm - hh.mm`
4. Write a one-paragraph summary of what was accomplished this session — this becomes the **"Summary"** field below.
5. Remove any LLMs from the co-author list.
6. Commit and push all changes using the git message format below:

---

<Short imperative summary title>

Work Start: <hh:mm>

Summary:
<One-paragraph summary of what changed and why>

Details:
1. <Specific change 1>
2. <Specific change 2>
```

---

#### Generating the Daily-Log Entry

After committing (with either prompt), generate the daily-log card entry by prompting the LLM to analyze the commit history filtered from the lab's GitHub Organizations. Review and edit the generated entry for accuracy before posting it to GitHub Projects.
