---
title: Grados-Hora de Disconfort
type: concepto
tags: [concepto, confort, metricas, bioclimatico, analisis, proyecto-final]
aliases: [grados hora, degree-hours, GH, GH-calido, GH-frio, disconfort acumulado]
clases: [008, 012, 013]
updated: 2026-05-22
---

# Grados-Hora de Disconfort

## Definición

Métrica que **acumula la severidad y duración del disconfort térmico** integrando, sobre el tiempo, las desviaciones de la temperatura interior fuera de la zona de confort.

$$
GH_{cálido} = \sum_t \max\!\big( T_{op}(t) - T_{conf,sup}(t),\ 0 \big) \cdot \Delta t
$$

$$
GH_{frío}   = \sum_t \max\!\big( T_{conf,inf}(t) - T_{op}(t),\ 0 \big) \cdot \Delta t
$$

donde:

- $T_{op}$ = [[Temperatura-Operativa]] (o $T_{aire}$ como aproximación)
- $T_{conf,sup}$, $T_{conf,inf}$ = límites superior e inferior de la zona de [[Confort-Adaptativo]]
- $\Delta t$ = paso temporal en horas (0.1667 h para paso de 10 minutos)

Unidades: **°C·h**.

> "Lo malo de los grados-hora es que sus unidades son horribles. Pero es uno de los mejores parámetros — combina tiempo + magnitud."

## Por qué supera otras métricas

Una métrica única casi nunca cuenta toda la historia. Anti-patrones:

| Métrica única | Qué pierde |
|---------------|-----------|
| **Promedio anual** | La oscilación. Una casa estable a 26 °C y otra que oscila 18-34 °C tienen el mismo promedio |
| **Amplitud** | El centro. Dos casas con la misma amplitud pero centros distintos tienen confortabilidad muy distinta |
| **Máximo** | El tiempo. Un pico aislado y un día completo en máximo cuentan igual |
| **% del año en confort** | La magnitud de la salida. 1°C arriba durante 10 h cuenta igual que 10°C arriba durante 1 h |

**Grados-hora** combina las dos dimensiones que faltaban a las anteriores: **tiempo** (al integrar) y **magnitud** (la desviación entra al cuadrado del peso si se quisiera, pero el integrando lineal ya distingue magnitudes).

> Una hora a 5°C arriba del confort cuenta **5 grados-hora**.
> Cinco horas a 1°C arriba cuentan **5 grados-hora también**.
> Esto es una elección — penaliza igualmente la duración y la magnitud.

Si quieres penalizar más el disconfort severo: usar **grados-hora al cuadrado** (raro en la práctica).

## Esquema visual

```
T (°C)
 │           ╱╲   ← T_op interior
 │          ╱  ╲
35 ──────╱──────╲─────────  ← T_conf,sup
 │       ▒▒▒▒▒▒▒        ← área cálida = GH_cálido
 │      ╱        ╲
 │     ╱          ╲
30 ───────────────────────  ← T_neutralidad
 │
26 ──────────────────────── ← T_conf,inf
 │
        06h  12h  18h
```

Las **áreas sombreadas** entre la curva interior y la banda de confort son los grados-hora — cálidos (arriba de la banda) y fríos (debajo). Se calculan separadamente y se reportan separadamente.

## Cálculo en Python

Patrón idiomático en pandas (replicado de [[../notebooks/007_DDH]]):

```python
from iertools.read import read_sql

sim  = read_sql("../osm/005_caso_base/run/eplusout.sql", alias=True).data
Ti   = sim["Ti_ESTE"]   # o T_op si se pidió
To   = sim["To"]

# T de neutralidad mensual (Humphreys-Nicol)
To_m = To.groupby(sim.index.month).mean()
Tn_m = 13.5 + 0.54 * To_m

# Banda — elegir según el modelo (ver advertencia abajo)
banda = 3.5     # Humphreys-Nicol / ASHRAE 55
# banda = 1.25  # Morillón (clima estable)

# Broadcast: serie mensual → serie temporal completa
Tn_serie = pd.Series(sim.index.month.map(Tn_m), index=sim.index)
Tn_sup   = Tn_serie + banda
Tn_inf   = Tn_serie - banda

# Δt en horas (paso de 10 min)
dt_h = 1/6

# Grados-hora cálidos y fríos
GH_calido = (Ti - Tn_sup).clip(lower=0).sum() * dt_h
GH_frio   = (Tn_inf - Ti).clip(lower=0).sum() * dt_h
```

### Notas sobre el snippet

- **`groupby(index.month).mean()`** agrega por número de mes (1..12) sin importar el año — más limpio que `resample("ME").mean()` cuando lo que se quiere es mapear de vuelta a la serie temporal.
- **`pd.Series(index.month.map(Tn_m), index=sim.index)`** vectoriza el broadcast del valor mensual a cada timestep. Más limpio que el lambda equivalente `df.index.to_series().map(lambda t: Tn_m[t.month])`.
- **`.clip(lower=0)`** es pandas-native; equivalente a `np.maximum(..., 0)` pero sin importar NumPy.

### Cuál `banda` (ΔT) usar — atención

El valor **NO es siempre 3.5 °C**. Depende del modelo adaptativo elegido:

