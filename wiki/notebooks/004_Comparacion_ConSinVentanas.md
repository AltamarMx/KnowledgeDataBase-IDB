---
title: 004 — Comparación caso base vs con protecciones
type: notebook
notebook: 004_Comparacion_ConSinVentanas
fuente: raw/notebooks/004_Comparacion_ConSinVentanas.ipynb
fecha_ingesta: 2026-05-02
tags: [notebook, comparacion, caso-base, sombreamiento, sunlit-fraction, mir-face, iertools]
aliases: [004 Comparacion, comparacion con sin ventanas, libreta sombreamiento]
clase_relacionada: 008
---

# 004 — Comparación caso base vs con protecciones

## Metadatos

- **Notebook:** `004_Comparacion_ConSinVentanas.ipynb`
- **Fuente:** `raw/notebooks/004_Comparacion_ConSinVentanas.ipynb`
- **Clase relacionada:** [[../classes/008-ShadingVentanas]]
- **Objetivo:** Aplicar el flujo de comparación caso base vs variante con [[../concepts/Sunlit-Fraction|Sunlit Fraction]] para validar el efecto de aleros y parteluces.

## Resumen

Libreta que **resuelve el bug de la clase 007** y aplica los aprendizajes de la clase 008:

1. **Función reusable `carga_datos(f)`** — patrón anti-anti-patrón de la clase 007.
2. **List comprehension sobre prefijos** (`Is_`, `Ti_`) para filtrar columnas.
3. **Comparación gráfica caso base vs con protecciones** sobre el muro padre (`FACE 6`) — no sobre la ventana, porque la ventana no muestra el efecto en `Incident Solar Radiation`.
4. **Panel con `Sunlit Fraction`** — la variable que captura el efecto de la protección sobre la radiación directa.
5. **Hallazgo nuevo**: aparición de `Mir-FACE` (mirror surfaces) en el output cuando se usa `*` como Key Value.

## Imports

```python
import pandas as pd
import matplotlib.pyplot as plt
from iertools.read import read_sql
from dateutil.parser import parse
```

Plantilla idéntica a [[001_EDA]] y [[003_EDA]].

## Función reusable de carga

Patrón formalizado en clase 007 — definir una función para no repetir el código de carga + renombrado.

```python
def carga_datos(f):
    sinv = read_sql(f, alias=True).data
    nombres = {
        'TECHO:Surface Outside Face Incident Solar Radiation Rate per Area (W/m2)':  'Is_TECHO',
        'VNORTE:Surface Outside Face Incident Solar Radiation Rate per Area (W/m2)': 'Is_VNORTE',
        'VOESTE:Surface Outside Face Incident Solar Radiation Rate per Area (W/m2)': 'Is_VOESTE',
    }
    sinv.rename(columns=nombres, inplace=True)
    return sinv
```

### Características del patrón

- **`rename(inplace=True)`** modifica el DataFrame in-place — no devuelve uno nuevo.
- **Diccionario fijo `nombres`** dentro de la función — útil cuando todas las simulaciones del proyecto comparten estos nombres custom.
- **El nombre genérico `sinv`** dentro de la función no importa — es local. La función la reusa el caller con cualquier nombre.

> **Nota**: aunque la variable interna se llama `sinv` (sin ventanas), la función se usa también para `conv` (con ventanas). Pequeña inconsistencia de naming que no afecta el comportamiento — la lección de la clase 007: usar nombres genéricos dentro de funciones reusables.

## Cargar dos simulaciones

```python
f = "../osm/005_CBs/run/eplusout.sql"
sinv = carga_datos(f)

f = "../osm/006_Protecciones/run/eplusout.sql"
conv = carga_datos(f)
```

- **`005_CBs`** = caso base (sin protecciones).
- **`006_Protecciones`** = caso con aleros + parteluces aplicados.

Estructura de proyecto típica de [[../concepts/Estudio-Parametrico]].

## Hallazgo — `Mir-FACE` (mirror surfaces)

Las columnas del caso con protecciones revelan algo nuevo:

