# NSR "30 to 300" — GoHighLevel Applicant Intake Funnel

A copy-paste GoHighLevel funnel for **New Standard Restoration's "30 to 300"**
sales-rep recruitment program. It replicates the branded landing page using GHL
Custom HTML blocks, captures applicants natively into the GHL CRM via a GHL
form, records a 30-second video intro in the browser, uploads it to Cloudinary,
and fires a workflow that emails the hiring team and logs every applicant to a
Google Sheet.

> **Branding:** teal `#3AB5C0` on near-black `#090909`. Fonts: Bebas Neue,
> Barlow Condensed, Barlow.
>
> **Design source:** these blocks reproduce `../reference/landing-page.html`
> (the original standalone page) — same copy, stats, compensation, animations,
> and logo — re-packaged as GHL Custom HTML blocks that integrate with a GHL
> native form, Cloudinary, and a GHL workflow.

---

## Files

| File | What it is | Where it goes |
|---|---|---|
| `page1-global-styles.html` | Google Fonts + CSS variables + form theming | **First** block on Page 1 |
| `page1-hero-section.html` | Fixed nav + hero + stats strip (real base64 logo embedded) | Page 1, below styles |
| `page1-how-it-works.html` | "How It Works" steps + "Compensation" cards | Page 1, below hero |
| `page1-video-recorder.html` | 30-sec recorder + Cloudinary upload + form intercept | Page 1, **directly below the GHL form** |
| `page2-thankyou.html` | Full "Application Received" page | Page 2 (only block) |
| `ghl-form-fields.md` | Exact form fields + styling to build in GHL | reference |
| `ghl-workflow-spec.md` | Step-by-step workflow build | reference |
| `cloudinary-setup.md` | Get Cloudinary creds + unsigned preset | reference |
| `README.md` | This walkthrough | reference |
| `../reference/landing-page.html` | The original standalone landing page this funnel reproduces | reference / design source |

---

## How the flow works

```
Applicant on /apply
  ├─ Fills GHL native form (name, DOB, phone, email)
  ├─ Records 30-sec video intro (in-browser, MediaRecorder)
  └─ Clicks "Submit My Application" (GHL's button)
        │  ← recorder INTERCEPTS the click
        ├─ uploads the clip to Cloudinary (unsigned) → secure_url
        ├─ writes secure_url into hidden field [name="video_url"]
        └─ re-fires the real submit
              │
GHL creates the contact (incl. video_url custom field)
  └─ Form "Submitted" trigger fires the workflow:
        ├─ Action 1: email hiring team (with video link)
        ├─ Action 2: add to "30 to 300 Applicants" pipeline
        ├─ Action 3: append row to Google Sheet
        └─ Action 4: (optional) confirmation SMS to applicant
              │
Applicant is redirected to /thank-you (Page 2)
```

---

## Setup walkthrough

### 0. Prereqs
- A GHL sub-account with **Funnels**, **Forms**, **Workflows**, and the
  **Google Sheets** workflow action available.
- A free **Cloudinary** account (see `cloudinary-setup.md`).
- A Google account connected to GHL that owns the applicant spreadsheet.

### 1. Cloudinary
Follow `cloudinary-setup.md`. Then edit the top of
`page1-video-recorder.html`:
```javascript
var CLOUDINARY_CLOUD_NAME    = 'your_cloud_name_here';
var CLOUDINARY_UPLOAD_PRESET = 'your_preset_name_here';
```

### 2. Create the funnel
**Sites → Funnels → New Funnel** → name it **`30 to 300 — Applicant Intake`**.
- **Step 1:** path `/apply` (name: *Application*)
- **Step 2:** path `/thank-you` (name: *Thank You*)
- Connect your subdomain / custom domain in funnel settings.
- **Page settings → Background color: `#090909`** on BOTH steps (prevents white
  gaps between Custom HTML rows).

### 3. Build the form
Follow `ghl-form-fields.md` to create the **`30 to 300 Application`** form
(5 visible fields + hidden `video_url`). Set its on-submit action to open
**Step 2** of the funnel, and style it to the theme.

### 4. Build Page 1 (`/apply`)
In the funnel builder, add full-width 1-column rows, each containing a
**Custom HTML / Code** element, in this order:

1. **Custom HTML** → paste `page1-global-styles.html`  *(must be first)*
2. **Custom HTML** → paste `page1-hero-section.html`
3. **Custom HTML** → paste `page1-how-it-works.html`
4. **GHL Form element** → select the `30 to 300 Application` form
5. **Custom HTML** → paste `page1-video-recorder.html`  *(directly below the form)*
6. *(optional)* a footer row with phone + copyright

