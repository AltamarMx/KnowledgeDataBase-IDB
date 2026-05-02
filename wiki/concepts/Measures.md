---
title: Measures (Open Studio)
type: concepto
tags: [concepto, openstudio, measures, scripting, ruby, parametricos]
aliases: [measures, openstudio measure]
clases: [004, 005]
updated: 2026-05-02
---

# Measures (Open Studio)

## Qué son

Scripts (escritos en **Ruby**) que **modifican automáticamente** una simulación entre etapas. Permiten agregar funcionalidades que la GUI de Open Studio no expone, automatizar cambios masivos, y ejecutar **estudios paramétricos**.

## Dónde encajan en el flujo

El pipeline real cuando se da `Run` en Open Studio:

```
OSM (texto plano)
   │
   ├── [OSM Measures]  ←  primer punto de inyección
   │
   ▼
OSM modificado
   │
   ▼
Traductor → IDF
   │
   ├── [IDF Measures]  ←  segundo punto de inyección
   │
   ▼
IDF modificado
   │
   ▼
Energy Plus corre el IDF
   │
   ▼
Resultados (SQL + ERR + HTML)
```

> Hay **dos puntos** donde el usuario puede inyectar measures: uno antes de la traducción a IDF (tipo `OpenStudio Measure`) y otro después (tipo `EnergyPlus Measure`). Distinguir importa porque algunos cambios solo se pueden hacer en el OSM (medidas geométricas, asignación de Construction Sets); otros solo en el IDF (objetos de E+ que Open Studio no expone).

## Tipos de Measure

| Tipo | Modifica | Ejemplos |
|------|----------|----------|
| **OpenStudio Measure** | El OSM antes de la traducción | Variar % área de ventana respecto al muro; cambiar Construction Set asignado; rotar la edificación; agregar particiones internas |
| **EnergyPlus Measure** | El IDF antes de correr | Insertar objetos de E+ no expuestos en Open Studio (ej. ciertos algoritmos de ventilación, controles avanzados de HVAC) |
| **Reporting Measure** | El postprocesamiento de resultados | Generar reportes custom a partir del SQL de E+ |

## Casos de uso típicos

### Estudios paramétricos

El uso más común. Ejemplos:

- **Variar el % de ventana**: una measure que toma el OSM y produce 4 variantes con ventanas al 25%, 50%, 75%, 100% del área del muro. Cada variante corre como simulación independiente.
- **Barrer sistemas constructivos**: aplicar 3 sistemas distintos a todos los muros exteriores → 3 simulaciones.
- **Barrer orientaciones**: rotar la edificación de 0° a 360° en pasos de 15°.

Útil para evaluar el impacto relativo de cada decisión de diseño.

### Acceso a features de E+ no expuestas

Open Studio expone solo un subconjunto de los objetos de E+ — los más usados en flujos ASHRAE/LEED estadounidenses. Cosas que típicamente requieren measures:

- **Ventanas operables** con controles avanzados de ventilación natural.
- **Airflow Network** completo.
- **Controles de iluminación** dinámicos.
- **Materiales con cambio de fase (PCM)**.

### Compliance con normativas

En EE.UU., los grandes despachos usan measures para validar que un proyecto cumple ASHRAE 90.1 / Title 24 / LEED. Los measures recorren el modelo y aplican reglas de la normativa.

En México **no se usa** ASHRAE; las normativas (NOM-008, NOM-020) usan modelos basados en U y no se evalúan con E+.

## Persistencia de measures en el OSM

Cuando se agrega un measure al OSM, su **configuración se guarda en el folder hermano** del archivo `.osm` (`<nombre>/measures/`). Si solo se comparte el `.osm` sin el folder, **los measures se pierden**.

Esto refuerza la regla de [[../procedures/Estructura-Proyecto-Simulacion]]: entregar siempre el ZIP del proyecto completo, no solo el OSM.

## Repositorio público

El **Building Component Library (BCL)** de NREL aloja measures publicados. Open Studio puede descargarlos desde la GUI (pestaña Measures → BCL). Hay measures para casi cualquier flujo común — antes de escribir uno desde cero, vale revisar el BCL.

## Reporting Measures usados en el taller

Aunque el taller no escribe measures (Ruby es horrible — palabras del profesor), sí **descarga y usa** dos Reporting Measures del BCL para pedir variables al output:

| Measure | Para qué | Ubicación BCL |
|---------|----------|---------------|
| **Add Output Variable** | Solicitar una variable específica al simulador (`Site Outdoor Air Drybulb Temperature`, `Zone Mean Air Temperature`, etc.). Uno por cada variable que se quiera. | Reporting → QAQC |
| **Create CSV Output** | Generar un CSV legible con los outputs. Útil como verificación visual de que las variables solicitadas se generaron. | Reporting → QAQC |

Procedimiento de configuración: [[../procedures/Solicitar-Output-Variables-Measures]].

> Nota práctica: cada Reporting Measure crea su propio sub-folder en `run/` con un número en orden de ejecución (`004_addOutputVariable`, `005_addOutputVariable`, ...). Si se agregan o quitan measures, los números cambian — por eso `ear_tools` lee del SQL (estable en `run/eplusout.sql`) en lugar del CSV. Detalle en [[Salida-SQL-EnergyPlus]].

## Cobertura en el curso

En el taller los measures **se usan para pedir output** pero **no se escriben**. Se ven brevemente para entender:

- Por qué hay un paso intermedio entre OSM e IDF.
- Por qué el flujo de Open Studio es más que "abrir el archivo y darle Run".
- Por qué algunas cosas avanzadas requieren scripting.

Quien quiera profundizar suele aprender Ruby y la API de OpenStudio SDK — fuera del alcance del taller.

## Clases relacionadas

- [[../classes/004-InterpretandoMensajesConstructionSets]] — introducción al flujo OSM→IDF y al rol de los measures
- [[../classes/005-AnalisisSimulacionesPython]] — uso concreto de `Add Output Variable` y `Create CSV Output` para pedir variables al simulador
