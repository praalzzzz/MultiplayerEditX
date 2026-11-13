# iOS Compatibility Analysis — MultiplayerEditX iOS Port by prlz

> **iOS Port Developer:** prlz  
> **Original Project:** MultiplayerEditX v0.5.3 by LichiGMDJ  
> **Analysis Date:** November 2026

---

## Summary

The source code of MultiplayerEditX v0.5.3 is **structurally iOS-compatible**. The `mod.json` already declares `"ios": "2.2081"`, and the `CMakeLists.txt` already contains the correct iOS architecture guard. The **sole blocking issue** is that the GitHub Actions CI workflow does not build the iOS target, so no `.ios.dylib` is ever produced or packaged.

There are **five issues** to address, ranked by severity:

| # | Issue | Severity | File(s) |
|---|---|---|---|
| 1 | iOS target missing from CI build matrix | **CRITICAL** | `.github/workflows/multi-platform-release.yml` |
| 2 | C++23 standard — not supported by Apple Clang for iOS | **HIGH** | `CMakeLists.txt` |
| 3 | Self-signed TLS certificate — iOS ATS may reject HTTPS | **MEDIUM** | `src/net/NetworkConfig.cpp`, `mod.json` |
| 4 | Cursor node uses mouse-position semantics | **LOW** | `src/ui/CursorNode.cpp` |
| 5 | `NO_WEBSOCKET ON` — signaling uses WebSocket | **LOW** | `CMakeLists.txt` |

---

## Issue 1 — iOS Target Missing from CI Build Matrix

### File
`.github/workflows/multi-platform-release.yml`

### Problem
The workflow matrix only defines `Windows64` and `Android64` targets. There is no `iOS` entry. When the CI runs, it builds and packages `.dll` and `.android64.so`, but never builds `.ios.dylib`. Without the dylib, the mod cannot be installed or loaded on iOS, even though `mod.json` declares iOS support.

This was fixed in v0.4.0 (per the changelog: *"Fixed the iOS binary (.ios.dylib) missing from the .geode package by adding the iOS platform to the automated GitHub Actions build matrix"*) but was later removed — presumably during a CI refactor around v0.5.0.

### Why It Fails on iOS
The Geode mod loader on iOS looks for `lichigmdj.multiplayereditx.ios.dylib` inside the `.geode` package. When the binary is absent, Geode refuses to load the mod on iOS.

### Proposed Fix
Add an `iOS` entry to the matrix using `macos-latest` runner (required, as iOS SDK is macOS-only):

```yaml
- name: iOS
  os: macos-latest
  target: iOS
```

Also update the package job to include iOS in its name and artifact handling.

### Affects Other Platforms?
No. The Windows and Android builds are unaffected.

### How to Test
After CI runs with the iOS entry added, the produced `.geode` file should contain `lichigmdj.multiplayereditx.ios.dylib`. Verify with:
```bash
unzip -l lichigmdj.multiplayereditx.geode | grep ios
```

---

## Issue 2 — C++23 Standard Not Supported by Apple Clang for iOS

### File
`CMakeLists.txt`, line 5

```cmake
set(CMAKE_CXX_STANDARD 23)
```

### Problem
Apple Clang (bundled with Xcode) does not fully support C++23 for iOS targets as of Xcode 15/16. The Geode SDK's iOS CI uses C++20. Setting `CMAKE_CXX_STANDARD 23` may cause build failures on the `macos-latest` runner targeting iOS, specifically around C++23 features like `std::print`, `std::expected` without workarounds, or modules.

### Why It Fails on iOS
Clang 16 (Xcode 16) has partial C++23 support but not complete. If any source file uses C++23-only features, the iOS build will fail to compile.

### Inspection of Source Files
After reviewing the source files, the codebase primarily uses C++17/20 constructs (structured bindings, `std::optional`, range-based for, `auto`). C++23 is requested but not strictly required based on current code. The `CMakeLists.txt` itself notes C++23, but the actual code is compatible with C++20.

### Proposed Fix
Add a platform-specific override in `CMakeLists.txt`:

```cmake
# Apple Clang for iOS does not fully support C++23; cap at C++20 for iOS builds.
if (IOS OR ("${CMAKE_SYSTEM_NAME}" STREQUAL "iOS"))
    set(CMAKE_CXX_STANDARD 20)
else()
    set(CMAKE_CXX_STANDARD 23)
endif()
set(CMAKE_CXX_STANDARD_REQUIRED ON)
```

This preserves C++23 for Windows and Android, while using C++20 for iOS.

### Affects Other Platforms?
No. Only the iOS build path is changed.

### How to Test
Build succeeds without C++ standard errors on the iOS CI runner.

---

## Issue 3 — Self-Signed TLS Certificate and iOS App Transport Security (ATS)

### Files
- `mod.json` — default signaling URL: `https://194.226.126.115:8443`
- `src/net/NetworkConfig.cpp` — URL loaded from settings

