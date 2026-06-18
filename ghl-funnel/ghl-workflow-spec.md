# GHL Workflow — `30 to 300 — New Applicant Alert` (Inbound Webhook flow)

This is for the **Vercel-hosted** landing page (`site/index.html`). The page
uploads the video to Cloudinary, then POSTs the lead to a GHL **Inbound Webhook**.
This workflow receives that payload, creates/updates the contact, and alerts the team.

## Payload the page sends (form-encoded)
| Key | Example |
|---|---|
| `first_name` | John |
| `last_name` | Smith |
| `date_of_birth` | 1999-04-20 |
| `phone` | (555) 000-0000 |
| `email` | john@email.com |
| `video_url` | https://res.cloudinary.com/.../interview.webm |
| `program` | 30 to 300 |

## Prereqs
- Custom field **Video URL** (text) and **Date of Birth** (date) on the Contact
  object (Settings → Custom Fields). Note their field keys.
- Pipeline **`30 to 300 Applicants`** with a **`New Application`** stage.
- Custom value **`Hiring Email`** (Settings → Custom Values), e.g.
  `hiring@newstandardrestoration.com`.

---

## 1. Create the workflow + trigger
1. **Automation → Workflows → Create Workflow → Start from scratch.**
   Name it `30 to 300 — New Applicant Alert`.
2. Add trigger **"Inbound Webhook."**
3. Copy the **Webhook URL** it shows.
4. Paste that URL into `site/index.html`:
   ```js
   const GHL_WEBHOOK_URL = 'https://services.leadconnectorhq.com/hooks/.....';
   ```
5. **Capture a sample** so GHL learns the field names: click "Capture sample
   request" / "Listen," then submit the live page once. After a sample lands, the
   keys above become mappable as `{{inboundWebhookRequest.first_name}}`, etc.

---

## 2. Action 1 — Create/Update Contact (upsert)
- Action: **Create/Update Contact** (dedupes by email/phone).
- Map:
  - First Name → `{{inboundWebhookRequest.first_name}}`
  - Last Name → `{{inboundWebhookRequest.last_name}}`
  - Email → `{{inboundWebhookRequest.email}}`
  - Phone → `{{inboundWebhookRequest.phone}}`
  - Date of Birth (custom) → `{{inboundWebhookRequest.date_of_birth}}`
  - Video URL (custom) → `{{inboundWebhookRequest.video_url}}`

This sets the contact as the workflow's context for the actions below.

## 3. Action 2 — Internal email to hiring team
- Action: **Send Email**
- To: `{{custom_values.hiring_email}}`
- Subject: `New Application — {{contact.first_name}} {{contact.last_name}} | 30 to 300`
- Body (HTML):
  ```
  New Application Received — 30 to 300

  Name:          {{contact.first_name}} {{contact.last_name}}
  Date of Birth: {{contact.date_of_birth}}
  Phone:         {{contact.phone}}
  Email:         {{contact.email}}

  ▶ Watch Video Interview:
  {{inboundWebhookRequest.video_url}}
  ```
  (You can use `{{contact.video_url}}` instead — both work once Action 1 has run.)

## 4. Action 3 — Add to pipeline
- Action: **Create/Update Opportunity**
- Pipeline: `30 to 300 Applicants` · Stage: `New Application`
- Name: `{{contact.first_name}} {{contact.last_name}}`

## 5. Action 4 — Google Sheets row
- Action: **Google Sheets → Create Row**
- Spreadsheet `NSR — 30 to 300 Applicants`, tab `Applications`. Map:
  - Timestamp → `{{right_now}}` · First → `{{contact.first_name}}` ·
    Last → `{{contact.last_name}}` · DOB → `{{contact.date_of_birth}}` ·
    Phone → `{{contact.phone}}` · Email → `{{contact.email}}` ·
    Video → `{{contact.video_url}}`

## 6. Action 5 (optional) — Confirmation SMS
- Action: **Send SMS** → To `{{contact.phone}}`
- `Hey {{contact.first_name}}, we got your 30 to 300 application! We'll review your video and reach out in 24–48 hours. — NSR`

---

## 7. Publish & test
1. **Publish** the workflow (toggle on).
2. Submit a real application from the live Vercel page (record a short video).
3. Confirm: webhook shows the captured request → contact created with **Video URL**
   populated → hiring email arrives with the working link → opportunity created →
   Google Sheet row appended.

> If a mapped field comes through empty, open the captured request to check the
> exact key name and re-map (keys are case-sensitive).
