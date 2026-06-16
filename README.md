# Capa Edge

Capa Edge de CafeLab para el flujo operativo entre Tracksilo y Microservicios.

## Secuencia de uso
Teniendo una cuenta, un perfil, un proveedor y un lote creados...

1) Ejecutar el **Edge** desde su terminal con `python app.py`.

2) Abrir `http://127.0.0.1:5000/onboarding` y colocar el correo y contraseña de la cuenta que se quiere vincular así como la URL del **API Gateway** `https://cafelab-api-gateway-abc123`. Así, el **Edge** hace Sign-In y se vincula a la cuenta del **API gateway**.

3) Encender/conectar por cable a **Tracksilo**.

4) Conectar un dispositivo (ejm. un celular) a la señal WiFi `TrackSilo-Setup` y abrir `192.168.4.1` para después seleccionar la red WiFi en que está actualmente el **Edge** corriendo y pasarle las credenciales. 

5) Esperar a que **Tracksilo** aparezca en estado `pendiente` en `http://127.0.0.1:5000/onboarding`.

6) Asignarle un lote con el botón de asignación y esperar a que **Tracksilo** aparezca en estado "asignado".

7) En el **Edge**, confirmas logs 201: Pulled thresholds y Pushed readings. Mientras que en el microservicio **IoT Monitoring**, confirmas con GET /telemetry-records/coffee-lot/{id}.

## Flujo real

1. **Tracksilo** se anuncia al **Edge** con `POST /api/v1/iam/devices/announce`.
2. El usuario vincula su cuenta en `/onboarding`.
3. El **Edge** consulta los lotes del microservicio **Management** mediante el **API Gateway**.
4. El usuario asigna un `coffeeLotId` al dispositivo desde `/onboarding`.
5. **Tracksilo** envia lecturas al **Edge** con `POST /api/v1/edge/readings`.
6. El **Edge** responde alertas locales al instante.
7. El worker sincroniza lecturas pendientes al microservicio **IoT Monitoring** `POST /api/v1/telemetry-records`.
8. El worker descarga los últimos umbrales desde `GET /api/v1/environment-thresholds/coffee-lot/{coffeeLotId}`.

## Ejecutar local

```powershell
pip install -r requirements.txt
python app.py
```

URL local:

```text
http://127.0.0.1:5000
```

Base local:

```text
edge_clean.db
```

El proyecto carga variables desde `.env`. En este flujo, la principal es del api gateway

## Endpoints principales del edge

| Metodo | Endpoint | Uso |
|---|---|---|
| GET | `/` | health check |
| GET | `/onboarding` | vincular cuenta y asignar lotes |
| POST | `/api/v1/iam/devices/announce` | auto-registro del ESP32 |
| POST | `/api/v1/edge/readings` | ingreso de lecturas del dispositivo |
| GET | `/api/v1/edge/readings/latest` | ultima lectura local |
| GET | `/api/v1/edge/readings` | historial local |
| GET | `/api/v1/edge/thresholds` | umbrales locales |
| GET | `/api/v1/edge/sensor-status` | estado online/offline |
| GET | `/api/v1/edge/actuator-events` | eventos locales |
| GET | `/api/v1/edge/sync/status` | estado del worker |
| POST | `/api/v1/edge/sync` | forzar sincronizacion |

## Notas operativas

- El edge sigue funcionando localmente aunque el backend falle por un rato.
- La telemetria se guarda primero en SQLite y luego se sincroniza.
- Un dispositivo sin `lotId` numerico valido no se sincroniza con el backend.
- Los LED o actuadores del ESP32 dependen del ultimo comando exitoso recibido desde el edge.
