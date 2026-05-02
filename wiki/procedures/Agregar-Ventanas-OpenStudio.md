---
title: Agregar ventanas en Open Studio
type: procedimiento
tags: [procedimiento, openstudio, ventanas, glazing, sub-superficie, floorspacejs]
aliases: [agregar ventanas, crear ventana openstudio, glazing material]
clases: [006]
updated: 2026-05-02
---

# Agregar ventanas en Open Studio

Procedimiento para colocar ventanas en un modelo y asignarles un material de vidrio. Las ventanas son [[../concepts/Subsuperficie|sub-superficies]] que viven dentro de un muro.

## Pre-requisitos

- Modelo con geometría (al menos una zona térmica con muros exteriores).
- El modelo debe correr limpio antes de agregar ventanas — facilita el debug si algo falla.

## 1. Configurar el grid del editor

Las ventanas se colocan **sobre los vértices del grid** de FloorspaceJS. Para ventanas de tamaños no múltiplos enteros conviene reducir el grid:

1. Pestaña **Geometry → Editor**.
2. Esquina superior derecha del editor → cambiar el grid spacing a `0.25` o `0.5` m.

## 2. Crear o seleccionar un componente de ventana

En la barra de herramientas del editor:

1. Click en **Components** (icono de bloques en la columna izquierda).
2. Sección **Window Definitions** → si no hay ninguna, crear una nueva con el botón **+**.

Configurar el componente:

| Parámetro | Valor sugerido (taller) |
|-----------|--------------------------|
| **Name** | `vent_1m_x_1.2m` o descriptivo |
| **Window Type** | `FixedWindow` (default — sin abrir) |
| **WWR** o **Height + Width + Sill Height** | Especificar uno u otro |
| **Sill Height** | `0.91` m (default — altura de antepecho típica) |
| **Overhang Projection Factor** | `0` por ahora (los aleros se agregan en [[Agregar-Aleros-OpenStudio]]) |
| **Fin Projection Factor** | `0` por ahora |

> Si necesitas varias ventanas con tamaños distintos, **crea un componente por cada tamaño**. Si no, todas usarán el mismo.

## 3. Insertar las ventanas

1. Con el componente seleccionado en el panel, click en **Place Component** (cursor cambia).
2. Hacer click sobre los muros donde quieres ventanas — cada click coloca una.
3. Posición: el click define el **centro** de la ventana sobre el grid.

> Verificar que las ventanas se ven en planta como rectángulos azul claro sobre el muro.

## 4. Merge al modelo

1. Botón **Merge with Current OSM** (esquina superior derecha del editor).
2. Pestaña **3D View → Refresh**.
3. Las ventanas deben aparecer **transparentes** en el preview 3D.

> Si NO se ven transparentes: aún no tienen Construction asignada. Sigue al paso 5.

## 5. Crear el material de la ventana

Pestaña **Construction**. Hay **dos opciones**:

### Opción A — Simple Glazing System (recomendado)

1. Sub-pestaña **Materials** → sub-sub-pestaña **Window Materials → Simple Glazing System**.
2. Botón verde **+** → renombrar (ej. `vidrio_simple_3mm`).
3. Llenar:

   | Campo | Valor típico (vidrio sencillo) |
   |-------|--------------------------------|
   | U-Factor | `5.8` W/m²K (vidrio sencillo de 3 mm) |
   | Solar Heat Gain Coefficient (SHGC) | `0.86` (vidrio sencillo claro) |
   | Visible Transmittance | `0.90` |

   Para vidrio doble con cámara de aire: U ~3.0, SHGC ~0.75, VT ~0.81.

### Opción B — Glazing Window Material (capas)

Para casos avanzados (vidrio caracterizado en laboratorio). Requiere todos los parámetros: transmitancia perpendicular, reflectancia frontal/trasera, transmitancia visible, etc. Detalle en [[../concepts/Ventanas]].

> En el taller usar **Simple Glazing**. Si se está haciendo investigación de un material nuevo, Opción B.

### Material precargado

Open Studio trae un material `glazing 3mm` listo para usar — sirve como punto de partida para vidrio sencillo del curso.

## 6. Crear la Construction de ventana

1. Sub-pestaña **Constructions**.
2. Botón verde **+** → renombrar (ej. `vent_3mm`).
3. Arrastrar el material de vidrio (de **My Model**) al construction.

> Una construction de ventana suele tener una sola "capa" cuando se usa Simple Glazing. Con Glazing Window Material multi-capa, agregar todas las capas en orden ext→int.

## 7. Asignar la Construction a las ventanas

Hay dos vías:

### Vía A — Por Construction Set (recomendado)

1. Pestaña **Construction → Construction Sets**.
2. Abrir el Construction Set ya asignado a la edificación.
3. Sección **Sub Surface Construction → Exterior Window** → arrastrar `vent_3mm`.

Todas las ventanas exteriores del modelo heredan automáticamente la construction.

Procedimiento detallado en [[Configurar-Construction-Set]].

### Vía B — Por superficie individual

1. Pestaña **Spaces → Sub Surfaces**.
2. Para cada ventana, columna `Construction` → arrastrar `vent_3mm`.

Útil si distintas ventanas tienen distintos vidrios.

## 8. Verificar

### En el preview 3D

- Pestaña **Geometry → 3D View** → las ventanas deben verse **transparentes**.
- Render By → **Construction** → cada ventana debe tener color de su construction.

### En la pestaña Sub Surfaces

- Cada ventana tiene una columna `Construction` no vacía.
- La columna `Outside Boundary Condition` heredada del muro padre.

### Run y `.err`

- `Run`. Si dice `Severe: Construction <vent_X> missing material assignments`, la construction quedó sin material — volver al paso 6.
- Si dice `Severe: SubSurface ... has no construction`, la ventana no recibió construction — verificar el slot Sub Surface del Construction Set o asignar localmente.

## Trampas comunes

| Síntoma | Causa |
|---------|-------|
| Ventana 100% del muro | No se permite — bajar a 95-98%. Caso típico: cafetería abierta — modelar muro virtual + ventana del 95% |
| Ventana se sale del muro padre | El click cayó cerca del borde — verificar que el centro + dimensiones quepan dentro del muro |
| Severe `outside layer not found` para construction de ventana | Material de vidrio no llegó al construction — re-arrastrar |
| WWR no coincide con lo configurado | El componente tiene Height+Width definidos también — uno u otro, no ambos |

## Marcos (no se modelan en el taller)

El componente window de FloorspaceJS no expone marcos. Para modelarlos:

1. Exportar IDF.
2. Editar manualmente agregando un objeto `WindowProperty:FrameAndDivider`.

En el taller los marcos **se ignoran** — la ventana entera se cuenta como cristal. Caricatura aceptable para evaluar estrategias bioclimáticas. Detalle en [[../concepts/Ventanas]].

## Para agregar aleros

Una vez que la ventana funciona, ver [[Agregar-Aleros-OpenStudio]].

## Clases relacionadas

- [[../classes/006-DosZonasTermicasVentanasAleros]] — demo del flujo
