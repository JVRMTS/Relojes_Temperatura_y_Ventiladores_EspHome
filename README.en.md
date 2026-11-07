<div align="center">

<h1>Home Clocks with ESPHome</h1>

<p><strong>Four connected clocks, an RF bridge, and a remote-capture tool.</strong></p>

<p>The Arduino-related programs previously in this repository have been removed, and their functionality has been ported to ESPHome. This repository contains ESPHome configurations for all four clocks and two CC1101 RF devices: an MQTT-to-RF bridge and a capture tool.</p>

<p>Three clocks feature an ILI9488 touchscreen and MQTT controls for the living-room and bedroom fans. The bathroom clock uses a character LCD and publishes its sensor readings. The CC1101 bridge transmits MQTT commands over 433.92 MHz RF and publishes received codes; the capture tool logs remote-control codes.</p>

<p><strong>English</strong> | <a href="./README.md">Español</a></p>

</div>

<div align="center">

<table>
  <thead>
    <tr><th>Devices</th><th>Connectivity</th><th>Time zone</th></tr>
  </thead>
  <tbody>
    <tr><td><strong>4 clocks + 2 ESP32 RF devices</strong></td><td><strong>Wi-Fi · MQTT · RF · OTA</strong></td><td><code>Europe/Madrid</code> (clocks)</td></tr>
  </tbody>
</table>

</div>

## Contents

<div align="center">

<p>
  <a href="#inventory">Inventory</a> ·
  <a href="#hardware-and-connections">Hardware</a> ·
  <a href="#network-and-secrets">Network and secrets</a> ·
  <a href="#building-and-flashing">Installation</a> ·
  <a href="#clock-features">Features</a> ·
  <a href="#mqtt-protocol">MQTT</a> ·
  <a href="#maintenance">Maintenance</a>
</p>

</div>

## Inventory

<table>
  <thead>
    <tr>
      <th>Room</th>
      <th>Configuration</th>
      <th>ESPHome name</th>
      <th>IP address</th>
      <th>Display</th>
      <th>Local sensor</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><strong>Bathroom</strong></td>
      <td><a href="./reloj_aseo_esphome.yaml">reloj_aseo_esphome.yaml</a></td>
      <td><code>reloj-aseo</code></td>
      <td><code>192.168.1.22</code></td>
      <td>20×4 LCD · I²C</td>
      <td>DHT22 · GPIO27</td>
    </tr>
    <tr>
      <td><strong>Office</strong></td>
      <td><a href="./reloj_despacho_esphome.yaml">reloj_despacho_esphome.yaml</a></td>
      <td><code>reloj-despacho</code></td>
      <td><code>192.168.1.21</code></td>
      <td>ILI9488 touchscreen</td>
      <td>DHT22 · GPIO22</td>
    </tr>
    <tr>
      <td><strong>Bedroom</strong></td>
      <td><a href="./reloj_dormitorio_esphome.yaml">reloj_dormitorio_esphome.yaml</a></td>
      <td><code>reloj-dormitorio</code></td>
      <td><code>192.168.1.23</code></td>
      <td>ILI9488 touchscreen</td>
      <td>DHT22 · GPIO22</td>
    </tr>
    <tr>
      <td><strong>Living room</strong></td>
      <td><a href="./reloj_salon_esphome.yaml">reloj_salon_esphome.yaml</a></td>
      <td><code>reloj-salon</code></td>
      <td><code>192.168.1.24</code></td>
      <td>ILI9488 touchscreen</td>
      <td>DHT22 · GPIO22</td>
    </tr>
  </tbody>
</table>

The three touchscreen clocks share the same design, hardware, and control logic, with room-specific settings for networking and MQTT topics. The bathroom clock is different: it has no touchscreen or fan controls.

### CC1101 RF bridge

The [cc1101_mqtt_central.yaml](./cc1101_mqtt_central.yaml) configuration belongs to a separate ESP32 device, not one of the clocks. It bridges MQTT and the fans' RF remotes: it listens on `casa/+/ventilador`, transmits commands over RF, and publishes received codes. It also exposes fan controls and buttons through the native ESPHome API; to see them in Home Assistant, add the device using the ESPHome integration. It uses the ESPHome name `cc1101-mqtt-central`, static IP address `192.168.1.18`, and has no display or environmental sensor.

### RF remote capture tool

The [capturar_mando_esphome.yaml](./capturar_mando_esphome.yaml) configuration is a separate tool for identifying codes from an RC-Switch-compatible remote or inspecting undecoded RF signals. It uses the ESPHome name `cc1101-captura-mando`, connects over Wi-Fi with a DHCP-assigned address, and exposes logs through the ESPHome API. It does not transmit or use MQTT. Press a remote button and inspect the device logs in ESPHome: RC-Switch codes appear with their protocol and decimal value, while the `raw` decoder helps inspect unrecognized signals.

