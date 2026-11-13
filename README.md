# Multiplayer Edit

<img src="logo.png" width="150" alt="Multiplayer Edit logo" />

Real-time collaborative level editing for Geometry Dash on **Windows x64, Android 64-bit, and iOS**.

This project is based on the original Multiplayer Edit mod and the MultiplayerEditX fork. The iOS port is developed and maintained by **prlz**.

The project uses Protocol v6 with reliable edit delivery, reconnect support, global shared-state recovery, Object Workshop synchronization, host-only music changes, host kick controls, and Room Settings.

## Project & Development

- **Original Multiplayer Edit:** Created by xXoanon / d050.
- **MultiplayerEditX:** Further development and improvements by LichiGMDJ.
- **iOS Port:** Ported and developed by prlz.
- **AI Assistance:** AI tools were used as development assistants for code analysis, research, debugging, compatibility investigation, and implementation support.

The iOS port is a continuation of the existing project, not a replacement or a claim of original authorship.

## Features

- **Real-Time Collaborative Editing** — host a session, share the 6-character room code, and build together.
- **Cross-Platform Multiplayer** — supported Windows, Android, and iOS clients can join the same room when using compatible builds.
- **Reliable Synchronization** — placement, deletion, movement, transform, settings and bulk operations use reliable delivery and retry logic.
- **Authoritative First Join Sync** — a newly connected player receives the current level before normal collaborative edits begin.
- **Global Shared State** — the room converges to one shared level state even when several players edit.
- **Object Workshop Sync** — RAW paste data and absolute placement are synchronized between players.
- **Host-only Music Changes** — the host controls the global level music and guests receive a notification when it changes.
- **Room Settings** — host controls max players, guest permissions, Auto Repair, room lock, Force TURN diagnostics and kick controls.
- **Reconnect + Recovery** — reconnect handling and integrity recovery help restore the room after temporary connection loss.
- **Cross-Platform Protocol** — compatible clients use the same multiplayer protocol.

## Installation

### Windows

1. Install Geode for Geometry Dash 2.2081.
2. Download the Win64 `.geode` file from this repository's Releases page.
3. Put the `.geode` file into your Geometry Dash `geode/mods/` folder.
4. Restart Geometry Dash.

### Android

1. Install the Android64 version of Geode for Geometry Dash 2.2081.
2. Download the Android64 `.geode` file from the Releases page.
3. Copy the actual `.geode` file into the Android Geode `mods` folder. Do not install the GitHub Actions ZIP itself.
4. Restart Geometry Dash completely.
5. Open Geode's mod list and confirm that **Multiplayer Edit** is enabled.

### iOS

The iOS port is developed by **prlz**.

1. Make sure your iOS Geometry Dash version is supported by the available Geode iOS environment.
2. Install and configure a compatible Geode iOS installation.
3. Download the iOS `.geode` package from the Releases page when an iOS release is available.
4. Install the package using the appropriate Geode iOS method.
5. Restart Geometry Dash.
6. Open Geode's mod list and confirm that **Multiplayer Edit** is enabled.

iOS availability depends on the supported Geode version, Geometry Dash version, and the release packages provided.

## Connecting

### Host

1. Open a level in the editor.
2. Open **Multiplayer Edit**.
3. Press **Host**.
4. Share the generated 6-character room code.

### Guest

1. Open **Multiplayer Edit**.
2. Press **Join**.
3. Enter the host's room code.
4. Wait for the initial level synchronization to finish.

Players can connect across supported platforms:

- Windows host -> Windows guest
- Windows host -> Android guest
- Android host -> Windows guest
- Android host -> Android guest
- Windows host -> iOS guest
- iOS host -> Windows guest
- Android host -> iOS guest
- iOS host -> Android guest
- iOS host -> iOS guest

Cross-platform combinations require compatible mod builds and protocol versions.

## Android / iOS <-> PC Networking

The mod uses WebRTC data channels.

