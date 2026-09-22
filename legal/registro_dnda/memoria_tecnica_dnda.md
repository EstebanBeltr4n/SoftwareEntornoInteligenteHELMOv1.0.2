# Anexo técnico HELMO

## Soporte documental para registro de derechos de autor de software (DNDA)

Este anexo combina los tres elementos que suele solicitar el trámite de registro de software ante la Dirección Nacional de Derecho de Autor (DNDA): (1) una descripción funcional detallada del programa, (2) un extracto representativo del código fuente y (3) un manual de usuario / guía de instalación. No incluye el repositorio íntegro: el código fuente completo permanece en el repositorio privado de control de versiones, disponible como respaldo si la entidad lo solicita.

La información declarada en este documento (nombre, versión, lenguajes) coincide con el contenido real del repositorio en la fecha indicada, en línea con el criterio de la DNDA de que "la información que se declara debe poder respaldarse en los archivos que se suben".

---

## 1. Identificación de la obra

| Campo | Valor |
|---|---|
| Nombre de la obra | HELMO |
| Descripción corta | Sistema de monitoreo inteligente de calidad de agua |
| Versión declarada | 1.0.0 |
| Fecha de la versión | 2026-09-22 |
| Tipo de obra | Software (firmware embebido + backend + integración de red + documentación) |
| Lenguajes de programación | C++ (Arduino/ESP32), JavaScript (Node.js) |
| Autores | Esteban Eduardo Escárraga Túquerres, Johan Felipe Ordoñez Jenoy, Dayerli Katerine Tamayo Solarte |
| Titularidad | Los autores, según `LICENSE` y `legal/titularidad.md` |
| Repositorio de referencia | Control de versiones Git, rama de trabajo `EstebanEscarraga`, historial completo de commits como evidencia de desarrollo incremental |

---

## 2. Descripción funcional detallada

### 2.1 Problema que resuelve

HELMO responde a la necesidad de vigilancia continua de variables de calidad y operación del agua (pH, turbidez, nivel) en contextos —como una planta de tratamiento de agua potable (PTAP)— donde el monitoreo manual es lento, discontinuo y costoso. El sistema automatiza la captura periódica de estas variables, reduce la dependencia del muestreo manual, deja trazabilidad histórica de los datos y permite generar alertas tempranas ante condiciones anómalas.

### 2.2 Cómo funciona (flujo funcional)

```text
Sensor (pH / turbidez / nivel)
   -> Nodo de campo (firmware C++, Heltec WiFi LoRa 32 V3)
   -> Empaquetado en payload binario compacto
   -> Transmisión LoRaWAN (OTAA, clase A)
   -> Gateway LoRa -> The Things Stack (TTN)
   -> Decodificador uplink (JavaScript) reconstruye variables desde bytes
   -> Backend (Node.js/Express o puente MQTT) recibe la telemetría decodificada
   -> Persistencia en MongoDB (local y, opcionalmente, Atlas en la nube)
   -> Visualización en Grafana, expuesto temporalmente vía ngrok para demostración
```

Cada nodo energiza su etapa de sensado mediante un MOSFET solo durante la adquisición (para ahorrar energía), toma varias muestras del sensor, las filtra (promedio, mediana u ordenamiento con recorte de extremos según el nodo) y construye un payload binario de 5 o 6 bytes según el tipo de nodo. El payload viaja por LoRaWAN hasta TTN, donde un decodificador uplink en JavaScript lo traduce a variables legibles (`nivel_cm`, `turbidez_ntu`, `ph`, metadatos de radio) antes de que el backend lo almacene y Grafana lo visualice.

### 2.3 Módulos principales