### Image assets

The [`assets/`](./assets/) folder contains the PNG icons used by the touchscreen clocks:

<table>
  <thead>
    <tr><th>File</th><th>Purpose</th></tr>
  </thead>
  <tbody>
    <tr><td><code>icono_temperatura.png</code></td><td>Temperature</td></tr>
    <tr><td><code>icono_humedad.png</code></td><td>Humidity</td></tr>
    <tr><td><code>icono_sensacion.png</code></td><td>Heat index</td></tr>
    <tr><td><code>logo_reloj.png</code></td><td>Clock logo</td></tr>
  </tbody>
</table>

The bathroom clock does not use these assets.

## Requirements and dependencies

- ESPHome 2026.9.1 or later.
- An ESP32 compatible with `esp32dev`; ESP-IDF framework.
- Wi-Fi network and an accessible MQTT broker with valid credentials.
- **RF devices:** CC1101 module compatible with 433.92 MHz.
- **Office, bedroom, and living room:** ILI9488 SPI display, XPT2046 touchscreen, DHT22 sensor, and the `assets/` folder alongside the YAML files.
- **Bathroom:** 20×4 character LCD with a PCF8574 expander (`0x27`), DHT22, photoresistor, and backlight circuit.

> [!NOTE]
> The touchscreen YAML files download Roboto fonts from Google Fonts during compilation. The build machine needs Internet access the first time.

## Hardware and connections

<details>
<summary><strong>Show wiring and technical notes</strong></summary>

### Office, bedroom, and living room

<table>
  <thead>
    <tr><th>Function</th><th>Connection</th></tr>
  </thead>
  <tbody>
    <tr><td>SPI CLK</td><td>GPIO18</td></tr>
    <tr><td>SPI MOSI</td><td>GPIO23</td></tr>
    <tr><td>SPI MISO</td><td>GPIO19</td></tr>
    <tr><td>ILI9488 CS</td><td>GPIO15</td></tr>
    <tr><td>ILI9488 D/C</td><td>GPIO2</td></tr>
    <tr><td>ILI9488 RESET</td><td>GPIO4</td></tr>
    <tr><td>XPT2046 CS</td><td>GPIO21</td></tr>
    <tr><td>DHT22</td><td>GPIO22</td></tr>
  </tbody>
</table>

LVGL uses a 90-degree rotation and the `mirror_x` transform; the touchscreen uses `mirror_y` and the calibration limits defined in each YAML file. If the wiring or physical orientation changes, check the rotation and recalibrate the touchscreen.

> [!WARNING]
> GPIO15 and GPIO2 are ESP32 strapping pins. Keep the intended wiring and avoid loads that could alter their levels during startup.

### CC1101 RF bridge

<table>
  <thead>
    <tr><th>Function</th><th>Connection</th></tr>
  </thead>
  <tbody>
    <tr><td>SPI CLK</td><td>GPIO18</td></tr>
    <tr><td>SPI MISO</td><td>GPIO19</td></tr>
    <tr><td>SPI MOSI</td><td>GPIO23</td></tr>
    <tr><td>CC1101 CS</td><td>GPIO5</td></tr>
    <tr><td>CC1101 GDO0 / transmit and receive</td><td>GPIO4</td></tr>
  </tbody>
</table>

The module is configured for 433.92 MHz, output power 10, and an 812 kHz filter bandwidth. GPIO4 is shared by the transmitter and receiver; keep `allow_other_uses` in both configurations. Check the module's power supply, logic levels, and wiring before powering it on.

The capture tool uses the same SPI bus and CS on GPIO5; connect the RF receiver's data output to GPIO4. It is configured for 433.92 MHz with an 812 kHz filter bandwidth and does not transmit.

### Bathroom

<table>
  <thead>
    <tr><th>Function</th><th>Connection</th></tr>
  </thead>
  <tbody>
    <tr><td>20×4 LCD with PCF8574</td><td>I²C, address <code>0x27</code></td></tr>
    <tr><td>I²C SDA</td><td>GPIO21</td></tr>
    <tr><td>I²C SCL</td><td>GPIO22</td></tr>
    <tr><td>DHT22</td><td>GPIO27</td></tr>
    <tr><td>Photoresistor (ADC)</td><td>GPIO32</td></tr>
    <tr><td>Backlight</td><td>GPIO18, LEDC output</td></tr>
  </tbody>
</table>

The photoresistor automatically adjusts the backlight. This clock does not use the SPI bus used by the other three units.

### Board and minimum chip revision

