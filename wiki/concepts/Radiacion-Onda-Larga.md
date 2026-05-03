---
title: Radiación de Onda Larga
type: concepto
tags: [concepto, radiacion, infrarrojo, balance-calor]
aliases: [long wave radiation, LWR, onda larga, radiacion infrarroja]
clases: [002, 008]
updated: 2026-05-02
---

# Radiación de Onda Larga

## Definición

Radiación electromagnética emitida por cuerpos a temperaturas terrestres (~250-350 K). Su espectro se concentra en el **infrarrojo lejano**, longitudes de onda ~3-100 μm.

Se distingue de la radiación de **onda corta** (espectro solar, ~0.3-3 μm), donde se concentra la energía del sol.

> **Insight didáctico:** en clases tradicionales de transferencia de calor a veces se enseña que "la radiación se desprecia" en problemas a temperatura ambiente. **Falso para edificaciones**: la onda larga puede representar el **60-70%** de la transferencia de calor en un muro.

## Las cuatro fuentes en el balance exterior

Para una superficie expuesta al exterior, el [[Balance-de-Calor]] incluye intercambio de onda larga con cuatro "fuentes":

$$
q''_{LWR} = q''_{ground} + q''_{sky} + q''_{air} + q''_{surroundings}
$$

Cada término sigue Stefan-Boltzmann:

$$
q''_{s \to i} = \varepsilon \, \sigma \, F_{s \to i} \, (T_i^4 - T_s^4)
$$

donde ε es la [[Emisividad]] de la superficie, $F_{s \to i}$ el [[Factor-de-Vista]], y $T_i$ la temperatura efectiva de la fuente.

### 1. Ground (suelo)

El suelo emite con su propia temperatura, que depende de:

- Temperatura del aire ambiente (acoplamiento)
- Propiedades del suelo (pastizal vs. zona volcánica vs. concreto)
- Humedad
- Radiación solar incidente sobre el suelo

Energy Plus calcula $T_{ground}$ internamente.

### 2. Sky (cielo)

> **Insight central:** el cielo tiene temperatura efectiva **muy baja** — típicamente **−15 °C** en una noche despejada (puede llegar a −30 °C en climas muy secos).
>
> Cualquier objeto que "vea" al cielo **se enfría** por intercambio de onda larga. De noche, una superficie expuesta al cielo puede llegar a **temperaturas inferiores a la del aire ambiente** — fenómeno aprovechable para enfriamiento radiativo.

La temperatura efectiva del cielo depende de la humedad, las nubes y la cantidad de cielo cubierto. Las **nubes** elevan T efectiva del cielo (no son tan frías) — las noches más frías son las despejadas.

Detalle del fenómeno y aplicaciones modernas en [[Enfriamiento-Radiativo-Cielo]].

### 3. Air (aire)

Tradicionalmente se enseña que el aire es "no participativo" en radiación de onda larga. Esto es válido para capas pequeñas (entre dos placas paralelas, por ejemplo). Pero la atmósfera completa **sí intercambia** porque hay mucha masa de aire en direcciones no perpendiculares.

El aire intercambia onda larga con la superficie a una temperatura cercana a la del aire ambiente.

### 4. Surroundings (alrededores)

Cualquier objeto sólido visible desde la superficie:

- Otro edificio cercano
- Un muro masivo de concreto
- Una placa de la planta solar adyacente

Si esos objetos están más fríos o más calientes, intercambian onda larga con la superficie según su factor de vista.

## Linealización con coeficiente HR

El intercambio radiativo en forma Stefan-Boltzmann es **no lineal** (T⁴). Para facilitar el acoplamiento numérico, se reescribe en forma "tipo convección":

$$
q''_{s \to i} = h_{r,i} \, (T_i - T_s)
$$

El coeficiente radiativo $h_{r,i}$ se calcula a partir de las temperaturas:

$$
h_{r,i} = \varepsilon \, \sigma \, F_{s \to i} \, (T_i + T_s)(T_i^2 + T_s^2)
$$

Esta es la forma típica que aparece en clases de transferencia de calor cuando se introduce un "coeficiente convectivo equivalente" para radiación.

## Aplicaciones de diseño

### Enfriamiento radiativo nocturno

Superficies con alta [[Emisividad]] expuestas al cielo nocturno se enfrían bajo la temperatura del aire. Aprovechable para:

- Sistemas de enfriamiento pasivo (techo radiativo)
- Materiales especiales que reflejan radiación solar y emiten en la ventana atmosférica → enfriamiento incluso bajo sol directo (ver el material que "se volvió famoso" — refleja visible y emite en 8-13 μm).

### Cool roofs (techos fríos)

Combinación de:
- α (solar) baja → no absorbe radiación solar
- ε (LW) alta → emite calor hacia el cielo eficientemente

Resultado: techo más frío que el ambiente bajo sol directo.

### Selección de emisividad

Para la mayoría de pinturas y acabados de construcción, ε ~ 0.85-0.95 (no hay mucho que elegir). Las excepciones son metales pulidos (ε muy baja) y materiales especializados.

## Clases relacionadas

- [[../classes/002-ConceptosBasicosBalancesCalor]] — introducción al concepto y a las cuatro fuentes
- [[../classes/008-ShadingVentanas]] — el cielo a −15 °C como sumidero radiativo; enfriamiento radiativo pasivo
