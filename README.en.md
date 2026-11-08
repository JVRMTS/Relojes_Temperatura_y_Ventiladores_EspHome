<div align="center">

<h1>Home Clocks with ESPHome</h1>

<p><strong>Four connected clocks, an RF bridge, and a remote-capture tool.</strong></p>

<p>This repository contains ESPHome configurations for all four clocks and two CC1101 RF devices: an MQTT-to-RF bridge and a capture tool.</p>

<p>Three clocks feature an ILI9488 touchscreen and MQTT controls for the living-room and bedroom fans. The bathroom clock uses a character LCD and publishes its sensor readings. The CC1101 bridge transmits MQTT commands over 433.92 MHz RF and exposes Home Assistant controls; the capture tool logs remote-control codes.</p>

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
  <a href="#customizing-fan-codes">RF codes</a> ·
  <a href="#home-assistant-entities">Home Assistant</a> ·
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

Before each RF command, the bridge queries the Zigbee2MQTT switches at `zigbee2mqtt/Ventilador Dormitorio` and `zigbee2mqtt/Ventilador Salon` through their `/get` topics. If the switch is off or does not respond within 2 seconds, it publishes `{"state":"ON"}` to the corresponding `/set` topic. It waits up to 10 seconds for `ON` confirmation, then allows 0.5 seconds for the RF receiver to start. Without power confirmation, it cancels the command and logs a warning. This applies to both Home Assistant controls and clocks; commands are queued.


The [cc1101_mqtt_central.yaml](./cc1101_mqtt_central.yaml) configuration runs on a separate ESP32, `cc1101-mqtt-central`, at `192.168.1.18`. It receives MQTT actions on `casa/dormitorio/ventilador/set` and `casa/salon/ventilador/set`, resolves their codes from [`assets/codigos_ventiladores.yaml`](./assets/codigos_ventiladores.yaml), and transmits them over RF. Fans, selectors, and buttons are also exposed through the ESPHome API for Home Assistant control. The bridge handles RF transmission. Remote capture uses the separate device below.

The bridge ESP32 also acts as an active Bluetooth proxy for Home Assistant, with active scanning configured and up to three simultaneous connections (`max_connections: 3` and `connection_slots: 3`). It shares the device with MQTT control and RF transmission.

Light color and timer selectors execute commands through `set_action` when a selection is requested from Home Assistant or MQTT. Publishing their initial state does not transmit an RF command. Their state is optimistic: it represents the requested selection, without confirmation from the fan receiver.

### RF remote capture tool

The [capturar_mando_esphome.yaml](./capturar_mando_esphome.yaml) configuration is a separate tool for identifying codes from an RC-Switch-compatible remote or inspecting undecoded RF signals. It uses the ESPHome name `cc1101-captura-mando`, connects over Wi-Fi with a DHCP-assigned address, and exposes logs through the ESPHome API. It does not transmit or use MQTT. Press a remote button and inspect the device logs in ESPHome: RC-Switch codes appear with their protocol and decimal value, while the `raw` decoder helps inspect unrecognized signals.

### Shared assets

The [`assets/`](./assets/) folder contains the touchscreen PNGs and the bridge RF code file:

<table>
  <thead>
    <tr><th>File</th><th>Purpose</th></tr>
  </thead>
  <tbody>
    <tr><td><code>icono_temperatura.png</code></td><td>Temperature</td></tr>
    <tr><td><code>icono_humedad.png</code></td><td>Humidity</td></tr>
    <tr><td><code>icono_sensacion.png</code></td><td>Heat index</td></tr>
    <tr><td><code>logo_reloj.png</code></td><td>Clock logo</td></tr>
    <tr><td><a href="./assets/codigos_ventiladores.yaml"><code>codigos_ventiladores.yaml</code></a></td><td>32 editable RF codes: 16 actions per fan</td></tr>
  </tbody>
</table>

The bathroom clock does not use these assets.

## Requirements and dependencies

- ESPHome 2026.10.2 (tested version).
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
    <tr><td>CC1101 TX data input (remote_transmitter)</td><td>GPIO2</td></tr>
  </tbody>
</table>

The bridge transmits on GPIO2 using ASK/OOK at 433.92 MHz, output power 10, symbol rate 5000, and an 812 kHz filter. Packet mode, CRC, and whitening are disabled. `on_transmit` enables TX and `on_complete` returns the CC1101 to idle. This YAML has no `remote_receiver` or shared RX pin.

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

The four clock configurations and the MQTT-to-RF bridge use static IP addresses on `192.168.1.0/24` and gateway `192.168.1.1`; the capture tool gets its address via DHCP. The four clocks specify DNS servers `192.168.1.201` and `192.168.1.204`. The clocks and MQTT-to-RF bridge use the broker defined by `!secret broker`, on port `1883`; the capture tool does not connect to MQTT. Check the addresses and adapt them to your network before flashing the firmware.

