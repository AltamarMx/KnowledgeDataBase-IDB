---
title: 013 — Cálculo de Grados-Hora de Disconfort
type: clase
clase: 013
profesor: Guillermo Barrios del Valle
fuente: raw/videos/013_CalculoDDH.md
fecha_clase: 2026-05-22
fecha_ingesta: 2026-05-22
tags: [clase, grados-hora, confort-adaptativo, humphreys, morillon, pandas, visualizacion, proyecto-final]
aliases: [Clase 013, Cálculo DDH, Cálculo GHDC GHDF]
---

# 013 — Cálculo de Grados-Hora de Disconfort

## Metadatos

- **Clase:** 013 (22 de mayo de 2026)
- **Profesor:** Guillermo Barrios del Valle
- **Fuente:** `raw/videos/013_CalculoDDH.md`
- **Tipo:** Clase técnica + código en vivo. Implementa en pandas el cálculo de **GHDC** y **GHDF** que el proyecto final pide. Es la narración didáctica detrás del [[../notebooks/007_DDH]].
- **Notebook que se crea en clase:** llamado `008-B-grados-horas-disconfort` durante la sesión (en la wiki está ingerido como [[../notebooks/007_DDH]]).

## Resumen

Dos bloques:

1. **Teoría del modelo de confort adaptativo** — temperatura de neutralidad de **Humphreys-Nicol** (`Tn = 13.5 + 0.54·To_mensual`), banda de **Morillón** (`±ΔTN/2`), advertencia de sobrestimación de confort, y hallazgo propio en Chilpancingo (Tn constante mensual).
2. **Cálculo en pandas** del notebook completo: `groupby(index.month).mean()`, broadcast con `index.month.map(Tn_m)`, integración con `.clip(lower=0).sum() * dt_h` (Δt = 1/6 h), y visualización con máscaras booleanas de tres colores (verde/rojo/azul).

> "Las visualizaciones no solamente son para presentar resultados, es para asegurarme que estoy haciendo las cosas bien."

## Temperatura de neutralidad — modelo de Humphreys-Nicol

$$
T_n = 13.5 + 0.54 \cdot \overline{T_o}_{\text{mensual}}
$$

- `To` = temperatura del aire exterior (drybulb).
- Tn varía **mes a mes** porque el modelo adaptativo dice que la población se aclimata al clima reciente.

> "Humphreys nos dice que si somos capaces de adaptarnos en un clima como el de Temixco… cada mes tenemos una temperatura de confort."

### Advertencia importante

**Humphreys + banda de Morillón sobrestima el confort.** A 28 °C el modelo dice que aún hay confort, pero "a 28 ya estamos sudando".

> "Lo malo de esto es que no les podría yo decir 'usen esto porque no hay [otra opción]'… ténganlo en conciencia."

### Hallazgo del grupo del profesor — Chilpancingo

En un estudio en Chilpancingo, Guerrero, **Tn no varía mensualmente**: se mantiene constante. Explicación: en climas con baja variabilidad anual, la población no se readapta mes a mes.

- Tn de confort ≈ 25.6 °C (rango ya no recordado con precisión durante la clase).
- Paper en revisión (similar al de Guadalupe).
- "Es un descubrimiento tal vez no tan novedoso, pero aporta información."

Lectura general: en climas extremosos (Cuernavaca, p.ej.), Tn **sí** varía mes a mes; en climas estables, **no**.

## Banda de Morillón

$$
T_{conf,sup} = T_n + \text{banda} \qquad T_{conf,inf} = T_n - \text{banda}
$$

- **Banda en Temixco ≈ 1.25 °C** (el delta total de Morillón es ~2.5 °C, dividido entre 2 = la "banda" hacia cada lado del centro).
- Aclaración del profesor: Morillón define el delta como total (superior − inferior), pero hay autores que lo manejan como hemiancho; en clase se usa **banda = hemiancho** para evitar confusión.

> "Para no crear confusión vamos a ponerle 'banda'."

El cálculo de la banda con la amplitud de la serie histórica **queda como pendiente** para la última clase.

## Implementación en pandas

### 1. Setup

