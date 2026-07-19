# Lista de Tareas — HogarSeguro IA

**Versión:** 0.1  
**Fecha:** 2026-07-19  
**Estado:** Fase 0 completada

---

## Fase 0 — Planificación ✅

- [x] Definición del problema y segmentos de usuarios
- [x] PRD con funcionalidades MVP y métricas de éxito
- [x] Arquitectura edge-first documentada
- [x] Diseño del sistema: modelo de datos, API, firmware FSM, pipeline ML
- [x] Documentación técnica: stack, integraciones, seguridad
- [x] Inicialización del repositorio GitHub

---

## Fase 1 — Infraestructura

### 1.1 Firmware base

- [ ] Configurar proyecto ESP-IDF con estructura de componentes
- [ ] Implementar lectura de sensores MQ-2, DHT22, MAX9814
- [ ] Implementar control de electroválvula y buzzer (GPIOs)
- [ ] Integrar cliente MQTT con reconexión automática y QoS 1
- [ ] Implementar máquina de estados (IDLE → MONITORING → ALERT → ACTUATING)
- [ ] Integrar TFLite for Microcontrollers con modelo stub (placeholder)
- [ ] Implementar particiones A/B para OTA con rollback
- [ ] Pipeline de compilación en GitHub Actions (esp-idf Docker)

### 1.2 Backend

- [ ] Scaffold FastAPI con estructura de módulos (api, models, services, core)
- [ ] Configurar Supabase: esquema inicial, migraciones, RLS
- [ ] Implementar autenticación mutual TLS para dispositivos
- [ ] Endpoints CRUD: households, devices, events, users
- [ ] Integrar Redis + Celery para cola de notificaciones
- [ ] Webhook de notificación push (FCM placeholder)
- [ ] Health check y endpoint de OTA check
- [ ] Pipeline CI/CD: GitHub Actions → tests → deploy Railway

### 1.3 App móvil

- [ ] Inicializar proyecto Expo React Native
- [ ] Configurar navegación (React Navigation)
- [ ] Pantalla de login/registro (Supabase Auth)
- [ ] Scaffold de pantallas principales (Home, Events, Drills, Settings)
- [ ] Integrar Expo Notifications (push token, FCM)
- [ ] Pantalla de pairing de dispositivo (QR + serial)

### 1.4 ML Pipeline

- [ ] Definir taxonomía de clases y protocolo de anotación
- [ ] Configurar CVAT self-hosted para anotación
- [ ] Script de augmentación con Albumentations
- [ ] Script de fine-tuning MobileNetV3 Small (PyTorch)
- [ ] Pipeline de exportación ONNX → TFLite INT8
- [ ] Benchmark de latencia en ESP32-S3

---

## Fase 2 — Estados vacíos + E2E

- [ ] Flujo completo: sensor → MCU → MQTT → backend → push → app
- [ ] Test E2E de evento crítico: gas_leak → electroválvula → notificación
- [ ] Test E2E de OTA: publicar nueva versión → dispositivo aplica → rollback
- [ ] Pantallas con estados de carga, vacío y error en app
- [ ] Suite de tests de integración del backend (pytest)
- [ ] Test de reconexión MQTT ante pérdida de WiFi
- [ ] Validación de RLS: usuario no accede a datos de otro hogar

---

## Fase 3 — Preentrega (Piloto)

- [ ] Ensamblaje de 5 prototipos físicos
- [ ] Instalación piloto en 5 hogares de estrato 1-2 (Barranquilla)
- [ ] Calibración de umbrales de sensores en condiciones reales
- [ ] Validación del clasificador visual con datos del piloto
- [ ] Integración real con línea 123 (acuerdo con DAGRD)
- [ ] App en TestFlight (iOS) y Play Store interno (Android)
- [ ] Dashboard de monitoreo de flota de dispositivos
- [ ] Documentación de instalación para técnico no especializado

---

## Fase 4 — Iteración

- [ ] Análisis de feedback del piloto (NPS, incidentes reales, falsos positivos)
- [ ] Reentrenamiento del modelo con datos del piloto
- [ ] Reducción de BOM hacia objetivo < 80.000 COP (PCB propia)
- [ ] Ampliación a 50 hogares
- [ ] Integración con primera aseguradora (API o CSV)
- [ ] Módulo de red mesh para múltiples dispositivos por hogar
- [ ] Localización de la app en comunidades con lengua nativa (piloto Mokaná)
- [ ] Evaluación de cofinanciación Ministerio de TIC / FONTIC

---

## Dependencias y bloqueos conocidos

| Bloqueo | Acción requerida |
|---------|-----------------|
| Acuerdo con línea 123 / DAGRD | Contacto formal con Secretaría de Gobierno del Atlántico |
| Dataset de comportamiento visual en hogares colombianos | Recolección con consentimiento en piloto (Fase 3) |
| Reducción BOM a < 80.000 COP | Diseño de PCB propia + compras por volumen (Fase 4) |
| Integración con aseguradoras | Negociación comercial paralela a Fase 3 |