All six configurations declare `board: esp32dev`, but the minimum chip revision is set per device. `minimum_chip_revision` is a requirement for the ESP32 silicon revision, not the physical board's name or model. Set this value according to the actual chip revision in each device: identify the revision and do not copy another unit's value without checking it.

<table>
  <thead>
    <tr><th>Device</th><th>Board declared in YAML</th><th><code>minimum_chip_revision</code></th><th>Status</th></tr>
  </thead>
  <tbody>
    <tr><td>Bathroom</td><td><code>esp32dev</code></td><td><code>3.0</code></td><td>Commented out; not applied</td></tr>
    <tr><td>Office</td><td><code>esp32dev</code></td><td><code>3.1</code></td><td>Active</td></tr>
    <tr><td>Bedroom</td><td><code>esp32dev</code></td><td><code>3.0</code></td><td>Commented out; not applied</td></tr>
    <tr><td>Living room</td><td><code>esp32dev</code></td><td><code>3.0</code></td><td>Active</td></tr>
    <tr><td>CC1101 bridge</td><td><code>esp32dev</code></td><td><code>3.1</code></td><td>Active</td></tr>
    <tr><td>CC1101 capture tool</td><td><code>esp32dev</code></td><td><code>3.0</code></td><td>Active</td></tr>
  </tbody>
</table>

Do not set a minimum above the chip's actual revision. If you replace a board, check its revision again before reusing the configuration.

> [!TIP]
> **First installation on a board with an unknown revision:** comment out the `minimum_chip_revision` line with `#` before building. Build and flash the firmware, then open the device logs in **ESPHome Device Builder**. At startup, the logs show the ESP32 revision, for example `ESP32 r3.0`. Next, uncomment the line and enter your unit's revision (`minimum_chip_revision: "3.0"` in this example); build and flash again to apply the setting. Building alone cannot detect the revision of a board that has not been queried.

</details>

## Network and secrets

<details>
<summary><strong>Show network and security settings</strong></summary>

The four clock configurations and the MQTT-to-RF bridge use static IP addresses on `192.168.1.0/24` and gateway `192.168.1.1`; the capture tool gets its address via DHCP. The four clocks specify DNS servers `192.168.1.201` and `192.168.1.204`. The clocks and MQTT-to-RF bridge use broker `192.168.1.203:1883`; the capture tool does not connect to MQTT. Check the addresses and adapt them to your network before flashing the firmware.

Create `secrets.yaml` in the ESPHome configuration directory. Do not store real credentials in the repository:

```yaml
wifi_ssid: "your-wifi-network"
wifi_password: "your-wifi-password"
mqtt_username: "mqtt-user"
mqtt_password: "mqtt-password"
api_key_reloj: "valid-32-byte-base64-api-key"
```

OTA uses `encryption:` and the API encryption key, so no additional OTA password is needed. The Wi-Fi fallback access point password is separate from OTA.

All four clocks synchronize their time via SNTP using the `Europe/Madrid` time zone; neither RF device has a clock or time synchronization.

</details>

## Building and flashing

<details>
<summary><strong>Show installation instructions</strong></summary>

Save the YAML files in the ESPHome configuration directory and place `assets/` alongside the three touchscreen configurations, preserving the relative paths. Build each configuration individually:

```shell
esphome compile reloj_aseo_esphome.yaml
esphome compile reloj_despacho_esphome.yaml
esphome compile reloj_dormitorio_esphome.yaml
esphome compile reloj_salon_esphome.yaml
esphome compile cc1101_mqtt_esphome.yaml
esphome compile capturar_mando_esphome.yaml
```

- **OTA:** the device must be connected to the network and its OTA service enabled.
- **First flash:** a USB connection and selecting the correct port may be required.
- **Network changes:** review the network values declared in the relevant YAML before building.

The touchscreen configurations use ESP-IDF and assign SRAM1 as IRAM to provide memory for the display. The LVGL buffer is limited to 10% to reduce pressure on internal RAM on boards without PSRAM. See the board and chip revision table in the hardware section before changing `esp32.framework.advanced` or moving firmware to another board.

</details>

## Clock features

<details>
<summary><strong>Show features for each model</strong></summary>

### Office, bedroom, and living room

The home screen shows the time, date, temperature, humidity, and heat index. It can display MQTT readings from the rooms and open the controls for the bedroom or living-room fan. The local DHT22 is sampled every 10 seconds.

The fan-control screen has four rows:

1. Main light, back, and fan.
2. Decrease brightness, yellow/white/blue lights, and increase brightness.
3. Speeds 1 to 5.
4. Reverse direction and 2-, 4-, and 8-hour timers.