```python
import pandas as pd
import matplotlib.pyplot as plt
from iertools.read import read_sql

f = "../osm/004_dos_zonas/run/eplusout.sql"   # caso base del modelo dos-zonas
base = read_sql(f, alias=True).data
Ti = base["Ti_ESTE"]
```

### 2. Temperatura de neutralidad mensual

```python
To_m = base["To"].groupby(base.index.month).mean()
Tn_m = 13.5 + 0.54 * To_m
```

**Por qué `groupby(index.month)` y no `resample("ME")`:**

- `resample("ME")` devuelve un timestamp por fin de mes (12 entries por año) — útil si quieres mantener la dimensión temporal.
- `groupby(index.month)` devuelve un Series indexado por 1..12 — perfecto para **mapear de vuelta** a la serie temporal completa con `.map`.
- Además `groupby` es más poderoso: acepta funciones personalizadas, no solo `mean`/`max`/`min`.

> "El `group` es como mucho más poderoso porque el `resample` solamente [hace agregaciones predefinidas]."

### 3. Broadcast de Tn mensual a serie temporal

```python
Tn_serie = pd.Series(base.index.month.map(Tn_m), index=base.index)
banda = 1.25                # Morillón para Temixco
Tn_sup = Tn_serie + banda
Tn_inf = Tn_serie - banda
```

**Por qué no `resample` para el broadcast:** si rellenas con `ffill` o `bfill` desde un resample mensual, **se pierde un día** al final (pandas no extiende el resample hasta el último timestamp de la serie original). `index.month.map(Tn_m)` garantiza que cada uno de los 52,560 timesteps recibe su Tn correspondiente sin gaps.

> "Esto yo lo hacía de otra manera mucho más rebuscada y me gustó más esta última manera."

### 4. Grados-hora con `.clip(lower=0)`

```python
dt_h = 10/60                # paso de 10 min en horas (= 1/6)

GHDC = (Ti - Tn_sup).clip(lower=0).sum() * dt_h
GHDF = (Tn_inf - Ti).clip(lower=0).sum() * dt_h
```

**El truco de `.clip(lower=0)`:**

- `(Ti - Tn_sup)` da valores **positivos** cuando hace calor (lo que queremos) y **negativos** cuando estamos en confort o frío (no queremos).
- `np.abs` los contaría todos — **mal**, suma frío y calor.
- `.clip(lower=0)` mantiene los positivos y convierte los negativos en 0 — **bien**.

> "Es muy bonito porque existe una función que se llama `clip`. Antes lo hacía muy rebuscado."

**Análogo:** la integral numérica por el método del rectángulo — cada timestep aporta `(desvío) × dt`.

### 5. Visualización por máscaras (verde/rojo/azul)

```python
confort = (Ti >= Tn_inf) & (Ti <= Tn_sup)
calor   = (Ti > Tn_sup)
frio    = (Ti < Tn_inf)

fig, ax = plt.subplots(figsize=(12, 3))
ax.plot(Ti[confort], color='green', alpha=0.5, label="confort")
ax.plot(Ti[calor],   color='red',   alpha=0.5, label="calor")
ax.plot(Ti[frio],    color='blue',  alpha=0.5, label="frio")
```

**Idea:** pintar la serie Ti con un color distinto según en qué banda cae, en lugar de dibujar líneas horizontales para los límites. Cuando se sobrepone `ax.plot(Ti)` original encima, **debe coincidir exactamente** — sanity check de que las máscaras están bien construidas.

> "Si yo graficara `ax.plot(base.Ti)`, caen encima. ¿Quiere decir que estoy bien?"

Antipatrones de esta visualización (ver detalles en [[../notebooks/007_DDH]]): `plot()` con índice booleano une puntos no contiguos → preferir `Ti.where(mask).plot()` o `scatter`.

## Resultados del caso base (zona ESTE)

```
GHDC ≈ 6,884 °C·h     (cálidos)
GHDF ≈ 5,001 °C·h     (fríos)
```

> "Calor mucho mayor, pero no tanto como uno pensaría."

