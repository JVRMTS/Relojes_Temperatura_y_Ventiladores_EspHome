<div align="center">

<h1>Relojes de casa con ESPHome</h1>

<p><strong>Cuatro relojes conectados, un puente RF y una herramienta para capturar mandos.</strong></p>

<p>Los programas relacionados con Arduino que estaban anteriormente en este repositorio se han eliminado y sus funciones se han portado a ESPHome. Este repositorio contiene las configuraciones ESPHome de los cuatro relojes y dos dispositivos RF con CC1101: un puente MQTT–RF y una herramienta de captura.</p>

<p>Tres relojes cuentan con pantalla táctil ILI9488 y control MQTT de los ventiladores del salón y del dormitorio. El reloj del aseo usa una pantalla LCD de caracteres y publica sus mediciones. El puente CC1101 transmite comandos MQTT por RF de 433,92 MHz y expone controles en Home Assistant; el capturador muestra en el registro los códigos del mando.</p>

<p><a href="./README.en.md">English</a> | <strong>Español</strong></p>

</div>

<div align="center">

<table>
  <thead>
    <tr><th>Dispositivos</th><th>Integración</th><th>Zona horaria</th></tr>
  </thead>
  <tbody>
    <tr><td><strong>4 relojes + 2 dispositivos RF ESP32</strong></td><td><strong>Wi-Fi · MQTT · RF · OTA</strong></td><td><code>Europe/Madrid</code> (relojes)</td></tr>
  </tbody>
</table>

</div>

## Navegación

<div align="center">

<p>
  <a href="#inventario">Inventario</a> ·
  <a href="#hardware-y-conexiones">Hardware</a> ·
  <a href="#red-y-secretos">Red y secretos</a> ·
  <a href="#compilar-y-cargar">Instalación</a> ·
  <a href="#qué-muestra-y-hace-cada-reloj">Funciones</a> ·
  <a href="#protocolo-mqtt">MQTT</a> ·
  <a href="#personalizar-los-códigos-de-los-ventiladores">Códigos RF</a> ·
  <a href="#entidades-en-home-assistant">Home Assistant</a> ·
  <a href="#mantenimiento">Mantenimiento</a>
</p>

</div>

## Inventario

<table>
  <thead>
    <tr>
      <th>Habitación</th>
      <th>Configuración</th>
      <th>Nombre ESPHome</th>
      <th>IP</th>
      <th>Pantalla</th>
      <th>Sensor local</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><strong>Aseo</strong></td>
      <td><a href="./reloj_aseo_esphome.yaml">reloj_aseo_esphome.yaml</a></td>
      <td><code>reloj-aseo</code></td>
      <td><code>192.168.1.22</code></td>
      <td>LCD 20×4 · I²C</td>
      <td>DHT22 · GPIO27</td>
    </tr>
    <tr>
      <td><strong>Despacho</strong></td>
      <td><a href="./reloj_despacho_esphome.yaml">reloj_despacho_esphome.yaml</a></td>
      <td><code>reloj-despacho</code></td>
      <td><code>192.168.1.21</code></td>
      <td>ILI9488 táctil</td>
      <td>DHT22 · GPIO22</td>
    </tr>
    <tr>
      <td><strong>Dormitorio</strong></td>
      <td><a href="./reloj_dormitorio_esphome.yaml">reloj_dormitorio_esphome.yaml</a></td>
      <td><code>reloj-dormitorio</code></td>
      <td><code>192.168.1.23</code></td>
      <td>ILI9488 táctil</td>
      <td>DHT22 · GPIO22</td>
    </tr>
    <tr>
      <td><strong>Salón</strong></td>
      <td><a href="./reloj_salon_esphome.yaml">reloj_salon_esphome.yaml</a></td>
      <td><code>reloj-salon</code></td>
      <td><code>192.168.1.24</code></td>
      <td>ILI9488 táctil</td>
      <td>DHT22 · GPIO22</td>
    </tr>
  </tbody>
</table>

Los tres relojes táctiles comparten diseño, hardware y lógica de control, con ajustes propios de habitación, red y tema MQTT. El del aseo es distinto: no tiene pantalla táctil ni control de ventiladores.

### Puente RF CC1101

