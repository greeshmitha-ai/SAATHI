# SAATHI — Requirements Specification

## 1. Purpose

This document specifies the functional and non-functional requirements
for the SAATHI wearable assistive system, covering the ESP32 firmware,
attached sensors/actuators, and the companion web dashboard.

## 2. Functional Requirements

### FR-1 — Fall Detection
- FR-1.1: The system shall sample accelerometer and gyroscope data at a
  minimum of 25 Hz.
- FR-1.2: The system shall classify motion as `NORMAL` or `MOVING` based
  on deviation from 1g and gyro magnitude.
- FR-1.3: The system shall run a multi-stage fall-detection state
  machine (freefall → impact → stillness confirmation) to reduce false
  positives.
- FR-1.4: On a confirmed fall, the system shall set a local alarm
  (buzzer) and expose `fall: true` in the telemetry stream within one
  dashboard update cycle (≤300 ms).
- FR-1.5: A confirmed fall shall be clearable only via the physical
  button or an explicit dashboard/serial command.

### FR-2 — Location & Geofencing
- FR-2.1: The system shall acquire GPS fixes from the SIM7000G at a
  minimum of once every 2.5 seconds.
- FR-2.2: The system shall compute the great-circle distance between the
  current fix and a configured home coordinate.
- FR-2.3: The system shall raise a geofence-breach alert (local buzzer +
  dashboard status) when that distance exceeds a configurable radius.
- FR-2.4: The system shall automatically clear a geofence alert once the
  patient returns inside the radius.
- FR-2.5: A fall alert shall take priority over a geofence alert on the
  shared buzzer output.

### FR-3 — Vitals Monitoring
- FR-3.1: The system shall detect finger/skin presence on the MAX30102
  before reporting heart rate or SpO2.
- FR-3.2: The system shall compute heart rate (BPM) from optical
  peak-detection, averaged over recent beats.
- FR-3.3: The system shall compute SpO2 using the AC/DC ratio-of-ratios
  method, bounded to a clinically plausible 88–100% range.
- FR-3.4: The system shall report ambient/skin temperature and
  barometric pressure/altitude at a minimum of 2 Hz.

### FR-4 — Emergency SOS
- FR-4.1: The system shall provide a physical push button that the
  patient can use to silence alerts or acknowledge a fall.
- FR-4.2 (planned): The system shall support a voice-triggered SOS via a
  microphone module.

### FR-5 — Caregiver Alerting
- FR-5.1: The system shall support caregiver notification via a
  Telegram bot (token + chat ID configured through the dashboard).
- FR-5.2: The system shall support SMS dispatch to a configured
  caretaker phone number via the SIM7000G cellular module.
- FR-5.3: Alert messages shall include the event type (fall/geofence)
  and, when available, current location.
- FR-5.4: The dashboard shall support sending a manual test notification
  independent of a real event.

### FR-6 — Telemetry & Dashboard
- FR-6.1: The firmware shall stream a single-line JSON telemetry packet
  over USB serial at a minimum of ~3 Hz.
- FR-6.2: The dashboard shall connect to the device via the Web Serial
  API without requiring a native app or backend server.
- FR-6.3: The dashboard shall visualize heart rate/SpO2, accelerometer,
  gyroscope, altitude, GPS location (on a map), fall status, geofence
  status, and buzzer state in real time.
- FR-6.4: The dashboard shall provide a Simulation mode that generates
  synthetic telemetry (including a simulated fall and geofence breach)
  without requiring connected hardware.
- FR-6.5: The dashboard shall display a raw serial/debug console of the
  incoming JSON stream.

### FR-7 — Medication Reminders (planned)
- FR-7.1: The system shall support caregiver-defined medication
  schedules.
- FR-7.2: The system shall trigger a distinct buzzer pattern and
  dashboard notification at each scheduled reminder time.
- FR-7.3: The patient/caregiver shall be able to acknowledge
  ("take pill") a reminder from the dashboard.

