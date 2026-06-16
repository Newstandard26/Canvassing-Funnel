# GHL Workflow Spec — `30 to 300 — New Applicant Alert`

This workflow fires when the application form is submitted. It emails the
hiring team (with the video link), adds the applicant to a pipeline, logs the
row to Google Sheets, and optionally texts the applicant a confirmation.

## Prerequisites

1. **Custom Value — Hiring Email**
   Go to **Settings → Custom Values → Add Custom Value**:
   - Name: `Hiring Email`
   - Value: `hiring@newstandardrestoration.com`
   - Merge tag: `{{custom_values.hiring_email}}`
2. **Pipeline** — create **Opportunities → Pipelines → Add Pipeline**:
   - Pipeline name: `30 to 300 Applicants`
   - First stage: `New Application` (add more stages as you like:
     `Video Reviewed`, `Interview Scheduled`, `Hired`, `Passed`).
3. **Google account connected** — **Settings → Integrations → Google** →
   connect the account that owns the applicant spreadsheet.
4. **Sending domain connected** — for the email action's "From" address.

---

## Create the workflow

**Automation → Workflows → Create Workflow → Start from scratch.**
Name it **`30 to 300 — New Applicant Alert`**.

### Trigger
- **Add Trigger → Form Submitted**
- **Filter:** `Form` **is** `30 to 300 Application`
- Save.

---

### Action 1 — Internal Email Notification
- **Add Action → Send Email**
- **From Name:** `NSR Recruiting`
- **From Email:** your connected sending domain (e.g. `recruiting@newstandardrestoration.com`)
- **To:** `{{custom_values.hiring_email}}`
- **Subject:**
  ```
  New Application — {{contact.first_name}} {{contact.last_name}} | 30 to 300 Program
  ```
- **Body** — switch the email editor to **HTML / source mode** and paste:
  ```html
  <div style="font-family:Arial,Helvetica,sans-serif;max-width:560px;margin:0 auto;background:#111;color:#fff;border:1px solid rgba(255,255,255,0.1);border-radius:12px;overflow:hidden;">
    <div style="background:#3AB5C0;padding:18px 24px;">
      <h2 style="margin:0;color:#fff;font-size:20px;">New Application Received</h2>
      <p style="margin:4px 0 0;color:#06343a;font-weight:bold;">30 to 300 Program — New Standard Restoration</p>
    </div>
    <div style="padding:24px;">
      <table style="width:100%;border-collapse:collapse;color:#fff;font-size:15px;">
        <tr><td style="padding:6px 0;color:#3AB5C0;width:150px;">Name</td><td>{{contact.first_name}} {{contact.last_name}}</td></tr>
        <tr><td style="padding:6px 0;color:#3AB5C0;">Date of Birth</td><td>{{contact.date_of_birth}}</td></tr>
        <tr><td style="padding:6px 0;color:#3AB5C0;">Phone</td><td>{{contact.phone}}</td></tr>
        <tr><td style="padding:6px 0;color:#3AB5C0;">Email</td><td>{{contact.email}}</td></tr>
        <tr><td style="padding:6px 0;color:#3AB5C0;">Submitted</td><td>{{right_now}}</td></tr>
      </table>
      <hr style="border:none;border-top:1px solid rgba(255,255,255,0.15);margin:20px 0;">
      <a href="{{contact.video_url}}" style="display:inline-block;background:#3AB5C0;color:#fff;text-decoration:none;font-weight:bold;padding:12px 22px;border-radius:8px;">▶ Watch Video Interview</a>
      <p style="margin:12px 0 0;color:rgba(255,255,255,0.55);font-size:13px;word-break:break-all;">{{contact.video_url}}</p>
    </div>
  </div>
  ```

> **Plain-text version** (if you prefer the simple layout from the spec):
> ```
> New Application Received
> 30 to 300 Program — New Standard Restoration
>
> Name:          {{contact.first_name}} {{contact.last_name}}
> Date of Birth: {{contact.date_of_birth}}
> Phone:         {{contact.phone}}
> Email:         {{contact.email}}
> Submitted:     {{right_now}}
> ──────────────────────────────────────
> ▶ Watch Video Interview:
> {{contact.video_url}}
> ──────────────────────────────────────
> ```
> GHL uses `{{right_now}}` / `{{right_now_field}}` for the current timestamp.
> If your sub-account supports Liquid date filters, you may use
> `{{ "now" | date: "%B %d, %Y at %I:%M %p" }}` instead.

---

### Action 2 — Add to Pipeline (Opportunity)
- **Add Action → Create / Add Opportunity**
- **Pipeline:** `30 to 300 Applicants`
- **Stage:** `New Application`
- **Opportunity Name:** `{{contact.first_name}} {{contact.last_name}}`
- **Status:** `Open`
- **Lead Value:** optional

---

### Action 3 — Google Sheets Log
- **Add Action → Google Sheets** (native integration; under "Premium Actions"
  in some sub-accounts).
- **Account:** select your connected Google account.
- **Spreadsheet:** `NSR — 30 to 300 Applicants`
- **Worksheet / Tab:** `Applications`
- **Column mapping** (GHL reads the header row and shows column names):

  | Sheet Column (header) | Map to value |
  |---|---|
  | Timestamp | `{{right_now}}` |
  | First Name | `{{contact.first_name}}` |
  | Last Name | `{{contact.last_name}}` |
  | Date of Birth | `{{contact.date_of_birth}}` |
  | Phone | `{{contact.phone}}` |
  | Email | `{{contact.email}}` |
  | Video Link | `{{contact.video_url}}` |

> If GHL shows columns as A/B/C instead of header names, map by position:
> A→Timestamp, B→First Name, C→Last Name, D→Date of Birth, E→Phone, F→Email,
> G→Video Link.

---

### Action 4 — Confirmation SMS to Applicant (optional)
- **Add Action → Send SMS**
- **To:** `{{contact.phone}}`
- **Message:**
  ```
  Hey {{contact.first_name}}, we received your 30 to 300 application! Our team will review your video and reach out within 24–48 hours. — NSR
  ```
- Requires a registered/verified sending number (A2P 10DLC) on the sub-account.

---

## Workflow Settings & Publish
- **Settings → Allow Re-Entry:** your call (off prevents duplicate alerts if a
  contact re-submits; on logs every submission).
- Toggle the workflow from **Draft → Publish** (top right).

## Test the full chain
1. Submit a real test application from `/apply` with a recorded video.
2. Confirm: contact created → email arrives with a working video link →
   opportunity appears in `30 to 300 Applicants / New Application` → a new row
   lands in the Google Sheet → (optional) SMS received.
3. If `{{contact.video_url}}` is blank in the email/sheet, the hidden field
   wasn't stamped — re-check the form field name and the recorder selectors
   (see README → "Confirm your selectors").
