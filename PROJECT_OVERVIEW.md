# SAATHI — Project Overview

**SAATHI** (Smart Alzheimer's Assistance Technology for Healthy
Independence), branded on the physical prototype as **CogniSafe**, is a
wearable IoT neckband that improves the safety and independence of
Alzheimer's and dementia patients while reducing the monitoring burden on
caregivers.

## Problem Statement

Alzheimer's and dementia patients face elevated risks of falling,
wandering away from safe/familiar locations, and being unable to
communicate a medical emergency. Family caregivers cannot supervise a
patient around the clock, and existing solutions are often either
expensive medical-grade equipment or single-purpose devices (a fall
alarm *or* a GPS tracker, rarely both, and rarely with vitals). This
project addresses that gap with one affordable, wearable device that
covers fall detection, location safety, and basic vitals monitoring
together, alerting caregivers automatically when something is wrong.

## Objectives

1. Detect falls automatically and reliably, distinguishing real falls
   from ordinary daily movement.
2. Track the patient's live location and raise an alert the moment they
   leave a caregiver-defined safe zone (geofence).
3. Continuously monitor basic vitals (heart rate, SpO2, temperature) to
   surface early signs of physiological distress.
4. Give the patient a simple, one-touch way to call for help (SOS
   button / voice).
5. Notify caregivers instantly and with actionable detail (what
   happened, where, and current vitals) via Telegram and SMS.
6. Keep the device low-cost, low-power, and comfortable enough to be
   worn all day.

## Target Users & Stakeholders

| Stakeholder | Interest |
|---|---|
| Patient (primary wearer) | Safety, comfort, dignity, ease of use (minimal interaction required) |
| Family caregiver | Real-time peace of mind, fast alerts, location visibility |
| Clinical/institutional caregiver (optional) | Multi-patient monitoring, adherence tracking |
| Developer/maintainer | Reliable firmware, clear diagnostics, extensibility |

## Scope

**In scope (current prototype):**
- Fall detection (accelerometer + gyroscope + barometric altitude)
- GPS location tracking and geofence breach detection
- Heart rate & SpO2 monitoring
- Ambient/skin temperature and pressure sensing
- Manual SOS button
- Local buzzer alerts (geofence, medication reminder, fall)
- Browser-based live telemetry dashboard (Web Serial)
- Caregiver alerting via Telegram bot and SMS (SIM7000G)

**Out of scope / future work (see `HARDWARE.md` "planned" list):**
- Voice-based SOS (INMP441 microphone) and spoken prompts (DFPlayer Mini)
- Battery fuel-gauge reporting (MAX17048) and onboard charge management (TP4056)
- Scheduled medication reminders backed by a real-time clock (DS3231)
- Cloud-hosted history/analytics (currently local/session-only via the dashboard)

## Key Features (from the prototype summary)

| Feature | Sensor/Module |
|---|---|
| Emergency SOS | SOS button / voice (planned) |
| Fall Detection | MPU6050 + BMP280 |
| Geofencing | SIM7000G GPS |
| Health Monitoring | MAX30102 |
| Medication Reminders | (planned — DS3231) |
| Battery Monitoring | (planned — MAX17048) |
| Voice Prompts | (planned — DFPlayer Mini) |
| Caregiver Connectivity | MQTT + SMS + Call (Telegram + SMS implemented; call not yet implemented) |

## Physical Form Factor

A lightweight, cardboard-prototyped neckband (final version intended in a
durable, skin-safe enclosure) housing the ESP32, sensors, SIM7000G,
battery, and a small speaker/mic array, worn around the neck for
comfortable all-day use and unobtrusive fall-relevant motion sensing.

## Success Criteria

- Fall events are detected within ~4 seconds of impact with a low false-
  positive rate during normal daily activity (walking, sitting, bending).
- Geofence breach alerts reach the caregiver within seconds of the GPS
  fix confirming the breach.
- Heart rate/SpO2 readings are within an acceptable clinical tolerance
  when a finger/skin contact is present.
- The system remains usable (auto-bridge fallback) even if an individual
  sensor is temporarily disconnected, so a demo or partial-hardware unit
  never appears "dead."

## Related Documents

- `REQUIREMENTS_SPECIFICATION.md` — functional & non-functional requirements
- `SYSTEM_DESIGN.md` / `SYSTEM_DESIGN_DOCUMENT.md` — architecture and detailed design
- `WIREFRAMES.md` — dashboard UI layout
- `RESEARCH_REFERENCES.md` — background research and prior art
- `COMPLETE_WORKFLOW_GUIDE.md` — build, flash, run, and demo instructions
- `HARDWARE.md`, `FIRMWARE.md`, `DASHBOARD.md` — implementation-level docs
