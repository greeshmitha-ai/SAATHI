# SAATHI — Complete Workflow Guide

End-to-end guide from unboxing hardware to running a live caregiver
demo. For deep implementation detail, see `HARDWARE.md`, `FIRMWARE.md`,
and `DASHBOARD.md`.

## 1. Prerequisites

- ESP32 development board + USB cable
- MPU6050/6500, BMP280, MAX30102, SIM7000G modules, SOS button, buzzer
- Activated SIM card with data/SMS (for GPS + Telegram/SMS features)
- Arduino IDE (or PlatformIO) with ESP32 board support installed
- A Chromium-based browser (Chrome or Edge) for the dashboard
- A Telegram account (to create a bot via @BotFather, optional but
  recommended)

## 2. Hardware Assembly

1. Wire each sensor per the verified pin map in `HARDWARE.md` — start
   with the primary I2C bus (GPIO 21/22); only use the secondary bus
   (GPIO 18/19) if a sensor must be split off due to address conflicts.
2. Wire the SIM7000G to UART2 (RX=16, TX=17) and its PWRKEY to GPIO 4,
   on its own adequately-rated power rail.
3. Wire the SOS button to GPIO 27 (internal pull-up, active LOW) and the
   buzzer to GPIO 25.
4. Insert the activated SIM card into the SIM7000G before powering on.
5. Double-check shared ground between all modules and the ESP32.

## 3. Firmware Setup & Flashing

1. Install required libraries via Arduino Library Manager:
   - Adafruit BMP280 Library
   - Adafruit Unified Sensor
   - SparkFun MAX3010x Pulse and Proximity Sensor Library
2. Open `SAATHI.ino` in the Arduino IDE.
3. Review the configurable constants near the top of the file — in
   particular `HOME_LAT`, `HOME_LON`, `GEOFENCE_RADIUS_METERS`, and
   `CARETAKER_PHONE` — and set them for your actual deployment location
   and caregiver contact.
4. Select your ESP32 board variant and correct COM/serial port.
5. Upload the sketch.
6. Open the Arduino Serial Monitor at **115200 baud** to confirm boot
   logs: I2C scan results, sensor connection status, and SIM7000G AT
   handshake result.

## 4. First-Boot Verification (via Serial Monitor)

1. Confirm the I2C scan reports your sensors at their expected
   addresses (`0x68`/`0x69` IMU, `0x76`/`0x77` BMP280, `0x57`
   MAX30102) — see `HARDWARE.md` for the full address table.
2. If a sensor is missing, the firmware will log an "Auto-Bridge
   active" message and continue with synthetic values — this is
   expected behavior, not a fatal error, but confirm real wiring before
   trusting live readings.
3. Send `SCAN` in the Serial Monitor to re-run detection after fixing
   any wiring issue, without re-flashing.
4. Send `TEST` (or `SYNC`) to confirm all sensors report a known test
   reading, verifying the JSON streaming path end to end.

## 5. Dashboard Setup

1. Open `index.htm` directly in Chrome or Edge (no server needed).
2. Click **Connect USB**, select the ESP32's serial port from the
   browser's picker, and confirm **115200 baud** is used.
3. Confirm the hardware status badge changes to "Live Feed Active" /
   "Wearable Telemetry Connected."
4. Watch the **Serial Debug Output** panel to confirm JSON packets are
   arriving roughly every 300 ms.

## 6. Configuring Caregiver Alerts

1. In the dashboard's Telegram section, click **Configure Telegram
   Bot**.
2. Create a bot via Telegram's **@BotFather** (message it, run
   `/newbot`, follow prompts) to obtain a bot token.
3. Find your caregiver **Chat ID** (per the in-app instructions — e.g.
   by messaging your new bot and using a chat-ID lookup bot).
4. Enter the token and chat ID, click **Save Settings**.
5. Send a manual test notification and confirm it's received in
   Telegram.
6. Confirm `CARETAKER_PHONE` in the firmware matches the intended SMS
   recipient for SIM7000G-based alerts.

## 7. Running a Live Demo (Real Hardware)

