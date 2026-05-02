---
title: Tipos de Superficie
type: concepto
tags: [concepto, geometria, conveccion, openstudio, energyplus]
aliases: [surface type, tipos de superficie, muro techo piso]
clases: [003]
updated: 2026-05-02
---

# Tipos de Superficie

## Tres tipos en Energy Plus

Toda superficie de la envolvente cae en uno de tres tipos:

| Tipo | Color en Open Studio (Render by Surface Type) | Inclinación |
|------|-----------------------------------------------|-------------|
| **Muro** (`Wall`) | Amarillo | Vertical (o casi) |
| **Techo** (`Roof`) | Rojo | Horizontal o inclinado, mirando hacia arriba |
| **Piso** (`Floor`) | Gris | Horizontal, mirando hacia abajo |

Open Studio lo asigna automáticamente según la orientación de la normal del polígono.

## Por qué importa el tipo

> "Eso es muy importante. Un techo no tiene el mismo coeficiente convectivo que un piso."

El **coeficiente de convección** `h_c` depende del tipo de superficie porque:

- La **convección natural** depende de si el flujo de calor va a favor o en contra de la flotación. En un techo caliente, el aire calentado sube y se aleja → buen intercambio. En un piso caliente, el aire calentado también sube y se aleja → buen intercambio. En un techo **frío** o un piso **frío** la dinámica cambia. El sentido del flujo (calentamiento vs enfriamiento) y la orientación importan.
- La **inclinación** determina qué correlación experimental aplica E+: hay correlaciones distintas para superficies verticales, horizontales mirando arriba, horizontales mirando abajo, e inclinadas.
- La **rugosidad** del material modifica `h_c` (ya en el material, no en el tipo de superficie).
- La **velocidad del viento** afecta solo a las superficies exteriores.

E+ tiene un catálogo de algoritmos para `h_c`; el tipo de superficie selecciona el conjunto válido.

## Render by Surface Type vs Render by Boundary

En Open Studio el preview 3D tiene un selector **Render by**:

- **Surface Type** → colorea por tipo (amarillo/rojo/gris). Útil para verificar que techos y pisos están bien clasificados.
- **Boundary Conditions** → colorea por condición de frontera (outdoor, surface, ground, adiabática). Ver [[Condiciones-de-Frontera]].

Conviene alternar entre los dos modos al revisar un modelo.

## Sub-superficies (no son tipo de superficie)

Las **ventanas** y **puertas** son [[Subsuperficie|sub-superficies]] — viven dentro de una superficie de tipo Muro. No tienen un "tipo de superficie" propio en este sentido; tienen su propia categoría (window, door).

## Clases relacionadas

- [[../classes/003-MiPrimeraSimulacion]] — primera vez que aparece el render by surface type y la dependencia del coeficiente convectivo
