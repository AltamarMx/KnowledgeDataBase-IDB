---
title: Caso Base
type: concepto
tags: [concepto, metodologia, caso-base, estudio-parametrico, proyecto-final]
aliases: [caso de referencia, caso base, baseline, modelo de referencia]
clases: [007, 012]
updated: 2026-05-08
---

# Caso Base

## Qué es

Modelo de simulación **sin estrategias bioclimáticas aplicadas** que sirve de **referencia** para evaluar el impacto de cada estrategia. Todas las variantes se construyen como **modificaciones del caso base** sobre una sola dimensión a la vez (color, orientación, masa, sombreamiento).

Comparar `Variante` vs `Caso Base` permite cuantificar el efecto **aislado** de la estrategia probada.

## Por qué importa que esté completamente revisado antes de ramificar

> "Si hacen un cambio de última hora a su modelo en una simulación, y ese cambio está en el caso base, lo van a tener que pasar a todas las variantes. Por eso es súper importante que cuiden que su caso base esté súper bien revisado."

Ramificar el caso base genera **N copias** que comparten todo lo que estaba en el momento de ramificar. Cualquier corrección posterior al caso base **no se propaga automáticamente** — hay que aplicarla manualmente a cada variante. En un proyecto con 5 variantes, una corrección sencilla se vuelve **5 ediciones repetidas + 5 simulaciones** + revisión.

## Checklist antes de ramificar

Antes de hacer la primera variante, el caso base debe tener:

| Aspecto | Confirmar |
|---------|-----------|
| **Geometría** | Render By Boundary correcto (Outdoor/Surface/Ground/Adiabatic). Render By Surface Type correcto. Sin avisos de geometría en `.err`. |
| **Sistemas constructivos** | Materiales con propiedades correctas (`iertools` permite auditarlas — ver [[../tools/iertools]]). Construction Set asignado a la edificación. Slots cubiertos para todas las combinaciones tipo+condición presentes. |
| **Zonas térmicas** | Cada espacio mapeado a una zona térmica. Nombres descriptivos sin acentos/eñes/espacios. |
| **EPW** | Asignado, con lat/lon/timezone correctos. |
| **Sub-superficies** | Ventanas con material y construction asignados. Construction Set tiene slot Sub Surface. |
| **Output Variables** | Las variables que vas a analizar **ya configuradas como measures**. Confirmadas en el RDD post-Run. |
| **Sanity check** | Una primera simulación corre limpia (cero severes, warnings entendidos). Las series temporales tienen valores plausibles. |

> "Definan métricas y output variables **antes** de ramificar — si no, cada variante va a quedar con un set distinto."

## Caso base del proyecto final 2026-2

Especificación fija dada por el profesor en la clase 012 — todos los equipos parten del mismo caso base, sólo cambia el bioclima asignado:

| Aspecto | Valor |
|---|---|
| Edificación | **Casa 11** del programa **Decide y Construye** (vivienda social MX, 60-65 m², dos plantas) |
| Absortancia solar (todas las superficies) | **0.4** |
| Sombreado | Sin elementos |
| Aire acondicionado | Sin AC |
| Cargas térmicas internas | Sin cargas |
| Piso | **Adiabático** |
| Infiltración | Sí (con la configuración que entregue el profesor) |
| Ventanas | Vidrio simple **3 mm**, dimensiones según planos |
| Sub-superficies interiores | No simular (bug de FloorspaceJS entre zonas) |
| Muros exteriores | Yeso 5 cm + tabique 14 cm + acabado interior (interior→exterior según plano) |
| Ventilación natural | No se modela |
| Cochera | Orientada al sur (define orientación de referencia; la rotación es estrategia válida) |

Las propiedades térmicas de los materiales las busca cada equipo y **reportan la fuente** (ASHRAE, Incropera, libros de transferencia de calor, [[../tools/EnerHabitat|EnerHabitat]]).

Detalle del encuadre completo en [[../classes/012-ProyectoFinal]].

## Estructura típica del proyecto final

> Para el proyecto final del taller: **caso base + 3 estrategias + caso combinado = 5 simulaciones**.

Las estrategias se eligen individualmente por equipo. Ejemplos:

| OSM | Contenido |
|-----|-----------|
| `005_caso_base.osm` | Modelo de referencia, sin estrategias |
| `006_estrategia_color.osm` | Caso base + cambio de color (absortancia) |
| `007_estrategia_aleros.osm` | Caso base + aleros |
| `008_estrategia_aislamiento.osm` | Caso base + aislamiento térmico |
| `009_combinado.osm` | Caso base + las 3 estrategias |

Análisis: comparar **cada variante vs caso base** para cuantificar la mejora aislada; comparar **combinado vs caso base** para evaluar sinergia.

## Cómo crear el caso base congelado

Cuando el modelo de desarrollo está estable y revisado:

1. `File → Save As` con un nombre descriptivo: `005_caso_base.osm`.
2. Renombrar también la zona térmica/folder hermano si conviene.
3. Correr una vez limpio.
4. Auditar las constructions con `iertools.read_sql(...).get_constructions(...)`.
5. **Etiquetarlo como congelado** (mentalmente o en el README del proyecto). De aquí en adelante: solo se duplica con `Save As` para crear variantes.

## Cómo NO copiar el caso base

> Una trampa común: copiar el OSM en el Explorador (Finder/File Explorer) con `Ctrl+C/V`.

Resultado: solo se duplica el `.osm` — **el folder hermano no se copia**, y con él se pierden los measures asociados. Detalle en [[../procedures/Estructura-Proyecto-Simulacion]].

**Vía correcta**: abrir Open Studio con el caso base → `File → Save As → 006_estrategia_X.osm`. Esto duplica el OSM **y** crea su folder hermano consistente.

## Comparación caso base vs variante en Python

```python
from iertools.read import read_sql

base    = read_sql("../OSM/005_caso_base/run/eplusout.sql", alias=True).data
alero   = read_sql("../OSM/007_estrategia_aleros/run/eplusout.sql", alias=True).data

# Ambas tienen las mismas columnas — comparación directa
fig, ax = plt.subplots(2, 1, sharex=True, figsize=(12, 4))
ax[0].plot(base.T_este,  label="base",  color="red", linestyle="-")
ax[0].plot(alero.T_este, label="alero", color="red", linestyle="--")
ax[0].legend()
```

Detalle del flujo en [[../procedures/Comparar-Simulaciones-Python]].

## Cuándo es válido cambiar el caso base

Solo cuando se descubre un **error de modelado** (un material mal escrito, una orientación equivocada, una superficie faltante). Si se cambia, hay que **propagar el cambio a todas las variantes** y volver a correr. Cualquier cambio "porque sí" después de ramificar es un anti-patrón — sugiere que el caso base no estaba realmente listo.

## Métricas relativas vs absolutas

El caso base es la base para reportar resultados como **diferencia relativa**:

| Métrica | Caso Base | Variante | Δ |
|---------|-----------|----------|---|
| Grados-hora cálidos al año | 8500 | 6200 | **−27%** |
| % del año en confort | 38% | 51% | **+13 pp** |

Las **diferencias relativas** son más robustas que los valores absolutos — la simulación es una caricatura ([[Caricatura-Computacional]]) que distorsiona absolutos pero conserva el orden y magnitud de los efectos.

## Clases relacionadas

- [[../classes/007-CasoBaseAleros]] — introducción al concepto y al workflow del proyecto final
- [[../classes/012-ProyectoFinal]] — especificación fija del caso base 2026-2 (Casa 11, α=0.4, sin AC, piso adiabático)
