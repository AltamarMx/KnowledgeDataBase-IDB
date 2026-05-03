---
title: 005 — Revisión de caso con AC (T constante)
type: notebook
notebook: 005_revision_1setpoint
fuente: raw/notebooks/005_revision_1setpoint.ipynb
fecha_ingesta: 2026-05-02
tags: [notebook, aire-acondicionado, ideal-loads, setpoint, energia, comparacion-zonas, iertools]
aliases: [005 setpoint, libreta AC, libreta ideal loads]
clase_relacionada: 009
---

# 005 — Revisión de caso con AC (T constante)

## Metadatos

- **Notebook:** `005_revision_1setpoint.ipynb`
- **Fuente:** `raw/notebooks/005_revision_1setpoint.ipynb`
- **Clase relacionada:** [[../classes/009-AireAcondicionadoSetPoints]]
- **Objetivo:** Verificar el caso `007_CB_aa` (caso base + Ideal Air Loads en modo T constante 20 °C) y cuantificar la energía de enfriamiento mensual y anual por zona.

## Resumen

Libreta corta que aplica los aprendizajes de la clase 009. Tres bloques:

1. **Cargar el caso con AC** y filtrar el año.
2. **Verificar visualmente** que la T se mantiene constante (modo `heating = cooling = 20 °C`).
3. **Cuantificar la energía** mensual con barras y total anual por zona.

Hallazgo central: la **zona oeste consume ~4× más energía de cooling** que la este — coherente con la radiación solar incidente en orientaciones oeste durante la tarde.

## Imports

```python
import pandas as pd
import matplotlib.pyplot as plt
from iertools.read import read_sql
from dateutil.parser import parse
```

Plantilla idéntica a las libretas anteriores.

## Cargar el caso con AC y filtrar el año

```python
f = "../osm/007_CB_aa/run/eplusout.sql"
aa = read_sql(f, alias=True).data
aa = aa[aa.index.year == 2006]
```

### Patrón nuevo — filtro `aa.index.year == 2006`

E+ reporta el **último timestep del año** con timestamp `2007-01-01 00:00:00` (cierre del año, no parte del año siguiente). Sin filtrar, el DataFrame tiene **52,560 filas** (52,559 de 2006 + 1 de 2007).

El filtro:

```python
aa = aa[aa.index.year == 2006]
```

Reduce a **52,559 filas** — solo 2006.

### Por qué importa

- Las **agregaciones anuales** (`resample("YE").sum()`) con el timestep extra producen un resultado parcial extra para 2007 con un solo dato.
- Los plots con `set_xlim` pueden mostrar un punto suelto en 2007 que rompe la escala.

> **Buena práctica**: tras `read_sql` siempre filtrar por año si se trabaja con un año completo:
>
> ```python
> df = read_sql(f, alias=True).data
> df = df[df.index.year == AÑO_DE_LA_SIMULACION]
> ```

## Estructura del DataFrame retornado

```
[52559 rows x 9 columns]
```

9 columnas — el caso base con AC pidió:

| Columna | Origen |
|---------|--------|
| `ESTE IDEAL LOADS AIR SYSTEM:Zone Ideal Loads Zone Total Cooling Energy (J)` | Sin alias |
| `ESTE:Zone Air Relative Humidity (%)` | Sin alias |
| `ESTE:Zone Air Temperature (C)` | Sin alias (frecuencia mezclada — ver [[003_EDA]]) |
| `Ti_ESTE` | Aliased (`Zone Mean Air Temperature`) |
| `To` | Aliased |
| `OESTE IDEAL LOADS AIR SYSTEM:Zone Ideal Loads Zone Total Cooling Energy (J)` | Sin alias |
| `OESTE:Zone Air Relative Humidity (%)` | Sin alias |
| `OESTE:Zone Air Temperature (C)` | Sin alias |
| `Ti_OESTE` | Aliased |

> **Nota**: las variables de Ideal Air Loads **no reciben alias automático** de `iertools`. Los nombres son largos. Conviene renombrar con un diccionario en una función `carga_datos()` reusable (patrón de la clase 007 — ver [[../procedures/Comparar-Simulaciones-Python]]).

## Verificar la T interior — primera gráfica

