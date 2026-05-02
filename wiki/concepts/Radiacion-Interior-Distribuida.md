---
title: Radiación Interior Distribuida
type: concepto
tags: [concepto, radiacion, suposicion, energyplus, ventanas, iluminacion]
aliases: [interior solar distribution, distribucion de radiacion interior, FullInteriorAndExterior]
clases: [003, 007]
updated: 2026-05-02
---

# Radiación Interior Distribuida

## Suposición default de Energy Plus

Cuando una fuente de **radiación de onda corta** entra a una zona térmica — radiación solar a través de una ventana, o luz emitida por luminarias y proyectores — Energy Plus **no proyecta esa radiación sobre la superficie real** donde caería físicamente. La distribuye en las superficies interiores siguiendo dos reglas:

| Componente | Qué hace E+ por default |
|------------|--------------------------|
| **Radiación difusa** (solar difusa, luz dispersa) | Se reparte entre **todas las superficies interiores** ponderada por área. |
| **Radiación directa** (rayo de sol que entra por una ventana, haz de un proyector) | Se asume que **cae toda en el piso**. |

> "Si entra un rayito de luz, ahí se la va a asumir al piso. Lo mismo pasa con la luz visible: 900 W de un proyector se reparten en todas las superficies de ese cuarto."

## Por qué E+ hace esta caricatura

Calcular dónde cae cada rayo de sol al interior requiere **trazar la radiación** — para cada paso de tiempo, para cada ventana, para cada hora del año. Es caro computacionalmente. La distribución uniforme es una aproximación que mantiene la simulación rápida y conserva la **energía total** que entra al espacio.

## El modelo `FullInteriorAndExterior`

E+ ofrece una opción más cara que sí proyecta la radiación directa sobre la superficie real:

- **Algoritmo:** `FullInteriorAndExterior` (en el objeto `Building` o `BuildingSurface:Detailed`).
- Lo que hace: traza el rayo solar directo y lo deposita en la superficie real donde cae (no asume "todo al piso").
- **La radiación difusa sigue distribuida uniformemente** — incluso con este modelo. La difusa entra y se reparte; solo la directa se proyecta.

Limitaciones:

- Requiere que la geometría sea convexa (sin auto-sombreamiento interno fuerte).
- Más caro: cada paso de tiempo se recalcula la proyección.
- Sigue siendo una aproximación: se calcula proyección sobre el polígono, no sobre objetos al interior.

## Equipos que emiten radiación (proyectores, luces)

Para una luminaria o un proyector, E+ aplica una caricatura aún más simple:

- Se reparte la potencia entre componentes: ej. **90% luz visible + 10% calor convectivo** (porque el aparato se calienta).
- La **luz visible se distribuye en todas las superficies** del cuarto, sin importar a dónde apunte el aparato.

> "No le voy a decir 'es un proyector y estoy apuntando sobre esa pared'. No puedo. Si no se vuelve muy complejo resolver no solo todos los procesos sino la radiación."

## Implicaciones para diseño bioclimático

- Si una estrategia depende de que el sol entre y caliente **un muro masivo específico** (ej. muro Trombe con vidrio), el comportamiento real puede diferir del simulado salvo que se use `FullInteriorAndExterior` y geometría adecuada.
- Para evaluación **comparativa de estrategias** la suposición default suele ser suficiente.

## Efecto local sobre ocupantes — el rayo de luz

> "Si yo tengo un rayo de luz que ingresa, calienta todo el espacio en Energy Plus. Pero si ese rayo de luz me pega a mí, mi temperatura va a ser diferente."

La distribución uniforme **subestima el disconfort radiativo local** sobre un ocupante específico. Para análisis fino:

- Pedir [[Temperatura-Operativa]] en lugar de solo `Zone Mean Air Temperature`.
- Si se requiere localización en un punto específico, configurar **sensores virtuales de confort** (no expuesto en Open Studio — vía IDF).
- Para comparaciones agregadas (% del año en confort), la T operativa de zona es suficiente.

## Clases relacionadas

- [[../classes/003-MiPrimeraSimulacion]] — introducción a la suposición y al modelo `FullInteriorAndExterior`
- [[../classes/007-CasoBaseAleros]] — refuerzo: efecto local sobre ocupantes que la suposición no captura
