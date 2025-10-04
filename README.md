# Nexus-22 ESPHome Fan Controller

This repository contains the ESPHome configuration for a custom **ESP32-based fan controller** that manages the cooling fans in the Nexus-22 server rack.

The controller drives **four fans via a single fan hub**. All fans run at the same speed, which can either be set dynamically by Home Assistant or automatically through an onboard fallback fan curve.

---

## ✨ Features

- **Dual control modes**
  - **Home Assistant Override**  
    Listens for a heartbeat signal (`input_boolean.ha_heartbeat`) from Home Assistant.  
    If active, fan speed is fully controlled by Home Assistant.
  - **Onboard Fallback Curve**  
    If the heartbeat is missed for >30 seconds, the controller automatically switches to an internal fan curve to ensure safe cooling.

- **Safety-first design**  
  Prevents overheating even if Home Assistant is offline or unavailable.

- **Fan curve (°F)**
  - >105°F → 100%  
  - >95°F → 85%  
  - >85°F → 70%  
  - >75°F → 50%  
  - >Else → 30%

- **Sensors & telemetry**
  - DS18B20 temperature sensor for rack monitoring (converted to °F).  
  - Virtual RPM calculation (`speed * 18`) for visibility in Home Assistant.  
  - Text sensor for raw fan PWM %.  

- **Controls**
  - **Fan Override Enabled** switch.  
  - **Fan Speed Override** number entity (0–100%).  

---

## 🛠 Hardware

- **Board**: ESP32 Devkit (ESP-IDF framework).  
- **Fans**: 4× PWM fans connected via a hub (single PWM channel).  
- **Temperature Sensor**: DS18B20 (1-Wire, GPIO 4).  
- **PWM Pin**: GPIO 18 @ 25 kHz.  

---

## 📡 Connectivity

- **Wi-Fi**: Credentials managed via `!secret` entries.  
- **Home Assistant API**: Encrypted key required (`!secret encryption_key`).  
- **OTA Updates**: Enabled with password protection.  
- **Fallback Access Point**: Captive portal enabled if Wi-Fi fails.  

---

## 🔄 Heartbeat Logic

- Every **30s**, the ESPHome node clears `input_boolean.ha_heartbeat`.  
- If HA is online, it resets the boolean, keeping the override alive.  
- If no reset occurs within 30s, the ESPHome node assumes HA is offline and switches to the onboard curve.  
- When HA heartbeat resumes, override mode re-engages automatically.  

---

## 🚀 Getting Started

1. Clone this repo and ensure you have [ESPHome](https://esphome.io/) installed.  
2. Update `secrets.yaml` with your values:
   ```yaml
   encryption_key: "..."
   ota_password: "..."
   iot_wifi_ssid: "..."
   iot_wifi_password: "..."
   ap_ssid: "..."
   ap_password: "..."
3. Deploy to your ESP32:

   ``` bash
   esphome run nexus-22-fan-controller-top.yaml
5. In Home Assistant, create an input_boolean.ha_heartbeat entity to act as the heartbeat.