### FR-8 — Diagnostics & Resilience
- FR-8.1: The firmware shall auto-scan both I2C buses at boot and log
  every detected device address.
- FR-8.2: If a sensor is not physically detected, the firmware shall
  fall back to a synthetic baseline reading so the rest of the system
  remains demonstrable.
- FR-8.3: The firmware shall accept a `SCAN` command to re-run sensor
  detection without a full reboot.
- FR-8.4: The firmware shall accept a `TEST`/`SYNC` command to force all
  sensors into a known test state for verification.

## 3. Non-Functional Requirements

| ID | Category | Requirement |
|---|---|---|
| NFR-1 | Performance | Fall confirmation shall complete within ~4 seconds of initial freefall onset (sum of state-machine timeouts). |
| NFR-2 | Performance | Dashboard telemetry updates shall render at ≥3 Hz with no more than one dropped packet per second under normal serial conditions. |
| NFR-3 | Reliability | The system shall continue operating (in degraded/simulated form) if any single sensor fails or disconnects. |
| NFR-4 | Usability | The patient shall need at most one physical action (press button) to silence any alert. |
| NFR-5 | Usability | The dashboard shall require no installation — a standard Chromium-based browser and a USB connection are sufficient. |
| NFR-6 | Power | The wearable shall support all-day operation on a single charge of a 1000–2000 mAh Li-ion cell (target, pending power-budget validation). |
| NFR-7 | Safety | The buzzer shall never exceed a hearing-safe volume at typical wearing distance (to be validated against local audio safety standards). |
| NFR-8 | Security/Privacy | Caregiver credentials (Telegram token, chat ID, phone number) shall not be committed to source control or hard-coded in firmware. |
| NFR-9 | Portability | The dashboard shall run on any OS supporting a Web Serial–capable browser (currently Chrome/Edge desktop). |
| NFR-10 | Maintainability | Sensor thresholds (fall, geofence, finger-detection) shall be centralized as named constants, not scattered magic numbers. |

## 4. Hardware Requirements

See `HARDWARE.md` for the full verified component list and pin map.
Summary: ESP32, MPU6050/6500, BMP280, MAX30102, SIM7000G, push button,
buzzer, Li-ion battery + charging circuit (planned), battery fuel gauge
(planned), microphone + audio playback module (planned).

## 5. Software Requirements

- Arduino IDE (or PlatformIO) targeting ESP32
- Libraries: Adafruit BMP280, Adafruit Unified Sensor, SparkFun MAX3010x
- A Web Serial–capable browser (Chrome or Edge, desktop) for the dashboard
- A Telegram bot (created via BotFather) for caregiver alerting

## 6. Constraints & Assumptions

- Assumes a stable cellular signal is available in the patient's
  environment for GPS/SMS/Telegram functionality.
- Assumes the patient can tolerate a neck-worn device; alternate form
  factors are out of scope for this iteration.
- Geofence center and radius are currently fixed constants in firmware
  (`HOME_LAT`, `HOME_LON`, `GEOFENCE_RADIUS_METERS`) rather than
  dashboard-configurable — treated as a known limitation, not a defect.
- SpO2/heart-rate values are estimates from a low-cost consumer sensor
  and are **not** a substitute for certified medical equipment.

## 7. Acceptance Criteria (sample)

| Requirement | Test |
|---|---|
| FR-1.4 | Simulate a fall (drop test / `simFallBtn`) and confirm dashboard shows `FALL DETECTED` and buzzer activates within ~4 s. |
| FR-2.3 | Move the device (or use `simBreachBtn`) beyond the configured radius and confirm geofence pill and buzzer trigger. |
| FR-3.1–3.3 | Place a finger on the MAX30102 and confirm BPM settles into a stable 50–190 range within a few seconds. |
| FR-5.1/5.2 | Trigger a test notification and confirm receipt on the configured Telegram chat and/or phone. |
| FR-8.2 | Disconnect a sensor and confirm the dashboard still shows plausible values instead of errors. |
