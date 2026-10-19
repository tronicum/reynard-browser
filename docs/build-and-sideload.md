# Building Reynard and running it on your own iPhone

This guide covers building Reynard from source and installing it on a device you own with a regular Apple Developer account. It is based on a build that ran on an **iOS 27.0.1** device. Other setups may need adjustments.

> [!NOTE]
> Placeholders used below: `com.example` is your own reverse-DNS prefix, and `YOUR_TEAM_ID` is your 10-character Apple Developer Team ID. Replace both with your own values. Use a prefix you control, since App IDs are global to Apple's developer system.

## What you need

- An Apple silicon Mac with enough free disk space. The Gecko build directory alone is around 30 GB, so plan for 60 GB or more including sources and toolchains.
- Xcode, plus [Python 3](https://www.python.org/downloads/) and Rust via [rustup](https://rustup.rs/).
- [sccache](https://github.com/mozilla/sccache) (`brew install sccache`). It makes later Gecko rebuilds much faster.
- A paid Apple Developer account, with the current Program License Agreement accepted at [developer.apple.com/account](https://developer.apple.com/account).
- An iPhone with **Developer Mode** turned on (Settings → Privacy & Security → Developer Mode).

### Which Xcode?

Two constraints pull in opposite directions:

| Need | Xcode |
|---|---|
| Talk to a device running **iOS 27** (mounting the developer disk image, installing) | Xcode 27 or newer |
| Compile Reynard as it is today (deployment target iOS 13) | An Xcode that still supports iOS 13, such as 26.x |

Xcode 27 only supports deployment targets from iOS 15, and the sources do not compile when you raise the target to 15 (for example `#unavailable(iOS 14.0)` in `main.swift`, and an API that the newer SDK marks iOS 16+ in `ToolbarButtonMenus.swift`).

So the working setup is to **build and sign with Xcode 26.x, then install with Xcode 27's `devicectl`**. You can keep both installed side by side. Select one per command with `DEVELOPER_DIR`:

```bash
export DEVELOPER_DIR=/Applications/Xcode-27.app/Contents/Developer   # use the path of your install
```

If your phone runs iOS 26 or earlier, a single Xcode 26.x is enough.

## 1. Get the sources and build the engine

```bash
git clone --recursive https://github.com/minh-ton/reynard-browser
cd reynard-browser

./tools/development/update-gecko.sh
./tools/development/apply-patches.sh
```

### Put the rustup toolchain first on your PATH

If you also have Homebrew's `rust` installed, it can shadow rustup. Homebrew's Rust has no iOS standard library, so `build-idevice.sh` fails with `can't find crate for 'core'` even though rustup has the iOS target. Fix it for the build session:

```bash
export PATH="$HOME/.rustup/toolchains/stable-aarch64-apple-darwin/bin:$PATH"
```

### Build

```bash
./tools/development/build-idevice.sh
./tools/development/build-gecko.sh --use-sccache
```

`build-idevice.sh` takes a few minutes. `build-gecko.sh` is the long step: expect roughly 2 hours on the first run, with no progress percentage in the output (mach only prints warnings and errors). Run it somewhere it will not be interrupted, and keep the Mac awake:

```bash
caffeinate -dims -w <pid-of-build-gecko>
```

The Gecko build can also be re-run later. It is incremental, and `--use-sccache` speeds up clean rebuilds.

> [!NOTE]
> Gecko does not have prebuilt artifact builds for `mobile/ios`, and Reynard patches Gecko's C++, so there is no shortcut around compiling it.

## 2. Use your own App IDs and team

The project ships with the original author's bundle IDs and team. Apple will not let you register those, so point all four targets at your own. In `browser/Reynard.xcodeproj/project.pbxproj`:

- Replace the prefix `com.minh-ton.` with `com.example.` in the four `PRODUCT_BUNDLE_IDENTIFIER` values. You end up with `com.example.Reynard`, `com.example.Reynard.Helper`, `com.example.Reynard.OpenIn` and `com.example.GeckoView`. The extensions must stay prefixed by the app's ID.
- Replace the existing `DEVELOPMENT_TEAM` value with `YOUR_TEAM_ID`.

For example:

```bash
sed -i '' -e 's/com\.minh-ton\./com.example./g' \
          -e 's/NAR5D6HWAD/YOUR_TEAM_ID/g' \
          browser/Reynard.xcodeproj/project.pbxproj
```

Do not commit these edits upstream. The other `com.minh-ton` strings in the source (dispatch queue labels, log domains, menu identifiers) are harmless internal labels and can be left alone.

The standard build uses `Reynard/Entitlements/Reynard.entitlements`, which only asks for the increased memory limit capability. Automatic signing enables that for your App ID. The `*.private.entitlements` files are for jailbroken and TrollStore builds and cannot be signed with a developer certificate.

## 3. Build and sign the app

With Xcode 26.x selected:

```bash
cd browser
xcodebuild -project Reynard.xcodeproj -scheme Reynard -configuration Release \
  -destination 'generic/platform=iOS' \
  -allowProvisioningUpdates -allowProvisioningDeviceRegistration \
  build
```

- `-allowProvisioningUpdates` lets Xcode create App IDs and provisioning profiles.
- `-allowProvisioningDeviceRegistration` registers your phone with your developer account. This uses one of your device slots.
- You need to be signed into Xcode with your Apple ID (Settings → Accounts).

The result is in Xcode's DerivedData folder:

```bash
ls ~/Library/Developer/Xcode/DerivedData/Reynard-*/Build/Products/Release-iphoneos/Reynard.app
```

Optional sanity check: the app is signed by your development identity and the embedded profile lists your device.

```bash
APP=$(ls -d ~/Library/Developer/Xcode/DerivedData/Reynard-*/Build/Products/Release-iphoneos/Reynard.app)
codesign -dvv "$APP" 2>&1 | grep -E "Identifier|TeamIdentifier"
security cms -D -i "$APP/embedded.mobileprovision" | grep -A8 ProvisionedDevices
```

## 4. Install and launch

Connect and **unlock** the phone, tap **Trust** if asked, and keep the screen on. Then, with Xcode 27 (or your single Xcode) selected:

```bash
xcrun devicectl list devices            # find your device UDID; it should say "connected"
APP=$(ls -d ~/Library/Developer/Xcode/DerivedData/Reynard-*/Build/Products/Release-iphoneos/Reynard.app)
xcrun devicectl device install app --device <UDID> "$APP"
xcrun devicectl device process launch --device <UDID> com.example.Reynard
```

If iOS says the developer is not trusted, open Settings → General → VPN & Device Management and trust your certificate.

## Troubleshooting

| Symptom | Cause and fix |
|---|---|
| `build-idevice.sh`: `can't find crate for 'core'` | Homebrew Rust is shadowing rustup. Put `~/.rustup/toolchains/stable-aarch64-apple-darwin/bin` first on `PATH`. |
| `build-gecko.sh` stalls for a long time during configure | It can hang while bootstrapping toolchains (the macOS SDK download). Stop it and re-run; the object directory is reused. |
| `devicectl`: "no DDI" / "developer disk image could not be mounted" | Usually the phone is locked, Developer Mode is off, or your Xcode is older than the phone's iOS version. Unlock the phone, then check Xcode's version. |
| `devicectl`: "device is currently locked" | Unlock the phone and keep the screen on. |
| `devicectl`: error 4016 or the device shows as `unavailable` | The connection dropped. Replug the cable, unlock, tap Trust. |
| `xcodebuild`: "PLA Update available" | Accept the new Program License Agreement in your developer account. |
| `xcodebuild`: "device isn't registered in your developer account" | Add `-allowProvisioningDeviceRegistration`. |
| `xcodebuild`: "deployment target … range of supported versions is 15.0 to 27.0" | You are using Xcode 27 to build. Build with Xcode 26.x and install with Xcode 27 (see above). |
| `Provisioning profile … doesn't include the Increased Memory Limit capability` | Usually a side effect of the license or device registration errors above. Fix those first and retry. |

## Notes

- This is a plain developer-signed sideload. The project's README describes TrollStore and jailbroken builds for automatic JIT. This guide does not cover those.
- Developer-signed apps expire according to your account type (profiles on free accounts last seven days). Re-run the build and install steps to refresh.
- No support is provided for build problems, as stated in the main README.
