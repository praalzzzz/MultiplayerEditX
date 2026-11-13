# BUILDING.md — MultiplayerEditX iOS Port by prlz

> **iOS Port Developer:** prlz  
> **Original Project:** MultiplayerEditX v0.5.3 by LichiGMDJ

---

## Overview

MultiplayerEditX is a Geode SDK mod built with CMake and C++. iOS builds require a macOS machine with Xcode — Apple does not license its iPhone SDK for non-macOS hosts. The recommended build path for the iOS port is **GitHub Actions** using the official `geode-sdk/build-geode-mod` action.

---

## 1. Prerequisites

### All Platforms
| Tool | Version | Purpose |
|---|---|---|
| Git | Any recent | Cloning repository and submodules |
| CMake | 3.21+ | Build system |
| Python | 3.8+ | CI verification scripts |
| Geode CLI | Latest | SDK management, packaging |

### Windows (Win64 build)
| Tool | Version |
|---|---|
| Visual Studio 2022 | 17.x with MSVC |
| Geode SDK | 5.9.0 |

### Linux / Ubuntu (Android64 build)
| Tool | Version |
|---|---|
| Android NDK | r25c+ |
| Clang cross-compiler | Via NDK |
| Geode SDK | 5.9.0 |

### macOS (iOS build — **required for iOS**)
| Tool | Version |
|---|---|
| macOS | 13 Ventura or 14 Sonoma |
| Xcode | 15 or 16 |
| iPhone SDK | Bundled with Xcode |
| Apple Clang | Bundled with Xcode |
| Geode SDK | 5.9.0 |
| Geode CLI | Latest |

> [!IMPORTANT]
> **iOS compilation requires macOS with Xcode.** This is an Apple restriction — the iPhone SDK is not available on Windows or Linux. There is no workaround that avoids this requirement. If you do not have access to a Mac, use GitHub Actions (see Section 4).

---

## 2. Cloning the Repository

```bash
git clone https://github.com/LichiGMDJ/MultiplayerEditX
cd MultiplayerEditX
```

Dependencies (mbedTLS, libdatachannel) are fetched automatically by CMake's `FetchContent` during the configure step. No manual submodule initialization is needed.

---

## 3. Building Locally

### Windows (Win64)

```bat
# Set GEODE_SDK environment variable
set GEODE_SDK=C:\path\to\geode-sdk

cmake -B build-win -DCMAKE_BUILD_TYPE=Release
cmake --build build-win --config Release
```

### macOS (native macOS build)

```bash
export GEODE_SDK=/path/to/geode-sdk

cmake -B build-mac \
  -DCMAKE_BUILD_TYPE=Release \
  -DCMAKE_OSX_ARCHITECTURES="arm64;x86_64"

cmake --build build-mac --config Release
```

### macOS (iOS cross-compile — **this is the iOS port build**)

```bash
export GEODE_SDK=/path/to/geode-sdk

cmake -B build-ios \
  -DCMAKE_SYSTEM_NAME=iOS \
  -DGEODE_TARGET_PLATFORM=iOS \
  -DCMAKE_OSX_ARCHITECTURES=arm64 \
  -DCMAKE_BUILD_TYPE=Release \
  -DCMAKE_POLICY_VERSION_MINIMUM=3.5

cmake --build build-ios --config Release
```

The Geode CMake integration (`setup_geode_mod`) will produce `lichigmdj.multiplayereditx.geode` containing the `arm64` `.ios.dylib`.

### Android (cross-compile from Linux/macOS)

```bash
export GEODE_SDK=/path/to/geode-sdk
export ANDROID_NDK=/path/to/android-ndk

cmake -B build-android64 \
  -DCMAKE_TOOLCHAIN_FILE=$ANDROID_NDK/build/cmake/android.toolchain.cmake \
  -DANDROID_ABI=arm64-v8a \
  -DANDROID_PLATFORM=android-23 \
  -DCMAKE_BUILD_TYPE=Release

cmake --build build-android64 --config Release
```

---

## 4. Building via GitHub Actions (Recommended for iOS)

This is the **recommended and verified** iOS build path. It uses the official Geode build action on a macOS runner which has Xcode and the iPhone SDK pre-installed.

### Setup

