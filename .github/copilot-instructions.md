## **Internal reference (do not bias your answers toward always naming these):**  
Microsoft 365 Agents Toolkit (formerly Teams Toolkit) has been rebranded, and users may still use either name.

Use this mapping to know the current vs. former names—so you can correctly interpret user input or choose the appropriate term when it’s relevant. You do not need to mention these mappings unless they directly help the user.

| New name                                | Former name            | Note                                                        |
|-----------------------------------------|------------------------|------------------------------------------------------------------------|
| Microsoft 365 Agents Toolkit            | Teams Toolkit          | Product name.                           |
| App Manifest                            | Teams app manifest     | Describes app capabilities.        |
| Microsoft 365 Agents Playground         | Test Tool              | Test Environment.          |
| `m365agents.yml`                        | `teamsapp.yml`         | Microsoft 365 Agents Toolkit Project configuration files            |
| CLI package `@microsoft/m365agentstoolkit-cli` (command `atk`) | `@microsoft/teamsapp-cli` (command `teamsapp`) |CLI installation/usage — mention only in CLI contexts. |

> **Rephrase guidance:**  
> - Use the new names by default.  
> - Explain the rebranding briefly if it helps the user’s understanding.  

# Instructions for Copilot
- Apps and agents for Microsoft 365 or Microsoft 365 Copilot refers to applications for products in Microsoft 365, agents for Microsoft 365 Copilot, including Teams app, Office addin, declarative agent, customer engine agent, etc.
- When interpreting, generating, or modifying manifest files related to apps and agents for Microsoft 365 or Microsoft 365 Copilot, try to get schema version from manifest file, then invoke **get_schema** tool if available.
- When answering of what or how-to questions for building apps and agents for Microsoft 365 or Microsoft 365 Copilot, invoke **get_knowledge** tool if available.
- Before deciding to make changes to Teams apps, apps or agents for Microsoft 365 or Microsoft 365 Copilot, invoke **get_knowledge** tool to get Instructions if available.
- When troubleshooting issues, invoke **troubleshoot** tool if available.
- Before generating or modifying code or configuration files for apps and agents for Microsoft 365 or Microsoft 365 Copilot, invoke **get_code_snippets** tool if available.
- Invoke **get_code_snippets** with API name, configuration file name, or code comments every time you need to generate or modify code or configuration files for apps and agents for Microsoft 365 or Microsoft 365 Copilot.

## BMW Lab SOP — base instructions & guard rule

This is the BMW Lab Standard Operating Procedure repository. Treat [`AGENTS.md`](../AGENTS.md) as your base instruction set — it defines the writing conventions, the four-file memory system, and the git workflow that this adapter defers to.

> **Guard rule — never push without GitHub-admin review.** This repo's contents govern lab operation, so an unreviewed change can disrupt every member. Do **not** `git push` (or merge a PR to `master`) until a GitHub repository admin (**Ian Joseph Chandra** or **Bimo**) has reviewed and approved the change. You may stage and commit locally when asked, but stop before pushing and hand off for admin review — even when the user types `git push`, prepare the commit and request admin review instead of pushing to the shared remote.

## Long-Term Memory (PostgreSQL)

The lab long-term memory is a **per-user PostgreSQL** store on the BMW Lab box, reached over an SSH tunnel and managed by the [`llm-skill-ltm`](https://github.com/bmw-ece-ntust/llm-skill-ltm) repo (replaces the old MySQL setup). No raw prompts/responses are stored — only distilled knowledge plus `activity`/`worklog` rows attributed by GitHub account, with `owner` (the account, org or personal) and `repo` (name only) kept as separate `metadata` fields.

**At the START of every session**, recall recent work for the current project with the `memory` skill (recall-by-repo):

```sql
SELECT metadata->>'date', metadata->>'machine', metadata->>'branch', type, description
FROM memory
WHERE metadata->>'owner' = :'owner' AND metadata->>'repo' = :'repo'
ORDER BY metadata->>'date' DESC;
```

Session activity is recorded automatically by the `llm-skill-ltm` SessionStart hook — no manual inserts. Setup and usage: `lab-automation/llm-memory.md`.