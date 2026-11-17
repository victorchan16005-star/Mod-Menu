# Mod Menu (phase 1)

Floating, draggable, collapsible dark-themed menu for **Julian's Editor** and **Super Bear Adventure**,
with a compatibility scanner and a feature framework. **Status: written but NOT yet compiled or run.**

## What works in this phase
- Game selection screen; each game has its own module, config and feature list.
- Scanner reads the installed game's APK(s): version, native ABIs, Unity or not, Unity version,
  scripting backend (IL2CPP / Mono). Nothing is assumed; both games are scanned independently.
- Floating round button: drag, tap to expand/collapse, animated, optional snap-to-edge and auto-collapse,
  position remembered (stored as screen fractions, so it survives rotation and other screen sizes).
- Foreground service owns the overlay; stop it from the panel (×) or the notification.
- Feature framework: toggle/action, per-feature config, compatibility check (Supported / Unsupported /
  Unverified), error handling, descriptions. Unverified features render disabled with a warning.

## What is intentionally NOT here
No game-modifying features (health, speed, items...). The rules for this project say only verified features
ship, and nothing has been verified on a real install yet. See "Technical limitations".

## Structure
    app/                      game selection screen, Application (registers both games)
    floatingmenu/             overlay service + draggable button + panel (shared by all games)
    core/                     Feature, GameModule, GameReport, GameRegistry
    compatibility/            GameInspector (APK scanner)
    configuration/            ConfigStore (position, per-feature settings, package override)
    features/                 shared, real features (menu settings, re-scan, copy report)
    games/julianseditor/      game module (profile + game-specific features)
    games/superbearadventure/ game module (profile + game-specific features)
    resources/                colors, strings, icons

Add a feature: implement `Feature` in the game's module, return it from `gameFeatures()`.
`checkCompatibility` must return `Supported` only for builds you have verified.

## Build: GitHub Actions (no PC needed)
1. Push this folder to a GitHub repo (branch `main`).
2. Actions tab -> "Build APK" -> Run workflow (it also runs on every push).
3. Download the `modmenu-debug-apk` artifact, unzip, install the APK.

## Build: Android Studio
1. File -> Open -> this folder. Accept the Gradle/AGP update prompts if offered.
2. Build -> Build APK(s). Debug APK: `app/build/outputs/apk/debug/`.
(The Gradle wrapper jar is not included; Android Studio uses `gradle-wrapper.properties`.
For the command line run `gradle wrapper` once, then `./gradlew :app:assembleDebug`.)

## Release APK
Create `keystore.properties` in the project root (never commit it):

    storeFile=my-release.jks
    storePassword=...
    keyAlias=...
    keyPassword=...

Then `./gradlew :app:assembleRelease`. Without that file the release APK is unsigned.
R8 shrinking is on for release.

## Use
Install, open, tap "Show menu" on a game card (grant "Display over other apps"), then open the game.
"Package..." lets you override the package name if your install uses a different one.

## Technical limitations
- A normal app cannot change another app's memory or code. Real game features need one of:
  (a) a patched copy of the game APK that loads a native library (new signature: Play updates, Play
  Integrity, logins and purchases can break), (b) a root/virtual-container setup. Not implemented.
- The right hooks depend on the scripting backend (IL2CPP needs metadata-driven hooking; Mono needs
  managed patching), which is why the scanner reports it per game instead of assuming.
- Julian's Editor is an online, user-generated-content platform. Only features that affect your own
  offline sessions are appropriate there.
- "Latest version" can only be checked against what is installed on the device.
