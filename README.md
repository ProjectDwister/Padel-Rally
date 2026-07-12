# Rally — Padel Tournament Scorer

An app for running a padel **Americano** or **Mexicano** tournament: build a roster, generate the draw, keep score court-by-court, and track a live leaderboard. Installable on Android as a real app icon on the home screen.

This is a static site with no build step — it loads React and Babel from a CDN and compiles the app's JSX right in the browser. No dependencies to install, no npm, no bundler.

## Files

```
index.html        the whole app
manifest.json     makes it installable as an app
sw.js             lets it keep working with a flaky courtside connection
icons/            app icons at the sizes Android/iOS/desktop expect
```

All five things need to be uploaded together, keeping this same folder structure — `manifest.json` and `sw.js` both reference the icons by relative path.

## Run it locally

Open `index.html` in a browser. Note: the "install as an app" and offline-caching behavior (the manifest and service worker) only activate once this is actually deployed to a real HTTPS URL — browsers require that for security reasons, so those two pieces won't do anything when just opened as a local file. Everything else works fine locally.

## Deploy to GitHub Pages

1. Create a new GitHub repo.
2. Add all five items above (`index.html`, `manifest.json`, `sw.js`, and the `icons/` folder with its contents), preserving the folder structure. On the repo's "Add file → Upload files" page, you can drag the whole `icons` folder in along with the other files and GitHub will preserve the structure — or use `git add . && git commit && git push` from the command line if you're comfortable with git, which is a bit more reliable for folders.
3. Commit to the `main` branch.
4. Go to **Settings → Pages**.
5. Under "Build and deployment", set **Source: Deploy from a branch**, **Branch: main**, folder **/ (root)**.
6. Save. GitHub will give you a URL like `https://yourusername.github.io/your-repo-name/` within a minute or two.

## Installing it as an app

**On Android (Chrome):** open the GitHub Pages link, then **⋮ menu → Add to Home screen → Install**. It'll get a real icon, launch full-screen with no browser bar, and show up in your app drawer like any other app. This is a PWA (Progressive Web App) — genuinely installable, works offline after the first load, no app store needed.

**Turning it into an actual installable `.apk` (or a Google Play listing):** once it's live on GitHub Pages, go to **[pwabuilder.com](https://www.pwabuilder.com)**, paste in your GitHub Pages URL, and it'll package this into a real Android app package for you — it builds in Microsoft's cloud, so neither of us needs Android Studio or the Android SDK installed anywhere. It uses Google's own recommended approach (Trusted Web Activity) for wrapping a PWA as an Android app, not a hacky WebView clone. You'll get:
- an `.apk` you can install directly on a phone for testing (Settings → allow installs from this source, then open the file)
- an `.aab` file, which is what you'd upload if you ever wanted to publish it to the Google Play Store

Nothing about the app itself needs to change for this step — it's just packaging the already-deployed site.

## Live sync across courtside phones (optional, ~5 minutes)

Out of the box, Rally saves to `localStorage` — it survives refreshes, but each browser/device has its own separate tournament. To make one tournament update live across every phone at the courts, connect a free Firebase Realtime Database:

1. Go to **[console.firebase.google.com](https://console.firebase.google.com)** and create a project (any name is fine — free tier, no credit card needed).
2. In the left sidebar: **Build → Realtime Database → Create Database**. Pick any region, and choose **"Start in test mode"** for now (we'll replace the rules in the next step, since test mode alone expires after 30 days).
3. Once created, open the **Rules** tab and replace the contents with:
   ```json
   {
     "rules": {
       ".read": true,
       ".write": true
     }
   }
   ```
   Click **Publish**. This keeps the board open to anyone with the link — same model as the rest of the app, and simplest for a casual tournament. (If you'd rather lock it down, Firebase's rules can require auth or a shared secret — ask if you want a hand with that.)
4. Back in the project's **Project Overview**, click the **`</>`** (web) icon to register a new web app. Skip Firebase Hosting when it's offered — you're using GitHub Pages. This gives you a `firebaseConfig` object with your `apiKey`, `databaseURL`, etc.
5. Open `index.html`, find `FIREBASE_CONFIG` near the top of the `<script type="text/babel">` block, and paste your values in:
   ```js
   const FIREBASE_CONFIG = {
     apiKey: "your-value-here",
     authDomain: "your-value-here",
     databaseURL: "your-value-here",
     projectId: "your-value-here",
     storageBucket: "your-value-here",
     messagingSenderId: "your-value-here",
     appId: "your-value-here",
   };
   ```
6. Save, and re-deploy (commit the change if it's already on GitHub Pages). Open the page — the small dot next to the round counter should read **"Live"** once it connects.

Every phone that opens the link after this is looking at the same tournament, live. Leave `FIREBASE_CONFIG` blank and it just quietly falls back to `localStorage` — nothing else to configure either way.

**How concurrent edits are handled:** scores are synced field-by-field (not as one big blob), so two phones entering scores on two different courts at the same moment never overwrite each other. Structural changes (roster, draw regeneration, starting a new tournament) sync as a whole and use simple last-write-wins — in the unlikely case two people do that at the exact same instant, whichever save lands last wins; just redo the other change if that happens.

## Editing

Most of the app code lives in the `<script type="text/babel">` block in `index.html`. It's plain React (hooks + a reducer for state), no JSX build step required — edit and refresh. If you change the app's colors or name, update `manifest.json` and the icons in `icons/` to match, and bump `CACHE_NAME` in `sw.js` (e.g. `rally-cache-v2`) so returning visitors actually pick up the change instead of getting served their old cached copy.

## Troubleshooting: blank page

This page loads React, Babel, and (optionally) Firebase from a CDN at runtime rather than being pre-built, so it needs a real internet connection and an unrestricted browser tab to render. If you see nothing:

- **Viewing it inside another app's embedded preview** (rather than a full browser tab) can block those outside CDN requests. Download the file and open it directly, or view it once it's deployed on GitHub Pages.
- Otherwise, the page itself will now show a visible error box instead of staying silently blank — if you see one, it'll say exactly which script failed to load or what broke, which is the fastest way to pin down what's going on.
