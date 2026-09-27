# SAATHI — Research & References

Background research and prior art that informed SAATHI's design choices
(sensor selection, fall-detection approach, geofencing method, and
caregiver-alerting model). Summaries below are paraphrased from the
source abstracts — consult the original papers for full methodology and
results.

## Fall Detection for Dementia/Alzheimer's Patients

- **Shamsudin, A. S., & Zainal Abidin, Z. (2025).** *Real-Time Monitoring
  and Fall Detection for Alzheimer's/Dementia Patients Using Internet of
  Thing (IoT) Technology.* Universiti Tun Hussein Onn Malaysia.
  Describes a real-time monitoring and fall-detection system combining
  an MPU6050 accelerometer/gyroscope with GPS via the Blynk platform,
  tested across activity states (still, walking, jogging, running,
  falling) with accurate differentiation of falls from normal activity.
  https://penerbit.uthm.edu.my/periodicals/index.php/eeee/article/view/19683

- **Kai Sheng, Y., & Zainal, N. (2022).** *Development of Wearable
  Sensor-Based Fall Detection System for Elderly using IoT.* Universiti
  Tun Hussein Onn Malaysia.
  Presents a chest-worn IoT fall detector for elderly users that sends
  automated email and Blynk push notifications to family/caregivers,
  with an indoor Wi-Fi mode and a smartphone-hotspot fallback for
  outdoor use.
  https://periodical.uthm.edu.my/index.php/eeee/article/view/8470

- **Mishra, S., Ngangbam, B., Raj, S., & Pradhan, N. R.** *CURA: Real
  Time Artificial Intelligence and IoT based Fall Detection Systems for
  patients suffering from Dementia.* VIT-AP University. EAI Endorsed
  Transactions on Scalable Information Systems.
  Proposes an AI/IoT-based real-time fall-detection device aimed at
  reducing caregiver workload in homes and care facilities, framed as
  virtual assistance for dementia patients.
  https://eudl.eu/pdf/10.4108/eetpht.9.3967

- **"Alzo" wearable monitoring system** (Petra Christian University),
  published via TELKOMNIKA.
  Combines an IMU, GPS module, and 2G cellular transceiver on a
  belt-worn device, reporting a falling-detection algorithm that
  achieved 93.33% accuracy alongside live tracking and geolocation
  route suggestions.
  https://telkomnika.uad.ac.id/index.php/TELKOMNIKA/article/download/25156/11779

- **Karnataka State Council for Science and Technology (KSCST) student
  project report** — Alzheimer's location + vitals monitoring device.
  A Wi-Fi-microcontroller-based service model tracking Alzheimer's
  patients' vital signs (temperature, pulse, oxygen) alongside GPS and
  a fall alarm, reporting 93% fall-detection sensitivity and 95%
  specificity across 13 test volunteers.
  https://kscst.org.in/spp/46_series/46s_spp/01_Seminar_Projects/129_46S_BE_4192.pdf

**Relevance to SAATHI:** these projects validate the core sensor pairing
(accelerometer + gyroscope, optionally + barometric altitude) for fall
classification and confirm that a multi-stage/threshold approach (rather
than a single accelerometer spike) is the common pattern for reducing
false positives — the same rationale behind SAATHI's 5-state fall
machine (see `FIRMWARE.md`).

## GPS Tracking, Geofencing & Wandering Prevention

- **Geddes, J., & Warwick, K. (2010).** *Cloud based global positioning
  system as a safety monitor for dementia patients.* IEEE 9th
  International Conference on Cybernetic Intelligent Systems (CIS).
  DOI: 10.1109/UKRICIS.2010.5898085.
  Describes an SMS-based notification service that alerts carers when a
  dementia patient leaves a custom contour boundary managed remotely
  through a website.
  https://centaur.reading.ac.uk/26559

- **Deepa, S., et al.** *Dementia People Tracking System.* IOS Press.
  Combines a GPS receiver, GSM module, and RF transmitter/receiver with
  a heartbeat and temperature sensor; sounds a local buzzer and sends an
  SMS with location when the patient moves out of a defined range or a
  vital-sign anomaly occurs.
  https://ebooks.iospress.nl/pdf/doi/10.3233/APC210273