```python
fig, ax = plt.subplots(figsize=(12, 3))

ax.plot(aa.Ti_ESTE,  "r.", label="este")
ax.plot(aa.Ti_OESTE, "k--", label="oeste")

ax.set_ylim(15, 30)
ax.grid()
ax.legend()
```

### Resultado esperado

- **Ambas zonas en línea recta a 20 °C** todo el año — confirma el modo de AC con T constante (heating = cooling = 20).
- Sin oscilación.

### `set_ylim(15, 30)` — workaround del bug de matplotlib

Cuando una serie es **constante**, matplotlib elige un ylim automático que se ve raro (el "0 padding" colapsa). Forzar manualmente con `set_ylim(15, 30)` resuelve.

> Documentado en clase 009 ("matplotlib se equivoca en el zoom cuando una variable no varía"). Workaround documentado en [[../procedures/Analizar-Resultados-Python]].

### Patrón rojo punto + negro dashed

```python
"r."   # rojo, marker punto, sin línea
"k--"  # negro, línea dashed, sin marker
```

Otra variante del patrón color/style observado a lo largo de las libretas. La capacidad de matplotlib para combinar `color + linestyle + marker` en un string corto (`"r-"`, `"go"`, `"k--"`, `"r."`) es flexible pero requiere conocer la sintaxis.

## Plot de doble panel — T + energía

```python
fig, (ax, ax2) = plt.subplots(2, 1, figsize=(12, 3), sharex=True)

f1 = parse("2006-04-17")
f2 = f1 + pd.Timedelta("3D")

ax.plot(aa.Ti_ESTE, "r.", label="este")

ax2.plot(aa["ESTE IDEAL LOADS AIR SYSTEM:Zone Ideal Loads Zone Total Cooling Energy (J)"])

ax.set_ylim(15, 30)
ax.set_xlim(f1, f2)
ax.grid()
ax.legend()
```

Día elegido: **17 de abril** (pico cálido en Cuernavaca según [[002_EDA_EPW]] que mostró abril como el mes más cálido).

### Lectura esperada

- **Panel superior**: `Ti_ESTE` mantenida en 20 °C (el AC compensa).
- **Panel inferior**: la energía de cooling **dispara** en horas con sol y picos de carga térmica — durante el día. De noche cae a cerca de 0.

### Acceso a columna sin alias

Como `ESTE IDEAL LOADS AIR SYSTEM:...` no recibe alias, hay que usar **corchetes**:

```python
aa["ESTE IDEAL LOADS AIR SYSTEM:Zone Ideal Loads Zone Total Cooling Energy (J)"]
```

Largo y propenso a errores de tipeo. Mejor renombrar con un diccionario al cargar — ver [[../procedures/Comparar-Simulaciones-Python]].

## Análisis energético mensual — barras

```python
zona = "ESTE IDEAL LOADS AIR SYSTEM:Zone Ideal Loads Zone Total Cooling Energy (J)"

fig, ax = plt.subplots(figsize=(12, 3))

ax.bar(
    range(13),
    aa[zona].resample("ME").sum()
)
```

### Resultado observado

12 barras (una por mes) con perfil:

| Mes | Energía cooling (J) | Comentario |
|-----|---------------------|------------|
| Enero | ~2.7e8 | Mínimo |
| Febrero | ~3.9e8 | |
| Marzo | ~6.1e8 | |
| **Abril** | **~9.1e8** | **Pico** |
| **Mayo** | **~9.2e8** | **Pico** |
| Junio | ~4.9e8 | Llegan las lluvias |
| Julio | ~4.2e8 | |
| Agosto | ~3.4e8 | |
| Septiembre | ~3.3e8 | |
| Octubre | ~5.0e8 | |
| Noviembre | ~3.3e8 | |
| Diciembre | ~2.8e8 | |

Patrón coherente con el clima de Cuernavaca: pre-monzón cálido (abril-mayo), monzón nublado (junio-septiembre), seco templado (resto del año).

### Bug observado — `range(13)` para 12 meses

```python
ax.bar(range(13), aa[zona].resample("ME").sum())
```

`range(13)` produce **13 posiciones X** (0 a 12), pero `resample("ME")` produce **12 valores**. Matplotlib alinea — la barra 13 queda vacía o sale en blanco. Bug visual menor.

**Fix**:

