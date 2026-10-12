# Week 6 Build Journal — Sprint 2 GitHub Issue Triage Automation

## Goal

Build a self-hosted, event-driven n8n workflow that triages new GitHub issues, records durable state, avoids duplicate route-log entries, and exposes route-log failures.

## What I built

I created the public repository `n8n-sprint2-issue-triage` and built the workflow `Sprint 2 — GitHub Issue Triage` in local n8n. The workflow accepts POST webhook events, allows only `opened` issue actions, normalizes the GitHub payload, and classifies issue titles as bug, feature, help, or other.

I created two n8n Data Tables: `github_issue_ledger` stores durable issue state, while `github_issue_route_log` stores the internal route outcome. A stable `issue_key` combines the repository name and issue number.

## Key design decisions

- I used deterministic title rules rather than an LLM so routing is explainable and repeatable.
- I placed the duplicate guard before the route-log write.
- New issues are inserted as `pending` before their route-log write is attempted.
- Existing issues update only `last_seen_at`, avoiding a second route-log row.
- An intentional failure phrase triggers a controlled error path that updates the ledger to `failed` with a safe error note.

## Testing and evidence

I verified bug, feature, help, and other classifications with their required route names. I replayed an existing issue payload and confirmed that no duplicate ledger or route-log row was added. I also tested the intentional route-log failure path and confirmed that the failed ledger record remained visible while no route-log row was inserted.

A later GitHub CLI forwarding/redelivery attempt returned HTTP 401 from the forwarding service. I documented that limitation and used a direct local replay of the same webhook-shaped payload to verify duplicate behavior.

## Challenge and resolution

A classification rule initially treated `title_lower` as plain text instead of an expression, causing valid bug and help titles to fall into the fallback route. I corrected the Switch input to use the expression `{{ $json.title_lower }}` and re-ran the tests. I also corrected a missing regex backslash in the Help rule.

## Extensibility observation

The workflow can be extended without replacing its core structure. A future version could add a priority field, labels, a Slack notification, or an assignee recommendation after classification. The stable `issue_key`, ledger-first design, and separate route-log table make those additions safer because duplicate protection and recovery state already exist.

## Next improvement

For a production version, I would host n8n on an always-on server, add monitoring and retry policies, secure the webhook path, add authentication controls, and replace local-only forwarding with a managed public endpoint.
