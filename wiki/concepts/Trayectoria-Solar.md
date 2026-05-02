---
title: Trayectoria Solar
type: concepto
tags: [concepto, sol, radiacion, orientacion, validacion, lectura-de-datos]
aliases: [trayectoria solar, sun path, posicion solar, radiacion incidente]
clases: [007]
updated: 2026-05-02
---

# Trayectoria Solar

## Por qué importa para el análisis

La radiación solar incidente sobre una superficie depende de **la orientación y la inclinación** de esa superficie y de **la posición del sol** en el cielo en cada instante. Saber cómo se ve la curva esperada de radiación incidente para cada orientación permite:

- **Validar** una simulación (un patrón inesperado es pista de error en la geometría).
- **Diseñar protecciones solares** que cubran los ángulos críticos.
- **Justificar** decisiones de orientación de ventanas y zonas térmicas.

En el hemisferio norte, donde está México, las orientaciones tienen comportamientos muy distintos.

## Patrón esperado por orientación (hemisferio norte, fuera del trópico)

| Orientación | Radiación directa | Radiación difusa |
|-------------|-------------------|-------------------|
| **Norte** | **Nunca** la recibe (el sol está al sur) | Sí, todo el día |
| **Sur** | Sí, desde mañana hasta tarde, **pico al mediodía** | Sí |
| **Este** | Sí, mañana solamente; **nada después del mediodía** | Sí, todo el día |
| **Oeste** | Nada hasta el mediodía; sí, tarde solamente | Sí, todo el día |
| **Horizontal (techo)** | Sí, todo el día con sol; **pico al mediodía** | Sí |

### Caso especial — franja intertropical

Dentro del trópico de Cáncer (~23.5° N, donde está parte de México), el sol pasa **por el cenit** dos veces al año (alrededor del solsticio de verano). En esos días:

- La superficie **horizontal** recibe el máximo de radiación directa (sol cenital).
- La superficie **norte** recibe radiación directa unos días alrededor del solsticio de verano (porque el sol pasa al norte del cenit).

Es un comportamiento **inusual** que aparece solo en climas tropicales/subtropicales.

## Cómo leer una gráfica de radiación incidente

Si pides `Surface Outside Face Incident Solar Radiation Rate per Area` para varias superficies orientadas (norte, este, oeste, horizontal), las series temporales típicamente se ven así para un día con sol:

```
W/m²
    │
800 │           ╱╲           ← horizontal (techo)
    │          ╱  ╲
600 │     ╱╲  ╱    ╲   ╱╲    ← oeste (pico de tarde)
    │ ╱╲ ╱  ╲╱      ╲ ╱  ╲
400 │╱  ╲     ↑      ╲╱   ╲
    │    ╲  mediodía  ↓
200 │     ╲           ╱╲   ← este (pico de mañana)
    │_____________________
   06h    12h         18h
```

- **Norte**: una "joroba" baja y plana — solo difusa.
- **Este**: pico en la mañana, cae a difusa después del mediodía.
- **Oeste**: difusa en la mañana, pico de directa en la tarde.
- **Horizontal**: pico simétrico al mediodía, máximo absoluto.

## Ejemplo de validación — caso de la clase 007

> "Ahí se ve clarísimo cuál es la superficie oeste."

El profesor mira las series y deduce sin etiquetas:

- La que tiene un **pico al final del día** es la oeste.
- La que tiene un **pico al inicio** es la este.
- La que está **plana y baja** es la norte.
- La que tiene **el pico más alto al mediodía** es el techo (horizontal).

Si la simulación produjera un pico de tarde en la superficie etiquetada como "Norte", hay un error de orientación o de etiquetas — la trayectoria solar no miente.

## Implicaciones para el diseño bioclimático

### Ventanas norte/sur preferidas, este/oeste evitadas

> "Por eso no queremos ventanas este/oeste — queremos ventanas norte y sur. Las ventanas este/oeste son bien difíciles de proteger."

Razón física:

- **Ventanas sur** (hem. norte): el sol incidente viene **alto** al mediodía → un alero horizontal corto bloquea efectivamente el verano (sol más alto) y deja entrar el invierno (sol más bajo).
- **Ventanas norte**: solo difusa → bajo riesgo de sobrecalentamiento, no necesitan sombreamiento.
- **Ventanas este/oeste**: el sol viene **bajo y oblicuo** en mañanas/tardes → un alero horizontal **no las protege** (el sol entra por debajo). Se requieren parteluces verticales o estrategias adicionales.

Por esto las orientaciones E/W son los casos difíciles que el proyecto final suele tomar como objetivo de protección.

### Diseño de aleros — ángulos críticos

Para una ventana orientada al sur (hem. norte), el alero se diseña pensando en el **ángulo solar máximo** del verano:

```
                 │ ↑ sol verano (alto)
                 │
   ──────────────┤
                 │  alero
                 │
                 │ ↓ sol invierno (bajo)
       ventana   │
   ──────────────┤
```

Si el alero está al ras de la ventana, **la mitad del día no protege** (cuando el sol viene oblicuo desde el este o el oeste). Detalle en [[Superficies-de-Sombramiento]].

## En Energy Plus

E+ resuelve la trayectoria solar aparente como función de **lat/lon/timezone** del EPW + **fecha y hora**. Las variables `Site Solar Altitude Angle` y `Site Solar Azimuth Angle` reportan la posición instantánea del sol — útiles para depurar.

### Capa límite atmosférica

E+ ajusta la **temperatura del aire** según altura (capa límite — ver [[Capa-Limite-Atmosferica]]) pero **no** ajusta la radiación: la difusa medida a 10 m se asume válida a la altura de cada superficie. Despreciable en edificaciones de baja altura.

### Shadow update

E+ recalcula sombras **cada 20 días** por default (no cada paso temporal — ver [[Calculo-Sombramientos]]). La trayectoria solar se interpola entre actualizaciones. Aproximación razonable para análisis térmico.

## Variable proxy para radiación global horizontal

E+ no expone una variable directa "radiación global incidente" sobre el sitio. Truco del profesor: pedir `Surface Outside Face Incident Solar Radiation Rate per Area` sobre **el techo** (superficie horizontal sin sombreamiento). Sirve como referencia de la radiación máxima del día. Detalle en [[Variables-Output-EnergyPlus]].

## Clases relacionadas

- [[../classes/007-CasoBaseAleros]] — lectura de trayectoria solar a partir de radiación incidente sobre ventanas norte/oeste
