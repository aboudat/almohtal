# المحتال (Al Mohtal / The Imposter)

A pass-the-phone party game. One device, three to twenty players. Everyone sees the same secret
word except the imposter, who sees only "أنت المحتال" and has to bluff their way through the
questioning.

Built the same way as [Mob Rush](https://github.com/aboudat/mob-rush): a single vanilla
`index.html` with no build step, shipped as a PWA on Netlify and wrapped as a TWA for Google Play.

## How a round runs

0. **Start screen.** Logo, title, a Play button, and a "how to play" sheet with the five steps.
1. **Setup.** Type the player names, each with its own colour. Pick how many imposters, how long
   the discussion runs, and which word categories are in play. Everything is saved on the device
   for next time. A "how to play" button under the start button opens the same sheet as the start
   screen.
2. **The rules.** Every round opens on a full screen of the five steps, with a line saying how
   many imposters are hiding among how many players, and a Next button that deals and starts the
   pass-around. It sits on the way in from both start and play again, so a table with a newcomer
   never has to be talked through the game.
3. **Pass around.** The screen names one player at a time. That player taps the button carrying
   their own name, then taps the face-down card, which flips over in 3D to show their role. They
   tap "تم، أخفِ ومرر" and hand the phone on. Nobody can see the previous player's card, because
   the app returns to a name-only screen between players and the next card starts face down.
4. **Who starts.** The app picks a random player to ask the first question.
5. **Quiet.** The phone is put down. Players question each other and vote out loud. An optional
   countdown runs on screen. The app does nothing else.
6. **Reveal.** One tap plus a confirmation shows who the imposter was and what the word was.
   Then either deal a new round with the same players or go back and edit them.

## Game rules baked in

- The imposter gets **no hint at all** by default. A setting can give them the category, never the word.
- With two imposters, a setting lets them see each other, so they can play off one another.
- Two imposters unlock at seven players or more, and the deal always leaves at least two players
  who know the word.
- Word choice uses `crypto.getRandomValues` with modulo rejection, so the deal is uniform.
- Both Arabic (RTL) and English (LTR) ship in the app, with a parallel word bank of about 300
  words across 14 categories. The toggle is on the start screen and on the setup screen.
- Proper nouns are written in Arabic the way people actually say them, not literally translated,
  and the original name is shown underneath. So The Matrix is `ماتريكس` with "The Matrix" beneath
  it, never `المصفوفة`. Categories carrying `latin: true` opt into that.
- A word is not drawn again until 40 other words have been played, so a long night stops repeating.
- Player colours deliberately exclude crimson, so a face-down card never reads as the imposter card.

## Look and feel

Light and dark themes, plus auto which follows the system. Violet brand (`#6d3bf5`), crimson
reserved for the imposter and for destructive actions. Screens slide in and out with
direction-aware transitions, the role card is a real 3D flip, and everything collapses to instant
under `prefers-reduced-motion`.

Theme switching is deliberately atomic. `.screen` repaints its background instantly, so any
transition on inherited colour leaves dark text on a dark ground for a frame. A `.theming` class
suppresses every transition for one frame while the palette swaps.

Content is capped at `--maxw` and centred, so the game stays a readable column on a tablet or a
desktop browser rather than stretching edge to edge.

Three set pieces carry the personality. The splash draws the mask on as a stroke, fills it, and
raises the bilingual lockup with the `by Aboudat` credit; it auto-advances at 2.35s or on a tap.
Starting a round fans five cards out and cross-fades into the first player. The reveal is the
loudest moment: a red bloom over the screen, a jolt, and the culprits stamped in with their
avatars. All three collapse under `prefers-reduced-motion`.

Two traps worth remembering, both of which shipped invisible bugs before the tests caught them:

- `@keyframes rise` only declares a `from` state, so its implicit `to` state is the element's own
  style. Never also write `opacity: 0` on an element that uses it, or it animates 0 to 0.
- Resetting the role card for the next player must be instant. Removing `.flipped` normally
  animates the un-flip over 620ms, which shows the previous player's face on the way back. The
  `.noanim` class exists solely to suppress that.

## Files

| File | Purpose |
| --- | --- |
| `index.html` | The whole game: screens, logic, word bank, both languages |
| `sw.js` | Service worker, offline cache |
| `manifest.json` | PWA manifest, drives the TWA wrapper |
| `privacy.html` | Privacy policy, required by the Play listing |
| `make-icons.html` | Canvas icon generator, open it and click to download both PNGs |
| `icon-192.png`, `icon-512.png` | App icons, generated from `make-icons.html` |
| `.well-known/assetlinks.json` | Digital Asset Links. **Fingerprints are still empty, see below** |

## Running it locally

Any static server works. The service worker needs `http://`, so `file://` will not fully exercise
it.

```bash
npx serve .
# or
python -m http.server 8000
```

## Deploying

Netlify serves the live origin. Content-only changes ship by redeploying, with no Play release
needed, **but bump the `CACHE` name in `sw.js` when you do** or players keep the old cached copy
and never see the change.

GitHub Pages serves a test origin from `main` at https://aboudat.github.io/almohtal/, which is how
the game gets onto an iPhone: open it in Safari, Share, Add to Home Screen. That is the only
practical route on iOS, since Apple has no sideloadable equivalent of an APK. Every push to `main`
redeploys it, and `.nojekyll` keeps Pages from hiding `.well-known/`.

## Android test builds

For trying the game on a real phone there is a Capacitor wrapper at `../almohtal-android`, kept
out of this repo the same way the TWA project is. It bundles the web files **inside** the APK, so
it needs no hosting and works with the phone offline. It is a test artifact, not the ship path.

```bash
cd ../almohtal-android
npm run apk          # stages www/, syncs, and runs gradlew assembleDebug
node serve-apk.js    # serves the apk over the LAN so the phone can download it
```

Notes on that wrapper:

- Application id is `com.almohtal.test`, deliberately different from the TWA id so a test build
  and a future Play install never collide on the same device.
- `sync-web.js` copies the shipping files but **skips `sw.js`**. Inside the APK the assets are
  already local, and the service worker's cache-first handler would keep serving the previous
  build after installing an updated APK.
- The toolchain lives in `C:\Users\tat_4\.androidtools` (Temurin JDK 21 plus the Android SDK).
  Gradle needs `JAVA_HOME` and `ANDROID_HOME` pointed there, and `android/local.properties`
  already records the SDK path.

## Packaging for Google Play

The Android wrapper project is **not** kept in this repo, same as Mob Rush. Regenerate it when a
store update is needed:

```bash
npm i -g @bubblewrap/cli
bubblewrap init --manifest https://<your-netlify-site>.netlify.app/manifest.json
bubblewrap build
```

Bubblewrap itself is not installed, but the JDK and Android SDK it needs already are, at
`C:\Users\tat_4\.androidtools`.

Two things to finish before the first release:

1. **`assetlinks.json` fingerprints are empty.** After the first build, run
   `bubblewrap fingerprint list` and paste in the SHA-256 of the upload key. Once the app is in
   Play App Signing, add the Play signing key fingerprint too, so the file carries both. Without
   this the TWA shows a browser URL bar instead of running full screen.
2. **Confirm the package name.** `.well-known/assetlinks.json` currently assumes
   `app.netlify.almohtal.twa`, which follows the Mob Rush convention. It has to match whatever
   Bubblewrap is actually configured with.

For every later release, bump `versionCode` past whatever is already live on Play.
