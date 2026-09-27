# SAATHI — Wireframes (Dashboard UI)

Describes the screen-level layout of the caregiver web dashboard
(`index.htm`), derived from its actual structure, so this can serve as a
reference for redesign or for building an alternate (e.g. mobile) client.

## 1. Overall Layout

```
┌──────────────────────────────────────────────────────────────────────┐
│  Header: "SAATHI · TELEMETRY v2.5"          [Connect USB] [Live|Sim] │
│  Sub-badges: UART2 // 115200 · Hardware status · Clock                │
├──────────────────────────────────────────────────────────────────────┤
│  ⚠ EMERGENCY BANNER (hidden unless active fall / geofence breach)     │
│     Title · Message · [Dismiss]                                       │
├───────────────────────────────┬──────────────────────────────────────┤
│  Primary Vitals &              │  Fall Detection & Motion Dynamics     │
│  Cardiovascular Telemetry      │  - Accelerometer chart + X/Y/Z legend │
│  - HR value + status badge     │  - Gyroscope magnitude                │
│  - SpO2 value                  │  - Posture status ("Normal Gait")     │
│  - Finger status dot + text    │  - Fall status pill ("NO FALL"/ARMED) │
│  - ECG-style waveform canvas   │  - Barometric altitude chart + delta  │
├───────────────────────────────┼──────────────────────────────────────┤
│  GPS Tracking & Geofence       │  Wearable Buzzer & Hardware Actuators │
│  Perimeter                     │  - Buzzer state badge                 │
│  - Live map (patient marker)   │  - [Silence Buzzer] [Test Audio Siren]│
│  - Coordinates + GPS fix badge │  - [Reset to Normal]                  │
│  - Distance from home          │                                        │
│  - Geofence pill (Inside/Breach)│                                       │
│  - [Recenter Map] [Open in     │                                        │
│     Google Maps]               │                                        │
├───────────────────────────────┼──────────────────────────────────────┤
│  Daily Medication Adherence    │  SIM7000G GSM & Telegram Dispatch     │
│  - Scheduled item (e.g.        │  - Telegram badge / SMS status        │
│    "Multivitamin & Omega-3")   │  - [Configure Telegram Bot] → modal    │
│  - Schedule badge              │  - Send test notification              │
│  - [Take Pill]                 │                                        │
├───────────────────────────────┴──────────────────────────────────────┤
│  Serial Debug Output                                                  │
│  - Raw JSON stream console            [Force Sync]                    │
└──────────────────────────────────────────────────────────────────────┘
```

## 2. Component Inventory (by section)

### 2.1 Header / Connection Bar
| Element | Type | Behavior |
|---|---|---|
| Connect USB | Button | Opens Web Serial port picker; label toggles once connected |
| Live Hardware / Simulation | Toggle (2 buttons) | Switches data source between real serial stream and generated demo data |
| Hardware status badge | Status pill | "Awaiting Device" → "Live Feed Active" / "Wearable Telemetry Connected" |
| Clock | Text | Current time |

### 2.2 Emergency Banner
| Element | Type | Behavior |
|---|---|---|
| Title / Message | Text | Populated from the active alert (fall or geofence breach) |
| Dismiss | Button | Hides banner without clearing the underlying alert state |

### 2.3 Primary Vitals & Cardiovascular Telemetry
| Element | Type | Notes |
|---|---|---|
| Heart rate value + badge | Number + status pill | Badge reflects validity ("Optimal" etc.) |
| SpO2 value | Number | Percentage |
| Finger status dot + text | Indicator + text | "Place finger on sensor" when absent |
| ECG-style waveform | Canvas chart | Illustrative live waveform synced to BPM |

### 2.4 Fall Detection & Motion Dynamics
| Element | Type | Notes |
|---|---|---|
| Accelerometer chart | Canvas chart | X/Y/Z legend swatches |
| Gyroscope magnitude | Number | °/s |
| Posture status | Text/badge | e.g. "Normal Gait Pattern", "Stationary" |
| Fall status pill | Status pill | "NO FALL" / "ARMED" / "FALL DETECTED" |
| Altitude chart | Canvas chart | With delta tag showing change vs. baseline |

### 2.5 GPS Tracking & Geofence Perimeter
| Element | Type | Notes |
|---|---|---|
| Map | Embedded map | Patient marker, home geofence circle |
| Coordinates | Text | Lat/long |
| GPS fix badge | Status pill | Fix quality/status |
| Distance from home | Number | Meters |
| Geofence pill | Status pill | "Inside Safe Zone" / breach state |
| Recenter Map | Button | Re-centers map on current position |
| Open on Google Maps | Link | Opens external maps app/site |

### 2.6 Wearable Buzzer & Hardware Actuators
| Element | Type | Notes |
|---|---|---|
| Buzzer state badge | Status pill | Reflects `BuzzerMode` |
| Silence Buzzer | Button | Sends `BUZZ:OFF` |
| Test Audio Siren | Button | Sends a test buzzer pattern |
| Reset to Normal | Button | Clears fall/geofence state |

### 2.7 Daily Medication Adherence
| Element | Type | Notes |
|---|---|---|
| Scheduled item | Text + description | e.g. medication name/time |
| Schedule badge | Status pill | "Scheduled" |
| Take Pill | Button | Acknowledges the reminder |

### 2.8 SIM7000G GSM & Telegram Dispatch
| Element | Type | Notes |
|---|---|---|
| Telegram/SMS status badge | Status pill | Delivery status |
| Configure Telegram Bot | Button → Modal | Opens settings modal |
| — Bot Token field | Text input | Within modal |
| — Caregiver Chat ID field | Text input | Within modal |
| — Inline instructions | Text | How to create a bot / find chat ID |
| Save Settings | Button (modal) | Persists to browser session |
| Cancel / Close | Button (modal) | Dismisses without saving |
| Toast notifications | Transient UI | Confirms SMS/Telegram send |

### 2.9 Serial Debug Output
| Element | Type | Notes |
|---|---|---|
| Debug console | Scrollable text log | Raw JSON packets as received |
| Force Sync | Button | Sends `SYNC`/`TEST` command |

## 3. Simulation-Mode Controls

| Element | Type | Notes |
|---|---|---|
| Simulate Fall | Button | Forces a fall event in Simulation mode |
| Simulate Geofence Breach | Button | Forces a breach event in Simulation mode |
| Simulation notice banner | Banner | Indicates data shown is synthetic, with a return-to-live control |

## 4. Responsive/Visual Notes

- Two-column grid for the mid-page sections (vitals/fall, map/buzzer,
  medication/telegram), collapsing to a single column on narrow
  viewports.
- Status pills use consistent color coding: green/neutral = normal,
  amber = caution/pending, red = active alert.
- All charts are canvas-based and update on each telemetry tick
  (~300 ms in Live mode).

## 5. Suggested Improvements for a Future Wireframe Pass

- Add a persistent event-history panel (fall/geofence timeline) since
  the current dashboard is live-only.
- Expose geofence center/radius as editable fields (currently firmware
  constants — see `SYSTEM_DESIGN_DOCUMENT.md` §10).
- Add a battery-level indicator once MAX17048 support lands (see
  `HARDWARE.md` planned components).
- Consider a condensed "at a glance" summary card for mobile/caregiver
  quick-check use.
