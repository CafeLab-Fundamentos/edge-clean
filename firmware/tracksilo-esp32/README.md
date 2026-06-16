# Firmware TrackSilo (ESP32 + DHT22)

Firmware del dispositivo que interactua con `edge-clean`.

## Flujo

1. El ESP32 se conecta al WiFi local.
2. Se anuncia al edge con `POST /api/v1/iam/devices/announce`.
3. El usuario le asigna un lote desde `/onboarding`.
4. El ESP32 envia lecturas a `POST /api/v1/edge/readings`.
5. El edge responde `humidityAlert` y `temperatureAlert`.
6. El firmware actualiza los pines de actuador segun esa respuesta.

## Configuracion relevante

El sketch sigue siendo generico: no lleva `coffeeLotId` ni `api_key` horneados.

```cpp
static const char* DEVICE_ID_OVERRIDE = "";
static const char* EDGE_SERVICE     = "cafelab";
static const char* EDGE_HOST        = "raspberrypi";
static const char* EDGE_FALLBACK_IP = "192.168.18.129";
static const uint16_t EDGE_PORT     = 5000;
```

`EDGE_HOST` y `EDGE_FALLBACK_IP` solo son mecanismos de descubrimiento del edge
en la red local. Ajustalos si tu edge corre en otra maquina o IP.

## Sobre los LED o actuadores

Los LED o actuadores quedan en el ultimo estado aplicado con exito por el
firmware. Si el edge se apaga despues de haber respondido una alerta, el ESP32
no recibe automaticamente un comando de apagado y los pines pueden quedarse
encendidos hasta:

- que llegue una nueva respuesta del edge con alertas en `false`
- que el dispositivo se reinicie
- o que el firmware implemente una politica explicita de fail-safe

## Validacion minima

- El dispositivo aparece en `/onboarding`
- Se le asigna un lote
- `POST /api/v1/edge/readings` responde `201`
- El edge empieza a registrar `Pushed ... reading(s) to backend`