| Módulo | Ruta en el repositorio | Lenguaje | Función |
|---|---|---|---|
| Nodo concentrador multisensor | `firmware/nodo_central/src/nodo_concentrador_multisensor.ino` | C++ (Arduino) | Mide nivel, turbidez y pH en un solo nodo; empaqueta 6 bytes y transmite por LoRaWAN. |
| Nodo dedicado de nivel | `firmware/nodo_nivel/src/nodo_nivel_tanque_ptap.ino` | C++ (Arduino) | Mide nivel por ultrasonido, clasifica el estado del tanque y detecta visibilidad del filtro de arena. |
| Nodo dedicado de turbidez | `firmware/nodo_turbidez/src/nodo_turbidez_ptap.ino` | C++ (Arduino) | Mide turbidez por sensor analógico y clasifica el estado del agua. |
| Nodo dedicado de pH | `firmware/nodo_ph/src/nodo_ph_casa_01.ino` | C++ (Arduino) | Mide pH mediante calibración lineal de dos puntos. |
| Decodificador de payload | `integrations/ttn/uplink_decoder.js` | JavaScript | Traduce los bytes recibidos por TTN a variables estructuradas, según el tipo de nodo emisor. |
| API de consulta | `backend/api/api-mongo.js` | JavaScript (Node.js/Express) | Recibe webhooks de TTN, almacena en MongoDB y expone consulta autenticada de lecturas recientes. |
| Puente MQTT-MongoDB | `backend/helmo-bridge/index.js` | JavaScript (Node.js) | Se suscribe al broker MQTT de TTN y persiste la telemetría en MongoDB local y Atlas, con reintento automático. |
| Documentación de despliegue | `backend/deploy/` | Markdown | Describe la visualización en Grafana y su exposición temporal vía ngrok. |

### 2.4 Tecnologías empleadas

- **Hardware de nodo:** Heltec WiFi LoRa 32 V3 (ESP32-S3), sensores analógicos de pH y turbidez, sensor ultrasónico HC-SR04, MOSFET de control de energía, pantalla OLED.
- **Comunicación:** LoRaWAN (activación OTAA, clase A), integración con The Things Stack (TTN) mediante MQTT y Custom JavaScript Formatter.
- **Backend:** Node.js, Express, MongoDB (driver oficial `mongodb`), MQTT.js, autenticación HTTP Basic.
- **Persistencia y visualización:** MongoDB local y MongoDB Atlas (réplica en la nube), Grafana como dashboard, ngrok para exposición temporal durante demostraciones.

---

## 3. Extracto representativo del código fuente

Los siguientes fragmentos son representativos y no corresponden al repositorio completo. Se seleccionaron por ilustrar, respectivamente, la lógica de adquisición embebida, la lógica de decodificación del protocolo y la lógica de persistencia del backend — las tres capas funcionales centrales del sistema.

### 3.1 Firmware embebido — adquisición y transmisión (C++ / Arduino)

Archivo de referencia: `firmware/nodo_central/src/nodo_concentrador_multisensor.ino`

```cpp
/* Preparación de telemetría — payload de 6 bytes:
 * Byte 0-1 -> nivel x10 en cm (uint16, big-endian)
 * Byte 2-3 -> turbidez x10 en NTU (uint16, big-endian)
 * Byte 4-5 -> pH x100 (uint16, big-endian)
 */
static void prepareTxFrame(uint8_t port) {
  (void)port;

  energizarSensores(true);

  float nivelCm = leerNivel();
  float turbidezNtu = leerTurbidez();
  float phValor = leerPH();

  energizarSensores(false);

  uint16_t nivelInt = (uint16_t)(nivelCm * 10.0f);
  uint16_t turbInt  = (uint16_t)(turbidezNtu * 10.0f);
  uint16_t phInt    = (uint16_t)(phValor * 100.0f);

  appDataSize = 6;
  appData[0] = (uint8_t)(nivelInt >> 8);
  appData[1] = (uint8_t)(nivelInt & 0xFF);
  appData[2] = (uint8_t)(turbInt >> 8);
  appData[3] = (uint8_t)(turbInt & 0xFF);
  appData[4] = (uint8_t)(phInt >> 8);
  appData[5] = (uint8_t)(phInt & 0xFF);

  mostrarEstadoOLED(nivelCm, turbidezNtu, phValor);
}

/* Máquina de estados LoRaWAN */
void loop() {
  switch (deviceState) {
    case DEVICE_STATE_INIT:
      LoRaWAN.init(loraWanClass, loraWanRegion);
      break;
    case DEVICE_STATE_JOIN:
      LoRaWAN.join();
      break;
    case DEVICE_STATE_SEND:
      prepareTxFrame(appPort);
      LoRaWAN.send();
      deviceState = DEVICE_STATE_CYCLE;
      break;
    case DEVICE_STATE_CYCLE:
      txDutyCycleTime = appTxDutyCycle + randr(-APP_TX_DUTYCYCLE_RND, APP_TX_DUTYCYCLE_RND);
      LoRaWAN.cycle(txDutyCycleTime);
      deviceState = DEVICE_STATE_SLEEP;
      break;
    case DEVICE_STATE_SLEEP:
      LoRaWAN.sleep(loraWanClass);
      break;
    default:
      deviceState = DEVICE_STATE_INIT;
      break;
  }
}
```

