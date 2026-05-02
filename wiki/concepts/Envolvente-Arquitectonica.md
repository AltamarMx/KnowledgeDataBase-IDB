---
title: Envolvente Arquitectónica
type: concepto
tags: [concepto, geometria, edificacion]
aliases: [envolvente, volumetría]
clases: [001, 002]
updated: 2026-05-02
---

# Envolvente Arquitectónica

## Definición

Conjunto de superficies que separan el interior de una edificación del exterior (o de otras zonas): muros, techo, piso, ventanas, puertas. Es la geometría sobre la que se resuelve el [[Balance-de-Calor]].

El profesor también la llama **volumetría**.

## Rol en una simulación

1. Define **qué superficies existen** y cómo están orientadas (orientación = clave para radiación solar).
2. Cada superficie recibe un **sistema constructivo** ([[Sistemas-Constructivos]]) que determina sus propiedades térmicas.
3. Cada superficie tiene una **condición de frontera** ([[Condiciones-de-Frontera]]) que la conecta con su entorno (otra zona, exterior, suelo, adiabática).
4. El motor calcula la transferencia de calor a través de cada superficie en cada paso de tiempo.

## Restricciones de Energy Plus

Independientemente de la GUI, **Energy Plus impone**:

- **Solo líneas rectas y superficies planas.** No hay líneas curvas, no hay superficies redondeadas, no hay ventanas circulares. Razón: el [[Factor-de-Vista]] de una superficie consigo misma se asume cero, lo que solo es válido para superficies planas.
- **Polígonos planos exclusivamente.** La envolvente se construye como un conjunto de polígonos planos cerrando un volumen.
- **Flujo de calor 1D perpendicular** a cada superficie (ver [[../tools/EnergyPlus]] para implicaciones).

## Limitaciones en este curso

Además de lo que impone Energy Plus, se usan **geometrías simples** (cubos con ventanas):

- Open Studio nativo no permite geometrías complejas.
- Convertir el curso en clase de dibujo aleja del objetivo.
- Las geometrías reales (techos a doble agua, volúmenes complejos) se modelan en programas de paga (Design Builder, Rhino+LadyBug, SketchUp) y en la siguiente materia.

## Vecinos / sombreamiento

En las simulaciones se pueden agregar **edificaciones vecinas** como objetos de sombreamiento. Decisión a tomar: incluirlos como geometría que sombrea, o representarlos como condición de frontera de la superficie afectada.

## Clases relacionadas

- [[../classes/001-IntroduccionTallerIDB]] — introducción a la envolvente y al editor de geometrías de Open Studio
- [[../classes/002-ConceptosBasicosBalancesCalor]] — restricción de líneas rectas, flujo 1D perpendicular
