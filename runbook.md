# Runbook

## Purpose

Operate and verify the local GitHub Issue Triage workflow safely.

## Normal operation

1. Start local n8n and open `http://localhost:5678`.
2. Activate `Sprint 2 — GitHub Issue Triage`.
3. Start GitHub CLI forwarding:

```powershell
gh webhook forward --repo=Chandravamshi1001/n8n-sprint2-issue-triage --events=issues --url="http://localhost:5678/webhook/sprint2-issue-triage"
```

4. Create a harmless GitHub issue with an `opened` action.
5. Expected result: one pending ledger row is inserted, one internal route-log row is written, and the ledger status becomes `recorded`.

Evidence: `evidence/successful-execution.png`, `evidence/ledger-final.png`, and `evidence/route-log-final.png`.

## Duplicate delivery

Replay the same opened issue payload or use GitHub redelivery when forwarding is available.

Expected result:

- The same `issue_key` remains one ledger row.
- `last_seen_at` may update.
- No additional route-log row is created.

Evidence: `evidence/duplicate-replay.png`.

## Route-log failure

Create a test issue whose title includes `intentional route-log failure test`.

Expected result:

- A ledger row remains present.
- `route_log_status` is `failed`.
- `error_note` is readable.
- No route-log row is created for that issue key.

Evidence: `evidence/failed-route-log.png`.

## Recovery action

1. Inspect the failed ledger row and n8n execution.
2. Correct the route-log node configuration if needed.
3. Replay the event only after confirming the duplicate guard and route-log mapping.
4. Verify the ledger and route-log table show the intended final state.

## Safe shutdown

After capturing evidence:

1. Export the workflow JSON and save documentation.
2. Deactivate the workflow in n8n.
3. Remove or disable the repository webhook if present.
4. Stop GitHub CLI forwarding with `Ctrl+C`.
5. Do not commit secrets, tokens, `.env` files, or webhook values.

## What this prototype does not test

This local course prototype does not test production hosting, high availability, retry queues, authentication, monitoring alerts, role-based access control, or real customer issues.
