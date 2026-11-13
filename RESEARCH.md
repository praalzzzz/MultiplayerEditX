# RESEARCH.md — MultiplayerEditX iOS Port by prlz

> **iOS Port Developer:** prlz  
> **Original Project:** MultiplayerEditX by LichiGMDJ  
> **Research Date:** November 2026

---

## 1. Original Repository

- **URL:** https://github.com/LichiGMDJ/MultiplayerEditX
- **Original Author:** LichiGMDJ
- **Upstream Origin:** Fork of [xXoanon/MultiplayerEdit](https://github.com/xXoanon/MultiplayerEdit)
- **License:** To be confirmed from repository (original credits preserved per license)
- **Mod ID:** `lichigmdj.multiplayereditx`
- **Mod Name:** Multiplayer Edit X
- **Current Version:** 0.5.2

---

## 2. Inspected .geode Package (`lichigmdj.multiplayereditx.geode`)

### Package Contents

| File | Size | Notes |
|---|---|---|
| `mod.json` | — | Metadata, declares iOS support |
| `about.md` | 1,889 bytes | Feature description |
| `changelog.md` | 9,459 bytes | Full version history |
| `logo.png` | 6,675 bytes | Mod icon |
| `.geode_cache` | 30 bytes | Build cache marker |
| `lichigmdj.multiplayereditx.dll` | ~3.1 MB | **Windows x86_64 binary** |
| `lichigmdj.multiplayereditx.android64.so` | ~4.9 MB | **Android arm64 binary** |
| ~~`lichigmdj.multiplayereditx.ios.dylib`~~ | MISSING | **iOS arm64 binary — NOT PRESENT** |

### mod.json Key Fields

```json
{
  "geode": "5.7.1",
  "gd": {
    "win": "2.2081",
    "android": "2.2081",
    "mac": "2.2081",
    "ios": "2.2081"
  },
  "id": "lichigmdj.multiplayereditx",
  "name": "Multiplayer Edit X",
  "version": "0.5.2",
  "developer": "LichiGMDJ"
}
```

**Critical finding:** The mod declares `"ios": "2.2081"` compatibility in `mod.json`, but the `.geode` package does **not** include an `ios.dylib` binary. The mod cannot load on iOS.

### Networking Configuration

The mod uses WebRTC (libdatachannel) with:
- **Signaling server:** `https://194.226.126.115:8443` (HTTPS + self-signed TLS)
- **TURN relay:** `194.226.126.115:3478` (UDP)
- **Protocol:** WebRTC P2P data channels, binary wire protocol (v1)
- **Fallback:** TURN relay for NAT traversal

---

## 3. Source Code Analysis

### Architecture Overview

MultiplayerEditX is a Geode mod built in C++17 that hooks into Geometry Dash's LevelEditorLayer and related classes to enable real-time collaborative editing.

**Primary Components:**
- **Networking layer:** WebRTC data channels via `libdatachannel`
- **Signaling:** HTTP long-polling → WebSocket (upgraded in v0.4.1)
- **Wire protocol:** Binary format (replaced JSON in v0.4.0), with protocol v1 handshake (`ProtocolHello`) added in v0.5.1
- **Editor hooks:** `$modify` macros on `LevelEditorLayer`, `EditorUI`, `GameObject`, and related GD classes
- **UI:** Geode-native `Popup` and `CCNode` subclasses for the multiplayer menu and HUD
- **Object sync:** UUID-based bidirectional map tracking GD object IDs to network UUIDs

**Key Source Files (from repository structure):**
- `src/main.cpp` — Mod entry point, hook registration
- `src/MultiplayerManager.cpp/.hpp` — Core networking manager (WebRTC peer lifecycle, message dispatch)
- `src/EditorHooks.cpp/.hpp` — `$modify` hooks on GD editor classes
- `src/MultiplayerPopup.cpp/.hpp` — Room creation/join UI
- `src/HUDOverlay.cpp/.hpp` — In-editor status HUD
- `src/Protocol.hpp` — Binary wire protocol definitions
- `src/MessageBatcher.cpp/.hpp` — Batched message queue for editor operations
- `CMakeLists.txt` — Geode mod build configuration
- `.github/workflows/build.yml` — CI build matrix (Windows + Android only in recent builds)

**Dependencies:**
- **Geode SDK** v5.7.1 — Cross-platform GD mod framework
- **libdatachannel** — WebRTC + WebSocket implementation (C++17, bundled as submodule)
- **zlib/libz** — Level data compression (used since v0.4.4)
- **OpenSSL / mbedTLS** — TLS for WebSocket signaling (platform-dependent)

### Networking Deep-Dive

The mod uses `libdatachannel` for WebRTC:
- Peer connections are established via WebSocket signaling
- ICE candidates and SDP are exchanged through the signaling server
- Data channel carries the binary multiplayer protocol
- TURN relay is used as fallback for symmetric NAT

**iOS Networking Compatibility:**
- `libdatachannel` explicitly supports iOS via `ios-cmake` toolchain
- WebRTC data channels work on iOS (used in many iOS apps)
- No platform-specific network code that blocks iOS
- HTTPS + TLS self-signed certificates: iOS requires App Transport Security (ATS) exemption or proper certificate trust

### Changelog — iOS History

- **v0.4.0:** "Fixed the iOS binary (.ios.dylib) missing from the .geode package by adding the iOS platform to the automated GitHub Actions build matrix"
- **v0.2.0:** "Added proper support for macOS and Android. Fixed TLS handshake connection failures on Android and macOS."
- **v0.5.1 (current):** CI builds only produce Windows artifact — iOS was dropped again from the matrix

---

## 4. Geode iOS Support Research

### Current Status (2025-2026)

- Geode **officially supports iOS** as a target platform
- iOS mods compile as `arm64` `.dylib` files
- Installed via the [Geode iOS Launcher](https://github.com/geode-sdk/ios-launcher) (based on LiveContainer)
- Supports iOS 14+ via TrollStore, SideStore, or jailbreak
- JIT required for some mods; many work JIT-less with proper signing

### Build Requirements

| Requirement | Detail |
|---|---|
| **macOS** | Required (Apple SDK is macOS-only) |
| **Xcode** | Required for iPhone SDK |
| **Geode CLI** | `geode sdk install-binaries --platform ios` |
| **Target arch** | `arm64` only (not universal) |
| **CMake** | 3.21+ |
| **C++ standard** | C++17 |
| **Alternative** | Docker container `ghcr.io/geode-sdk/geode:ios-X.Y.Z` on Linux/Windows |

### GitHub Actions Path (Recommended)

The official `geode-sdk/build-geode-mod@main` GitHub Action supports iOS via matrix:
```yaml
- name: iOS
  os: macos-latest
  target: iOS
```
This is the same approach used by the mod itself in v0.4.0 (since reverted).

---

## 5. iOS Compatibility Findings

### Root Cause: Missing iOS Binary

The **sole reason** the mod does not work on iOS is that the iOS `.dylib` binary is missing from the compiled `.geode` package. The source code **already has iOS support** (confirmed by the v0.4.0 changelog entry and the `mod.json` iOS declaration).

### Secondary Issues to Verify

1. **CI workflow**: The `.github/workflows/build.yml` does not include iOS in the build matrix (dropped since v0.5.x)
2. **CMakeLists.txt**: May be missing iOS architecture guard (`CMAKE_OSX_ARCHITECTURES = arm64`)
3. **libdatachannel submodule**: Must be initialized and built for iOS (`ios-cmake` toolchain)
4. **TLS certificates**: The signaling server uses a self-signed certificate — iOS ATS may reject it unless the mod configures `NSAllowsArbitraryLoads` or certificate pinning
5. **Touch UI**: The original mod was designed with mouse input in mind; Geode handles basic touch abstraction but cursor-based UI may need minor adjustments

### What Does NOT Need Changing

- Core multiplayer protocol — identical on all platforms
- WebRTC peer logic — `libdatachannel` is cross-platform
- Editor hooks (`$modify`) — Geode handles platform abstraction
- Room creation/join UI — Geode UI is cross-platform
- Binary wire protocol — same on all platforms
- Signaling server — same endpoint for all platforms

---

## 6. Conclusion

MultiplayerEditX v0.5.2 declares iOS support but ships without the iOS binary. The porting task is primarily:

1. **Restore the iOS CI build target** in `.github/workflows/build.yml`
2. **Fix `CMakeLists.txt`** for proper iOS architecture handling
3. **Verify libdatachannel** builds correctly for iOS
4. **Address TLS/ATS** for the self-signed signaling server certificate
5. **Minor touch input adjustments** for iOS usability
6. **Re-enable iOS in the build matrix** and produce the missing `.ios.dylib`

The networking protocol, multiplayer logic, and cross-platform compatibility with Windows/Android users is preserved as-is — no protocol changes are needed.
