# EDA de Archivo Climático EPW

## Metadatos
- **Fuente:** `raw/notebooks/002_EDA_EPW.ipynb`
- **Tipo:** Análisis exploratorio de archivo climático EPW
- **Archivo climático:** `MEX_MOR_Cuernavaca-Matamoros.Intl.AP.767260_TMYx.2009-2023.epw`

---

## Objetivo

Demostrar cómo cargar y explorar un archivo EPW (clima) usando `iertools.read.read_epw`, visualizar todas las variables climáticas del año meteorológico típico y calcular temperaturas promedio mensuales (necesarias para el modelo adaptativo de confort).

---

## Flujo de trabajo

### 1. Importaciones

```python
import pandas as pd
import matplotlib.pyplot as plt
from iertools.read import read_epw
```

### 2. Carga del EPW

```python
f = "../EPW/MEX_MOR_Cuernavaca-Matamoros.Intl.AP.767260_TMYx.2009-2023.epw"
cue = read_epw(f, alias=True)
```

- `read_epw` devuelve un DataFrame con todas las variables del EPW
- `alias=True` renombra columnas principales (`To` para temperatura exterior, `Ig` para radiación global, `Ib` para directa, `RH` para humedad relativa, `P` para presión)
- 8,759 filas (horas del año, resolución horaria)

### 3. Manejo del 29 de febrero

```python
cue = cue[~((cue.index.month == 2) & (cue.index.day == 29))]
cue.index = cue.index.map(lambda x: x.replace(year=2023))
```

- Algunos EPW incluyen 29 de febrero (año bisiesto) — se filtra para evitar inconsistencias
- Se reasigna el año a 2023 para tener un índice temporal consistente
- **Nota:** este es el bug mencionado en la clase 007 que `ear_tools` no maneja automáticamente

### 4. Variables disponibles en el EPW

El DataFrame contiene 30 columnas. Las principales:

| Variable (alias) | Significado | Unidades |
|-------------------|-------------|----------|
| `To` | Temperatura exterior (bulbo seco) | °C |
| `Dew Point Temperature` | Temperatura de punto de rocío | °C |
| `RH` | Humedad relativa | % |
| `P` | Presión atmosférica | Pa |
| `Ig` | Radiación solar global horizontal | Wh/m² |
| `Ib` | Radiación solar directa normal | Wh/m² |
| `Id` | Radiación solar difusa horizontal | Wh/m² |
| `Horizontal Infrared Radiation Intensity` | Radiación infrarroja horizontal | Wh/m² |
| `Wind Speed` | Velocidad del viento | m/s |
| `Wind Direction` | Dirección del viento | grados |
| `Albedo` | Albedo del suelo | — |
| `Precipitable Water` | Agua precipitable | mm |

### 5. Visualización panorámica

```python
cue.plot(subplots=True, figsize=(12, 20))
```

Genera una gráfica con un subplot por cada variable del EPW (30 paneles). Es una exploración rápida para identificar:
- **Patrones estacionales** — temperaturas más altas en abril-mayo, más bajas en diciembre-enero
- **Radiación solar** — máximos en primavera (antes de lluvias), mínimos en época de lluvias (junio-septiembre)
- **Humedad y precipitación** — temporada de lluvias claramente visible
- **Anomalías** — valores atípicos, datos faltantes, flags de incertidumbre

### 6. Temperatura promedio mensual

```python
cue.To.resample("ME").mean()
```

Resultado para Cuernavaca:

| Mes | To promedio (°C) |
|-----|-----------------|
| Enero | 19.2 |
| Febrero | 20.6 |
| Marzo | 23.3 |
| Abril | 25.1 |
| Mayo | 24.9 |
| Junio | 23.1 |
| Julio | 22.2 |
| Agosto | 22.0 |
| Septiembre | 21.0 |
| Octubre | 21.5 |
| Noviembre | 20.2 |
| Diciembre | 18.8 |

- El mes más cálido es **abril** (25.1°C), no julio/agosto — patrón típico de climas con temporada de lluvias
- La oscilación anual es moderada (~6.3°C entre el mes más frío y el más cálido) — clima templado subtropical
- Estos promedios mensuales son la entrada directa para calcular la **temperatura de neutralidad** del modelo adaptativo de confort: `T_n = 0.54 × To_promedio_mensual + 13.5`

---

## Patrones observados (Cuernavaca)

- **Clima templado subtropical** a ~1,500 msnm — sin extremos de calor ni frío severo
- **Temporada seca** (nov-may): mayor radiación solar, temperaturas más altas en abril-mayo
- **Temporada de lluvias** (jun-oct): menor radiación, mayor humedad, temperaturas ligeramente menores
- La presión atmosférica (~86,600-87,500 Pa) refleja la altitud de Cuernavaca
- Albedo constante (~0.13-0.14), sin nieve

---

## Herramientas y funciones usadas

- [[Python]]: pandas (resample, index manipulation), matplotlib
- `iertools.read.read_epw` — carga EPW con alias automáticos
- `.resample("ME").mean()` — promedio mensual (clave para modelo de confort)
- `.plot(subplots=True)` — visualización rápida de todas las variables

## Conexiones

- [[TMY]] — El EPW contiene un Año Meteorológico Típico
- [[Confort-Termico]] — Los promedios mensuales de To alimentan el modelo adaptativo de Humphreys-Nicol
- [[005-AnalisisSimulacionesPython]] — Uso de ear_tools/iertools para carga de datos
- [[EDA-Resultados-Simulacion]] — Libreta complementaria (001_EDA) que analiza resultados de simulación
