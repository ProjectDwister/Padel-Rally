# Rally — Padel Tournament Scorer

A single-page app for running a padel **Americano** or **Mexicano** tournament: build a roster, generate the draw, keep score court-by-court, and track a live leaderboard.

This is a plain static site — one `index.html` file, no build step, no dependencies to install. It loads React and Babel from a CDN and compiles the JSX right in the browser.

## Run it locally

Just open `index.html` in a browser (double-click it, or `open index.html`). No server needed.

## Deploy to GitHub Pages

1. Create a new GitHub repo and add `index.html` to it (drag-and-drop on the repo's "Add file → Upload files" page works fine).
2. Commit it to the `main` branch.
3. Go to **Settings → Pages**.
4. Under "Build and deployment", set **Source: Deploy from a branch**, **Branch: main**, folder **/ (root)**.
5. Save. GitHub will give you a URL like `https://yourusername.github.io/your-repo-name/` within a minute or two.

## About saved data

The tournament (players, draw, scores) is saved to **`localStorage`** in whatever browser you have it open in — it survives refreshes and closing the tab, but it's local to that one browser/device. Opening the page on a second phone starts a separate, independent tournament rather than showing the same live one.

If you want the same tournament to update live across multiple courtside phones, that needs a real shared backend (e.g. Firebase or Supabase) in place of `localStorage` — happy to wire that up if it'd be useful, just ask.

## Editing

All the app code lives in the `<script type="text/babel">` block in `index.html`. It's plain React (hooks + a reducer for state), no JSX build step required — edit and refresh.
