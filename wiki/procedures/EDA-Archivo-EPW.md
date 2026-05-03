---
title: EDA del archivo EPW
type: procedimiento
tags: [procedimiento, python, iertools, epw, eda, confort-adaptativo, clima]
aliases: [eda epw, analizar epw, leer epw python]
clases: [005, 008]
updated: 2026-05-02
---

# EDA del archivo EPW

Procedimiento para hacer un Análisis Exploratorio de Datos del archivo EPW del proyecto antes (o en paralelo a) las simulaciones. Se usa para entender el clima de la ciudad escogida y para preparar el cálculo de [[../concepts/Confort-Adaptativo]].

## Pre-requisitos

- Entorno Python listo. Ver [[Setup-Entorno-Python-uv]].
- EPW descargado en `EPW/`. Ver [[Descargar-EPW-OneBuilding]].

## 1. Crear la libreta

En `notebooks/`, crear `002_EDA_EPW.ipynb`.

## 2. Imports

```python
import pandas as pd
import matplotlib.pyplot as plt
from iertools.read import read_epw
```

## 3. Cargar el EPW

```python
f = "../EPW/MX_MO_Cuernavaca.AP.766800_TMYx.2009-2023.epw"
epw = read_epw(f, alias=True)
```

> **Importante**: `read_epw` **NO tiene parámetro `year`**. Lo que la transcripción de la clase 005 sugiere es un error — el reemplazo del año del TMY se hace manualmente. Confirmado en el notebook [[../notebooks/002_EDA_EPW]].

### Por qué hay que normalizar el año

El EPW es un [[../concepts/TMY]] — un Frankenstein con cada mes de un año real distinto. Tal cual viene del archivo, el datetime tiene saltos entre meses:

```
2010-01-31 23:00
2018-02-01 00:00   ← salto al año del mes elegido para febrero
```

…lo que rompe series temporales en pandas (no se puede iterar continuamente). Hay que normalizar a un año único.

## 4. Workaround del 29-feb + reemplazo de año

```python
# Filtrar 29 de febrero antes (si el año destino no es bisiesto)
epw = epw[~((epw.index.month == 2) & (epw.index.day == 29))]

# Reemplazar año de cada timestamp
epw.index = epw.index.map(lambda x: x.replace(year=2023))
```

### Por qué filtrar el 29-feb antes

Si el EPW base contiene un febrero proveniente de un año bisiesto (con 29 de febrero) y el año destino **no** es bisiesto:

- `datetime(2020, 2, 29).replace(year=2023)` → **excepción** (29-feb no existe en 2023).

Filtrar el 29-feb evita el crash. Pérdida: 24 horas de datos al año (despreciable para análisis bioclimático).

Resultado: **8759 filas** (8760 − 1) en lugar de fallar.

### Año destino — qué elegir

Cualquier año arbitrario sirve. Convenciones:

- **`2023`** — usado en el notebook 002.
- **`2006`** — el default histórico de Open Studio cuando reporta resultados.

Para **comparar simulaciones de E+ con datos del EPW** en una misma gráfica, conviene usar **el mismo año** en ambos lados. Si E+ reporta en 2006 y el EPW se normalizó a 2023, los plots no se alinean.

## 5. Inspección rápida con subplots

Una variable por subplot (lectura visual del año completo):

```python
epw.plot(subplots=True, figsize=(12, 20))
```

Genera un panel por columna (T, RH, Ib, Id, WS, WD, P, etc.). **No está documentado oficialmente** en pandas pero funciona ("un huevo de Pascua en pandas").

> Si los paneles aparecen con años distintos en el eje X (2010, 2018, 2019…), olvidaste el reemplazo de año del paso 4.

### Estructura del DataFrame

- **Índice**: nombrado `tiempo`, datetimes horarios.
- **Columnas**: 30+ del EPW estándar.

`alias=True` renombra solo el catálogo conocido (`To`, `RH`, `Ib`, `Id`, `Ig`, `WS`, `WD`, `P`). Las demás conservan el nombre original (`Dew Point Temperature`, `Aerosol Optical Depth`, etc.) — acceder con corchetes:

```python
epw["Dew Point Temperature"]
```

## 6. Agregaciones temporales

