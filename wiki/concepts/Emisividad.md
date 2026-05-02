---
title: Emisividad
type: concepto
tags: [concepto, radiacion, propiedad-optica, onda-larga]
aliases: [emisividad, emissivity, epsilon]
clases: [002]
updated: 2026-05-02
---

# Emisividad

## Definición

Eficiencia con la que una superficie emite radiación térmica relativa a la de un **cuerpo negro** a la misma temperatura. Se denota ε (epsilon).

$$
\varepsilon = \frac{\text{poder emisivo de la superficie}}{\text{poder emisivo del cuerpo negro a misma } T}
$$

Adimensional, $\varepsilon \in [0, 1]$.

Por la **ley de Kirchhoff** (válida en equilibrio térmico, longitud de onda por longitud de onda): la emisividad espectral es igual a la absortividad espectral. Para una banda específica (típicamente onda larga, ~3-100 μm), $\varepsilon \approx \alpha_{LW}$.

## Onda larga vs. onda corta

- **Emisividad** se refiere típicamente al **infrarrojo lejano** (longitudes de onda ~3-100 μm), que es donde emiten cuerpos a temperaturas terrestres (~250-350 K).
- La [[Absortancia-Solar]] se refiere al espectro **solar** (visible + infrarrojo cercano), donde está concentrada la energía del sol.
- **Una superficie puede tener emisividad muy distinta de absortancia solar.** Ejemplo: un "cool roof" tiene α (solar) ~0.2 pero ε (LW) ~0.9.

## Rol en el balance de calor

En la componente de [[Radiacion-Onda-Larga]] del [[Balance-de-Calor]] en superficie exterior, cada sub-término sigue Stefan-Boltzmann:

$$
q''_{s \to i} = \varepsilon \, \sigma \, F_{s \to i} \, (T_i^4 - T_s^4)
$$

donde:

- ε = emisividad de la superficie
- σ = constante de Stefan-Boltzmann ($5.67 \times 10^{-8} \text{ W/m}^2\text{K}^4$)
- $F_{s \to i}$ = [[Factor-de-Vista]] entre la superficie y la fuente (ground / sky / air / surroundings)

## Valores típicos

| Material | ε (onda larga) |
|----------|----------------|
| Acero pulido | 0.05 - 0.10 |
| Aluminio anodizado | 0.7 - 0.9 |
| Concreto | 0.85 - 0.95 |
| Pintura (cualquier color) | 0.85 - 0.95 |
| Vidrio | 0.84 - 0.95 |
| Agua | 0.96 |

> Notar que **la pintura tiene emisividad alta independientemente del color** — el color afecta la absortancia solar, no la emisividad de onda larga. Una pintura blanca tiene α (solar) ~0.25 y ε (LW) ~0.90: ideal para climas cálidos.

## En Energy Plus

La emisividad de onda larga se especifica al definir un material superficial expuesto en un [[Sistemas-Constructivos]]. Es la propiedad que controla cuánto calor **pierde** una superficie hacia el cielo nocturno (gran enfriador) y cuánto intercambia con suelo, aire y vecinos.

## Clases relacionadas

- [[../classes/002-ConceptosBasicosBalancesCalor]] — introducción al concepto y su rol en el intercambio de onda larga
