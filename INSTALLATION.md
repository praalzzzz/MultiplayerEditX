# INSTALLATION.md — MultiplayerEditX iOS Port by prlz

> **iOS Port Developer:** prlz  
> **Original Project:** MultiplayerEditX v0.5.3 by LichiGMDJ

---

## Overview

This document covers installing MultiplayerEditX on iOS using the Geode mod loader. The iOS port requires the Geode iOS Launcher to be set up on your device first.

> [!CAUTION]
> Do not distribute or publish this mod without explicit permission from the original developer (LichiGMDJ). The iOS port is maintained by prlz for compatibility purposes.

---

## 1. Requirements

| Requirement | Detail |
|---|---|
| **iOS version** | iOS 14.0 or newer |
| **Geometry Dash** | Version 2.2081 (App Store) |
| **Geode iOS Launcher** | Latest from [github.com/geode-sdk/ios-launcher](https://github.com/geode-sdk/ios-launcher) |
| **Device** | Any 64-bit iPhone or iPad (all modern devices) |
| **Jailbreak** | Not required (TrollStore or SideStore methods available) |

---

## 2. Install the Geode iOS Launcher

The Geode iOS Launcher wraps Geometry Dash and loads `.geode` mods. You must install it before installing any mods.

### Method A — TrollStore (Recommended, iOS 14–16.x on supported devices)

1. Install TrollStore on your device (follow the TrollStore documentation for your iOS version)
2. Download the latest Geode iOS Launcher `.ipa` from [github.com/geode-sdk/ios-launcher/releases](https://github.com/geode-sdk/ios-launcher/releases)
3. Open the `.ipa` in TrollStore and tap **Install**
4. Launch the Geode app and follow its first-run setup to hook into Geometry Dash

### Method B — SideStore / AltStore (iOS 14+, no jailbreak)

1. Install SideStore or AltStore on your device (requires a computer for initial setup)
2. Download the Geode iOS Launcher `.ipa`
3. Open the `.ipa` in SideStore/AltStore and sideload it
4. Re-sign every 7 days with a free Apple developer certificate, or use a paid certificate for 1 year

### Method C — Jailbreak (Full JIT, all iOS versions up to your jailbreak)

1. Install Geode via your package manager (Sileo/Zebra) if a jailbreak package is available
2. Or sideload the launcher `.ipa` via your preferred jailbreak method with unrestricted signing

> [!NOTE]
> TrollStore provides the best experience: no re-signing required, full JIT support, and works without a jailbreak on supported firmware versions. Check [ios.cfw.guide](https://ios.cfw.guide) for TrollStore compatibility with your iOS version.

---

## 3. Install the MultiplayerEditX .geode File

Once the Geode iOS Launcher is installed and working:

### From GitHub Actions (CI Build)
1. Go to the GitHub Actions runs for this repository
2. Find the latest successful `Build Multiplayer Edit X` run
3. Download the artifact named `lichigmdj.multiplayereditx-0.5.3-ios-prlz`
4. The downloaded `.zip` contains the `.geode` file

### Copy the .geode File to Your Device

**Method 1 — Files app / iCloud Drive:**
1. Upload the `.geode` file to your iCloud Drive or Files app
2. On your iOS device, open the Files app
3. Navigate to your Geode mods folder: `On My iPhone → Geode → mods`
4. Copy the `.geode` file into this folder

**Method 2 — iTunes / Finder File Sharing:**
1. Connect your device to your Mac or PC
2. Open Finder (macOS) or iTunes (Windows)
3. Select your device → Files → Geode
4. Drag the `.geode` file into the Geode Documents folder

**Method 3 — SSH / AFC (Jailbreak):**
```bash
scp lichigmdj.multiplayereditx.geode root@<device-ip>:/var/mobile/Documents/Geode/mods/
```

---

## 4. Enable and Configure the Mod

1. Launch Geometry Dash via the Geode iOS Launcher
2. The Geode overlay should appear — tap the Geode icon
3. Navigate to **Mods** → find **Multiplayer Edit X**
4. Tap to enable it
5. Restart Geometry Dash if prompted

### Configure Network Settings

Open **Mods → Multiplayer Edit X → Settings**:

| Setting | Recommended Value for iOS |
|---|---|
| **Signaling Server URL** | `https://194.226.126.115:8443` (default) |
| **Connection Mode** | `HTTP Relay` (use if WebRTC signaling fails due to TLS) |
| **Player Name** | Your display name |
| **Show Player Cursors** | Enabled |

> [!WARNING]
> The default signaling server uses a self-signed TLS certificate. If the WebRTC connection mode fails on iOS (due to App Transport Security), switch **Connection Mode** to `HTTP Relay`. This routes all traffic through the signaling server as a relay, bypassing direct DTLS negotiation. Cross-platform compatibility with Windows and Android users is maintained in this mode.

---

## 5. Using the Mod

1. Open any level in the **editor**
2. Tap the **pause menu** button (or the editor toolbar)
3. A **Multiplayer** button should appear
4. To **host**: tap Host, share the 6-character room code with friends
5. To **join**: tap Join and enter the room code

---

## 6. Signing Requirements

### Free Apple Certificate (SideStore/AltStore)
- Valid for 7 days, requires re-signing
- Sufficient for personal testing
- No JIT enabled by default (some mods may not work)

### Paid Apple Developer Certificate (\$99/year)
- Valid for 1 year
- Full functionality
- Can be distributed via TestFlight or MDM

### TrollStore Signing
- Uses platform-signed entitlements, no expiry
- Requires a supported iOS version
- Recommended for personal use

> [!NOTE]
> The mod itself does not require any special entitlements or signing configurations. Standard iOS app signing is sufficient.

---

## 7. Troubleshooting

### Mod doesn't appear in Geode's mod list
- Verify the `.geode` file is in the correct mods folder
- Verify the file is named `lichigmdj.multiplayereditx.geode` (exact case)
- Check that your Geode iOS Launcher version supports Geode SDK 5.9.0

### "Mod not compatible with this platform" error
- This means the `.geode` file does not contain an `ios.dylib`
- You need the iOS port build — see Section 3 above
- Verify the `.geode` file contains `lichigmdj.multiplayereditx.ios.dylib`

### Cannot connect to other players
- Check your internet connection
- Try switching **Connection Mode** to `HTTP Relay` in mod settings
- Ensure the signaling server is reachable (try opening `https://194.226.126.115:8443` in Safari — you may see a certificate warning, which confirms the server is reachable)
- If using WebRTC mode, ensure your network allows UDP traffic (some corporate/school networks block it)

### Cursors appear in wrong position
- This can occur if the editor camera panning is not synced correctly
- Reported as fixed in v0.2.0 for mobile; if occurring on iOS, report as a new issue

### App crashes on launch
- Ensure Geometry Dash version is exactly **2.2081** (the App Store version)
- Ensure the Geode iOS Launcher version matches Geode SDK 5.9.0
- Check Geode's crash log in the Geode overlay → Logs

### "Could not load mod" error
- Likely a Geode SDK version mismatch
- Update your Geode iOS Launcher to the version that supports SDK 5.9.0
