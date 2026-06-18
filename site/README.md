# NSR "30 to 300" — Vercel Landing Page

A single static page (`index.html`) that hosts the full branded application:
hero, how-it-works, compensation, the form, and the 30-second video recorder.
No backend required.

## How it works
1. Applicant fills the form and records a 30-second intro in the browser.
2. On submit, the page uploads the video to **Cloudinary** (unsigned) and gets a
   permanent `secure_url`.
3. It then POSTs the form fields + `video_url` to a **GHL Inbound Webhook**
   (form-encoded, `no-cors`).
4. GHL upserts the contact (by email/phone) and runs your workflow
   (hiring email + pipeline + Google Sheet). The page shows "Application Received."

Because it's our own page (not embedded in GHL), there's no iframe — the recorder
and styling just work.

## Configure (top of the `<script>` in `index.html`)
```js
const CLOUDINARY_CLOUD_NAME    = 'dmivshj7i';      // your cloud name
const CLOUDINARY_UPLOAD_PRESET = 'nsr-interviews'; // UNSIGNED preset
const GHL_WEBHOOK_URL = 'PASTE_YOUR_GHL_INBOUND_WEBHOOK_URL';
```
- Cloudinary: Settings → Upload → Upload presets → your preset must be **Unsigned**.
- GHL webhook: build a Workflow with an **Inbound Webhook** trigger and copy its URL
  (see `../ghl-funnel/ghl-workflow-spec.md`).

## Deploy on Vercel
**Option A — from GitHub (recommended):**
1. vercel.com → **Add New → Project** → import this repo.
2. **Root Directory:** `site`
3. Framework preset: **Other** (it's static). Build command: none. Output dir: `./`
4. Deploy. You get a `https://<project>.vercel.app` URL.

**Option B — Vercel CLI:**
```bash
npm i -g vercel
cd site
vercel        # preview
vercel --prod # production
```

## Custom domain
- In Vercel: **Project → Settings → Domains → Add** (e.g. `apply.newstandardrestoration.com`).
- Vercel shows a DNS record (usually a CNAME to `cname.vercel-dns.com`). Add that
  record where your DNS is managed.
- Use a **subdomain** for this page so it doesn't collide with anything pointing
  the root domain at GHL.

## Requirements / notes
- The camera recorder needs **HTTPS** — Vercel provides that automatically.
- Cloudinary free tier: 25GB storage + 25GB bandwidth/month.
- The GHL webhook URL is a capture endpoint (safe to expose); the Cloudinary
  unsigned preset is also public-safe. No private keys live in this page.
