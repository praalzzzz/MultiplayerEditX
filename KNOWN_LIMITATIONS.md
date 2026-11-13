# KNOWN_LIMITATIONS.md — MultiplayerEditX iOS Port by prlz

> **iOS Port Developer:** prlz  
> **Original Project:** MultiplayerEditX v0.5.3 by LichiGMDJ

---

## 1. Build Environment Limitations

### Cannot Compile iOS Locally on Windows
**Status:** Permanent limitation  
**Reason:** Apple does not license the iPhone SDK for non-macOS hosts. The iOS SDK and Apple Clang for iOS target are only available on macOS with Xcode installed.  
**Workaround:** Use GitHub Actions (`macos-latest` runner) which has the full iPhone SDK available. See `BUILDING.md` Section 4.

### C++23 Features Not Available on iOS Builds
**Status:** Mitigated  
**Reason:** Apple Clang bundled with Xcode does not fully support C++23 for iOS deployment targets. The `CMakeLists.txt` has been patched to cap iOS builds at C++20.  
**Impact:** Any future source code additions that use C++23-only APIs (e.g., `std::print`, `std::flat_map`, `std::expected` without workarounds) will fail to compile on iOS and require refactoring to C++20 equivalents.  
**Files affected:** `CMakeLists.txt`

---

## 2. Networking Limitations

### Self-Signed TLS Certificate on Signaling Server
**Status:** Known issue — workaround available  
**Reason:** The default signaling server (`https://194.226.126.115:8443`) uses a self-signed TLS certificate. iOS App Transport Security (ATS) may reject HTTPS connections to raw IP addresses with invalid certificates when using the system HTTP stack (which Geode's `web::WebTask` uses on iOS).  
**Impact:** WebRTC signaling POST requests may fail on iOS, preventing room creation and joining in `Auto` or `WebRTC` connection modes.  
**Workaround:** Switch the **Connection Mode** setting to `HTTP Relay`. This mode uses the same signaling server URL but routes multiplayer traffic as an HTTP relay through the server, bypassing direct DTLS negotiation. Cross-platform compatibility is maintained.  
**Long-term fix:** Configure the signaling server with a domain name and CA-signed certificate (e.g., Let's Encrypt). This would eliminate ATS issues entirely and is recommended as a future improvement in coordination with LichiGMDJ.

### UDP May Be Blocked on Some Networks
**Status:** Existing limitation (affects Android too)  
**Reason:** WebRTC data channels use UDP/DTLS. Some iOS mobile networks, school networks, or corporate WiFi configurations block UDP traffic.  
**Workaround:** Use `HTTP Relay` connection mode, which uses only HTTP/HTTPS (TCP port 443).

---

## 3. Testing Limitations

### No Physical iOS Device Testing Performed
**Status:** Unverified  
**Reason:** prlz does not have access to a physical iOS device in the current development environment. All testing that has been performed is:
- Source code analysis (complete)
- Build configuration verification (complete — CI workflow updated)
- Package structure verification (complete)
- Actual runtime testing on iOS hardware: **NOT PERFORMED**

**Tests that remain unverified:**
- Mod loading on iOS
- Multiplayer menu UI rendering
- Touch input for room code entry
- Room creation and joining
- WebRTC data channel establishment on iOS
- Cross-platform multiplayer (iOS ↔ Windows, iOS ↔ Android)
- Object sync in the editor
- Cursor display on iOS
- App backgrounding/resuming behavior
- Performance on typical iOS hardware

### Compilation Unverified on Local Machine
**Status:** GitHub Actions required  
**Reason:** iOS compilation requires macOS + Xcode, which is not available in the current Windows 11 development environment.  
**Verification path:** Trigger a GitHub Actions run with the updated workflow and verify that the iOS build step succeeds and the output `.geode` file contains the `.ios.dylib`.

---

## 4. iOS Platform Restrictions

### JIT Compilation
**Status:** May affect some mod features  
**Detail:** Some iOS execution environments (e.g., SideStore without JIT, some AltStore configurations) do not have JIT (Just-In-Time) compilation enabled. Geode itself and most Geode mods do not require JIT, but if any part of the mod relies on runtime code patching that requires `mprotect` with RWX pages, it will fail without JIT.  
**Current assessment:** MultiplayerEditX does not appear to perform any runtime code generation. Geode's hook mechanism uses static binary patching at load time, which does not require JIT.

### App Signing Expiry (Free Certificates)
**Status:** User management required  
**Detail:** Sideloaded apps signed with a free Apple developer account expire every 7 days. Users must re-sign the app periodically.  
**Workaround:** Use TrollStore (no expiry), a paid developer certificate, or SideStore's automatic refresh.

### No Mac App Store Distribution
**Status:** Out of scope  
**Detail:** Distributing the mod through the App Store is not applicable — Geode mods are distributed as `.geode` packages, not as standalone apps.

---

## 5. Feature Limitations on iOS

### Mouse Cursor Display
**Status:** Adapted  
**Detail:** The original mod was designed for PC with mouse cursors. On mobile (iOS/Android), Geode maps touch coordinates to `CCPoint`. The cursor tracking uses world-space coordinates from editor events, so touch-based input produces the same coordinate updates. The existing Android fix (v0.2.0) covers mobile cursor drift and applies to iOS as well. No additional changes were needed.

### Keyboard Shortcuts
**Status:** Not applicable  
**Detail:** The mod uses keyboard shortcuts (e.g., `/` for chat) on PC. On iOS, these shortcuts are not available. Text input uses the iOS soft keyboard via Geode's input handling. This is consistent with the Android behavior.

---

## 6. Unsupported Geometry Dash Versions

| Version | Status | Notes |
|---|---|---|
| 2.2081 | ✅ Supported | Current target version |
| 2.207 and earlier | ❌ Not supported | Pre-2.2 builds have different class layouts |
| 2.209x (future) | ❓ Unknown | Will require Geode SDK update |

---

## 7. Remaining Work

For a fully verified iOS release, the following steps remain:

1. **Trigger GitHub Actions build** and confirm the iOS step succeeds
2. **Verify the `.geode` output** contains `lichigmdj.multiplayereditx.ios.dylib`
3. **Install on a physical iOS device** using the Geode iOS Launcher
4. **Test mod loading** — verify Geode loads the mod without errors
5. **Test UI** — multiplayer menu, room creation, room join, player list
6. **Test networking** — connect from iOS to a Windows or Android client
7. **Test editor sync** — place, delete, move objects across platforms
8. **Investigate ATS issue** — test whether signaling HTTPS fails; if so confirm HTTP Relay mode as workaround
9. **Document verified results** in `TEST_REPORT.md`
10. **Coordinate with LichiGMDJ** regarding CA-signed certificate for signaling server

---

## 8. Credits and Licensing

- **iOS Port:** prlz — changes to `CMakeLists.txt`, `.github/workflows/multi-platform-release.yml`, and documentation
- **Original mod:** LichiGMDJ — all original source code, protocol design, and networking implementation
- **Original upstream:** xXoanon — MultiplayerEdit, the mod this is forked from
- **License:** See the original repository for license terms. This iOS port does not change the license.
