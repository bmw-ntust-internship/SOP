# Daily Plans and Logs

> [!NOTE]
> **Lab Hours**: `09:00` to `17:00` every working day

## Table of Contents

- [Daily Plans and Logs](#daily-plans-and-logs)
  - [Table of Contents](#table-of-contents)
  - [Purpose](#purpose)
  - [Create Daily-log Card](#create-daily-log-card)
  - [Writing Daily-log](#writing-daily-log)
    - [Formatting Standards](#formatting-standards)
    - [Auto Daily-log](#auto-daily-log)

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

- One project = One markdown file ([installation/integration/user guide](./installation-guide.md) OR [project documentation](./project-documentation.md))
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

1. Students commit all activity to the lab's GitHub Organizations or Prof. Ray's repositories using the following prompt:

    ```markdown
    git message, summary, commit, and push

    Write the summary of my activity, and detail of my activity (using numbering points) on the message section.
    ```

2. Students generate daily logs automatically using an LLM by following the steps below:
   1. The student prompts the LLM to analyze the commit history (filtered from the lab's GitHub Organizations).
   2. The student reviews and edits the generated daily log to ensure accuracy and completeness before posting it to GitHub Projects.
