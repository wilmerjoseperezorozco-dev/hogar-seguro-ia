# HogarSeguro IA

**Sistema inteligente de prevención de accidentes domésticos para Colombia**

HogarSeguro IA integra sensores multi-modal, visión por computadora y procesamiento en el borde (edge computing) para detectar situaciones de riesgo en el hogar —explosiones de gas, incendios en cocina, asfixia, electrocución— y activar respuestas automatizadas en segundos, antes de que el daño sea irreversible.

El sistema prioriza poblaciones vulnerables: niños menores de 5 años, adultos mayores de 60 y familias en estratos 1-3, donde el acceso a sistemas de seguridad comerciales es limitado y la mortalidad por accidentes domésticos prevenibles es desproporcionadamente alta.

---

## Problema

En Colombia, los accidentes domésticos representan una de las principales causas de muerte y discapacidad evitables. Las decisiones críticas ocurren en segundos; los sistemas de alerta existentes son reactivos, no predictivos, y están fuera del alcance económico de la mayoría de los hogares.

## Solución

Dispositivo de bajo costo con:
- Sensores de gas, humo, temperatura y audio
- Cámara de bajo consumo con inferencia local (no cloud)
- Cierre automático de válvula de gas
- Alertas a familiares y línea de emergencias 123
- App móvil con simulacros mensuales y contenido educativo de 1 minuto

## Modelo de negocio

Suscripción mensual (~15.000 COP) con posibilidad de cofinanciación pública a través del Ministerio de TIC y esquemas de descuento por parte de aseguradoras.

---

## Estructura del repositorio

```
hogar-seguro-ia/
├── docs/
│   ├── PRD.md                 # Documento de requisitos del producto
│   ├── arquitectura.md        # Arquitectura del sistema
│   ├── diseno_sistema.md      # Diseño técnico detallado
│   ├── doc_tecnica.md         # Documentación técnica de componentes
│   └── lista_tareas.md        # Plan de fases y tareas
├── README.md
└── .gitignore
```

## Estado actual

**Fase 0 — Planificación** (completada 2026-07-19)

> Fase 1 (infraestructura), Fase 2 (estados vacíos + E2E), Fase 3 (preentrega) y Fase 4 (iteración) pendientes.

---

## Investigador principal

Wilmer José Pérez Orozco  
Barranquilla, Colombia