**Lectura:** el modelo dos-zonas en Temixco tiene **GHDC > GHDF** pero los dos son grandes — clima predominantemente cálido con frío significativo. Es justo el tipo de clima en que el profesor advierte contra escoger para el proyecto final por el trade-off severo cálido↔frío ([[002-ConceptosBasicosBalancesCalor]]).

## Promedio pesado por volumen — pendiente para el proyecto

Para reportar el desempeño global de la vivienda **no basta** una zona térmica. El profesor confirma que actualizará el documento del proyecto:

$$
\overline{T_i} = \frac{\sum_z T_{i,z} \cdot V_z}{\sum_z V_z}
$$

- Una temperatura promedio simple asume que **un baño chiquito vale igual que la sala** — sesgo.
- Lo "ideal" sería reportar **por zona térmica** (lo que harían consultores reales), pero para no convertir el proyecto en un trabajo de análisis de datos, se acepta la temperatura pesada por volumen.

> "Si fueran consultores, consultoras, tendrían que hacerlo por zona térmica."

Acuerdo del profesor: documentar este requisito en el enunciado del proyecto y consultarlo con Miriam.

## Crítica al uso indebido de simulaciones

Cierre de clase con la idea de **caricatura computacional** ([[../concepts/Caricatura-Computacional]]):

- El modelo simulado es una **caricatura** — sin radiación, sin humedad, sin viento, sin vestimenta, sin actividad. Solo temperatura.
- "El confort tiene que ver con la temperatura radiante, la humedad, la velocidad del viento, lo que estamos vistiendo y con lo que estamos haciendo."

### Anécdota — artículo con Design Builder

El profesor revisó un artículo donde se usó **Design Builder** para evaluar el consumo energético de una casa de **adobe en Oaxaca**:

- Configuraron AC ideal con enfriamiento **y** calentamiento — en Oaxaca, en adobe, esto no tiene sentido físico.
- Reportaron "consumo" en **watts** (es potencia, no consumo — debería ser J o Wh).
- Reportaron los **picos máximos** como si fueran el consumo representativo.
- Sin documentar cargas internas, infiltración, ventilación, ventanas, ocupación, actividades.

> "La gente ni siquiera tiene conciencia de lo que está haciendo… son un montón de cosas que se tendrían que especificar para que esa casa esté bien hecha."

### Notebook LM como crítica externa

El profesor sube las clases a YouTube → las pasa a **Notebook LM** → usa el modo "podcast crítico" para que un agente le cuestione. Una de las críticas recurrentes: "¿y por qué dices que esto es una caricatura?" — lo que lo empuja a justificar mejor las simplificaciones.

## Conexiones

- ← **Anterior:** [[012-ProyectoFinal]]
- → **Siguiente:** _(infiltración + ventanas complejas, clase final 29 de mayo)_
- → Concepto principal: [[../concepts/Grados-Hora-Disconfort]]
- → Modelo de confort: [[../concepts/Confort-Adaptativo]]
- → Notebook con la implementación: [[../notebooks/007_DDH]]
- → Caricatura computacional: [[../concepts/Caricatura-Computacional]]
- → Temperatura operativa (siguiente nivel de precisión): [[../concepts/Temperatura-Operativa]]

## Patrones de pandas introducidos / consolidados

| Patrón | Para qué |
|---|---|
| `series.groupby(idx.month).mean()` | Promedio por mes calendario (sin importar año) |
| `idx.month.map(serie_mensual)` | Broadcast vectorizado de Series mensual a serie temporal |
| `pd.Series(data, index=...)` | Recuperar el índice temporal después del `.map` |
| `.clip(lower=0)` | Integración positiva — equivalente a `max(x, 0)` |
| Máscaras booleanas con `&` y `\|` | Particionar visualización por banda |
| `dt_h = 10/60` | Paso temporal en horas (timestep de 10 min) |

## Pendientes anunciados en clase

1. **Cálculo de la banda** a partir de la amplitud de la serie histórica de To — para la última clase.
2. **Temperatura pesada por volumen** — actualizar enunciado del proyecto final.
3. **Infiltración + ventanas complejas + cambios de aire** — siguiente clase (22 mayo).
4. **Cafecito** — última clase del taller, 29 de mayo.
