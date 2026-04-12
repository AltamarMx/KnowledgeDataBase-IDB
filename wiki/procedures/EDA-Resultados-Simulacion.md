# EDA de Resultados de Simulación

## Metadatos
- **Fuente:** `raw/notebooks/001_EDA.ipynb`
- **Tipo:** Análisis exploratorio de datos (EDA) de simulación
- **Modelo analizado:** Primer cubo (`Mi_primer_cubo_002`)

---

## Objetivo

Demostrar el flujo básico de análisis exploratorio de resultados de una simulación en EnergyPlus usando Python: cargar datos desde el SQL, inspeccionar sistemas constructivos y graficar temperaturas y radiación solar.

---

## Flujo de trabajo

### 1. Importaciones

```python
import pandas as pd
import matplotlib.pyplot as plt
from iertools.read import read_sql
from dateutil.parser import parse
```

> **Nota:** el paquete se importa como `iertools` (IER = Instituto de Energías Renovables). Es el mismo paquete referido como `ear_tools` en otras clases.

### 2. Carga de datos

```python
f = "../Osm/Mi_primer_cubo_002/run/eplusout.sql"
cubo = read_sql(f, alias=True)
data_cubo = cubo.data
```

- `read_sql` devuelve un objeto con `.data` (DataFrame) y `.construction_systems`
- `alias=True` renombra columnas largas a nombres cortos (`Ti_CUBO`, `Id`, `Ib`, `To`)
- La columna de radiación en el techo no se renombra automáticamente: `TECHO:Surface Outside Face Incident Solar Radiation Rate per Area (W/m2)`

### Variables cargadas

| Columna | Significado |
|---------|-------------|
| `Ti_CUBO` | Temperatura interior de la zona "CUBO" (°C) |
| `To` | Temperatura exterior (°C) |
| `Id` | Radiación solar difusa (W/m²) |
| `Ib` | Radiación solar directa/beam (W/m²) |
| `TECHO:...Incident Solar Radiation...` | Radiación solar global incidente sobre el techo (W/m²) |

- Resolución temporal: cada 10 minutos (52,560 filas para un año)
- Periodo: 2006-01-01 a 2007-01-01

### 3. Inspección de sistemas constructivos

```python
sc = cubo.construction_systems
cubo.get_construction(sc)
```

Dos sistemas constructivos en el modelo:

| Sistema | Material | Conductividad (W/m·K) | Densidad (kg/m³) | Calor específico (J/kg·K) | Espesor (m) | Absorptancia solar |
|---------|----------|----------------------|-------------------|---------------------------|-------------|-------------------|
| CDA15CMA0P7 | Concreto de alta densidad | 2.0 | 2,500 | 1,400 | 0.15 | 0.7 |
| LADRILLO14CMA0P7 | Ladrillo | 1.4 | 1,400 | 1,000 | 0.14 | 0.7 |

Ambos tienen absorptancia visible 0.7, emitancia térmica 0.9 (interior).

### 4. Visualización: temperaturas y radiación

```python
fig, ax = plt.subplots(2, 1, figsize=(12, 4), sharex=True)
f1 = parse("2006-03-13")
f2 = f1 + pd.Timedelta("7D3h50min")

ax[0].plot(data_cubo.Ti_CUBO, label="Ti")
ax[0].plot(data_cubo.To, label="To")
ax[1].plot(data_cubo.Id, label="Id")
ax[1].plot(data_cubo.Ib, label="Ib")
ax[1].plot(data_cubo["TECHO:..."], label="Ig")

ax[0].legend(); ax[1].legend()
ax[1].set_xlim(f1, f2)
```

**Gráfica superior (temperaturas):** muestra 7 días a partir del 13 de marzo de 2006. La temperatura interior (Ti) sigue a la exterior (To) con un desfase temporal y amplitud amortiguada por la masa térmica del concreto y ladrillo. Ti oscila por encima de To durante la noche (inercia térmica libera calor) y puede superar To durante el día por ganancia solar.

**Gráfica inferior (radiación):** muestra tres componentes de radiación solar sobre el techo:
- **Id (difusa):** perfil suave tipo campana, presente incluso en días nublados
- **Ib (directa/beam):** picos agudos en días despejados, cae a cero en días nublados
- **Ig (global sobre techo):** envolvente de ambas, siempre ≥ Id. Al ser una superficie horizontal, recibe la máxima radiación global

---

## Patrones observados

- La **temperatura interior siempre es mayor** que la exterior en este modelo simple (cubo sin ventilación, solo conducción + radiación)
- La **radiación global sobre el techo** (Ig) es consistentemente la más alta, como se espera de una superficie horizontal en estas latitudes
- Los días nublados se identifican por la caída de Ib a ~0 mientras Id se mantiene
- El desfase térmico entre Ti y To refleja la masa térmica de los materiales (concreto 15cm + ladrillo 14cm)

---

## Herramientas y funciones usadas

- [[Python]]: pandas, matplotlib, dateutil
- `iertools.read.read_sql` — carga SQL con alias automáticos
- `cubo.get_construction()` — inspección de sistemas constructivos
- `pd.Timedelta` — definición de ventanas temporales
- `sharex=True` — ejes X compartidos entre subplots

## Conexiones

- [[003-MiPrimeraSimulacion]] — El modelo "Mi primer cubo" que genera estos resultados
- [[005-AnalisisSimulacionesPython]] — Flujo de análisis más completo con ear_tools
- [[Analizar-Resultados-Python]] — Procedimiento general de análisis
