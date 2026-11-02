# EmuKit

EmuKit is a fork of [Lemuroid](https://github.com/Swordfish90/Lemuroid), an open-source emulation app for Android built on [Libretro](https://www.libretro.com/). Its main goals are ease of use, good Android integration and a great user experience.

Lemuroid started as an offshoot of [Retrograde](https://github.com/retrograde/retrograde-android) and became a standalone project built around [LibretroDroid](https://github.com/Swordfish90/LibretroDroid).

> **Note:** For now this fork keeps the upstream package name (`com.swordfish.lemuroid`), app name and core download URLs. That means a release build can't be installed next to an official Lemuroid install (see [Fork notes](#fork-notes)).

|Screen 1|Screen 2|Screen 3|
|---|---|---|
|![Screen1](fastlane/metadata/android/en-US/images/phoneScreenshots/1.jpg)|![Screen2](fastlane/metadata/android/en-US/images/phoneScreenshots/2.jpg)|![Screen3](fastlane/metadata/android/en-US/images/phoneScreenshots/3.jpg)|

## Supported systems

| System | Core |
|---|---|
| Atari 2600 (A26) | [stella](https://docs.libretro.com/library/stella/) |
| Atari 7800 (A78) | [prosystem](https://docs.libretro.com/library/prosystem/) |
| Atari Lynx | [handy](https://docs.libretro.com/library/handy/) |
| Nintendo (NES) | [fceumm](https://docs.libretro.com/library/fceumm/) |
| Super Nintendo (SNES) | [snes9x](https://docs.libretro.com/library/snes9x/) |
| Game Boy / Game Boy Color | [gambatte](https://docs.libretro.com/library/gambatte/) |
| Game Boy Advance | [mgba](https://docs.libretro.com/library/mgba/) |
| Nintendo 64 | [mupen64plus-next](https://docs.libretro.com/library/mupen64plus/) |
| Nintendo DS | [desmume](https://docs.libretro.com/library/desmume/) / [melonDS](https://docs.libretro.com/library/melonds/) |
| Nintendo 3DS | [citra](https://docs.libretro.com/library/citra/) |
| Sega Genesis / Mega Drive, Sega CD, Master System, Game Gear | [genesis_plus_gx](https://docs.libretro.com/library/genesis_plus_gx/) |
| PlayStation | [PCSX-ReARMed](https://docs.libretro.com/library/pcsx_rearmed/) |
| PlayStation Portable | [ppsspp](https://docs.libretro.com/library/ppsspp/) |
| Arcade | [FinalBurn Neo](https://github.com/libretro/FBNeo/), [MAME 2003-Plus](https://docs.libretro.com/library/mame2003_plus/) |
| NEC PC Engine | [beetle_pce_fast](https://docs.libretro.com/library/beetle_pce_fast/) |
| Neo Geo Pocket / Pocket Color | [mednafen_ngp](https://docs.libretro.com/library/beetle_neopop/) |
| WonderSwan / WonderSwan Color | [beetle_cygne](https://docs.libretro.com/library/beetle_cygne/) |
| DOS | [DOSBox Pure](https://docs.libretro.com/library/dosbox_pure/) |

## Features

- Android TV support
- Automatic save and restore of game state
- ROM scanning and indexing
- Optimized, customizable touch controls (size and position)
- Quick save/load
- Zipped ROM support
- Display simulation (LCD/CRT)
- Gamepad support and local multiplayer
- Tilt input
- Cloud save sync
- HD mode

## Building

### Requirements

- JDK 17
- Android SDK with platform 35 and build tools 34.0.0 (Android Studio installs these for you)
- Git with submodule support. The prebuilt Libretro cores live in the [`lemuroid-cores`](https://github.com/Swordfish90/LemuroidCores) submodule, which is about 600 MB.

### Get the source

```sh
git clone --recurse-submodules https://github.com/tech2mar/EmuKit.git
cd EmuKit

# If you already cloned without submodules:
git submodule update --init --recursive
```

### Build variants

The app module has two flavor dimensions:

| Dimension | Flavor | Meaning |
|---|---|---|
| `opensource` | `free` | No Google dependencies (the F-Droid / GitHub build) |
| | `play` | Uses Google Play feature delivery, in-app review and Play Services |
| `cores` | `bundle` | All cores are packaged inside the APK (large, works offline) |
| | `dynamic` | Cores are downloaded on demand, from GitHub (`free`) or as Play dynamic feature modules (`play`) |

Common commands:

```sh
# Debug build with every core bundled. Installs as com.swordfish.lemuroid.debug
./gradlew assembleFreeBundleDebug
./gradlew installFreeBundleDebug

# Release APK, the same one CI publishes
./gradlew assembleFreeDynamicRelease

# Play bundle with per-core dynamic feature modules
./gradlew bundlePlayDynamicRelease
```

APKs are written to `lemuroid-app/build/outputs/apk/<flavor>/<buildType>/`.

The `:lemuroid_core_*` dynamic feature modules are only included in the Gradle build when the requested task name contains both `Play` and `Dynamic` (see [settings.gradle.kts](settings.gradle.kts)).

### Signing

- **Debug** builds use the checked-in `debug.keystore`.
- **Release** builds expect a `release.jks` in the project root with key alias `lemuroid` (see [lemuroid-app/build.gradle.kts](lemuroid-app/build.gradle.kts)). Supply your own keystore, or change the signing config, before building a release.

### Code style

The project uses [ktlint](https://github.com/pinterest/ktlint). CI runs the same check on every push and pull request to `master`:

```sh
./gradlew assembleDebug ktlintCheck
./gradlew ktlintFormat   # auto-fix formatting
```

## Project structure

| Module | Purpose |
|---|---|
| `lemuroid-app` | Main application: UI (Compose, Leanback for TV), game library, emulation screen |
| `lemuroid-app-ext-free` | `free` flavor implementations, such as downloading cores from GitHub |
| `lemuroid-app-ext-play` | `play` flavor implementations: Play feature delivery, reviews, Play Services |
| `retrograde-app-shared` | Shared logic: game systems and cores, storage, ROM scanning, saves |
| `retrograde-util` | Generic utilities |
| `lemuroid-touchinput` | On-screen touch controls |
| `lemuroid-metadata-libretro-db` | Game metadata lookup from the Libretro database |
| `lemuroid-cores` (submodule) | Prebuilt Libretro cores: `bundled-cores` plus one dynamic feature module per core |
| `baselineprofile` | Baseline profile generation for startup performance |
| `buildSrc` | Shared SDK and dependency versions ([deps.kt](buildSrc/src/main/java/deps.kt)) |

## Releases

Pushing any git tag triggers the [Publish workflow](.github/workflows/publish.yml). It builds `assembleFreeDynamic` and attaches the APK to a GitHub pre-release. The workflow decrypts the release keystore from the `RELEASE_KEYSTORE` and `RELEASE_KEYSTORE_PASSPHRASE` repository secrets, so a fork has to set its own secrets.

## Fork notes

EmuKit still matches upstream Lemuroid. Before shipping it as a separate app, you'll probably want to change these:

- `applicationId` and the `lemuroid_name` string in [lemuroid-app/build.gradle.kts](lemuroid-app/build.gradle.kts)
- The core download URL in [CoreUpdaterImpl.kt](lemuroid-app-ext-free/src/main/java/com/swordfish/lemuroid/ext/feature/core/CoreUpdaterImpl.kt) and [PPSSPPAssetsManager.kt](retrograde-app-shared/src/main/java/com/swordfish/lemuroid/lib/core/assetsmanager/PPSSPPAssetsManager.kt). Both point to `Swordfish90/LemuroidCores`, and the `versionName` must match a tag there.
- The release signing config and the CI secrets

## Credits and license

Lemuroid is created by [Filippo Scognamiglio](https://github.com/Swordfish90) and contributors. You can get the official app on [F-Droid](https://f-droid.org/packages/com.swordfish.lemuroid/) and [Google Play](https://play.google.com/store/apps/details?id=com.swordfish.lemuroid). Upstream translations are managed on [Crowdin](https://crowdin.com/project/lemuroid).

This project is licensed under the GNU General Public License v3.0. See [COPYING](COPYING).
