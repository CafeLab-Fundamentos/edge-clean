# edge-clean

Edge local de CafeLab para el flujo real entre dispositivo IoT, edge y backend.

- `app.py`
- `iam/`
- `iotmonitoring/`
- `shared/`
- `firmware/tracksilo-esp32/`


## Flujo real validado

1. El ESP32 se anuncia al edge con `POST /api/v1/iam/devices/announce`.
2. El usuario vincula su cuenta en `/onboarding`.
3. El edge consulta los lotes del backend por el API Gateway.
4. El usuario asigna un `coffeeLotId` al dispositivo desde `/onboarding`.
5. El ESP32 envia lecturas al edge con `POST /api/v1/edge/readings`.
6. El edge responde alertas locales al instante.
7. El worker sincroniza pendientes a `POST /api/v1/telemetry-records`.
8. El worker baja umbrales desde `GET /api/v1/environment-thresholds/coffee-lot/{coffeeLotId}`.

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
