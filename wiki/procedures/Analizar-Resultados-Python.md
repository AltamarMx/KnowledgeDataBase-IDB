# Analizar Resultados de Simulación con Python

Flujo de trabajo para cargar, verificar y graficar resultados de simulaciones EnergyPlus usando Python, pandas y ear_tools.

## Requisitos previos

- Simulación corrida con variables de salida configuradas ([[Configurar-Variables-Salida]])
- Entorno Python configurado con uv: `uv add pandas jupyter notebook matplotlib ear_tools`

## Paso a paso

### 1. Iniciar Jupyter Notebook

```bash
uv run jupyter notebook
```

### 2. Imports estándar

```python
import pandas as pd
import matplotlib.pyplot as plt
from ear_tools.read import read_sql
```

### 3. Cargar la simulación

```python
f = "../osm/mi_proyecto/run/eplusout.sql"
sim = read_sql(f, alias=True)
```

El objeto `sim` contiene:
- `sim.data` — DataFrame con todas las variables solicitadas (índice datetime)
- `sim.construction_systems` — lista de sistemas constructivos usados

### 4. Verificar sistemas constructivos (QA)

```python
sc = sim.construction_systems
sim.get_constructions(sc)
```

Revisar: conductividad, densidad, espesor, absortancia, orden de capas (exterior → interior). Es lo primero que se debe hacer como buena práctica — confirmar que las propiedades térmicas son correctas.

### 5. Explorar los datos

```python
df = sim.data
df.head()
df.columns  # verificar nombres de columnas (alias)
```

Con alias activado, las columnas se renombran:
- `Ti_<zona>` — temperatura interior
- `To` — temperatura exterior
- `Id` — radiación difusa
- `Ib` — radiación directa
- `Is` — radiación incidente en superficie

### 6. Graficar temperaturas y radiación

```python
fig, ax = plt.subplots(2, 1, sharex=True, figsize=(12, 4))

ax[0].plot(df['Ti_cubo'], label='Ti')
ax[0].plot(df['To'], label='To')
ax[0].legend()
ax[0].set_ylabel('Temperatura [°C]')

ax[1].plot(df['Id'], label='Difusa')
ax[1].plot(df['Ib'], label='Directa')
ax[1].legend()
ax[1].set_ylabel('Radiación [W/m²]')
```

### 7. Hacer zoom temporal

```python
from dateutil.parser import parse

f1 = parse("2006-03-13")
f2 = f1 + pd.Timedelta(days=7)
ax[1].set_xlim(f1, f2)  # aplica a ambos ejes por sharex
```

**Tip:** usar `pd.Timedelta(days=7, hours=3, minutes=50)` para intervalos arbitrarios.

## Buenas prácticas

- **Siempre incluir referencia climática** (To, radiación) junto con temperatura interior
- **No usar doble eje Y** en presentaciones — usar subplots
- **Restart & Run All** periódicamente para verificar que el notebook es reproducible
- **Rutas relativas** desde la ubicación del notebook
- **Nunca `pip install`** dentro del notebook — usar `uv add` desde la terminal

## Aparece en

- [[005-AnalisisSimulacionesPython]] — Demostración completa del flujo