The interface is not scrollable. The selected fan determines the destination MQTT topic. Published codes are specific to each fan: the codes configured here are the ones I obtained for my own fans and will not work with other fans. Each user must obtain the codes for their own fans and adapt the `enviar_comando_ventilador` table to their receiver.

### Bathroom

The LCD shows the time, date, temperature, humidity, and heat index. The photoresistor on GPIO32 adjusts the backlight through GPIO18. This device publishes readings via MQTT and does not include graphical fan controls.

</details>

## MQTT protocol

<details>
<summary><strong>Show topics and message formats</strong></summary>

All clocks use the `192.168.1.203:1883` broker. They publish every two seconds with QoS 2 and `retain: true`, when valid temperature and humidity readings are available.

### Measurement topics

<table>
  <thead>
    <tr><th>Room</th><th>Topic</th></tr>
  </thead>
  <tbody>
    <tr><td>Bathroom</td><td><code>casa/aseo/variables</code></td></tr>
    <tr><td>Office</td><td><code>casa/despacho/variables</code></td></tr>
    <tr><td>Bedroom</td><td><code>casa/dormitorio/variables</code></td></tr>
    <tr><td>Living room</td><td><code>casa/salon/variables</code></td></tr>
  </tbody>
</table>

Example JSON payload:

```json
{"temperatura":22.5,"humedad":48.0,"sen_termica":22.1,"WiFi_Signal":-55}
```

Temperature and heat index are in °C; humidity is in %; and `WiFi_Signal` is in dBm. The office clock listens to the living-room, bathroom, and bedroom topics. The bedroom and living-room clocks listen to all four rooms. Before data arrives, the interface may show `--.-`; missing keys do not overwrite the last received value for that measurement.

### Fan-control topics

The touchscreen clocks send `{"opcion": <code>}` to either `casa/dormitorio/ventilador` or `casa/salon/ventilador`, depending on the selected fan.

<table>
  <thead>
    <tr><th>Internal index</th><th>Action</th></tr>
  </thead>
  <tbody>
    <tr><td>0</td><td>Main light</td></tr>
    <tr><td>1</td><td>Fan</td></tr>
    <tr><td>2–4</td><td>Yellow, white, and blue lights</td></tr>
    <tr><td>5–6</td><td>Decrease and increase brightness</td></tr>
    <tr><td>7–11</td><td>Speeds 1–5</td></tr>
    <tr><td>12</td><td>Reverse direction</td></tr>
    <tr><td>13–15</td><td>2-, 4-, and 8-hour timers</td></tr>
  </tbody>
</table>

Commands are published with QoS 2 and `retain: false` to prevent the broker from replaying an old command after reconnecting. The `{"opcion":0}` value is ignored as a reset placeholder and is not transmitted over RF. Final numeric codes differ between the bedroom and living-room fans and are defined in each YAML file. These are codes for my own fans, not universal codes: each user must obtain the codes for each of their fans and configure their own before using the controls.

### CC1101 RF bridge

The bridge subscribes to `casa/+/ventilador` and accepts a JSON object with `opcion` as an unsigned 32-bit integer, for example `{"opcion": 123456}`. It converts the integer to a 32-bit binary sequence and transmits it using the RC-Switch protocol configured in the YAML (390 µs pulse, `[23, 1]` sync, `zero: [1, 2]` and `one: [2, 1]`, inverted signal, and two repeats). The RF parameters and transmitted code must match the receiver and remote being controlled; they are not universal.

When an RC-Switch code is received, the bridge publishes a JSON object containing the protocol and code to `casa/ventilador/mando`, for example `{"protocolo":1,"codigo":"123456"}`. This publication uses QoS 0 and is not retained. The bridge also updates the ESPHome **Último código mando ventilador** text sensor and emits the Home Assistant event `esphome.rf_fan_remote_received` with `protocol` and `code` fields.

### RF remote capture tool

The capture tool is not part of the MQTT flow. Connect to it through the ESPHome API and open the device logs while pressing remote buttons: `dump: [rc_switch, raw]` reports codes it can decode as RC-Switch and displays raw signal data when it cannot recognize the protocol. Record the protocol and code so they can be configured on the transmitting device; signals and codes depend on the remote.

</details>

## Maintenance

<details>
<summary><strong>Show maintenance recommendations</strong></summary>

- When moving a clock to another network, update its IP, gateway, subnet mask, DNS, and broker; avoid duplicate static addresses.
- When changing a sensor or board, update the GPIOs in the relevant YAML and check the wiring.
- To replace icons, keep the PNG filenames referenced in the `image` section.
- Adjust fonts, orientation, and calibration under `lvgl`/`touchscreen`; for the bathroom clock, update the ADC sensor and `light` output.
- If the MQTT or fan-control protocol changes, update the corresponding publishers and consumers and validate the installation before deployment.

</details>
