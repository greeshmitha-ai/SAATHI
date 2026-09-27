# SAATHI — System Design Document (SDD)

> Formal, detailed design specification. For a shorter architecture/data-
> flow summary with diagrams, see `SYSTEM_DESIGN.md`.

## 1. Introduction

### 1.1 Purpose
This SDD describes the detailed design of the SAATHI wearable assistive
system: its modules, interfaces, data structures, and design rationale,
sufficient for a new engineer to implement, extend, or maintain it.

### 1.2 Scope
Covers the ESP32 firmware (`SAATHI.ino`), the attached sensor/actuator
hardware, and the web-based caregiver dashboard (`index.htm`). Cloud
backend, mobile apps, and voice features are out of scope (see
`PROJECT_OVERVIEW.md` §Scope).

### 1.3 Definitions

| Term | Meaning |
|---|---|
| Auto-Bridge | Firmware's synthetic-data fallback mode when a physical sensor isn't detected |
| Geofence | Virtual circular boundary around a home coordinate |
| PPG | Photoplethysmography — optical blood-volume sensing used for HR/SpO2 |
| Fix | A GPS position reading with valid coordinates |

## 2. Design Considerations

### 2.1 Assumptions & Dependencies
- ESP32 board with dual I2C bus capability (`Wire` and `Wire1`) and a
  spare hardware UART (UART2).
- Sensors wired per `HARDWARE.md`; addresses `0x68/0x69` (IMU),
  `0x76/0x77` (BMP280), `0x57` (MAX30102).
- Caregiver browser supports the Web Serial API (Chrome/Edge desktop).
- Cellular SIM with data/SMS active in the SIM7000G.

### 2.2 Design Constraints
- Single-threaded cooperative scheduling (no RTOS) — all subsystem
  logic must remain non-blocking.
- Dashboard has no backend server — all state lives in the browser
  session (refreshing the page loses history, not live sensor state).
- Fixed geofence center/radius (compile-time constants), not yet
  runtime-configurable.

### 2.3 General Constraints
- Must remain demonstrable with partial or no physical hardware
  (auto-bridge requirement).
- Alert-worthy state changes must reach the dashboard within one
  telemetry cycle (300 ms).

## 3. Architectural Design

See `SYSTEM_DESIGN.md` §1–2 for the block diagram and layered view. In
SDD terms, the system follows a **sense → process → actuate/transport →
present → notify** pipeline, implemented as five cooperating firmware
subsystems plus one browser-based presentation/notification layer.

## 4. Module Design

### 4.1 Module: I2C Bus Management
- **Files/functions:** `clearI2CBus()`, `initSensors()`
- **Responsibility:** Recover a stuck I2C bus (manual clock-pulse
  bit-bang), enable pull-ups, scan both buses for every address 1–126,
  and log discovered devices with a best-guess label.
- **Inputs:** none (runs at boot / on `SCAN` command)
- **Outputs:** `mpuConnected`, `bmpConnected`, `bmpPhysicalFound`,
  `maxConnected`, `maxPhysicalFound` flags; serial log

### 4.2 Module: MPU6500/6050 Driver
- **Files/functions:** `initMPU6500()`, `readRawMPU6500()`, `readMPU()`
- **Responsibility:** Direct-register I2C driver (no external library);
  wakes the sensor, configures ±8g accel / ±500 dps gyro ranges, and
  converts raw 16-bit values to physical units (m/s², °/s).
- **Interfaces:** I2C, address `0x68` or `0x69`, either bus
- **Design rationale:** A direct-register driver was chosen over a
  packaged library to support both MPU6050 and MPU6500 variants
  transparently and to allow dual-bus auto-detection.

### 4.3 Module: Fall Detection State Machine
- **Files/functions:** `readMPU()` (embeds the state machine),
  `FallState` enum
- **States:** `MONITORING → FREEFALL_DETECTED → IMPACT_DETECTED →
  STILLNESS_CHECK → FALL_CONFIRMED`
