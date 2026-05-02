---
title: Absortancia Solar
type: concepto
tags: [concepto, radiacion, propiedad-optica, color]
aliases: [absortancia, solar absorptance, alpha solar]
clases: [002]
updated: 2026-05-02
---

# Absortancia Solar

## Definición

Fracción de la radiación solar (de **onda corta**) incidente sobre una superficie que es **absorbida**. Se denota α (alpha).

$$
\alpha = \frac{\text{radiación absorbida}}{\text{radiación incidente}}
$$

Adimensional, en el rango $[0, 1]$. La fracción complementaria $(1 - \alpha)$ se reflecta (asumiendo superficie opaca).

## Valores típicos

| Color de superficie | α aproximada | Interpretación |
|---------------------|--------------|----------------|
| Blanco | 0.2 - 0.3 | Refleja 70-80% |
| Colores medios | 0.4 - 0.6 | |
| Negro / oscuro | 0.8 - 0.9 | Absorbe 80-90% |

## Rol en el balance de calor

En el [[Balance-de-Calor]] de la superficie exterior, la radiación de onda corta absorbida es:

$$
q''_{\alpha sol} = \alpha \cdot (I_{directa,\perp} + I_{difusa})
$$

donde la radiación directa se proyecta sobre la superficie según trayectoria solar aparente.

**La absortancia es una de las palancas más baratas y efectivas del diseño bioclimático**: cambiar de color (negro → blanco) puede reducir la radiación absorbida en ~70%.

## Onda corta vs. onda larga

La absortancia solar corresponde al espectro **visible + infrarrojo cercano** (donde está la mayor parte de la energía solar). Es **distinta** de la [[Emisividad]], que corresponde al infrarrojo lejano (intercambio radiativo entre cuerpos a temperaturas terrestres ~300 K).

Una superficie puede tener:

- α (solar) bajo y ε (onda larga) alto → "cool roof" — refleja sol, emite calor al cielo nocturno.
- α (solar) alto y ε (onda larga) bajo → metales oscurecidos — calientes en sol, retienen calor.

## En Energy Plus

La absortancia solar se especifica al definir el material exterior de un [[Sistemas-Constructivos]]. Es una propiedad **óptica de la superficie expuesta**, no del volumen.

## Estrategias bioclimáticas relacionadas

- Pintura blanca de techos en climas cálidos → estrategia bioclimática barata y de gran impacto.
- Selección de acabados exteriores según orientación.
- Materiales especiales: pinturas con baja absortancia solar y alta emisividad ("cool coatings", "radiative cooling materials").

## Clases relacionadas

- [[../classes/002-ConceptosBasicosBalancesCalor]] — introducción y rol en el balance
