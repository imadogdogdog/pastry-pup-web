# Pastry Pup AI handoff

Last updated: 2026-08-06

This file is the starting point for any AI or developer taking over the Pastry Pup website.

## Non-negotiable project constraints

- Keep the existing folder structure and URLs unchanged. Do not move the arcade pages, assets, or root HTML files into a framework-specific layout.
- Preserve the visual identity: playful bakery/puppy styling, pink and purple palette, rounded cards, bold friendly type, and mobile-first controls.
- The site must remain deployable as a static GitHub Pages site.
- Checkout uses Square for payment. Never put a Square access token or other private credential in browser code.
- Firebase Cloud Functions are not required for the current checkout and should not be reintroduced merely to accept payment.
- Do not commit secrets. The Firebase client configuration in `assets/js/app.js` is a public web-app identifier; privileged credentials are not stored in this repository.

## Product overview

Pastry Pup is a static bakery storefront and mini-game arcade. The storefront displays a live Firestore-backed menu, provides a staff-only menu editor, keeps a cart in the browser, and creates a pre-filled email order request. The arcade contains several games and stores per-player best scores in Firestore.

Treat Tap Revolution now includes:

- Three sequential music levels: Dungeon Run, Better Days, and Glitch Beat.
- Real music streamed from OpenGameArt, with attribution in the game screen.
- Regular notes, hold notes, and linked multi-finger chord notes.
- Easy, Normal, and Hard modes with increasing lane and chord complexity.
- Keyboard and touch controls, level progression, combo tracking, lives, and saved best scores.

## Architecture

There is no package manager, build step, server runtime, or framework CLI. GitHub Pages serves the files directly.

- `index.html` is the storefront entry page.
- `arcade/index.html` is the arcade menu.
- `arcade/<game>/index.html` is a thin entry page for each game. Each page sets a `data-page` value on `<body>`.
- `assets/js/app.js` is the shared React application. It reads `document.body.dataset.page` and renders the corresponding storefront or game view.
- `assets/css/site.css` contains shared CSS.
- React 18, Firebase 10, Tailwind's CDN runtime, and Babel Standalone are loaded from CDN import maps/scripts in the HTML files.
- `firestore.rules` defines menu and score access rules.
- `firebase.json` only points Firebase CLI at the Firestore rules. It intentionally contains no Functions configuration.
- `CNAME` maps GitHub Pages to `pastrypup.shop`.

Current folders and important files:

```text
Pastry Pup/
|-- index.html
|-- arcade.html                     # legacy/root arcade entry; preserve it
|-- interactives.html               # legacy entry; preserve it
|-- disney_persona.html             # legacy entry; preserve it
|-- arcade/
|   |-- index.html
|   |-- catching-treats/index.html
|   |-- disney-personality/index.html
|   |-- snake-battle/index.html
|   |-- treat-tap-revolution/index.html
|   |-- watermelon-jump/index.html
|   `-- wings-of-fire-quiz/index.html
|-- assets/
|   |-- css/site.css
|   |-- js/app.js
|   `-- images/latest-bakes/
|-- firebase.json
|-- firestore.rules
|-- CNAME
|-- README.md
`-- AI_HANDOFF.md
```

Do not replace this with Vite, Next.js, Create React App, npm-generated folders, or a different routing scheme unless the owner explicitly authorizes a structural migration.

## Storefront and checkout

The cart is stored under `pastryPupCart:v3` in `localStorage`. Legacy v1/v2 cart keys are migrated and removed by `assets/js/app.js`.

Checkout is deliberately a two-step Square invoice flow:

1. The customer builds a cart.
2. The site calculates the CAD total from the menu's price tiers.
3. **Email order for Square invoice** opens a pre-filled email to `lucy.the.headchef@gmail.com` with item quantities, pricing, total, and customer placeholders.
4. Pastry Pup confirms availability and sends the customer a secure Square invoice/payment link.
5. The customer pays in Square.

The **Copy order details** button is the fallback when a browser cannot open a mail client.

This design avoids Firebase Functions and a Firebase Blaze upgrade. Firebase is still used for the live menu, staff authentication, anonymous player identity, and leaderboards. A future one-click Square checkout requires a secure server-side endpoint or a stable public Square payment link; it must not expose Square credentials in `app.js`.

## Firebase data model and access

Firebase project: `pastry-pup`

Application namespace: `artifacts/pastry-pup-final/public/data`

Important collections:

- `menuItems/{itemId}`: publicly readable; only the configured staff Firebase user can create, update, or delete.
- `arcadeScores/{gameId}/players/{scoreId}`: publicly readable; authenticated anonymous users can create/update only their own player scores, and scores may only increase.
- `watermelonScores/{scoreId}`: legacy Watermelon Jump scores with equivalent ownership rules.

The staff identity check is enforced in both the app and `firestore.rules`. If the staff Firebase account changes, update the UID in both places and deploy the rules. Do not weaken the rules to make the editor work.

Deploy Firestore rules separately when they change:

```powershell
firebase deploy --only firestore:rules
```

This Firebase deploy is unrelated to publishing the static website and does not require Cloud Functions.

## Editing the shared app

`assets/js/app.js` is intentionally large because every entry page uses it. Before editing:

1. Find the relevant component or helper with `rg` instead of scanning the whole file.
2. Preserve the `data-page` routing contract and absolute site routes.
3. If `app.js` changes, update its cache-busting query string in every HTML entry page that loads it. Search with:

   ```powershell
   rg -n 'assets/js/app.js\?v=' -g '*.html'
   ```

4. Keep Firebase import maps aligned with imports used by `app.js`. Firebase Functions are not currently imported by the application.
5. Test the storefront and every changed game on desktop and a narrow mobile viewport.

## Local development and verification

Run a static server from the repository root. Python is one simple option:

```powershell
python -m http.server 4173
```

Then open `http://127.0.0.1:4173/`.

