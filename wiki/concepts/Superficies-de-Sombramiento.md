---
title: Superficies de Sombramiento (Aleros y Parteluces)
type: concepto
tags: [concepto, sombreamiento, aleros, parteluces, fins, overhangs, celosias, energyplus]
aliases: [aleros, overhangs, fins, parteluces, sombramiento, shading surfaces, celosias]
clases: [006]
updated: 2026-05-02
---

# Superficies de Sombramiento (Aleros y Parteluces)

## Qué son

Superficies opacas que se colocan **fuera de la envolvente térmica** — no encierran volumen ni separan zonas — cuyo único propósito es **bloquear radiación solar** sobre superficies de la edificación (típicamente ventanas).

| Tipo | Geometría | Bloquea sobre todo |
|------|-----------|---------------------|
| **Alero / Overhang** | Plano horizontal arriba de la ventana | Sol alto (verano, mediodía, fachadas sur en hemisferio norte) |
| **Parteluz / Fin** | Plano vertical a un lado de la ventana | Sol bajo y oblicuo (mañanas y tardes, fachadas este y oeste) |
| **Celosía** | Conjunto de aleros pequeños en rejilla | Combina ambos efectos con buen paso de aire |

## Qué SÍ y qué NO hacen en E+

> Aleros, parteluces y otras superficies de sombramiento **no participan en la transferencia de calor**. Solo bloquean radiación solar. Son una caricatura de su efecto físico.

| Mecanismo | ¿Lo hace E+? |
|-----------|--------------|
| Bloquear radiación solar **directa** sobre la superficie sombreada | **Sí** |
| Bloquear radiación solar **difusa** | **Sí** (parcial, según factor de vista al cielo) |
| Bloquear/atenuar el **viento** sobre la superficie | **No** — E+ no resuelve mecánica de fluidos externa |
| Tener una **temperatura** propia (calentarse al sol y reradiar) | **No** — los aleros no tienen masa térmica ni T en el modelo |
| **Conducir** calor hacia el muro al que están adosados | **No** |
| **Intercambio LWR** con superficies de la edificación | Despreciado por simplicidad |
| **Reflejar** radiación solar (por su reflectancia) | **Sí** — la radiación reflejada puede iluminar otras superficies |

> "Los aleros son superficies opacas que sí tienen reflectancia o absortancia. Pero la absortancia me gusta pensar más en reflectancia, porque la absortancia gana temperatura, pero éste no — solo va a reflejar."

### Lo que esto implica

Si un alero está al sol y se calienta, en la **realidad** irradia LWR hacia las superficies cercanas y conduce calor por contacto al muro adosado. **E+ ignora ambos efectos**. La caricatura es buena cuando el alero es delgado y bien ventilado; mala cuando es una losa gruesa de concreto en contacto con el muro.

## Projection Factor

Cuando se generan aleros desde el componente window de FloorspaceJS, el parámetro es el **Projection Factor**:

$$
PF = \frac{\text{longitud del alero (proyección horizontal)}}{\text{altura de la ventana}}
$$

| PF | Geometría |
|----|-----------|
| 0 | Sin alero |
| 0.5 | Alero la mitad de la altura de la ventana |
| 1.0 | Alero igual a la altura de la ventana |
| 2.0 | Alero el doble de la altura |

Análogo para parteluces verticales (`Fin Projection Factor`).

## Limitación crítica de Open Studio

> El alero generado por FloorspaceJS tiene **el mismo ancho que la ventana** — no se puede extender lateralmente desde la GUI.

Esto es **terrible** porque el sol no siempre viene perpendicular al muro:

- En el sur (hemisferio norte), el sol al mediodía se proyecta sobre la línea media del alero — bien.
- Pero a las 9 AM o 3 PM, el sol viene oblicuo y la sombra se proyecta lateralmente — fuera del ancho del alero. **El alero no protege.**

### Ángulos importantes en el diseño

Al diseñar un alero sobre una ventana, los ángulos relevantes son:

- **Ángulo desde el centro de la ventana al borde del alero** (en el plano vertical) — controla cuánta altitud solar bloquea.
- **Ángulo desde el borde de la ventana al borde lateral del alero** (en planta) — controla qué acimuts solares cubre.

