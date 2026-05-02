---
title: Debuggear una simulación de Open Studio
type: procedimiento
tags: [procedimiento, debugging, openstudio, energyplus]
aliases: [debug simulacion, depurar simulacion, debug openstudio]
clases: [004, 007]
updated: 2026-05-02
---

# Debuggear una simulación de Open Studio

Flujo end-to-end para diagnosticar y corregir una simulación que falla, corre con warnings sospechosos, o produce resultados implausibles.

## 0. Antes de empezar — sanity de archivos

Si la simulación falla **al abrir** o **al correr**, antes de mirar mensajes:

1. ¿El OSM está en `OSM/` y no dentro del folder hermano? Ver [[Estructura-Proyecto-Simulacion]] (regla crítica: nunca mover el OSM al folder que crea Open Studio automáticamente).
2. ¿El path al EPW sigue siendo válido? Si moviste el folder, hay que volver a hacer `Set Weather File`.
3. ¿La versión de Open Studio coincide con la que creó el archivo? Versiones nuevas abren archivos viejos, pero no al revés.

## Bugs recurrentes a chequear primero

Cuando una simulación corre pero los resultados son **sospechosos** (idénticos a los del caso base, o un severe del piso aparecido de repente), revisar primero estos bugs conocidos antes de bucear más profundo:

### Piso adiabático que se revierte a Ground

Tras cualquier cambio geométrico en FloorspaceJS, el piso puede revertir su condición de frontera de `Adiabatic` a `Ground`:

1. Pestaña **Spaces → Surfaces**, columna `Outside Boundary Condition` del piso.
2. Si dice `Ground`: cambiar a `Adiabatic`.
3. Si dice `Ground` y el slot Ground del Construction Set está sobre-definido: la simulación corre sin severe pero el resultado puede estar **silenciosamente incorrecto**.

Detalle en [[../concepts/Mensajes-EnergyPlus]] sección "Bugs recurrentes".

### Nombres de superficies borrados

Si pediste variables por nombre específico (`Techo`, `vNorte`, `vOeste`) y al cargar el SQL falta una variable:

1. Pestaña **Spaces → Surfaces** y **Sub Surfaces**.
2. Verificar que los nombres custom siguen ahí (no se revirtieron a `Face N`).
3. Si se perdieron: re-nombrar y re-correr.

**Pista numérica**: contar variables esperadas vs columnas en el SQL. Si esperabas 6 y hay 5, una se perdió.

> "Si me hubiera fijado en el output, hubiera visto que no encontró esa variable. Y `df.rename` no marca error cuando la columna no existe."

## 1. Correr y leer el `.err`

Procedimiento detallado en [[Leer-Archivo-ERR]]. Resumen:

1. `Run` desde la pestaña Run Simulation.
2. **Show Simulation** → abrir `eplusout.err`.
3. **Buscar Severes / Fatals primero** — corregir antes de continuar.
4. Una vez sin severes, agrupar warnings y filtrar el catálogo "ignorable" (LCA, design days). Ver [[../concepts/Mensajes-EnergyPlus]].

## 2. Inspección estructural del modelo

Antes de confiar en los resultados, recorrer el modelo con la siguiente checklist.

### Site

- [ ] Pestaña **Site** → Weather File con datos (lat, lon, time zone visibles).
- [ ] El time zone corresponde a la ciudad esperada.

### Geometría

- [ ] Pestaña **Geometry → 3D View → Render By → Surface Type**: muros amarillos, techos rojos, pisos grises.
- [ ] **Render By → Boundary Conditions**: Outdoor (azul), Surface (verde donde dos zonas se tocan), Ground (café), Adiabatic (rojo).
- [ ] No hay superficies "sueltas" sin construction (en preview no se renderizan).

Ver [[../concepts/Tipos-Superficie]] y [[../concepts/Condiciones-de-Frontera]].

### Espacios y zonas térmicas

- [ ] Pestaña **Spaces** → cada espacio tiene una zona térmica asignada.
- [ ] Nombres descriptivos (no `Thermal Zone 1`, `Space 2`).
- [ ] Sin acentos, sin eñes, sin espacios.

### Superficies

- [ ] Cada espacio reporta el número correcto de superficies (un cubo = 6).
- [ ] Cada superficie tiene **construction** asignada (verde si viene del Construction Set, negra si es local).
- [ ] **No hay sub-superficies** si no se modelan ventanas/puertas.
- [ ] **No hay superficies de sombreamiento internas** si no se modelan.

### Construction Set

- [ ] Pestaña **Facility** → Default Construction Set asignado.
- [ ] Slots ocupados para cada combinación tipo+condición presente en el modelo (ver [[Configurar-Construction-Set]]).

### Cargas

En el alcance del curso (sin cargas internas):

- [ ] Pestaña **Loads** vacía o con cargas a 0.
- [ ] Pestaña **Space Types** sin tipos asignados (o con tipos vacíos).

## 3. Sanity check de resultados

Pasar el `.err` no garantiza que el modelo sea físicamente correcto. Sospechar siempre:

### Inspección rápida del HTML

Pestaña **Results Summary**. Para el caso del curso (cascarón sin cargas, sin HVAC):