> The hero and how-it-works CTAs link to `#nsr-apply`, an anchor placed just
> above the video recorder, so "Apply" buttons scroll users to the form/recorder.

### 5. Build Page 2 (`/thank-you`)
Add one full-width 1-column row → **Custom HTML** → paste `page2-thankyou.html`.
The footer already shows `224-302-1177` — update it if your number changes.

### 6. Build the workflow
Follow `ghl-workflow-spec.md` to create **`30 to 300 — New Applicant Alert`**
(email → pipeline → Google Sheet → optional SMS). Set the `Hiring Email`
custom value and create the `30 to 300 Applicants` pipeline first.

### 7. Create the Google Sheet
Spreadsheet **`NSR — 30 to 300 Applicants`**, tab **`Applications`**, Row 1
headers:

| A | B | C | D | E | F | G |
|---|---|---|---|---|---|---|
| Timestamp | First Name | Last Name | Date of Birth | Phone | Email | Video Link |

Share it with the Google account connected to GHL.

### 8. Publish & test
Publish the funnel and the workflow, then run a real end-to-end test
(see "Final checklist" below).

---

## ⚠️ Confirm your selectors (do this before going live)

GHL's form HTML differs between themes/versions. The recorder needs to find
**(a)** the submit button and **(b)** the hidden `video_url` input.

1. Open the published `/apply` page in Chrome → right-click the **Submit My
   Application** button → **Inspect**.
2. Check the button's tag/classes. The recorder's default selector is:
   ```javascript
   var GHL_SUBMIT_SELECTOR = '.ghl-footer-custom-button, button[type="submit"], .hl-btn[type="submit"]';
   ```
   If your button doesn't match, add/replace with the actual class
   (e.g. `.hl-form-submit-btn`).
3. Inspect the hidden field. Confirm its `name` attribute is exactly
   `video_url`. If GHL renders something else, update:
   ```javascript
   var VIDEO_URL_FIELD = '[name="video_url"]';
   ```
4. Test: leave the video unrecorded and click submit → you should be blocked
   with a prompt to record. Record, then submit → you should see "Uploading
   your video…", then the form submits and redirects to `/thank-you`.
5. Confirm the new contact in GHL has the `Video URL` field populated.

> The recorder also re-binds for ~20 seconds after load (GHL renders forms
> asynchronously), so it works even if the form mounts late.

---

## Implementation notes (GHL specifics)

- Custom HTML blocks run in the **page's global scope** (no iframes), so the
  recorder can reach the GHL form's DOM. All classes are prefixed `nsr-` and
  script vars are wrapped in an IIFE to avoid collisions.
- **Do not** re-import jQuery — GHL injects its own.
- **Do not** add a separate submit button in the recorder; it intercepts GHL's
  button. Adding a second submit button will break the single-submit flow.
- The real NSR logo (a 500×500 PNG) is embedded as a **base64 data-URI** in the
  hero, nav, and thank-you blocks — no external URL (per spec). The canonical
  copy lives in `../reference/nsr-logo.b64.txt`; if the logo changes, drop the
  new `data:image/png;base64,…` value into that file and re-paste it into each
  `src="data:image/png;base64,…"` attribute.
- The recorder caches the upload, so the JS-triggered re-submit never uploads
  the video twice.

---

## Browser support
- **Chrome / Edge / Firefox (desktop & Android):** WebM recording. ✅
- **Safari (macOS / iOS):** MP4 recording; the recorder auto-detects the MIME
  type and Cloudinary accepts it. ✅
- Camera/mic access requires **HTTPS** — published GHL funnels are served over
  HTTPS, so this works in production. (It will not work over plain `http://`.)

---

## Final checklist
- [ ] Global styles block loads fonts and CSS variables (it's the first block)
- [ ] Hero renders with logo, headline, and stats strip
- [ ] How It Works + Earnings render correctly
- [ ] GHL form captures all five fields + hidden `video_url`
- [ ] Recorder enables camera, counts down 30s, auto-stops at 0
- [ ] Retake works
- [ ] On submit: video uploads to Cloudinary, URL injected, form submits
- [ ] Contact created with all fields incl. video URL
- [ ] Workflow fires: email sent, pipeline stage set, Sheet row appended
- [ ] Thank You page renders correctly
- [ ] Mobile responsive across sections
- [ ] Page background `#090909` set on both steps (no white gaps)
- [ ] Selectors confirmed in DevTools on the live page
