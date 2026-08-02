# RHI Badminton Bracket — Setup Guide

This turns `index.html` into a live, mobile-friendly page anyone can open — with
one admin (PIN-protected) editing scores/courts, and everyone else seeing a
read-only view that updates instantly, no refresh needed.

Two things to set up, both free: **Firebase** (holds the live tournament data)
and **GitHub Pages** (hosts the page itself).

---

## Part 1 — Firebase (real-time database)

1. Go to [console.firebase.google.com](https://console.firebase.google.com) and sign in with a Google account.
2. Click **Add project** → name it anything (e.g. `rhi-badminton`) → you can disable Google Analytics for this project → **Create project**.
3. In the left sidebar, click **Build → Firestore Database** → **Create database** → choose a location close to you → start in **production mode** → **Enable**.
4. Go to **Rules** (tab at the top of the Firestore page) and replace the contents with:

   ```
   rules_version = '2';
   service cloud.firestore {
     match /databases/{database}/documents {
       match /tournaments/{tournamentId} {
         allow read: if true;
         allow write: if true;
       }
     }
   }
   ```

   **Note on security:** this makes the tournament doc writable by anyone who has the page's Firebase config (which is visible in the page source — that's normal for Firebase web apps). The admin PIN in the app is a soft gate for your organizers, not a hard security wall. That's an appropriate tradeoff for a casual internal event; it is *not* appropriate for sensitive data. Click **Publish** to save the rules.
5. Go to **Project settings** (gear icon, top left) → scroll to **Your apps** → click the **`</>`** (Web) icon → give it a nickname (e.g. `bracket-page`) → **Register app**. Don't bother with Firebase Hosting when prompted — you're using GitHub Pages instead.
6. Copy the `firebaseConfig` object shown (it looks like the block below) — you'll need it in Part 2.

   ```js
   const firebaseConfig = {
     apiKey: "AIza...",
     authDomain: "rhi-badminton-xxxx.firebaseapp.com",
     projectId: "rhi-badminton-xxxx",
     storageBucket: "rhi-badminton-xxxx.appspot.com",
     messagingSenderId: "...",
     appId: "..."
   };
   ```

That's it for Firebase — the free "Spark" plan covers this easily (way under the daily free quota for an event this size).

---

## Part 2 — Wire the config into `index.html`

Open `index.html` in a text editor, find this block near the top (inside the first `<script>` tag):

```js
window.FIREBASE_CONFIG = {
  apiKey: "PASTE_YOUR_API_KEY",
  authDomain: "PASTE_YOUR_PROJECT.firebaseapp.com",
  projectId: "PASTE_YOUR_PROJECT_ID",
  storageBucket: "PASTE_YOUR_PROJECT.appspot.com",
  messagingSenderId: "PASTE_YOUR_SENDER_ID",
  appId: "PASTE_YOUR_APP_ID"
};
```

Replace each `PASTE_YOUR_...` value with the matching value from the config you copied in step 6 above. Save the file.

(If you'd rather not edit this by hand, send me the six values and I'll do it.)

---

## Part 3 — Host it on GitHub Pages

1. On [github.com](https://github.com), create a new **public** repository (e.g. `rhi-badminton-bracket`).
2. Upload `index.html` to the repo (drag-and-drop on the repo's page, or `git push`).
3. Go to the repo's **Settings → Pages**. Under "Build and deployment", set **Source** to **Deploy from a branch**, branch `main`, folder `/ (root)` → **Save**.
4. After a minute, GitHub shows your live URL, something like:
   `https://<your-username>.github.io/rhi-badminton-bracket/`
5. That single URL is what you share with everyone. It works the same on phones and desktops — no app install needed.

---

## How it works day-to-day

- **Everyone** opens the link → sees a read-only view that updates live as the admin records results (Firestore pushes changes instantly, no polling delay).
- **Admin(s)** tap "Admin sign-in" and enter the PIN (default `0821`, set in `index.html` — change the `ADMIN_PIN` constant before deploying if you want a different one). Once unlocked on a device, that device stays unlocked (stored locally on that phone/laptop only).
- Admin can: set the number of teams (up to 30) and courts, enter/paste player names, shuffle into doubles teams, record match winners and scores, and reassign any match to a different court (dropdown next to each match — useful if a court gets tied up).
- Anyone — admin or not — can use the **"Find your match"** search bar to type their name and instantly see which match they're in, the court, and the status.
- Round 1 and Round 2 guarantee everyone plays twice (odd team counts get one bye per round, marked clearly). Top teams by record then advance to a single-elimination playoff.

## Changing things later

- **Multiple events**: change `window.TOURNAMENT_ID` in `index.html` to a new value (e.g. `"rhi-nov-outing"`) to start a fresh bracket in the same Firebase project without touching old data.
- **Admin PIN**: edit the `ADMIN_PIN` constant, re-upload the file to GitHub.
- **Reset a tournament**: in the Firebase console, go to Firestore Database → the `tournaments` collection → delete the document for your `TOURNAMENT_ID`. The app will recreate it fresh next time someone saves.
