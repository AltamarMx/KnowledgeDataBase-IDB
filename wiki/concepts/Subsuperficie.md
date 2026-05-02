---
title: Sub-superficie
type: concepto
tags: [concepto, ventanas, puertas, geometria, energyplus]
aliases: [subsurface, sub-superficie, ventana, puerta]
clases: [003, 006]
updated: 2026-05-02
---

# Sub-superficie

## Qué es

Una **sub-superficie** (`SubSurface` en E+) es una abertura **contenida en una superficie**: ventanas, puertas, lucernarios. No es una superficie independiente — está siempre **anidada** en una superficie padre.

## Reglas de modelado

- Una sub-superficie **debe vivir en una superficie** existente. No se puede colocar "flotando".
- Una ventana siempre necesita un **muro padre**. Si en la realidad no hay muro (un espacio abierto, una pared totalmente acristalada), igual hay que **crear el muro virtual** y luego poner una ventana muy grande dentro.
- La sub-superficie **no puede ocupar el 100%** de su superficie padre. Hay un margen mínimo. Una "ventana del 100%" se modela como una ventana del ~95-98% para satisfacer la restricción.

## Por qué la jerarquía superficie → sub-superficie

E+ resuelve el balance térmico **por superficie**. Las sub-superficies modifican el balance de su superficie padre (la ventana sustrae área opaca, agrega un camino con propiedades ópticas y térmicas distintas), pero el contador es la superficie padre.

Esto permite por ejemplo:

- Una ventana puede tener un **shading device** asociado (persiana, alero) que la sombrea.
- Las propiedades térmicas (U, SHGC) se definen como un **construction de ventana** independiente del muro.
- Las puertas se tratan igual aunque sean opacas.

## Caso típico — cafetería sin muro

En el ejemplo del instituto: la cafetería actual tiene una pared **abierta**. Para modelarla:

1. Crear un muro en esa cara (geometría virtual).
2. Insertar una ventana que cubra ~95% del muro (representa la apertura).
3. Asignar a la ventana un construction con propiedades cercanas a "aire libre" o controlar el flujo con otros mecanismos.

Es una caricatura — ver [[Caricatura-Computacional]].

## Window-to-Wall Ratio (WWR)

Para sub-superficies tipo ventana, una métrica clave:

$$
WWR = \frac{A_{ventana}}{A_{muro}}
$$

Valores normativos en México (NOM-008):

| Tipo | WWR máximo |
|------|------------|
| Vivienda | 20% |
| Comercial | 25% |

Detalle del concepto y materiales de ventana en [[Ventanas]]. Procedimiento de inserción en [[../procedures/Agregar-Ventanas-OpenStudio]].

## Clases relacionadas

- [[../classes/003-MiPrimeraSimulacion]] — introducción a la jerarquía superficie/sub-superficie
- [[../classes/006-DosZonasTermicasVentanasAleros]] — caso completo de ventanas con WWR, materiales y aleros
