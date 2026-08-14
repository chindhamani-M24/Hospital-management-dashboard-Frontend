# MediFlow deployment guide

## What is included

- `hospital-dashboard.html` — staff dashboard frontend.
- `index.html` — opens the dashboard at the hosted site root.
- `n8n-workflows/` — importable n8n webhook starter templates.
- `netlify.toml` — optional Netlify security headers.

## 1. Prepare Google Sheets

Create one spreadsheet with these sheets and use these exact headers:

### Doctors

`doctorId, doctorName, department, location, active`

### Availability

`slotId, doctorName, location, date, time, status, appointmentId`

### Appointments

`appointmentId, patientName, phone, email, doctor, location, date, time, status, reminder24hSent, reminder2hSent, createdAt`

### Notification Log

`appointmentId, channel, notificationType, status, sentAt`

Keep Google Form submissions in their own `Form Responses` sheet. Do not book directly in it.

## 2. Configure the booking workflow

1. Import `n8n-workflows/appointment-booking-webhook.json`.
2. After **Normalise request**, add Google Sheets **Get Row(s)** for `Availability` and filter by doctor, location, date, time, and `status = Available`.
3. Add an **If** node: slot found?
4. On **true**, update the matching availability row to `Booked`, append a `Confirmed` row to `Appointments`, and send confirmation.
5. On **false**, append a `Pending` appointment and notify the patient that staff will follow up.
6. Connect both paths to **Respond to dashboard**. Change the response’s `status` field to match the branch.
7. Activate the workflow, then copy its **Production URL**.

Set workflow concurrency to one while using Google Sheets, to prevent two patients booking the same slot simultaneously.

## 3. Add email and SMS notification logic

After creating or updating an appointment, add an **If** node:

```text
Email field is not empty?
  Yes → Gmail or SMTP node
  No  → Twilio / MSG91 SMS node
```

Log every sent message in `Notification Log`.

Create a second workflow with a **Schedule Trigger** every hour. Read `Confirmed` appointments and send a 24-hour or 2-hour reminder only if its corresponding reminder flag is false; update the flag after sending.

For cancellation, create `POST /webhook/cancel-appointment`: find appointment ID, update it to `Cancelled`, release its availability row to `Available`, then run the same email-first/SMS-fallback notification logic.

## 4. Configure cross-origin access (CORS)

Your dashboard will be on a different domain from n8n. Configure your n8n deployment to allow the deployed dashboard domain. For example, allow only:

```text
https://your-project.netlify.app
```

Do not use `*` for a production hospital system. Also use HTTPS for both n8n and the dashboard.

## 5. Deploy the frontend

### Netlify drag-and-drop

1. Create a Netlify account.
2. Go to **Add new project → Deploy manually**.
3. Drag the entire `outputs` folder into the upload area.
4. Open the resulting site URL. It redirects to the dashboard automatically.

### Vercel

Upload the `outputs` folder to a GitHub repository, import that repository in Vercel, and set the output directory to the repository root. No build command is needed.

## 6. Connect the deployed dashboard

1. Open the deployed site.
2. In the dashboard sidebar select **Connect webhook**.
3. Paste the active n8n booking workflow’s production URL, for example:

```text
https://your-n8n-domain/webhook/appointment-booking
```

4. Submit a test booking.
5. Confirm the n8n execution, Google Sheet update, and email/SMS result.

The current frontend shows demo records. After the booking flow works, connect `GET /webhook/appointments` to Google Sheets and update the dashboard table to load those live records.

## Evaluator checklist

- Dashboard opens from a public HTTPS URL.
- Google Form submission creates/updates an appointment through n8n.
- Dashboard booking triggers the n8n booking webhook.
- An available slot becomes booked and cannot be double-booked.
- Email is sent when email is present; SMS is used otherwise.
- Cancellation frees the slot and notifies the patient.
- Reminder logs show 24-hour and 2-hour reminder attempts.

## Safety for the demo

Use fictitious patient names, test email addresses, and test SMS numbers. Do not show public n8n editor access or private Google credentials to evaluators.
