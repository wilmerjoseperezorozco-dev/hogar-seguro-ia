# Arquitectura del Sistema — HogarSeguro IA

**Versión:** 0.1  
**Fecha:** 2026-07-19  
**Estado:** Fase 0 — Planificación

---

## 1. Visión general

HogarSeguro IA adopta una arquitectura **edge-first**: la lógica de detección y actuación crítica reside en el dispositivo local. La nube complementa con alertas, telemetría, actualizaciones de modelos y experiencia de usuario, pero nunca es un punto de falla para la seguridad del hogar.

```
┌─────────────────────────────────────────────────────────┐
│                   HOGAR (Edge Layer)                     │
│                                                          │
│  ┌──────────┐   ┌──────────┐   ┌──────────────────────┐ │
│  │ Sensores │──▶│  MCU     │──▶│  Edge AI Module       │ │
│  │ Gas/Humo │   │ ESP32-S3 │   │  TFLite / ONNX        │ │
│  │ Temp/Mic │   │          │   │  Clasificador visual  │ │
│  └──────────┘   └────┬─────┘   └──────────────────────┘ │
│                      │                                   │
│              ┌───────▼────────┐                          │
│              │  Actuadores    │                          │
│              │  Electroválvula│                          │
│              │  Alarma sonora │                          │
│              └────────────────┘                          │
└─────────────────────────┬───────────────────────────────┘
                          │ WiFi / 4G (alertas)
                          ▼
┌─────────────────────────────────────────────────────────┐
│                   CLOUD LAYER                            │
│                                                          │
│  ┌──────────────┐  ┌──────────────┐  ┌───────────────┐  │
│  │  API Gateway │  │  Supabase    │  │  OTA Service  │  │
│  │  (FastAPI)   │  │  PostgreSQL  │  │  Firmware +   │  │
│  │              │  │  + Realtime  │  │  Modelos IA   │  │
│  └──────┬───────┘  └──────┬───────┘  └───────────────┘  │
│         │                 │                              │
│  ┌──────▼───────┐  ┌──────▼───────┐                     │
│  │  Notif. Push │  │  Dashboard   │                     │
│  │  FCM / APNs  │  │  Admin       │                     │
│  └──────────────┘  └──────────────┘                     │
└─────────────────────────┬───────────────────────────────┘
                          │
                          ▼
             ┌────────────────────────┐
             │  CLIENTES              │
             │  App móvil (Expo RN)   │
             │  Línea 123 (webhook)   │
             │  API aseguradoras      │
             └────────────────────────┘
```

---

## 2. Capas del sistema

### 2.1 Edge Layer (Dispositivo)

**Responsabilidades:**
- Lectura continua de sensores (polling 200 ms)
- Inferencia local del modelo de clasificación de comportamiento (< 100 ms)
- Decisión de actuación física sin dependencia de red
- Publicación de eventos a la nube cuando hay conectividad

**Tecnologías:**
- MCU: ESP32-S3 (dual-core 240 MHz, WiFi integrado, soporte aceleración NN)
- RTOS: ESP-IDF + FreeRTOS
- Inferencia: TensorFlow Lite for Microcontrollers
- Comunicación nube: MQTT sobre TLS (broker: AWS IoT Core o HiveMQ)

**Principio de diseño clave:** toda decisión crítica de seguridad (cierre de gas, alarma) se ejecuta localmente. La pérdida de conectividad no degrada la seguridad.

### 2.2 Cloud Layer (Backend)

**Responsabilidades:**
- Recepción y persistencia de eventos del dispositivo
- Orquestación de notificaciones push
- Almacenamiento de historial para análisis
- Serving de actualizaciones OTA (firmware + modelos)
- Autenticación y gestión de usuarios/hogares

**Tecnologías:**
- API: FastAPI (Python 3.12)
- Base de datos: Supabase (PostgreSQL + Row Level Security)
- Tiempo real: Supabase Realtime (websockets)
- Colas: Redis + Celery (procesamiento asíncrono de alertas)
- OTA: bucket S3-compatible con firma de artefactos (Ed25519)
- Infraestructura: Railway o Render (MVP), migrable a VPS propio

### 2.3 App Móvil

**Responsabilidades:**
- Visualización del estado del dispositivo en tiempo real
- Recepción de alertas push
- Simulacros y contenido educativo
- Configuración del hogar y contactos de emergencia

**Tecnologías:**
- React Native + Expo (cross-platform iOS/Android)
- Estado: Zustand
- Datos servidor: TanStack Query
- Notificaciones: Expo Notifications + FCM/APNs

---

## 3. Flujos críticos

### 3.1 Detección de fuga de gas → actuación

```
Sensor gas (lectura) → MCU compara umbral → supera umbral
  → Cierre electroválvula (< 500 ms)
  → Alarma sonora local
  → Publicación evento MQTT {tipo: "gas", nivel: X, timestamp}
    → API recibe → guarda en DB → dispara notificación push
    → Si persiste > 30s → llamada automatizada 123
```

### 3.2 Detección visual de comportamiento anómalo

```
Cámara captura frame (1 fps en standby, 5 fps en alerta)
  → TFLite clasifica: {normal, caída, asfixia, fuego_visual}
  → Si clase != normal y confianza > 0.85:
    → Escala a modo alerta
    → Activa micrófono para confirmación por audio
    → Si confirmado: genera evento + notificación
```

### 3.3 Actualización OTA de modelo

```
OTA Service publica nueva versión firmada
  → Dispositivo verifica firma Ed25519
  → Descarga a partición secundaria (A/B update)
  → Valida checksum SHA-256
  → Reinicio en nueva partición
  → Si fallo: rollback automático a versión anterior
```

---

## 4. Decisiones de arquitectura

| Decisión | Alternativa descartada | Razón |
|----------|----------------------|-------|
| Inferencia 100% local | Streaming de video a nube | Privacidad del hogar; latencia inadmisible para actuación crítica |
| ESP32-S3 como MCU | Raspberry Pi | Costo (< 15 USD vs > 50 USD), consumo de energía, disponibilidad local |
| MQTT sobre HTTP polling | REST webhooks desde dispositivo | Bidireccionalidad, reconexión automática, QoS garantizado |
| Supabase como DB | Firebase | SQL nativo, Row Level Security granular, sin vendor lock-in de SDK |
| Expo React Native | Flutter | Ecosistema JS unificado con backend Python/FastAPI; menor curva equipo |
| A/B OTA con rollback | Actualización in-place | Garantía de disponibilidad ante fallo de actualización en campo |

---

## 5. Seguridad y privacidad

- Video nunca sale del dispositivo; solo metadatos clasificados se envían a la nube.
- Comunicación dispositivo-nube cifrada (TLS 1.3 + certificados de dispositivo).
- Firmware y modelos firmados digitalmente antes de distribución OTA.
- Base de datos con RLS: cada hogar accede solo a sus propios datos.
- Sin identificadores biométricos almacenados.
