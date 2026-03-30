# 🌱 FarmBot — Autonomous Plant Watering Robot

<img width="820" height="683" alt="image" src="https://github.com/user-attachments/assets/ba785699-8f4f-40bf-9ea6-1e6f02b77a4d" />
> ESP32-based roving robot with soil & air sensing, automated watering, and Firebase cloud sync (image is just illustration)

---

## 📖 Overview

FarmBot is an autonomous robot that navigates a plant environment, monitors soil moisture and ambient conditions, and decides when and where to water — syncing all sensor data to Firebase Realtime Database for remote monitoring.

The system uses a **dual ESP32 architecture**:
- **Sender ESP32** — reads 4 soil moisture sensors + DHT11 (temp & humidity), pushes data to Firebase every 3 seconds
- **Receiver ESP32** — reads Firebase data and triggers the water pump or mist sprayer based on thresholds

---

## ❓ Problem Statement

Manual plant watering is inconsistent — plants get over- or under-watered depending on human schedule. In multi-plant environments this is especially hard to maintain, leading to plant stress or crop loss.

<img width="709" height="945" alt="image" src="https://github.com/user-attachments/assets/9801da6d-1db3-40f9-a591-223df6b78199" />
> *Can a low-cost autonomous robot equipped with environmental sensors automatically determine which plants need water, dispatch water or mist accordingly, and report real-time conditions to a cloud dashboard — without human intervention?*

---

## 🎯 Goals

| | Goal |
|---|---|
| **Primary** | Build a roving robot that autonomously monitors and waters plants based on real-time soil moisture and humidity readings |
| **Secondary** | Stream all sensor telemetry to Firebase RTDB for remote monitoring from any device |

---

## ✅ Objectives

- Read soil moisture from **4 capacitive sensors** (pins D34, D35, D32, D33) every 3 seconds
- Read ambient temperature and humidity from **DHT11** sensor (pin D15)
- Push all sensor readings to **Firebase Realtime Database** via ESP32 Wi-Fi
- Second ESP32 reads Firebase and triggers **water pump** or **mist sprayer** based on thresholds
- Robot navigates the plant environment in a loop, stopping at each plant station to sense and act
- Use mist sprayer to raise ambient humidity when DHT11 readings fall below target threshold

---

## ✨ Features

- 🤖 Autonomous plant environment patrol — robot navigates between plant stations in a loop
- 💧 4-channel soil moisture sensing via capacitive sensors on ESP32 analog pins
- 🌡️ Ambient temperature and humidity monitoring via DHT11
- ☁️ Real-time data push to Firebase Realtime Database (Asia Southeast 1 region)
- 📊 Indexed historical data storage per sensor per reading cycle (`soilMoisture_Data/1–5/Moisture`)
- 🔌 Second ESP32 as actuator controller — reads Firebase and drives water pump + mist sprayer
- 🔁 Wi-Fi reconnect handling and Firebase anonymous sign-in
- 💡 On-board LED heartbeat during active Firebase write

---

## 🛠️ Tech Stack

