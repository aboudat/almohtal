# المحتال (Al Mohtal / The Imposter)

A pass-the-phone party game. One device, three to twelve players. Everyone sees the same secret
word except the imposter, who sees only "أنت المحتال" and has to bluff their way through the
questioning.

Built the same way as [Mob Rush](https://github.com/aboudat/mob-rush): a single vanilla
`index.html` with no build step, shipped as a PWA on Netlify and wrapped as a TWA for Google Play.

## How a round runs

0. **Start screen.** Logo, title, a Play button, and a "how to play" sheet with the five steps.
1. **Setup.** Type the player names, each with its own colour. Pick how many imposters, how long
   the discussion runs, and which word categories are in play. Everything is saved on the device
   for next time.
2. **Pass around.** The screen names one player at a time. That player taps the button carrying
   their own name, then taps the face-down card, which flips over in 3D to show their role. They
   tap "تم، أخفِ ومرر" and hand the phone on. Nobody can see the previous player's card, because
   the app returns to a name-only screen between players and the next card starts face down.
3. **Who starts.** The app picks a random player to ask the first question.
4. **Quiet.** The phone is put down. Players question each other and vote out loud. An optional
   countdown runs on screen. The app does nothing else.
5. **Reveal.** One tap plus a confirmation shows who the imposter was and what the word was.
   Then either deal a new round with the same players or go back and edit them.

## Game rules baked in

- The imposter gets **no hint at all**, not the category and not a related word.
- Two imposters unlock at seven players or more, and the deal always leaves at least two players
  who know the word.
- Word choice uses `crypto.getRandomValues` with modulo rejection, so the deal is uniform.
- Both Arabic (RTL) and English (LTR) ship in the app, with a parallel word bank of about 290
  words across 14 categories. The toggle is on the start screen and on the setup screen.
- Player colours deliberately exclude crimson, so a face-down card never reads as the imposter card.

## Look and feel

Light theme on `#f4f5fa`, violet brand (`#6d3bf5`), crimson reserved for the imposter and for
destructive actions. Screens slide in and out with direction-aware transitions, the role card is a
real 3D flip, and everything collapses to instant under `prefers-reduced-motion`.

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

## Packaging for Google Play

The Android wrapper project is **not** kept in this repo, same as Mob Rush. Regenerate it when a
store update is needed:

```bash
npm i -g @bubblewrap/cli
bubblewrap init --manifest https://<your-netlify-site>.netlify.app/manifest.json
bubblewrap build
```

Bubblewrap needs a JDK and the Android SDK, neither of which is installed on this machine yet.

Two things to finish before the first release:

1. **`assetlinks.json` fingerprints are empty.** After the first build, run
   `bubblewrap fingerprint list` and paste in the SHA-256 of the upload key. Once the app is in
   Play App Signing, add the Play signing key fingerprint too, so the file carries both. Without
   this the TWA shows a browser URL bar instead of running full screen.
2. **Confirm the package name.** `.well-known/assetlinks.json` currently assumes
   `app.netlify.almohtal.twa`, which follows the Mob Rush convention. It has to match whatever
   Bubblewrap is actually configured with.

For every later release, bump `versionCode` past whatever is already live on Play.