| Modelo | ΔT |
|--------|-----|
| Humphreys-Nicol / ASHRAE 55 estándar | 3.5 °C |
| Morillón (climas mexicanos, variable) | 1.25 - 4 °C |

Ver [[Confort-Adaptativo#modelo-de-morillón-el-δt-mexicano|Morillón en Confort-Adaptativo]]. Para el [[../notebooks/007_DDH]] se usa 1.25 (Morillón).

> **Antes de comparar GH entre simulaciones**: usar siempre la **misma banda** en todas. Comparar GH calculados con bandas distintas no tiene sentido — banda estrecha siempre dará GH altos.

### Magnitudes típicas

Para una zona en Cuernavaca con un cubo simple sin protecciones, con banda 3.5 los GH cálidos típicamente caen en el rango **5,000-15,000 °C·h/año**. Con banda 1.25 (Morillón) los números se multiplican por ~2-3× — números grandes, sus unidades son "feas" como advierte el profesor.

## Aplicación al proyecto final 2026-2

La clase 012 fija cómo se aplica al proyecto:

- Calcular GH **sólo en el / los mes(es) crítico(s)** identificados con CONUEE — no anualmente.
- Acompañar GH cálido / frío con el **promedio mensual del máximo (o mínimo) diario** de la T del aire interior — sirve de sanity check al lector.
- Reportar **caso × mes × estrategia**: una matriz si hay dos meses críticos.
- Si el clima asignado es extremoso (cálido + frío), **priorizar** explícitamente — una estrategia que reduce GH cálido suele subir GH frío y viceversa.

Detalle del encuadre completo en [[../classes/012-ProyectoFinal]].

## Reporte comparativo — la tabla del proyecto final

Métrica natural para el [[Estudio-Parametrico|estudio paramétrico]] del proyecto final. Se reporta como **diferencia relativa** vs caso base:

| Caso | GH cálido | GH frío | ΔGH cálido vs base | ΔGH frío vs base |
|------|-----------|---------|---------------------|--------------------|
| Caso base | 8,500 | 200 | — | — |
| + Color claro | 6,200 | 220 | **−27%** | +10% |
| + Aleros | 7,100 | 210 | **−16%** | +5% |
| + Aislamiento | 7,800 | 180 | −8% | −10% |
| Combinado | 4,500 | 240 | **−47%** | +20% |

Lectura:

- **Caso base** está en clima cálido (GH cálido >> GH frío).
- **Color claro** es la estrategia individual más efectiva para reducir GH cálido.
- **Aislamiento** ayuda menos para reducir GH cálido pero baja GH frío.
- **Combinado** sinergiza mejor que la suma de individuales.

## Trade-off cálido vs frío

Una estrategia que reduce GH cálido a menudo **aumenta** GH frío:

- Color claro → menos absorción solar → más frío en invierno.
- Aleros → bloquean sol → más frío en invierno.
- Más masa térmica → atenúa picos pero también valles.

El reporte separado de GH cálido/frío revela este trade-off. **Sumar los dos en una sola métrica los oculta** — anti-patrón.

En climas con **doble extremo** (Monterrey, Sonora) el trade-off es severo. En climas predominantemente cálidos (Yucatán, Guerrero) el GH cálido domina y el trade-off es manejable. Por eso el profesor recomienda **no escoger climas con doble extremo** para el proyecto final (consejo de [[../classes/002-ConceptosBasicosBalancesCalor]]).

## Sobre qué temperatura se calcula

El cálculo "correcto" se hace sobre [[Temperatura-Operativa]] — captura el efecto radiativo local. Cuando no se pidió T operativa como output, $T_{aire}$ es una aproximación aceptable salvo en casos con:

- Sol incidiendo sobre superficies internas (ventanas grandes orientadas al sol).
- Superficies muy calientes/frías cerca del usuario.
- Plancha de concreto exterior en línea de vista.

Detalle en [[Temperatura-Operativa]].

## Métricas auxiliares útiles

Aunque GH es la métrica principal, conviene reportar también:

| Métrica | Para qué |
|---------|----------|
| **% del año en confort** | Más comprensible para audiencia no técnica |
| **T máxima / mínima / promedio** | Sanity check de la simulación |
| **Reducción relativa de radiación incidente** | Para evaluar específicamente sombreamiento |

Nunca reportar **una sola métrica** — siempre acompañarla con su contexto.

## Clases relacionadas

- [[../classes/008-ShadingVentanas]] — explicación pizarrón del concepto y comparación con otras métricas
- [[../classes/012-ProyectoFinal]] — aplicación al proyecto: mes crítico, matriz caso × mes × estrategia, priorización en climas extremosos
- [[../classes/013-CalculoGradosHoraDisconfort]] — implementación en vivo en pandas: Humphreys-Nicol mensual, banda de Morillón, `.clip(lower=0)`, máscaras de tres colores; hallazgo Chilpancingo (Tn constante en climas estables)

## Libretas relacionadas

- [[../notebooks/007_DDH]] — implementación completa del cálculo con banda de Morillón sobre el modelo de dos zonas; patrones `groupby(index.month)`, `index.month.map`, `.clip(lower=0)`
