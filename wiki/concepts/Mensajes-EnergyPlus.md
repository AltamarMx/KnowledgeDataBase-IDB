---
title: Mensajes de Energy Plus (Errores y Warnings)
type: concepto
tags: [concepto, energyplus, debugging, errores, warnings]
aliases: [errores energyplus, warnings energyplus, archivo err, eplusout.err]
clases: [004, 005, 006, 007, 008]
updated: 2026-05-02
---

# Mensajes de Energy Plus (Errores y Warnings)

## El archivo `.err`

Cuando E+ ejecuta una simulación genera un archivo de mensajes `eplusout.err` (o equivalente con prefijo distinto en Open Studio). Es el **primer lugar** que hay que revisar tras correr — antes incluso de mirar resultados.

En Open Studio: botón **Show Simulation** → abre la carpeta de outputs → abrir el `.err` con un editor de texto plano (Notepad/TextEdit). En Mac/Windows preguntará con qué programa abrir un archivo desconocido — elegir el editor de texto.

### Archivos hermanos generados en la misma carpeta

| Archivo | Para qué |
|---------|----------|
| `eplusout.err` | Mensajes de la simulación (este concepto) |
| `eplusout.rdd` | **Diccionario de variables** disponibles para reportar — ver [[RDD-Variables-Disponibles]] |
| `eplusout.mdd` | Diccionario de medidores (consumos por uso final) |
| `eplusout.sql` | Resultados en base SQL — ver [[Salida-SQL-EnergyPlus]] |
| `eplusout.csv` | Resultados en CSV (si se solicitó) |
| `eplustbl.htm` | Reporte HTML pre-formateado |
| `eplusout.eio` | Informe de inicialización con resúmenes de la edificación |

## Niveles de severidad

| Nivel | Comportamiento | Acción del usuario |
|-------|----------------|---------------------|
| **Severe** / **Fatal** | E+ **detiene** la simulación. No produce resultados utilizables. | Obligatorio corregir antes de continuar. |
| **Warning** | E+ **continúa** la simulación, generalmente aplicando un valor por default o una suposición. | El usuario debe **decidir** si la suposición es aceptable para su caso. |
| **Info** / **Note** | Información general (ej. "EPW no tiene design days") | Usualmente ignorable. |

> "Yo como experto en simulaciones debo tener la certeza si esos warnings me permiten seguir mi simulación o no."

El curso enfatiza esa distinción: un warning **no** es señal automática de que algo está mal, pero **sí** es señal de que hay que entender qué está suponiendo E+.

## Errores severos típicos

### Outside layer not found

Mensaje típico:

```
** Severe ** Construction <name>: missing material assignments / outside layer not found
```

Causa: una construction quedó **sin materiales asignados** o con la lista incompleta. E+ no puede calcular conducción si no sabe con qué material la haría.

Fix: pestaña **Constructions** → abrir la construction afectada → arrastrar material(es) desde `My Model` en orden ext→int.

### EPW missing / no weather file

Mensaje típico:

```
** Severe ** GetEnvironmentList: No design environments specified / no weather file found
```

Causa: el path al EPW se perdió. Pasa típicamente cuando se mueve el OSM dentro del folder hermano (ver [[../procedures/Estructura-Proyecto-Simulacion]]) o se mueve el folder del proyecto sin rehacer el `Set Weather File`.

Fix: pestaña **Site → Set Weather File** → re-seleccionar el `.epw`.

### Geometría inválida

Polígonos no cerrados, vértices duplicados, normales mal orientadas. Suele ser más complicado que volver a dibujar la geometría desde cero — "a veces es más fácil empezar de nuevo que tratar de arreglar la geometría".

## Warnings comunes ignorables (en el alcance del curso)

Open Studio está pensado para análisis de **consumo energético** y **ciclo de vida** según ASHRAE. Por eso emite warnings cuando faltan inputs típicos de ese flujo, aunque para análisis bioclimático (térmico puro) sean irrelevantes:

| Warning | Por qué aparece | Por qué se puede ignorar |
|---------|-----------------|---------------------------|
| Falta de **design days** en el EPW | Open Studio espera días de diseño para dimensionar HVAC | El curso no dimensiona HVAC |
| `Many overlapping shadows` | Geometría con muchos elementos sombreadores (aleros, parteluces, vecinos) que se traslapan | El algoritmo de overlapping los resuelve correctamente — ignorable salvo si la simulación es muy lenta. Ver [[Algoritmo-Sombreamiento]] |
| **Output variables faltantes** para Lifecycle Assessment | Esperaba consumo por uso final, fuentes de energía, etc. | El curso no hace LCA |
| Site/Source factors no especificados | Esperaba factores de conversión sitio→fuente | El curso no calcula consumo neto |
| Falta de **schedules** para cargas internas | Espera ocupación, iluminación, equipos | El curso no modela cargas internas |
| **Coliniar vertices** | E+ detectó vértices alineados redundantes en un polígono y los eliminó | Inocuo — la geometría sigue siendo correcta |
| **Weather location difference** | Pequeñas diferencias (~0) en lat/lon entre OSM e IDF tras la traducción | Despreciable cuando la diferencia es minúscula |