- **UCF Senior Design Group 16 — Dementia patient tracker.**
  Documents using the Haversine formula (via a GPS library's
  distance-between method) to compare current position against a home
  coordinate, switching a HOME/WANDER state and sending a text alert on
  breach — the same geofencing approach used in SAATHI's
  `calcGeofenceDist()`.
  https://ece.ucf.edu/seniordesign/fa2015sp2016/g16/docs/Group%2016%20Final%20Presentation.pdf

- **Smart location tracking system for dementia patients** (IEEE, 2017),
  VIT.
  Integrates GSM and GPS with a microcontroller so a caretaker's Android
  app can display the patient's live coordinates on Google Maps.
  https://research.vit.ac.in/publication/smart-location-tracking-system-for-dementia-patients

**Relevance to SAATHI:** confirms the Haversine/great-circle distance
method as the standard technique for geofence-breach detection on
embedded GPS trackers, and shows SMS/app notification as the established
caregiver-alerting channel that SAATHI extends with Telegram bot
integration for lower cost and richer message formatting.

## Pulse Oximetry / Heart-Rate Sensing (MAX30102)

- **Analog Devices (Maxim Integrated).** MAX30102 product page and
  datasheet.
  Documents the MAX30102 as a single-chip pulse-oximetry and heart-rate
  module with integrated LEDs, photodetectors, and ambient-light
  rejection, communicating over I2C and drawing under 1 mW in
  heart-rate-only mode.
  https://www.analog.com/en/products/max30102

- **IISc DESE Embedded Lab project notes** — MAX30102 wearable vitals
  monitor.
  Summarizes the standard PPG signal-processing pipeline for this
  sensor class: digital filtering to remove motion artifacts, peak
  detection on the waveform for heart rate, and the ratio-of-ratios
  method (AC/DC of red vs. infrared) to estimate SpO2.
  https://labs.dese.iisc.ac.in/embeddedlab/?p=5385

**Relevance to SAATHI:** the firmware's `readMAX()` routine implements
this same DC-baseline/AC-pulsatile ratio-of-ratios approach (see
`FIRMWARE.md` for the exact filter coefficients and empirical SpO2
curve used).

## Consumer Context — Existing Commercial/Hackathon Solutions

- **Rewire Security** — overview of commercial GPS trackers for
  Alzheimer's/dementia patients.
  Highlights real-time tracking, geo-fence zones with entry/exit
  alerts, and a one-touch SOS button that notifies designated contacts
  with precise location as standard features of consumer
  dementia-tracking devices.
  https://www.rewiresecurity.co.uk/blog/personal-trackers-for-alzheimers-dementia

- **MedCrack** (hackathon project) and **Dementia-Band** (student
  project) — both combine a caregiver-defined geofence with an SOS/panic
  button and automatic SMS alerting, similar in scope to SAATHI's core
  feature set.
  https://devfolio.co/projects/medcrack · https://www.makersasylum.com/project/dementia-band/

## How This Research Shaped SAATHI's Design

1. **Multi-signal fall confirmation** (freefall → impact → stillness),
   rather than a single accelerometer threshold, follows the pattern
   established across the fall-detection literature above, and is why
   SAATHI adds a barometric-altitude check as a secondary confirmation
   signal alongside the IMU.
2. **Haversine-based geofencing** with a fixed home coordinate and
   radius mirrors the UCF and Reading University approaches.
3. **Dual-channel caregiver alerting** (Telegram + SMS) builds on the
   established SMS-alert pattern but adds a richer, free messaging
   channel (Telegram) that many prior projects use Blynk push
   notifications or a custom app for instead.
4. **Sensor auto-detection with graceful fallback** (see `FIRMWARE.md`)
   is a SAATHI-specific addition not emphasized in the reviewed papers,
   aimed at keeping demos and partial builds usable.

## Suggested Further Reading (not yet reviewed in depth)

- Peer-reviewed comparisons of accelerometer-only vs. accelerometer +
  barometer fall-detection accuracy.
- Clinical validation studies of low-cost reflective PPG sensors
  (MAX30102-class) against certified pulse oximeters.
- UX research on wearable-device acceptance among dementia patients
  (comfort, stigma, ease of donning/removal).

> Update this file as you add your own literature review, cite course
> materials, or record your own testing results.
