# Diseño del Sistema — HogarSeguro IA

**Versión:** 0.1  
**Fecha:** 2026-07-19  
**Estado:** Fase 0 — Planificación

---

## 1. Modelo de datos

### 1.1 Entidades principales

```sql
-- Hogar: unidad residencial registrada
households (
  id          uuid PRIMARY KEY,
  address     text NOT NULL,
  city        text NOT NULL,
  stratum     int CHECK (stratum BETWEEN 1 AND 6),
  created_at  timestamptz DEFAULT now()
)

-- Dispositivo: hardware instalado en el hogar
devices (
  id            uuid PRIMARY KEY,
  household_id  uuid REFERENCES households(id),
  serial        text UNIQUE NOT NULL,
  firmware_ver  text,
  model_ver     text,
  last_seen     timestamptz,
  status        text CHECK (status IN ('online','offline','alert'))
)

-- Evento: lectura de sensor o incidente detectado
events (
  id           uuid PRIMARY KEY,
  device_id    uuid REFERENCES devices(id),
  type         text NOT NULL,  -- gas_leak | smoke | fire_visual | fall | asphyxia | normal
  severity     text CHECK (severity IN ('info','warning','critical')),
  sensor_data  jsonb,          -- {gas_ppm, temp_c, smoke_level, confidence}
  actuated     boolean DEFAULT false,
  notified_at  timestamptz,
  created_at   timestamptz DEFAULT now()
)

-- Usuario: cuidador primario o familiar registrado
users (
  id           uuid PRIMARY KEY,  -- sincronizado con Supabase Auth
  household_id uuid REFERENCES households(id),
  role         text CHECK (role IN ('primary','secondary')),
  push_token   text,
  phone        text
)

-- Simulacro: registro de ejercicio educativo completado
drills (
  id           uuid PRIMARY KEY,
  household_id uuid REFERENCES households(id),
  type         text,            -- gas | fire | choking
  completed_at timestamptz,
  score        int CHECK (score BETWEEN 0 AND 100)
)

-- Suscripción
subscriptions (
  id              uuid PRIMARY KEY,
  household_id    uuid REFERENCES households(id),
  plan            text CHECK (plan IN ('basic','subsidized')),
  price_cop       int,
  active          boolean DEFAULT true,
  renews_at       timestamptz
)
```

### 1.2 Índices críticos

```sql
CREATE INDEX idx_events_device_created ON events(device_id, created_at DESC);
CREATE INDEX idx_events_severity ON events(severity) WHERE severity = 'critical';
CREATE INDEX idx_devices_status ON devices(status) WHERE status = 'alert';
```

---

## 2. API REST (FastAPI)

### 2.1 Endpoints del dispositivo (autenticación por certificado de dispositivo)

```
POST /device/events          Publica evento desde dispositivo
POST /device/heartbeat       Actualiza last_seen y estado
GET  /device/config          Descarga configuración (umbrales, intervalos)
GET  /device/ota/check       Consulta si hay actualización disponible
GET  /device/ota/download    Descarga artefacto firmado
```

### 2.2 Endpoints de usuario (autenticación Supabase JWT)

```
GET  /households/{id}/status          Estado en tiempo real del hogar
GET  /households/{id}/events          Historial de eventos (paginado)
GET  /households/{id}/devices         Dispositivos registrados
POST /households/{id}/devices/pair    Empareja nuevo dispositivo
GET  /households/{id}/drills          Historial de simulacros
POST /households/{id}/drills          Registra simulacro completado
GET  /households/{id}/subscription    Estado de suscripción
```

### 2.3 Endpoints de emergencias (webhook, autenticación por API key)

```
POST /emergency/notify       Recibe confirmación de 123 / despachador
GET  /emergency/events/{id}  Detalle de incidente para primera respuesta
```

### 2.4 Estructura de respuesta estándar