1. Power the assembled wearable and connect it to the dashboard (§5).
2. **Fall test:** with the device secured to a soft test surface,
   perform a controlled drop/tilt to trigger freefall → impact →
   stillness. Confirm the fall status pill changes, the buzzer sounds
   the fall pattern, and (if configured) a caregiver alert arrives.
3. Acknowledge the fall via the physical SOS button; confirm the
   dashboard clears back to "NO FALL."
4. **Geofence test:** physically move the device beyond
   `GEOFENCE_RADIUS_METERS` from `HOME_LAT/HOME_LON` (or temporarily
   lower the radius constant for an easier indoor test) and confirm the
   geofence pill and buzzer activate, then clear on return.
5. **Vitals test:** place a finger on the MAX30102 and confirm BPM/SpO2
   settle into a stable, plausible reading within a few seconds.

## 8. Running a Demo Without Full Hardware (Simulation Mode)

1. Open the dashboard as in §5 (Connect USB is optional in this mode).
2. Switch to **Simulation** mode using the mode toggle.
3. Use **Simulate Fall** and **Simulate Geofence Breach** to walk
   through the same UI states (banner, pills, buzzer badge) as a real
   event, without requiring wired sensors or being physically present
   at the deployment location.
4. Use **Return to Live** to switch back once real hardware is
   connected.

## 9. Troubleshooting

| Symptom | Likely Cause | Fix |
|---|---|---|
| Dashboard shows "Awaiting Device" indefinitely | Wrong serial port or baud, or device not flashed | Reselect port in Connect USB; confirm 115200 baud both sides |
| Sensor stuck on Auto-Bridge values | Physical wiring/address issue | Re-check wiring against `HARDWARE.md`; send `SCAN`; inspect Serial Monitor I2C scan output |
| No GPS fix ("SEARCHING") | Cold GNSS start, indoor location, or antenna issue | Move outdoors with clear sky view; allow a few minutes for first fix |
| SIM7000G "No AT response" | Power supply insufficient, wiring reversed, or SIM not seated | Check dedicated power rail; verify TX/RX not swapped; reseat SIM |
| Telegram alert not received | Bad token/chat ID, or bot not started | Re-verify token/chat ID; message the bot at least once before testing |
| False fall alarms during normal activity | Thresholds too sensitive for this patient's gait | Tune `FALL_FREEFALL_G_THRESH`/`FALL_IMPACT_G_THRESH`/`FALL_GYRO_STILLNESS` in firmware and re-flash |
| SpO2/HR reads 0 or implausible | No finger contact, or `MAX30102_IR_FINGER_MIN` miscalibrated for skin tone/placement | Confirm firm sensor contact; adjust threshold per `FIRMWARE.md` notes |

## 10. Iteration Workflow (ongoing development)

1. Change firmware constants or logic in `SAATHI.ino`.
2. Re-flash and re-verify via Serial Monitor (§4) before trusting the
   dashboard.
3. Re-run the relevant demo scenario in §7/§8 to confirm behavior.
4. Update `REQUIREMENTS_SPECIFICATION.md` / `SYSTEM_DESIGN_DOCUMENT.md`
   if the change affects documented behavior, thresholds, or the JSON
   schema.
5. Note any newly wired "planned" component (from `HARDWARE.md` §
   Prototype-only components) once it's actually implemented in
   firmware, and move it out of the "planned" list.

## 11. Document Map

| Need to... | Read |
|---|---|
| Understand the project's goals and features | `PROJECT_OVERVIEW.md` |
| Know exactly what the system must do | `REQUIREMENTS_SPECIFICATION.md` |
| See the architecture and data flow | `SYSTEM_DESIGN.md` |
| Get full module-by-module design detail | `SYSTEM_DESIGN_DOCUMENT.md` |
| Wire the hardware | `HARDWARE.md` |
| Understand firmware internals | `FIRMWARE.md` |
| Understand the dashboard's features | `DASHBOARD.md` |
| See the dashboard's screen layout | `WIREFRAMES.md` |
| Find background research/prior art | `RESEARCH_REFERENCES.md` |
| Build, flash, run, and demo the system | this file |
