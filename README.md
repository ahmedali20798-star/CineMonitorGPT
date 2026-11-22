[README.md](https://github.com/user-attachments/files/35238056/README.md)
# CineMonitor Pro 3.0 (Development Build)

Direct USB PTP Live View and cinema field monitor for Canon EOS cameras on Android, using Kotlin, Jetpack Compose, and OpenGL ES 3.0.

> **Development status:** this is an engineering build derived from the supplied v2.9 source. It has not been compiled in this environment and has not been tested against a physical Canon EOS R6 Mark II. Do not treat it as a production release until the build and hardware test checklist has passed.

## Preserved features

- Direct USB Host / Canon PTP Live View (`0x9153`).
- OpenGL ES 3.0 monitor renderer.
- 3D `.cube` LUT loading and intensity control.
- False Color, Focus Peaking, Zebra, Monochrome, H/V Flip, and Anamorphic De-squeeze.
- Camera telemetry and controls already present in the v2.9 source, including exposure, Kelvin WB, recording control, and Touch AF.
- Existing Jetpack Compose monitor UI.

## Changes in 3.0-dev.1

- Canon remote-view session initialization now checks the success response for each required PTP command. A failed initialization closes the partially opened USB resources and reports connection failure instead of returning a false-positive connected state.
- PTP simple responses are checked for response container type, success response code, and matching transaction ID.
- PTP data containers are checked for the expected transaction ID before their payload is used.
- USB Bulk OUT writes are treated as successful only when the complete packet was transferred.
- Live View request payload is reused instead of allocating a new 4-byte array on every iteration.
- A monotonic-clock frame watchdog detects the case where USB commands continue to respond but no new decoded frame arrives.
- Stream state is returned to `false` when an active stream exits, including cancellation.
- OpenGL rendering now uses `RENDERMODE_WHEN_DIRTY`; new frames and monitor setting changes request a render.
- Disconnect-related GL state changes are deferred to the GL render thread.
- Video aspect-ratio state updates are dispatched to the Android UI thread.
- Development version updated to `3.0.0-dev.1`.

## Important implementation notes

- The existing USB/PTP transaction path is still serialized using the controller's shared USB lock. This development build does **not** yet introduce the full independent PTP scheduler / transport module described in the future architecture plan.
- JPEG payload allocation and decoding are still based on the existing implementation. The full pooled transfer-buffer redesign and explicit frame ownership model remain future work.
- The frame watchdog thresholds are initial engineering defaults (15 seconds for first frame, 2.5 seconds after streaming starts) and must be validated with real camera/phone combinations.
- PTP response validation is intentionally strict. If a supported Canon firmware returns a non-standard response shape, capture a USB/PTP trace and adjust only after verifying the protocol behavior.

## Project structure

- `app/src/main/java/com/ahmedmahmoud/cinemonitor/MainActivity.kt` — USB Host lifecycle, permissions, and connection handling.
- `camera/` — Canon PTP controller, event parser, exposure codes, and Touch AF mapping.
- `gl/` — OpenGL ES renderer, LUT parser, and texture loader.
- `ui/` — Compose monitor interface and overlays.
- `docs/` — development notes and hardware validation checklist.

## Build requirements

- Android Studio Iguana / Koala or newer.
- JDK 17.
- Android SDK 34.
- Gradle 8.7.

The supplied archive did not contain `gradle-wrapper.jar` or the `gradlew` launch scripts. Open the project in Android Studio and allow it to sync with Gradle 8.7, or regenerate the wrapper from a local Gradle 8.7 installation before using `./gradlew`.

## Build

From the project root, after Gradle 8.7 is installed:

```bash
gradle assembleDebug
```

To regenerate the wrapper files:

```bash
gradle wrapper --gradle-version 8.7
```

## Hardware validation required

1. Connect a Canon EOS R6 Mark II over a known-good USB-C data cable.
2. Verify USB permission and Still Image interface selection.
3. Verify Open Session, Canon Remote Mode, Event Mode, EVF Mode, and EVF Output Device initialization.
4. Confirm the first Live View frame arrives within the startup timeout.
5. Run a 30-minute test, then a 2-hour test, while monitoring FPS, memory, and thermal state.
6. Test ISO, shutter, aperture, Kelvin WB, recording, and Touch AF while Live View is active.
7. Disconnect and reconnect the USB cable; verify clean recovery and no stale frame or GL errors.
8. Repeat on at least one flagship, one mid-range, and one other Android device/GPU family.

See [docs/VALIDATION_CHECKLIST.md](docs/VALIDATION_CHECKLIST.md).
