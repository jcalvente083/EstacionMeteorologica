# Monitor IoT con ESP32 + MQTT + InfluxDB + Grafana

Proyecto educativo para monitorizar temperatura y humedad con un ESP32, visualizar los datos en tiempo real con Grafana y recibir alertas por Telegram.

---

## Arquitectura

```
ESP32  ──MQTT──►  Mosquitto  ──►  Telegraf  ──►  InfluxDB  ──►  Grafana
                                                                    │
                                                              Dashboard en
                                                              tiempo real
         │
         └──Telegram Bot──► Alertas por temperatura/humedad
```

---

## Requisitos previos

- [Docker Desktop](https://www.docker.com/products/docker-desktop/) instalado y en ejecución
- [Arduino IDE](https://www.arduino.cc/en/software) con soporte para ESP32
- Cuenta de Telegram (para el bot de alertas)
- Librerías Arduino necesarias:
  - `PubSubClient`
  - `UniversalTelegramBot`
  - `WiFiClientSecure` (incluida en el core ESP32)

---

## Estructura del repositorio

```
├── docker-compose.yml        # Define los 4 servicios del stack
├── ESP32.txt                 # Código Arduino para el ESP32 (con TODOs)
├── mosquitto/
│   └── config/
│       └── mosquitto.conf    # Configuración del broker MQTT
└── telegraf/
    └── telegraf.conf         # Pipeline MQTT → InfluxDB
```

---

## 1. Despliegue del stack con Docker

Clona el repositorio y levanta todos los servicios con un único comando:

```bash
git clone <url-del-repo>
cd Grafana
docker compose up -d
```

Esto levanta automáticamente:

| Servicio    | Puerto | Descripción                        |
|-------------|--------|------------------------------------|
| Mosquitto   | 1883   | Broker MQTT                        |
| InfluxDB    | 8086   | Base de datos de series temporales |
| Telegraf    | —      | Puente MQTT → InfluxDB             |
| Grafana     | 3000   | Panel de visualización             |

Verifica que los contenedores están corriendo:

```bash
docker compose ps
```

---

## 2. Configurar Grafana

1. Abre el navegador en `http://localhost:3000`
2. Credenciales por defecto: **admin / admin** (te pedirá cambiarla)
3. Ve a **Connections → Data Sources → Add data source**
4. Selecciona **InfluxDB** y configura:
   - URL: `http://influxdb:8086`
   - Database: `sensores`
5. Pulsa **Save & Test** — debe aparecer un mensaje verde
6. Crea un nuevo Dashboard y añade paneles para `temperatura` y `humedad`

---

## 3. Configurar el ESP32

Abre `ESP32.txt` en el Arduino IDE y rellena todos los `#TODO`:

### Red y MQTT
```cpp
const char* ssid       = "NOMBRE_DE_TU_RED";
const char* password   = "CONTRASEÑA_WIFI";
const char* mqtt_server = "192.168.X.X";  // IP de tu PC (ipconfig)
```

### Bot de Telegram
```cpp
#define TOKEN_BOT "TOKEN_DE_BOTFATHER"
#define CHAT_ID   "TU_CHAT_ID"
```

### Pines del hardware
```cpp
const int thermistorPin = 34;           // Pin ADC del termistor
const byte ledPins[] = {25, 26, 27};    // Pines R, G, B del LED RGB
```

### Intervalos y umbrales
```cpp
const int delayTime       = 5000;   // ms entre lecturas
float TEMPERATURA_MINIMA  = 18.0;
float TEMPERATURA_MAXIMA  = 30.0;
float HUMEDAD_MINIMA      = 40.0;
float HUMEDAD_MAXIMA      = 80.0;
```

### Funciones a implementar

| Función | Descripción |
|---------|-------------|
| `getHumedad()` | Retornar un `float` aleatorio entre 40 y 80 |
| `loop()` | Llamar a `getTemperatura()` y `getHumedad()` |
| `avisar()` | Encender LED y enviar Telegram según umbrales |
| `encenderLed()` | Completar el `switch` para `'R'`, `'G'`, `'B'` |

---

## 4. Montaje del hardware

El circuito incluye:
- **Termistor NTC 10kΩ** conectado a un pin ADC del ESP32 (divisor de tensión con resistencia de 10kΩ a 3.3V)
- **LED RGB de cátodo común** con resistencias limitadoras en cada canal

> Los esquemas de montaje están disponibles en las fotos del material de clase.

---

## 5. Flujo de datos

1. El ESP32 lee la temperatura del termistor y genera una humedad simulada
2. Publica el JSON `{"temperatura": X.XX, "humedad": X.XX}` en el tópico MQTT `esp32/clima`
3. Telegraf suscribe ese tópico y escribe los valores en InfluxDB
4. Grafana consulta InfluxDB y actualiza los paneles en tiempo real
5. Si los valores superan los umbrales, el ESP32 envía una alerta por Telegram y cambia el color del LED

---

## Comandos útiles

```bash
# Ver logs de todos los servicios
docker compose logs -f

# Parar el stack
docker compose down

# Reiniciar un servicio concreto
docker compose restart telegraf
```

---

## Licencia

Proyecto educativo de uso libre para clase.