La configuración [cc1101_mqtt_central.yaml](./cc1101_mqtt_central.yaml) corresponde a un ESP32 independiente, `cc1101-mqtt-central`, con IP `192.168.1.18`. Recibe acciones MQTT en `casa/dormitorio/ventilador/set` y `casa/salon/ventilador/set`, busca el código en [`assets/codigos_ventiladores.yaml`](./assets/codigos_ventiladores.yaml) y lo transmite por RF. Expone además ventiladores, selectores y botones mediante la API ESPHome para controlarlos desde Home Assistant. El puente actual solo transmite: no recibe mandos, publica códigos recibidos ni genera eventos de recepción RF. La captura corresponde al dispositivo independiente descrito a continuación.

### Capturador de mandos RF

La configuración [capturar_mando_esphome.yaml](./capturar_mando_esphome.yaml) es una herramienta independiente para identificar los códigos de un mando compatible con RC-Switch o inspeccionar señales RF sin decodificar. Usa el nombre ESPHome `cc1101-captura-mando`; se conecta por Wi-Fi con dirección asignada por DHCP y expone los mensajes mediante la API de ESPHome. No transmite ni usa MQTT. Al pulsar un botón del mando, consulta el registro del dispositivo en ESPHome: los códigos RC-Switch aparecen con protocolo y valor decimal, y el decodificador `raw` permite inspeccionar señales no reconocidas.

### Recursos compartidos

La carpeta [`assets/`](./assets/) contiene los PNG de los relojes táctiles y el archivo de códigos RF del puente:

<table>
  <thead>
    <tr><th>Archivo</th><th>Uso</th></tr>
  </thead>
  <tbody>
    <tr><td><code>icono_temperatura.png</code></td><td>Temperatura</td></tr>
    <tr><td><code>icono_humedad.png</code></td><td>Humedad</td></tr>
    <tr><td><code>icono_sensacion.png</code></td><td>Sensación térmica</td></tr>
    <tr><td><code>logo_reloj.png</code></td><td>Logotipo del reloj</td></tr>
    <tr><td><a href="./assets/codigos_ventiladores.yaml"><code>codigos_ventiladores.yaml</code></a></td><td>32 códigos RF editables: 16 acciones por ventilador</td></tr>
  </tbody>
</table>

El reloj del aseo no utiliza estos recursos.

## Requisitos y dependencias

- ESPHome 2026.10.2 (versión probada).
- ESP32 compatible con `esp32dev`; framework ESP-IDF.
- Red Wi-Fi y broker MQTT accesible, con credenciales válidas.
- **Dispositivos RF:** módulo CC1101 compatible con 433,92 MHz.
- **Despacho, dormitorio y salón:** pantalla SPI ILI9488, panel XPT2046, sensor DHT22 y carpeta `assets/` junto a los YAML.
- **Aseo:** LCD 20×4 con expansor PCF8574 (`0x27`), DHT22, fotoresistencia y circuito de retroiluminación.

> [!NOTE]
> Los YAML táctiles descargan fuentes Roboto desde Google Fonts durante la compilación. La máquina de compilación debe tener acceso a Internet la primera vez.

## Hardware y conexiones

<details>
<summary><strong>Ver cableado y notas técnicas</strong></summary>

### Despacho, dormitorio y salón

<table>
  <thead>
    <tr><th>Función</th><th>Conexión</th></tr>
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

LVGL usa rotación de 90 grados y transformación `mirror_x`; el panel táctil usa `mirror_y` y los límites de calibración definidos en cada YAML. Si cambia el cableado o la orientación física, comprueba la rotación y recalibra el táctil.

> [!WARNING]
> GPIO15 y GPIO2 son pines de *strapping* del ESP32. Mantén el cableado previsto y evita cargas que alteren su nivel durante el arranque.

### Puente RF CC1101

<table>
  <thead>
    <tr><th>Función</th><th>Conexión</th></tr>
  </thead>
  <tbody>
    <tr><td>SPI CLK</td><td>GPIO18</td></tr>
    <tr><td>SPI MISO</td><td>GPIO19</td></tr>
    <tr><td>SPI MOSI</td><td>GPIO23</td></tr>
    <tr><td>CC1101 CS</td><td>GPIO5</td></tr>
    <tr><td>Entrada de datos TX del CC1101 (remote_transmitter)</td><td>GPIO2</td></tr>
  </tbody>
