# RHI Badminton Bracket — Setup Guide

This turns `index.html` into a live, mobile-friendly page anyone can open — with
one admin (password-protected, real server-side auth) editing scores/courts,
and everyone else seeing a read-only view that updates instantly, no refresh
needed.

Two things to set up, both free: **Firebase** (holds the live tournament data
and the admin login) and **GitHub Pages** (hosts the page itself).

---

## Part 1 — Firebase (real-time database + admin login)

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
         allow write: if request.auth != null;
       }
     }
   }
   ```

   Click **Publish** to save the rules. Reads stay open to everyone (that's the
   read-only view participants see); writes now require a real signed-in
   session — this check happens on Firebase's servers, so it can't be
   bypassed from the browser, unlike a client-side PIN.
5. Go to **Build → Authentication** → **Get started** → choose **Email/Password**
   → toggle it **Enable** → **Save**.
6. Go to the **Users** tab → **Add user** → enter any email-shaped identifier
   (it never needs to receive real mail, e.g. `admin@yourevent.app`) and a
   strong password. This is the one admin login for the tournament — anyone
   who needs edit access uses this same email/password.
7. Go to **Project settings** (gear icon, top left) → scroll to **Your apps** → click the **`</>`** (Web) icon → give it a nickname (e.g. `bracket-page`) → **Register app**. Don't bother with Firebase Hosting when prompted — you're using GitHub Pages instead.
8. Copy the `firebaseConfig` object shown — you'll need it in Part 2.

That's it for Firebase — the free "Spark" plan covers this easily (way under the daily free quota for an event this size).

---

## Part 2 — Wire the config into `index.html`

Open `index.html`, find the `window.FIREBASE_CONFIG` block near the top, and replace each value with the matching value from the config you copied in Part 1. Also update the `ADMIN_EMAIL` constant a bit further down to match the email you created in step 6 above (this value isn't secret — it's just an identifier, the same way the Firebase config itself is public). Save the file.

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
- **Admin(s)** tap "Admin sign-in" and enter the password set up in Part 1. This is a real login checked by Firebase's servers, not a client-side PIN — it can't be bypassed via browser dev tools or by editing local storage. Firebase keeps the session signed in across reloads on its own.
- Admin can: set the number of teams (up to 30) and courts, enter/paste player names (or enter pre-decided doubles pairs directly), shuffle into doubles teams, record match winners and scores (auto-derived from entered points — no separate manual winner click needed), and reassign any match to a different court.
- Anyone — admin or not — can use the **"Find your match"** search bar to type their name and instantly see which match they're in, the court, and the status, and can tap any team in Standings for a round-by-round explanation of why they advanced (or didn't).
- Round 1 and Round 2 guarantee everyone plays twice (odd team counts get one bye per round, marked clearly). Top teams by record (wins, then point differential) then advance to a single-elimination playoff.

## Changing things later

- **Multiple events**: change `window.TOURNAMENT_ID` in `index.html` to a new value (e.g. `"rhi-nov-outing"`) to start a fresh bracket in the same Firebase project without touching old data. The same admin login works across all events in the same project — no need to create a new one each time.
- **Admin password**: change it anytime in Firebase console → Authentication → Users (click the user → reset password). No code change or redeploy needed.
- **Reset a tournament**: in the Firebase console, go to Firestore Database → the `tournaments` collection → delete the document for your `TOURNAMENT_ID`. The app will recreate it fresh next time someone saves.