```python
# T promedio, mínima, máxima por mes
T_mes_avg = epw.TO.resample("ME").mean()
T_mes_min = epw.TO.resample("ME").min()
T_mes_max = epw.TO.resample("ME").max()
```

> **Nota**: el alias del resample mensual cambió a `"ME"` en pandas reciente (antes era `"M"`, ahora deprecated). Si tu versión muestra warning, ya está actualizada.

Otros resamples útiles:

| Frecuencia | Alias |
|------------|-------|
| Hora | `"h"` |
| Día | `"D"` |
| Semana | `"W"` |
| Mes | `"ME"` (Month-End) |
| Año | `"YE"` |

## 7. Cálculo de la zona de confort adaptativo

Modelo de Humphreys-Nicol (ver [[../concepts/Confort-Adaptativo]]):

```python
# T promedio mensual
T_mes = epw.TO.resample("ME").mean()

# T de neutralidad por mes
T_neut = 0.54 * T_mes + 13.5

# Zona de confort (banda ±3.5°C, 80% aceptabilidad)
delta = 3.5
T_conf_min = T_neut - delta
T_conf_max = T_neut + delta

# Visualizar
fig, ax = plt.subplots(figsize=(12, 4))
ax.plot(T_mes,    label="T_out promedio mensual")
ax.plot(T_neut,   label="T_neutralidad")
ax.fill_between(T_neut.index, T_conf_min, T_conf_max, alpha=0.3, label="Zona de confort")
ax.set_ylabel("Temperatura (°C)")
ax.legend()
```

## 8. Cruzar con T interior de la simulación

Para evaluar el % del año en confort:

```python
from iertools.read import read_sql

sim = read_sql("../OSM/<caso>/run/eplusout.sql", alias=True)
df = sim.data

# Mapear cada timestamp de la simulación al T_neut del mes
T_neut_at_t = df.index.to_series().map(
    lambda t: T_neut[T_neut.index.month == t.month].iloc[0]
)

en_confort = (df.T_cubo >= T_neut_at_t - delta) & (df.T_cubo <= T_neut_at_t + delta)
porcentaje_confort = en_confort.mean() * 100
print(f"% del año en confort: {porcentaje_confort:.1f}")
```

## 9. Grados-hora de disconfort (métrica central del proyecto final)

Detalle del concepto en [[../concepts/Grados-Hora-Disconfort]]. Cálculo:

```python
import numpy as np

dt = 10 / 60  # paso de 10 min en horas

# Variable de análisis — preferir T_op si la pediste; T del aire es aproximación
T_int = df.T_cubo

# Banda de confort (T_neut_at_t mapeada por mes — ver paso 8)
T_sup = T_neut_at_t + delta
T_inf = T_neut_at_t - delta

# Grados-hora cálidos y fríos — separados
gh_calido = np.maximum(T_int - T_sup, 0).sum() * dt
gh_frio   = np.maximum(T_inf - T_int, 0).sum() * dt

print(f"Grados-hora cálidos:   {gh_calido:>8.0f} °C·h")
print(f"Grados-hora fríos:     {gh_frio:>8.0f} °C·h")
```

> **No sumar** GH cálido + GH frío en una sola métrica — oculta el trade-off (estrategias que reducen calor a menudo aumentan frío).

Para el reporte del proyecto final: tabla comparativa caso base vs variantes con columnas separadas para GH cálido y GH frío, y diferencia relativa vs base. Detalle en [[../concepts/Grados-Hora-Disconfort]] sección "Reporte comparativo".

## Otras herramientas de comparación

**Climate Consultant** (mencionado en clase): GUI que aplica modelos adaptativos automáticamente y sugiere estrategias bioclimáticas. Útil para una primera mirada, pero "las gráficas son horrorosas, juntan demasiada información, no se pueden modificar". Por eso en el curso se hace todo en Python.

## Clases relacionadas

- [[../classes/005-AnalisisSimulacionesPython]] — primera demo del análisis del EPW
- [[../classes/008-ShadingVentanas]] — métrica grados-hora como métrica central del proyecto final, anti-patrón de sumar cálido+frío

## Procedimientos relacionados

- [[Descargar-EPW-OneBuilding]] — cómo conseguir el EPW
- [[Analizar-Resultados-Python]] — análisis de la simulación
