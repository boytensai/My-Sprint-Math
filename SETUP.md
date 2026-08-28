# Sprint Math — hosting & monetization setup

This game is a single static file (`index.html`). Everything below is optional
and additive — the game fully works with zero setup, just with a local-only
leaderboard (each player's best scores are saved in their own browser only).

## 1. Host it on GitHub Pages (free)

1. Create a GitHub account if you don't have one: https://github.com/signup
2. Create a new repository (e.g. `sprint-math`), public, no README/gitignore
   (this folder already has them).
3. From this folder, push it up:
   ```bash
   git remote add origin https://github.com/YOUR_USERNAME/sprint-math.git
   git branch -M main
   git push -u origin main
   ```
4. On GitHub: repo → **Settings** → **Pages** → under "Build and deployment",
   set Source to **Deploy from a branch**, branch `main`, folder `/ (root)`.
5. Wait a minute, then your game is live at:
   `https://YOUR_USERNAME.github.io/sprint-math/`

You can point a custom domain at it later from that same Pages settings page.

## 2. Shared leaderboard (Firebase, free tier)

Without this, `index.html` still works — the leaderboard just stays local to
each device. To make it shared across everyone:

1. Go to https://console.firebase.google.com and create a free project.
2. In the project, click **Build → Firestore Database → Create database**.
   Start in **production mode** (we'll set rules below).
3. Click the gear icon → **Project settings** → scroll to "Your apps" →
   click the **</> (web)** icon → register an app (no hosting needed).
4. Firebase shows you a config object like:
   ```js
   const firebaseConfig = {
     apiKey: "AIza...",
     authDomain: "sprint-math-xxxxx.firebaseapp.com",
     projectId: "sprint-math-xxxxx",
     storageBucket: "sprint-math-xxxxx.appspot.com",
     messagingSenderId: "...",
     appId: "..."
   };
   ```
   Copy those six values into `FIREBASE_CONFIG` near the top of the
   `<script type="module">` block in `index.html`.

   Note: this config is meant to be public/client-side — it's not a secret
   key. Firestore security rules (next step) are what actually control
   access, not hiding this config.

5. In Firestore → **Rules**, replace the default rules with:
   ```
   rules_version = '2';
   service cloud.firestore {
     match /databases/{database}/documents {
       match /scores/{scoreId} {
         allow read: if true;
         allow create: if request.resource.data.keys().hasOnly(['name','score','ts','difficulty'])
                       && request.resource.data.name is string
                       && request.resource.data.name.size() <= 20
                       && request.resource.data.score is int
                       && request.resource.data.score >= 0
                       && request.resource.data.score <= 200
                       && request.resource.data.difficulty in ['easy','medium','hard'];
         allow update, delete: if false;
       }
     }
   }
   ```
   This lets anyone submit a valid score and read the board, but nobody can
   edit or delete existing scores from the client.
6. The first time the leaderboard query runs, Firestore may show an error
   in the browser console with a link to create a required index — click
   it, accept the defaults, wait ~1 minute. This only happens once per
   difficulty.
7. Commit and push the updated `index.html` (with your config filled in).

Firebase's free "Spark" tier covers far more reads/writes than a small game
like this will ever use.

## 3. Tip jar (Ko-fi)

1. Create a free account at https://ko-fi.com and pick a username.
2. In `index.html`, find `var KOFI_USERNAME = "yourusername";` near the
   bottom of the file and replace it with your real username.
3. A small floating "Support" button will appear on the page.

## 4. Ads (Google AdSense)

Realistic expectations first: AdSense requires an approved account, a
reasonable amount of original content, and it only earns meaningful money
with real, sustained traffic — a few visitors a day won't produce anything
worth mentioning. A tip jar or a paid unlock is usually more realistic for a
small game like this.

If you still want to add it:
1. Apply at https://www.google.com/adsense — you'll need the site live on
   its own domain first (GitHub Pages URL is fine to start).
2. Once approved, you'll get a publisher ID like `ca-pub-1234567890123456`.
3. In `index.html`'s `<head>`, uncomment the AdSense `<script>` tag and
   replace `ca-pub-XXXXXXXXXXXXXXXX` with your real ID.
4. Replace the `<div class="ad-slot">` comment block with the ad unit code
   AdSense gives you for that unit.

## 5. Other monetization worth considering

- **One-time unlock**: keep Easy/Medium free, sell a "Pro" pack (Hard mode,
  extra themes, an untimed practice mode) via a Stripe Payment Link or
  Gumroad — no backend required, just a link.
- **Sponsorship**: a tutoring service or school could pay a flat fee to
  have their name on the page — more realistic income than ads at small
  scale.

None of these require touching the game's core code — they're additions
layered on top whenever you're ready.
