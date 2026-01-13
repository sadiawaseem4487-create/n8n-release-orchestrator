# n8n Release Orchestrator

A reusable **n8n workflow** that generates **structured release notes** between two Git tags for a GitHub repository.  
It enriches results using **GitHub PRs + linked GitHub Issues**, optionally enriches using **Jira issues**, and can optionally **post to Slack**.  
Release notes are produced by an **LLM-based agent** with a strict JSON output schema.

---

## Repository contents

| Path | Description |
|---|---|
| `workflows/ReleaseOrchestrator.json` | Exported n8n workflow (importable artifact) |
| `examples/request.minimal.json` | Minimal webhook payload |
| `examples/request.full.json` | Full webhook payload (Slack + Jira options) |

---

## What we have achieved so far

This workflow automates release note generation end-to-end:

1. Receives a **Webhook POST** request with `repo`, `fromTag`, `toTag`, and optional configuration.
2. Calls **GitHub Compare API** to compute differences between tags.
3. Extracts/normalizes:
   - commits
   - changed files
   - PR numbers (associated with commits)
   - Jira keys and GitHub issue references found in PR titles/bodies
4. Fetches **PR details** and filters out noise (bots/release/changelog PRs).
5. (Optional) Fetches **Jira issue details** and attaches them to related PRs.
6. (Optional) Fetches **GitHub issue details** and attaches them to related PRs.
7. Uses an **AI Agent** to generate:
   - `releaseNotesMarkdown` (human-friendly)
   - `releaseNotesJson` (structured categories + stats)
   - `warnings` (missing data / low-confidence areas)
8. Returns everything as a clean **Webhook response payload**.
9. (Optional) Posts final release notes to **Slack**.

---

## Implemented components (workflow capabilities)

| Capability | What it does | Implementation notes |
|---|---|---|
| Webhook trigger + request validation | Validates required inputs and normalizes option keys | Prevents missing/invalid requests early |
| GitHub tag-range comparison | Compares `fromTag` → `toTag` | Uses GitHub REST compare endpoint |
| Commit/file parsing | Normalizes commit list and file changes | Produces consistent internal data |
| PR association | Finds PRs connected to commits | Improves reliability vs string parsing alone |
| PR details enrichment | Fetches title/body/author/labels/etc. | Enables richer release notes |
| PR filtering | Removes bot PRs and “release/changelog” PRs | Reduces noise in notes |
| Ticket extraction | Extracts Jira keys + GitHub issue refs from PR text | Supports mixed conventions |
| Jira enrichment (optional) | Fetches Jira issues and maps them back | Includes allow-list for project keys |
| GitHub issue enrichment (optional) | Fetches issue details and maps them back | Adds context from issues |
| AI release notes generation | Produces Markdown + strict JSON + warnings | Uses strict schema to reduce hallucination |
| Slack posting (optional) | Sends final notes to Slack | Can be toggled via options |
| Webhook response output | Returns clean JSON response | Designed for automation/CI usage |

---

## Integrations and credentials (required after import)

> Workflow exports do **not** include secrets. After importing, each user must configure credentials in n8n.

| Integration | Required for | Where you configure it |
|---|---|---|
| GitHub | Compare, PR fetch, issue fetch | Credentials on GitHub-related HTTP/GitHub nodes |
| OpenAI (or other model provider) | AI release-note generation | Chat model node used by the AI Agent |
| Jira (optional) | Jira issue enrichment | Jira node credentials |
| Slack (optional) | Posting release notes | Slack node credentials |

---

## Quick start

### 1) Import the workflow into n8n

**Option A (Recommended): Import from URL**
1. Open n8n
2. Workflows → top-right menu (**…**) → **Import from URL**
3. Paste the **Raw GitHub URL** of:  
   `workflows/ReleaseOrchestrator.json`

**Option B: Import from file**
1. Download `workflows/ReleaseOrchestrator.json`
2. In n8n → top-right menu (**…**) → **Import from File**

### 2) Post-import setup checklist
After import, open the workflow and do:

- [ ] Configure **GitHub** credentials on all GitHub HTTP/GitHub tool nodes
- [ ] Configure **OpenAI / LLM provider** credential for the Chat Model used by the AI Agent
- [ ] If using Jira: configure **Jira** credentials and set your base URL
- [ ] If posting to Slack: configure **Slack** credentials
- [ ] Review any node fields that may be environment-specific (Slack channel, Jira base URL, etc.)
- [ ] Enable the workflow

---

## Webhook API

### Endpoint
After importing, open the **Webhook** node and copy the **Test** or **Production** URL.

### Request schema

| Field | Type | Required | Example |
|---|---|---:|---|
| `repo` | string (`owner/repo`) | ✅ | `octocat/Hello-World` |
| `fromTag` | string | ✅ | `v1.2.0` |
| `toTag` | string | ✅ | `v1.3.0` |
| `options.postToSlack` | boolean | ❌ | `true` |
| `options.slackChannel` | string | ❌ (required if posting) | `#release-notes` |
| `options.jira.enabled` | boolean | ❌ | `true` |
| `options.jira.baseUrl` | string | ❌ (required if Jira enabled) | `https://YOURDOMAIN.atlassian.net` |
| `options.jira.allowedProjects` | string[] | ❌ | `["ABC","XYZ"]` |
| `githubToken` | string | ❌ | `ghp_...` |

> Note: Even if `githubToken` is supplied, users should still configure GitHub credentials in n8n unless you explicitly modify the workflow to use token-per-request headers.

---

## Examples

### Minimal request (`examples/request.minimal.json`)
```json
{
  "repo": "owner/repo",
  "fromTag": "v1.2.0",
  "toTag": "v1.3.0"
}