### Problem
The signaling server at `194.226.126.115:8443` uses a **self-signed TLS certificate** (confirmed by changelog v0.5.1 which mentions switching to self-hosted signaling). iOS enforces **App Transport Security (ATS)** which rejects:
1. Connections to raw IP addresses (not domain names) over HTTPS with self-signed certificates
2. Connections that fail certificate validation

Since GD/Geode on iOS uses `NSURLSession` or `CFNetwork` underneath, and Geode's web utilities are built on top of platform HTTP APIs, the HTTPS connection to the signaling server **may fail on iOS** due to ATS.

### Additional Note
The WebRTC data channel itself (via `libdatachannel`) does not use the system HTTP stack for its DTLS/SRTP — it handles TLS internally via mbedTLS. Only the **signaling HTTP POST requests** go through Geode's `web::WebTask`, which uses the platform HTTP stack.

### Proposed Fix (Three Options, in order of preference)

**Option A (Best):** Configure the signaling server with a valid domain name and a CA-signed certificate (or Let's Encrypt). This eliminates the iOS ATS issue entirely.

**Option B:** Add `NSAppTransportSecurity` exception to the GD app's `Info.plist` via the iOS Launcher's patching mechanism. This requires coordination with the Geode iOS Launcher.

**Option C (Temporary):** Document as a known limitation and recommend using the **HTTP Relay** connection mode as fallback when WebRTC signaling fails. The HTTP Relay transport uses the same signaling URL but can function without direct DTLS.

**Recommended for prlz port:** Implement Option C as documentation, and investigate Option A with LichiGMDJ.

### Affects Other Platforms?
No. macOS, Windows, and Android are unaffected by iOS ATS.

### How to Test
On iOS, enable the HTTP Relay connection mode and verify rooms can be created and joined. Then test with WebRTC mode and observe whether the signaling connection succeeds.

---

## Issue 4 — Cursor Node Mouse-Position Semantics on Touch Devices

### Files
- `src/ui/CursorNode.cpp`
- `src/EditorHooks.cpp`

### Problem
The `CursorNode` displays remote player cursors based on `CCPoint` world coordinates from the editor. On PC, cursor position is derived from mouse events. On mobile (Android is already supported), Geode translates touch events to `CCPoint` via `CCTouchDispatcher`.

The original mod already fixed *"mobile player cursors drifting when panning the camera"* in v0.2.0, so Android touch handling is already implemented. iOS uses the same Cocos2d-x touch system as Android, so the same fix should apply.

### Why This Could Affect iOS
If any iOS-specific touch coordinate mapping differs from Android (e.g., due to safe area insets or screen resolution), cursors may appear slightly offset.

### Proposed Fix
No code changes required for initial port. The Android fix already covers the common mobile touch drift issue. Monitor for iOS-specific cursor drift during testing and add coordinate adjustments if needed.

### Affects Other Platforms?
No.

### How to Test
On an iOS device in a multiplayer session, pan the editor camera and verify that remote player cursors track correctly with the editor view.

---

## Issue 5 — `NO_WEBSOCKET ON` with WebSocket-Based Signaling

### File
`CMakeLists.txt`, lines 79–80

```cmake
set(NO_MEDIA ON CACHE BOOL "" FORCE)
set(NO_WEBSOCKET ON CACHE BOOL "" FORCE)
```

### Problem
`libdatachannel` is built with `NO_WEBSOCKET ON`, which disables the WebSocket module inside `libdatachannel`. However, the signaling mechanism uses HTTP long-polling (since v0.4.2) and POST requests via Geode's `web::WebTask`, **not** `libdatachannel`'s WebSocket. This is intentional and correct — `libdatachannel` is only used for WebRTC data channels, not for signaling WebSockets.

### Assessment
This is **not an issue**. The `NO_WEBSOCKET ON` flag is correct and intentional. No fix needed.

---

## Modified Files

### Files Changed for iOS Port

| File | Change Type | Description |
|---|---|---|
| `.github/workflows/multi-platform-release.yml` | **Modified** | Add iOS to build matrix |
| `CMakeLists.txt` | **Modified** | iOS C++ standard cap at C++20 |
| `mod.json` | **Modified** | Add iOS port attribution comment, bump to iOS port version |
| `about.md` | **Modified** | Add iOS port attribution |
| `CHANGELOG.md` | **New** | iOS port changelog |

### Files NOT Changed

All `src/` files are unchanged. The core C++, networking, protocol, editor hooks, and UI code requires no iOS-specific modifications.

---

## Remaining Limitations

1. **TLS certificate validation** — The self-signed certificate on the signaling server may cause ATS failures on iOS. Users may need to use HTTP Relay mode until a proper certificate is deployed.
2. **Physical device testing** — Compilation and packaging can be verified on CI, but actual multiplayer testing requires a physical iOS device with Geode's iOS Launcher installed.
3. **C++23 features** — If future source code additions use C++23-only APIs, the iOS build will need to be updated to handle them.
4. **Geode SDK version** — The mod requires Geode SDK 5.9.0. The iOS Launcher must be on a compatible version.

See `KNOWN_LIMITATIONS.md` for full details.
