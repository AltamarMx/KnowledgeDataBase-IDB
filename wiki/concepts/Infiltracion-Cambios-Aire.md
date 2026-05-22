---
title: Infiltración y Cambios de Aire (ACH)
type: concepto
tags: [concepto, infiltracion, ach, ventilacion, energyplus, schedule, proyecto-final]
aliases: [infiltración, cambios de aire, ACH, air changes per hour, ventilación constante, infiltración constante]
clases: [014]
updated: 2026-05-22
---

# Infiltración y Cambios de Aire (ACH)

## Definición

**Infiltración** = flujo de aire **involuntario** entre el interior y el exterior, a través de grietas, juntas, marcos de ventanas y puertas, sin pasar por un sistema de ventilación intencional. En Energy Plus se modela como un flujo de aire entrante que se mezcla instantáneamente con el aire de la zona ([[Mezclado-Perfecto]]).

**Air Changes per Hour (ACH)** = unidad de medida de la infiltración:

$$
ACH = \frac{\dot V_{inf}}{V_{zona}}
$$

donde $\dot V_{inf}$ es el flujo volumétrico de aire infiltrado [m³/h] y $V_{zona}$ el volumen interior de la zona [m³]. Unidades: $h^{-1}$.

> "Los cambios de aire por hora son una unidad rara. Si tenemos dos espacios con diferente volumen, el flujo másico no es el mismo. Es el equivalente a que entra y sale el volumen completo cuántas veces por hora."

## Por qué ACH y no flujo másico en normativas

Las normativas usan ACH porque es **adimensional respecto al volumen** — un valor de ACH se puede aplicar a cualquier espacio sin recalcular. El flujo másico depende del volumen específico y obliga a hacer un cálculo distinto para cada cuarto.

Valor común en normativas para ventilación mínima: **0.5 ACH** (renovación del aire del cuarto cada 2 horas).

## La ecuación enriquecida de Energy Plus

E+ aplica **siempre** la misma ecuación a la infiltración, parametrizada para que se pueda activar diferentes comportamientos:

$$
Q_{inf}(t) = Q_{diseño} \cdot S(t) \cdot \left[ A + B\,|\Delta T(t)| + C\,v(t) + D\,v(t)^2 \right]
$$

donde:

| Símbolo | Significado |
|---|---|
| $Q_{diseño}$ | Valor de diseño (e.g. 0.5 ACH, o flujo másico, según el método) |
| $S(t)$ | Schedule fraccional 0-1 que modula a lo largo del tiempo |
| $A$ | Constant Coefficient — término base |
| $B\,\|\Delta T\|$ | Término de **flotabilidad** (diferencia de temperatura interior-exterior) |
| $C\,v$ | Término lineal de viento |
| $D\,v^2$ | Término cuadrático de viento |

### Modos comunes

| Modo | A | B | C | D | Comportamiento |
|---|---|---|---|---|---|
| **Constante** | 1 | 0 | 0 | 0 | Valor de diseño × schedule, sin dependencia del clima |
| **Por flotabilidad** | 0 | b | 0 | 0 | Solo `|ΔT|` afecta; útil cuando se quiere modelar efecto chimenea |
| **Por viento** | 0 | 0 | c | d | Velocidad del viento incidente sobre la ventana |
| **Físico realista** | a | b | c | d | Combinación de los anteriores |

> "Cuando uno le dice 'quiero que el coeficiente sea constante', hace todo esto cero y le pone aquí 1. Pero E+ siempre usa la misma ecuación."

### Coeficiente de descarga típico

Para infiltración física realista, $A + B\,\|\Delta T\| + C\,v + D\,v^2$ representa un coeficiente de descarga $C_d$. En ventanas y rendijas suele estar **alrededor de 0.6 hacia abajo**.

## El método de "Air Changes per Hour" en Open Studio

Cuando se elige `Air Changes per Hour` como método:

- E+ convierte internamente el valor a flujo volumétrico usando $V_{zona}$.
- Aplica la ecuación enriquecida (siempre).
- Si los coeficientes están en `[1, 0, 0, 0]` y el schedule en 1, el flujo se mantiene **prácticamente constante** — la única variación residual es por cambios de densidad del aire con la temperatura (3er decimal).

## Schedule fraccional

El schedule asociado a la infiltración **debe ser fraccional 0-1**. Si se intenta poner un valor `> 1`, E+ da error de validación al correr.

> "Que no les pase lo que me pasó: le quise poner un 2 y la fracción va entre 0 y 1."

### Multiplicación con el valor de diseño

El valor efectivo de ACH en un timestep dado es:

$$
\text{ACH}_{efectivo}(t) = \text{ACH}_{diseño} \cdot S(t)
$$

Patrón típico de ventilación nocturna:

| ACH de diseño | Schedule (24 h) | Resultado |
|---|---|---|
| 1.0 | 0 entre 8:00-20:00, 0.5 entre 20:00-8:00 | 0 ACH de día, **0.5 ACH** de noche |
| 0.5 | 1 entre 20:00-8:00, 0 resto | 0 ACH de día, **0.5 ACH** de noche |

Las dos formas son equivalentes. La segunda es más legible si solo se ventila por la noche.

## Por qué es una caricatura

Ver [[Caricatura-Computacional]]:

- **Mezclado instantáneo**: E+ asume que el aire entrante se mezcla perfectamente con el aire de la zona en cada timestep. En la realidad hay **jets direccionales** (la corriente que entra por una puerta hacia una ventana abierta). E+ no resuelve esto sin un módulo CFD externo.
- **Sin estratificación**: no hay gradientes verticales de temperatura por aire frío que se acumula abajo.
- **Constante en el tiempo**: la infiltración real depende del viento, de las diferencias de presión y de cómo están abiertas las ventanas — todo eso se colapsa en un número.

> "Aunque parezca limitación, es lo que está al alcance de esta clase. Modelos avanzados pueden calcular jets, ventilación cruzada, etc. — pero se sale del alcance."

### Cuándo es defendible la caricatura

- **Caso base sin ventilación real**: ACH = 0 elimina infiltración. La casa queda "sellada" — útil para comparaciones controladas.
- **Ventilación mecánica controlada**: un sistema HVAC con cambios de aire fijos sí es razonablemente constante.
- **Estrategia paramétrica**: comparar 0 ACH vs 0.5 vs 2 ACH para evaluar el orden de magnitud del efecto.

### Cuándo traiciona

- Análisis de **ventilación natural real** (depende del viento incidente, dinámica de plumas, presiones).
- **Estrategias de ventilación cruzada** que dependen de la geometría.
- **Estratificación térmica** en cuartos altos (lofts, naves industriales).

## Ventilación nocturna como estrategia bioclimática

Una estrategia válida cuando hay [[Masa-Termica|masa térmica]] interior:

- **Clima cálido**: ventilar de noche cuando la T exterior baja → el aire frío enfría las paredes y techos → durante el día la masa absorbe calor sin elevar la temperatura interior.
- **Clima frío**: ventilar en horas pico (2-4 pm) cuando la T exterior es máxima → aire caliente entra y calienta la masa → durante la noche la masa libera calor lentamente.

Implementación: schedule del ACH alto en los horarios deseados, cero el resto.

> "Si hace mucho calor, puedes ventilar en la noche y si tienes masa térmica eso va a enfriar la casa."

**Para el proyecto final 2026-2**: el profesor consultará con Miriam si se acepta como estrategia (clase 014). Ver [[../classes/014-InfiltracionFloorspaceWindowLBNL]].

## Variables de output asociadas

Solicitar al RDD ([[RDD-Variables-Disponibles]]):

| Variable | Unidades | Para qué |
|---|---|---|
| `Zone Infiltration Standard Density Volume Flow Rate` | m³/s | Flujo volumétrico con densidad estándar — útil para verificar ACH |
| `Zone Infiltration Outdoor Density Volume Flow Rate` | m³/s | Flujo con densidad del aire exterior — más físico |
| `Zone Infiltration Mass Flow Rate` | kg/s | Flujo másico — útil cuando se compara con HVAC |
| `Zone Infiltration Air Change Rate` | 1/h | ACH directo |
| `Zone Infiltration Sensible Heat Gain Energy` | J | Energía sensible que entra por infiltración — positivo o negativo |

Si se eligió ACH constante y el schedule está en 1, todas estas series deben ser **casi planas** (varían en el tercer decimal por densidad).

## Caso base del proyecto final 2026-2

Tras la consulta con Miriam (clase 014):

- **Caso base**: 0.5 ACH constante (schedule = 1).
- **Estrategia ventilación nocturna** (pendiente Miriam): schedule con valor 0 de día y 1 de noche; ACH de diseño 1 (resultado: 0 de día, 1 de noche).

> "Punto cinco de infiltración va a ser constante para todos. Realmente nada más es un pasito que nos acerca más a una edificación real."

Detalle del encuadre en [[../classes/012-ProyectoFinal]] y la actualización en [[../classes/014-InfiltracionFloorspaceWindowLBNL]].

## Procedimiento detallado

Ver [[../procedures/Agregar-Infiltracion-OpenStudio]].

## Clases relacionadas

- [[../classes/014-InfiltracionFloorspaceWindowLBNL]] — explicación + procedimiento + bug confesional del profesor (fraction=2)

## Ver también

- [[Schedules]] — los schedules fraccionales 0-1
- [[Mezclado-Perfecto]] — el aire entrante se mezcla en cada timestep
- [[Masa-Termica]] — por qué la ventilación nocturna funciona
- [[Caricatura-Computacional]] — lo que se descarta al usar ACH constante
