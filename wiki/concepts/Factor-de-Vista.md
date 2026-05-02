---
title: Factor de Vista
type: concepto
tags: [concepto, radiacion, geometria, view-factor]
aliases: [factor de forma, view factor, factor-vista]
clases: [002, 003]
updated: 2026-05-02
---

# Factor de Vista

## Definición

Fracción de la radiación que sale de una superficie A y llega directamente a una superficie B. Se denota $F_{A \to B}$.

> "La fracción que recibe una cara respecto a la que emite [otra]."

Es una propiedad puramente **geométrica** — depende de las posiciones, orientaciones y tamaños relativos, no de las temperaturas o materiales.

## Propiedades fundamentales

- **Reciprocidad:** $A_A \cdot F_{A \to B} = A_B \cdot F_{B \to A}$. (Y si las áreas son iguales, $F_{A \to B} = F_{B \to A}$.)
- **Suma:** la suma de los factores de vista desde A a todas las superficies que la rodean es 1: $\sum_i F_{A \to i} = 1$.
- **Superficie plana consigo misma:** $F_{A \to A} = 0$ (una superficie plana no puede "verse a sí misma").
- **Superficie cóncava consigo misma:** $F_{A \to A} > 0$ (una superficie curva sí puede verse a sí misma).

## Por qué Energy Plus solo permite líneas rectas

Energy Plus **asume $F_{s \to s} = 0$ siempre**. Esto solo es físicamente correcto para superficies **planas**. Permitir superficies curvas implicaría:

- Tratar el auto-intercambio radiativo de cada superficie consigo misma.
- Resolver geometrías en coordenadas cilíndricas (o más complejas).
- Mucha más complejidad de cómputo.

Por eso Energy Plus **no permite superficies curvas, líneas curvas, ni ventanas circulares** — solo polígonos planos.

## Uso en el balance de calor

El [[Balance-de-Calor]] en la superficie exterior incluye intercambio radiativo de [[Radiacion-Onda-Larga]] con cuatro fuentes (ground, sky, air, surroundings). Cada término contiene un factor de vista:

$$
q''_{s \to i} = \varepsilon \, \sigma \, F_{s \to i} \, (T_i^4 - T_s^4)
$$

### Factores de vista para superficies del exterior

Algunos casos típicos que Energy Plus calcula automáticamente:

| Superficie | $F$ con cielo | $F$ con suelo |
|------------|---------------|---------------|
| Techo horizontal | 1 | 0 |
| Muro vertical | 0.5 | 0.5 |
| Muro inclinado | entre 0 y 1 | entre 0 y 1 |

(Aproximaciones cuando suelo y cielo se asumen infinitos — entonces solo importa la inclinación de la superficie analizada.)

## Para superficies interiores

Dentro de una zona térmica, también hay intercambio radiativo de onda larga entre superficies interiores. Se requieren los factores de vista entre cada par de superficies internas. Energy Plus los calcula a partir de la geometría.

> **Nota:** la radiación de onda larga **no atraviesa el vidrio**. Aunque haya ventanas en un cuarto, el balance LWR interior se cierra solo entre las superficies internas (muros, piso, techo). Una superficie ya no "ve" el cielo por LWR si tiene una ventana en el muro frente a ella.

### Factores de vista nulos por geometría

En geometrías reales muchas superficies **no se ven entre sí**:

- Superficies **paralelas** que están detrás una de la otra (ej. dos muros opuestos en un cuarto cuando se evalúa entre superficies del mismo lado): $F = 0$.
- Cualquier obstrucción geométrica entre dos superficies anula su factor de vista.

Por eso E+ resuelve un sistema con factores de vista entre **cada par válido** de superficies internas.

## Clases relacionadas

- [[../classes/002-ConceptosBasicosBalancesCalor]] — introducción al concepto y su rol en la restricción de líneas rectas
- [[../classes/003-MiPrimeraSimulacion]] — el balance LWR interior solo entre superficies del cuarto, no atraviesa vidrios