> **Política del grupo IER en investigación**: se eliminan **todos** los warnings (incluso los inocuos). En el taller se vive con los del catálogo conocido si están entendidos.

### Analogía con compiladores C

Los warnings se parecen a los de un compilador de C: hay warnings **inocuos** (un cast implícito que no introduce error) y **peligrosos** (usar un float como int — el redondeo depende del compilador). Cuando se ignora un warning hay que **saber por qué** es inocuo en el caso. Si no se sabe, no se ignora.

## Bugs recurrentes de Open Studio / FloorspaceJS

### Piso adiabático que se revierte a Ground

**Síntoma**: tras un cambio geométrico en FloorspaceJS, al hacer Run aparece severe `Construction missing material assignments` en el piso.

**Causa**: el cambio geométrico hace que Open Studio "rehaga" el modelo internamente. La condición de frontera del piso se revierte de `Adiabatic` a `Ground`, y como el slot `Ground Contact → Floor` del Construction Set suele estar vacío en modelos del taller, el piso queda sin construction.

**Workaround**: sobre-definir el slot `Ground Contact → Floor` en el Construction Set con un sistema constructivo válido (ej. `Concreto_15cm`).

**Riesgo silencioso**: con el slot lleno, la simulación corre sin severe — pero si la condición acabó como `Ground` (no `Adiabatic`), E+ usa T del ground por default (~18 °C) → resultado **incorrecto sin error visible**. Por eso siempre revisar que el piso sigue en `Adiabatic` tras cambios.

### Nombres custom de superficies borrados tras cambios geométricos

**Síntoma**: una variable solicitada por nombre específico (`Techo`, `vNorte`) aparece **vacía** o ausente en el SQL post-simulación.

**Causa**: cambios geométricos pueden revertir nombres custom (`Techo` → `Face 9`). Los measures de Add Output Variable que apuntaban al nombre original ya no encuentran la superficie.

**Detección**: contar variables esperadas vs variables presentes en el SQL. Si esperabas 6 y hay 5, una se perdió.

> "Si me hubiera fijado en el output, hubiera visto que no encontró esa variable. Y `df.rename` no marca error cuando la columna no existe — desde ahí me hubiera dado cuenta."

**Fix**: re-nombrar las superficies en Spaces → Surfaces / Sub Surfaces, re-correr.

> "Open Studio está pensado para cumplir métricas de consumo de energía y de Análisis de Ciclo de Vida (LCA), pero aquí ni siquiera hay consumo de energía y no he definido las salidas necesarias. Por eso aparecen 14 warnings — todos esos van a estar sucediendo una y otra vez y está bien."

## Warnings que SÍ importan

Algunos warnings son señales reales de un problema físico:

- **Surface convection algorithm fallback** — E+ no pudo aplicar la correlación esperada y usó una alternativa.
- **Window/material outside expected range** — propiedades térmicas u ópticas atípicas (puede ser intencional o un error de tipeo).
- **Convergence not achieved during warm-up** — el [[Warm-up-Period]] llegó al máximo de días sin converger; la condición inicial puede contaminar resultados.
- **Solar radiation never reaches a surface** — geometría sospechosa (orientación o sombreamiento que oculta totalmente una superficie expuesta).
- **Temperature outside reasonable bounds** — la simulación produjo valores irreales (ej. >100 °C en una zona); revisar materiales y condiciones de frontera.

## Estrategia de lectura del `.err`

1. **Buscar primero "Severe" / "Fatal"** — corregir todos antes de leer los warnings.
2. Tras corregir, **re-correr** y volver a leer.
3. Para warnings: agruparlos por tipo. Si todos los del mismo tipo son del catálogo "ignorable" (LCA, design days, etc.), seguir.
4. Cualquier warning fuera del catálogo conocido → investigar antes de interpretar resultados.

## Cuándo "es válida" la simulación

Que E+ haya corrido al **100%** y producido un HTML no es garantía de validez. El profesor enfatiza:

> "No porque funcione quiere decir que esté bien."

La simulación es válida cuando:

- No hay severos.
- Los warnings están comprendidos y aceptados (o son del catálogo ignorable).
- Los resultados pasan **sanity checks** (ver [[../procedures/Debuggear-Simulacion-OpenStudio]]).

## Clases relacionadas

- [[../classes/004-InterpretandoMensajesConstructionSets]] — primera lectura completa del `.err` y catálogo de warnings ignorables
- [[../classes/005-AnalisisSimulacionesPython]] — uso del `.rdd` (hermano del `.err`) para descubrir variables
- [[../classes/006-DosZonasTermicasVentanasAleros]] — warnings nuevos: vértices colineales y weather location difference; analogía con compiladores C; política del grupo
- [[../classes/007-CasoBaseAleros]] — bugs recurrentes: piso adiabático que se revierte, nombres de superficies borrados
- [[../classes/008-ShadingVentanas]] — warning "many overlapping shadows"; reflexión sobre cuándo culpar a E+ (10:2 reglas)
