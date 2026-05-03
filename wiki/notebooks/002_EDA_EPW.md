---
title: 002 — EDA del EPW con iertools.read_epw
type: notebook
notebook: 002_EDA_EPW
fuente: raw/notebooks/002_EDA_EPW.ipynb
fecha_ingesta: 2026-05-02
tags: [notebook, eda, iertools, read_epw, clima, primera-libreta-epw, workaround-29-feb]
aliases: [002 EDA EPW, libreta EPW, primer EDA EPW]
clase_relacionada: 005
---

# 002 — EDA del EPW con `iertools.read_epw`

## Metadatos

- **Notebook:** `002_EDA_EPW.ipynb`
- **Fuente:** `raw/notebooks/002_EDA_EPW.ipynb`
- **Clase relacionada:** [[../classes/005-AnalisisSimulacionesPython]]
- **Objetivo:** Cargar un EPW de Cuernavaca, normalizar el año del TMY (con workaround del 29-feb), explorar visualmente las 30+ variables, calcular T promedio mensual.

## Resumen

Libreta corta pero con **dos patrones críticos**:

1. **Workaround del 29 de febrero** — `read_epw` no tiene parámetro `year`; el reemplazo se hace manualmente, y si el EPW base es bisiesto hay que filtrar el 29-feb antes.
2. **`subplots=True` para exploración** rápida — pandas grafica todas las columnas en paneles automáticos.
3. **Resample mensual** con `.resample("ME").mean()` para promedios calendario.

## Imports

```python
import pandas as pd
import matplotlib.pyplot as plt
from iertools.read import read_epw
```

## Carga + workaround del 29-feb

```python
f = "../EPW/MEX_MOR_Cuernavaca-Matamoros.Intl.AP.767260_TMYx.2009-2023.epw"
cue = read_epw(f, alias=True)
cue = cue[~((cue.index.month == 2) & (cue.index.day == 29))]
cue.index = cue.index.map(lambda x: x.replace(year=2023))
cue
```

### El problema del año del TMY

Un [[../concepts/TMY|TMY]] se construye con cada mes proveniente de un año real distinto. Sin reemplazar el año:

```
2010-01-31 23:00
2018-02-01 00:00   ← salto al año del mes elegido
```

Esto rompe la continuidad temporal en pandas → toca normalizar a un año único.

### Por qué hay que filtrar el 29-feb

`read_epw` **no tiene parámetro `year`** — el reemplazo es manual con `.index.map(lambda x: x.replace(year=...))`.

Si el EPW base contiene un febrero proveniente de un año bisiesto (con 29 de febrero) **y** el año destino **no** es bisiesto:

- `datetime(2020, 2, 29).replace(year=2023)` → **lanza excepción** porque el 29-feb no existe en 2023.

Solución: **filtrar el 29-feb antes** de aplicar el reemplazo:

```python
cue = cue[~((cue.index.month == 2) & (cue.index.day == 29))]
```

Resultado: 8759 filas (8760 − 1) en lugar de fallar.

> El profesor lo descubrió en vivo durante la clase 005 — hasta entonces el patrón era `read_epw(f, year=2006)` que **no existía**. La libreta refleja la API real del paquete.

## Estructura del DataFrame retornado

```
[8759 rows x 30 columns]
```

- **Índice**: nombrado `tiempo` (no `date` como en `read_sql`), datetimes horarios.
- **30 columnas**: las del EPW estándar.

### Columnas con alias (renombradas)

`alias=True` renombra solo el catálogo conocido:

| Variable original | Alias |
|-------------------|-------|
| Dry Bulb Temperature | `To` |
| Relative Humidity | `RH` |
| Atmospheric Station Pressure | `P` |
| Global Horizontal Radiation | `Ig` |
| Direct Normal Radiation | `Ib` |
| Diffuse Horizontal Radiation | `Id` |
| Wind Speed | `WS` |
| Wind Direction | `WD` |

### Columnas que conservan el nombre original

El alias **no toca** las demás 22+ columnas:

- `Data Source and Uncertainty Flags`
- `Dew Point Temperature`
- `Extraterrestrial Horizontal Radiation`
- `Extraterrestrial Direct Normal Radiation`
- `Horizontal Infrared Radiation Intensity`
- `Ceiling Height`
- `Present Weather Observation`
- `Present Weather Codes`
- `Precipitable Water`
- `Aerosol Optical Depth`
- `Snow Depth`
- `Days Since Last Snowfall`
- `Albedo`
- `Liquid Precipitation Depth`
- `Liquid Precipitation Quantity`

→ Acceder a estas con corchetes: `cue["Dew Point Temperature"]`.

## Visualización rápida — `subplots=True`

```python
cue.plot(subplots=True, figsize=(12, 20))
```

Pandas genera **un panel por columna** automáticamente. Útil para:

