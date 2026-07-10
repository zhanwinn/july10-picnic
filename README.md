# Scarborough Bluffs Picnic — Shared Dashboard

A single-file HTML dashboard for planning the picnic. Everyone who opens the link
sees and edits the **same** live data (checklists, assignments, expenses, etc.),
backed by a free Firebase project.

---

## 1. Create your Firebase project (~5 min)

1. Go to https://console.firebase.google.com and sign in with any Google account.
2. Click **Add project** → give it a name (e.g. `scarborough-picnic`) → you can
   disable Google Analytics for this, it's not needed → **Create project**.
3. In the left sidebar, click **Build → Firestore Database** → **Create database**.
   - Choose **Start in test mode** (we'll lock it down properly below).
   - Pick any region close to you (e.g. `nam5` / `us-east`).
4. Once created, go to **Project settings** (gear icon, top left) → scroll to
   **Your apps** → click the **</>** (Web) icon → register an app (any nickname,
   e.g. `picnic-web`) → **you do not need Firebase Hosting**, skip that step.
5. Firebase will show you a config object that looks like this:

   ```js
   const firebaseConfig = {
     apiKey: "AIzaSy...",
     authDomain: "scarborough-picnic.firebaseapp.com",
     projectId: "scarborough-picnic",
     storageBucket: "scarborough-picnic.appspot.com",
     messagingSenderId: "123456789",
     appId: "1:123456789:web:abcdef123456"
   };
   ```

   Copy these six values.

## 2. Paste your config into `index.html`

Open `index.html`, search for `firebaseConfig` (near the top of the `<script>`
block), and replace the placeholder values:

```js
const firebaseConfig = {
  apiKey: "YOUR_API_KEY",
  authDomain: "YOUR_PROJECT_ID.firebaseapp.com",
  projectId: "YOUR_PROJECT_ID",
  storageBucket: "YOUR_PROJECT_ID.appspot.com",
  messagingSenderId: "YOUR_SENDER_ID",
  appId: "YOUR_APP_ID"
};
```

with the real values from step 1. Save the file.

> This "API key" is not a secret — it's normal and safe for it to be visible in
> client-side code. Firestore's **security rules** (next step) are what actually
> control who can read/write your data, not this key.

## 3. Lock down Firestore security rules

By default "test mode" allows anyone to read/write for 30 days, then locks
everyone out. Instead, set a permanent rule scoped to just this app's data:

1. In the Firebase console, go to **Firestore Database → Rules**.
2. Replace the contents with:

   ```
   rules_version = '2';
   service cloud.firestore {
     match /databases/{database}/documents {
       match /picnic/{document} {
         allow read, write: if true;
       }
     }
   }
   ```

3. Click **Publish**.

This keeps things simple (no login required for your group) but means anyone
who has your Firebase config could technically write to this one document.
That's an acceptable tradeoff for a picnic checklist among friends — just don't
reuse this same Firebase project for anything sensitive later.

## 4. Push to GitHub and turn on Pages

```bash
git init
git add index.html README.md
git commit -m "Picnic dashboard with shared Firebase sync"
git branch -M main
git remote add origin https://github.com/YOUR_USERNAME/YOUR_REPO.git
git push -u origin main
```

Then on GitHub:
1. Go to your repo → **Settings → Pages**.
2. Under **Build and deployment**, set **Source** to `Deploy from a branch`.
3. Branch: `main`, folder: `/ (root)` → **Save**.
4. After a minute, your dashboard will be live at:
   `https://YOUR_USERNAME.github.io/YOUR_REPO/`

Send that link to everyone — they'll all see the same checklist, in real time.

## 5. Making future changes

- Edit `index.html` directly (or ask Claude to make the change), then:
  ```bash
  git add index.html
  git commit -m "describe your change"
  git push
  ```
- GitHub Pages redeploys automatically within about a minute.
- Your Firebase config never needs to change once it's set — only the app code does.

## How the sync works

- All shared data (attendee checklists, responsibility matrix, missing items,
  games, expenses, carpool assignments, event date/time) lives in one Firestore
  document at `picnic/shared`.
- Every edit writes to that document immediately.
- A real-time listener (`onSnapshot`) pushes other people's changes into your
  browser automatically — no refresh needed.
- Dark/light mode is the one thing that stays **per-device** (stored in
  `localStorage`), since that's a personal preference, not shared data.
- If Firebase can't be reached (e.g. you haven't filled in the config yet, or
  you're offline), the app falls back to working locally in that browser only,
  and shows "⚠️ Offline mode" in the top bar.
