# ESP Home Projects

A collection of [ESPHome](https://esphome.io/) configurations for ESP32-C5-based home automation devices, integrated with Home Assistant.

## Projects

| File | Description |
|------|--------------|
| [`garage-controller.yaml`](./garage-controller.yaml) | Garage door controller: door open/closed sensor, relay-controlled opener, ToF distance sensor for car presence detection, BME680 air quality sensor with fire/smoke warning logic |
| [`gwf-water-meter.yaml`](./gwf-water-meter.yaml) | Wireless water meter reader for a GWF Sonico NANO meter via wM-Bus (868 MHz) |
| [`waterproof-ultrasonic-range-finder.yaml`](./waterproof-ultrasonic-range-finder.yaml) | Standalone ultrasonic distance sensor with configurable color-coded thresholds (green/blue/red) stored persistently in flash |

All three run on an **ESP32-C5-DevKitC-1** board and expose a local web server (port 80) in addition to the Home Assistant native API.

### ESP32-C5-DevKitC-1 pinout

![ESP32-C5-DevKitC-1 pinout](https://raw.githubusercontent.com/olaoups/esp-home-projects/main/ESP32-C5-DevKitC-1-pinout.jpg)

---

## 1. Garage Controller

**Hardware:**
- ESP32-C5-DevKitC-1
- VL53L1X Time-of-Flight distance sensor (car presence)
- BME680 environmental sensor (temperature, humidity, pressure, IAQ, CO₂ equivalent)
- Reed switch / door contact (garage door open/closed)
- Relay (garage door opener trigger)
- WS2812 RGB LED (status indicator)

**Features:**
- Detects car presence via the ToF sensor and drives the LED color accordingly (red = arriving, green = present)
- Monitors air quality (IAQ index translated into human-readable French labels) with a smoke/fire alert derived from a rapid temperature rise combined with an elevated CO₂ equivalent
- Momentary relay trigger to open/close the garage door (500 ms pulse)
- Custom external component for the VL53L1X sensor

**Sensor pinouts:**

<table>
<tr>
<td><img src="https://raw.githubusercontent.com/olaoups/esp-home-projects/main/TOF400c-pinout.png" alt="TOF400c (VL53L1X) pinout" width="400"></td>
<td><img src="https://raw.githubusercontent.com/olaoups/esp-home-projects/main/bme680-pinout.jpg" alt="BME680 pinout" width="400"></td>
</tr>
<tr>
<td align="center">TOF400c (VL53L1X)</td>
<td align="center">BME680</td>
</tr>
</table>

**Note:** `GPIO5` is reserved for boot and must not be reused.

---

## 2. GWF Water Meter Reader

**Hardware:**
- ESP32-C5-DevKitC-1
- CC1101 radio module (SPI) for wM-Bus reception at 868.95 MHz
- WS2812 RGB LED (status indicator)

**Features:**
- Reads a **GWF Sonico NANO** water meter over wireless M-Bus (wM-Bus), mode C1
- Uses the [`esphome-components`](https://github.com/SzczepanLeon/esphome-components) external component for `wmbus_radio` / `wmbus_meter`
- Publishes total consumption (m³) and radio signal strength (RSSI) to Home Assistant

**Important:** This meter requires an AES-128 encryption key to decode readings. Without it (`key:` left commented out), the configuration compiles and runs, but frames cannot be decrypted. Add your meter's key as a secret before expecting real data.

**Radio module pinout:**

![CC1101 868 MHz radio module pinout](https://raw.githubusercontent.com/olaoups/esp-home-projects/main/CC1101-868mhz-radio-module-pinout.jpg)

**Note:** `GPIO5` is reserved for boot and must not be reused.

---

## 3. Waterproof Ultrasonic Range Finder

**Hardware:**
- ESP32-C5-DevKitC-1
- Waterproof ultrasonic sensor (e.g. JSN-SR04T), trigger/echo pins
- WS2812 RGB LED (status indicator)

**Features:**
- Measures distance (cm) with median filtering to reject noise/outliers
- Three configurable distance thresholds (green / blue / red), adjustable live from Home Assistant or the local web UI, and persisted across reboots (NVS)
- LED lights up in the corresponding color based on the last measured distance, or turns off beyond the red threshold

**Note:** this file is set up as a **standalone/offline demo config** — WiFi credentials, the Home Assistant API encryption key, and the web server auth are commented out, and the AP password is hardcoded (`12345678`) for quick bring-up on the bench. Fill in your own `secrets.yaml` values and set a real AP password before any real-world / networked deployment.

---

## Setup

1. Install ESPHome:

   ```bash
   pip install -U esphome
   ```

2. Create a `secrets.yaml` file in the same directory as these configs, containing:

   ```yaml
   wifi_ssid24: "your-2.4ghz-ssid"
   wifi_password24: "your-wifi-password"
   ha_api_key: "your-home-assistant-api-encryption-key"
   web_server_username: "your-web-ui-username"
   web_server_password: "your-web-ui-password"
   ```

3. Flash a device:

   ```bash
   esphome run garage-controller.yaml
   ```

## Wiring reference

Pin assignments are documented as comments inside each YAML file (I2C, SPI, GPIO). Double-check the ESP32-C5 pinout above before wiring, and avoid `GPIO5` on all boards in this repo, as it's used during boot.

## Disclaimer

These configurations are shared as-is for reference and inspiration. Adjust pins, addresses, and thresholds to match your own hardware and wiring.
