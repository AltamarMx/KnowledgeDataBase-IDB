---
title: Comparar simulaciones (caso base vs variantes) en Python
type: procedimiento
tags: [procedimiento, python, iertools, comparacion, caso-base, variantes, plotting]
aliases: [comparar simulaciones, caso base variante, plotting comparativo]
clases: [007, 008]
updated: 2026-05-02
---

# Comparar simulaciones (caso base vs variantes) en Python

Procedimiento para cargar **dos o más** simulaciones en una libreta, alinear sus columnas, y producir gráficas comparativas legibles. Es el flujo central del proyecto final.

## Pre-requisitos

- **[[../concepts/Caso-Base|Caso base]] congelado** y al menos una variante corrida.
- Ambas simulaciones generaron las **mismas variables** de output (con los mismos nombres). Si difieren, las comparaciones no son válidas.
- Entorno Python con `iertools` listo. Ver [[Setup-Entorno-Python-uv]].

## 1. Imports

```python
import pandas as pd
import matplotlib.pyplot as plt
from iertools.read import read_sql
from dateutil.parser import parse
```

## 2. Función de carga reutilizable

> Anti-patrón: copiar la celda de carga 5 veces para 5 simulaciones — fuente de errores cuando hay que cambiar el flujo.

Patrón recomendado: definir una función de carga, llamarla con cada path:

```python
NOMBRES = {
    "TZ_ESTE:Zone Mean Air Temperature [C]":  "T_este",
    "TZ_OESTE:Zone Mean Air Temperature [C]": "T_oeste",
    "Environment:Site Outdoor Air Drybulb Temperature [C]": "TO",
    "VNORTE:Surface Outside Face Incident Solar Radiation Rate per Area [W/m2]": "IS_vNorte",
    "VOESTE:Surface Outside Face Incident Solar Radiation Rate per Area [W/m2]": "IS_vOeste",
    "TECHO:Surface Outside Face Incident Solar Radiation Rate per Area [W/m2]": "IS_techo",
}

def carga_df(f):
    df = read_sql(f, alias=False).data
    df.rename(columns=NOMBRES, inplace=True)
    return df
```

> **Cuidado: error común.** Si dentro de la función defines `f = "ruta_fija"` accidentalmente, la función ignora el argumento y siempre regresa lo mismo:
>
> ```python
> def carga_df(f):
>     f = "../OSM/005_caso_base/run/eplusout.sql"  # ❌ ignora el parámetro
>     return read_sql(f).data
> ```
>
> El profesor lo confiesa: "no saben cuántas veces me ha pasado y a veces no me doy cuenta — es lo peligroso."

### Por qué `alias=False` y renombrar manualmente

[[../tools/iertools|`iertools`]] con `alias=True` da nombres genéricos (`T_<zona>`, `TO`, etc.) — pero **no** sabe los nombres custom de superficies (`vNorte`, `vOeste`, `techo`). Para esos hay que renombrar a mano. Mezclar `alias=True` y un diccionario de renombrado custom termina mal — mejor un solo paso de renombrado.

### Construir el diccionario sin escribir a mano

Truco para no escribir el diccionario letra por letra:

```python
# 1. Cargar una vez sin renombrar
df = read_sql(F_CASO_BASE, alias=False).data

# 2. Generar el diccionario base con dict comprehension
nombres = {col: col for col in df.columns}

# 3. Imprimir, copiar la salida y editar las llaves de interés
print(nombres)
```

Pegar la salida en una celda nueva, dejar solo las columnas que quieres renombrar y editar los valores.

> **Tip de `df.rename`**: si una llave del diccionario **no existe** en las columnas, `rename` **no falla** — simplemente la ignora. Útil para reusar el mismo diccionario en simulaciones que difieren ligeramente, pero peligroso porque enmascara typos. El profesor: "si yo me equivoco en algo, también puedo ir hasta allá."

## 3. Cargar cada simulación