Normal connection mode is **automatic ICE selection**:

1. Direct/local and STUN-discovered routes are preferred.
2. If a direct route is not usable and TURN credentials are configured, TURN is available as a relay candidate.
3. **Force TURN** is intended only for diagnostics and should normally remain disabled.

Players do not need to enable Force TURN for ordinary sessions.

### If Join stays on `Joining`

First verify that both devices are using compatible mod versions and that the room code is correct.

Then open the Multiplayer Edit settings and verify:

- **Signaling Server URL** points to the multiplayer signaling server.
- If your network requires a relay, TURN host, username and password are configured.
- **Force TURN Relay** should normally be OFF. Enable it only to test whether a strict NAT/CGNAT is preventing a direct connection.

For diagnostics, check the Geode log for `P2PManager:` lines. Useful milestones are:

- `Joined room ... as player ...`
- `Received host's SDP offer`
- `Reliable channel ... opened`
- `Unreliable channel ... opened`
- `protocol ... verified`
- `requested authoritative initial sync from host`

If signaling succeeds but data channels never open, the problem is normally in ICE/NAT traversal rather than the room-code server.

## Room Settings

The host can open **Room Settings** and control:

- Max players
- Allow guests to build/edit
- Allow guests to delete
- Allow guests to use Object Workshop
- Allow guests to change level settings
- Auto Repair
- Force TURN diagnostic mode
- Kick players
- Lock room

Locking a room prevents new joins but does not remove already connected players.

## Connection Settings

Current builds support a configurable signaling URL and TURN server settings through Geode mod settings.

The signaling server is used only to exchange WebRTC negotiation data. Level editing traffic is sent through WebRTC data channels after the connection is established.

## Build Instructions

The original MultiplayerEditX project uses Geode SDK 5.7.1 and targets Geometry Dash 2.2081.

```sh
# Clone the original MultiplayerEditX repository
git clone https://github.com/LichiGMDJ/MultiplayerEditX.git
cd MultiplayerEditX

# Windows
geode build --platform win

# Android 64-bit
geode build --platform android64
```

The iOS port by **prlz** may require additional iOS-specific build configuration, SDK components, and macOS/Xcode tooling.

Refer to the iOS build documentation and platform-specific instructions before attempting an iOS build.

The repository also contains GitHub Actions workflows for Win64 and Android64 builds.

## Release Checklist

Before publishing a release, verify at minimum:

- Win64 build succeeds.
- Android64 build succeeds.
- iOS build succeeds, if an iOS release is included.
- Windows host -> Android guest connects.
- Android host -> Windows guest connects.
- iOS clients connect to supported Windows and Android clients.
- Normal object placement/movement/deletion sync works.
- Object Workshop paste position matches on both devices.
- Room Settings permissions are enforced.
- Host music changes propagate to guests.
- Reconnect and initial level synchronization work.
- Cross-platform compatibility has been tested.

Only mark a platform as supported after its build and functionality have been verified.

## Credits & Acknowledgements

### Original Project

**Multiplayer Edit** was originally created by **xXoanon / d050**.

### MultiplayerEditX

The MultiplayerEditX fork builds upon the original project and includes additional networking, synchronization, Room Settings, and cross-platform development by **LichiGMDJ**.

Original project and fork contributions remain credited to their respective developers.

### iOS Port

The iOS port was developed by **prlz**, who worked on adapting the existing MultiplayerEditX project for iOS compatibility while preserving its original multiplayer functionality.

### AI-Assisted Development

AI tools were used as development assistants during the iOS porting process, including:

- Source code analysis.
- Technical research.
- iOS compatibility investigation.
- Code implementation assistance.
- Debugging and troubleshooting.
- Documentation assistance.

AI tools assisted with development, but the iOS port's development and integration are credited to **prlz**.

### Thank You

Thanks to the original developers and contributors for creating and maintaining the projects that made this iOS port possible.

All original code, contributions, and licenses remain attributed to their respective authors.