```json
{
  "success": true,
  "data": { ... },
  "error": null,
  "meta": { "total": 100, "page": 1, "limit": 20 }
}
```

---

## 3. Firmware — Máquina de estados

```
IDLE ──────────────────────────────────────────────────┐
  │  sensor reading OK                                  │
  │  inference: normal                                  │
  ▼                                                     │
MONITORING ────────── threshold exceeded ──────▶ ALERT  │
  │                   or class != normal                 │
  │  (low power, 1 fps camera)                          │
  │                                          ┌──────────┘
  └─────────── all clear ◀──────────────────-┤
                                             │
                                    ACTUATING (electrovalve)
                                             │
                                    NOTIFYING (MQTT + local alarm)
                                             │
                                    WAITING_RESET (manual only)
```

**Transiciones de estado:**
- `IDLE → MONITORING`: arranque exitoso, sensores calibrados
- `MONITORING → ALERT`: cualquier sensor supera umbral O clasificador > 0.85 en clase anómala
- `ALERT → ACTUATING`: gas_leak o fire_visual confirmado
- `ACTUATING → NOTIFYING`: electroválvula cerrada exitosamente
- `NOTIFYING → WAITING_RESET`: notificaciones enviadas
- `WAITING_RESET → MONITORING`: usuario confirma en app que el área es segura

---

## 4. Modelo de IA — Clasificador de comportamiento visual

### 4.1 Clases objetivo

| Clase | Descripción | Umbral actuación |
|-------|-------------|-----------------|
| `normal` | Actividad cotidiana sin riesgo | — |
| `fall` | Persona caída en el suelo | Push + audio confirmación |
| `asphyxia` | Indicadores visuales de asfixia | Push + 123 si persiste > 10s |
| `fire_visual` | Llamas visibles en cocina u otra área | Actuación inmediata |
| `unattended_child` | Niño < 5 años solo en área de riesgo | Push a cuidador |

### 4.2 Pipeline de entrenamiento

```
Recolección de datos → Anotación (CVAT) → Aumentación
  → Entrenamiento (MobileNetV3 Small / EfficientDet-Lite)
  → Cuantización INT8 para TFLite
  → Validación en hardware (latencia < 100 ms en ESP32-S3)
  → Firma y empaquetado para OTA
```

### 4.3 Métricas de validación

- Precisión global: > 90 % en conjunto de prueba
- Recall en clases críticas (fire, fall, asphyxia): > 95 %
- Falsos positivos diarios por hogar: < 2
- Latencia de inferencia en dispositivo: < 100 ms

---

## 5. Notificaciones — Flujo detallado

```
Evento crítico generado
  │
  ├─▶ Push inmediata (FCM/APNs) → contactos primario y secundario
  │     Payload: {tipo, hogar, timestamp, acción_tomada}
  │
  ├─▶ Si no hay ACK en 60s → llamada automatizada al primario
  │
  └─▶ Si no hay ACK en 120s → payload estructurado a línea 123
        {incidente, dirección, coordenadas GPS, sensores activos}
```

---

## 6. Estructura de carpetas del proyecto (futura Fase 1)

```
hogar-seguro-ia/
├── firmware/               # ESP-IDF project
│   ├── main/
│   ├── components/
│   │   ├── sensors/
│   │   ├── ai_inference/
│   │   ├── actuators/
│   │   └── mqtt_client/
│   └── models/             # .tflite cuantizados
├── backend/                # FastAPI
│   ├── app/
│   │   ├── api/
│   │   ├── models/
│   │   ├── services/
│   │   └── core/
│   └── supabase/
│       └── migrations/
├── mobile/                 # Expo React Native
│   ├── src/
│   │   ├── screens/
│   │   ├── components/
│   │   ├── hooks/
│   │   └── store/
│   └── assets/
├── ml/                     # Training pipeline
│   ├── datasets/
│   ├── training/
│   └── export/
├── docs/
└── tests/
```
