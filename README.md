# n8n workflow templates

Import the JSON files from n8n: **Workflows → Import from File**.

`appointment-booking-webhook.json` is a tested starting webhook: it accepts a frontend booking request and returns a structured JSON response. Add the Google Sheets and notification nodes described in the deployment guide after `Normalise request` and before `Respond to dashboard`.

`appointments-list-webhook.json` creates the GET endpoint for the dashboard. Add a Google Sheets **Get Row(s)** node before its response node and set the response expression to return those records.

Use the **Production URL** displayed by each Webhook node only after activating the workflow.
