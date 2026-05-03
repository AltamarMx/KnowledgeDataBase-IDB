---
title: Factor de Decremento
type: concepto
tags: [concepto, metricas, bioclimatico, sol-aire, enerhabitat, amplitud, retraso]
aliases: [decrement factor, factor decremento, FD, FD sol-aire, decrement factor sol-air, tiempo de retraso, time lag]
clases: [010, 011]
updated: 2026-05-02
---

# Factor de Decremento

## Definición

Métrica adimensional que mide **qué fracción de la amplitud térmica exterior** se transmite al interior de una edificación — captura el efecto combinado de **inercia térmica** (masa) y **aislamiento** (resistencia) del sistema constructivo.

$$
FD = \frac{\Delta T_{interior}}{\Delta T_{exterior}}
$$

donde $\Delta T = T_{max} - T_{min}$ en un ciclo (típicamente un día) en [[Estado-Oscilatorio-Permanente]].

| Valor | Interpretación |
|-------|----------------|
| 0 | El interior no oscila — masa térmica infinita o aislamiento perfecto |
| 1 | El interior oscila igual que el exterior — sin atenuación |
| > 1 | **Anómalo** con $T_{aire}$ exterior; explicación en la siguiente sección |

## Dos versiones — la trampa del FD ingenuo

Hay **dos versiones** del factor de decremento, según qué se use como referencia exterior:

### Factor de decremento ingenuo (FD)

$$
FD = \frac{\Delta T_i}{\Delta T_o}
$$

donde $\Delta T_o$ es la amplitud de la **T del aire exterior**.

**Problema crítico**: con sistemas constructivos de **alta absortancia** (color oscuro), la radiación solar puede calentar la pared **por encima** de la T del aire → la T interior puede oscilar más que la T del aire exterior → **FD > 1**.

> "El factor de decremento puede ser mayor que uno. La amplitud, si tengo un color con absortancia muy alta, va a absorber más energía, se va a calentar más y la T del interior puede ser mayor que la T del exterior. La T del aire **no es mi límite termodinámico**."

Conclusión: **FD ingenuo no es buen indicador** cuando la radiación solar es relevante. No es comparable entre climas con distinta intensidad solar.

### Factor de decremento sol-aire (FD_sa)

$$
FD_{sa} = \frac{\Delta T_i}{\Delta T_{sa}}
$$

donde $\Delta T_{sa}$ es la amplitud de la [[Temperatura-Sol-Aire|temperatura sol-aire]] — que sí incluye el efecto de la radiación absorbida.

**Propiedad clave**: $T_{sa}$ es el **límite termodinámico** real para la temperatura de la pared exterior. La T interior nunca puede oscilar más que $\Delta T_{sa}$:

$$
0 \leq FD_{sa} \leq 1
$$

> "Si le sale mayor que uno, algo está mal — porque ahora sí, este es mi límite termodinámico."

**Recomendación**: usar **siempre FD sol-aire** para análisis comparativo entre sistemas constructivos. Detalle en [[Temperatura-Sol-Aire]].

## Interpretación física

| FD_sa | Comportamiento del sistema constructivo |
|-------|------------------------------------------|
| > 0.9 | **Sin masa, sin aislante** — lámina metálica, vidrio sencillo. La oscilación entra casi sin atenuar |
| 0.5-0.9 | Sistemas ligeros con poco aislamiento |
| 0.2-0.5 | Tabique, ladrillo (masa media) |
| 0.05-0.2 | Concreto grueso, adobe — alta inercia |
| < 0.05 | Sistemas muy aislados o muy masivos |

Para [[Confort-Termico|diseño bioclimático]] en climas con alta amplitud diaria (climas secos): **buscar FD_sa bajo** — atenúa el pico cálido del día y el valle frío de la noche, manteniendo T interior estable.

## Tiempo de retraso (time lag)

Métrica complementaria al FD: **desfase temporal** entre el pico de la T exterior (o sol-aire) y el pico de la T interior:

$$
\phi = t(T_i^{max}) - t(T_{sa}^{max})
$$

Mayor desfase → mayor inercia térmica → la onda se "atrasa" más al pasar por el muro.

| Sistema | Tiempo de retraso típico |
|---------|---------------------------|
| Lámina metálica | < 1 h |
| Tabique 14 cm | 4-6 h |
| Concreto 15 cm | 6-8 h |
| Adobe 30 cm | 10-12 h |

**Aplicación bioclimática**: un retraso de 8 h significa que el pico térmico del exterior (mediodía) llega al interior **a las 20:00** — cuando la noche ya está fría afuera y se puede ventilar para disipar. Combinación clásica: **adobe + ventilación nocturna** en climas cálidos secos.

## Por qué FD bajo + retraso alto > solo FD bajo

Una pared con **mucho aislante** puede tener FD bajo pero retraso bajo (el aislante atenúa pero no retrasa). Una pared con **mucha masa** tiene FD bajo + retraso alto. La segunda es preferible para confort en climas con oscilación diaria fuerte.

> Las dos métricas juntas caracterizan el sistema constructivo. Reportar siempre **ambas** para comparar.

## Limitación — depende del clima

A diferencia de FD_sa (adimensional, comparable entre climas), el **tiempo de retraso depende del clima**:

- En climas con oscilación rápida (clima seco con noches frías), un retraso de 6 h es óptimo.
- En climas con oscilación lenta (clima húmedo con noches templadas), el retraso importa menos — el día y la noche son térmicamente similares.

## En EnerHabitat

[[../tools/EnerHabitat]] reporta automáticamente:

- **Factor de decremento** (FD ingenuo).
- **Factor de decremento sol-aire** (FD_sa).
- **Tiempo de retraso** (en horas).
- **Energía transmitida** (con bug actual — sale 0; mantener para versión corregida).

> Recomendación del profesor: confiar en **FD_sa** y **tiempo de retraso**. El FD ingenuo es solo informativo.

## Comparación con NOM-008 / NOM-020

Las normativas mexicanas usan el **coeficiente global de transferencia** $U = 1/R$ como métrica — basado en modelo independiente del tiempo. Esto **ignora la masa térmica** completamente.

FD_sa **sí captura** el efecto de la masa (un muro con misma U pero distinta masa tiene FD distinto). Por eso es métrica más rica para evaluar diseño bioclimático en climas mexicanos con oscilación diaria. Detalle en [[Posicion-Aislante]].

## Cálculo en EnerHabitat (simplificado)

```python
# Asumiendo serie temporal de un día en oscilatorio permanente
T_int = wall.solve()             # T interior (24 valores)
T_sa  = wall.tsa()               # T sol-aire (24 valores)
T_amb = location.epw["T_aire"]   # T del aire exterior (24 valores)

FD_ingenuo = (T_int.max() - T_int.min()) / (T_amb.max() - T_amb.min())
FD_sa      = (T_int.max() - T_int.min()) / (T_sa.max()  - T_sa.min())

# Tiempo de retraso en horas
hora_pico_int = T_int.idxmax()
hora_pico_sa  = T_sa.idxmax()
retraso       = (hora_pico_int - hora_pico_sa).total_seconds() / 3600
```

## Clases relacionadas

- [[../classes/010-EnerHabitatParte1]] — introducción al concepto y a las dos versiones (ingenua y sol-aire)
- [[../classes/011-EnerHabitatParte2]] — uso en estudios paramétricos en Python
