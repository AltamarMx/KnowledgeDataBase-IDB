---
title: Enfriamiento Radiativo al Cielo
type: concepto
tags: [concepto, radiacion, onda-larga, cielo, enfriamiento, bioclimatico]
aliases: [enfriamiento radiativo, radiative cooling, sky cooling, sky temperature]
clases: [008]
updated: 2026-05-02
---

# Enfriamiento Radiativo al Cielo

## Por qué una superficie puede estar más fría que el aire

Hecho contraintuitivo pero real: una superficie expuesta al cielo puede tener **temperatura inferior** a la del aire ambiente, sin violar la termodinámica.

> "Vamos a decir que la temperatura del aire está a 10 °C y la lámina podría estar a 7 °C. ¿Por qué? Porque está extrayendo calor por radiación al cielo."

## El cielo como sumidero radiativo

El **cielo despejado** tiene una temperatura efectiva radiativa de aproximadamente **−15 °C** (puede llegar a −30 °C en climas muy secos). Cualquier superficie con factor de vista no nulo al cielo intercambia [[Radiacion-Onda-Larga]] con él:

$$
q''_{LWR,sky} = \varepsilon \, \sigma \, F_{s \to sky} \, (T_{sky}^4 - T_s^4)
$$

Cuando $T_s > T_{sky}$ — siempre, durante operación normal — el flujo neto es **hacia el cielo**: la superficie pierde calor.

Si la superficie tiene **baja inercia térmica** (tarda poco en perder calor) y **buen factor de vista al cielo** (techo horizontal), puede enfriarse por debajo de $T_{aire}$ porque la pérdida radiativa supera la ganancia convectiva.

## Caso típico — techo de lámina metálica

Las **láminas metálicas** son el ejemplo clásico:

| Propiedad | Valor | Efecto |
|-----------|-------|--------|
| **Conductividad** | k ~50-200 W/m·K | Conduce calor casi instantáneamente |
| **Espesor** | ~1 mm | Casi sin masa térmica |
| **Emisividad** | ε ~0.9 (lámina común) | Buen radiador LWR |
| **Factor de vista al cielo** | ~1 (techo horizontal) | Máximo intercambio |

Resultado: en una **noche despejada y seca**, la superficie de la lámina puede estar **3-5 °C** debajo del aire ambiente. Esto se siente al tocar el techo de un cobertizo metálico de madrugada — está helado.

## Por qué solo se nota de noche

> "Las noches más frías son las noches menos nubosas — porque entonces el cielo está disponible para hacer intercambio radiativo de onda larga directo."

De día, la superficie recibe **radiación solar** que enmascara la pérdida LWR. De noche (o con sol bajo) la pérdida LWR domina y se nota.

Las **nubes** "tapan" el cielo — actúan como un cuerpo cercano a la temperatura del aire (no a −15 °C). Por eso noches despejadas son más frías que noches nubladas a la misma T inicial.

## Aplicación moderna — enfriamiento radiativo pasivo

Investigadores han desarrollado **películas selectivas** que:

- **Reflejan** toda la radiación solar (no se calientan al sol).
- Pero son **transparentes** o emisoras en la ventana atmosférica de longitud de onda larga (8-13 μm).

Resultado: un panel cubierto con la película, expuesto al cielo, **se enfría incluso de día** sin consumir energía.

> Patente referida por el profesor (~2017, Stanford et al.). Aplicaciones: enfriar agua de procesos, paneles solares (mejorar eficiencia), refrigeración pasiva en climas cálidos secos.

> "Si tú le pasas un fluido, lo voy a enfriar — porque está viendo al cielo y el cielo está a una temperatura menor."

## Implicaciones para diseño bioclimático en simulación

Energy Plus **modela este intercambio** vía la condición de frontera Outdoors (componente LWR del balance — ver [[Balance-de-Calor]]). Para que aparezca correctamente en una simulación:

1. La superficie debe estar como `Outdoors` con `Sun Exposure` y `Wind Exposure` activos.
2. La emisividad (`Thermal Absorptance` en Open Studio) debe ser realista (típicamente 0.8-0.9 para superficies arquitectónicas).
3. El EPW debe tener datos de **cobertura de nubes** — E+ ajusta la T efectiva del cielo según la nubosidad.

Variables de output relevantes:

- `Surface Outside Face Net Thermal Radiation Heat Gain Rate per Area` — balance LWR neto (negativo cuando la superficie pierde al cielo).
- `Surface Outside Face Sky Temperature` — T efectiva del cielo que E+ usa.

## Estrategias bioclimáticas que aprovechan el efecto

- **Techos de baja inercia** (lámina) en climas cálidos secos con buena ventilación nocturna — la lámina se enfría de noche, ventila el espacio.
- **Mass cooling** — masa térmica con vista al cielo (paredes, suelos exteriores) que disipa por la noche el calor acumulado de día.
- **Estanques sombreados** orientados al cielo — el agua se enfría por radiación nocturna.
- **Sistemas de enfriamiento radiativo activo** con fluidos circulando bajo paneles selectivos.

## Limitaciones

El efecto se diluye en climas:

- **Húmedos** — el vapor de agua bloquea la ventana atmosférica de 8-13 μm.
- **Nublados** — las nubes elevan T efectiva del cielo.
- **Urbanos** — la contaminación y el calor antropogénico reducen el contraste.

Por eso el enfriamiento radiativo pasivo es más útil en **climas cálidos secos** y zonas rurales — perfil de varias regiones de México.

## Clases relacionadas

- [[../classes/008-ShadingVentanas]] — discusión del fenómeno como respuesta a "¿por qué una edificación puede estar más fría que el aire exterior?"

## Ver también

- [[Radiacion-Onda-Larga]] — el mecanismo físico subyacente
- [[Factor-de-Vista]] — la geometría que controla cuánta superficie ve el cielo
- [[Balance-de-Calor]] — donde se incorpora el intercambio
