---
title: EDA del archivo EPW
type: procedimiento
tags: [procedimiento, python, ear-tools, epw, eda, confort-adaptativo, clima]
aliases: [eda epw, analizar epw, leer epw python]
clases: [005]
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
from ear_tools.read import read_epw
```

## 3. Cargar el EPW

```python
f = "../EPW/MX_MO_Cuernavaca.AP.766800_TMYx.2009-2023.epw"
epw = read_epw(f, alias=True, year=2006, suppress_warnings=True)
```

### Por qué `year=2006`

El EPW es un [[../concepts/TMY]] — un Frankenstein con cada mes de un año real distinto. Sin `year`, el datetime tiene saltos:

```
2010-01-31 23:00
2018-02-01 00:00   ← salto al año del mes elegido para febrero
```

…lo que rompe series temporales en pandas. Con `year=2006`, **todos** los timestamps quedan en el mismo año arbitrario:

```
2006-01-31 23:00
2006-02-01 00:00   ← continuo
```

`2006` es el default histórico de Open Studio para mantener consistencia con sus salidas.

### Por qué `suppress_warnings=True`

Cambiar el año emite un warning ("estás cambiando el año"). Si lo haces a propósito, `suppress_warnings=True` lo silencia. Si no lo silencias, sale un warning largo cada ejecución.

## 4. Bug conocido — años bisiestos

Si el EPW base usa un año bisiesto en su mes de febrero, `read_epw` con `year=2006` puede fallar al intentar reemplazar el año porque el 29-feb no existe en el año destino.

**Workaround** mientras `ear_tools` no lo parchea:

```python
epw_raw = read_epw(f, alias=True, suppress_warnings=True)  # sin year= todavía

# Filtrar el 29 de febrero (si existe)
mask_29feb = (epw_raw.index.month == 2) & (epw_raw.index.day == 29)
epw_raw = epw_raw[~mask_29feb]

# Ahora reemplazar año manualmente
epw_raw.index = epw_raw.index.map(lambda x: x.replace(year=2006))
epw = epw_raw
```

Resultado: 8760 horas (año no bisiesto) en lugar de 8784.

> El profesor mencionó que parcheará esto en la próxima versión de `ear_tools`. Si tu versión ya lo tiene, `year=2006` directo funciona.

## 5. Inspección rápida con subplots

Una variable por subplot (lectura visual del año completo):

```python
epw.plot(subplots=True, figsize=(12, 20))
```

Genera un panel por columna (T, RH, IB, ID, WS, WD, P, ...). **No está documentado oficialmente** en pandas pero funciona ("un huevo de Pascua en pandas").

> Si los paneles aparecen con años distintos en el eje X (2010, 2018, 2019…), olvidaste `year=2006`.

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
from ear_tools.read import read_sql

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

## 9. Grados-hora de disconfort (preview de tareas siguientes)

```python
# Disconfort cálido: cuánto excede la banda superior
gh_calido = ((df.T_cubo - (T_neut_at_t + delta)).clip(lower=0) * (10/60)).sum()

# Disconfort frío: cuánto cae debajo de la banda inferior
gh_frio = (((T_neut_at_t - delta) - df.T_cubo).clip(lower=0) * (10/60)).sum()

print(f"Grados-hora cálidos:   {gh_calido:.0f} °C·h")
print(f"Grados-hora fríos:     {gh_frio:.0f} °C·h")
```

(Multiplicador `10/60` porque el paso temporal es 10 minutos = 1/6 de hora.)

Esto será una de las métricas centrales del proyecto final.

## Otras herramientas de comparación

**Climate Consultant** (mencionado en clase): GUI que aplica modelos adaptativos automáticamente y sugiere estrategias bioclimáticas. Útil para una primera mirada, pero "las gráficas son horrorosas, juntan demasiada información, no se pueden modificar". Por eso en el curso se hace todo en Python.

## Clases relacionadas

- [[../classes/005-AnalisisSimulacionesPython]] — primera demo del análisis del EPW

## Procedimientos relacionados

- [[Descargar-EPW-OneBuilding]] — cómo conseguir el EPW
- [[Analizar-Resultados-Python]] — análisis de la simulación
