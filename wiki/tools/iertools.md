---
title: iertools
type: herramienta
tags: [herramienta, paquete, python, sql, epw, ier, postprocesamiento]
aliases: [iertools, IER tools, ear_tools, ear-tools, paquete del profesor, paquete del grupo]
clases: [005, 006, 007, 008, 009, 011]
updated: 2026-05-02
---

# iertools

> **Nota sobre el nombre:** el paquete se llama **`iertools`** (de "IER tools" — Instituto de Energías Renovables). En las primeras clases del curso (transcripciones automáticas) se transcribió como **"ear_tools"** por la pronunciación del profesor. **El nombre correcto es `iertools`**.

## Qué es

Paquete de Python desarrollado por el grupo de **Energía en Edificaciones** del IER (UNAM) — utilidades de postprocesamiento para análisis de simulaciones de Energy Plus y archivos de clima EPW. Mantiene una API estable para que todo el grupo trabaje con la misma versión.

> "En lugar de estar reinventando la rueda — que además es fuente de error — automatizamos y todo el grupo usamos la misma versión, está mucho más bonito."

## Alcance

| Módulo | Lee | Devuelve |
|--------|-----|----------|
| `iertools.read.read_sql` | `eplusout.sql` (salida nativa de E+) | Objeto con `.data` (DataFrame de series temporales), `.construction_systems`, `.get_construction()` |
| `iertools.read.read_epw` | Archivo EPW de OneBuilding u otro | DataFrame con datetime como índice (nombre del índice: `tiempo`) |

(Hay módulos adicionales — descarga de datos de la plataforma IoT del grupo, análisis de EPWs avanzados — fuera del alcance del taller.)

## Por qué leer SQL en lugar de CSV

E+ produce los resultados en SQL y CSV en paralelo. El paquete lee SQL por dos razones específicas:

### 1. El CSV pone "24:00:00" para el último paso del día

```
12/31  23:50:00   ...
12/31  24:00:00   ←  rompe pandas to_datetime
01/01  00:10:00   ...
```

Pandas no parsea `24:00`. Toca arreglarlo manualmente cada vez. El SQL ya viene normalizado a `00:00 del día siguiente`.

### 2. Los reporting measures numeran sus folders dinámicamente

Cuando se agrega o quita un measure, el folder de salida cambia de nombre:

```
run/000_init/
run/001_translator/
run/002_addOutputVariable/
run/003_createCSVoutput/   ← este número cambia si agregas otro measure antes
```

Las libretas Jupyter que apuntan al CSV con un path fijo se rompen. El SQL siempre está en `run/eplusout.sql` — robusto.

## Instalación

Desde el repositorio público del grupo (URL en el README del proyecto del taller):

```bash
uv add git+https://github.com/<grupo-IER>/iertools.git
```

Ver flujo completo en [[../procedures/Setup-Entorno-Python-uv]].

## API — `read_sql`

```python
from iertools.read import read_sql

cubo = read_sql("OSM/mi_primer_cubo/run/eplusout.sql", alias=True)

# Acceder a las series temporales
df = cubo.data            # DataFrame con datetime como índice

# Inspeccionar sistemas constructivos
nombres = cubo.construction_systems       # lista de nombres
detalles = cubo.get_construction(nombres)  # propiedades, capas, espesores
```

> **Nombre del método**: `get_construction()` (singular). Acepta una lista de nombres y devuelve los detalles de cada uno.

### Parámetro `alias`

| `alias` | Comportamiento |
|---------|----------------|
| `False` (default) | Columnas con nombres completos: `CUBO:Zone Mean Air Temperature [C]` |
| `True` | Columnas renombradas con convención corta: `Ti_<zona>`, `To`, `Ib`, `Id`, etc. |

### Convención de alias

| Variable original | Alias |
|-------------------|-------|
| `Zone Mean Air Temperature` (zona X) | `Ti_<X>` |
| `Site Outdoor Air Drybulb Temperature` | `To` |
| `Site Direct Solar Radiation Rate per Area` | `Ib` (beam) |
| `Site Diffuse Solar Radiation Rate per Area` | `Id` (diffuse) |
| Global solar (calculada o sobre horizontal) | `Ig` |
| `Site Wind Speed` | `WS` |
| `Site Wind Direction` | `WD` |
| `Site Outdoor Air Relative Humidity` | `RH` |
| Presión atmosférica | `P` |

