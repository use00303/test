# Dependency licenses and source availability

AirMirror's own source is MIT-licensed (`LICENSE`). It is independently branded and not affiliated with Samsung, Google, Microsoft, VideoLAN or Genymobile. Their names identify supported devices/dependencies only. No trademark rights are granted.

## Shipped runtime components

| Component | Version | License / upstream |
|---|---|---|
| scrcpy client/server | 5.0.1 | Apache-2.0; https://github.com/Genymobile/scrcpy/tree/v5.0.1 |
| ADB and Windows API DLLs | platform-tools 37.0.1 | Android open-source components and included notices; https://developer.android.com/tools/releases/platform-tools |
| .NET / WPF desktop runtime | 10.0.13 in the current self-contained build | MIT and third-party notices; https://github.com/dotnet/runtime and https://github.com/dotnet/wpf |
| FFmpeg shared libraries | 9.0.2 per scrcpy's pinned dependency script | LGPL-2.1-or-later and component notices; https://ffmpeg.org/ |
| libusb | 1.0.30 | LGPL-2.1-or-later; https://github.com/libusb/libusb |
| SDL | 3.4.18 | zlib license and component notices; https://github.com/libsdl-org/SDL |
| dav1d (in FFmpeg) | 1.5.4 | BSD-2-Clause; https://code.videolan.org/videolan/dav1d |
| zlib / toolchain components | As used by official Windows binaries | Permissive component notices, source notices and upstream dependency build scripts |

Official unmodified scrcpy binaries are fetched from its release, not third-party download sites. The v5.0.1 Windows archive has SHA-256 `b12a2c4ee8be317422451fc7dcf8ee20a71b5ea7ef9ad73ddd825a25316227a5`. Its ADB/DLL bytes are compared with Google's official platform-tools 37.0.1 archive (SHA-256 `45f4d63113e895ebde0c90f194099a4676b6ac653bd28d54314a9e022bbc1a99`). Version inventory derives from the tagged upstream build scripts; those Windows executables have not been run in this Linux environment.

The full scrcpy Apache license is retained in `tools/scrcpy/LICENSE.txt`. The official platform-tools NOTICE and full dependency license/third-party notice texts are supplied under `tools/scrcpy/licenses`. Google's NOTICE covers the platform-tools distribution broadly; AirMirror ships only the official ADB components from it, not the entire SDK. Their applicable open-source license terms remain controlling.

## Corresponding source and replacement

The installer and portable distribution accompany the unmodified libraries with full upstream release source archives at `tools/scrcpy/sources`:

- `ffmpeg-9.0.2.tar.xz`
- `libusb-1.0.30.tar.gz`
- `dav1d-1.5.4.tar.xz`
- `SDL-3.4.18.tar.gz`
- `scrcpy-v5.0.1.tar.gz` (including `app/deps/*`, cross-build scripts, configuration and dependency provenance)

The manifest `tools/scrcpy/provenance.json` records original URLs and verified/calculated SHA-256 hashes. For the exact library build configuration, extract the scrcpy source archive and consult `app/deps/ffmpeg.sh`, `libusb.sh`, `sdl.sh`, `dav1d.sh`, `_init` and the release/build documentation. FFmpeg's tagged script does not enable optional GPL/nonfree configurations. The archive includes applicable component copyright/license notices.

Libraries remain dynamically replaceable in the tool directory. Nothing in AirMirror's license restricts modifying/replacing LGPL components or reverse engineering for debugging modifications as permitted by their licenses. Replacement builds must remain ABI-compatible with the client; the tool override in Settings can point to a complete replacement folder. No DRM or signature lock prevents replacement. The build-time dependency preparer stages the source archives inside the output rather than relying on a future written source offer.

For any public redistribution, retain all notices and corresponding sources, verify the final actual binary dependency inventory, and review licensing obligations for any changed dependency build flags. This inventory is not legal advice. Do not apply AirMirror's MIT license to third-party source/binaries or strip their notices.

## Build/test tools (not application runtime dependencies)

- .NET SDK/MSBuild/Roslyn: Microsoft/.NET open-source terms.
- xUnit 2.9.3 and Visual Studio runner 2.8.2: Apache-2.0; https://github.com/xunit/xunit.
- Microsoft.NET.Test.Sdk 17.14.1 / VSTest: MIT; https://github.com/microsoft/vstest.
- NSIS 3: compiler primarily zlib/libpng with documented component terms; https://nsis.sourceforge.io/License. NSIS's license explicitly permits generated installers to be used/distributed under the desired license; NSIS compiler itself is not bundled.
- Python 3: developer-only dependency. The app never invokes Python.

Third-party sources and bundled .NET runtime notices remain available after installation in the application tools directory. The installer does not grant rights beyond the respective licenses.
