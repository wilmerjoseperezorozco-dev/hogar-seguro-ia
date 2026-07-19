# PRD — HogarSeguro IA

**Versión:** 0.1  
**Fecha:** 2026-07-19  
**Autor:** Wilmer José Pérez Orozco  
**Estado:** Borrador — Fase 0

---

## 1. Contexto y problema

Los accidentes domésticos prevenibles —explosiones de gas, incendios en cocina, asfixia y electrocución— constituyen una causa de muerte y discapacidad subestimada en Colombia. Las intervenciones críticas deben ocurrir dentro de una ventana de segundos para ser efectivas. Los hogares de estratos 1-3, donde la incidencia es mayor, carecen de acceso a sistemas de seguridad que exigen instalaciones costosas o suscripciones inalcanzables.

Los grupos de mayor riesgo —niños menores de 5 años y adultos mayores de 60— presentan menor capacidad de respuesta autónoma ante emergencias, lo que amplifica la necesidad de automatización.

---

## 2. Objetivo del producto

Proveer un dispositivo doméstico de bajo costo y una plataforma de software asociada que:

1. Detecte situaciones de riesgo en tiempo real mediante sensores y visión por computadora.
2. Active respuestas físicas automatizadas (cierre de válvula de gas).
3. Notifique a cuidadores primarios y servicios de emergencia con información de contexto estructurada.
4. Eduque a los miembros del hogar mediante simulacros y contenido preventivo periódico.

---

## 3. Usuarios objetivo

| Segmento | Descripción |
|----------|-------------|
| Hogares vulnerables | Familias en estratos 1-3 con presencia de niños < 5 años o adultos > 60 |
| Cuidadores primarios | Persona responsable del hogar, receptor principal de alertas |
| Servicios de emergencia | Bomberos, defensa civil, línea 123 — receptores de datos estructurados |
| Aseguradoras | Aliadas potenciales para cofinanciación mediante descuentos en primas |

---

## 4. Funcionalidades principales (MVP)

### 4.1 Detección multi-sensor

- Sensor de gas (MQ-2 o equivalente) con umbral configurable
- Sensor de humo y temperatura (DHT22 + sensor óptico de humo)
- Micrófono para detección de patrones sonoros (llanto, tos persistente, alarmas)
- Cámara de bajo consumo (ESP32-CAM o equivalente) para inferencia local

### 4.2 Inferencia en el borde

- Modelo de clasificación de comportamiento (TFLite / ONNX Runtime)
- Sin transmisión de video a la nube — procesamiento 100 % local para privacidad
- Actualización de modelos mediante OTA firmado

### 4.3 Actuación física

- Cierre automático de electroválvula de gas al superar umbral
- Alarma sonora local
- Reconexión manual obligatoria con confirmación de seguridad

### 4.4 Alertas y comunicación

- Notificación push a app móvil (cuidadores registrados)
- Llamada automática a línea 123 con payload estructurado: tipo de incidente, ubicación GPS, estado del sensor
- Registro de eventos para análisis posterior

### 4.5 App móvil

- Panel de estado del dispositivo en tiempo real
- Simulacros de emergencia mensuales con evaluación
- Contenido educativo de 1 minuto (video corto) adaptado al tipo de hogar
- Historial de eventos e incidentes
- Configuración de umbrales y contactos de emergencia

### 4.6 Sistema de incentivos

- Integración con API de aseguradoras participantes para reportar historial limpio
- Descuento progresivo en prima según meses sin incidentes
- Puntaje de seguridad del hogar visible en app

---

## 5. Funcionalidades fuera de alcance (MVP)

- Reconocimiento facial de miembros del hogar
- Integración con sistemas de domótica de terceros
- Dashboard para entidades gubernamentales (Fase 2+)
- Múltiples dispositivos por hogar en red mesh (Fase 2+)

---

## 6. Métricas de éxito

| Métrica | Objetivo MVP |
|---------|-------------|
| Tiempo de detección → actuación (gas) | < 3 segundos |
| Precisión del clasificador de comportamiento | > 90 % en condiciones de laboratorio |
| Tasa de falsos positivos (alarmas innecesarias) | < 2 % mensual por hogar |
| Costo del hardware por unidad (BOM) | < 80.000 COP |
| Adopción piloto | 50 hogares en estrato 1-3 (Barranquilla, Atlántico) |
| Satisfacción de cuidadores | NPS > 40 a los 3 meses |

---

## 7. Restricciones y supuestos

- El hogar debe contar con conexión WiFi o datos móviles para alertas remotas.
- El dispositivo no reemplaza la responsabilidad del cuidador primario.
- El modelo de visión por computadora debe entrenarse y validarse con datos representativos del contexto colombiano.
- La integración con la línea 123 requiere acuerdo formal con entidades de emergencias del Atlántico.

---

## 8. Fases de desarrollo

| Fase | Descripción | Entregables |
|------|-------------|-------------|
| 0 | Planificación | PRD, arquitectura, diseño de sistema, doc técnica, lista de tareas |
| 1 | Infraestructura | Firmware base, backend API, esquema DB, pipeline CI/CD |
| 2 | Estados vacíos + E2E | Flujos mínimos funcionales, pruebas extremo a extremo |
| 3 | Preentrega | Integración completa, pruebas de campo piloto |
| 4 | Iteración | Ajustes por feedback, escalado, nuevas funcionalidades |
