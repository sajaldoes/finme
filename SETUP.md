# Money Tracker - Standalone App Setup

This is a real standalone app (no Apps Script backend) that talks to your Google Sheet
directly via the Sheets API. It works as a normal website AND installs as an app-like
icon on Android (and iPhone) with no browser address bar.

One-time setup, roughly 15 minutes. You only do this once.

---

## 1. Find your Spreadsheet ID

Open your Google Sheet. The URL looks like:

```
https://docs.google.com/spreadsheets/d/1AbCdeFGhIJKlmnOPQRstuVWXyz1234567890/edit
```

The long string between `/d/` and `/edit` is your **Spreadsheet ID**. Copy it.

---

## 2. Create a Google Cloud OAuth Client ID

1. Go to https://console.cloud.google.com/
2. Top-left, click the project dropdown -> **New Project**. Name it anything
   (e.g. "money-tracker"), click **Create**. Wait for it to finish, then make
   sure it's the selected project (top-left dropdown).
3. In the search bar at the top, search **"Google Sheets API"** -> open it ->
   click **Enable**.
4. In the left sidebar: **APIs & Services -> OAuth consent screen**.
   - User type: **External** (this is fine even for personal use).
   - Fill in App name (e.g. "Money Tracker"), your email for the two email
     fields. Save and continue through the remaining steps (Scopes, Test users
     - you can skip adding scopes/test users here, just click through).
   - On the **Test users** step, click **+ Add users** and add your own Google
     account email. This keeps the app in "Testing" mode, which is fine since
     only you will use it - no Google verification review needed.
   - Finish and go back to the dashboard.
5. Left sidebar: **APIs & Services -> Credentials -> + Create Credentials ->
   OAuth client ID**.
   - Application type: **Web application**.
   - Name: anything (e.g. "Money Tracker Web").
   - **Authorized JavaScript origins**: add the URL(s) you'll host this app on.
     You need this for every place you'll open the app from:
     - `https://YOUR_GITHUB_USERNAME.github.io` (for the deployed app, see step 3)
     - `http://localhost:8000` (optional, only if you want to test locally first)
   - Click **Create**. Copy the **Client ID** shown (ends in
     `.apps.googleusercontent.com`).

---

## 3. Fill in the config

Open `index.html` and edit the top of the `<script>` block:

```js
var APP_CONFIG = {
  CLIENT_ID: 'PASTE_YOUR_CLIENT_ID_HERE.apps.googleusercontent.com',
  SPREADSHEET_ID: 'PASTE_YOUR_SPREADSHEET_ID_HERE',
  ...
};
```

Double-check `EXPENSE_FALLBACK_START_ROW` / `EARNING_FALLBACK_START_ROW` and the
column letters match your sheet (only used as a fallback if a sheet is ever
completely empty - normally the app auto-detects the real start row, same as
before).

---

## 4. Deploy to GitHub Pages (free hosting)

1. Create a new **public** repository on GitHub (e.g. `money-tracker`).
2. Upload these files to the repo root, keeping the folder structure:
   - `index.html`
   - `manifest.json`
   - `sw.js`
   - `icons/icon-192.png`
   - `icons/icon-512.png`
   - `icons/icon-512-maskable.png`
3. Repo -> **Settings -> Pages** -> Source: **Deploy from a branch** -> Branch:
   `main` / folder `/ (root)` -> Save.
4. Wait a minute, then your app is live at:
   `https://YOUR_GITHUB_USERNAME.github.io/money-tracker/`
5. Go back to Google Cloud Console -> Credentials -> your OAuth client -> make
   sure this exact URL (without a trailing path after the repo name, just the
   origin `https://YOUR_GITHUB_USERNAME.github.io`) is in **Authorized
   JavaScript origins**. Save.

---

## 5. Use it

- **On desktop/web**: just open the GitHub Pages URL, click "Sign in with
  Google," approve access, done.
- **On Android**: open the URL in Chrome -> menu (⋮) -> **Install app** (or
  **Add to Home Screen**). It'll install as a real standalone app icon, no
  browser bar.
- **On iPhone**: open in Safari -> Share -> **Add to Home Screen**. Same
  standalone behavior.

The first sign-in on each device shows Google's consent screen once (since the
app is in "Testing" mode, you may see an "unverified app" warning - click
**Advanced -> Go to Money Tracker (unsafe)** the first time; this is normal for
apps you built for yourself and haven't submitted for Google's verification
review, and it only ever asks for access to your own Sheets, not anything
else).

---

## Notes

- No Apps Script project is used anymore for this new app. Your old Apps
  Script Web App still works fine if you want to keep it as a backup - they
  both read/write the same sheet.
- Sign-in is stored for your browser session only (`sessionStorage`) - closing
  all tabs/the browser clears it, and you'll sign in again next time. This was
  a deliberate choice over persistent storage.
- Every add-entry does exactly 2 API calls (append the row, then sort) instead
  of going through an Apps Script execution - this is the main source of the
  speed improvement.
- Data loading does exactly 2 API calls total (one batched read to find where
  your data starts, one batched read for the actual rows across both sheets)
  instead of the multiple round-trips the Apps Script version needed.
