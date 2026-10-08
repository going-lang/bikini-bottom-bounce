# Bikini Bottom Bounce

A Doodle Jump style vertical platform jumper themed on SpongeBob SquarePants.
Built for Android-first touch play, and it runs in any modern mobile browser.

## Run it

The browser version has no build step. It is plain ES modules and needs any static file server:

```bash
cd spongebob-jump
python3 -m http.server 8080 --bind 0.0.0.0
# open http://localhost:8080 (or the device's LAN IP on port 8080)
```

To run the headless generator check:

```bash
node tools/test_generator.mjs
```

It runs an autopilot over 40 generated levels up to 1500 m and fails if the
autopilot falls. The current run passes.

## Controls

- **Bounce:** automatic. SpongeBob bounces on every platform he lands on.
- **Move:** hold the left or right half of the screen. Tap-and-hold works with
  a mouse on desktop. Keyboard arrows or A/D also work.
- **Swap left/right:** Settings → Swap left/right.
- **Pause:** the pause button in the HUD.

## Gameplay

- **Platforms:** normal, small, large, moving, bounce, breakable, and special
  (jelly). Every generated row is reachable. The generator caps vertical gaps
  at 172 px against a 250 px bounce apex, and caps horizontal offsets.
- **Hits:** there is no health bar. Each hit costs half of the coins you collected
  in the current run (rounded up). Coins saved before the run are never lost. When
  you have no run coins left, the next hit ends the run. `HIT_COIN_LOSS` in
  `js/config.js` sets the share.
- **Collectibles:** coins (currency), bubbles, jellyfish tokens (magnet fuel),
  golden spatulas (premium), and treasure chests.
- **Power-ups:** Super Bounce, Bubble Shield, Jellyfish Magnet, Speed Boost,
  Double Coins, Coin Guard, Slow Motion, and Mega Bounce. Each shows a duration
  indicator in the HUD.
- **Hazards:** normal, fast, and large jellyfish; sea urchins; moving sea mines;
  dangerous bubbles; and falling rocks. Hazards are placed away from the
  landing path, and difficulty ramps with height.
- **Zones:** Bikini Bottom, Jellyfish Fields, Goo Lagoon, Krusty Krab Area, and
  Rock Bottom. Zones are defined in `js/config.js` (`ZONES`), so new zones can
  be added there.
- **Revive:** once per run. Either watch the rewarded-ad stub or pay 100 coins
  (`REVIVE_COST_COINS`). No ads are forced.

## Power-ups

All eight power-ups use artwork cut from the supplied power-up reference sheet
(`assets/reference/powerups_reference.jpg`). Each icon is a separate transparent
PNG in `assets/powerups/`. Regenerate them with `python3 tools/extract_powerups.py`.

| Power-up       | Effect                                                   | Duration |
|----------------|----------------------------------------------------------|----------|
| Super Bounce   | Stronger bounces, with an immediate launch on pickup     | 8 s      |
| Bubble Shield  | Absorbs one enemy or hazard hit, then is removed         | 15 s     |
| Jellyfish Magnet | Pulls in coins, bubbles and jellyfish tokens           | 8 s      |
| Speed Boost    | Faster sideways movement, with a green trail             | 6 s      |
| Double Coins   | Coins worth 2x, with a 2X badge over SpongeBob           | 10 s     |
| Coin Guard     | Your next hit costs no coins                             | instant  |
| Slow Motion    | Enemies, hazards and moving platforms slow; player does not | 6 s   |
| Mega Bounce    | Next landing launches very high                          | until used |

Power-ups are rare and spawn at most one per platform row. Weights are set per
power-up in `js/config.js` (`POWERUPS`). The HUD shows an active power-up list
with each icon, name and remaining time.

## Scoring and progression

- **Score** = height points + item points. Height points are worth more in higher
  zones (1x in Bikini Bottom, up to 2x in Rock Bottom), so climbing higher is
  rewarded more.
- **Coins** come from coins, bonus items and jelly pops. Double Coins doubles them.
- **Bubbles** add points and a bubble count (used by the "Pop 10 bubbles" mission).
- **Jellyfish tokens** add coins and a token count. Lifetime tokens are saved and
  count toward the "Collect 50 jellyfish tokens" mission.
- **Golden spatulas** are premium currency. **Treasure chests** give large coin and
  premium rewards.

## Characters

| Character  | Rarity  | Price          | Ability                                 |
|------------|---------|----------------|-----------------------------------------|
| SpongeBob  | Common  | Owned          | None                                    |
| Patrick    | Rare    | 1500 coins     | Slightly stronger bounce (×1.07)        |
| Sandy      | Rare    | 1800 coins     | Better horizontal control               |
| Squidward  | Epic    | 20 premium     | Occasional extra bounce (12% chance)    |
| Mr. Krabs  | Epic    | 25 premium     | +10% coin value                         |
| Gary       | Common  | 800 coins      | Slightly larger magnet range (×1.15)    |