```
'FACE 1', 'FACE 2', 'FACE 3', 'FACE 4', 'FACE 6', 'FACE 8',
'FACE 13', 'FACE 15', 'FACE 16', 'FACE 18', 'FACE 19', 'FACE 20',
'FACE 21', 'FACE 22', 'FACE 23',
'Mir-FACE 8', 'Mir-FACE 18', 'Mir-FACE 19', 'Mir-FACE 20',
'Mir-FACE 22', 'Mir-FACE 23',
'SURFACE 1'
```

**Total: 31 columnas** (vs ~10 en el caso base sin protecciones).

### Qué son las `Mir-FACE`

Cuando E+ procesa **superficies de sombramiento** (aleros, parteluces, vecinos), internamente crea **superficies espejo** (mirror surfaces) para el cálculo de intercambio radiativo. Cada cara que necesita el cálculo del "lado opuesto" genera su mirror.

Aparecen en el output cuando:

- Se solicita una variable de superficie con `Key Value = *` (todas las superficies).
- El motor las incluye porque, técnicamente, son superficies del modelo.

**No están en el OSM**. No son superficies dibujadas por el usuario. Son **construcciones internas** del solver de E+.

### Qué hacer con ellas

| Caso | Acción |
|------|--------|
| **Análisis comparativo simple** | Ignorarlas — usar columnas con nombres específicos (FACE N o renombradas con alias custom) |
| **Auditoría avanzada del sombreamiento** | Estudiar — los `Mir-FACE` corresponden a las superficies específicas con sombras |
| **Reducir explosión de columnas** | **No usar `*` como Key Value** — pedir cada variable con el nombre específico de la superficie |

Detalle de la advertencia en [[../procedures/Solicitar-Output-Variables-Measures]].

## Filtrado de columnas con prefijos

Patrón documentado en clase 007 — selección por prefijo:

```python
columnas = conv.columns
Iss = [columna for columna in columnas if "Is_" in columna]
Tis = [columna for columna in columnas if "Ti_" in columna]
```

Resultado:

```python
Iss → ['Is_TECHO', 'Is_VNORTE', 'Is_VOESTE']
Tis → ['Ti_ESTE',  'Ti_OESTE']
```

> Aquí brilla la convención: si los nombres custom siguen un prefijo consistente, el filtrado por list comprehension recupera solo las columnas relevantes y deja fuera todas las `FACE N` y `Mir-FACE N`.

## Plot de un solo panel — radiación sobre muro padre

Patrón crítico de la clase 008: **la radiación incidente sobre una sub-superficie (ventana) no refleja sombreamiento**, pero sobre el muro padre (`FACE 6` = pared norte en este modelo) **sí**.

```python
fig, ax = plt.subplots(figsize=(12, 4))
f1 = parse("2006-08-27")
f2 = f1 + pd.Timedelta("1D")

ax.plot(sinv['FACE 6:Surface Outside Face Incident Solar Radiation Rate per Area (W/m2)'],
        "g-",  label="Is_PNORTE_sv")
ax.plot(conv['FACE 6:Surface Outside Face Incident Solar Radiation Rate per Area (W/m2)'],
        "go",  label="Is_PNORTE_cv")

ax.set_xlim(f1, f2)
ax.legend()
```

### Convención de etiquetas

- **`PNORTE`** = pared norte (no la ventana). El usuario nombra `Is_P*` para distinguir de `Is_V*` (ventanas).
- **`sv`** = sin ventanas (mejor: sin protecciones; el caso base tiene ventanas pero no aleros).
- **`cv`** = con ventanas (mejor: con protecciones).

### Patrón color + marker distinto al sólido/dashed

```python
"g-"   # línea sólida verde — caso base
"go"   # puntos verdes circulares — caso con protección
```

**Variante** del patrón documentado en [[../procedures/Comparar-Simulaciones-Python]]:

| Patrón documentado | Patrón del notebook |
|---------------------|----------------------|
| color = ubicación | color = mismo (variable comparable) |
| linestyle sólido = base | linestyle sólido = base |
| linestyle dashed = variante | **marker `o` = variante** (sin línea) |

Razón posible: cuando las dos series son muy parecidas, los **marcadores** dejan ver claramente las dos curvas en la misma área del plot. Una línea dashed sobre una sólida puede confundirse visualmente.

> Trade-off: marcadores son legibles cuando hay pocos puntos; en una serie temporal de un día con paso de 10 minutos hay 144 puntos — los marcadores se densifican. Ambos patrones son válidos según el contexto.