</table>

El puente transmite por GPIO2, con modulación ASK/OOK, frecuencia 433,92 MHz, potencia de salida 10, symbol rate 5000 y filtro de 812 kHz; `packet_mode`, CRC y whitening están desactivados. `on_transmit` activa TX y `on_complete` deja el CC1101 en reposo. No hay `remote_receiver` ni pin RX compartido en este YAML.

El capturador usa el mismo bus SPI y CS en GPIO5; conecta la salida de datos del receptor RF a GPIO4. Está configurado a 433,92 MHz con un ancho de banda de filtro de 812 kHz y no transmite.

### Aseo

<table>
  <thead>
    <tr><th>Función</th><th>Conexión</th></tr>
  </thead>
  <tbody>
    <tr><td>LCD 20×4 con PCF8574</td><td>I²C, dirección <code>0x27</code></td></tr>
    <tr><td>I²C SDA</td><td>GPIO21</td></tr>
    <tr><td>I²C SCL</td><td>GPIO22</td></tr>
    <tr><td>DHT22</td><td>GPIO27</td></tr>
    <tr><td>Fotoresistencia (ADC)</td><td>GPIO32</td></tr>
    <tr><td>Retroiluminación</td><td>GPIO18, salida LEDC</td></tr>
  </tbody>
</table>

La fotoresistencia regula automáticamente la retroiluminación. Este reloj no usa el bus SPI de las otras tres unidades.

### Placa y revisión mínima del chip

Las seis configuraciones declaran `board: esp32dev`, pero la revisión mínima del chip se configura por dispositivo. `minimum_chip_revision` es un requisito de la revisión del silicio ESP32, no el nombre o modelo de la placa física. Configura este valor según la revisión real del chip montado en cada dispositivo: comprueba qué revisión tienes y no copies sin verificar el valor de otra unidad.

<table>
  <thead>
    <tr><th>Dispositivo</th><th>Placa declarada en YAML</th><th><code>minimum_chip_revision</code></th><th>Estado</th></tr>
  </thead>
  <tbody>
    <tr><td>Aseo</td><td><code>esp32dev</code></td><td><code>3.0</code></td><td>Comentada; no se aplica</td></tr>
    <tr><td>Despacho</td><td><code>esp32dev</code></td><td><code>3.1</code></td><td>Activa</td></tr>
    <tr><td>Dormitorio</td><td><code>esp32dev</code></td><td><code>3.0</code></td><td>Comentada; no se aplica</td></tr>
    <tr><td>Salón</td><td><code>esp32dev</code></td><td><code>3.0</code></td><td>Activa</td></tr>
    <tr><td>Puente CC1101</td><td><code>esp32dev</code></td><td><code>3.1</code></td><td>Activa</td></tr>
    <tr><td>Capturador CC1101</td><td><code>esp32dev</code></td><td><code>3.0</code></td><td>Activa</td></tr>
  </tbody>
</table>

No establezcas un mínimo superior a la revisión real del chip. Si cambias una placa, vuelve a comprobar su revisión antes de reutilizar la configuración.

> [!TIP]
> **Primera instalación en una placa cuya revisión desconoces:** lo recomendable es comentar la línea `minimum_chip_revision` con `#` antes de compilar. Compila e instala el firmware y abre los registros del dispositivo en **ESPHome Device Builder**. Al arrancar, el registro muestra la revisión del ESP32, por ejemplo `ESP32 r3.0`. Después, descomenta la línea y escribe la revisión de tu unidad (`minimum_chip_revision: "3.0"` en este ejemplo); vuelve a compilar e instalar para aplicar el ajuste. La compilación por sí sola no detecta la revisión de una placa que aún no se ha consultado.

</details>

## Red y secretos

<details>
<summary><strong>Ver configuración de red y seguridad</strong></summary>

Las cuatro configuraciones de reloj y el puente MQTT–RF usan IP estática dentro de `192.168.1.0/24` y gateway `192.168.1.1`; el capturador obtiene su IP por DHCP. Los cuatro relojes declaran los DNS `192.168.1.201` y `192.168.1.204`. Los relojes y el puente MQTT–RF usan el broker definido por `!secret broker`, puerto `1883`; el capturador no se conecta a MQTT. Comprueba las direcciones y adáptalas a tu red antes de cargar el firmware.