No ability raises the score or height ceiling directly, so none of them are
pay-to-win.

Only SpongeBob has supplied art. The other five are **locked placeholder
cards** (`art: false` in `js/config.js`). No character designs were invented.
Add the official art to `assets/sprites/`, set `art: true`, and the card will
show it.

## Currencies and progression

- **Coins:** earned in play, spent in the shop.
- **Premium (golden spatulas):** earned in play. No real-money purchases exist,
  and none are needed for core progression.
- Progress is saved to `localStorage` under `bikinibottom-bounce.save`
  (version 1). It stores high score, best height, total coins, unlocks,
  equipped items, best zone, power-up stats, achievements, and missions.
  Restarting a run keeps all progress. Only Settings → Reset progress wipes it,
  and that asks for confirmation first.

## Project layout

```
index.html              entry point and screen markup
css/style.css           all UI styling
js/
  config.js             tuning constants, zones, platforms, characters, items,
                        missions, achievements
  main.js               boot, screen wiring, shop/missions/settings, RAF loop
  game.js               run state, update loop, render, revive
  generator.js          procedural row generation (reachability rules)
  physics.js            player motion, landing, collision helpers
  fx.js                 particle and floating-text pools
  audio.js              procedural WebAudio music and SFX
  assets.js             sprite and background loader with timeouts
  input.js              touch/mouse/keyboard steering
  storage.js            versioned save with validation and merge defaults
  ui.js                 HUD, menus, shop, modals, toasts
assets/
  sprites/              one PNG per sprite, plus pose_meta.json
  backgrounds/          one JPG per zone
tools/
  extract_sprites.py    sprite extraction from the reference sheets
  pose_meta.py          measures pose bounds for scaling
  extract_powerups.py   power-up icon extraction from the reference sheet
  test_generator.mjs    headless reachability test
```

## Assets and audio

- **Sprites:** every sprite, platform, enemy, collectible, and power-up is a
  separate PNG. Nothing is combined into a sheet at runtime.
- **Character and object art:** taken from the supplied reference sheets.
  SpongeBob is the only character with art.
- **Backgrounds:** `zone1_bikini_bottom.jpg` comes from the supplied zone 1
  reference. **Zones 2–5 backgrounds were not part of the supplied references.
  Confirm their origin and licence before release, or replace them with
  approved art.**
- **Audio:** all music and sound effects are synthesized procedurally with
  WebAudio. No third-party audio files are included.

## Performance notes

- Platforms, coins, enemies, particles, and power-ups use object pools.
- Off-screen objects are culled every frame.
- There is one `requestAnimationFrame` loop. Restarting a run does not add
  loops or listeners.
- Particle count is capped (240) and floating text is capped (14).

## Android app (Capacitor)

The game runs as a native Android app through Capacitor. The web game in
`index.html` is unchanged; Capacitor packages a copy of it.

Requirements: Node 22 or newer, JDK 21, and the Android SDK (platform 36,
build-tools 36.0.0). `local.properties` in `android/` points at the SDK.

```bash
npm install
npm run android:debug       # copies the game to www/, syncs, builds app-debug.apk
npm run android:open        # open the project in Android Studio
```

The APK is at `android/app/build/outputs/apk/debug/app-debug.apk`.
`npm run build:www` only refreshes `www/`. Run `npm run cap:sync` after editing
game files if you open the project in Android Studio. `www/` is generated, so
edit the source files in `index.html`, `css/`, `js/` and `assets/`.

App identity lives in `capacitor.config.json` (`appId`, `appName`) and
`android/app/build.gradle`. The app ID `com.wrldgames.bikinibottombounce` is a
placeholder. Change it before publishing, because it cannot be changed after
the app is published.

Release builds: the signed release APK is built by the GitHub Actions release job
(see "Release build (Uptodown)" below). `npm run android:release` only builds an
unsigned AAB locally.

## Build the APK on GitHub

The APK is built by GitHub Actions (`.github/workflows/android-apk.yml`), so you
don't need Android Studio.

1. Create a new repository on GitHub (empty, no README).
2. Upload this whole folder to it, or push it with git:
   ```bash
   git remote add origin https://github.com/<you>/<repo>.git
   git push -u origin main
   ```
   Do not upload `node_modules/`, `www/` or `android/app/build/`. The `.gitignore`
   already excludes them.
3. Open the repository's **Actions** tab, choose **Build Android app**, and click
   **Run workflow** (it also runs automatically on every push).
4. When the run finishes, open it and download the **bikini-bottom-bounce-debug**
   artifact. It contains `app-debug.apk`.
5. Copy the APK to your phone and install it. Allow installs from unknown sources
   when Android asks.

The debug APK uses Google's test ads only. For the signed release APK for Uptodown,
see "Release build (Uptodown)" below.

