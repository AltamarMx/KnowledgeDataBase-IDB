---
title: Capa Límite Atmosférica
type: concepto
tags: [concepto, energyplus, clima, capa-limite, conveccion]
aliases: [boundary layer, capa limite, atmospheric boundary layer]
clases: [005]
updated: 2026-05-02
---

# Capa Límite Atmosférica

## El problema

Las **estaciones meteorológicas** miden la temperatura del aire típicamente a **10 m sobre el suelo**. Pero una edificación tiene fachadas, ventanas y zonas térmicas a **alturas variables**: 0, 3, 6, 9 m, etc. La temperatura del aire **no es la misma a todas las alturas** — varía siguiendo el perfil de la capa límite atmosférica.

Si E+ usara directamente la T del EPW (a 10 m) para todos los cálculos convectivos, introduciría un error sistemático en edificaciones bajas o muy altas.

## Cómo lo resuelve E+

E+ aplica un **perfil de capa límite** para ajustar la T del aire a la altura del **centroide de la zona térmica** (o de la superficie):

$$
T_{aire}(z) = T_{met} \cdot \left( \frac{z}{z_{met}} \right)^{\alpha}
$$

(forma estándar; los parámetros $\alpha$ y rugosidad terrenos varían por tipo de entorno: urbano, suburbano, rural).

E+ aplica este ajuste por default al calcular el coeficiente convectivo exterior y la condición de frontera de Outdoors. El usuario rara vez lo manipula.

## Implicación para variables de output

Por eso hay variables paralelas con prefijos distintos:

| Variable | Significado |
|----------|-------------|
| `Site Outdoor Air Drybulb Temperature` | T del EPW **sin ajustar** (a la altura de la estación) |
| `Outdoor Air Drybulb Temperature` (a nivel de superficie/zona) | T ajustada a la altura del objeto |

Para análisis bioclimático general (graficar el clima del año) se usa la `Site:*`. Para auditar el cálculo convectivo específico de una superficie alta o baja, se inspecciona la versión ajustada.

## Por qué importa para edificaciones altas

En un rascacielos los pisos superiores experimentan una T del aire **distinta** (típicamente más fresca y con más viento) que los pisos inferiores. Sin este ajuste, las cargas térmicas calculadas para los pisos altos serían incorrectas.

En el alcance del curso (cubos a baja altura) la diferencia es marginal — pero el concepto justifica por qué hay variables `Site:*` y variables ajustadas.

## Para iluminación natural / radiación

La **radiación solar** del EPW también está medida a una altura específica (normalmente a nivel del piso, en plano horizontal). E+ no aplica un perfil de altura para radiación porque la atenuación atmosférica adicional en los primeros 50-100 m es despreciable. La radiación se proyecta sobre cada superficie usando la trayectoria solar aparente.

## Clases relacionadas

- [[../classes/005-AnalisisSimulacionesPython]] — primera mención al recorrer el RDD y notar las variantes Site / Outdoor
