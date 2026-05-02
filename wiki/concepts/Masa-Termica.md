---
title: Masa Térmica
type: concepto
tags: [concepto, masa-termica, inercia, transferencia-calor]
aliases: [thermal mass, inercia termica, capacidad termica]
clases: [002, 004]
updated: 2026-05-02
---

# Masa Térmica

## Definición

Capacidad de un material o sistema constructivo para **almacenar energía térmica**. Físicamente:

$$
C = \rho \cdot c_p \cdot V \quad \text{[J/K]}
$$

donde ρ es la densidad, $c_p$ el calor específico y V el volumen. A nivel de superficie, también suele expresarse por área:

$$
C/A = \rho \cdot c_p \cdot \text{espesor} \quad \text{[J/m²K]}
$$

## Por qué importa

La masa térmica es el motivo por el que se necesita un **modelo de transferencia de calor dependiente del tiempo**. Sin masa térmica, la temperatura interior respondería instantáneamente al clima exterior. Con masa térmica:

- **Atenuación:** los picos de temperatura interior son menores que los del exterior.
- **Desfase:** los picos interiores ocurren con retraso respecto al exterior.

Estos dos efectos son palancas centrales del **diseño bioclimático en climas con oscilación día-noche** (mucha amplitud térmica): un muro masivo absorbe calor durante el día y lo libera durante la noche, suavizando la temperatura interior.

## Modelos sin masa térmica

Modelos basados solo en la **U** (resistencia térmica, $U = 1/R$) **ignoran la masa térmica**: asumen que el flujo de calor responde instantáneamente al gradiente de temperatura. Son válidos cuando:

- La masa del componente es despreciable (ej. **ventanas**).
- El sistema está en estado estable (no hay oscilación).

Por eso el [[Balance-de-Calor]] dependiente del tiempo (que sí incluye ρ, $c_p$) es esencial para edificaciones reales en climas con variación diurna. Modelos basados en U **no son adecuados** para evaluar diseño bioclimático en México.

## En Energy Plus

### Como propiedad del sistema constructivo

Cada material en un [[Sistemas-Constructivos]] tiene ρ, $c_p$ y espesor → contribuye a la masa térmica del sistema. Energy Plus la incluye automáticamente en el módulo **Conduction Transfer Function (CTF)**.

### Como objeto independiente

Energy Plus tiene un objeto especial **"masa térmica"** (`InternalMass`) para representar masa que **no es** parte de la envolvente — por ejemplo:

- Trabes y columnas que sobresalen del plano de un muro.
- Mobiliario interior pesado (libreros, paredes interiores no estructurales).
- Volúmenes de masa que se quitaron del modelo por simplificación geométrica.

La masa térmica adicional **no transfiere calor por conducción a otra zona** — solo intercambia con el aire de su zona por **convección** y con superficies cercanas por **radiación**. Almacena y libera energía manteniéndose cerca de la temperatura ambiente con desfase.

> **Caso típico:** un techo de losa con trabes integradas. Si modelas el techo como una superficie plana del espesor de la losa, pierdes la inercia de las trabes. Solución: agregar `InternalMass` con el volumen equivalente de las trabes.

### Particiones interiores y mobiliario

Una aplicación común de `InternalMass` (en Open Studio aparece como **Interior Partitions**) es modelar **paredes ligeras a media altura** (cubículos), **mobiliario** (libreros, escritorios, sofás), y otros volúmenes que aportan masa pero no separan zonas térmicas.

> "Esa idea de que las casas abandonadas son frías es porque no hay masa térmica. Una casa sin muebles tiene menos masa térmica — los muebles, libros, todo lo que esté ahí genera masa térmica y eso permite que las variaciones de temperatura sean menores."

Físicamente: la masa interior absorbe radiación que entra por ventanas durante el día (no la deja calentar el aire instantáneamente) y la libera de noche. Reduce la amplitud térmica que el habitante percibe.

En Open Studio, las particiones internas se asignan desde el [[Construction-Set]] (slot **Interior Partitions**) o se crean como objetos `InternalMass` directamente en E+.

## Compromiso espesor vs. área

Cuando se intenta consolidar masa en un volumen equivalente, hay que tener cuidado:

- Si dejas el área igual y subes el espesor → más masa, pero el flujo de calor se difiere más → desfase mayor.
- Si subes el área y bajas el espesor → más masa, pero también más superficie de intercambio → comportamiento distinto.

Mantener la físicia equivalente requiere preservar tanto la masa total ($\rho \cdot c_p \cdot V$) como el área de intercambio relevante.

## En el curso

**No se usa explícitamente** en los ejercicios del taller (simplificación de [[../classes/001-IntroduccionTallerIDB]]), pero se introduce como concepto. Las diferencias entre estrategias (color, ventanas, aleros) se evalúan **incluyendo la masa térmica de los sistemas constructivos** (CTF la maneja automáticamente).

## Clases relacionadas

- [[../classes/002-ConceptosBasicosBalancesCalor]] — introducción al concepto y su rol en el modelo dependiente del tiempo
- [[../classes/004-InterpretandoMensajesConstructionSets]] — particiones interiores y mobiliario como masa térmica vía `InternalMass`