## Plot de doble panel — comparación + Sunlit Fraction

Patrón completo de la clase 008:

```python
fig, ax = plt.subplots(2, 1, figsize=(12, 4), sharex=True)
f1 = parse("2006-08-27")
f2 = f1 + pd.Timedelta("1D")

# Panel 1: radiación incidente sobre muro padre, ambos casos
ax[0].plot(sinv['FACE 6:...'], "g-", label="Is_PNORTE_sv")
ax[0].plot(conv['FACE 6:...'], "go", label="Is_PNORTE_cv")

# Panel 2: Sunlit Fraction (solo del caso con protección)
ax[1].plot(conv['VNORTE:Surface Outside Face Sunlit Fraction ()'],
           label="fraccion sombreamiento")

ax[0].set_xlim(f1, f2)
ax[0].legend()
```

### Lectura esperada

- **Panel superior**: la curva de `cv` (con protección) debe estar **por debajo** de `sv` durante las horas de sol — el alero/parteluz bloquea radiación que llegaría al muro norte.
- **Panel inferior**: `VNORTE:Sunlit Fraction` debe **caer a cero** durante las horas en que el alero protege a la ventana norte. Cuando vale 1, no hay protección efectiva (el sol llega directo).

> El día elegido (27 de agosto) es **cercano al solsticio de verano** + post-solsticio en hemisferio norte → momento donde el sol pasa muy alto, y un muro norte casi no recibe directa. Buen día para ver el efecto del parteluz sobre cualquier directa residual.

## Variables que pidió la simulación

A partir de la lista de columnas se infiere que el caso base / con protecciones pidió:

| Variable | Key Value |
|----------|-----------|
| `Site Outdoor Air Drybulb Temperature` | `*` → alias `To` |
| `Zone Mean Air Temperature` | `*` → alias `Ti_<zona>` |
| `Zone Air Temperature` | `*` (no aliased — frecuencias mezcladas, ver 003) |
| `Zone Air Relative Humidity` | `*` (no aliased) |
| `Surface Outside Face Incident Solar Radiation Rate per Area` | **`*`** ← genera explosión incluyendo `Mir-FACE` |
| `Surface Outside Face Sunlit Fraction` | `VNORTE` (key específico) |

> **Lección**: para evitar explosión usar Key Value específico (`Techo`, `vNorte`, `vOeste`) en lugar de `*`. Procedimiento en [[../procedures/Solicitar-Output-Variables-Measures]].

## Patrones para reusar

1. **Función `carga_datos(f)`** con `rename` in-place para todas las simulaciones del proyecto.
2. **List comprehension con prefijos** (`Is_`, `Ti_`) para filtrar columnas relevantes.
3. **Comparación sobre el muro padre**, no la ventana, cuando se evalúa sombreamiento (clase 008).
4. **`Sunlit Fraction`** como variable principal de auditoría — pedirla con Key Value específico.
5. **Día representativo cerca del solsticio** para visualizar efectos de aleros/parteluces.
6. **Recorte a 1 día** (`Timedelta("1D")`) para ver detalle horario.
7. **Marker `o` vs línea sólida** como alternativa al sólido/dashed.

## Limitaciones observadas

- **Explosión de columnas** por usar `*` como Key Value en `Surface Outside Face Incident Solar Radiation`. 31 columnas en lugar de 5-7.
- **No calcula reducción cuantitativa** — solo grafica. La métrica de % de bloqueo se calcula con `(sinv.col * dt).sum() - (conv.col * dt).sum()`.
- **Diccionario de renombrado limitado**: solo cubre `Is_TECHO`, `Is_VNORTE`, `Is_VOESTE`. Las `FACE N` quedan con nombre largo.

## Clase relacionada

- [[../classes/008-ShadingVentanas]] — descubrimiento de Sunlit Fraction y truco del muro padre

## Procedimientos relacionados

- [[../procedures/Auditar-Sombreamiento-Ventanas]] — flujo completo formal
- [[../procedures/Comparar-Simulaciones-Python]] — patrones de plotting comparativo
- [[../procedures/Solicitar-Output-Variables-Measures]] — Key Value específico

## Notebooks anteriores

- [[001_EDA]] — patrón básico de carga
- [[003_EDA]] — caso base sin la complicación de comparación