### 3.2 Decodificador de payload LoRaWAN (JavaScript — The Things Stack)

Archivo de referencia: `integrations/ttn/uplink_decoder.js` (módulo completo, por ser una unidad funcional autocontenida y de tamaño reducido)

```javascript
function decodeUplink(input) {
  var data = {};
  var bytes = input.bytes;
  var longitud = bytes.length;
  var puerto = input.fPort;

  // Caso 1: Puerto 2, 6 bytes -> nodo monolítico multisensor
  if (puerto === 2 && longitud === 6) {
    var nivelRaw = (bytes[0] << 8) | bytes[1];
    var turbidezRaw = (bytes[2] << 8) | bytes[3];
    var phRaw = (bytes[4] << 8) | bytes[5];

    data.tipo_nodo = "monolito";
    data.familia_nodo = "multisensor";
    data.nivel_cm = nivelRaw / 10.0;
    data.turbidez_ntu = turbidezRaw / 10.0;
    data.ph = phRaw / 100.0;
  }

  // Caso 2: Puerto 3, 5 bytes -> nodo dedicado de pH
  else if (puerto === 3 && longitud === 5) {
    var nodeId = bytes[0];
    var phRaw5 = (bytes[1] << 8) | bytes[2];
    var voltRaw5 = (bytes[3] << 8) | bytes[4];

    data.node_id = nodeId;
    data.tipo_nodo = getPhNodeAlias(nodeId);
    data.familia_nodo = "ph";
    data.ph = phRaw5 / 100.0;
    data.ph_voltage = voltRaw5 / 1000.0;
  }

  // Caso 3: Puerto 2, 4 bytes -> formato heredado turbidez+pH
  else if (puerto === 2 && longitud === 4) {
    var turbidezRaw4 = (bytes[0] << 8) | bytes[1];
    var phRaw4 = (bytes[2] << 8) | bytes[3];

    data.tipo_nodo = "legacy_tp";
    data.familia_nodo = "legacy";
    data.turbidez_ntu = turbidezRaw4 / 10.0;
    data.ph = phRaw4 / 100.0;
    data.nivel_cm = 0.0;
  }

  else {
    return { errors: ["Payload no reconocido. fPort=" + puerto + ", bytes=" + longitud] };
  }

  return { data: data, warnings: [], errors: [] };
}
```

### 3.3 Backend de ingestión — persistencia en MongoDB (Node.js)

Archivo de referencia: `backend/helmo-bridge/index.js`

```javascript
/* Persistencia dual del documento de telemetría:
 * 1. Guarda en Mongo local
 * 2. Intenta replicar en Atlas si está disponible
 */
async function guardarDocumento(documento) {
  let localId = null;
  let atlasId = null;

  try {
    const localRes = await localCollection.insertOne(documento);
    localId = localRes.insertedId;
  } catch (err) {
    console.error('Error guardando en Mongo local:', err.message);
  }

  if (atlasDisponible && atlasCollection) {
    try {
      const atlasRes = await atlasCollection.insertOne(documento);
      atlasId = atlasRes.insertedId;
    } catch (err) {
      atlasDisponible = false;
      atlasCollection = null;
      console.error('Error guardando en Atlas. Se desactiva temporalmente:', err.message);
    }
  }
}

/* Procesamiento de cada mensaje MQTT recibido desde TTN */
client.on('message', async (_topic, message) => {
  const ttnJson = JSON.parse(message.toString());
  const datosSensores = ttnJson.uplink_message?.decoded_payload;
  if (!datosSensores) return;

  const documento = {
    dispositivo: ttnJson.end_device_ids?.device_id || 'desconocido',
    timestamp: ttnJson.received_at ? new Date(ttnJson.received_at) : new Date(),
    variables: datosSensores,
    radio_metadata: {
      fcnt: ttnJson.uplink_message?.f_cnt,
      rssi: ttnJson.uplink_message?.rx_metadata?.[0]?.rssi,
      snr: ttnJson.uplink_message?.rx_metadata?.[0]?.snr,
      gateway: ttnJson.uplink_message?.rx_metadata?.[0]?.gateway_ids?.gateway_id
    }
  };

  await guardarDocumento(documento);
});
```

