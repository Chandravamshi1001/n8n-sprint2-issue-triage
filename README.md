# Sprint 2 — GitHub Issue Triage Automation

## Purpose

This self-hosted n8n workflow triages newly opened GitHub issues from a public test repository. It normalizes the issue data, classifies the title with deterministic rules, writes a durable ledger row, records an internal route result, prevents duplicate route-log records, and retains a visible failure state when a route-log write fails.

## Public repository

- Repository: https://github.com/Chandravamshi1001/n8n-sprint2-issue-triage
- Visibility: Public
- n8n workflow: `Sprint 2 — GitHub Issue Triage`
- Local n8n URL: `http://localhost:5678`

## Services used

- GitHub public repository for harmless test issues
- GitHub CLI webhook forwarding during local testing
- Self-hosted n8n running locally
- n8n Data Tables: `github_issue_ledger` and `github_issue_route_log`

## Workflow node sequence

1. `GitHub issue webhook`
2. `Opened action?`
3. `Normalize issue`
4. `Classify title`
5. One of: `Set bug route`, `Set feature route`, `Set help route`, or `Set other route`
6. `If ledger row exists` → `Update existing last seen`
7. `If ledger row does not exist` → `Insert pending ledger row`
8. `Intentional failure test?`
9. Normal path: `Record internal route` → `Update route-log recorded`
10. Failure path: `Simulate route-log failure` → `Update route-log failed`

## Normalized field mapping

| Field | Source |
|---|---|
| `repository_full_name` | GitHub webhook repository full name |
| `issue_number` | GitHub webhook issue number |
| `issue_key` | `repository_full_name + "#" + issue_number` |
| `issue_url` | GitHub issue HTML URL |
| `title` | GitHub issue title |
| `title_lower` | Lowercased issue title |
| `reporter` | GitHub username |
| `opened_at` | GitHub issue creation timestamp |
| `last_seen_at` | n8n processing timestamp |

## Classification rules

| Title rule | Category | Route name |
|---|---|---|
| Contains `bug` or `error` | `bug` | `bug-triage` |
| Contains `feature` or `request` | `feature` | `feature-requests` |
| Contains `help` or `question` | `help` | `support` |
| Fallback | `other` | `general-triage` |

The workflow uses deterministic Switch rules, not an LLM.

## Duplicate protection

`issue_key` is checked before a route-log write. A new issue creates one ledger row and one route-log row. A repeated delivery updates only `last_seen_at`; it does not create another route-log entry.

## Evidence

See [test_evidence.md](test_evidence.md) and the screenshots in the `evidence/` folder.

## Safety

This repository contains only harmless course-test issues. No credentials, tokens, webhook secrets, personal data, or production data are committed.