Device YAMLs read credentials through `!secret` (for example, `broker: !secret broker`); they do not contain the broker address. Create **`secrets.yaml`** with your own values. The following block is an example of that file, not of the device YAML. Do not publish your secrets in the repository:

```yaml
wifi_ssid: "your-wifi-network"
wifi_password: "your-wifi-password"
mqtt_username: "mqtt-user"
mqtt_password: "mqtt-password"
broker: "your-mqtt-broker.local" # Replace with your broker hostname or address
api_key_reloj: "valid-32-byte-base64-api-key"
```

OTA uses `encryption:` and the API encryption key, so no additional OTA password is needed. The Wi-Fi fallback access point password is separate from OTA.

All four clocks synchronize their time via SNTP using the `Europe/Madrid` time zone; neither RF device has a clock or time synchronization.

</details>

## Building and flashing

<details>
<summary><strong>Show installation instructions</strong></summary>

Place the six main YAML files and the `assets/` folder in the ESPHome configuration directory, preserving relative paths. Touchscreen clocks need the PNGs; the bridge needs `assets/codigos_ventiladores.yaml`, an included package rather than a separately compiled device. In Device Builder, paste each complete YAML and copy the assets too. Build each device individually:

```shell
esphome compile reloj_aseo_esphome.yaml
esphome compile reloj_despacho_esphome.yaml
esphome compile reloj_dormitorio_esphome.yaml
esphome compile reloj_salon_esphome.yaml
esphome compile cc1101_mqtt_central.yaml
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

1. Fan light, back, and fan on/off.
2. Decrease brightness, yellow/white/blue lights, and increase brightness.
3. Speeds 1 to 5.
4. Reverse direction and 2-, 4-, and 8-hour timers.

The interface is not scrollable. The local-room view shows the time in the heading; selecting another room shows its name instead, with the date below. Temperature and heat index appear as `23.4°`, without `C`; humidity appears as `48.0%`. Each icon and value are centered together horizontally and vertically, with a 4-pixel gap. Display data is refreshed every second.

RF control buttons use a debounce filter: one command is accepted per contact, and input is rearmed after **200 ms with no touch**. A new contact during this interval cancels rearming. The `tactil` log distinguishes accepted and ignored commands. This filter covers all 16 fan and light actions, not room selection, navigation, or Back. Back only changes the page.

Clocks send action names rather than numeric RF codes. Customize all codes in `assets/codigos_ventiladores.yaml`; the clocks' tables do not need editing.

### Bathroom

The LCD shows time, date, temperature, humidity, and heat index. The GPIO32 photoresistor is read every second. It exposes `Luz ambiente relativa` as a percentage of the ADC range (0–4095 → 0–100 %), not volts or lux; the raw reading is internal. The original reading drives the GPIO18 backlight PWM (5 kHz), limited to 100/255–255/255. The light is internal (`internal: true`): it is not exposed as a Home Assistant switch or light entity, or through MQTT. Automatic local control remains active. The DHT22 reads temperature and humidity every 10 s; heat index is also recalculated every 10 s and RSSI every 60 s. MQTT publishing runs every 10 s. There are no fan controls.

</details>

## MQTT protocol

<details>
<summary><strong>Show topics and message formats</strong></summary>

All clocks use `!secret broker` on port `1883`. They publish every 10 seconds with QoS 2 and `retain: true`, when valid temperature and humidity readings are available.

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

| Fan | Topic |
|---|---|
| Bedroom | `casa/dormitorio/ventilador/set` |
| Living room | `casa/salon/ventilador/set` |

Brightness-up example:

```json
{"accion":"subir_luz"}
```

Clocks publish with QoS 2 and `retain: false`. The bridge accepts a text `accion` and rejects unknown actions.

| Clock internal index | MQTT action |
|---|---|
| 0 | `luz` |
| 1 | `ventilador` |
| 2 | `luz_amarilla` |
| 3 | `luz_blanca` |
| 4 | `luz_azul` |
| 5 | `bajar_luz` |
| 6 | `subir_luz` |
| 7 | `velocidad_1` |
| 8 | `velocidad_2` |
| 9 | `velocidad_3` |
| 10 | `velocidad_4` |
| 11 | `velocidad_5` |
| 12 | `invertir_giro` |
| 13 | `temporizador_2h` |
| 14 | `temporizador_4h` |
| 15 | `temporizador_8h` |

### CC1101 RF bridge

MQTT actions and native Home Assistant controls use the same table, with values supplied by `assets/codigos_ventiladores.yaml`. The selected code is converted to 32 bits and transmitted with a 390 µs pulse, `[23, 1]` sync, `[1, 2]` zero, `[2, 1]` one, inverted signal, and **2 transmissions separated by 10 ms**. `non_blocking: true` is enabled and the scripts use `queued` mode.

RF capture uses [`capturar_mando_esphome.yaml`](./capturar_mando_esphome.yaml), which shows captured codes in its logs.

The bridge ESP32 also acts as an active Bluetooth proxy for Home Assistant, with active scanning configured and up to three simultaneous connections (`max_connections: 3` and `connection_slots: 3`). It shares the device with MQTT control and RF transmission.

Light color and timer selectors execute commands through `set_action` when a selection is requested from Home Assistant or MQTT. Publishing their initial state does not transmit an RF command. Their state is optimistic: it represents the requested selection, without confirmation from the fan receiver.

### RF remote capture tool

The capture tool is not part of the MQTT flow. Connect to it through the ESPHome API and open the device logs while pressing remote buttons: `dump: [rc_switch, raw]` reports codes it can decode as RC-Switch and displays raw signal data when it cannot recognize the protocol. Record the protocol and code so they can be configured on the transmitting device; signals and codes depend on the remote.

</details>

## Customizing fan codes

[`assets/codigos_ventiladores.yaml`](./assets/codigos_ventiladores.yaml) contains **32 decimal 32-bit codes**: 16 bedroom actions and 16 living-room actions. These are examples for this installation's remotes, not universal codes.

1. Flash the capture tool and inspect its logs while pressing each remote button. Record the code and protocol.
2. Replace the quoted values in the code file, keeping all 32 variable names.
3. Validate, build, and flash only `cc1101_mqtt_central.yaml`. The clocks and native Home Assistant controls automatically use the bridge's same table.

```yaml
substitutions:
  rf_dormitorio_subir_luz: "1606450958"
  rf_dormitorio_bajar_luz: "4123033358"
