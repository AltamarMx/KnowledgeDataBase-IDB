---
title: Grados-Hora de Disconfort
type: concepto
tags: [concepto, confort, metricas, bioclimatico, analisis, proyecto-final]
aliases: [grados hora, degree-hours, GH, GH-calido, GH-frio, disconfort acumulado]
clases: [008]
updated: 2026-05-02
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

Para cada paso temporal:

```python
import numpy as np
from iertools.read import read_sql, read_epw

sim = read_sql("../OSM/005_caso_base/run/eplusout.sql", alias=True)
epw = read_epw("../EPW/cuernavaca.epw", alias=True, year=2006, suppress_warnings=True)

df = sim.data
T_int = df.T_este  # o T_op si la pediste

# T de neutralidad mensual
T_mes = epw.TO.resample("ME").mean()
T_neut = 0.54 * T_mes + 13.5
delta = 3.5

# Mapear cada timestamp al T_neut de su mes
T_neut_at_t = df.index.to_series().map(
    lambda t: T_neut[T_neut.index.month == t.month].iloc[0]
)
T_sup = T_neut_at_t + delta
T_inf = T_neut_at_t - delta

# Δt en horas (paso de 10 min)
dt = 10 / 60

# Grados-hora cálidos y fríos
GH_calido = np.maximum(T_int - T_sup, 0).sum() * dt
GH_frio   = np.maximum(T_inf - T_int, 0).sum() * dt

print(f"GH cálidos: {GH_calido:>8.0f} °C·h")
print(f"GH fríos:   {GH_frio:>8.0f} °C·h")
```

Para una zona en Cuernavaca con un cubo simple sin protecciones, los GH cálidos típicamente caen en el rango **5,000-15,000 °C·h/año** — números grandes, sus unidades son "feas" como advierte el profesor.

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
