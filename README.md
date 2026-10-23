# AirMirror Desktop — 0.1.0 validation build

A real Windows 10/11 x64 WPF application for local Android wireless pairing, connection management and a managed scrcpy mirror window. No USB is required for the Android 11+ Wireless debugging workflow. Primary hardware target: **Samsung Galaxy F15 5G**; its Android/One UI version must be detected on connection, not assumed.

**Release status:** source implementation, automated cross-platform service tests and Windows cross-compilation are available. Windows launch, installation and real-phone operation have not been validated in this Linux development environment. This is a validation build, not a hardware-certified final release. See `HARDWARE-TESTS.md` for release gates.

## Install and run on Windows

1. Download `AirMirror-Setup-0.1.0-win-x64.exe`, verify it against the accompanying SHA256SUMS file, and run it on Windows 10/11 x64. The installer is currently unsigned; no trusted publisher signature is claimed. Follow your organization's execution policy rather than disabling security protections.
2. It installs for the current user under `%LOCALAPPDATA%\Programs\AirMirror`, creates Start menu/desktop shortcuts, and registers an uninstaller. Administrator rights, firewall changes and PATH modifications are not requested. Close AirMirror/scrcpy before updating or uninstalling.
3. Launch **AirMirror Desktop** from Start. The installer includes the .NET 10 desktop runtime, scrcpy 5.0.1, official ADB 37.0.1, DLLs, license notices and corresponding third-party library sources. No separate .NET installation or Python is needed to run it.
4. A portable ZIP is also supplied: extract the **entire** directory and run `AirMirror.exe`. Do not move the executable away from its runtime or `tools` directory.
5. In Settings, choose **Validate & check tool versions**. A custom tool directory overrides bundled tools; leave it blank normally.

Installation and first launch still require the Windows checklist below. This native app is not a browser app: there is no web preview, HTTP API, production website or Cloudflare deployment.

## First-time wireless pairing — two different ports

Use only a private, trusted Wi-Fi network. ADB development access is powerful. Never forward debugging ports through a router, expose them publicly, disable the firewall globally, or enter your phone PIN into AirMirror.

1. Join the phone and Windows PC to the same trusted Wi-Fi.
2. On Samsung: **Settings → About phone → Software information → Build number**; tap seven times. Enter any requested device PIN **on the phone**. Return to **Settings → Developer options → Wireless debugging**; enable it and allow this Wi-Fi.
3. In AirMirror choose **Pair a new device**. On the phone choose **Pair device with pairing code**. Keep that dialog open.
4. Enter that dialog's private IP and **pairing port** into *Pairing IP:port*, and its current six-digit code into the masked code box. Choose **Pair device**. The app runs official `adb pair` with the code supplied over stdin, not a command-line argument. Codes are never persisted/logged.
5. Close the phone's pairing dialog. The **main Wireless debugging page** shows its IP and a separate **connection port**. Enter this in Mirror → *Connection IP:port* → **Connect**. Do not reuse the pairing port unless Android actually displays the same value.
6. The UI reports device identity and Android release/API level only after ADB confirms it online and returns the relevant properties. Alternatively, Refresh ADB devices, explicitly select an online device and choose **Use selected**. Devices are never chosen arbitrarily when multiple are present.
7. **Discover mDNS endpoints** lists official `_adb-tls-connect._tcp` advertisements, not pairing services or a subnet scan. Verify the address on the phone and explicitly select it. Discovery does not pair/connect automatically. QR pairing is not implemented; pairing-code assistance is implemented.
8. **Start mirroring** opens a **separate** AirMirror-branded scrcpy window. This release does not embed video in the WPF control panel. Mouse/keyboard control is provided by scrcpy, not reimplemented. A window opening is not proof that secure content can be captured: inspect live video on your device.

Wireless Debugging requires Android 11/API 30+. If the option is missing or blocked by device policy, this Wi-Fi-only setup cannot be guaranteed. No root, USB-to-TCP bootstrap or security bypass is performed.

## Everyday operation

- Connect with the main phone screen's current connection port, or explicitly select the already-connected ADB device, then Start mirroring.
- The scrcpy window receives mouse/keyboard input. International/IME text behavior follows scrcpy/Android input support; not all app/keyboard combinations accept every input method.
- Defaults: longest dimension **1280 pixels**, **30 FPS**, **H.264**, **8 Mbps**. Performance: 800 px/24 FPS/2 Mbps. Quality: 1920 px/60 FPS/12 Mbps. Higher quality is a cap/request, not a promise the device can sustain it.
- Settings also offer 1024/2560 px, 15/24/30/60 FPS and 2/4/8/12/16 Mbps. Save and stop/restart the mirror to apply changes.
- Audio and automatic clipboard synchronization are disabled. Recording is not started. Scrcpy's own explicit clipboard/file transfer actions still exist if you invoke them in its window.
- **Stop mirror** gracefully closes only the owned scrcpy process, with a bounded forced-termination fallback. ADB remains connected until Disconnect. On app exit, the selected wireless transport is disconnected best-effort. The shared ADB daemon is not killed because other developer tools may be using it. If attached by USB, unplugging the cable is necessary to remove that USB transport.
- **Disconnect** stops mirroring and disconnects the selected wireless ADB transport; pairing remains. ADB can automatically reconnect paired devices via its own mDNS behavior while running; use Android's Wireless debugging toggle/revoke pairing for a firm access boundary.
- **Disconnect & forget local device** clears AirMirror's remembered endpoint/hardware ID. For actual authorization revocation use **phone → Wireless debugging → Paired devices → this PC → Forget**. ADB has no documented per-device `unpair` command. AirMirror deliberately does not delete shared PC ADB keys or falsely claim revocation.

