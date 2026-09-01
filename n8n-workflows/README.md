# n8n-workflows — James ↔ Claude Code build loop

This folder is the bridge between James (Delos AI worker) and Claude Code for building n8n workflows.

## How the loop works
1. James writes a task spec in `tasks/` (one file per workflow) and opens a matching GitHub issue.
2. You run Claude Code locally on this repo: "pick up the open issue in n8n-workflows and build the workflow."
3. Claude Code commits the finished workflow JSON to `workflows/` and closes the issue.
4. Import the JSON into n8n (or push via the n8n API key), activate it, and tell James the webhook URL.
5. James tests the webhook end-to-end and reports back.

## Conventions
- `tasks/` — specs written by James. Use `_TEMPLATE.md` as the format.
- `workflows/` — finished, importable n8n workflow JSON built by Claude Code.
- Never commit credentials or API keys. Reference n8n credential *names* only (e.g. "Header Auth: n8n API Key").
- Webhook paths use the `james-` prefix (e.g. `james-run-template`, `james-sms`).

## Current status
- [ ] Builder workflow: `james-run-template` (see tasks/001-james-run-template.md)