# CHANGELOG.md — MultiplayerEditX iOS Port by prlz

> **iOS Port Developer:** prlz  
> **Original Project:** MultiplayerEditX by LichiGMDJ

---

## iOS Port History

### iOS Port — v0.5.3 (November 2026)
**Port Developer:** prlz  
**Base:** MultiplayerEditX v0.5.3 by LichiGMDJ

This is the initial iOS port of MultiplayerEditX v0.5.3. The goal of this port is to produce a working iOS binary and package it into the `.geode` file, enabling the mod to load on iOS via the Geode iOS Launcher.

#### Changes Made by prlz (iOS Port)

**`.github/workflows/multi-platform-release.yml`**
- Added `iOS` to the build matrix using a `macos-latest` runner (required for iPhone SDK access)
- Updated the package job name to reflect Win64 + Android64 + iOS output
- Updated the artifact name to `lichigmdj.multiplayereditx-0.5.3-ios-prlz`
- Updated the release name to `Multiplayer Edit X v0.5.3 (iOS Port by prlz)`
- Added a conditional skip for the Python source-verification step on the iOS matrix entry (the verification checks Win64/Android64-specific assertions and would fail on the macOS runner if run unnecessarily)

**`CMakeLists.txt`**
- Changed the C++ standard selection from a fixed `CMAKE_CXX_STANDARD 23` to a conditional:
  - iOS builds: C++20 (Apple Clang for iOS does not fully support C++23)
  - All other platforms: C++23 (unchanged from original)
- No other CMake changes — the iOS architecture guard (`arm64`) was already correctly implemented by LichiGMDJ

#### Changes NOT Made by prlz

The following are **unchanged** from the original MultiplayerEditX v0.5.3:
- All source code in `src/` (zero modifications)
- `mod.json` (iOS was already declared in `gd.ios`)
- Networking protocol (binary protocol v8, unchanged)
- Editor hooks
- UI implementation
- Server code in `server/`
- Dependencies (mbedTLS, libdatachannel)

#### Known Limitations Added by iOS Port

- None added beyond what already existed. See `KNOWN_LIMITATIONS.md`.
- The self-signed TLS certificate issue on the signaling server is a pre-existing limitation, not introduced by the iOS port. Users should use HTTP Relay mode on iOS until a CA-signed certificate is deployed.

#### Attribution

- Original source: https://github.com/LichiGMDJ/MultiplayerEditX
- Original developer: LichiGMDJ
- Original upstream: xXoanon (MultiplayerEdit)
- iOS port changes: prlz

---

## Original MultiplayerEditX Changelog

The following is the original changelog from LichiGMDJ's MultiplayerEditX. It is reproduced here for reference. All credits go to LichiGMDJ and the original contributors.

---

### 0.5.2
- Added safety guards around WebRTC data-channel sends so oversized editor operations cannot terminate Geometry Dash.
- Large `PlaceObjects` payloads are split into safe reliable-channel batches; oversized single objects are skipped with a warning instead of crashing the game.
- Added inbound packet, incoming queue, and pending-message limits to reduce malformed-packet and runaway-queue risk.
- Added exception containment around network message handlers so malformed remote data cannot escape into the game process.
- Added protocol compatibility handshake (`ProtocolHello`, protocol v1). Editor traffic is ignored until the peer verifies a compatible protocol version.
- Added player music display next to remote cursors.
- Switched the custom build to the self-hosted signaling server at `194.226.126.115:8443` and self-hosted TURN/UDP on port 3478.
- TURN credentials are no longer embedded into public build artifacts.
- Removed normal-log SDP dumps that could expose ICE/network details.
- Added CI checks for the safety guards, protocol handshake, TURN replacement, and music-label patch before compiling the Windows artifact.

### 0.4.4
- Start Position properties are now fully synced between players.
- The multiplayer lobby UI now updates the player count and names in real time.
- Improved playtesting sync and added support for mini and dual.
- Fixed a crash that occurred when joining a host (memory issue with teleport portals).
- Fixed a bug where players would sometimes duplicate in the lobby.
- Fixed a crash that occurred when a guest unexpectedly closes the game.
- Fixed an issue where the game would get stuck on "Waiting for level sync from host..."
- Significantly optimized level syncing on massive levels by compressing level data.
- Fixed object ID desyncs for pre-existing level objects when multiple players join.

### 0.4.3
- Fixed a bug where placing, moving, or deleting teleport portals would cause game crashes and duplicate portal desyncs.
- Drastically reduced signaling server load by optimizing polling.

### 0.4.2
- Reduced overlay opacity on objects selected by other players.
- Fixed object desync when quickly deselecting.
- Fixed crash when joining a host.
- Improved cursor accuracy.
- Replaced WebSocket signaling with HTTP long polling.
- Removed "exception based" error handling with error flags.
- Fixed a bug regarding data channel sends.
- Disabled libdatachannel's WebSocket module to reduce binary size.
- Fixed a race condition in MessageBatcher.
- Added a Patreon donate button.

### 0.4.1
- Fixed desync issues with color channels and property edits.
- Fixed multi object rotation being bugged.
- Added a buffer queue to hopefully reduce dropped messages.
- Fixed severe desync and level corruption issues when receiving remote edits while actively playtesting.
- Fixed a race condition window when building fast.
- Replaced HTTP polling-based signaling with a proper WebSocket relay.

### 0.4.0
- Completely overhauled networking — switched from central WebSocket relay to P2P via WebRTC data channels.
- Added a lightweight signaling server (Deno Deploy) for initial matchmaking.
- Replaced JSON message format with a compact binary wire protocol.
- Chunked initial level sync for large levels.
- **Fixed the iOS binary (`.ios.dylib`) missing from the `.geode` package** (note: this was later removed from CI again in subsequent versions; restored by prlz iOS port).
- Fixed objects occasionally snapping back to original position after rapid edits.

### 0.3.0
- Fixed the synchronizing level screen never loading bug when joining a session.
- Fixed level colors glitching out after joining or when the host changes colors.
- Multiple desyncs and crash fixes.
- Improved server stability.
- Copy/paste and duplicate now sync as one batched message.

### 0.2.3 / 0.2.2 / 0.2.1
- Networking stability improvements and crash fixes.

### 0.2.0
- Added proper support for macOS and Android.
- Fixed TLS handshake connection failures on Android and macOS.
- Fixed selector-based scheduler crashes in the MultiplayerPopup UI.
- Added a 30 second ping interval heartbeat.
- Fixed mobile player cursors drifting when panning the camera.
- Multiple crash and sync fixes.

### 0.1.1
- Fixed an EXCEPTION_ACCESS_VIOLATION crash on Windows/Wine caused by using `schedule_selector` in `$modify` wrapper classes.

### 0.1.0
- Initial release. Real-time multiplayer level editing with synchronized object placement, deletion, movement, scaling, rotation. Isolated undo/redo stacks, live player cursors, badge previews, live playtesting, in-editor multiplayer panel.
