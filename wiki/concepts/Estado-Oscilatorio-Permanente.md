---
title: Estado Oscilatorio Permanente
type: concepto
tags: [concepto, numerico, regimen, enerhabitat, transferencia-calor, simplificacion]
aliases: [estado oscilatorio permanente, oscillatory steady state, regimen permanente, dia representativo]
clases: [010]
updated: 2026-05-02
---

# Estado Oscilatorio Permanente

## Definición

Régimen al que converge un sistema térmico cuando se le aplica un **forzamiento periódico idéntico** repetidas veces — la respuesta del sistema acaba siendo **periódica y reproducible** entre ciclos.

Para una edificación con clima de un día repetido N veces:

- **Día 1**: arranca con condición inicial arbitraria. Termina con cierto perfil de T en el muro.
- **Día 2**: arranca con el perfil del día 1, no con la condición inicial. Termina con un perfil ligeramente distinto.
- ...
- **Día N**: el perfil al inicio = perfil al final → **estado oscilatorio permanente alcanzado**.

> "Voy a resolver para 12, 18, 24, 6 y voy a buscar que el resultado deje de cambiar. Hago un permanente oscilatorio."

## Por qué importa

Permite **eliminar la dependencia de la condición inicial arbitraria** — un problema que también enfrentan simulaciones más complejas (ver [[Warm-up-Period]] de Energy Plus).

Una vez alcanzado:

- El **flujo de calor que entra** al muro = el que **sale** (en promedio en el ciclo) → balance energético neto = 0.
- La **T al inicio del día** = la **T al final del día** en cada nodo del muro.
- El comportamiento térmico es **comparable** entre simulaciones (todas en el mismo régimen).

## Aplicación en EnerHabitat

[[../tools/EnerHabitat]] usa este régimen como **objetivo del cálculo**:

1. Define un **día representativo** (promedio del mes seleccionado del EPW).
2. Inicializa T en todos los nodos = $\overline{T}_{aire,día}$.
3. Resuelve la PDE de transferencia de calor con paso temporal de 1 hora.
4. **Repite el día** hasta que el perfil de T en el muro converge entre repeticiones.
5. Reporta solo el último día — ese es el "estado oscilatorio permanente".

Cuántos días tarda en converger:

| Sistema constructivo | Días típicos |
|------|--------------|
| Aluminio o vidrio (sin masa) | 1-2 |
| Tabique (masa media) | 3-5 |
| Concreto masivo o adobe | 5-10 |
| Sistema con aislante grueso | 10+ |

## Diferencia con Warm-up de Energy Plus

| Aspecto | Warm-up E+ | Oscilatorio permanente EnerHabitat |
|---------|------------|-------------------------------------|
| Propósito | **Pre-cálculo** para olvidar la condición inicial | **Es el cálculo principal** — el resultado se reporta |
| Días repetidos | El primer día de simulación | Un día representativo del mes |
| Después de converger | Continúa la simulación al día 2 (real) | Reporta el ciclo convergido y termina |
| Realismo | Simula días distintos (clima real día a día) | Asume que todos los días son idénticos al representativo |

Detalle de Warm-up en [[Warm-up-Period]].

## Limitación de validez

> El oscilatorio permanente **asume que todos los días son iguales** — caricatura útil pero no real.

En la realidad:

- El día se va calentando o enfriando según la estación.
- Hay días anómalos (frente frío, lluvia, ola de calor).
- La T al inicio del día NO es igual a la del final → no hay oscilatorio permanente real.

Esto es por lo que EnerHabitat es una herramienta de **primeras decisiones**, no de cálculo realista de energía / emisiones / confort. Para eso → Energy Plus.

> "En la vida real no, porque estoy agarrando el día y lo estoy repitiendo. Es como si todos los días fueran iguales, pero eso no sucede. Por eso EnerHabitat es una caricatura, muy buena, pero caricatura."

## Día representativo — cómo se construye

EnerHabitat construye el día representativo del mes seleccionado promediando, paso temporal por paso temporal, los datos del EPW:

```
T_aire(00:00, día_rep) = mean(T_aire(00:00, día 1 del mes), T_aire(00:00, día 2), ..., T_aire(00:00, día N))
T_aire(01:00, día_rep) = mean(T_aire(01:00, día 1), ..., T_aire(01:00, día N))
...
T_aire(23:00, día_rep) = mean(T_aire(23:00, día 1), ..., T_aire(23:00, día N))
```

Análogo para radiación solar y demás variables del EPW. El resultado captura el **comportamiento promedio del mes** sin extremos — apto para análisis comparativo de sistemas constructivos pero no para casos extremos.

## Forma equivalente — análisis de Fourier

Matemáticamente, un sistema lineal con forzamiento periódico converge al estado oscilatorio permanente cuyas armónicas son la respuesta del sistema a las armónicas del forzamiento. Cada armónica del forzamiento se atenúa y se desfasa según la **función de transferencia** del muro.

Esto es lo que Energy Plus hace explícitamente con **Conduction Transfer Functions** (CTF) — descomposición en armónicas. EnerHabitat lo hace numéricamente (TDMA), pero el principio físico es el mismo.

## Métricas derivadas

Una vez en oscilatorio permanente, se calculan métricas como:

- **Amplitud** de la T interior ($\Delta T_i$) y de la T sol-aire ($\Delta T_{sa}$).
- **Pico** y **valle** de cada T.
- **Fase / desfase** entre el pico exterior y el interior → **tiempo de retraso**.
- **Energía transmitida por ciclo** = integral del flujo en una pared.
- **Factor de decremento**, etc.

Todas estas métricas requieren el oscilatorio permanente para tener sentido — antes de converger los valores son contaminados por la condición inicial. Detalle en [[Factor-de-Decremento]].

## Clases relacionadas

- [[../classes/010-EnerHabitatParte1]] — introducción al concepto y su uso en EnerHabitat