## Release build (Uptodown)

The release job in `.github/workflows/android-apk.yml` builds a **signed release APK**
(`app-release.apk`) with **production ad units**. It runs when you start it
from the Actions tab, or when you push a version tag such as `v1.0.0`.

### One-time setup

1. **Create a signing key** (on your computer, with Java installed). Keep the file and
   passwords safe. If you lose them, you cannot update the app on Uptodown:
   ```bash
   keytool -genkeypair -v -keystore bikini-bottom-release.keystore \
     -alias bikini-bottom -keyalg RSA -keysize 2048 -validity 10000
   ```
2. **Encode the keystore** as one line of text (for the secret below):
   ```bash
   base64 -w 0 bikini-bottom-release.keystore > keystore.b64   # macOS: base64 -i ... -o ...
   ```
3. **Add four GitHub Secrets** (Settings > Secrets and variables > Actions):
   - `ANDROID_KEYSTORE_BASE64`: the contents of `keystore.b64`
   - `ANDROID_KEYSTORE_PASSWORD`: the keystore password
   - `ANDROID_KEY_ALIAS`: `bikini-bottom` (or the alias you chose)
   - `ANDROID_KEY_PASSWORD`: the key password
4. Delete the local `keystore.b64` file once it's in the secret. Never commit the
   keystore or the passwords to the repository.

### Build it

- Actions tab > **Build Android app** > **Run workflow**, or push a tag:
  `git tag v1.0.0 && git push origin v1.0.0`.
- Download the **bikini-bottom-bounce-release** artifact. It contains `app-release.apk`.
- Upload that file to Uptodown.

### Before each Uptodown upload

- Raise `versionCode` in `android/app/build.gradle` (it must increase on every
  upload). `versionName` is what users see, for example `1.0.1`.
- Check the AdMob account is approved and the app is registered in AdMob (see
  "Ads (AdMob)" below).

If the release job says secrets are missing, add the four secrets above and run it again.

### Uptodown and ads

- Uptodown's developer console is free. Sign up, choose **Add new app**, and upload
  `app-release.apk`. Uptodown editors review each submission manually.
- Uptodown is **not** one of the stores AdMob supports for app review. Google says
  apps listed only in unsupported stores can't be reviewed and get limited ad serving.
  Expect limited or no ad revenue from an Uptodown-only release.
- Use the same signing key for every update.

## Ads (AdMob)

Ads use `@capacitor-community/admob` (Android app only). `js/ads.js` is the only
module that calls the ad SDK. Gameplay and UI code call its functions:

- **Banner**: an adaptive banner at the bottom of gameplay (run, pause and game
  over). The playfield is fitted above it through `--ad-inset`, so the banner
  never covers the controls or level info. It is hidden on menus. It is created
  once and then hidden or resumed.
- **Revive**: the game-over button "WATCH AD TO REVIVE" shows a rewarded
  interstitial. The revive is granted only after the SDK's `Rewarded` event. Closing
  the ad early grants nothing. The coin revive is unchanged.
- **App open**: shown on launch (after the menu is ready) and when the app resumes
  to a menu screen. It is never shown during a run, pause, game over, or over a
  modal. Cooldown: 30 minutes (`APP_OPEN_COOLDOWN_MS`). Preloaded ads older than 4 h
  are discarded.
- **Consent**: `requestConsentInfo()` runs first. If the consent form is required
  (EU/UK), `showConsentForm()` runs. The SDK initialises once, and only if
  `canRequestAds` is true.

In the browser, or before the plugin is installed, the ads module does nothing,
except that the revive uses the in-game stand-in ad from `js/main.js`.

### Test vs release units

- **Development builds** (`npm run android:debug`, `npm run build:www`) use
  Google's **test** ad units and the test App ID (`debug` build type in
  `android/app/build.gradle`).
- **Release builds** (`npm run android:release`, `npm run build:www:release`) use
  the production App ID (`ca-app-pub-2098402970512007~7305315882`) and the
  production ad units. The release build copies `js/ads.js` with `AD_BUILD` set to
  `'release'`.

Do not test with real units on a real device. Google's policy forbids clicking
your own ads. Use test ads for all development.

### Before shipping

1. In AdMob, create the app, the three ad units (banner, rewarded interstitial,
   app open), and the consent message (UMP) for EU/UK users.
2. Check the plugin version still matches the Capacitor version (`npm ls`).
3. Make sure the release build is built with `--release`, then check
   `grep "AD_BUILD" android/app/src/main/assets/public/js/ads.js` shows `'release'`.
4. Set up a privacy policy URL for the app.
5. Test on a device with test ads before a production release.

## Known limitations

- Zone 2–5 background provenance (see above).
- Five of six characters are locked placeholders until official art is added.
- Ads: in the browser, the revive uses a stand-in ad. Ads cannot be verified
  without a device, live ad serving, and an approved AdMob account.
