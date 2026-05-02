---
title: Construction Set
type: concepto
tags: [concepto, openstudio, construction, asignacion-masiva]
aliases: [construction set, conjunto de construcciones, default construction set]
clases: [004, 006]
updated: 2026-05-02
---

# Construction Set

## Qué es

Plantilla de Open Studio que **mapea automáticamente** un sistema constructivo a cada superficie del modelo según dos criterios:

1. **Tipo de superficie** — Wall, Roof, Floor (ver [[Tipos-Superficie]]).
2. **Condición de frontera** — Outdoors, Surface (interzona), Ground, Adiabatic (ver [[Condiciones-de-Frontera]]).

En lugar de arrastrar la construction una por una a cada superficie (insoportable cuando hay 200 superficies), se define un Construction Set, se asigna a la edificación, y Open Studio rellena las constructions de todas las superficies que **caen en el match**.

## Estructura del mapeo

Un Construction Set tiene **slots** organizados por la combinación tipo × condición. Slots típicos:

| Slot | Aplica a |
|------|----------|
| **Exterior Surface — Wall** | Muros con condición Outdoors |
| **Exterior Surface — Roof** | Techos con condición Outdoors |
| **Exterior Surface — Floor** | Pisos con condición Outdoors (raro) |
| **Interior Surface — Wall** | Muros con condición Surface (interzona) |
| **Interior Surface — Roof / Floor** | Techos/pisos entre zonas |
| **Ground Contact — Wall / Roof / Floor** | Superficies con condición Ground |
| **Sub Surface — Exterior Window** | [[Ventanas]] en muros con condición Outdoors |
| **Sub Surface — Interior Window** | Ventanas entre zonas térmicas |
| **Sub Surface — Door / Glass Door / Skylight** | Otras sub-superficies |
| **Interior Partitions** | Particiones internas (no separan zonas — solo agregan masa) |
| **Adiabatic** | Slots análogos para superficies adiabáticas |

Para cada slot ocupado, Open Studio asigna esa construction a las superficies que cumplen el match.

## Defaults vs. sobreescritura local

En la pestaña **Spaces → Surfaces**, la columna **Construction** muestra:

- **Verde** = la construction viene del Construction Set asignado a la edificación (es el default heredado). Si cambias el set, esta superficie cambia automáticamente.
- **Sin color (negro)** = la construction está sobreescrita localmente para esta superficie. Ya no responde a cambios del Construction Set.

> Pista visual útil: si **ves todo verde**, el Construction Set está bien definido y rige todo el modelo. Si ves negros mezclados, hay sobreescrituras locales — intencionales o accidentales.

## Cómo se asigna a la edificación

1. Pestaña **Facility** (icono de edificio, columna izquierda).
2. Sección **Default Construction Set** → arrastrar el Construction Set creado.
3. Verificar en preview 3D con **Render By → Construction** (o Boundary Conditions).

> En Facility también se puede:
> - Rotar la edificación respecto al norte sin tocar la geometría (campo `North Axis`).
> - Asignar **Default Schedule Set** (análogo para horarios).
> - Asignar **Default Space Type**.

## Slots vacíos en el set

Si una superficie del modelo cae en un slot **vacío** del Construction Set, Open Studio **no le asigna nada**:

- En el preview 3D la superficie puede **desaparecer** (no se renderiza sin construction).
- Al correr, E+ devolverá un severe ("missing material assignments"). Ver [[Mensajes-EnergyPlus]].

Fix: completar el slot del set, o asignar localmente esa construction a la superficie.

## Por qué importa

- **Productividad** — un edificio con 200 superficies es manejable.
- **Reusabilidad** — un Construction Set bien armado se puede aplicar a otros proyectos cambiando solo la geometría.
- **Auditabilidad** — toda la lógica de asignación de constructions queda en un solo objeto, no dispersa en cada superficie.
- **Estudios paramétricos** — cambiar el Construction Set de la edificación cambia el modelo completo en un click; combinado con [[Measures]] se pueden barrer escenarios.

## Construction Sets vs. Constructions

| Concepto | Qué es | Cuándo se define |
|----------|--------|------------------|
| **Material** | Capa única con propiedades térmicas | Pestaña Materials |
| **Construction** | Secuencia ordenada de materiales (ext→int) | Pestaña Constructions |
| **Construction Set** | Plantilla de asignación (qué construction va a qué tipo+condición) | Pestaña Construction → sub-pestaña Construction Sets |

Cadena: Materials → Constructions → Construction Set → Facility.

## Caso típico del curso

Para el cubo del curso 2 (dos zonas térmicas, sin ventanas, piso adiabático):

| Slot | Construction sugerida |
|------|----------------------|
| Exterior Surface — Wall | `Tabique_14cm` |
| Exterior Surface — Roof | `Concreto_15cm` |
| Adiabatic Floor | `Concreto_15cm` |
| Interior Surface — Wall | `Tabique_14cm` (si se quiere igual al exterior) |

## Clases relacionadas

- [[../classes/004-InterpretandoMensajesConstructionSets]] — introducción al concepto y demo en vivo
- [[../classes/006-DosZonasTermicasVentanasAleros]] — uso del slot Sub Surface para asignar construction de ventana