Crea `secrets.yaml` en el directorio de configuración de ESPHome. No guardes credenciales reales en el repositorio:

```yaml
wifi_ssid: "nombre-de-la-red"
wifi_password: "clave-wifi"
mqtt_username: "usuario-mqtt"
mqtt_password: "clave-mqtt"
broker: "192.168.1.203" # Ejemplo: adapta esta IP a tu broker
api_key_reloj: "clave-api-base64-de-32-bytes"
```

OTA usa `encryption:` y la clave de cifrado de la API, por lo que no necesita una contraseña OTA adicional. La contraseña del punto de acceso Wi-Fi de respaldo es independiente de OTA.

Los cuatro relojes sincronizan la hora por SNTP con la zona horaria `Europe/Madrid`; los dos dispositivos RF no disponen de reloj ni sincronización horaria.

</details>

## Compilar y cargar

<details>
<summary><strong>Ver instrucciones de instalación</strong></summary>

Guarda los seis YAML principales y la carpeta `assets/` en el directorio de configuración de ESPHome, conservando las rutas relativas. Los táctiles necesitan los PNG y el puente necesita `assets/codigos_ventiladores.yaml`; este último es un paquete incluido, no un dispositivo que se compile por separado. En Device Builder, pega cada YAML completo y copia también los recursos. Compila cada dispositivo individualmente:

```shell
esphome compile reloj_aseo_esphome.yaml
esphome compile reloj_despacho_esphome.yaml
esphome compile reloj_dormitorio_esphome.yaml
esphome compile reloj_salon_esphome.yaml
esphome compile cc1101_mqtt_central.yaml
esphome compile capturar_mando_esphome.yaml
```

- **OTA:** el dispositivo debe estar conectado a la red y el servicio OTA habilitado.
- **Primer flasheo:** puede requerir una conexión USB y seleccionar el puerto adecuado.
- **Cambios de red:** revisa los valores de red declarados en el YAML correspondiente antes de compilar.

Las configuraciones táctiles usan ESP-IDF y asignan SRAM1 como IRAM para disponer de memoria para la pantalla. El búfer LVGL está limitado al 10 % para reducir la presión sobre la RAM interna de placas sin PSRAM. Consulta la tabla de placa y revisión del chip en la sección de hardware antes de cambiar `esp32.framework.advanced` o trasladar un firmware a otra placa.

</details>

## Qué muestra y hace cada reloj

<details>
<summary><strong>Ver funciones de cada modelo</strong></summary>

### Despacho, dormitorio y salón

La pantalla principal muestra hora, fecha, temperatura, humedad y sensación térmica. Permite consultar los valores MQTT de las habitaciones y abrir el control del ventilador del dormitorio o del salón. El DHT22 local se muestrea cada 10 segundos.

El control de ventilador tiene cuatro filas:

1. Luz general, volver y ventilador.
2. Bajar brillo, luces amarilla/blanca/azul y subir brillo.
3. Velocidades 1 a 5.
4. Invertir giro y temporizadores de 2, 4 y 8 horas.

La interfaz no es desplazable. La habitación local muestra la hora en la cabecera; al consultar otra habitación, la cabecera muestra su nombre y mantiene la fecha debajo. Temperatura y sensación térmica se muestran como `23.4°`, sin `C`; humedad como `48.0%`. Cada icono y su valor forman un conjunto centrado horizontal y verticalmente, separado por 4 píxeles. Los datos de pantalla se actualizan cada segundo.

Los botones de control RF usan un filtro antirrebote: se acepta una orden por contacto y se rearma después de **200 ms sin tocar la pantalla**. Un contacto nuevo durante esa espera cancela el rearme. El registro `tactil` distingue órdenes aceptadas e ignoradas. Este filtro se aplica a las 16 acciones de ventilador y luz, no a selección de habitación, navegación o «Atrás». «Atrás» solo cambia de página.

Los relojes envían nombres de acción, no códigos numéricos. Todos los códigos se personalizan en `assets/codigos_ventiladores.yaml`; no es necesario editar las tablas de los relojes.

