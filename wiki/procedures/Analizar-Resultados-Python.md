---
title: Analizar resultados de simulación con Python
type: procedimiento
tags: [procedimiento, python, iertools, pandas, matplotlib, sql, analisis]
aliases: [analizar resultados, leer sql python, plot simulacion]
clases: [005, 006, 007, 009, 011]
updated: 2026-05-02
---

# Analizar resultados de simulación con Python

Procedimiento para cargar el SQL de Energy Plus en Python y producir las gráficas básicas del taller (T del aire interior vs T exterior, radiación incidente).

## Pre-requisitos

- Simulación ya ejecutada con las variables solicitadas. Ver [[Solicitar-Output-Variables-Measures]].
- Entorno Python listo con `iertools`. Ver [[Setup-Entorno-Python-uv]].

## 1. Lanzar Jupyter Notebook

Desde el folder del proyecto:

```bash
uv run jupyter notebook
```

Crear una nueva libreta dentro de `notebooks/`. Renombrarla con la convención del taller: `001_EDA_simulacion.ipynb`.

## 2. Imports

```python
import pandas as pd
import matplotlib.pyplot as plt
from iertools.read import read_sql
from dateutil.parser import parse
```

## 3. Localizar el archivo SQL con ruta relativa

Convención: la libreta vive en `notebooks/`. El SQL está en `OSM/<nombre>/run/eplusout.sql`.

```python
f = "../OSM/mi_primer_cubo/run/eplusout.sql"
```

> **Truco**: usar el **autocompletado de Jupyter** (`Tab`) para confirmar el path. Si Tab no completa, estás en otro directorio del que crees. Si el path tiene acentos/eñes/espacios, las rutas relativas pueden fallar — ver [[Estructura-Proyecto-Simulacion]].

## 4. Cargar el SQL

```python
cubo = read_sql(f, alias=True)
```

Devuelve un objeto con varias propiedades. Detalle en [[../tools/iertools]].

### Inspeccionar sistemas constructivos (auditoría)

Antes de creer en los resultados, verificar que los **materiales y propiedades** son los esperados — uno de los errores más comunes:

```python
sc = cubo.construction_systems
sc  # lista de nombres

cubo.get_constructions(sc)  # detalle de cada uno con capas y propiedades
```

Para cada construction se ven:

- Nombre, espesor total, número de capas.
- Capas en orden ext→int con: conductivity, density, specific heat, thickness, thermal absorptance, roughness.

> "Eso es parte de mi deber. En el grupo solemos tener — Memo, ahí está la simulación, revísala. Lo primero que hago es verificar que las propiedades estén bien."

### Acceder al DataFrame de series temporales

```python
df = cubo.data
df.head()
```

Con `alias=True`, las columnas tienen nombres cortos: `T_cubo`, `TO`, `IB`, `ID`, `RH`, `WS`, `WD`, etc. Ver convención completa en [[../tools/iertools]].

## 5. Hacer la gráfica de doble panel

```python
fig, ax = plt.subplots(2, 1, sharex=True, figsize=(12, 4))

# Panel superior — temperaturas
ax[0].plot(df.T_cubo, label="T_cubo")
ax[0].plot(df.TO,     label="TO")
ax[0].set_ylabel("Temperatura (°C)")
ax[0].legend()

# Panel inferior — radiación
ax[1].plot(df.ID, label="diffuse")
ax[1].plot(df.IB, label="beam")
ax[1].plot(df["IG_techo"], label="incidente techo")  # con corchetes si el alias contiene caracteres especiales
ax[1].set_ylabel("Radiación (W/m²)")
ax[1].legend()
```

### Recomendaciones de plotting

- **Doble panel con eje X compartido** (`sharex=True`) > doble eje Y. Las gráficas de doble eje son **horrorosas en presentaciones** ("nos toma 10 minutos entenderlas").
- **`figsize=(12, 4)`** es un tamaño cómodo para series temporales horizontales.
- **`label`** en cada `plot` + `legend()` para que la gráfica se autodocumente.

### Filtrado de columnas con list comprehension

Cuando hay muchas variables de la misma familia (varias zonas, varias superficies), filtrar por prefijo:

```python
cols_T  = [c for c in df.columns if c.startswith("T_")]
cols_IS = [c for c in df.columns if c.startswith("IS_")]
```

