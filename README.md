# n8n Release Orchestrator

A reusable **n8n workflow** that generates **release notes** between two Git tags for a GitHub repository.  
It enriches the notes using **GitHub PRs + linked GitHub Issues**, optionally enriches using **Jira issues**, and can optionally **post the result to Slack**.  
Release notes are produced by an **LLM-based agent** with a strict JSON output schema.

---

## What we have built so far

This workflow automates release note generation end-to-end:

- Compares two Git tags in GitHub
- Collects commits and changed files
- Discovers associated pull requests and enriches them with details
- Extracts Jira keys and GitHub issue references from PR title/body
- (Optional) Fetches Jira issue details and merges them back
- (Optional) Fetches GitHub issue details and merges them back
- Uses an AI Agent to produce:
  - human-readable release notes (Markdown)
  - structured release notes (JSON)
  - warnings for missing/unclear data
- (Optional) Posts final release notes to Slack
- Returns a clean result payload from the workflow

---

## Implemented tools and integrations

| Tool / Integration | Purpose | Status |
|---|---|---|
| GitHub (Compare API) | Identify changes between tags | ✅ Implemented |
| GitHub (PR enrichment) | Fetch PR metadata, normalize, filter noise | ✅ Implemented |
| GitHub (Issue enrichment) | Fetch linked issue details | ✅ Implemented |
| Jira (optional) | Fetch issue details for extracted Jira keys | ✅ Implemented |
| Slack (optional) | Post generated release notes | ✅ Implemented |
| AI Agent (LLM) | Generate structured release notes (JSON + Markdown) | ✅ Implemented |

---

## How to use this repository

### Import the workflow into n8n
- Import the workflow JSON file:  
  `workflows/ReleaseOrchestrator.json`

You can import either from file or from a raw GitHub URL (recommended for easy reuse).

---

## Post-import setup (required)

Workflow exports do **not** include secrets. After import, you must configure credentials in n8n:

| Integration | What you must configure |
|---|---|
| GitHub | A GitHub credential/token with access to the target repository |
| AI Model Provider | OpenAI (or equivalent) credential for the Chat Model used by the AI Agent |
| Jira (optional) | Jira credential + your Jira base URL |
| Slack (optional) | Slack credential + target channel/workspace settings |

**Setup checklist**
- [ ] Re-link GitHub credentials on GitHub-related nodes
- [ ] Re-link AI model credentials for the AI Agent
- [ ] If using Jira: configure Jira credentials and any project allowlist rules
- [ ] If using Slack: configure Slack credentials and confirm posting destination
- [ ] Run a test and validate the output

---

## Current limitations / changes to implement next

| Area | Current state | Needed improvement |
|---|---|---|
| Slack destination | May require manual adjustment after import | Make Slack posting fully driven by workflow input options |
| Portability | Credentials must be re-linked after import | Add a short “first-run setup” guide (already included) |
| Custom categorization | Categories depend mainly on agent output | Add label-based or repo-specific category mapping rules |

---

## Suggested next steps (roadmap)

- Make Slack destination fully dynamic (no hard-coded channel behavior)
- Add a “dry run” mode (generate notes but skip Slack)
- Improve classification using PR labels and conventional-commit patterns
- Add validation tests for the JSON output schema
- Add a “template” version for broader reuse

---

## License

Select a license in GitHub (MIT recommended if you want others to reuse easily).
