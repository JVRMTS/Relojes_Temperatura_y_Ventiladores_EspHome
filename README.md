<div align="center">

<h1>Relojes de casa con ESPHome</h1>

<p><strong>Cuatro relojes conectados, un puente RF y una herramienta para capturar mandos.</strong></p>

<p>Los programas relacionados con Arduino que estaban anteriormente en este repositorio se han eliminado y sus funciones se han portado a ESPHome. Este repositorio contiene las configuraciones ESPHome de los cuatro relojes y dos dispositivos RF con CC1101: un puente MQTT–RF y una herramienta de captura.</p>

<p>Tres relojes cuentan con pantalla táctil ILI9488 y control MQTT de los ventiladores del salón y del dormitorio. El reloj del aseo usa una pantalla LCD de caracteres y publica sus mediciones. El puente CC1101 transmite comandos MQTT por RF de 433,92 MHz y publica los códigos recibidos; el capturador muestra en el registro los códigos del mando.</p>

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

La configuración [cc1101_mqtt_central.yaml](./cc1101_mqtt_central.yaml) corresponde a un ESP32 independiente del inventario de relojes. Actúa como pasarela entre MQTT y los mandos RF de los ventiladores: escucha los comandos de `casa/+/ventilador`, los transmite por RF y publica los códigos recibidos. También expone los ventiladores y botones de control mediante la API nativa de ESPHome; para verlos en Home Assistant, añade el dispositivo con la integración ESPHome. Usa el nombre ESPHome `cc1101-mqtt-central`, IP estática `192.168.1.18` y no tiene pantalla ni sensor ambiental.

### Capturador de mandos RF

La configuración [capturar_mando_esphome.yaml](./capturar_mando_esphome.yaml) es una herramienta independiente para identificar los códigos de un mando compatible con RC-Switch o inspeccionar señales RF sin decodificar. Usa el nombre ESPHome `cc1101-captura-mando`; se conecta por Wi-Fi con dirección asignada por DHCP y expone los mensajes mediante la API de ESPHome. No transmite ni usa MQTT. Al pulsar un botón del mando, consulta el registro del dispositivo en ESPHome: los códigos RC-Switch aparecen con protocolo y valor decimal, y el decodificador `raw` permite inspeccionar señales no reconocidas.

### Recursos gráficos

La carpeta [`assets/`](./assets/) contiene los iconos PNG usados por los relojes táctiles:

<table>
  <thead>
    <tr><th>Archivo</th><th>Uso</th></tr>
  </thead>
  <tbody>
    <tr><td><code>icono_temperatura.png</code></td><td>Temperatura</td></tr>
    <tr><td><code>icono_humedad.png</code></td><td>Humedad</td></tr>
    <tr><td><code>icono_sensacion.png</code></td><td>Sensación térmica</td></tr>
    <tr><td><code>logo_reloj.png</code></td><td>Logotipo del reloj</td></tr>
  </tbody>
</table>

El reloj del aseo no utiliza estos recursos.

## Requisitos y dependencias

- ESPHome 2026.9.1 o posterior.
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
    <tr><td>CC1101 GDO0 / transmisión y recepción</td><td>GPIO4</td></tr>
  </tbody>
</table>

El módulo está configurado para 433,92 MHz, potencia de salida 10 y ancho de banda de filtro de 812 kHz. GPIO4 se comparte entre transmisor y receptor; conserva `allow_other_uses` en ambas configuraciones. Comprueba alimentación, niveles lógicos y conexiones del módulo antes de encenderlo.

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
    <tr><td>Aseo</td><td><code>esp32dev</code></td><td>No especificada</td><td>No hay requisito explícito en el YAML</td></tr>
    <tr><td>Despacho</td><td><code>esp32dev</code></td><td><code>3.1</code></td><td>Activa</td></tr>
    <tr><td>Dormitorio</td><td><code>esp32dev</code></td><td><code>3.1</code></td><td>Comentada; no se aplica</td></tr>
    <tr><td>Salón</td><td><code>esp32dev</code></td><td><code>3.0</code></td><td>Activa</td></tr>
    <tr><td>Puente CC1101</td><td><code>esp32dev</code></td><td><code>3.0</code></td><td>Activa</td></tr>
    <tr><td>Capturador CC1101</td><td><code>esp32dev</code></td><td><code>3.0</code></td><td>Activa</td></tr>
  </tbody>
</table>

Antes de compilar, ajusta `minimum_chip_revision` en `esp32.framework.advanced` de cada YAML según la revisión del chip de esa unidad. No establezcas un mínimo superior al que soporta el chip; si no conoces la revisión, identifícala consultando el marcado o la documentación del módulo/chip antes de fijar el valor. Si se cambia una placa o se activa una revisión mínima comentada, vuelve a verificarla y actualiza únicamente la configuración del dispositivo correspondiente.

</details>

## Red y secretos

<details>
<summary><strong>Ver configuración de red y seguridad</strong></summary>

Las cuatro configuraciones de reloj y el puente MQTT–RF usan IP estática dentro de `192.168.1.0/24` y gateway `192.168.1.1`; el capturador obtiene su IP por DHCP. Los cuatro relojes declaran los DNS `192.168.1.201` y `192.168.1.204`. Los relojes y el puente MQTT–RF usan el broker `192.168.1.203:1883`; el capturador no se conecta a MQTT. Comprueba las direcciones y adáptalas a tu red antes de cargar el firmware.

Crea `secrets.yaml` en el directorio de configuración de ESPHome. No guardes credenciales reales en el repositorio:

```yaml
wifi_ssid: "nombre-de-la-red"
wifi_password: "clave-wifi"
mqtt_username: "usuario-mqtt"
mqtt_password: "clave-mqtt"
api_key_reloj: "clave-api-base64-de-32-bytes"
```

OTA usa `encryption:` y la clave de cifrado de la API, por lo que no necesita una contraseña OTA adicional. La contraseña del punto de acceso Wi-Fi de respaldo es independiente de OTA.

Los cuatro relojes sincronizan la hora por SNTP con la zona horaria `Europe/Madrid`; los dos dispositivos RF no disponen de reloj ni sincronización horaria.

</details>

## Compilar y cargar

<details>
<summary><strong>Ver instrucciones de instalación</strong></summary>

Guarda los YAML en el directorio de configuración de ESPHome y coloca `assets/` junto a las tres configuraciones táctiles, conservando las rutas relativas. Compila cada configuración individualmente:

```shell
esphome compile reloj_aseo_esphome.yaml
esphome compile reloj_despacho_esphome.yaml
esphome compile reloj_dormitorio_esphome.yaml
esphome compile reloj_salon_esphome.yaml
esphome compile cc1101_mqtt_esphome.yaml
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

La interfaz no es desplazable. El ventilador seleccionado determina el tema MQTT de destino. Los códigos publicados son específicos de cada ventilador: los configurados aquí son los que obtuve para mis propios ventiladores y no sirven para otros. Cada usuario debe obtener los códigos de los ventiladores que tenga y adaptar la tabla de `enviar_comando_ventilador` a su receptor.

### Aseo

La LCD muestra hora, fecha, temperatura, humedad y sensación térmica. La fotoresistencia de GPIO32 regula la retroiluminación a través de GPIO18. El dispositivo publica sus lecturas por MQTT y no incluye controles gráficos de ventiladores.

</details>

## Protocolo MQTT

<details>
<summary><strong>Ver temas y formatos de mensajes</strong></summary>

Todos los relojes usan el broker `192.168.1.203:1883`. Publican cada dos segundos con QoS 2 y `retain: true`, cuando hay lecturas válidas de temperatura y humedad.

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

Los relojes táctiles envían `{"opcion": <código>}` a `casa/dormitorio/ventilador` o `casa/salon/ventilador`, según el ventilador seleccionado.

<table>
  <thead>
    <tr><th>Índice interno</th><th>Acción</th></tr>
  </thead>
  <tbody>
    <tr><td>0</td><td>Luz general</td></tr>
    <tr><td>1</td><td>Ventilador</td></tr>
    <tr><td>2–4</td><td>Luz amarilla, blanca y azul</td></tr>
    <tr><td>5–6</td><td>Bajar y subir brillo</td></tr>
    <tr><td>7–11</td><td>Velocidades 1–5</td></tr>
    <tr><td>12</td><td>Invertir giro</td></tr>
    <tr><td>13–15</td><td>Temporizadores de 2, 4 y 8 horas</td></tr>
  </tbody>
</table>

Los comandos se publican con QoS 2 y `retain: false` para evitar que el broker retransmita una orden antigua al reconectar. El valor `{"opcion":0}` se ignora como marcador de reinicio y no se transmite por RF. Los códigos finales difieren entre dormitorio y salón y están definidos en cada YAML. Son códigos propios de mis ventiladores, no universales: cada usuario debe obtener los códigos de cada uno de sus ventiladores y configurar los suyos antes de usar el control.

### Puente RF CC1101

El puente se suscribe a `casa/+/ventilador` y acepta un objeto JSON con `opcion` como entero sin signo de 32 bits, por ejemplo `{"opcion": 123456}`. Convierte ese entero en una secuencia binaria de 32 bits y la transmite con el protocolo RC-Switch configurado en el YAML (pulso de 390 µs, sincronización `[23, 1]`, códigos `zero: [1, 2]` y `one: [2, 1]`, señal invertida y dos repeticiones). Los parámetros RF y el código enviado deben coincidir con el receptor y el mando que se quieran controlar; no son universales.

Al recibir un código RC-Switch, publica en `casa/ventilador/mando` un JSON con el protocolo y el código, por ejemplo `{"protocolo":1,"codigo":"123456"}`. Esta publicación usa QoS 0 y no es retenida. También actualiza el sensor de texto ESPHome **Último código mando ventilador** y emite el evento de Home Assistant `esphome.rf_fan_remote_received` con los campos `protocol` y `code`.

### Capturador de mandos RF

El capturador no forma parte del flujo MQTT. Conéctalo en ESPHome usando la API y abre el registro mientras pulsas botones del mando: `dump: [rc_switch, raw]` informa los códigos que pueda decodificar como RC-Switch y muestra la señal sin procesar cuando no reconozca el protocolo. Anota el protocolo y código recibidos para configurar después el dispositivo que transmite; las señales y códigos dependen del mando.

</details>

## Mantenimiento

<details>
<summary><strong>Ver recomendaciones de mantenimiento</strong></summary>

- Al cambiar un reloj de red, adapta IP, gateway, máscara, DNS y broker; evita duplicar direcciones estáticas.
- Al cambiar sensor o placa, actualiza los GPIO del YAML correspondiente y comprueba el cableado.
- Para reemplazar iconos, conserva los nombres PNG referenciados en `image`.
- Ajusta fuentes, orientación y calibración en `lvgl`/`touchscreen`; para el aseo, modifica el sensor ADC y la salida `light`.
- Si cambia el protocolo MQTT o el de los ventiladores, actualiza productores y consumidores y valida la instalación antes del despliegue.

</details>