Por esto se eligen prefijos consistentes (`T_`, `IS_`, `IB_`, `ID_`) — para iterar y filtrar fácilmente.

### Renombrado custom con diccionario

Cuando los alias automáticos de `iertools` no cubren todos los casos (típicamente: nombres custom de superficies como `Techo`, `vNorte`, `vOeste`):

```python
# Construir el diccionario base con dict comprehension
df = read_sql(F).data
nombres = {col: col for col in df.columns}
print(nombres)
```

Pegar la salida en una celda nueva, dejar solo las columnas de interés, editar los valores. Después aplicar:

```python
df.rename(columns=NOMBRES, inplace=True)
```

> Tip: `rename` **no falla** si una llave del diccionario no existe — útil para reusar el mismo diccionario en variantes con columnas distintas, pero peligroso porque enmascara typos. Verificar con `df.columns` después de renombrar.

### Función de carga reutilizable (anti-anti-patrón)

Para varias simulaciones, definir una función:

```python
def carga_df(f):
    df = read_sql(f, alias=False).data
    df.rename(columns=NOMBRES, inplace=True)
    return df

base    = carga_df("../OSM/005_caso_base/run/eplusout.sql")
variante = carga_df("../OSM/006_protecciones/run/eplusout.sql")
```

> **Bug confesional del profesor**: redefinir el argumento dentro de la función accidentalmente.
>
> ```python
> def carga_df(f):
>     f = "../OSM/005_caso_base/run/eplusout.sql"  # ❌ ignora el parámetro
>     return read_sql(f).data
> ```
>
> "No saben cuántas veces me ha pasado y a veces no me doy cuenta — es lo peligroso."

Detalle del flujo de comparación caso base vs variante en [[Comparar-Simulaciones-Python]].

### Encontrar el día más cálido — explicitar el criterio

> "Día más cálido" no es un criterio único. Hay varias interpretaciones:

```python
# Día con T máxima absoluta
dia_max_abs = df.TO.idxmax().date()

# Día con T promedio diario más alto (recomendado por el profesor)
dia_max_prom = df.TO.resample("D").mean().idxmax()

# Día con más grados-hora cálidos (modelo adaptativo)
# Requiere primero calcular T_neut mensual — ver EDA-Archivo-EPW
```

> Cuando reportes un análisis: **explicitar el criterio**. "Día más cálido" no es suficiente — siempre acompañar con la métrica usada.

Para zoom alrededor del día más cálido:

```python
f1 = df.TO.resample("D").mean().idxmax()
f2 = f1 + pd.Timedelta(days=2, hours=7)
ax.set_xlim(f1, f2)
```

## 6. Recortar el rango temporal con dateutil + Timedelta

Para enfocar en una semana específica (en lugar de mostrar todo el año):

```python
f1 = parse("2006-03-13")               # fecha de inicio
f2 = f1 + pd.Timedelta(days=7)         # 7 días después

ax[1].set_xlim(f1, f2)
```

Beneficios de este patrón:

- **`dateutil.parser.parse`** acepta strings flexibles: `"2006-03-13"`, `"March 13, 2006"`, etc.
- **`pd.Timedelta`** acepta combinaciones humanas: `days=7`, `hours=3`, `minutes=50`, `seconds=30`.
- Cambiar el rango = cambiar dos líneas. No hay que calcular fechas a mano.

> "Ya defino mi intervalo inicial, mi fecha inicial, más un Timedelta. Le digo: 7 días, 3 horas, 50 minutos. Y ya."

## 7. Mantener la libreta robusta

Antes de cerrar el día, **Restart and Run All**:

> Kernel → Restart & Run All

Esto reinicia el kernel y re-ejecuta todas las celdas en orden. Si la libreta corre limpia, está reproducible. Si falla, hay variables fantasma (creadas y luego borradas pero aún en memoria) o celdas que dependen de orden no-lineal — corregir.

> "Cada cierto tiempo hagan un Restart & Run All. A mí ya casi no me pasa, pero al principio se les arruinan las libretas y al siguiente día ya no funciona y ya no se acuerdan qué hicieron."

## 8. Año en el datetime — el "2006" del EPW

Las simulaciones de E+ por default ponen año `2006` en todos los timestamps (independiente del año real del EPW, que es un TMY mezclado). Si te molesta:

- En el OSM cambiar el año (objeto `RunPeriod`).
- O en pandas: `df.index = df.index.map(lambda x: x.replace(year=2024))`.

> En el alcance del curso `2006` es solo una etiqueta — los datos siguen siendo válidos. No hay que cambiarlo a menos que se quiera coincidir con un EPW real con fecha.

## 9. Patrón general — cualquier análisis

```python
# 1. Cargar
cubo = read_sql("../OSM/<...>/run/eplusout.sql", alias=True)

# 2. Auditar
cubo.get_constructions(cubo.construction_systems)

# 3. Trabajar en DataFrame
df = cubo.data

# 4. Filtrar/agregar
mes = df.resample("ME").mean()       # promedio mensual
dia = df.resample("D").max()         # máximo diario
energia_mes = df.resample("ME").sum()  # energía acumulada mensual (J)

# 5. Comparar — muchas simulaciones
casos = {
    "base":  read_sql("../OSM/caso_base/run/eplusout.sql", alias=True),
    "alero": read_sql("../OSM/con_alero/run/eplusout.sql", alias=True),
}
for nombre, sim in casos.items():
    plt.plot(sim.data.T_cubo, label=nombre)
plt.legend()
```

## Patrones de plotting adicionales (clase 009)

### Resample mensual con `resample()`

Para series temporales, `resample` es la herramienta clave. Frecuencias:

| Alias | Significado |
|-------|-------------|
| `"H"` | Hora |
| `"D"` | Día |
| `"W"` | Semana |
| `"ME"` | Month-End (mes calendario) |
| `"YE"` | Year-End (año) |

Operaciones: `.sum()`, `.mean()`, `.max()`, `.min()`, `.std()`.

### Gráfica de barras mensual (consumo AC)

```python
fig, ax = plt.subplots(figsize=(10, 4))
energia_mes_kWh = df["cooling_energy_J"].resample("ME").sum() / 3.6e6  # J → kWh

ax.bar(range(1, 13), energia_mes_kWh.values)
ax.set_xticks(range(1, 13))
ax.set_xticklabels(["E", "F", "M", "A", "M", "J", "J", "A", "S", "O", "N", "D"])
ax.set_ylabel("Energía cooling (kWh)")
```

`ax.bar(x, height)` requiere posiciones X explícitas — `range(1, 13)` para los 12 meses.

### Variable constante — workaround del ylim

Cuando una serie es **constante** (ej. T en setpoint fijo), matplotlib se confunde con el ylim automático. Forzar manualmente:

```python
ax.plot(df.T_zona)
ax.set_ylim(15, 25)  # límites manuales
```

> Caso real: el profesor descubre esto en vivo en clase 009 graficando una T mantenida en 20 °C constante.

## Anti-patrones de Python observados (clase 011)

### Referencias compartidas — `df_b = df_a` no es copia

> "En Python, si yo digo `df_b = df_a`, esos dos quedan enlazados. Si modifico uno se cambia el otro porque están apuntando a arreglos dinámicos en memoria."

```python
# ❌ MAL — df_b y df_a son la MISMA referencia
df_b = df_a
df_b["nueva_col"] = ...   # también afecta df_a

# ✅ BIEN — copia independiente
df_b = df_a.copy()
df_b["nueva_col"] = ...   # df_a no se afecta
```

Bug silencioso típico — los cambios se "propagan" sin error visible. Especialmente peligroso al hacer copias para modificarlas individualmente.

### Iteración sobre DataFrames es lenta — usar NumPy

> "Iterar un DataFrame para resolver problemas de transferencia de calor es muy lento. Pásenlos a NumPy y aquello vuela."

EnerHabitat originalmente usaba DataFrames internamente → 3 minutos por simulación. Migración a NumPy arrays → 3 segundos.

```python
# ❌ LENTO — iterar DataFrame
for i, row in df.iterrows():
    ... # cálculos sobre row

# ✅ RÁPIDO — vectorizar con NumPy
arr = df.values  # convertir a array
... # operaciones vectorizadas sobre arr
```

**Buena práctica**: para cálculos numéricos pesados, usar **NumPy arrays**. Reservar DataFrames para análisis exploratorio y postprocesamiento.

### Numba para acelerar loops numéricos

