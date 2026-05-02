---
title: Confort Térmico
type: concepto
tags: [concepto, confort, disconfort, vivienda-social]
aliases: [confort termico, disconfort termico]
clases: [001, 005]
updated: 2026-05-02
---

# Confort Térmico

## Definición

Estado en el que una persona se siente térmicamente bien en un espacio: ni con frío, ni con calor. No es solo función de la temperatura del aire — incluye **temperatura radiante**, humedad, velocidad del aire, vestimenta y nivel de actividad.

El **disconfort térmico** es el complementario: se está pasando mal por causas térmicas.

## Ejes del "desempeño térmico" en este curso

El curso evalúa **dos ejes**:

1. **Consumo de energía** — iluminación, calefacción, aire acondicionado. Se mide en kWh, se traduce a costo y a emisiones.
2. **Confort térmico** — se mide en horas de disconfort, temperaturas máximas/promedio, índices de confort (PMV, PPD, modelos adaptativos).

## Por qué importa el énfasis en confort/disconfort

El grupo de investigación enfatiza el **disconfort térmico** porque es un **problema invisibilizado** en México:

- En vivienda social, el disconfort no se traduce fácil a emisiones (la gente no enciende aire acondicionado porque no lo tiene), entonces los ahorros energéticos no aparecen.
- Las métricas de sostenibilidad están dominadas por huella de carbono y eficiencia energética → el disconfort no se cuantifica → se pierde del radar.
- Una casa con buenas estrategias bioclimáticas puede usar **más material y más energía embebida** (huella ligeramente mayor) pero ofrecer **vida adecuada**. El eje social de la sostenibilidad pide considerar esto.

> Carla (del instituto) ha trabajado **índices de pobreza multidimensional** que incluyen confort térmico junto con injusticia energética.

## Contexto de la vivienda mexicana

- Las edificaciones gastan ~20% de la energía total en México.
- Solo ~20% de las viviendas usan aire acondicionado → la mayoría sufre disconfort sin que aparezca como consumo energético.
- Los espacios de México suelen depender de **ventilación natural** (paradigma distinto al de EE.UU., donde se sella y se ventila mecánicamente).
- Las normativas mexicanas relevantes son **NOM-008** y **NOM-020**, pero el profesor las considera deficientes ("no sirven").

## Cómo se cuantifica en simulación

Aunque las simulaciones del taller están simplificadas (sin ventilación natural, sin cargas internas), permiten comparar el **orden de magnitud del disconfort** entre estrategias bioclimáticas: el orden relativo se conserva incluso si los valores absolutos no son los reales.

Métricas típicas:

- **Temperatura interior** (máxima, mínima, promedio) — preferentemente [[Temperatura-Operativa]] sobre $T_{aire}$.
- **Horas fuera de confort** — % del año en que $T_{op}$ cae fuera de la banda.
- **Grados-hora de disconfort** — suma de las desviaciones temporales más allá de la banda. Penaliza más el disconfort severo.

## Modelos de evaluación

| Modelo | Premisa | Cuándo se usa |
|--------|---------|---------------|
| **PMV-PPD** (ASHRAE 55 estático, Fanger) | Zona de confort fija (~22-26 °C) | Edificios con HVAC, climas templados |
| **Adaptativo** (Humphreys-Nicol, ASHRAE 55 adaptativo) | Zona de confort = función del clima reciente | Edificios **naturalmente ventilados** — relevante para el taller |

El modelo **adaptativo** es el que aplica el grupo del IER y el taller, porque refleja la adaptación cultural y fisiológica al clima local — lo apropiado para evaluar diseño bioclimático sin AC. Detalle en [[Confort-Adaptativo]].

## Clases relacionadas

- [[../classes/001-IntroduccionTallerIDB]] — introducción al concepto y a su lugar en el curso
- [[../classes/005-AnalisisSimulacionesPython]] — modelo adaptativo Humphreys-Nicol y métricas (grados-hora) en Python
