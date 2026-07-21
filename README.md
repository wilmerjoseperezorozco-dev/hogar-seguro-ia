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

---

## 🌐 Overview · Resumen

<table>
<tr>
<td width="50%">

### 🇬🇧 English

**Intelligent domestic accident prevention for Colombia** — edge AI, multi-modal sensors, automatic gas shut-off.

**What it solves:** In Colombia, domestic accidents (gas explosions, kitchen fires, asphyxiation) are a leading cause of preventable death and disability. Critical decisions happen in seconds. Existing alert systems are reactive and unaffordable for most households. This system detects hazards *before* they become irreversible — no cloud inference, no subscription to a data platform.

**At maturity:** A low-cost device (gas, smoke, temperature, audio sensors + low-power camera) runs local AI inference → detects the hazard → automatically closes the gas valve → sends alerts to family members and emergency line 123 → the mobile app delivers monthly drills and 1-minute safety education content.

Target population: children under 5, adults over 60, and families in strata 1–3 — where preventable accident mortality is disproportionately high and commercial safety systems are out of reach.

**Business model:** ~15,000 COP / month subscription. Co-funding pathway via Colombia's Ministry of ICT and insurer discount schemes.

**Quick start:**
```bash
# Phase 0 complete — planning documents available
cat docs/PRD.md            # product requirements
cat docs/arquitectura.md   # system architecture
cat docs/diseno_sistema.md # technical design
```

**Status:** Phase 0 complete (planning) · Phases 1–4 in roadmap: infrastructure → empty states + E2E → pre-release → iteration.

</td>
<td width="50%">

### 🇨🇴 Español

**Prevención inteligente de accidentes domésticos para Colombia** — IA en el borde (edge), sensores multimodal, cierre automático de gas.

**Qué resuelve:** En Colombia, los accidentes domésticos (explosiones de gas, incendios en cocina, asfixia) son una de las principales causas de muerte y discapacidad evitables. Las decisiones críticas ocurren en segundos. Los sistemas de alerta existentes son reactivos y están fuera del alcance económico de la mayoría de los hogares. Este sistema detecta los peligros *antes* de que sean irreversibles — sin inferencia en la nube, sin suscripción a una plataforma de datos.

**En fase madura:** Un dispositivo de bajo costo (sensores de gas, humo, temperatura, audio + cámara de bajo consumo) ejecuta inferencia local de IA → detecta el peligro → cierra automáticamente la válvula de gas → envía alertas a familiares y a la línea de emergencias 123 → la app móvil entrega simulacros mensuales y contenido educativo de 1 minuto.

Población objetivo: niños menores de 5 años, adultos mayores de 60 y familias en estratos 1–3 — donde la mortalidad por accidentes prevenibles es desproporcionadamente alta y los sistemas comerciales de seguridad están fuera de alcance.

**Modelo de negocio:** ~15.000 COP / mes de suscripción. Vía de cofinanciación a través del Ministerio de TIC y esquemas de descuento de aseguradoras.

**Inicio rápido:**
```bash
# Fase 0 completa — documentos de planificación disponibles
cat docs/PRD.md            # requisitos del producto
cat docs/arquitectura.md   # arquitectura del sistema
cat docs/diseno_sistema.md # diseño técnico
```

**Estado:** Fase 0 completa (planificación) · Fases 1–4 en roadmap: infraestructura → estados vacíos + E2E → preentrega → iteración.

</td>
</tr>
</table>
