# SAATHI (CogniSafe) — Wearable IoT Assistive System

> **Smart Alzheimer's Assistance Technology for Healthy Independence**

SAATHI is an ESP32-based wearable IoT neckband designed to enhance the safety, mobility, and dignity of Alzheimer's and dementia patients while minimizing caregiver supervision stress.

---

## 🌟 Key Highlights & Core Capabilities

- **Multi-Stage Fall Detection:** State-machine powered inertial algorithm (Freefall → Impact → Stillness verification) combining accelerometer, gyroscope, and barometric altitude data to eliminate false alarms.
- **GPS Safe-Zone Geofencing:** Autonomous continuous distance tracking from configurable home coordinates using the SIM7000G cellular/GNSS module, sounding alarms and dispatching notifications upon breach.
- **Continuous Vitals Telemetry:** Integrated optical photoplethysmography (MAX30102) for real-time heart rate (BPM) and blood oxygen saturation (SpO2) estimation with skin-contact detection.
- **Emergency SOS & Two-Way Alerting:** One-touch physical button for patient alert silencing and immediate caregiver SOS dispatch via Telegram Bot API and cellular SMS.
- **Real-Time Web Serial Dashboard:** Single-page Chromium browser dashboard communicating directly over USB Web Serial at ~3.3 Hz with live telemetry plotting, Leaflet GPS mapping, buzzer override, and full hardware simulation mode.

---

## 📂 Project Architecture & Documentation

| Document | Purpose & Contents |
| :--- | :--- |
| [**PROJECT_OVERVIEW.md**](./PROJECT_OVERVIEW.md) | Mission, problem statement, target demographics, and high-level system objectives. |
| [**REQUIREMENTS_SPECIFICATION.md**](./REQUIREMENTS_SPECIFICATION.md) | Formal functional (FR-1 to FR-8) & non-functional requirements, safety constraints, and acceptance criteria. |
| [**SYSTEM_DESIGN_DOCUMENT.md**](./SYSTEM_DESIGN_DOCUMENT.md) | In-depth hardware/firmware subsystem design, state machine transitions, PPG filtering mathematics, and JSON schemas. |
| [**COMPLETE_WORKFLOW_GUIDE.md**](./COMPLETE_WORKFLOW_GUIDE.md) | Step-by-step setup: component pinouts, flashing guide, first boot calibration, serial debugging, and demo protocols. |
| [**WIREFRAMES.md**](./WIREFRAMES.md) | Complete visual layout, component hierarchy, responsive specifications, and UX behavior for the web dashboard. |
| [**RESEARCH_REFERENCES.md**](./RESEARCH_REFERENCES.md) | Scientific literature citations, medical threshold references, and algorithm validation benchmarks. |

---

## 🛠️ Hardware Specifications

- **Microcontroller:** ESP32-WROOM-32 (Dual-core Xtensa 32-bit LX6, 240 MHz)
- **IMU:** MPU-6050 / MPU-6500 (6-axis accelerometer & gyroscope on I2C)
- **Altimeter & Environmental:** BMP280 (High-precision barometric pressure & temperature)
- **Vitals Sensor:** MAX30102 (High-sensitivity pulse oximeter and heart-rate sensor)
- **Cellular & GNSS:** SIMCom SIM7000G (LTE-CAT-M1 / NB-IoT & GNSS on Hardware UART2)
- **Actuators & Inputs:** Active Piezo Buzzer (PWM audio alert patterns), Tactile SOS Pushbutton

---

## 📊 Telemetry Format

The device continuously streams compact, newline-delimited JSON over USB Serial at 115200 baud (approx. 3.3 Hz / 300 ms interval):

\\\json
{
  "accel": {"x": 0.05, "y": -0.12, "z": 9.81},
  "gyro": {"x": 1.2, "y": -0.8, "z": 0.4},
  "altitude": 542.3,
  "temp": 36.5,
  "bpm": 74,
  "spo2": 98,
  "gps": {"lat": 17.4455, "lon": 78.3498, "fix": true},
  "fall": false,
  "geofence": false,
  "buzzer": "OFF"
}
\\\

---

## 🚀 Getting Started

1. **Firmware:** Open the project in Arduino IDE or PlatformIO. Ensure dependencies (Adafruit_BMP280, SparkFun_MAX3010x) are installed. Flash to ESP32 at 115200 baud.
2. **Dashboard:** Open the companion web dashboard directly in Chrome or Edge (no backend server required).
3. **Connect:** Click **Connect USB** to establish direct Web Serial streaming, or activate **Simulation Mode** for demonstrations without physical sensors.

---

## 👥 Contributors & Maintainers
- Developed as an assistive IoT safety platform for patient care and caregiver peace of mind.
