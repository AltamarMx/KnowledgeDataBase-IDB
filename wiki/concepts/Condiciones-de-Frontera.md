---
title: Condiciones de Frontera
type: concepto
tags: [concepto, fisica, frontera, boundary-conditions]
aliases: [condicion de frontera, boundary condition]
clases: [001, 002, 003, 004]
updated: 2026-05-02
---

# Condiciones de Frontera

## Definición

Especificación matemática de cómo interactúa una superficie con su entorno. Son **necesarias** para que el [[Balance-de-Calor]] sea un problema bien planteado: sin condiciones de frontera, las ecuaciones no tienen solución única.

## Tipos en transferencia de calor

| Tipo | Qué se especifica | Ejemplo |
|------|-------------------|---------|
| **Temperatura** (Dirichlet) | Temperatura impuesta en la superficie | Piso en contacto con suelo a una temperatura del ground |
| **Flujo de calor** (Neumann) | Flujo de calor a través de la superficie | Flujo constante, variable, o **cero** (caso especial: adiabática) |
| **Convectiva** (Robin / mixta) | Coeficiente de convección + temperatura del fluido | Superficie en contacto con aire exterior |

### Caso especial: condición adiabática

Una **condición adiabática** es una condición de flujo de calor con valor **cero**: por esa superficie no hay transferencia de calor. Equivale a "aislar" perfectamente esa cara del modelo.

## Uso en este curso

### Piso adiabático

El piso de las simulaciones del taller se modela como **adiabático**.

**Razón:** la temperatura del suelo (ground) depende de muchos factores — clima, tipo de material (no es lo mismo un pastizal que una zona volcánica), humedad, profundidad. Determinarla bien es "todo un arte" y se sale del alcance del curso. Se aborda en Energía en Edificaciones.

### Otras superficies

- **Muros y techo exteriores:** condición **mixta** — convección con el aire ambiente + intercambio radiativo de onda corta (radiación solar absorbida) + intercambio radiativo de [[Radiacion-Onda-Larga]] con ground/sky/air/surroundings. La forma combinada es:

  $$
  q''_{\alpha sol} + q''_{LWR} + q''_{conv} = -k \frac{\partial T}{\partial x}\bigg|_{x=0}
  $$

  Esta es **la** condición de frontera típica del exterior — ver detalle en [[Balance-de-Calor]].
- **Muros interiores entre zonas:** condición de frontera entre zonas (cada zona resuelve su balance y el muro es interfaz).
- **Vecinos / sombreamiento:** decisión de modelado — se pueden meter como geometría que sombrea, o como condición de frontera específica.

## Catálogo en Open Studio (Render by Boundary)

En el preview 3D de Open Studio el selector **Render By → Boundary Conditions** colorea cada superficie según su condición. Aprenderse los colores acelera el debug visual:

| Color | Condición OS | Significado físico |
|-------|---------------|---------------------|
| **Azul** | `Outdoors` | Expuesto al sol y al viento — radiación incidente, convección con aire ambiente, intercambio LWR con ground/sky/air/surroundings |
| **Verde** | `Surface` (interzona) | Frontera entre dos zonas térmicas — el calor que sale por una entra a la otra; la superficie ya **no recibe radiación incidente ni LWR** del exterior |
| **Café/marrón** | `Ground` | En contacto con el suelo — temperatura del ground del modelo |
| **Rojo** | `Adiabatic` | Flujo de calor cero — superficie aislada del modelo |

> Hay **diferentes tonos** dentro de cada categoría (variantes del azul, etc.) — corresponden a sub-tipos.

### Cómo se asigna en Open Studio

- Default al dibujar: muros y techo en `Outdoors`, piso en `Ground`.
- Cuando se **unen físicamente** dos espacios en FloorspaceJS (paredes tocándose, línea punteada al hacer merge), Open Studio convierte automáticamente Outdoor → Surface en el muro común.
- Si se dejan **separados por un margen** (1 cm, 10 cm), Open Studio **no detecta interzona** — los muros quedan como Outdoor y las dos zonas pierden el acoplamiento térmico. Truco a evitar.
- Para forzar `Adiabatic` (caso típico: piso del curso): pestaña Spaces → Surfaces → columna `Outside Boundary Condition` → cambiar a `Adiabatic`.

### Pisos adiabáticos en edificios multi-piso

En un edificio con varios pisos, los **pisos intermedios** pueden modelarse adiabáticos arriba y abajo cuando la temperatura es similar entre niveles (lo que sale de un piso entra al de arriba; los flujos se aniquilan). Solo el piso de planta baja toca el ground; solo el techo del último piso toca outdoors.

## Sun Exposure y Wind Exposure (Outdoors)

Para superficies con condición `Outdoors`, Open Studio expone dos columnas adicionales que actúan como **dimensiones independientes**:

| Columna | Valores | Efecto físico |
|---------|---------|---------------|
| **Sun Exposure** | `SunExposed` / `NoSun` | Si `NoSun` se desactiva la radiación de onda corta sobre esa superficie |
| **Wind Exposure** | `WindExposed` / `NoWind` | Si `NoWind` el coeficiente convectivo no usa la velocidad del viento del EPW (solo convección natural) |

### Combinaciones útiles

| Caso | Sun | Wind | Por qué |
|------|-----|------|---------|
| Muro o techo expuesto (default) | SunExposed | WindExposed | Comportamiento normal |
| **Estacionamiento subterráneo** (techo del estacionamiento, piso del edificio) | NoSun | WindExposed | Aire del estacionamiento sin sol pero con convección |
| Edificios muy pegados sin espacio | NoSun | NoWind | Cuando no se quiere modelar al vecino como geometría |
| Caverna o sótano técnico cerrado | NoSun | NoWind | Sin convección al ambiente |

> "Donde más se usa es en pisos que tienen estacionamiento subterráneo. No tengo cielo, no tengo exposición al sol, pero sí tengo convección. Es lo más común — un edificio con un estacionamiento, en lugar de simularlo como zona térmica."

Es una caricatura: en estos casos la temperatura del aire que toca la superficie se asume = T_amb del EPW. Es imperfecto pero pragmático cuando no se quiere modelar el espacio del estacionamiento como zona térmica adicional.

## Por qué importa la calidad de las condiciones de frontera

> "Si las condiciones de frontera están mal, el resultado es incierto."

Una crítica del profesor a interfaces como Design Builder es que **automatizan las condiciones de frontera sin que el usuario sea consciente**, lo que propicia simulaciones mal planteadas. Por eso el curso enfatiza entender qué condición se está aplicando y por qué.

## Clases relacionadas

- [[../classes/001-IntroduccionTallerIDB]] — introducción a los tipos y al uso de piso adiabático
- [[../classes/002-ConceptosBasicosBalancesCalor]] — la condición de frontera externa típica como combinación radiativa + convectiva
- [[../classes/003-MiPrimeraSimulacion]] — catálogo de colores Open Studio, conversión automática Outdoor→Surface al unir espacios, forzar piso adiabático
- [[../classes/004-InterpretandoMensajesConstructionSets]] — Sun y Wind Exposure como dimensiones independientes; caso del estacionamiento subterráneo