```python
F_BASE  = "../OSM/005_caso_base/run/eplusout.sql"
F_ALERO = "../OSM/006_protecciones/run/eplusout.sql"

base  = carga_df(F_BASE)
alero = carga_df(F_ALERO)
```

> Reusar el mismo nombre de variable temporal `f` dentro de la función está bien siempre que se respete su alcance local. El profesor: "es una variable temporal — siempre y cuando yo la defina y me asegure que está bien definida."

## 4. Sanity check — ¿son realmente distintas?

Antes de graficar, verifica numéricamente:

```python
print("base.T_este[0]:",  base.T_este.iloc[0])
print("alero.T_este[0]:", alero.T_este.iloc[0])
```

Si los primeros valores coinciden **exactamente**, sospecha:

- Las dos simulaciones cargan el mismo SQL (typo en uno de los paths).
- Una de las simulaciones no aplicó el cambio (el bug del piso adiabático que se revierte — ver [[Debuggear-Simulacion-OpenStudio]]).
- Las protecciones no llegaron al IDF (caso real de la clase 007 que el profesor no logró depurar en vivo).
- **Estás comparando radiación incidente sobre ventanas** — esa variable no refleja sombreamiento en sub-superficies (resuelto en clase 008). Pedir `Surface Outside Face Sunlit Fraction` y la radiación sobre el muro padre. Ver [[Auditar-Sombreamiento-Ventanas]].

> El profesor cierra la clase 007 con dos simulaciones que dan resultados casi iguales. Su primera sospecha: revisar el flujo de datos antes de culpar a Energy Plus. La clase 008 resuelve el bug: las simulaciones SÍ eran distintas, pero la variable elegida no mostraba el efecto.

## 5. Filtrar columnas con list comprehension

Útil cuando hay muchas variables de la misma familia:

```python
# Todas las temperaturas de zona
cols_T = [c for c in base.columns if c.startswith("T_")]

# Todas las radiaciones incidentes sobre superficies
cols_IS = [c for c in base.columns if c.startswith("IS_")]

print(cols_T)   # ['T_este', 'T_oeste']
print(cols_IS)  # ['IS_vNorte', 'IS_vOeste', 'IS_techo']
```

> Por esto se eligen prefijos consistentes (`T_`, `IS_`, `IB_`, etc.) — para iterar y filtrar fácilmente. La convención es del paquete del grupo.

## 6. Plot comparativo — convención color/estilo

> "**No hagan dobles ejes Y.** Si tienen 6 líneas, usen el mismo color para variables comparables y distinto estilo para distinguir casos. Las soluciones suelen ser bien sencillas."

Patrón recomendado: **color = ubicación, estilo = caso**.

```python
fig, ax = plt.subplots(2, 1, sharex=True, figsize=(12, 6))

# Panel superior — temperaturas
ax[0].plot(base.T_este,   label="este (base)",  color="red",  linestyle="-")
ax[0].plot(alero.T_este,  label="este (alero)", color="red",  linestyle="--")
ax[0].plot(base.T_oeste,  label="oeste (base)", color="blue", linestyle="-")
ax[0].plot(alero.T_oeste, label="oeste (alero)", color="blue", linestyle="--")
ax[0].plot(base.TO,       label="exterior",     color="black", linestyle=":")
ax[0].set_ylabel("Temperatura (°C)")
ax[0].legend(ncol=3)

# Panel inferior — radiación incidente sobre ventanas
ax[1].plot(base.IS_vNorte,  label="norte (base)",  color="green", linestyle="-")
ax[1].plot(alero.IS_vNorte, label="norte (alero)", color="green", linestyle="--")
ax[1].plot(base.IS_vOeste,  label="oeste (base)",  color="orange", linestyle="-")
ax[1].plot(alero.IS_vOeste, label="oeste (alero)", color="orange", linestyle="--")
ax[1].set_ylabel("Radiación incidente (W/m²)")
ax[1].legend(ncol=2)

# Recorte temporal
f1 = parse("2006-05-31")
f2 = f1 + pd.Timedelta(days=2)
ax[1].set_xlim(f1, f2)
```