Cuando los loops son inevitables (cálculos iterativos como solvers numéricos), `numba` compila la función a código nativo:

```python
from numba import jit

@jit(nopython=True)
def mi_solver(arr_in):
    for i in range(len(arr_in)):
        ...  # cálculos numéricos
    return arr_in
```

Speed-up típico: 50-100×. Imprescindible para producción.

### Reproducibilidad frágil de Jupyter — Restart and Run All

> "Reproducibilidad en libretas Jupyter es bien frágil, bien frágil."

Caso típico: una variable definida en una celda se borra del código pero queda en memoria → la libreta corre aparentemente bien hasta que se hace **Restart and Run All** y revela el bug.

Hacer **Kernel → Restart and Run All** periódicamente — al menos antes de:

- Cerrar el día.
- Compartir la libreta.
- Reportar resultados.

Si falla al reiniciar, hay variables fantasma o orden no-lineal — corregir.

### Plot con índice booleano conecta puntos no adyacentes (anti-patrón)

Caso típico: pintar la T interior con un color por banda (verde dentro de confort, rojo arriba, azul abajo).

```python
confort = (Ti >= Tn_inf) & (Ti <= Tn_sup)
calor   = (Ti > Tn_sup)
frio    = (Ti < Tn_inf)

# ❌ Anti-patrón — plot() con filtro booleano
ax.plot(Ti[confort], color='green')   # selecciona timestamps NO contiguos
ax.plot(Ti[calor],   color='red')     # plot() los conecta con líneas falsas
ax.plot(Ti[frio],    color='blue')    # → líneas que cruzan zonas de otra banda
```

`Ti[mask]` devuelve sólo los timestamps que cumplen la condición — pero `plot()` traza una línea **entre puntos consecutivos del filtro**, lo que produce líneas espurias que cruzan las otras bandas.

Soluciones:

```python
# ✅ Opción A: .where() inserta NaN en los huecos — la línea se rompe en cada cambio de banda
ax.plot(Ti.where(confort), color='green', label="confort")
ax.plot(Ti.where(calor),   color='red',   label="calor")
ax.plot(Ti.where(frio),    color='blue',  label="frio")
ax.legend()

# ✅ Opción B: scatter — cada punto aislado, sin línea
ax.scatter(Ti.index[confort], Ti[confort], color='green', s=1, label="confort")
```

**Antipatrón complementario**: olvidar `ax.legend()`. Los `label="..."` quedan inertes hasta llamarlo. Visto en [[../notebooks/007_DDH]].

## Trampas comunes

| Síntoma | Causa |
|---------|-------|
| Columnas con nombres como `CUBO:Zone Mean Air Temperature [C]` | Olvidaste `alias=True` |
| `KeyError: 'T_cubo'` con `alias=True` | El alias se construye del nombre de la zona; verificar `df.columns` |
| Tab no autocompleta archivos | Estás en un directorio distinto al esperado — `pwd` o `!ls` |
| Una variable sale con NaN en muchas filas | Tu measure tiene frecuencia distinta al resto — todas a `Timestep` ([[Solicitar-Output-Variables-Measures]]) |
| El plot se ve "compacto" sin detalle | No has aplicado `set_xlim`, está mostrando todo el año |
| Líneas falsas que cruzan bandas en plot por color | `plot(serie[mask])` conecta puntos no adyacentes — usar `.where(mask)` o `scatter` |
| `label="..."` no aparece en la leyenda | Falta `ax.legend()` |

## Para análisis del EPW (no de la simulación)

Ver [[EDA-Archivo-EPW]] — flujo paralelo usando `read_epw` en lugar de `read_sql`.

## Clases relacionadas

- [[../classes/005-AnalisisSimulacionesPython]] — demo completa del flujo
- [[../classes/006-DosZonasTermicasVentanasAleros]] — patrón de "día más cálido" con criterio explícito
- [[../classes/007-CasoBaseAleros]] — list comprehensions sobre columnas, renombrado con diccionario, función de carga reutilizable
- [[../classes/009-AireAcondicionadoSetPoints]] — `resample("ME").sum()` para energía mensual, gráfica de barras, workaround ylim
- [[../classes/011-EnerHabitatParte2]] — anti-patrones (referencias compartidas, NumPy vs DataFrame, fragilidad Jupyter)
