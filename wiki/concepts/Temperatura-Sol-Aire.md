---
title: Temperatura Sol-Aire
type: concepto
tags: [concepto, fisica, sol-aire, condicion-frontera, enerhabitat, balance-de-calor]
aliases: [sol aire, sol-aire, sa temperature, T_sa, temperatura sol-aire]
clases: [010, 011]
updated: 2026-05-02
---

# Temperatura Sol-Aire

## Definición

Temperatura **equivalente** del exterior que un muro o techo "siente", combinando los tres mecanismos de transferencia de calor en la superficie exterior en un solo número:

$$
T_{sa} = T_{aire} + \frac{\alpha \cdot I}{h_c} + \Delta T_{LWR}
$$

donde:

- $T_{aire}$ = temperatura ambiente del aire exterior (del EPW).
- $\alpha$ = [[Absortancia-Solar]] de la superficie.
- $I$ = radiación solar **incidente** sobre la superficie (proyectada, en W/m²).
- $h_c$ = coeficiente convectivo exterior (W/m²K).
- $\Delta T_{LWR}$ = corrección por intercambio de [[Radiacion-Onda-Larga]] con el cielo.

Análisis dimensional: $\frac{[W/m^2]}{[W/m^2 K]} = K$ → cada término tiene unidades de temperatura. ✓

## Por qué importa

La $T_{sa}$ encapsula los tres componentes del [[Balance-de-Calor|balance de calor exterior]] (radiación de onda corta absorbida + convección + LWR) en una **única temperatura equivalente**. Esto permite:

- **Simplificar** la condición de frontera: en lugar de tres flujos, un solo coeficiente convectivo equivalente actuando sobre $T_{sa}$.
- **Visualizar** el "clima térmico real" que la superficie experimenta — incluye el efecto del color, la orientación y la radiación.
- **Definir un límite termodinámico** para el [[Factor-de-Decremento|factor de decremento sol-aire]].

> "La temperatura sol-aire es una temperatura equivalente que toma en cuenta convección, radiación de onda corta y radiación de onda larga."

## El factor LWR

El término $\Delta T_{LWR}$ ajusta por el intercambio de onda larga con el cielo (que está más frío que el aire — ver [[Enfriamiento-Radiativo-Cielo]]):

| Geometría | $\Delta T_{LWR}$ |
|-----------|-------------------|
| **Muro vertical** (90° de inclinación) | **0 °C** (no ve mucho cielo, ve el ground también) |
| **Techo horizontal** (180°) | **−3.4 °C** (ve completamente el cielo, pierde calor por LWR) |
| Inclinación intermedia | Variación lineal entre los dos valores |

La corrección negativa para techos refleja que **un techo horizontal pierde calor adicional al cielo** que un muro no pierde con la misma intensidad.

## Comportamiento típico durante un día

Para un muro al **este** en un día despejado:

```
T (°C)
 │
50 │                ╱╲
   │               ╱  ╲           ← T_sa (sol incide directo en la mañana)
40 │              ╱    ╲
   │             ╱      ╲
30 │            ╱        ╲___       
   │           ╱             ╲      ← T_aire (línea base)
25 │ ━━━━━━━━╱
   │
   └─────────────────────────────
   00h  06h    12h    18h    24h
```

- Antes del amanecer: $T_{sa} \approx T_{aire}$ (no hay radiación, $\alpha I = 0$).
- En la mañana (sol directo en muro este): $T_{sa}$ **dispara muy arriba** de $T_{aire}$.
- Después del mediodía: solo radiación difusa → $T_{sa}$ baja a cerca de $T_{aire}$.
- Pico para muro este: ~9-10 AM. Para muro oeste: ~3-4 PM. Para muro sur (hem. norte): mediodía.

> "Después llega el mediodía y baja, pero esto tiene que ver con la radiación solar — la T sol-aire combina T + radiación."

## Asociación con [[Trayectoria-Solar]]

La forma de la curva $T_{sa}$ a lo largo del día **codifica la orientación del muro** — igual que la radiación incidente sobre la superficie. Patrones esperados:

