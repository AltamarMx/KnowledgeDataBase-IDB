---
title: Salida SQL de Energy Plus
type: concepto
tags: [concepto, energyplus, salida, sql, csv, html, postprocesamiento]
aliases: [eplusout.sql, salida sql, csv energyplus, reporte html]
clases: [004, 005]
updated: 2026-05-02
---

# Salida SQL de Energy Plus

## Tres formatos de salida

E+ produce los resultados de una simulación en **varios formatos en paralelo**:

| Formato | Archivo típico | Contenido | Uso |
|---------|----------------|-----------|-----|
| **SQL** | `eplusout.sql` | Base de datos relacional con todas las series temporales | Análisis con scripts |
| **CSV** | `eplusout.csv` | Tabla plana de series temporales | Excel u otros (formato "medio feo") |
| **HTML** | `eplustbl.htm` | Reporte tabular pre-formateado | Vista preliminar, ASHRAE/LCA |
| **ERR** | `eplusout.err` | Mensajes de simulación | Debug — ver [[Mensajes-EnergyPlus]] |

Hay otros (`eplusout.eso`, `eplusout.mtr`, etc.) pero los cuatro de arriba son los principales.

## Por qué SQL

El **SQL** (SQLite, base de datos estructurada) es el formato que E+ recomienda para postprocesamiento porque:

1. **Eficiencia en almacenamiento** — la base normaliza los datos: en lugar de almacenar literalmente 365 fechas como strings, guarda los componentes (días 1-31, meses 1-12, horas 0-23) y los compone con joins.
2. **Lectura selectiva** — se puede pedir un subconjunto de variables sin cargar el archivo entero a memoria.
3. **Queries con SQL estándar** — para quien sabe SQL, sacar la temperatura de una zona en julio entre 14:00-18:00 es trivial.

> "Si yo voy a estar usando fechas, descompone las fechas en sus componentes básicos. En lugar de tener 365 fechas, tengo los días del 1 al 31 y nada más los mando a llamar, los meses del 1 al 12 y nada más los mando a llamar."

## Limitación: no se ve "abriendo" el archivo

Un `.sql` no es texto plano — abrirlo con un editor muestra binario incomprensible. Para verlo se necesita:

- Un **navegador SQLite** (DB Browser for SQLite, TablePlus).
- Un **script** que abra la base y haga queries (Python con `sqlite3`, R, etc.).
- Un **reporting measure** de Open Studio que extraiga lo que se necesite y lo escriba en HTML/CSV.

## El paquete del grupo para Python — `ear_tools`

El grupo de Energía en Edificaciones del IER mantiene **`ear_tools`** — paquete que lee el SQL directamente y devuelve dataframes de pandas con las series temporales. Detalle de la API en [[../tools/ear-tools]].

Ventajas vs CSV nativo:

### 1. El CSV pone `24:00:00` para el último paso del día

```
12/31  23:50:00   ...
12/31  24:00:00   ←  pandas.to_datetime no parsea "24:00"
01/01  00:10:00   ...
```

El SQL ya viene con el timestamp normalizado a `00:00 del día siguiente`.

> "En Open Studio salen bien, pero en Energy Plus en lugar de ponerme las 23:50 y luego las 0:00 del siguiente día, me pone 24:00 y eso rompe el datetime de pandas."

### 2. Los reporting measures numeran sus folders dinámicamente

Cuando se agrega o quita un measure, el folder de salida cambia de nombre:

```
run/004_addOutputVariable/eplusout.csv   ← este número cambia si agregas otro measure
run/005_createCSVoutput/...
```

Las libretas Jupyter que apuntan al CSV con un path fijo se rompen. **El SQL siempre está en `run/eplusout.sql`** — robusto.

> "Si yo hago un cambio y meto un measure más, ya me arruinó mis paths. Hacerlo desde el SQL se vuelve más robusto y eso es lo que yo quiero."

### 3. Renombre de columnas (alias) integrado

`ear_tools` con `alias=True` renombra las columnas largas como `CUBO:Zone Mean Air Temperature [C]` a nombres cortos como `T_cubo` — accesibles con punto-atributo en pandas. Convención completa en [[../tools/ear-tools]].

## Reporte HTML

Tras la simulación E+ genera un reporte HTML pre-formateado. Está pensado para análisis de **consumo energético** y **Análisis de Ciclo de Vida** según ASHRAE — no para análisis bioclimático puro.

Secciones típicas:

- **Site and Source Energy** — consumo en site y source con [[Site-Source-Factor|factores aplicados]].
- **Annual Building Utility Performance Summary (ABUPS)** — desglose por uso final (HVAC, iluminación, equipos).
- **Climatic Data Summary** — resumen del EPW.
- **Envelope Summary** — superficies y sus propiedades.
- **Component Sizing Summary** — dimensionamiento de equipos HVAC.

Para el alcance del curso (cubo sin cargas internas, sin HVAC) el HTML está casi vacío — la mayoría de tablas reportan 0 o "No data".

## CSV — cuando sí conviene

El CSV nativo de E+ se pide configurando salidas específicas en el IDF/OSM (`Output:Variable`, `OutputControl:Files` con `OutputCSV=Yes`). Aunque el formato es "medio feo", para análisis ad hoc en Excel sigue siendo útil:

- Compartir resultados con quien no sabe Python.
- Verificar rápido que una variable se está reportando.
- Para series temporales pequeñas (días, no años).

## Volumen de datos típico

Para una simulación anual con paso de 10 minutos y dos zonas térmicas:

- **Pasos por día**: 24 × 6 = 144
- **Días al año**: 365
- **Pasos al año por zona**: ~52,560
- **Variables típicas**: temperatura del aire, temperaturas de superficies, ganancias, etc.
- Total fácilmente: cientos de miles de filas → **pandas obligado**.

## Clases relacionadas

- [[../classes/004-InterpretandoMensajesConstructionSets]] — introducción a los formatos de salida y mención del paquete Python del profesor
- [[../classes/005-AnalisisSimulacionesPython]] — `ear_tools.read_sql` en uso, problemas concretos del CSV (`24:00`, numeración de folders)