## Critical screen-off distinction

### Mode A — physical panel off, device running

Enable *Turn the physical phone display off when mirroring starts* to pass scrcpy's documented `--turn-screen-off`. During an active session **Display off** sends **Right Ctrl+O** to its window; **Display on** sends **Right Ctrl+Shift+O**. **Fullscreen** sends Right Ctrl+F; F11 can also be used directly in scrcpy.

Buttons activate the scrcpy window and use Windows `SendInput` with an explicit right-Ctrl shortcut modifier. They fail with instructions if foreground focus is blocked or a physical modifier is held. They do **not** inject Android `KEYCODE_POWER`, which would conflate panel power with locking. Windows shortcut delivery has no Android acknowledgement: the UI says *requested/sent*, never falsely claims the physical screen state was measured.

Verify that live video and input continue on your Galaxy F15 while its physical panel is off. This hardware test has **not** been run here. To restore, use Display on, its direct scrcpy shortcut, or the physical power button. Do not rely on a software restore if Wi-Fi has already disconnected. Restoring the panel is not unlocking it. AirMirror passes scrcpy's `--keep-active` option during mirroring so a Wi-Fi/TCP session periodically signals Android activity and the normal inactivity timeout does not turn the mirror off.

Android 15+ documents `cmd display power-off/on`, but this release consistently uses scrcpy's supported mechanism rather than assuming an OEM command works based only on API level.

### Keeping the mirror alive while the panel is off

The in-app Display on, Display off and Fullscreen buttons were removed from this build. Use Right Ctrl+O, Right Ctrl+Shift+O and Right Ctrl+F inside the scrcpy window instead.

Android stops sending frames when it goes to sleep, which turns the mirror black. With **Keep the phone awake while mirroring** enabled (Settings, on by default) AirMirror reads the phone's `screen_off_timeout`, saves it to `settings.json`, raises it to 24 hours with `adb shell settings put system screen_off_timeout`, and restores the saved value on Stop, Disconnect or exit. If the phone is unreachable at that moment, the saved value is restored the next time the same phone connects. Combine it with *Turn the physical phone display off when mirroring starts* to keep the panel dark. AirMirror also asks Windows (`SetThreadExecutionState`) not to blank this PC's display or sleep during a session.

This prevents the normal inactivity timeout from putting the phone to sleep while mirroring. An explicit physical power-button action, OEM security policy, or protected/DRM content may still blank or stop capture; AirMirror does not bypass Android security or unlock the phone. Without the option on, Android's own timeout will put the phone to sleep as before.

### Mode B — device locked

With **Keep the phone awake while mirroring** enabled, scrcpy's `--keep-active` mode is passed to the running session and the desktop mirror remains active when the physical panel is turned off or Android reaches its normal inactivity boundary. Android/One UI can still restrict capture of an explicitly locked or security-protected screen; unlock on the phone using normal credentials if that happens. AirMirror never changes PINs, biometric settings, security policy or lock-screen configuration.

## Recovery and diagnostics

While attached, ADB is checked approximately every 8 seconds (no disconnected-device polling). Loss stops the stale mirror and enters Reconnecting. Retries back off to at most one per minute; only a previously remembered private endpoint is retried. Any already-connected device must match the prior `ro.serialno` identity. A different device at the old IP is disconnected, not adopted. Missing/unknown hardware ID disables automatic recovery. Already-paired ADB mDNS auto-reconnection may expose the same identity under a changed serial/port. AirMirror does not pair or connect to arbitrary advertised peers.

After recovery, **Start mirroring** again explicitly; video is not silently restarted. After reboot, unlock the phone and enable Wireless debugging if necessary. The connection port may have changed: Disconnect, enter the new main-screen IP:port, Connect. Cancellation cannot revoke a pairing that Android had already completed; inspect/revoke on the phone if needed.

Settings and rotating JSON-lines diagnostics live at `%LOCALAPPDATA%\AirMirror` (`settings.json`, `diagnostics.jsonl`, one prior log). Settings contain only tool path, performance preferences and necessary remembered identity/endpoint. These are ordinary per-user metadata, not authentication secrets. Pairing keys are managed by official ADB in its standard Android user profile; AirMirror never copies them. Logs contain process diagnostics and may include local addresses/device identifiers; review before sharing. No logs/video/screenshots/phone data are uploaded by the application.