```python
ax.bar(range(12), aa[zona].resample("ME").sum())
# o mejor:
ax.bar(range(1, 13), aa[zona].resample("ME").sum())   # 1-12 para etiquetas E-D
```

### Etiquetas de meses

El plot del notebook usa los índices 0-12 sin etiquetas legibles. Para reportes:

```python
ax.set_xticks(range(1, 13))
ax.set_xticklabels(["E", "F", "M", "A", "M", "J", "J", "A", "S", "O", "N", "D"])
```

## Total anual por zona

```python
este  = "ESTE IDEAL LOADS AIR SYSTEM:Zone Ideal Loads Zone Total Cooling Energy (J)"
oeste = "OESTE IDEAL LOADS AIR SYSTEM:Zone Ideal Loads Zone Total Cooling Energy (J)"

aa[[este, oeste]].resample("YE").sum().values
```

Resultado:

```
array([[5.78833154e+09, 2.30698337e+10]])
```

| Zona | Energía anual cooling | kWh equivalente |
|------|------------------------|-----------------|
| ESTE | 5.79 GJ | ~1,608 kWh |
| OESTE | **23.07 GJ** | **~6,408 kWh** |

**OESTE consume ~4× más** que ESTE.

### Por qué tanto desbalance

- La **zona OESTE** recibe radiación directa fuerte durante la **tarde** (el clima de Cuernavaca tiene tardes muy soleadas).
- La **zona ESTE** recibe directa solo en la mañana, cuando el aire exterior aún está fresco — la carga térmica neta es menor.
- Las **ventanas** orientadas oeste son las más problemáticas (sol bajo y oblicuo, alero horizontal no protege — ver [[../concepts/Trayectoria-Solar]]).

> Confirmación física: este es el patrón típico que el profesor reitera — **evitar ventanas este/oeste** en climas cálidos. Ver [[../classes/006-DosZonasTermicasVentanasAleros]].

### Patrón de extracción de escalares

```python
aa[[este, oeste]].resample("YE").sum().values.flatten()
```

`values` convierte el DataFrame a array NumPy. `.flatten()` aplana de forma `(1, 2)` a `(2,)` — útil para pasar como argumento de `ax.bar`.

```python
ax.bar([1, 2], aa[[este, oeste]].resample("YE").sum().values.flatten())
```

## Conversión de Joules a kWh para reportes

```python
# 1 kWh = 3.6 × 10^6 J
energia_kWh = energia_J / 3.6e6
```

Los Joules son la unidad nativa de E+. Para reportes legibles:

| Magnitud | Unidad | Valor típico anual zona |
|----------|--------|--------------------------|
| GJ | 5.79 | OK para artículo técnico |
| MJ | 5,790 | Verboso |
| **kWh** | **1,608** | **Comprensible para audiencia general** |

## Patrones para reusar

1. **`df = df[df.index.year == AÑO]`** después de cargar — eliminar el timestep extra de 2007.
2. **`set_ylim(min, max)`** cuando una variable es constante (T en setpoint).
3. **`resample("ME").sum()`** para totales mensuales (energía).
4. **`resample("YE").sum().values.flatten()`** para totales anuales como array.
5. **Conversión J → kWh** dividiendo por 3.6e6 para reportes.
6. **Renombrar variables del AC** con diccionario al cargar (no se hizo en este notebook — antipatrón).

## Limitaciones observadas

- **Sin renombrado de variables del AC** — los nombres largos hacen difícil el código.
- **`range(13)` en lugar de `range(12)`** — bug visual menor.
- **No hay etiquetas de meses** en el bar plot — difícil de leer sin contexto.
- **No compara con caso sin AC** — pendiente para libretas posteriores. Solo audita el caso AC.

## Clase relacionada

- [[../classes/009-AireAcondicionadoSetPoints]] — Aire Acondicionado Ideal y schedules de setpoint

## Procedimiento relacionado

- [[../procedures/Configurar-Aire-Acondicionado-Ideal]] — cómo configurar este modo en Open Studio
- [[../procedures/Analizar-Resultados-Python]] — patrones de análisis (resample, bar plots)

## Notebooks anteriores

- [[004_Comparacion_ConSinVentanas]] — patrón de comparación de simulaciones (no aplicado aquí)
- [[003_EDA]] — caso base sin AC (la T flotaba)