- **Transition table:**

  | From | Condition | To |
  |---|---|---|
  | MONITORING | a_mag < 0.42g | FREEFALL_DETECTED |
  | FREEFALL_DETECTED | a_mag > 2.80g | IMPACT_DETECTED |
  | FREEFALL_DETECTED | >850 ms elapsed, no impact | MONITORING |
  | IMPACT_DETECTED | altitude drop ≥0.45 m OR >400 ms elapsed | STILLNESS_CHECK |
  | STILLNESS_CHECK | gyro magnitude > 70°/s (2× stillness threshold) | MONITORING |
  | STILLNESS_CHECK | >2500 ms stationary | FALL_CONFIRMED |
  | FALL_CONFIRMED | button press / dashboard command | MONITORING |

- **Design rationale:** a single accelerometer threshold produces
  frequent false positives (sitting down hard, dropping the device);
  the 4-stage confirmation sequence (validated against the approaches
  in `RESEARCH_REFERENCES.md`) trades ~3–4 seconds of latency for much
  higher specificity.

### 4.4 Module: BMP280 Driver
- **Files/functions:** `readBMP()`, `bmpPrimary`/`bmpSec` objects
  (Adafruit_BMP280 library)
- **Responsibility:** Temperature, pressure, sea-level-relative altitude
  (used both for its own dashboard values and as fall-confirmation
  input in §4.3).
- **Sampling:** `MODE_NORMAL`, 2× temp oversampling, 16× pressure
  oversampling, 16× IIR filter, 1 ms standby.

### 4.5 Module: MAX30102 Driver (Heart Rate & SpO2)
- **Files/functions:** `initMAX30102Sensor()`, `readMAX()`
- **Responsibility:** Finger-presence gating, DC/AC signal filtering,
  ratio-of-ratios SpO2 estimation, peak-based heart-rate calculation.
- **Key internal state:** `irDC/redDC` (0.95/0.05 low-pass), `irAC/redAC`
  (0.90/0.10 low-pass), `spo2Filter` (0.92/0.08 smoothing), 4-slot
  rolling BPM buffer.
- **Algorithm:** see `FIRMWARE.md` §"MAX30102 SpO2/Heart-Rate Algorithm"
  for the full formula and bounds; grounded in the standard PPG
  ratio-of-ratios method (`RESEARCH_REFERENCES.md`).

### 4.6 Module: SIM7000G GNSS/Cellular Driver
- **Files/functions:** `initSIM7000G()`, `readGPS()`,
  `calcGeofenceDist()`
- **Responsibility:** Power-cycle the module via PWRKEY, establish UART2
  AT-command session, poll `AT+CGNSINF` for position, parse the CSV
  response, and compute geofence distance via the Haversine formula.
- **Error handling:** malformed/empty responses leave `gpsFix =
  "SEARCHING"`/`"NOT AVAILABLE"` rather than crashing the parser;
  0,0 coordinates are explicitly rejected as invalid fixes.

### 4.7 Module: Buzzer Actuator
- **Files/functions:** `setBuzzerMode()`, `updateBuzzer()`
- **Responsibility:** Non-blocking waveform generation for 3 alert
  patterns (geofence, medication, fall) plus off, using `millis()`-based
  toggling rather than `tone()`/`delay()`.
- **Priority rule:** fall alarm always overrides a geofence alert on the
  shared buzzer pin.

### 4.8 Module: Command & Input Handling
- **Files/functions:** `handleSerialCommands()`, `handleHardwareInputs()`
- **Responsibility:** Parse newline-delimited text commands from the
  dashboard (`BUZZ:*`, `TEST`/`SYNC`, `SCAN`) and debounce the physical
  SOS/dismiss button (300 ms).

### 4.9 Module: Telemetry Streaming
- **Files/functions:** `streamDashboardJSON()`
- **Responsibility:** Serialize all subsystem state into one JSON object
  per 300 ms tick (see `FIRMWARE.md` for the exact schema). Uses manual
  string building rather than a JSON library to minimize memory/CPU
  overhead on the ESP32.

