---
title: Warm-up Period
type: concepto
tags: [concepto, energyplus, simulacion, condicion-inicial, masa-termica]
aliases: [warmup, periodo de calentamiento, periodo de warming up]
clases: [004]
updated: 2026-05-02
---

# Warm-up Period

## Qué es

Mecanismo de Energy Plus para "olvidar" la **condición inicial arbitraria** y entrar en un régimen oscilatorio permanente. E+ inicializa todas las temperaturas (zonas térmicas y temperaturas nodales de los sistemas constructivos) a **23 °C** — un valor fijo, no negociable.

Como en una simulación dependiente del tiempo la condición inicial **importa mucho** (sobre todo con masa térmica), arrancar el cálculo directo desde 23 °C contaminaría los primeros días de resultados con un transitorio artificial.

> "Si yo voy a hacer un experimento de 30 minutos, no es lo mismo que vaya a poner en contacto un material con algo a −30 °C. Por supuesto que el resultado después de 30 segundos va a depender de cuál era la condición inicial de mi material."

## Cómo lo resuelve E+

Antes de empezar la simulación "real", E+ **repite el primer día** (o la primera semana, según configuración) hasta que el cambio de temperatura entre repeticiones cae bajo un criterio de convergencia (típicamente ~0.1 °C).

Esquemáticamente, suponiendo que el primer día es el 1 de enero:

1. Día 1 inicializado a 23 °C → la simulación termina ese día en, por ejemplo, 25 °C.
2. Repetición: vuelve a empezar el día 1 con 25 °C de condición inicial → termina en 24.5 °C.
3. Repetición: empieza con 24.5 °C → termina en 24.4 °C.
4. La diferencia entre repeticiones cae bajo el criterio (~0.1 °C) → **converge**.
5. Solo entonces E+ avanza al día 2 (2 de enero) y empieza la simulación real.

## Cuántos días tarda

Depende de **dos factores**:

| Factor | Efecto |
|--------|--------|
| **Masa térmica** del sistema constructivo | Más masa → tarda más en olvidar la condición inicial (más inercia). Un cubo de aluminio sin masa converge en pocos pasos; uno de concreto puede requerir varias repeticiones. |
| **Severidad del clima** vs. los 23 °C iniciales | En climas templados (Temixco) la diferencia con los 23 °C es chica — converge rápido. En climas extremos (invierno canadiense) la diferencia es grande — tarda más. |

E+ reporta en el `.err` cuántos días se requirieron. Para un cubo simple sin mucha masa: típicamente 3 días.

## Implicación crítica: el día de inicio importa

Después del warm-up, **la simulación arranca el día específico** que el usuario configuró (default: 1 de enero). Ese día queda permanentemente afectado por:

- El warm-up convergió usando ese día como base.
- El estado de la edificación al cerrar ese día depende de los días previos del clima — pero el warm-up "inventa" esos días previos repitiendo el mismo día, **no** los días reales anteriores.

> La edificación tiene **memoria del clima reciente** (típicamente días, no semanas). El primer día simulado no tiene esa memoria correctamente.

### Regla práctica para comparaciones

Si se van a comparar varias simulaciones (caso base vs. variantes), **todas deben empezar el mismo día**. Comparar:

- Una simulación que arranca el 1 de enero.
- Una variante que arranca el 1 de febrero.

…introduce un sesgo: la edificación de febrero recordará un "enero ficticio" repetido, distinto al enero real que recordaría la simulación de enero. Para evaluar un día específico (ej. 2 de febrero), conviene arrancar varios días antes y descartar el principio.

## Configuración

En el IDF/OSM:

- `Building` → **Maximum Number of Warmup Days**, **Minimum Number of Warmup Days**, **Loads Convergence Tolerance Value**, **Temperature Convergence Tolerance Value**.
- Por default: max 25 días, min 1 día, tolerancia de temperatura 0.4 °C.

## Clases relacionadas

- [[../classes/004-InterpretandoMensajesConstructionSets]] — explicación detallada con dibujo a mano del proceso de convergencia