> **Importante**: el alias **no renombra todas las columnas** — solo las del catálogo conocido. Variables custom (ej. `TECHO:Surface Outside Face Incident Solar Radiation Rate per Area (W/m2)`) **conservan el nombre completo**. Hay que acceder a ellas con corchetes:
>
> ```python
> df["TECHO:Surface Outside Face Incident Solar Radiation Rate per Area (W/m2)"]
> ```

Ventaja: las columnas alias se vuelven nombres válidos de Python — se accede con `df.Ti_cubo` en lugar de `df["CUBO:Zone Mean Air Temperature [C]"]`.

### Inspección de sistemas constructivos

`cubo.get_construction(nombres)` devuelve, para cada construction:

- Nombre, espesor total, número de capas.
- Propiedades agregadas (absortancia visible, solar, térmica, rugosidad).
- Lista de capas en orden exterior → interior, cada una con conductivity, density, specific heat, thickness.

Ejemplo de salida (del notebook 001_EDA):

```
Construction system: CDA15CMA0P7
Total thickness: 0.15 m
Total layers: 1
InsideAbsorpVis: 0.7
OutsideAbsorpVis: 0.7
OutsideAbsorpSolar: 0.7
InsideAbsorpThermal: 0.9
OutsideRoughness: 2

  NameMaterial  Conductivity  Density  SpecHeat  Thickness
0  CDA15cma0p7           2.0   2500.0    1400.0       0.15
```

Útil para **auditar** que las propiedades térmicas que se usaron en la simulación son las que se quería usar — uno de los errores más comunes según el profesor.

## API — `read_epw`

```python
from iertools.read import read_epw

epw = read_epw("EPW/MX_MO_Cuernavaca.epw", alias=True)
```

### Parámetros

| Parámetro | Default | Para qué |
|-----------|---------|----------|
| `alias` | `False` | Renombra columnas clave (`To`, `RH`, `Ib`, `Id`, `Ig`, `WS`, `WD`, `P`) |

> **Importante**: `read_epw` **no tiene parámetro `year`**. El reemplazo del año del TMY se hace manualmente:
>
> ```python
> epw = read_epw(f, alias=True)
> # Filtrar 29 de febrero (si el EPW base es bisiesto y el año destino no)
> epw = epw[~((epw.index.month == 2) & (epw.index.day == 29))]
> # Reemplazar año
> epw.index = epw.index.map(lambda x: x.replace(year=2023))
> ```

### Estructura del DataFrame retornado

- **Índice**: nombrado `tiempo`, datetimes con la frecuencia del EPW (típicamente horaria).
- **Columnas**: las 30+ del EPW estándar. El alias renombra solo las clave; las demás conservan el nombre original (ej. `Dew Point Temperature`, `Extraterrestrial Horizontal Radiation`, `Aerosol Optical Depth`).

### Workaround del 29-feb

Si el EPW base contiene un mes de un año bisiesto con el 29 de febrero, reemplazar el año a uno no bisiesto **falla** en el `.replace(year=...)` (el 29-feb no existe en el año destino).

Solución: filtrar el 29-feb antes:

```python
epw = epw[~((epw.index.month == 2) & (epw.index.day == 29))]
epw.index = epw.index.map(lambda x: x.replace(year=2023))
```

Resultado: 8759 filas (8760 − 1) en lugar de fallar.

## Filosofía

- **Reproducibilidad**: todos en el grupo usan la misma versión → resultados comparables.
- **Robustez**: el SQL es estable, no depende del orden de measures ni del formato del CSV.
- **Productividad**: alias cortos + acceso punto-atributo → análisis exploratorio más rápido.
- **Auditabilidad**: inspeccionar materiales y constructions sin abrir el OSM en la GUI.

## Cuándo NO usar el paquete

Para análisis muy específicos que requieren consultas directas al SQL (ej. extraer una sub-tabla de medidores HVAC con joins manuales), conviene usar `sqlite3` directo. El paquete es la opción default — los casos esquina justifican bajar al SQL crudo.

## Clases relacionadas

- [[../classes/005-AnalisisSimulacionesPython]] — introducción y demo de uso

## Notebooks que usan iertools

- [[../notebooks/001_EDA]] — primera demo de `read_sql` con `alias=True`, auditoría de constructions, plots de doble panel
- [[../notebooks/002_EDA_EPW]] — primera demo de `read_epw` con workaround del 29-feb

## Procedimientos

- [[../procedures/Setup-Entorno-Python-uv]] — instalación
- [[../procedures/Analizar-Resultados-Python]] — uso típico de `read_sql`
- [[../procedures/EDA-Archivo-EPW]] — uso típico de `read_epw`
