# OVERSEER Springboard — Yayın Panosu

An **offline-first PWA** (Progressive Web App) that manages the publishing schedule, workload, and an idea/note board for 5 YouTube channels — all from a single screen.

Built as a single file (`index.html`) in plain JavaScript: no framework, no build step, no CDN dependency. It installs to your phone's home screen, works without internet, and can optionally sync between two devices via Firebase.

---

## ✨ Features

### 📊 Overview
- **Workload** stats: today / this week / this month / overdue / total planned / published
- **Capacity** math: hours-per-video × hours-per-day → weekly & monthly load, with an over-capacity warning
- **Publish tempo**: bar chart of the last 8 weeks
- **Channel breakdown**: each channel's next date + planned/published/overdue counts (tap to jump to that channel)
- **Next 5 publishes** (merged across all channels)

### 📺 Channels
- 5 channels with editable **name** and **color**
- Per video: **status** (to publish / published), **date**, **drag-to-reorder**, delete
- Two sections: **Published** and **To publish** (published sorted by date)
- Video detail:
  - **Prep** checklist: Voiceover · SRT · Edit · Pinned comment (N/4 badge)
  - **YouTube link** → "Open in Studio" (rewrites the link to the Studio URL so opening it doesn't count as a view)
  - **Note** field
  - **Add to calendar · 19:00** → `.ics` file (with a 1-hour-before reminder)
- **⚡ Bulk assign dates**: pick a pattern (Odd 1,3,5… / Even 2,4,6… / All) + channel + publish days + date range, then auto-distribute across consecutive matching days
  - **Filled dates: Keep / Overwrite** option — defaults to **Keep** (never overwrites existing dates; only fills empty ones)

### 🗓️ Calendar
- Monthly grid with channel-colored dots; tap a day to see that day's videos

### 🗒️ Board (notes / ideas)
- **Active** and **Archive** views (with counts on the toggle)
- Each note: **title**, **note text**, **link**, **date**, **multiple images**, **tags**, **favorite**
- Icons on the collapsed card: 🔗 opens the link · 📝 opens the note full-size (URLs clickable) · 🖼 opens images full-screen
- **Full-screen image viewer**: **pinch zoom**, drag to pan, swipe left/right between multiple images
- **Complete → Archive**: tap the ○ circle on the card header to move a note to Archive; tap ✓ in Archive to bring it back
- **Search**, **filter by tag**, **favorites filter**, **sort** (Manual drag / By date)
- **Add images**: upload from file or (on desktop) **paste** into a note; images are auto-shrunk to fit the cloud
- **Undo**: 5-second undo on delete and archive actions
- **Add via Share menu**: sharing a link/text from another app turns it into a note (Android/desktop PWA; not supported on iOS)

### 🌐 General
- **TR / EN** language support (per-device; not overwritten by the cloud)
- **Cloud sync** (optional, Firebase Firestore — no Google login, shared-code model)
- **Backup**: export/import/copy JSON
- **Works offline** (Service Worker), installs to the home screen

---

## 🚀 Setup (GitHub Pages)

1. Make this repo **public** (GitHub Pages requires a public repo on the free tier).
2. Go to **Settings → Pages → Branch: `main` / root** and save.
3. After a few minutes the app is live at:
   `https://<username>.github.io/<repo-name>/`
4. Open that URL on your phone → from the browser menu choose **"Add to Home Screen"** → it opens like an app and works offline.

> Updating: upload the changed file (usually just `index.html`) to the repo. HTML is served network-first, so it auto-updates while online; you may need to close and reopen the PWA for it to show. For changes that affect the Service Worker cache, bump the version (`yayin-panosu-vN`) in `sw.js`.

---

## ☁️ Cloud Sync (optional)

If two devices (e.g. you + your editor) want to share the same board, do a **one-time** Firebase setup:

1. [console.firebase.google.com](https://console.firebase.google.com) → **create project**
2. **Add a Web app** (`</>` icon) → copy the `firebaseConfig` object
3. **Build → Firestore Database → create database**
4. Set Firestore **rules** (shared-code model):
   ```
   rules_version = '2';
   service cloud.firestore {
     match /databases/{database}/documents {
       match /boards/{code} {
         allow read, write: if true;
       }
     }
   }
   ```
5. In the app go to **Overview → Cloud → Connect to cloud**:
   - **Owner**: paste `firebaseConfig` → generate a code → connect
   - **Joiner**: paste the code the owner shares via "Share invite code"

**Model:** there is no Google login; the board is accessed with a **long random code**. Anyone with the code can edit (like a shared password) — only give it to someone you trust. Data is stored in Firestore under the `boards/{code}` document.

---

## 💾 Data & Backup

- All data is stored on the device in **`localStorage`**:
  - `yayin-panosu-v1` → channels, videos, settings, notes
  - `yp-lang` → language preference (`tr` / `en`, per-device, never sent to the cloud)
- **Export backup**: Overview → Backup → **Export backup** (JSON). Save to Files/Drive or AirDrop it.
- **Restore** overwrites your current data — export a backup first.
- Get in the habit of backing up before any major change.

---

## 🧱 Architecture

- **Single file**: all HTML/CSS/JS lives in `index.html` (vanilla JS, no framework, no build)
- **PWA**: `manifest.webmanifest` + `sw.js` (Service Worker)
  - The app shell is cached → works offline
  - HTML is **network-first** (always fresh while online), other assets are **cache-first**
  - Firebase requests (gstatic SDK, googleapis, firebaseio) are bypassed/handled appropriately in the Service Worker
- **State management**: a single `state` object; every change triggers `autosave` (debounced) to `localStorage`, and pushes to Firestore if the cloud is enabled
- **Cloud**: Firebase Firestore with `persistentLocalCache` + `onSnapshot` (live); last-write-wins on concurrent updates

---

## 📁 File Structure

```
.
├── index.html            # The entire app (UI + logic)
├── sw.js                 # Service Worker (offline cache)
├── manifest.webmanifest  # PWA manifest (+ share target)
├── icon-180.png          # iOS home-screen icon
├── icon-192.png          # PWA icon
└── icon-512.png          # PWA icon (incl. maskable)
```

---

## ⚠️ Limitations & Notes

- **Add-via-Share does not work on iOS Safari** (Apple doesn't support Web Share Target). It works on Android/desktop PWA; it causes no problems on iPhone, it just won't appear in the share sheet.
- **Images** are stored as base64 inside `state` and sync to the cloud. Firestore has a **1 MB per-document** limit; images are auto-compressed on upload, and if the limit is exceeded you get a "cloud not updated" warning (local saving is unaffected). It's best not to pile up very heavy images in one board.
- `localStorage` is limited to ~5 MB per browser.
- Anyone who knows the board code can edit it (shared-code model).

---

## 🔒 Privacy

- By default, data stays only on your device. The cloud is off unless you explicitly connect.
- No third-party tracking/analytics; the app runs standalone.

---

*A companion tool for the OVERSEER production system. For personal use.*