```

This snippet is only an example; keep all actions in the complete file. The bridge includes it as follows:

```yaml
packages:
  codigos_rf: !include assets/codigos_ventiladores.yaml
```

**Copy `assets/codigos_ventiladores.yaml` into Device Builder too**, alongside the other assets. Pasting only the main YAML is not enough. For a remote with different protocol, code length, or timing, also adapt transmission and the bridge's 32-bit conversion.

## Home Assistant entities

Add each device through Home Assistant's **ESPHome integration**, using discovery or its IP and the `api_key_reloj` encryption key. The API exposes the entities below; MQTT carries inter-clock readings and clock-to-bridge commands. Actual `entity_id` values depend on names and your Home Assistant entity registry.

| Device | Declared entities |
|---|---|
| Office, bedroom, living room | 4 sensors per clock: temperature, humidity, heat index, Wi-Fi signal |
| Bathroom | 5 sensors: `temp_aseo`, `hum_aseo`, `Sensación Térmica`, `Luz ambiente relativa`, `Aseo senal wifi` |
| CC1101 bridge | 2 fans, 4 selectors, 8 buttons (14 controls) |
| CC1101 capture tool | No custom sensors, fans, or buttons; captured codes appear in the logs |

The bathroom backlight is internal and automatic, not a controllable entity. The bathroom and bridge YAMLs do not disable MQTT discovery; if you also use the MQTT integration, check for duplicate entities. The three touchscreen clocks explicitly set `discovery: false`.

### Bridge controls

These are the names declared in the YAML (kept in Spanish):

| Type | Name |
|---|---|
| `fan` | Ventilador Dormitorio |
| `fan` | Ventilador Salón |
| `select` | Ventilador Dormitorio - Color Luz |
| `select` | Ventilador Salón - Color Luz |
| `select` | Ventilador Dormitorio - Timer |
| `select` | Ventilador Salón - Timer |
| `button` | Ventilador Dormitorio - Luz General |
| `button` | Ventilador Dormitorio - Luz Subir |
| `button` | Ventilador Dormitorio - Luz Bajar |
| `button` | Ventilador Dormitorio - Invertir giro |
| `button` | Ventilador Salón - Luz General |
| `button` | Ventilador Salón - Luz Subir |
| `button` | Ventilador Salón - Luz Bajar |
| `button` | Ventilador Salón - Invertir giro |

Fans provide 5 speeds. Selectors offer `Amarilla`, `Blanca`, `Azul` and `2h`, `4h`, `8h` timers. The fan light, brightness up/down, and reverse are controlled through action buttons. The buttons named “Luz General” in the YAML and the table above operate each fan’s integrated light. **Turning a fan on or off sends the same toggle code**; there is no RF feedback of its physical state. Selectors are optimistic. Using the physical remote can leave Home Assistant's displayed state out of sync with the fan.

## Maintenance

<details>
<summary><strong>Show maintenance recommendations</strong></summary>

- When moving a clock to another network, update its IP, gateway, subnet mask, DNS, and broker; avoid duplicate static addresses.
- When changing a sensor or board, update the GPIOs in the relevant YAML and check the wiring.
- To replace icons, keep the PNG filenames referenced in the `image` section.
- Adjust fonts, orientation, and calibration under `lvgl`/`touchscreen`; for the bathroom clock, update the ADC sensor and `light` output.
- If the MQTT or fan-control protocol changes, update the corresponding publishers and consumers and validate the installation before deployment.

</details>
