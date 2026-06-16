# Cloudinary Setup — NSR 30 to 300 Video Uploads

The video recorder uploads each applicant's 30-second clip to **Cloudinary**
using an **unsigned** upload preset. Unsigned uploads work directly from the
browser with no secret API key exposed — only the cloud name and preset name,
both of which are safe to ship in client-side code.

## 1. Create a free account
1. Go to [cloudinary.com](https://cloudinary.com) and sign up (free tier).
2. **Free tier limits:** 25 GB storage + 25 GB monthly bandwidth (counted as
   "credits"). A 30-second WebM clip is typically 3–10 MB, so this comfortably
   covers early hiring rounds (hundreds of applicants).

## 2. Copy your Cloud Name
- From the **Dashboard** (or **Settings → Account**), copy your **Cloud Name**.
  It looks like `dxy12abcd`. This is **not** secret.

## 3. Create an unsigned upload preset
1. Go to **Settings (gear icon) → Upload → Upload presets**.
2. Click **Add upload preset**.
3. Set:
   - **Signing Mode:** **Unsigned** ← required
   - **Upload preset name:** copy this (e.g. `nsr_interviews_unsigned`)
   - **Folder:** `nsr-interviews`
   - **Resource type / Asset type:** `Video` (or leave Auto — the recorder
     posts to the `/video/upload` endpoint explicitly).
4. *(Optional hardening)* In the preset, you may restrict **Allowed formats**
   to `webm,mp4` and set a **Max file size** (e.g. 50 MB) to prevent abuse.
5. **Save.**

## 4. Plug the values into the recorder
Open `page1-video-recorder.html` and edit the top of the `<script>`:

```javascript
var CLOUDINARY_CLOUD_NAME    = 'dxy12abcd';                 // your Cloud Name
var CLOUDINARY_UPLOAD_PRESET = 'nsr_interviews_unsigned';   // your preset name
var CLOUDINARY_FOLDER        = 'nsr-interviews';             // keep or change
```

## 5. How the upload works (reference)

```javascript
const formData = new FormData();
formData.append('file', recordedBlob, 'interview.webm');
formData.append('upload_preset', CLOUDINARY_UPLOAD_PRESET); // unsigned preset
formData.append('folder', 'nsr-interviews');

const res = await fetch(
  `https://api.cloudinary.com/v1_1/${CLOUDINARY_CLOUD_NAME}/video/upload`,
  { method: 'POST', body: formData }
);
const data = await res.json();
// data.secure_url is the permanent, HTTPS, streamable video URL
```

The returned `secure_url` is written into the form's hidden `video_url` field
and travels into GHL as a contact field, then into the email + Google Sheet.

## 6. Verify
- After a test submission, open your Cloudinary **Media Library → nsr-interviews**
  folder — the clip should appear.
- Open the `secure_url` in a new tab; it should stream on any device.

## Notes & alternatives
- **iOS Safari** records `video/mp4`; desktop Chrome records `video/webm`. The
  recorder auto-detects the supported MIME type, and Cloudinary accepts both.
- If you outgrow the free tier, Cloudinary's paid plans scale up, or you can
  swap the `uploadToCloudinary()` function for another unsigned-upload host
  (e.g. an S3 presigned URL or Bunny Stream) without touching the rest of the
  flow — just keep returning a URL string.
