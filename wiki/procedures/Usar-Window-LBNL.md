---
title: Usar Window LBNL — construir un sistema y exportar SHGC/U
type: procedimiento
tags: [procedimiento, window-lbnl, ventanas, shgc, u-value, simple-glazing, energyplus]
aliases: [usar window, construir vidrio en window, exportar shgc, calcular shgc, workflow window]
clases: [014]
updated: 2026-05-22
status: pendiente-clase-015
---

# Usar Window LBNL — construir un sistema y exportar SHGC/U

⚠️ **Estado: pendiente de complementar.** El procedimiento se introduce conceptualmente en [[../classes/014-InfiltracionFloorspaceWindowLBNL]], pero **el uso paso a paso quedó programado para la clase 015** (29 mayo 2026). Esta página captura lo que está disponible hoy y se actualizará tras la última clase.

## Objetivo

Dado un sistema de ventana real (vidrios + gases + low-E + marco), obtener los **3 valores** que necesita el `WindowMaterial:SimpleGlazingSystem` de [[../tools/EnergyPlus]]:

1. **U-factor** [W/m²K]
2. **SHGC** (Solar Heat Gain Coefficient) [0-1]
3. **Visible Transmittance** [0-1]

## Workflow conceptual

```
1. Window LBNL → New Glazing System
2. Seleccionar vidrios de la base de datos (Glass Library)
3. Seleccionar gases entre vidrios (Gap Library)
4. Definir orden de capas (exterior → interior)
5. Calcular → Window devuelve U, SHGC, VT
6. Copiar los valores a Open Studio (Simple Glazing System)
7. Asignar el material a las construcciones de ventana
8. Correr la simulación
```

## Paso 1 — Abrir Window LBNL

Después de la instalación ([[Instalar-Window-LBNL]]):

1. Start Menu → `Window 7.8`.
2. Aparece la GUI con paneles (clásico estilo Windows XP).

## Paso 2 — Construir el sistema de vidrio

Usando la Glass Library:

1. `Glazing System` → `New`.
2. Asignar nombre descriptivo (e.g. `Doble_Argón_LowE_v1`).
3. Para cada capa, seleccionar de la **Glass Library** un vidrio con las propiedades deseadas.
4. Entre vidrios, asignar **Gap** con gas y espesor.

Capas típicas para un doble vidrio + low-E:

| Capa | Material | Espesor |
|---|---|---|
| 1 (exterior) | Vidrio claro 6 mm | 6 mm |
| 2 | Argón | 12 mm |
| 3 (interior) | Vidrio low-E 6 mm | 6 mm |

## Paso 3 — Calcular

`Calc` o `F9`:

1. Window resuelve el balance de energía multi-capa en estado estacionario.
2. Devuelve:
   - **U-factor** — calculado con condiciones de borde NFRC (∆T = 39°C aprox.).
   - **SHGC** — calculado con normal solar incidence + difusa estandarizada.
   - **VT** — calculado con espectro fotopico.

⚠️ Las condiciones internas de Window son **estandarizadas** (no son las del clima de la simulación). Esto es deliberado — permite comparar sistemas entre sí de forma reproducible.

## Paso 4 — Exportar los valores

En la pestaña de resumen del Glazing System aparecen los tres valores. Anotarlos o exportar reporte.

## Paso 5 — Pasar a Open Studio

En Open Studio:

1. `Materials → Add WindowMaterial:SimpleGlazingSystem`.
2. Nombre descriptivo (e.g. `Doble_Argon_LowE`).
3. Pegar los valores:

| Campo OS | Valor de Window |
|---|---|
| `U-Factor` | U-factor calculado |
| `Solar Heat Gain Coefficient` | SHGC calculado |
| `Visible Transmittance` | VT calculado |

## Paso 6 — Asignar a la construcción de ventana

1. `Constructions → New`.
2. Nombre (e.g. `Construccion_Doble_LowE`).
3. Agregar el material `Doble_Argon_LowE`.
4. Asignar la construcción a la ventana relevante (vía Construction Set o directamente al objeto window).

Detalle en [[../procedures/Agregar-Ventanas-OpenStudio]] y [[../procedures/Configurar-Construction-Set]].

## Paso 7 — Verificar y correr

1. Verificar en Open Studio que la nueva construcción aparece asignada a la(s) ventana(s) deseada(s).
2. Correr la simulación.
3. En Python: comparar con [[../procedures/Comparar-Simulaciones-Python]] caso base vs caso con vidrio nuevo.
4. Métricas relevantes: SHGC empíricamente verificable con `Surface Window Transmitted Solar Radiation Rate` ÷ `Surface Outside Face Incident Solar Radiation Rate`.

## Para qué sirve en el proyecto final 2026-2

Como **estrategia bioclimática**, cambiar el vidrio:

- Caso base: vidrio simple 3 mm — usar SHGC ≈ 0.84, U ≈ 5.8 W/m²K.
- Variante doble vidrio claro: calcular con Window, esperar U ≈ 2.8, SHGC ≈ 0.75.
- Variante con argón + low-E: U ≈ 1.8, SHGC ≈ 0.40-0.60.

Comparar resultados con métricas del proyecto (grados-hora cálidos/fríos).

> "Cada vez que pongo una ventana doble, también disminuye la iluminación, porque va de la mano."

Considerar también la reducción de **Visible Transmittance** — la casa pierde luz natural.

## Cuándo NO valdrá la pena

[[../concepts/Solar-Heat-Gain-Coefficient#veredicto-ventanas-dobles-según-contexto]]:

- Clima cálido sin AC: la mejora será marginal (las ganancias dominantes son por absortancia y radiación, no por conducción).
- Costo alto sin beneficio térmico → reportar y descartar.

## Pendientes para esta página

- Captura de pantalla de la GUI de Window.
- Demo concreto del Caso 3 con vidrios disponibles en MX.
- Comparación medida vs calculada para una ventana real.
- Cómo manejar **marcos** (separadamente o como parte del Glazing System).
- Cómo extraer **propiedades a múltiples ángulos** (input al modelo Complex de E+).

Se completará tras la clase 015.

## Clases relacionadas

- [[../classes/014-InfiltracionFloorspaceWindowLBNL]] — introducción conceptual + instalación
- _(clase 015 — pendiente de ingerir cuando se grabe)_

## Ver también

- [[../tools/Window-LBNL]] — qué es y cuándo usarlo
- [[Instalar-Window-LBNL]] — cómo instalarlo
- [[../concepts/Solar-Heat-Gain-Coefficient]] — qué calcula
- [[Agregar-Ventanas-OpenStudio]] — cómo asignar la construcción al objeto window
- [[Configurar-Construction-Set]] — cómo asignarla por defecto
