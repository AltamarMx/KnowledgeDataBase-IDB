---
title: Sistemas Constructivos
type: concepto
tags: [concepto, materiales, construccion]
aliases: [construction, sistema constructivo]
clases: [001, 002, 003, 004, 006]
updated: 2026-05-02
---

# Sistemas Constructivos

## Definición

Secuencia ordenada de **materiales** (cada uno con propiedades térmicas y un espesor) que componen una superficie de la envolvente. Por ejemplo, un muro repellado:

```
[repellado exterior] - [tabique] - [repellado interior]
```

Cada capa tiene sus propias propiedades térmicas (conductividad, densidad, calor específico, absortancia/emitancia para superficies expuestas) y un espesor. El sistema constructivo recibe un nombre por el usuario y se asigna a una o más superficies de la envolvente.

> **Convención de orden:** los sistemas constructivos se describen **de exterior a interior**. La primera capa es la que da al exterior (o a otra zona); la última es la interior de la zona analizada.

## Cadena de definición en Energy Plus / Open Studio

1. **Materials** — definir cada material y sus propiedades.
2. **Construction** — ordenar materiales en una secuencia → sistema constructivo.
3. **Surface** — asignar el sistema constructivo a una superficie de la envolvente.
4. **Construction Set** (Open Studio) — agrupar sistemas constructivos por tipo de superficie y condición de frontera, y asignar el set a la edificación. Hace más rápida la asignación masiva. Detalle en [[Construction-Set]] y [[../procedures/Configurar-Construction-Set]].

## Por qué importa para el balance

Las propiedades de cada capa determinan:

- **Conducción** a través del muro (k, espesor) — entra en la ecuación de [[Balance-de-Calor]] dependiente del tiempo.
- **Capacidad térmica** del muro (ρ, cₚ, espesor) — controla la [[Masa-Termica]] de la edificación.
- [[Absortancia-Solar]] y [[Emisividad]] de la superficie expuesta — controlan ganancia/pérdida radiativa.

Cambiar el sistema constructivo es una de las palancas principales del diseño bioclimático.

## Restricción importante de Energy Plus

Energy Plus asume **material homogéneo en cada capa y a lo largo de toda la superficie**: no captura cambios laterales de material en una misma superficie (ej. una trabe embebida en un muro). Para representar esos casos:

- **Subdividir la superficie** en sub-superficies con sistemas constructivos distintos, o
- Usar el objeto **[[Masa-Termica]]** (`InternalMass`) para sumar la inercia que se "pierde" en la simplificación.

## Flujo concreto en Open Studio

1. Pestaña **Materials → Materials** (sub-pestaña). **No usar `No Mass Materials`** — esos no respetan masa térmica y son incorrectos para análisis dinámico.
2. Botón verde **+** → crear material → llenar `Roughness`, `Thickness`, `Conductivity`, `Density`, `Specific Heat`, `Thermal Absorptance` (=emisividad), `Solar Absorptance`, `Visible Absorptance`.
3. Pestaña **Constructions** → crear construction.
4. **Arrastrar materiales** desde el panel `My Model` (derecha) al construction — **uno por uno, en orden exterior → interior**.
5. Pestaña **Spaces → Surfaces** → asignar la construction a cada superficie individualmente (drag-and-drop), o usar **Default Construction Sets** para asignación masiva (ver clases siguientes).

> Los campos en **verde** en Open Studio son valores default. En cuanto se modifican pierden el color verde — pista visual útil para identificar qué se ha tocado.

## Convención típica del taller

Para los primeros ejercicios:

- **Muros**: tabique (un material) o tabique + repellado.
- **Piso y techo**: concreto.
- **Ventanas**: construction tipo `WindowMaterial:SimpleGlazingSystem` (U y SHGC).

## Materiales de ventana (no opacos)

Las ventanas no usan los mismos materiales que los muros. Para ventanas, E+ tiene categorías propias:

| Tipo | Caracterización | Cuándo usar |
|------|-----------------|-------------|
| **Glazing Window Material** | Capa por capa con propiedades ópticas (transmitancia/reflectancia/emisividad espectral) y conductividad | Ventana caracterizada experimentalmente; investigación de materiales nuevos |
| **Simple Glazing System** | 3 parámetros: U-factor, SHGC, Visible Transmittance | **Recomendado para el taller** — basta con datos de ficha técnica |

Detalle en [[Ventanas]].

## Relación densidad ↔ conductividad

Los materiales **siguen una correlación monotónica** entre densidad y conductividad — más denso, más conductivo. Útil como sanity check:

| Material | ρ (kg/m³) | k (W/m·K) |
|----------|-----------|-----------|
| EPS (poliestireno expandido) | 45 | 0.035 |
| Tabique rojo | 1400 | 0.7 |
| Concreto alta densidad | 2400 | 2.0 |

> Si una fuente da ρ y k que no respetan la correlación, sospechar — pueden estar mezcladas con datos de otros materiales.

## Pinturas e impermeabilizantes — no se modelan como capa

Espesores < 1 mm. Efecto térmico por conducción **despreciable**. Lo que importa es el **color** (absortancia solar). En el modelo no se agregan como capa — se asigna la absortancia a la superficie expuesta.

## Clases relacionadas

- [[../classes/001-IntroduccionTallerIDB]] — introducción al concepto y al editor de Open Studio
- [[../classes/002-ConceptosBasicosBalancesCalor]] — convención de orden, restricciones de homogeneidad, masa térmica
- [[../classes/003-MiPrimeraSimulacion]] — flujo concreto Materials → Construction → Surface en Open Studio
- [[../classes/004-InterpretandoMensajesConstructionSets]] — Construction Sets como atajo de asignación masiva
- [[../classes/006-DosZonasTermicasVentanasAleros]] — materiales de ventana, relación ρ-k, pinturas e impermeabilizantes
