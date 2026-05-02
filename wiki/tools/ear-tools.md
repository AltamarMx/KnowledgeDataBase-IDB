---
title: ear_tools
type: herramienta
tags: [herramienta, paquete, python, sql, epw, ier, postprocesamiento]
aliases: [ear_tools, ear-tools, ear tools, paquete del profesor]
clases: [005]
updated: 2026-05-02
---

# ear_tools

## Qué es

Paquete de Python desarrollado por el grupo de **Energía en Edificaciones** del IER (UNAM) — utilidades de postprocesamiento para análisis de simulaciones de Energy Plus y archivos de clima EPW. Mantiene una API estable para que todo el grupo trabaje con la misma versión.

> "En lugar de estar reinventando la rueda — que además es fuente de error — automatizamos y todo el grupo usamos la misma versión, está mucho más bonito."

## Alcance

| Módulo | Lee | Devuelve |
|--------|-----|----------|
| `read.read_sql` | `eplusout.sql` (salida nativa de E+) | Objeto con `.data` (DataFrame de series temporales), `.construction_systems`, `.get_constructions()` |
| `read.read_epw` | Archivo EPW de OneBuilding u otro | DataFrame con datetime como índice |

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
uv add git+https://github.com/<grupo-IER>/ear-tools.git
```

Ver flujo completo en [[../procedures/Setup-Entorno-Python-uv]].

## API — `read_sql`

```python
from ear_tools.read import read_sql

cubo = read_sql("OSM/mi_primer_cubo/run/eplusout.sql", alias=True)

# Acceder a las series temporales
df = cubo.data            # DataFrame con datetime como índice

# Inspeccionar sistemas constructivos
nombres = cubo.construction_systems       # lista de nombres
detalles = cubo.get_constructions(nombres)  # propiedades, capas, espesores
```

### Parámetro `alias`

| `alias` | Comportamiento |
|---------|----------------|
| `False` (default) | Columnas con nombres completos: `CUBO:Zone Mean Air Temperature [C]` |
| `True` | Columnas renombradas con convención corta: `T_cubo`, `TO`, `IB`, `ID`, etc. |

### Convención de alias

| Variable original | Alias |
|-------------------|-------|
| `Zone Mean Air Temperature` (zona X) | `T_<X>` |
| `Site Outdoor Air Drybulb Temperature` | `TO` |
| `Site Direct Solar Radiation Rate per Area` | `IB` (beam) |
| `Site Diffuse Solar Radiation Rate per Area` | `ID` (diffuse) |
| Global solar (calculada o sobre horizontal) | `IG` |
| `Site Wind Speed` | `WS` |
| `Site Wind Direction` | `WD` |
| `Site Outdoor Air Relative Humidity` | `RH` |
| Presión atmosférica | `P` |

Ventaja: las columnas se vuelven nombres válidos de Python — se accede con `df.T_cubo` en lugar de `df["CUBO:Zone Mean Air Temperature [C]"]`.

### Inspección de sistemas constructivos

`cubo.get_constructions(nombres)` devuelve, para cada construction:

- Nombre, espesor total, número de capas.
- Propiedades agregadas (absortancia visible, etc.).
- Lista de capas en orden exterior → interior, cada una con conductivity, density, specific heat, thickness, thermal absorptance, roughness.

Útil para **auditar** que las propiedades térmicas que se usaron en la simulación son las que se quería usar — uno de los errores más comunes según el profesor.

## API — `read_epw`

```python
from ear_tools.read import read_epw

epw = read_epw(
    "EPW/MX_MO_Cuernavaca.epw",
    alias=True,
    year=2006,
    suppress_warnings=True,
)
```

### Parámetros

| Parámetro | Default | Para qué |
|-----------|---------|----------|
| `alias` | `False` | Renombra columnas (TO, RH, IB, ID, WS, WD, P, ...) |
| `year` | `None` | Reemplaza el año en cada timestamp por el especificado. **Recomendado** porque un TMY tiene años distintos por mes (Frankenstein) y eso rompe el orden temporal en pandas |
| `suppress_warnings` | `False` | Silencia el warning de "estás cambiando el año" cuando es intencional |

### Bug conocido — años bisiestos

Si el EPW base contiene un mes de un año bisiesto con el 29 de febrero, reemplazar el año puede fallar (en la versión vista en clase 005). Workaround: filtrar el 29-feb antes de aplicar el reemplazo (ver [[../procedures/EDA-Archivo-EPW]]). El profesor planea parchearlo.

## Filosofía

- **Reproducibilidad**: todos en el grupo usan la misma versión → resultados comparables.
- **Robustez**: el SQL es estable, no depende del orden de measures ni del formato del CSV.
- **Productividad**: alias cortos + acceso punto-atributo → análisis exploratorio más rápido.
- **Auditabilidad**: inspeccionar materiales y constructions sin abrir el OSM en la GUI.

## Cuándo NO usar el paquete

Para análisis muy específicos que requieren consultas directas al SQL (ej. extraer una sub-tabla de medidores HVAC con joins manuales), conviene usar `sqlite3` directo. El paquete es la opción default — los casos esquina justifican bajar al SQL crudo.

## Clases relacionadas

- [[../classes/005-AnalisisSimulacionesPython]] — introducción y demo de uso

## Procedimientos

- [[../procedures/Setup-Entorno-Python-uv]] — instalación
- [[../procedures/Analizar-Resultados-Python]] — uso típico de `read_sql`
- [[../procedures/EDA-Archivo-EPW]] — uso típico de `read_epw`
