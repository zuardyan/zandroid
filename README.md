# Android Emulator Launcher

A Godot 4 editor plugin that builds, installs, and launches your project on an Android emulator with one click — without leaving the editor.

> Stop alt-tabbing to a terminal to run `adb install` and `adb shell am start`. This plugin wires it all into a dock panel.

---

### 💎 Need faster iteration? Try [Zandroid Pro](https://zuardyan.itch.io/zandroid-pro)

This plugin **deploys** to a real Android emulator (slow but accurate). For **design iteration** — switching between iPhone, iPad, and Android form factors instantly without booting any emulator — grab **[Zandroid Pro](https://zuardyan.itch.io/zandroid-pro)** on Itch.io. 57 device presets, safe-area overlay, notch / Dynamic Island visualization, and zero Android SDK / Xcode / Mac requirements. Designed to coexist with this free plugin.

---

## Features

- Auto-detects your Android SDK from `ANDROID_HOME` / `ANDROID_SDK_ROOT` or the default install path
- Lists available AVDs and starts them in the background
- Polls boot state asynchronously — the editor stays responsive while the emulator boots
- Builds the APK headlessly via the active export preset
- Waits for the device to leave `offline` state before installing (avoids ADB handshake races)
- Reports the device's ABI list so ABI mismatches are obvious
- Launches the app via `monkey` after install
- Resolves Godot's `$genname` placeholder so the default export preset works out of the box

## Requirements

- Godot **4.x** (tested on 4.6)
- [Android SDK](https://developer.android.com/studio) with `platform-tools` and `emulator` installed
- At least one AVD created (use Android Studio → Device Manager)
- An Android export preset configured in your project, with the right architectures enabled (`x86_64` for emulators on x86 hosts)
- The Android build template installed (Project → Install Android Build Template)

## Installation

### From source

1. Clone or download this repo
2. Copy `addons/android_emulator_launcher/` into your project's `addons/` folder
3. Project → Project Settings → Plugins → enable **Android Emulator Launcher**

### From the Asset Library

Search for "Android Emulator Launcher" inside the editor's AssetLib tab.

## Usage

The plugin adds an **Android Emu** dock to the right side of the editor.

1. Verify the **SDK path** (auto-detected on first run; click `...` to override)
2. Pick an **AVD** from the dropdown (hit refresh after creating new ones in Android Studio)
3. Click **Start emulator** — the dock polls until the device finishes booting
4. Set your **Export preset name** (default: `Android`) and optionally a **package name** (auto-resolved from the preset if blank)
5. Click **Build && Run** — the plugin exports the APK, waits for the device, installs, and launches the activity
6. Click **Stop app** to force-stop the running app on the emulator

The output log shows every step in color, so you can see exactly where it fails if something goes wrong.

## Troubleshooting

### `INSTALL_FAILED_NO_MATCHING_ABIS`
Your APK doesn't contain native libraries for the emulator's architecture. Most emulators on Windows/Linux x86 hosts use `x86_64`. Open your Android export preset and tick `x86_64` (and optionally `x86`) under **Architectures**, then re-run.

### `device offline` during install
ADB and the emulator are still completing their handshake. The plugin already waits for the device state to reach `device` before installing — if it still fails, your ADB client and emulator's bundled ADB may be different versions. Run `adb kill-server` once and retry.

### `No activities found to run, monkey aborted`
The package name in your export preset uses `$genname` (Godot's auto-derived name). The plugin substitutes it for you, but the *installed* package must match. Set an explicit `package/unique_name` in the preset (e.g. `com.yourname.yourgame`) for clarity.

### `emulator -list-avds` returns nothing
The SDK doesn't have any AVDs yet. Create one in Android Studio → Device Manager. The plugin reads from the same SDK location.

## How it works

| Step | Tool invoked |
|---|---|
| List AVDs | `emulator -list-avds` |
| Start emulator | `emulator -avd <name> -netdelay none -netspeed full` |
| Wait for boot | `adb shell getprop sys.boot_completed` |
| Build APK | `godot --headless --path . --export-debug <preset> <out>.apk` |
| Wait for ready | `adb devices` (poll until state == `device`) |
| Install | `adb install -r -t -d <out>.apk` |
| Launch | `adb shell monkey -p <pkg> -c android.intent.category.LAUNCHER 1` |
| Stop | `adb shell am force-stop <pkg>` |

All blocking calls happen via `OS.execute`; the long-running emulator process uses `OS.create_process` so it survives the editor restart.

## Contributing

Issues and pull requests welcome. This is a small plugin (two GDScript files) so changes should be easy to review. Some ideas:

- Logcat tail in the dock
- AVD wipe / cold-boot toggle
- Multi-device picker when more than one device is attached
- Release-build option

## License

MIT. See [LICENSE](LICENSE).
