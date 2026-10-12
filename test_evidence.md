# Test Evidence

## Test environment

- Public repository: `Chandravamshi1001/n8n-sprint2-issue-triage`
- Workflow: `Sprint 2 — GitHub Issue Triage`
- n8n: local self-hosted instance at `http://localhost:5678`
- State tables: `github_issue_ledger` and `github_issue_route_log`

## Classification tests

| Test | Example title | Expected category | Expected route | Result |
|---|---|---:|---|---|
| Bug | `Bug: dashboard shows an error after login` | bug | bug-triage | Passed |
| Feature | `Feature request: add email notifications` | feature | feature-requests | Passed |
| Help | `Help: question about account setup` | help | support | Passed |
| Other | `Update documentation wording` | other | general-triage | Passed |

The final ledger and route-log tables show one recorded route for each category.

Evidence:

- `evidence/ledger-final.png`
- `evidence/route-log-final.png`
- `evidence/successful-execution.png`

## Duplicate-delivery test

An existing issue payload was replayed locally for issue `#4`.

Expected result: the existing ledger row is retained, `last_seen_at` can change, and no second route-log row is created.

Result: Passed. The workflow followed `If ledger row exists` to `Update existing last seen`.

Evidence: `evidence/duplicate-replay.png`.

## Opened-action gate

The workflow's `Opened action?` node accepts only `opened` issue actions. Non-opened actions stop before a ledger or route-log write.

Evidence: `evidence/opened-action-gate.png`.

## Intentional route-log failure

A temporary issue containing `intentional route-log failure test` triggered the intentional failure path.

Expected result: the ledger row is retained with `route_log_status = failed`, a readable error note is stored, and no route-log row is inserted.

Result: Passed.

Evidence: `evidence/failed-route-log.png`.

## GitHub CLI forwarding note

GitHub CLI forwarding was started for normal local testing. During later GitHub delivery/redelivery attempts, GitHub's managed webhook-forwarder returned HTTP 401, so those deliveries did not reach n8n. The duplicate guard was additionally verified by a direct local replay of the same GitHub-style webhook payload. This note documents the environment limitation without changing the workflow behavior.