---

## 4. Manual de usuario y guía de instalación

### 4.1 Requisitos previos

- Visual Studio Code (IDE principal) y Git para control de versiones.
- Arduino IDE con soporte para la placa Heltec WiFi LoRa 32 V3 (librerías `LoRaWan_APP.h`, `HT_SSD1306Wire.h`).
- Node.js y npm para los componentes de backend.
- Cuenta en The Things Stack (TTN) con una aplicación creada.
- MongoDB local (o instancia de MongoDB Atlas) para persistencia de datos.
- Grafana para visualización, y ngrok si se requiere exposición pública temporal.

### 4.2 Instalación del firmware en un nodo

1. Abrir el archivo `.ino` correspondiente al nodo (`firmware/nodo_central`, `nodo_nivel`, `nodo_turbidez` o `nodo_ph`) en Arduino IDE.
2. Instalar la placa Heltec WiFi LoRa 32 V3 y las librerías LoRaWAN requeridas.
3. Reemplazar los marcadores nulos de `devEui`, `appEui` y `appKey` por las credenciales OTAA reales, generadas en la consola de TTN para ese dispositivo. **Estas credenciales nunca deben subirse al repositorio.**
4. Compilar y cargar el firmware en la placa.
5. Verificar por el monitor serial que el nodo se une a la red (`DEVICE_STATE_JOIN`) y transmite periódicamente.

### 4.3 Configuración de la integración con TTN

1. En la consola de TTN, crear el dispositivo final con las credenciales OTAA usadas en el paso anterior.
2. En la aplicación de TTN, configurar un **Custom JavaScript Formatter** como decodificador de uplink, pegando el contenido de `integrations/ttn/uplink_decoder.js`.
3. Configurar un webhook o una integración MQTT hacia el backend, según el módulo que se vaya a usar (`backend/api` para webhook HTTP, `backend/helmo-bridge` para MQTT).

### 4.4 Despliegue del backend

1. Clonar el repositorio y ubicarse en `backend/api` o `backend/helmo-bridge` según el mecanismo de ingestión elegido.
2. Ejecutar `npm install`.
3. Crear un archivo `.env` a partir de la plantilla `env.example` del módulo, con las variables de conexión a MongoDB y, si aplica, las credenciales MQTT de TTN. El archivo `.env` real nunca se sube al repositorio (está excluido por `.gitignore`).
4. Ejecutar el servicio (`node index.js` o mediante un gestor de procesos como `pm2`).
5. Verificar en consola que el servicio se conecta a MongoDB y, según el módulo, al broker MQTT o queda a la espera de webhooks.

### 4.5 Visualización y operación

1. Configurar Grafana apuntando a la base de datos MongoDB donde el backend persiste las lecturas.
2. Construir o importar el dashboard con paneles de nivel, pH, turbidez, RSSI y SNR.
3. Si se requiere acceso remoto temporal para demostración, exponer el puerto de Grafana con `ngrok http <puerto>` y compartir la URL generada.
4. Para operación diaria: verificar periódicamente la conectividad de los nodos, revisar el dashboard y las alertas, y comprobar que el backend permanece activo.

---

## 5. Nota de coherencia documental

El nombre (HELMO), la versión (1.0.0) y los lenguajes declarados en la sección 1 coinciden con el contenido efectivamente presente en el repositorio a la fecha de este anexo (ver `CHANGELOG.md`, entrada `[1.0.0] - 2026-09-22`). Los extractos de código de la sección 3 son transcripciones literales de archivos existentes en el repositorio, referenciados por ruta exacta. El presente documento no reemplaza el código fuente completo ni los demás documentos del repositorio (`README.md`, `docs/`, `legal/`), que permanecen disponibles como respaldo adicional del expediente.

## Autoría

- Esteban Eduardo Escárraga Túquerres.
- Johan Felipe Ordoñez Jenoy.
- Dayerli Katerine Tamayo Solarte.