### Aseo

La LCD muestra hora, fecha, temperatura, humedad y sensación térmica. Temperatura y sensación térmica incluyen un símbolo de grado propio del LCD, sin `C`. La fotoresistencia de GPIO32 se lee cada segundo y regula la retroiluminación por PWM en GPIO18 (5 kHz), limitada entre 100/255 y 255/255. La luz se mantiene interna (`internal: true`): no aparece como interruptor o entidad de luz en Home Assistant ni se expone por MQTT. Cualquier ajuste automático continúa funcionando localmente. El DHT22 lee temperatura y humedad cada 10 s; la sensación térmica también se recalcula cada 10 s y el RSSI cada 60 s. La publicación MQTT se realiza cada 10 s. No tiene controles de ventiladores.

</details>

## Protocolo MQTT

<details>
<summary><strong>Ver temas y formatos de mensajes</strong></summary>

Todos los relojes usan el broker definido por `!secret broker`, puerto `1883`. Publican cada 10 segundos con QoS 2 y `retain: true`, cuando hay lecturas válidas de temperatura y humedad.

### Temas de mediciones

<table>
  <thead>
    <tr><th>Habitación</th><th>Tema</th></tr>
  </thead>
  <tbody>
    <tr><td>Aseo</td><td><code>casa/aseo/variables</code></td></tr>
    <tr><td>Despacho</td><td><code>casa/despacho/variables</code></td></tr>
    <tr><td>Dormitorio</td><td><code>casa/dormitorio/variables</code></td></tr>
    <tr><td>Salón</td><td><code>casa/salon/variables</code></td></tr>
  </tbody>
</table>

Ejemplo de payload JSON:

```json
{"temperatura":22.5,"humedad":48.0,"sen_termica":22.1,"WiFi_Signal":-55}
```

Temperatura y sensación térmica se expresan en °C; humedad, en %; y `WiFi_Signal`, en dBm. El reloj del despacho escucha salón, aseo y dormitorio. Los relojes del dormitorio y del salón escuchan las cuatro habitaciones. Antes de recibir datos, la interfaz puede mostrar `--.-`; las claves ausentes no sustituyen el último valor recibido para ese dato.

### Temas de control de ventiladores

| Ventilador | Tema |
|---|---|
| Dormitorio | `casa/dormitorio/ventilador/set` |
| Salón | `casa/salon/ventilador/set` |

Ejemplo para subir el brillo:

```json
{"accion":"subir_luz"}
```

Los relojes publican con QoS 2 y `retain: false`. El puente acepta `accion` como texto y rechaza las acciones desconocidas. El formato antiguo `opcion` y los temas sin `/set` se han eliminado.

| Índice interno del reloj | Acción MQTT |
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

### Puente RF CC1101

Las acciones MQTT y los controles nativos de Home Assistant utilizan la misma tabla, cuyos valores proceden de `assets/codigos_ventiladores.yaml`. El código elegido se convierte en 32 bits y se transmite con pulso de 390 µs, sincronización `[23, 1]`, cero `[1, 2]`, uno `[2, 1]`, señal invertida y **2 transmisiones con 10 ms entre ellas**. Se conserva este valor porque una sola transmisión no funcionó correctamente en las pruebas. `non_blocking: true` está activo y los scripts están configurados en modo `queued`.

La captura RF se realiza con `capturar_mando_esphome.yaml`. El puente no publica `casa/ventilador/mando`, no tiene sensor «Último código mando ventilador» ni emite `esphome.rf_fan_remote_received`.

### Capturador de mandos RF

El capturador no forma parte del flujo MQTT. Conéctalo en ESPHome usando la API y abre el registro mientras pulsas botones del mando: `dump: [rc_switch, raw]` informa los códigos que pueda decodificar como RC-Switch y muestra la señal sin procesar cuando no reconozca el protocolo. Anota el protocolo y código recibidos para configurar después el dispositivo que transmite; las señales y códigos dependen del mando.

</details>



## Personalizar los códigos de los ventiladores