Minimum storefront checks:

- The page loads without a blank screen or uncaught console error.
- Firestore menu items appear.
- Adding, increasing, decreasing, and removing cart items works.
- Bundle prices and totals are correct.
- The Square invoice email contains every cart item, quantity, price, total, and the request for a secure Square invoice/payment link.
- **Copy order details** produces one visible success message.
- The cart drawer has no horizontal overflow at approximately 390 px wide.
- No customer-facing message asks for a Firebase upgrade.

Minimum Treat Tap Revolution checks:

- `/arcade/treat-tap-revolution/` loads.
- Starting a difficulty begins level 1 music after the user gesture.
- Tap, hold, chord, keyboard, and multi-touch input paths work.
- Completing a song advances to the next named song.
- Music failure produces a readable retry message instead of hanging.
- Exiting or unmounting stops audio.
- A signed-in player can save a score without breaking unauthenticated play.

Expected development warnings: the current CDN setup may warn that Tailwind's CDN and Babel's in-browser transformer are not ideal for production. They do not prevent the static site from working. Removing them would be a separately approved structural/performance project.

## Deployment

Repository: `imadogdogdog/pastry-pup-web`

Production branch: `main`

Public site: <https://pastrypup.shop>

Pushing `main` triggers GitHub's **pages build and deployment** workflow. Verify both the workflow and the actual public asset before declaring deployment complete:

1. Confirm the Pages workflow for the new commit succeeds in GitHub Actions.
2. Open the public site with a cache-busting query string.
3. Inspect the loaded `assets/js/app.js?v=...` URL and confirm it matches the version in the committed HTML.
4. Exercise the changed interaction on the public domain.

On 2026-08-06, the first Pages run for the Square redesign commit was cancelled after waiting 15 minutes without receiving a GitHub-hosted runner. It had no build steps or code error. An empty commit was pushed to trigger a fresh run. If this repeats, re-run the Pages workflow from GitHub Actions or push a harmless follow-up commit; do not change Firebase billing.

## Current implementation state

- Square invoice checkout redesign is implemented in commit `a965dd4`.
- The invoice flow was locally verified on desktop and mobile.
- Treat Tap Revolution has three songs, holds, and multi-finger chords.
- The first GitHub Pages deployment attempt for `a965dd4` was cancelled before runner assignment; commit `fb9707d` retriggered deployment.
- Three unrelated local files may appear as untracked (`Natural_Peanut_Butter_Cookies_Toner_Friendly.docx`, `Pastry_Pup_Chewy_Chocolate_Chip_Cookies_Low_Toner.docx`, and `logo_alpha.png`). They belong to the owner. Do not modify, add, delete, or commit them unless explicitly requested.

## Suggested first prompt for the next AI

> Read `AI_HANDOFF.md` and `README.md` completely before making changes. Preserve the current folder structure and static GitHub Pages architecture. Inspect the working tree without touching unrelated untracked files. Continue from the current deployment state, verify the latest GitHub Pages workflow, and test the public site before reporting completion.