| Orientación | Curva de $T_{sa}$ |
|-------------|---------------------|
| Norte (hem. norte) | Plana, cercana a $T_{aire}$ |
| Sur | Pico al mediodía solar |
| Este | Pico de mañana |
| Oeste | Pico de tarde |

Patrón inesperado en una simulación → pista de error en orientación o etiquetas.

## Aproximación, no exactitud

$T_{sa}$ es una **simplificación** — no captura todos los efectos del balance:

- **No considera la T transitoria de la pared** (asume $h_c$ constante).
- **No captura totalmente el LWR** (la corrección por inclinación es lineal — aproximación gruesa).
- **No incluye convección variable con el viento** (asume $h_c$ promedio).

Suficiente para análisis de **primeras decisiones** (EnerHabitat) — no apta para análisis fino tipo Energy Plus que sí resuelve cada componente por separado.

## Uso en EnerHabitat

[[../tools/EnerHabitat]] usa $T_{sa}$ como **forzamiento exterior** para resolver la transferencia de calor 1D dependiente del tiempo a través del sistema constructivo. La condición de frontera exterior se simplifica a:

$$
q''_{exterior} = h_c (T_{sa} - T_{superficie})
$$

Un solo coeficiente convectivo "equivalente" con $T_{sa}$ como temperatura de entrada — más sencillo de discretizar que los tres flujos por separado.

## En Energy Plus

E+ **no** simplifica con $T_{sa}$ — resuelve cada componente del balance directamente. Por eso E+ es más fiel pero también más caro computacionalmente. $T_{sa}$ se calcula igualmente como variable derivada útil para análisis.

## "Susto feliz" — el primer time step engaña

Caso real observado en clase 011: si miras solo el **primer valor** del día (típicamente las 00:00), $T_{sa}$ **no cambia** cuando modificas la absortancia o el coeficiente convectivo.

Razón: a las 00:00 la radiación solar es **cero** ($I = 0$). Como:

$$
T_{sa} = T_{aire} + \frac{\alpha \cdot I}{h_c} + \Delta T_{LWR}
$$

con $I = 0$ los términos $\alpha I / h_c$ se anulan → cualquier $\alpha$ o $h_c$ produce el mismo valor.

> "Es el primer time step del día. La radiación al inicio es cero. Se va a ver cuando empieza a haber radiación solar."

**Buena práctica**: para verificar efectos de cambios en α o $h_c$, **mirar el día completo** (especialmente horas con sol — mediodía para muros sur, mañana para este, tarde para oeste). No fijarse solo en el primer valor.

## Asociación T sol-aire ↔ wall específico

Observación crítica de la clase 011: la temperatura sol-aire **pertenece a un wall específico**. Si cambias:

- **Color** (absortancia α)
- **Orientación** (acimut)
- **Inclinación** (tilt)
- **Lugar** (otra geolocalización)
- **Período** (otro mes)

…la T sol-aire **cambia**. No es una variable global del clima — depende de las propiedades de la superficie.

> "La temperatura sol-aire le pertenece a ese muro y solo a ese muro."

**Anti-patrón**: pegar la T sol-aire de `wall_1` a los resultados de `wall_2` cuando los walls difieren en cualquiera de las dimensiones de arriba → análisis incorrecto sin error visible.

Solo es válido **compartir** la T sol-aire entre walls cuando todos los parámetros excepto el sistema constructivo son **idénticos**.

## Métricas asociadas

[[Factor-de-Decremento]] sol-aire = $\Delta T_i / \Delta T_{sa}$ — fracción de la amplitud de $T_{sa}$ que se transmite al interior. Adimensional, comparable entre climas, **siempre < 1** (porque $T_{sa}$ es el límite termodinámico real para una pared expuesta a sol + clima).

## Clases relacionadas

- [[../classes/010-EnerHabitatParte1]] — introducción al concepto en el contexto de EnerHabitat
- [[../classes/011-EnerHabitatParte2]] — "susto feliz" del primer time step y asociación T_sa ↔ wall específico