- **Pairing fails:** keep the pairing dialog open, use its port and a fresh six-digit code. Check clock/network and authorization prompts.
- **Connection refused:** use the main-page connection port, not the pairing port; verify same Wi-Fi, no client isolation and local VPN routes.
- **No mDNS:** enter IP:port manually; multicast may be filtered on corporate/guest networks.
- **Offline/unauthorized:** unlock and approve on the phone; revoke/re-pair if necessary.
- **Missing executable/DLL:** extract the complete official archive, not just scrcpy.exe. Check custom tool folder and quarantine. No global firewall disable is needed.
- **ADB conflict:** close relevant developer tools and align ADB versions. AirMirror does not indiscriminately kill the shared server.
- **Black/frozen mirror:** test an ordinary unlocked screen; lower quality; secure content cannot be bypassed.
- **Shortcut rejected:** click the scrcpy window and use the documented shortcut there; release physical modifiers; run both apps at the same ordinary user privilege level.
- **Corrupt settings:** close the app and rename `%LOCALAPPDATA%\AirMirror\settings.json` to reset. Startup reports the problem rather than discarding it silently.

## Source build and tests

**.NET 10 LTS** is used because it is the current stable long-term-support release with WPF support for Windows x64. The supplied distribution is self-contained. `global.json` accepts the latest installed .NET 10 feature SDK. Use Visual Studio with .NET 10 WPF support or the .NET 10 SDK; .NET Framework is not used.

On Windows, from the repository root:

```powershell
# Internet needed only to restore packages/download pinned official tools.
python scripts/prepare_tools.py
# Or, all-in-one build/test/publish/installer, with NSIS 3 on PATH:
powershell -ExecutionPolicy RemoteSigned -File scripts/build.ps1 -Installer

# Individual developer operations:
dotnet restore AirMirror.sln
dotnet build AirMirror.sln -c Release
dotnet test AirMirror.sln -c Release
dotnet run --project src/AirMirror.App

dotnet publish src/AirMirror.App -c Release -r win-x64 --self-contained true -o artifacts/win-x64
makensis installer/AirMirror.nsi
```

Python 3 is only a developer dependency for verified dependency preparation. It is not needed by the installed app. NSIS compiles the actual installer on Windows or Linux. Do not trim WPF assemblies. Linux can cross-compile with `EnableWindowsTargeting=true` but cannot run WPF. The root directory holds this native desktop solution; the unused web starter has been removed.

### Current verification

- Release solution cross-build: executed on Linux; 0 warnings/errors in the completed verification run.
- Unit/service tests: executed using .NET 10 on Linux; see `BUILD-REPORT.md` and supplied TRX results for the final counts. ADB responses in unit tests are explicit fixtures, not real-phone validation.
- Actual process output/error capture, missing executable, timeout and cancellation paths: exercised with local test child processes.
- Self-contained `win-x64` publish: executed; native Windows payload generated.
- Installer compilation: executed using NSIS; installing/running/uninstalling still requires Windows.
- WPF rendering, UI bindings at runtime, Windows focus/SendInput, real ADB network operation, Galaxy F15 display-off/lock behavior and clean-machine installation: **not tested in this sandbox**.

### Modules

- `AirMirror.App`: dark WPF interface, MVVM commands, composition-root injection and Windows shortcut adapter.
- `AirMirror.Core`: models, parsing, endpoint validation and interfaces.
- `AirMirror.Infrastructure`: bounded/cancelable process runner, ADB pairing/discovery, tool resolution, local metadata/logging and identity-checked recovery.
- `AirMirror.Mirroring`: validated scrcpy arguments and owned-process lifecycle; `IMirrorWindowController` isolates native-window actions for future embedding work.
- `tests`: Core/Infrastructure and Mirroring xUnit projects. Production runtime has no third-party .NET package dependency beyond the platform.
- `scripts/prepare_tools.py`: pinned official binary verification, dependency notices and source staging.
- `installer/AirMirror.nsi`: per-user Windows installer/uninstaller.

## Remaining release gates / next steps

1. Install, launch and exercise all UI pages on clean Windows 10/11 x64; check keyboard navigation, scaling and bindings.
2. Complete every Galaxy F15 test in `HARDWARE-TESTS.md`, especially uninterrupted video/input with panel off and safe restoration.
3. Characterize One UI lock/sleep behavior and recovery after reboot/port changes; record actual OS/API/One UI build.
4. Validate upgrade/uninstall with the app closed, restricted accounts, antivirus and differing ADB versions. Sign release binaries/installer for public distribution.
5. Optional future work: supported embedded scrcpy/native rendering integration, QR-code workflow with official ADB protocols, configurable audio/IME features and a multi-device remembered-device library. These are not implemented or claimed.

## Licensing and privacy

AirMirror source is supplied under MIT (`LICENSE`). Third-party components remain under their own licenses; see `LICENSES.md`, bundled `tools/scrcpy/licenses` and `tools/scrcpy/sources`. The source bundles include the upstream scrcpy dependency-build scripts. Local operation only; no cloud accounts, analytics or hidden listeners are added by AirMirror. ADB itself runs its standard local daemon and opens documented device debugging transports.