### 4.10 Module: Dashboard (Presentation & Notification)
- **Responsibility:** Web Serial connection management, real-time
  charting (accelerometer, gyroscope, altitude, ECG-style HR waveform),
  map display, alert banner, Telegram/SMS configuration and dispatch,
  medication reminder UI, debug console, and Live/Simulation mode
  switching.
- **State ownership:** all caregiver-entered settings (Telegram token,
  chat ID) are held in the browser session only — never sent to or
  stored on the ESP32.

## 5. Data Design

### 5.1 In-Memory Firmware State (selected)
- Sensor readings: `ax/ay/az`, `gx/gy/gz`, `bmpTemperature/Pressure/
  Altitude`, `currentBPM`, `currentSpO2`, `gpsLatitude/Longitude`
- Status flags: `mpuConnected`, `bmpConnected`, `maxConnected`,
  `simConnected`, `gpsValid`, `fallDetected`
- Enumerated state: `FallState fallMachineState`, `BuzzerMode
  currentBuzzMode`

### 5.2 Wire-Format Data (JSON telemetry)
See `FIRMWARE.md` §"Dashboard JSON Telemetry Schema" for the full,
authoritative schema.

### 5.3 Persistent Data
None on-device (no SD/flash storage of history in the current
firmware). The dashboard does not persist across page reloads. This is
a known limitation — see `REQUIREMENTS_SPECIFICATION.md` §6 Constraints.

## 6. Interface Design

| Interface | Direction | Format |
|---|---|---|
| ESP32 → Dashboard | Serial out | Single-line JSON, `\n`-terminated |
| Dashboard → ESP32 | Serial in | Plain-text command strings (see `FIRMWARE.md` §Serial Command Interface) |
| ESP32 → SIM7000G | UART2 | AT command set (`AT`, `ATE0`, `AT+CMGF=1`, `AT+CGNSPWR=1`, `AT+CGNSINF`) |
| Dashboard → Telegram | HTTPS | Telegram Bot API `sendMessage` |
| Dashboard → Caregiver | UI | Live charts, map, status pills, emergency banner |

## 7. Human Interface Design

See `WIREFRAMES.md` for the dashboard's screen-level layout and
component inventory.

## 8. Error Handling & Recovery Strategy

| Failure | Detection | Recovery |
|---|---|---|
| Sensor not found at boot | I2C scan returns no ACK | Auto-bridge synthetic data; background retry every 3 s (MAX30102) |
| I2C bus stuck low | N/A (preventive) | `clearI2CBus()` bit-bang run unconditionally at boot |
| SIM7000G no AT response | Timeout on `AT` handshake | `simConnected = false`; logged, GPS reports "NOT AVAILABLE" |
| Malformed GPS response | String parse indices not found | `gpsFix = "NOT AVAILABLE"`, no crash |
| False-positive fall onset | Stillness-check gyro exceeds threshold | State machine reverts to `MONITORING` automatically |
| Dashboard loses serial connection | Web Serial read failure/close event | `hardwareStatusBadge` reflects disconnect; user reconnects via Connect USB |

## 9. Traceability to Requirements

Each module in §4 maps to the functional requirements in
`REQUIREMENTS_SPECIFICATION.md` as follows: §4.2–4.3 → FR-1; §4.6 →
FR-2; §4.4–4.5 → FR-3; §4.8 → FR-4; §4.10 (Telegram/SMS) → FR-5; §4.9–
4.10 → FR-6; §4.1 → FR-8.

## 10. Open Design Questions / Future Revisions

- Make geofence center/radius runtime-configurable from the dashboard
  instead of firmware constants.
- Add persistent history (SD card or cloud sync) for post-event review.
- Introduce an RTOS or interrupt-driven MPU sampling if 25 Hz polling
  proves insufficient for faster fall onsets.
- Formalize SMS message templates alongside the existing Telegram
  templates.