- **Inspección visual** de todas las variables del clima de una sola vez.
- Detectar **anomalías** (valores fuera de rango, gaps de datos).
- Identificar la **estacionalidad** (¿la radiación tiene su forma anual esperada? ¿la T sigue un patrón mensual?).

> Detalle no documentado oficialmente en pandas — "huevo de Pascua" según el profesor. Funciona y es útil para EDA exploratorio.

`figsize=(12, 20)` da paneles altos suficientes para ver detalle en 30 variables.

## Resample mensual

```python
cue.To.resample("ME").mean()
```

Salida — T promedio mensual de Cuernavaca:

```
tiempo
2023-01-31    19.20
2023-02-28    20.63
2023-03-31    23.27
2023-04-30    25.08    ← pico (abril)
2023-05-31    24.85
2023-06-30    23.08
2023-07-31    22.24
2023-08-31    22.02
2023-09-30    21.02
2023-10-31    21.53
2023-11-30    20.19
2023-12-31    18.81
Freq: ME, Name: To, dtype: float64
```

### Observaciones del clima de Cuernavaca

- **Mes más cálido**: abril (~25 °C promedio). Coherente con la lógica climática mexicana — abril/mayo es el pico cálido antes de las lluvias.
- **Más frío**: diciembre-enero (~19 °C). Templado, no frío extremo.
- **Amplitud anual**: ~6 °C entre el mes más cálido y el más frío. Clima estable comparado con climas extremos.

→ El clima de Cuernavaca tiene **un solo extremo** (cálido) — el caso ideal para diseño bioclimático según el consejo del profesor en clase 002 (evitar climas con doble extremo).

### Nota sobre `"ME"` en pandas reciente

Frecuencia `"ME"` = Month-End (mes calendario). En versiones anteriores de pandas era `"M"` — **deprecated en pandas 2+**. Usar siempre `"ME"`.

## Patrón de cálculo de T de neutralidad

A partir de la T promedio mensual se calcula la **temperatura de neutralidad** del modelo de Humphreys-Nicol:

$$
T_{neutralidad,mes} = 0.54 \cdot \overline{T}_{out,mes} + 13.5
$$

```python
T_mes = cue.To.resample("ME").mean()
T_neut = 0.54 * T_mes + 13.5
delta = 3.5  # 80% aceptabilidad
T_sup = T_neut + delta
T_inf = T_neut - delta
```

Para Cuernavaca:

| Mes | $\overline{T}_{out}$ | $T_{neut}$ | $T_{sup}$ (cálido) |
|-----|----------------------|------------|---------------------|
| Enero | 19.2 °C | 23.9 °C | 27.4 °C |
| Abril | 25.1 °C | 27.0 °C | 30.5 °C |
| Diciembre | 18.8 °C | 23.7 °C | 27.2 °C |

Detalle en [[../concepts/Confort-Adaptativo]].

## Path del EPW

```
"../EPW/MEX_MOR_Cuernavaca-Matamoros.Intl.AP.767260_TMYx.2009-2023.epw"
```

Convención de nombres de OneBuilding:

- `MEX_MOR` = México, Morelos.
- `Cuernavaca-Matamoros.Intl.AP` = aeropuerto internacional Mariano Matamoros (en realidad está en Temixco, no en Cuernavaca — confusión topónimo / IATA común).
- `767260` = WMO station code.
- `TMYx.2009-2023` = TMY construido con datos del periodo 2009-2023.

Detalle del flujo de descarga en [[../procedures/Descargar-EPW-OneBuilding]].

## Patrones de la libreta para reusar

1. **`read_epw(f, alias=True)`** — siempre con alias para `To`, `RH`, `Ig`, etc.
2. **Filtro del 29-feb antes** del reemplazo de año (si el año destino no es bisiesto).
3. **`.index.map(lambda x: x.replace(year=...))`** para normalizar a un año único.
4. **`cue.plot(subplots=True, figsize=(12, 20))`** para explorar las 30 variables rápidamente.
5. **`.resample("ME").mean()`** para análisis mensual (T de neutralidad, gradientes).
6. Acceder a columnas no-aliased con **corchetes**: `cue["Dew Point Temperature"]`.

Procedimiento detallado en [[../procedures/EDA-Archivo-EPW]].

## Limitaciones observadas

- El alias **no cubre** todas las columnas (solo el catálogo conocido).
- `read_epw` **no tiene parámetro `year`** — el manejo del año es responsabilidad del usuario.
- El bug del 29-feb no está resuelto en el paquete (a diferencia de lo que se pensó inicialmente).

## Clase relacionada

- [[../classes/005-AnalisisSimulacionesPython]] — primera demostración del flujo (con el bug en vivo del 29-feb)

## Herramienta

- [[../tools/iertools]] — paquete que se usa

## Notebook anterior

- [[001_EDA]] — EDA de la simulación SQL (mismo patrón sin la complicación del año)