### Por qué este patrón

- **Mismo color = misma variable comparable** → mirada al panel ve "los rojos" (T_este) en sus dos casos.
- **Sólido = base, dasheado = variante** → distingue casos sin recurrir a 6 colores distintos.
- **Negro punteado** para la T exterior — referencia universal.
- `ncol=3` en `legend()` distribuye la leyenda horizontalmente.

## 7. Recortar al día más cálido (caso típico)

```python
# Día con T promedio diaria más alta
dia_max = base.TO.resample("D").mean().idxmax()
f1 = dia_max
f2 = f1 + pd.Timedelta(days=2)

ax[1].set_xlim(f1, f2)
```

Detalle del criterio en [[Analizar-Resultados-Python]] sección "Encontrar el día más cálido — explicitar el criterio".

## 8. Comparación cuantitativa — tabla de métricas

```python
def metricas(df, etiqueta):
    return pd.Series({
        "T_este_max":  df.T_este.max(),
        "T_oeste_max": df.T_oeste.max(),
        "IS_vNorte_total":  (df.IS_vNorte * 10/60).sum(),  # W*h
        "IS_vOeste_total":  (df.IS_vOeste * 10/60).sum(),
    }, name=etiqueta)

tabla = pd.concat([metricas(base, "base"), metricas(alero, "alero")], axis=1)
tabla["Δ%"] = (tabla["alero"] - tabla["base"]) / tabla["base"] * 100
print(tabla)
```

Resultado típico para una variante con aleros bien diseñados: reducción del 30-60% en radiación incidente sobre ventanas → reducción de 1-3°C en T máxima.

## 9. Iteración con función de plotting

Si hacen las mismas gráficas para varios casos:

```python
def plot_case(df, etiqueta, ax):
    ax[0].plot(df.T_este,  label=f"este ({etiqueta})")
    ax[0].plot(df.T_oeste, label=f"oeste ({etiqueta})")
    ax[1].plot(df.IS_vNorte, label=f"vNorte ({etiqueta})")
    ax[1].plot(df.IS_vOeste, label=f"vOeste ({etiqueta})")

fig, ax = plt.subplots(2, 1, sharex=True)
for df, etiq in [(base, "base"), (alero, "alero")]:
    plot_case(df, etiq, ax)
```

Trade-off:

- **Pros**: menos repetición, fácil agregar un caso más.
- **Cons**: pierde control fino del color/estilo; menos explícito.

Para 2 casos vale la pena copiar/pegar; para 5 casos conviene la función.

## Trampas comunes

| Síntoma | Causa probable |
|---------|----------------|
| Las dos series se ven idénticas | Mismo SQL cargado dos veces, o el cambio del modelo no llegó al IDF |
| `KeyError: 'T_este'` | El renombrado falló porque el nombre original cambió (zona renombrada en OSM) |
| Una columna que pediste no aparece | El measure de Add Output Variable tiene typo, o la superficie referenciada perdió su nombre custom (bug FloorspaceJS — ver [[Debuggear-Simulacion-OpenStudio]]) |
| `len(base.columns) != len(alero.columns)` | Una simulación no incluye una variable — variantes desincronizadas, regresar a [[../concepts/Caso-Base]] |
| Diferencia entre casos sospechosamente pequeña | Ventana muy chica → efecto absoluto bajo. Aumentar el tamaño o probar con día más cálido. O: el cambio no se aplicó (ver bug del piso adiabático) |

## Para profundizar análisis con confort adaptativo

Calcular grados-hora cálidos/fríos para cada caso y comparar — ver [[EDA-Archivo-EPW]] sección "Grados-hora de disconfort".

## Clases relacionadas

- [[../classes/007-CasoBaseAleros]] — primera demo en vivo del flujo (con bug no resuelto al final)
- [[../classes/008-ShadingVentanas]] — resolución del bug: la variable de radiación incidente no refleja sombreamiento en sub-superficies