Un alero efectivo se extiende **lateralmente más allá del ancho de la ventana** para cubrir ambos ángulos. Open Studio no permite esto desde GUI.

### Workaround

Editar el OSM directamente (texto plano):

1. Identificar el `Face N` del alero en el preview 3D de Open Studio.
2. Abrir el OSM con un editor (Notepad++, VS Code).
3. `Ctrl+F` por `Face <N>`.
4. Modificar las coordenadas de los 4 vértices del polígono para extender el alero.
5. Recargar el OSM en Open Studio y verificar visualmente.

Alternativa: pedir a una **IA** (Claude/ChatGPT) que aplique la transformación geométrica sobre las coordenadas del polígono. El profesor lo ha hecho — funciona la mitad de las veces.

Procedimiento detallado: [[../procedures/Agregar-Aleros-OpenStudio]].

## Aleros equivalentes — celosías

Una **celosía** (rejilla horizontal de listones) bloquea radiación tanto como un alero **mucho más grande** mientras se conserve el ángulo crítico.

Esquema:

```
Ventana:               Alero único:           Celosía equivalente:
                       ___________            ___
|====================|/         /             ___
|                    |          /             ___
|                    |          /             ___
|                    |          /             ___
                                              ___
```

Si la celosía tiene listones espaciados verticalmente con espesor `h` cada uno, y el ángulo formado entre listón y muro es 45°, **una celosía con `n` listones bloquea como un alero único de longitud `n·h`**.

### Por qué las celosías son una buena estrategia

- **Bloquean radiación solar** directa y difusa con la misma efectividad que un alero gigante.
- **No obstruyen el paso de aire** (entre listones) → permiten ventilación natural.
- Se acompañan bien de **parteluces verticales** para cubrir ángulos azimutales bajos.
- Estética arquitectónica reconocida — patrón común en climas cálidos.

### Caso del paper de la cafetería del IER

El grupo modeló la cafetería del IER (con celosía + sistema evaporativo) y validó contra mediciones. La celosía se modela en E+ como un **alero equivalente** de gran tamaño — preserva los ángulos sin tener que dibujar cada listón individual. El paper acaba de salir; lo dirige el grupo de Miriam.

## Reflectancia de la superficie de sombramiento

El material del alero importa porque parte de la radiación se **refleja**, y esa radiación reflejada puede **caer en la ventana** que se intentaba sombrear. Estrategias:

- **Aleros con perfil reflectante hacia afuera** (no hacia abajo/adentro) — desvían la radiación al exterior.
- **Análogo en iluminación natural**: light shelves con perfil que redirige la luz hacia el techo (donde se difunde) en lugar de hacia el plano de trabajo.
- Aplicación bioclimática: combinar la geometría del alero con un material reflectivo orientado al exterior.

## Vecinos y geometrías de sombramiento manuales

Además de aleros adosados a ventanas, en E+ se pueden agregar **superficies de sombramiento independientes**:

- Edificios vecinos.
- Vegetación.
- Muros virtuales para representar obstrucciones.

Mismo comportamiento físico: bloquean radiación, no transfieren calor, no obstruyen viento. Son la única forma de modelar el efecto sombreador de un vecino sin convertirlo en una zona térmica adicional.

## Estrategias del proyecto final

> Para el proyecto final el profesor recomienda **alero horizontal + parteluz vertical** combinados.

Patrón de diseño bioclimático:

| Orientación | Estrategia primaria |
|-------------|----------------------|
| **Sur** (hem. norte) | Alero horizontal — bloquea sol alto del verano, deja entrar sol bajo del invierno |
| **Norte** | Sombreamiento mínimo (sol siempre bajo) |
| **Este / Oeste** | Parteluces verticales — el sol viene bajo y oblicuo en mañanas/tardes |

Las orientaciones E/W son las más complicadas — un alero horizontal solo no las protege.

## Clases relacionadas

- [[../classes/006-DosZonasTermicasVentanasAleros]] — introducción al objeto, Projection Factor, limitación de Open Studio, aleros equivalentes y celosías