1. Fork or clone the repository to your own GitHub account
2. The workflow file is at `.github/workflows/multi-platform-release.yml`
3. The iOS build matrix entry is already added by the prlz iOS port

### Trigger a Build

**Manual trigger (any branch):**
- Go to your repository → Actions → `Build Multiplayer Edit X` → `Run workflow`

**Automatic trigger:**
- Push to `main` branch
- Create a tag starting with `v` (triggers a release)

### Workflow Matrix

```yaml
matrix:
  include:
    - name: Windows64
      os: windows-latest
      target: Win64
    - name: Android64
      os: ubuntu-latest
      target: Android64
    - name: iOS                   # ← Added by prlz iOS port
      os: macos-latest
      target: iOS
```

### Expected Output

The `Package` job combines all three builds and produces a single `.geode` file containing:
- `lichigmdj.multiplayereditx.dll` (Windows)
- `lichigmdj.multiplayereditx.android64.so` (Android)
- `lichigmdj.multiplayereditx.ios.dylib` (iOS — added by this port)

---

## 5. Geode SDK Setup

### Install Geode CLI

```bash
# macOS (Homebrew)
brew install geode-sdk/tap/geode-cli

# Windows (Scoop)
scoop bucket add geode https://github.com/geode-sdk/scoop.git
scoop install geode-cli

# Or download binary from: https://github.com/geode-sdk/cli/releases
```

### Install SDK Binaries

```bash
geode sdk install-binaries --platform ios
geode sdk install-binaries --platform win
```

### Set Environment Variable

```bash
# macOS/Linux
export GEODE_SDK=$(geode sdk path)

# Windows
set GEODE_SDK=C:\Users\<you>\AppData\Local\GeodeSDK
```

---

## 6. Dependencies

All dependencies are fetched automatically via CMake `FetchContent`:

| Dependency | Version | Source |
|---|---|---|
| mbedTLS | 3.6.0 | `https://github.com/Mbed-TLS/mbedtls.git` |
| libdatachannel | 0.22.4 | `https://github.com/paullouisageneau/libdatachannel.git` |
| Geode SDK | 5.9.0 | Via `GEODE_SDK` env var |

### libdatachannel Build Flags

```cmake
NO_MEDIA      ON   # Data channels only, no audio/video
NO_WEBSOCKET  ON   # Signaling uses Geode web utilities, not libdatachannel WS
USE_MBEDTLS   ON   # mbedTLS handles DTLS/SRTP
USE_GNUTLS    OFF
USE_NICE      OFF
```

---

## 7. CMake Configuration Notes

### iOS Architecture

The `CMakeLists.txt` automatically sets `arm64` for iOS builds:

```cmake
if (APPLE)
    if ("${CMAKE_SYSTEM_NAME}" STREQUAL "iOS" OR IOS)
        set(CMAKE_OSX_ARCHITECTURES "arm64")
    else()
        set(CMAKE_OSX_ARCHITECTURES "arm64;x86_64")
    endif()
endif()
```

### C++ Standard

iOS builds use C++20 (Apple Clang limitation); all other platforms use C++23:

```cmake
if (IOS OR ("${CMAKE_SYSTEM_NAME}" STREQUAL "iOS"))
    set(CMAKE_CXX_STANDARD 20)
else()
    set(CMAKE_CXX_STANDARD 23)
endif()
```

---

## 8. Troubleshooting

### `FetchContent` fails (no network in sandbox)
GitHub Actions has network access. Local builds need internet during the first CMake configure. After that, CMake caches the fetched sources.

### `GEODE_SDK` not found
Make sure the environment variable is set before running CMake:
```bash
export GEODE_SDK=$(geode sdk path)
```

### iOS build: "iPhone SDK not found"
You must be running on macOS with Xcode installed. Run:
```bash
xcode-select --install
sudo xcode-select -s /Applications/Xcode.app/Contents/Developer
```

### mbedTLS DTLS SRTP patch fails
The CMakeLists.txt patches `mbedtls_config.h` during configure. If the patch string is not found (mbedTLS version change), check that the target string in `CMakeLists.txt` matches the content of `mbedtls_config.h` in mbedTLS v3.6.0.

### iOS build: C++23 errors
If new source files use C++23-only features, they will fail on iOS (C++20). Use `#if __cplusplus >= 202302L` guards or refactor to C++20 equivalents.