- **Site Energy** = 0 (esperable — no hay equipos).
- **Climatic Data Summary** debe mostrar la ciudad correcta (T_máx, T_mín del año del EPW).
- **Envelope Summary** muestra las áreas y constructions — verificar que coinciden con lo que dibujaste.

### Verificar que las variables solicitadas están en el output

Si configuraste measures de output (ver [[Solicitar-Output-Variables-Measures]]), verificar que se generaron:

- Abrir `eplusout.rdd` y confirmar que las variables esperadas están listadas (el RDD muestra qué **se puede pedir**, no necesariamente lo que **se pidió**).
- Abrir el CSV (si pediste `Create CSV Output`) y confirmar las columnas.
- O en Python: `read_sql(file).data.columns` muestra todas las series temporales solicitadas.

Si una variable no está: el measure de Add Output Variable tiene typo, mayúscula equivocada o coma sobrante en el nombre. Comparar contra la línea exacta del RDD.

### Series temporales — pandas

El análisis serio requiere bajar el SQL/CSV a pandas. Patrones a verificar:

| Patrón | Diagnóstico |
|--------|-------------|
| **T_zona = T_amb** todo el año | La edificación no está aislando — quizás todas las superficies son virtualmente abiertas o no hay masa. |
| **T_zona constante** todo el año | Sobre-amortiguamiento extremo (masa irreal) o se quedó un setpoint impuesto. |
| **T_zona oscila pero amortiguada y desfasada** respecto a T_amb | Comportamiento esperado de una edificación masiva — bien. |
| **T_zona sale del rango realista** (>50°C, <-20°C en climas templados) | Problema de balance: revisar absortancias, condiciones de frontera, ventanas mal asignadas. |
| **Saltos abruptos diarios** entre días consecutivos | El [[../concepts/Warm-up-Period]] no convergió, o hay un cambio de schedule sin transición. |

### Comparación con caso conocido

Si el modelo es similar a uno que ya corrió bien antes, comparar resultados:

- ¿Las temperaturas tienen el mismo orden de magnitud?
- ¿Las amplitudes (T_max − T_min) son comparables?

Diferencias grandes en modelos similares = indicio de que algo se rompió (un material mal escrito, una orientación distinta).

## 4. Iteración

Tras corregir cada problema:

1. **Save As** con nuevo número de versión (ver [[Estructura-Proyecto-Simulacion]]).
2. `Run`.
3. Re-leer `.err`.
4. Re-hacer la checklist estructural si tocaste geometría / constructions / Construction Set.
5. Re-hacer el sanity de resultados.

Iteración rápida en cubos simples toma minutos. En modelos complejos, puede ser horas.

## 5. Cuándo "tirar y empezar de nuevo"

> "A veces es más fácil empezar de nuevo que tratar de arreglar la geometría."

Casos típicos para reiniciar:

- Geometría con polígonos rotos que la GUI no permite editar limpiamente.
- OSM que se corrompió por mover archivos al folder hermano.
- Construction Set con configuración acumulada de pruebas que ya no se sabe qué hace.

La práctica de **versionado numerado** (`001_*.osm`, `002_*.osm`, …) hace este reinicio barato — se regresa a la última versión que funcionaba y se rehace el camino.

## 6. Cuándo pedir ayuda

Si tras varias iteraciones no se identifica el problema:

- Mensaje al chat del grupo describiendo:
  - Qué pasos seguiste.
  - Qué dice el `.err` (copiar la sección relevante).
  - Adjuntar el ZIP del proyecto completo (NO solo el OSM).
- El profesor responde "en algún momento de la madrugada".

## Cuando dos simulaciones dan resultados iguales

Síntoma específico de la clase 007: caso base y variante con cambio aparente producen series temporales **casi idénticas**.

Diagnóstico en orden de probabilidad:

1. **Flujo de datos en Python**: ¿la función `carga_df(f)` redefine `f` adentro? ¿estás cargando el mismo SQL dos veces? Imprimir `f` dentro de la función para verificar.
2. **Cambio no llegó al IDF**: el measure de la variable solicitada por nombre específico (ej. `Techo`) no encontró la superficie porque el nombre se perdió. Contar columnas en el SQL.
3. **Solo al final culpar a Energy Plus** — es lo menos probable.

> "Es bien fácil desconfiar de Energy Plus, pero la mayoría de las veces son errores del usuario. Mi primer pista es voy a revisar que estoy cargando bien los datos."

## Clases relacionadas

- [[../classes/004-InterpretandoMensajesConstructionSets]] — debugging en vivo del proyecto de un equipo
- [[../classes/007-CasoBaseAleros]] — bug del piso adiabático y de nombres custom borrados; debugging de simulaciones aparentemente idénticas

## Procedimientos relacionados

- [[Leer-Archivo-ERR]] — primera lectura del `.err`
- [[Configurar-Construction-Set]] — verificar el set asignado
- [[Estructura-Proyecto-Simulacion]] — versiones, naming, ZIP
- [[Crear-Primera-Simulacion-OpenStudio]] — flujo base que se está debuggeando
- [[Solicitar-Output-Variables-Measures]] — verificar que las variables solicitadas se generaron
- [[Analizar-Resultados-Python]] — análisis post-debug
