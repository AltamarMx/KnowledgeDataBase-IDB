---
title: Zona Térmica
type: concepto
tags: [concepto, zona-termica, modelado, simulacion]
aliases: [zona termica, thermal zone]
clases: [002, 003]
updated: 2026-05-02
---

# Zona Térmica

## Definición

**Volumen de aire delimitado por superficies**, donde Energy Plus resuelve la temperatura del aire interior aplicando un [[Balance-de-Calor]] de energía y masa.

Es la **unidad fundamental de modelado**: toda simulación se compone de una o más zonas térmicas conectadas entre sí (a través de superficies internas) y con el exterior.

## Por qué importa el volumen

Sin volumen definido **no se puede calcular el cambio de temperatura del aire**. Un mismo flujo de calor entrante:

- Eleva mucho la temperatura si el volumen es pequeño (poca masa de aire que calentar).
- Eleva poco la temperatura si el volumen es grande.

Por eso una zona térmica requiere superficies que **cierren** un volumen.

## Heurística para decidir si un espacio es zona térmica

> ¿En este espacio se siente una temperatura **del aire** diferente a la del exterior?
>
> - **Siempre sí** → es zona térmica.
> - **A veces sí, a veces no** → podría serlo.
> - **Siempre no** → no es zona térmica.

Importante distinguir **temperatura del aire** vs. **temperatura radiante**: uno puede sentir frío o calor por temperatura radiante (una pared muy caliente cerca, un piso muy frío) sin que el aire esté a una temperatura distinta. Eso no califica como zona térmica.

## Ejemplos del instituto

| Espacio | ¿Es zona térmica? | Razón |
|---------|-------------------|-------|
| Salón cerrado | Sí | Aire claramente diferenciado del exterior |
| Pasillos abiertos del edificio nuevo | No | Mismo aire que el exterior |
| Cubo de escaleras (cerrado, hace calor) | A veces sí | Difícil delimitar volumen |
| Cafetería actual (muy ventilada) | Difícil | Está prácticamente al exterior |
| Cafetería anterior (más cerrada) | Sí | Se sentía espacio diferenciado |

Cuando un espacio **podría** ser zona térmica pero el volumen es difícil de delimitar (por ejemplo, una cafetería con una pared abierta al exterior), una opción es **modelar un muro virtual con una ventana abierta** — pero hay que tomar decisiones de modelado y la incertidumbre crece.

## Cómo se conecta una zona térmica con el resto

- **Superficies exteriores** → balance con condiciones del clima (radiación, viento, T_amb).
- **Superficies interiores entre zonas** → cada zona resuelve su balance, el muro común es la interfaz.
- **Piso** → en el curso se modela como **adiabático** (ver [[Condiciones-de-Frontera]]).
- **Aperturas / ventanas** → permiten radiación + (eventualmente) intercambio de masa de aire.

## Cargas internas (no se usan en el curso)

Una zona térmica real tiene además:

- Personas (~70-100 W cada una; respiran, generan CO₂ y humedad)
- Equipos eléctricos
- Iluminación
- Cañones de luz / proyectores (con componente radiante y convectiva — la física difiere de un foco simple)

**En este curso** estas cargas no se modelan (simplificación de [[../classes/001-IntroduccionTallerIDB]]).

## Restricciones desde Energy Plus

- La zona térmica se define a partir de superficies; las superficies a su vez tienen las restricciones de E+ (1D perpendicular, solo líneas rectas).
- Para que el modelo sea válido, el aire dentro de la zona se asume **bien mezclado** ([[Mezclado-Perfecto]]) — toda la zona, una sola temperatura. Si esto no es físico (estratificación fuerte), conviene subdividir.

## Espacio vs Zona Térmica en Open Studio

Open Studio expone **dos entidades distintas** que en E+ son una sola: **Space** (volumen geométrico, puede agrupar Space Types reusables) y **Thermal Zone** (lo que E+ resuelve). Para casos sencillos se mapean 1:1 pero hay que crearlas y conectarlas por separado, con nombres distintos. Detalle en [[Espacio-vs-ZonaTermica]].

## Clases relacionadas

- [[../classes/002-ConceptosBasicosBalancesCalor]] — definición e introducción al concepto
- [[../classes/003-MiPrimeraSimulacion]] — distinción Space/Thermal Zone en Open Studio, mezclado perfecto
