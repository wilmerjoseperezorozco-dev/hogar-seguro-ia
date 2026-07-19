# Documentación Técnica — HogarSeguro IA

**Versión:** 0.1  
**Fecha:** 2026-07-19  
**Estado:** Fase 0 — Planificación

---

## 1. Stack tecnológico completo

### 1.1 Hardware

| Componente | Modelo | Función | Costo estimado (COP) |
|-----------|--------|---------|---------------------|
| MCU principal | ESP32-S3 DevKit | Procesamiento central, WiFi, inferencia NN | 35.000 |
| Sensor gas | MQ-2 | Detección de GLP, metano, humo | 8.000 |
| Sensor humo/temp | DHT22 + MQ-135 | Temperatura ambiente, partículas | 12.000 |
| Sensor audio | MAX9814 | Micrófono con AGC para detección de patrones | 10.000 |
| Cámara | OV2640 | Captura visual para inferencia | 18.000 |
| Electroválvula | Electroválvula 1/2" 12V | Cierre automatizado de gas | 45.000 |
| Relé | Módulo relé 5V | Control de electroválvula y alarma | 5.000 |
| Buzzer | Buzzer activo 12V 110dB | Alarma sonora local | 7.000 |
| Fuente | HLK-PM01 5V | Alimentación desde red domiciliaria | 15.000 |
| Carcasa | PLA/ABS impreso 3D | Enclosure resistente | 15.000 |
| **Total BOM** | | | **~170.000** |

> Objetivo: reducir BOM a < 80.000 COP mediante compras por volumen y diseño optimizado de PCB propia en Fase 3.

### 1.2 Firmware

| Tecnología | Versión | Uso |
|-----------|---------|-----|
| ESP-IDF | 5.x | Framework base RTOS |
| FreeRTOS | Incluido en IDF | Multitarea: sensores, inferencia, MQTT |
| TFLite for Microcontrollers | 2.x | Inferencia local del clasificador |
| esp-mqtt | Incluido en IDF | Publicación de eventos a broker |
| mbedTLS | Incluido en IDF | TLS 1.3 para comunicaciones seguras |
| esp_https_ota | Incluido en IDF | Actualizaciones OTA con verificación |

### 1.3 Backend

| Tecnología | Versión | Uso |
|-----------|---------|-----|
| Python | 3.12 | Lenguaje principal |
| FastAPI | 0.115+ | Framework API REST |
| Supabase | Cloud / self-hosted | PostgreSQL + Auth + Realtime + Storage |
| Redis | 7.x | Cola de tareas y caché |
| Celery | 5.x | Workers asíncronos (notificaciones, 123) |
| Pydantic v2 | 2.x | Validación de esquemas |
| pytest | 8.x | Tests unitarios e integración |

### 1.4 App Móvil

| Tecnología | Versión | Uso |
|-----------|---------|-----|
| React Native | 0.74+ | Framework móvil cross-platform |
| Expo | SDK 51+ | Toolchain, OTA de app, notificaciones |
| Zustand | 4.x | Estado global cliente |
| TanStack Query | 5.x | Caché y sincronización de datos servidor |
| Expo Notifications | SDK 51 | Push notifications FCM/APNs |
| React Navigation | 6.x | Navegación entre pantallas |

### 1.5 ML Pipeline

| Tecnología | Uso |
|-----------|-----|
| Python 3.12 + PyTorch | Entrenamiento |
| MobileNetV3 Small / EfficientDet-Lite | Arquitectura base (transfer learning) |
| CVAT (self-hosted) | Anotación de datasets |
| Albumentations | Aumentación de datos |
| ONNX → TFLite | Exportación y cuantización INT8 |
| Weights & Biases | Tracking de experimentos |

---

## 2. Integraciones externas

### 2.1 Línea de Emergencias 123

- **Mecanismo:** webhook HTTP POST con payload estructurado
- **Payload mínimo:**
  ```json
  {
    "incident_type": "gas_leak",
    "address": "Calle 45 #23-10, Barranquilla",
    "coordinates": {"lat": 10.9639, "lng": -74.7964},
    "sensors": {"gas_ppm": 850, "temp_c": 32},
    "household_occupants": 4,
    "vulnerable_present": true,
    "contact_phone": "+573001234567"
  }
  ```
- **Requerimiento:** acuerdo formal con Secretaría de Gobierno del Atlántico o DAGRD

### 2.2 Aseguradoras

- API REST con autenticación OAuth2
- Reporte mensual de historial: `{household_id, drills_completed, incidents_zero_months, score}`
- Piloto inicial: gestión directamente sin API (exportación CSV)

### 2.3 Firebase Cloud Messaging / APNs

- Notificaciones push estándar usando Expo Notifications
- Prioridad HIGH para eventos críticos (bypass modo silencio en iOS/Android)

---

## 3. Seguridad — Implementación técnica

### 3.1 Autenticación del dispositivo

- Cada dispositivo genera par de claves Ed25519 en primera inicialización
- Clave privada almacenada en eFuse del ESP32 (no extraíble)
- Certificado de dispositivo firmado por CA propia durante provisioning
- Mutual TLS en todas las comunicaciones dispositivo-nube

### 3.2 Firma de artefactos OTA

```
Generar artefacto → SHA-256 hash → Firmar con clave privada del servidor (Ed25519)
Dispositivo: descarga artefacto + firma → verifica firma con clave pública embedida
→ Si OK: aplica actualización → Si falla: descarta y reporta error
```

### 3.3 Row Level Security (Supabase)

```sql
-- Cada usuario solo ve eventos de su hogar
CREATE POLICY "household_isolation" ON events
  USING (device_id IN (
    SELECT id FROM devices WHERE household_id = auth.jwt()->>'household_id'
  ));
```

---

## 4. Variables de entorno (backend)

```env
# Supabase
SUPABASE_URL=
SUPABASE_SERVICE_KEY=

# MQTT Broker
MQTT_BROKER_URL=
MQTT_USERNAME=
MQTT_PASSWORD=

# Firebase (push)
FIREBASE_SERVICE_ACCOUNT_JSON=

# Redis
REDIS_URL=

# OTA signing
OTA_SIGNING_KEY_PATH=

# Emergency services
EMERGENCY_123_WEBHOOK_URL=
EMERGENCY_123_API_KEY=

# App
SECRET_KEY=
DEBUG=false
ALLOWED_ORIGINS=
```

---

## 5. Consideraciones de despliegue (Fase 1+)

- **Railway / Render (MVP):** deploy desde GitHub Actions en merge a `main`
- **CI/CD:** GitHub Actions → lint + tests → build Docker → deploy
- **Firmware:** pipeline de compilación en GitHub Actions (esp-idf Docker image)
- **Monitoreo:** Sentry para backend y app móvil; logs estructurados JSON
- **Backups:** respaldo diario automático de PostgreSQL (Supabase incluido)
