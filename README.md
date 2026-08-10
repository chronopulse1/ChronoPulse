[README.md](https://github.com/user-attachments/files/30917413/README.md)
# ChronoPulse

**Precision Acoustic Watch Analysis**

ChronoPulse is a cross-platform desktop application for analyzing the acoustic timing behavior of mechanical watches. It can measure a live microphone signal, replay compatible WAV recordings, or generate repeatable simulator data through the same detector and analytics pipeline.

Software reports **Rate**, **Beat Error**, **Amplitude**, **BPH**, and **Lift Angle**, while providing confidence/trust states, multi-position watch records, sound-card timebase calibration, specialized analysis views, and auditable exports.

## Highlights

- Real-time mechanical-watch acoustic analysis
- Live, Playback, and Simulator operating modes
- Rate, Beat Error, Amplitude, BPH, and Lift Angle measurement
- Separate trust badges for Rate, Beat Error, and Amplitude
- Internet-based sound-card clock calibration using validated SNTP observations
- Operational, Precision, and optional Reference calibration grades
- Immutable calibration history with JSON diagnostics export
- Recording-specific `.cal` sidecars that preserve timing provenance
- Multi-position watch records with transactional storage
- Built-in and per-watch Custom tolerance profiles
- Rate Scope, Event Scope, Advanced Event Scopes, Beat Scope, Sound Scope, and Signal Scopes
- Deviation, Polar Display, Reg Pins, Dial Trace, and Total Trace analysis
- PNG, XLSX, WAV, calibration-sidecar, and diagnostics export
- Dark and Light / Bright themes
- Windows, macOS, Linux, and Raspberry Pi support

## Primary Measurements

| Measurement | Unit | Meaning |
|---|---:|---|
| **Rate** | s/day | Estimated daily gain or loss. Positive values indicate gaining; negative values indicate losing. |
| **Beat Error** | ms | Timing asymmetry between alternating half-oscillations. |
| **Amplitude** | degrees | Estimated balance swing derived from A-to-C lift time, BPH, and the configured Lift Angle. |
| **BPH** | beats/hour | Detected escapement rate or a manually selected rate. |
| **Lift Angle** | degrees | Movement-specific input used for Amplitude calculation. It is not detected from the microphone. |

ChronoPulse separates numeric values from confidence state. A value may be available while its trust badge is still warming or unstable, so measurements should be interpreted together with the trust badges, detector lock state, scopes, and repeatability.

## Operating Modes

| Mode | Source | Typical use |
|---|---|---|
| **Live** | Selected audio input device | Measure a watch in real time and optionally record the analyzed stream. |
| **Playback** | Compatible WAV file | Re-run a previous recording through the detector and analysis pipeline. |
| **Simulator** | Built-in acoustic movement synthesizer | Training, display testing, repeatable analysis, and troubleshooting without a physical watch. |

### Playback format

Playback accepts mono **32-bit IEEE-float WAV** files at:

- 48,000 Hz
- 96,000 Hz
- 192,000 Hz
- 384,000 Hz

If a valid recording-specific calibration sidecar is present, ChronoPulse uses its effective sample rate for timing interpretation. Audio is **not resampled**. A missing or rejected sidecar does not block playback; nominal WAV timing is used instead.

## Quick Start

### Live measurement

1. Select **Live** mode.
2. Use **Auto BPH** unless a known manual movement rate is required.
3. Enter the correct **Lift Angle** for the movement.
4. In **Input Device > Input Setup**, select the input device and a supported sample rate.
5. Adjust hardware **Microphone Gain** first, then use **Stream Gain** for fine adjustment.
6. Press **Run** and allow stabilization to finish.
7. Wait for detector lock, accepted A-event flashes, and settled trust badges.
8. Inspect the Rate Scope, Event Scope, and other analysis views as needed.
9. Press **Pause** to finish.

Leaving all position buttons unselected allows analysis without storing a position run.

### Stored six-position test

1. Create or open the correct **Watch Record** and save any edits.
2. Select the six-position set: **DU, DD, CL, CD, CR, CU**.
3. Select **DU** and run for the configured duration.
4. Repeat the test for **DD, CL, CD, CR, and CU**.
5. Review the stored-position table and the Deviation, Polar Display, Reg Pins, Dial Trace, and Total Trace views.

A selected position requires an active Watch Record with no unsaved edits. A successful new run for the same watch and position replaces the previous completed run.

### Playback

1. Select **Playback** while stopped.
2. Press **Run** and choose a compatible WAV file.
3. ChronoPulse validates the WAV and checks for the exact `<complete WAV filename>.cal` sidecar.
4. Open **Live Control > Playback Calibration** to see whether the sidecar was applied and which effective rate is being used.
5. Press **Pause** or allow playback to reach end-of-file.

### Simulator

1. Select **Simulator**.
2. Configure sample rate, BPH, Rate Error, Amplitude, Beat Error, and movement archetype.
3. Confirm the Lift Angle.
4. Press **Run**.
5. Use the scopes and analysis views just as you would with a live source.

## Sound-Card Clock Calibration

ChronoPulse can estimate the effective sample rate of a selected physical input device and nominal sample-rate combination using callback-based audio frame counting and validated multi-provider SNTP observations.

Calibration grades are:

| Grade | Timebase target |
|---|---:|
| **Operational** | <= 2.5 ppm |
| **Precision** | <= 1.0 ppm |
| **Reference** | < 0.5 ppm |

**Precision** is the normal recommended stopping point. Reference is optional.

### First calibration

1. Stop any normal measurement.
2. Select the physical input device and nominal sample rate you intend to use.
3. Open **Input Device > Calibration**.
4. Review the visible Time Providers list.
5. Keep the default **60 s** steady-state interval and **250 ms** maximum RTT unless you have a specific reason to change them.
6. Press **Start Calibration**.
7. Keep the computer awake and keep the same device, USB path, power source, and network connection active.
8. Continue until the desired grade is supported by three consecutive accepted observations.
9. Press **Finish / Apply**.
10. Enable **Use calibrated sample rate for Live measurements** for that device/rate if the result should be active.

No watch or microphone signal is required for sound-card calibration.

> Internet calibration uses SNTP over UDP port 123. A network can allow ordinary web traffic while still blocking SNTP.

## Recording and Calibration Sidecars

A physical Live recording writes the analyzed mono float WAV stream and a sidecar named:

```text
<complete WAV filename>.cal
```

For example:

```text
MyWatch.wav
MyWatch.wav.cal
```

The sidecar preserves the timing context active when recording began, including nominal and measured rates, effective rate, calibration grade, uncertainty, method, calibration identity, producing application version, device identity, recording metadata, and optional diagnostics.

Playback validates the sidecar before using it. It never silently substitutes the current computer's Live calibration for a recording-specific timebase.

## Watch Records and Position Analysis

ChronoPulse stores watch records and timestamped position measurements in SQLite.

Supported position sets use canonical internal position keys:

```text
DU, DD, CL, CD-, CD, CD+, CR, CU+, CU, CU-
```

The user interface can display Crown Positions or Dial Positions without changing those canonical database keys.

Stored analysis includes:

- Position summaries
- Deviation
- Polar Display and tolerance summary
- Reg Pins
- Dial Trace
- Total Trace
- DVm / Phi calculations where the required positions are available

### Custom Watch Parameters

A **Custom** Watch Type can store per-watch limits for:

- Minimum / maximum Rate Error
- Maximum Beat Error
- Horizontal Amplitude range
- Vertical Amplitude range

These limits are used by Polar Display for watch-specific tolerance evaluation.

## Data and Persistence

ChronoPulse uses **SQLite** for watch data, timestamped samples, application settings, calibration metadata, and calibration history.

Version 11.4 uses database **schema 45** and supports a controlled transaction-based migration from schema 44.

Use:

```text
Settings / Advanced > Misc > Show Database Location...
```

to display the platform-specific application-data location.

**Back up the database** before operating-system changes, disk migration, or major maintenance.

## Exporting

ChronoPulse supports several export paths:

- **PNG** — analysis content from supported scopes and analysis pages
- **XLSX** — active Watch Record, measurement data, position summaries, overall summaries, and Custom tolerance values when applicable
- **WAV** — analyzed source stream
- **`.cal` sidecar** — recording-specific calibration/timebase provenance
- **JSON diagnostics** — immutable sound-card calibration history evidence for audit and troubleshooting

## Architecture

ChronoPulse 11.4 is a **modular monolithic Qt desktop application**. Acquisition, signal detection, watch-timing analytics, calibration, persistence, and presentation run in one process with explicit module boundaries and dedicated worker threads for timing-sensitive or blocking work.

Key responsibility boundaries:

- **TimeGrapher** — event detection, BPH estimation, and synchronization
- **ChronoAnalytics** — Rate, Beat Error, Amplitude, integration, and cumulative measurement statistics
- **AudioClockCalibrationEstimator** — sound-card timebase estimation
- **MainWindow** — application composition, session orchestration, processing coordination, and presentation
- **WatchDatabase** — SQLite persistence boundary

Runtime concurrency includes the GUI thread, one acquisition worker thread per source session, a separate calibration worker thread, and a Qt audio callback thread used during calibration.

## Repository Layout

```text
/
├── TimeGrapher/       # Detector DSP, A/C event detection, BPH and synchronization
├── WatchRecords/      # SQLite persistence, records, custom parameters, XLSX export
├── WatchAnalysis/     # Polar Display, Reg Pins, Dial Trace, Total Trace, position policy
├── BeatScope/         # Beat-scope model, detector adapter, QImage rendering
├── AdvEventScope/     # Advanced event packets, averages, renderers, widgets
├── SignalScopes/      # Raw / Smoothed / Enhanced / Rectified diagnostic views
├── SoundScope/        # Beat-column acoustic-intensity rendering and event markers
├── Theme/             # Dark/light styles, shared colors, plot styling, resources
├── Splash/            # Video splash implementation and media
├── Tests/             # CTest numerical and persistence/custom-tolerance tests
├── Tools/             # Static and focused verification helpers
├── verification/      # Source/UI/regression verification
├── Packaging/         # Platform packaging assets
├── Documents/         # Design notes, release notes, algorithms, feature documentation
├── CMakeLists.txt
└── ChronoPulseVersion # Application/package version source of truth
```

Root C++ sources contain the application shell, acquisition/playback/simulation workers, audio output, calibration, sidecars, platform audio controls, analytics, and the main UI integration classes.

## Building from Source

### Requirements

- **C++17**
- **CMake 3.16+**
- **Qt 6.11+**

Required Qt components include:

- Widgets
- Core
- SQL
- Network
- PrintSupport
- Multimedia
- MultimediaWidgets

Platform-specific native integration includes CoreAudio on macOS, Windows multimedia/COM APIs on Windows, and ALSA on Linux where applicable.

> The supplied Version 11.4 architecture reference identifies the build system, requirements, and package scripts, but does not prescribe a canonical command-line CMake invocation. Configure the project through `CMakeLists.txt` using a Qt 6.11+ toolchain appropriate for your platform.

### Platform packaging entry points

| Platform | Packaging entry point |
|---|---|
| Windows | `BuildWindowsPackage.bat` + `Packaging/` assets |
| macOS | `BuildMacOSPackage.sh` + signing/notarization settings |
| Linux / Raspberry Pi | `BuildLinuxPackage.sh` / CPack DEB configuration |

The packaging system targets Qt Installer Framework packages on Windows/macOS and self-contained Debian packages for Linux/Raspberry Pi.

## Testing and Verification

CMake enables **CTest** and registers focused test executables, including:

- `ChronoPulseCalibrationNumericalTests`
- `ChronoPulseCustomWatchParametersTests`

Coverage includes calibration estimator/policy behavior, frame-timeline and sidecar validation, deterministic numerical scenarios, schema migration, Custom tolerance persistence and validation, record hydration/search behavior, type changes, and cascade deletion.

The repository also includes Python and small C++ verification tools for source/UI structure and focused regressions such as calibration wiring, sidecars, audio monitor modes, position sets, header selectors, Watch Records behavior, spreadsheet export, reset behavior, packaging, installer upgrades, and low-BPH analytics.

The built-in **Simulator** provides a repeatable hardware-independent path through the detector and analytics pipeline.

## Supported Platforms

ChronoPulse supports builds for:

- Windows
- macOS
- Linux
- Raspberry Pi / Debian-family systems

Platform packaging and audio endpoint controls are platform-specific, while most application logic is shared through Qt.

## Measurement Responsibility

ChronoPulse is a diagnostic instrument. A displayed value is only as reliable as the acoustic coupling, Lift Angle selection, BPH lock, signal quality, integration time, sound-card calibration, and movement condition.

Use repeated measurements and qualified watchmaking judgment before making adjustments. Built-in tolerance profiles are ChronoPulse evaluation policies and should not be treated as a substitute for the exact manufacturer or certification specification for a particular movement.

## Troubleshooting Notes

- **Live mode unavailable:** connect/enable an input device, grant microphone permission, and rescan.
- **No supported sample rates:** the device must expose a supported mono float format through Qt at 48/96/192/384 kHz.
- **Detector does not lock:** check coupling, noise, gain, BPH selection, and Event Scope.
- **Calibration timeouts:** check DNS, firewall/VPN/router policy, and UDP port 123.
- **Monitor/Beat Tone causes false detections:** use headphones, reduce Output Gain, or disable audio output.
- **Playback sidecar rejected:** inspect Playback Calibration; playback continues using nominal WAV timing.

## Versioning

The application and packaging version are driven from the repository's `ChronoPulseVersion` source-of-truth file.

This README describes **ChronoPulse Version 11.4**.

## License

ChronoPulse is released under the **MIT License**.

## Contact

For problem reports or technical questions:

**ChronoPulse1@gmail.com**

Include the ChronoPulse version, operating system, input/output hardware, sample rate, BPH, Lift Angle, operating mode, reproduction steps, expected and observed behavior, and a screenshot or short WAV file when relevant. For calibration issues, include an exported diagnostics JSON record when possible.
