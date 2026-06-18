# GHL Form Fields — 30 to 300 Applicant Intake

Create this form in **Sites → Forms → Builder → Add Form** (or **Marketing →
Forms** depending on your GHL menu version). Name it **`30 to 300 Application`**.

## Fields (create in this exact order)

| # | Field Label | Field Type | GHL Field Name (Query Key) | Required |
|---|---|---|---|---|
| 1 | First Name | Text / Standard "First Name" | `first_name` | ✅ Yes |
| 2 | Last Name | Text / Standard "Last Name" | `last_name` | ✅ Yes |
| 3 | Date of Birth | Date | `date_of_birth` | ✅ Yes |
| 4 | Phone Number | Phone | `phone` | ✅ Yes |
| 5 | Email Address | Email | `email` | ✅ Yes |
| 6 | Video URL | Hidden (Text) | `video_url` | ❌ No |

### Field notes

- **First Name / Last Name / Phone / Email** — use GHL's built-in *standard*
  contact fields where available. These map automatically to the contact record.
- **Date of Birth** — GHL has a standard contact field "Date of Birth". Use it
  so the merge tag `{{contact.date_of_birth}}` works in the workflow. If you
  build it as a custom field instead, note its real merge tag and update the
  workflow email/sheet mappings accordingly.
- **Video URL** — must be a **Custom Field** of type **Text**, added to the form
  as a **Hidden** element.
  1. Go to **Settings → Custom Fields → Add Field → Text**, name it
     `Video URL`. GHL will generate a field key like `contact.video_url`.
  2. In the form builder, drag in the **Hidden Field** element (or set the
     Video URL field's visibility to *Hidden*) and bind it to the
     `Video URL` custom field.
  3. Confirm the rendered input's `name` attribute is `video_url`. The video
     recorder block writes to `[name="video_url"]`. If GHL renders a different
     name/key, update `VIDEO_URL_FIELD` at the top of
     `page1-video-recorder.html` to match.

## Form Settings

- **On Submit action:** `Go to URL` / `Open Funnel Step` → **Step 2 (Thank You)**
  of the `30 to 300 — Applicant Intake` funnel.
- **Create contact on submission:** Enabled (this is GHL default for forms).
- **Sticky contact:** optional.
- **reCAPTCHA:** optional — if enabled, test that it does not block the
  JS-triggered re-submit in the recorder intercept.

## Form Styling

Apply in the form builder **Styles** panel (matches the page theme):

| Property | Value |
|---|---|
| Form background | `#111111` |
| Label color | `#3AB5C0` |
| Input background | `#1a1a1a` |
| Input border | `rgba(255,255,255,0.1)` |
| Input text color | `#ffffff` |
| Button background | `#3AB5C0` |
| Button text color | `#ffffff` |
| Button label | **Submit My Application** |
| Field spacing | comfortable / 16px |

> The `page1-global-styles.html` block also force-styles `.hl-form` as a
> fallback in case the builder styles don't fully apply inside the funnel.

## Critical: Do NOT add a second submit button

The video recorder block intercepts **GHL's own** "Submit My Application"
button. Do not add a custom submit button in the recorder HTML. The recorder
listens for clicks on the GHL submit button, uploads the video first, stamps
the hidden `video_url` field, then lets the form submit.
