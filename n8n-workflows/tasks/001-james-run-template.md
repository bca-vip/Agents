# Task 001: james-run-template builder workflow

## Goal
One webhook that lets James instantiate pre-approved n8n workflow templates with variables, via the n8n public API. This is the core of the "template library" approach (Option B).

## Trigger
- Type: Webhook (POST)
- Path: james-run-template
- Expected input JSON:
```json
{
  "template": "sms_alert",
  "name": "SMS alert - demo",
  "vars": {
    "to": "+18653865454",
    "message": "Hello from James"
  }
}
```

## Nodes / flow
1. **Webhook** (POST, path `james-run-template`, respond immediately with 200 + `{"status": "received"}`).
2. **HTTP Request** — `POST https://bcvips.app.n8n.cloud/api/v1/workflows` with:
   - Header Auth credential: "n8n API Key" (header `X-N8N-API-KEY`)
   - Body: the template's workflow JSON (from a lookup table or a Set node keyed on `template`), with `vars` substituted into the relevant node parameters and `name` set from input. `active: false` on creation.
3. **HTTP Request** (optional, second step) — activate the cloned workflow via the API so it is live immediately.
4. **Respond to Webhook** — return `{"workflowId": "...", "status": "created"}`.

## Templates to support first
- `sms_alert` — reuse the existing James SMS Sender logic (email-to-SMS via Mailgun, no Twilio).
- `email_digest` — send an email summary via Mailgun to shell@michellesnow360.com.
- `lead_notification` — email + SMS ping when a new lead arrives.

## Credentials
- "n8n API Key" (Header Auth, header X-N8N-API-KEY)
- Mailgun credential (already exists, used by James SMS Sender)

## Output / success criteria
- POST to https://bcvips.app.n8n.cloud/webhook/james-run-template with the JSON above returns 200 and a workflowId.
- The cloned workflow appears in n8n, active, and triggering it sends the SMS/email.
- James will run a live end-to-end test once Shell confirms it's deployed.

## Notes
- Keep template JSON in the workflow itself (Set node or Code node with a lookup) so no external storage is needed.
- Reject unknown template names with a clear error message in the response.
- Do not hardcode any phone numbers or emails in the template JSON; always take them from `vars`.