El archivo [`assets/codigos_ventiladores.yaml`](./assets/codigos_ventiladores.yaml) contiene **32 códigos decimales de 32 bits**, separados en 16 acciones para dormitorio y 16 para salón. Son ejemplos de los mandos de esta instalación, no códigos universales.

1. Instala el capturador y consulta sus registros al pulsar cada botón del mando. Anota código y protocolo.
2. Edita los valores entre comillas en el archivo de códigos, conservando las 32 variables y sus nombres.
3. Valida, compila e instala únicamente `cc1101_mqtt_central.yaml`. Los relojes y los controles de Home Assistant usan automáticamente la misma tabla del puente.

```yaml
substitutions:
  rf_dormitorio_subir_luz: "1606450958"
  rf_dormitorio_bajar_luz: "4123033358"
```

Este fragmento es un ejemplo; el archivo completo debe mantener todas las acciones. Se incluye desde el puente así:

```yaml
packages:
  codigos_rf: !include assets/codigos_ventiladores.yaml
```

**En Device Builder debes copiar también `assets/codigos_ventiladores.yaml`** junto al resto de recursos. Pegar solo el YAML principal no basta. Si un mando utiliza otro protocolo, longitud o tiempos, adapta además la transmisión y la conversión de 32 bits en el puente.

## Entidades en Home Assistant

Añade cada dispositivo mediante la **integración ESPHome** de Home Assistant, por descubrimiento o por su IP, usando la clave `api_key_reloj`. La API expone las entidades siguientes; MQTT sigue transportando las mediciones entre relojes y las acciones al puente. Los identificadores `entity_id` finales dependen de los nombres y del registro de tu Home Assistant.

| Dispositivo | Entidades declaradas |
|---|---|
| Despacho, dormitorio y salón | 4 sensores por reloj: temperatura, humedad, sensación térmica y señal Wi-Fi |
| Aseo | 5 sensores: `temp_aseo`, `hum_aseo`, `Sensación Térmica`, `LDR ADC raw`, `Aseo senal wifi` |
| Puente CC1101 | 2 ventiladores, 4 selectores y 8 botones (14 controles) |
| Capturador CC1101 | Sin sensores, ventiladores ni botones propios; códigos visibles en los registros |

La retroiluminación del aseo es interna y automática, no una entidad controlable. Si ya aparecía en Home Assistant, puede quedar una entrada antigua no disponible tras actualizar; elimínala de su registro si persiste. El aseo y el puente no desactivan el descubrimiento MQTT en sus YAML; si también usas la integración MQTT, revisa posibles entidades duplicadas. Los tres táctiles sí declaran `discovery: false`.

### Controles del puente

Los nombres declarados en el YAML son:

| Tipo | Nombre |
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

Los ventiladores tienen 5 velocidades; los selectores ofrecen `Amarilla`, `Blanca`, `Azul` y temporizadores `2h`, `4h`, `8h`. Luz general, subir/bajar brillo e invertir giro son botones de acción. **Encender y apagar envían el mismo código de alternancia**; no hay confirmación RF del estado físico. Los selectores son optimistas. Una pulsación del mando físico puede dejar los estados mostrados en Home Assistant sin correspondencia con el ventilador real.

## Mantenimiento

<details>
<summary><strong>Ver recomendaciones de mantenimiento</strong></summary>

- Al cambiar un reloj de red, adapta IP, gateway, máscara, DNS y broker; evita duplicar direcciones estáticas.
- Al cambiar sensor o placa, actualiza los GPIO del YAML correspondiente y comprueba el cableado.
- Para reemplazar iconos, conserva los nombres PNG referenciados en `image`.
- Ajusta fuentes, orientación y calibración en `lvgl`/`touchscreen`; para el aseo, modifica el sensor ADC y la salida `light`.
- Si cambia el protocolo MQTT o el de los ventiladores, actualiza productores y consumidores y valida la instalación antes del despliegue.

</details>

## Archivos para GitHub

Incluye los seis YAML de dispositivo, `assets/` completo (PNG y códigos), ambos README y `.gitignore`. El `.gitignore` excluye `.esphome/`, `.venv/`, `__pycache__/`, `*.pyc` y `secrets.yaml`. No publiques credenciales reales; un archivo de ejemplo debe contener valores ficticios. Estas exclusiones afectan a Git, no a OneDrive.