| Category | Technology |
|---|---|
| Microcontroller | ESP32 (dual-board: sender + receiver) |
| Firmware | Arduino IDE / ESP32 Arduino Core 2.x |
| Cloud Database | Firebase Realtime Database (asia-southeast1) |
| Firebase Library | [mobizt/Firebase-ESP-Client](https://github.com/mobizt/Firebase-ESP-Client) |
| Environmental Sensor | DHT11 + Adafruit DHT Library |
| Moisture Sensors | Capacitive Soil Moisture Sensor x4 |
| Actuators | Water pump + relay, Mist sprayer + relay |
| Connectivity | Wi-Fi 2.4GHz (IEEE 802.11 b/g/n) |

---

## 📡 Sensor & Firebase Data Schema

```
Firebase RTDB
├── soilMoisture1_Data/
│   ├── 1/Moisture   (int)
│   ├── 2/Moisture   (int)
│   └── .../
├── soilMoisture2_Data/  ...
├── soilMoisture3_Data/  ...
├── soilMoisture4_Data/  ...
└── DHT11_Data/
    ├── 1/Humidity     (int)
    ├── 1/Temperature  (float)
    └── .../
```

Readings are updated every **3 seconds**. Each cycle stores **5 indexed snapshots** per sensor using a for-loop (`i = 1` to `5`).

---

## 🔌 Hardware Wiring

| Sensor / Module | ESP32 Pin |
|---|---|
| Soil Moisture Sensor 1 | D34 |
| Soil Moisture Sensor 2 | D35 |
| Soil Moisture Sensor 3 | D32 |
| Soil Moisture Sensor 4 | D33 |
| DHT11 Data | D15 |
| On-board LED | D2 |
| Water Pump Relay | *(receiver ESP32)* |
| Mist Sprayer Relay | *(receiver ESP32)* |

> ⚠️ Pins D34, D35 are input-only on ESP32 — do not use as output.

---

## 📁 Project Structure

```
farmbot/
├── esp_to_firebase_dht_6_soilmoisture_for_loop.ino   # Sender ESP32 — sensor reading & Firebase push
├── firebase_to_esp_read_dht_6soilsensor.ino           # Receiver ESP32 — Firebase read & actuator control
└── addons/
    ├── TokenHelper.h    # Firebase token generation helper
    └── RTDBHelper.h     # Firebase RTDB payload helper
```

---

## 🚀 Getting Started

### Prerequisites

1. Install **Arduino IDE 2.x**
2. Add ESP32 board support in Arduino IDE:
   ```
   https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json
   ```
3. Install required libraries via Arduino Library Manager:
   - `Firebase ESP Client` by Mobizt
   - `DHT sensor library` by Adafruit
4. Create a **Firebase project** and enable Realtime Database (region: `asia-southeast1`)

### Configuration

In **both** `.ino` files, update the following defines:

```cpp
#define WIFI_SSID       "your_wifi_ssid"
#define WIFI_PASSWORD   "your_wifi_password"
#define API_KEY         "your_firebase_api_key"
#define DATABASE_URL    "https://your-project-default-rtdb.asia-southeast1.firebasedatabase.app/"
```

### Flashing

```bash
# Upload to Sender ESP32
# Open esp_to_firebase_dht_6_soilmoisture_for_loop.ino → Select board → Upload

# Upload to Receiver ESP32
# Open firebase_to_esp_read_dht_6soilsensor.ino → Select board → Upload
```

Both boards will auto-connect to Wi-Fi and Firebase on power-up.

### Monitoring

- Open **Arduino Serial Monitor** at `115200` baud for live sensor logs
- Open **Firebase Console** → Realtime Database for live data tree view

---

## 💡 Key Insights

- **Dual ESP32 decoupling** — separating sensing from actuation makes the system modular and fault-tolerant; one board failure doesn't crash the whole pipeline
- **Indexed RTDB paths** — `soilMoisture1_Data/1–5/Moisture` provides lightweight rolling history without a separate time-series DB
- **For-loop snapshots** — the sender collects 5 readings per cycle, enabling downstream averaging before triggering actuators
- **Capacitive > resistive sensors** — capacitive sensors last significantly longer in wet soil and don't corrode
- **Mist vs. pump** — mist sprayer provides fine-grained humidity control, critical for seedlings or enclosed grow-box environments
- **NaN guarding** — failed reads default to `0` before Firebase write, preventing null/corrupt entries that would break receiver logic

---

## 🔮 Future Work

- [ ] Add motor driver + wheels firmware for true autonomous navigation between plant stations
- [ ] Implement soil moisture threshold calibration (map raw ADC 0–4095 to % moisture per sensor)
- [ ] Build Flutter / React Native mobile dashboard consuming Firebase RTDB in real-time
- [ ] Integrate ESP32-CAM for visual plant health monitoring
- [ ] Replace rule-based watering logic with a lightweight ML model trained on historical moisture + growth data
- [ ] Add battery management and solar charging for outdoor deployment
- [ ] Expand to 6 soil moisture sensors *(pins already stubbed in code — uncomment to enable)*

---

## 👤 Author

**Raphael Nazareth** — Computer Engineer | AI / Data Science

[![GitHub](https://img.shields.io/badge/GitHub-RaphaelNazareth-181717?style=flat-square&logo=github)](https://github.com/RaphaelNazareth)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-raphael--nazareth-0A66C2?style=flat-square&logo=linkedin)](https://www.linkedin.com/in/raphael-nazareth)
[![Email](https://img.shields.io/badge/Email-Raph00707@gmail.com-D14836?style=flat-square&logo=gmail&logoColor=white)](mailto:Raph00707@gmail.com)
