---
title: Configurar y asignar un Construction Set
type: procedimiento
tags: [procedimiento, openstudio, construction-set, asignacion-masiva]
aliases: [crear construction set, asignar construction set, default construction set]
clases: [004, 006]
updated: 2026-05-02
---

# Configurar y asignar un Construction Set

Procedimiento para crear un [[../concepts/Construction-Set]] en Open Studio y asignarlo a la edificación, evitando arrastrar constructions superficie por superficie.

## Pre-requisitos

Antes de configurar el Construction Set, hay que tener:

- **Materials** definidos (pestaña Materials → Materials).
- **Constructions** ensambladas — al menos una para muros, una para techos, una para pisos. Ver [[../concepts/Sistemas-Constructivos]].

## 1. Crear el Construction Set

1. Pestaña **Construction** (icono de muro, columna izquierda).
2. Sub-pestaña **Construction Sets**.
3. Botón verde **+** → se crea `Construction Set 1`.
4. Renombrar a algo descriptivo, ej. `casa_ladrillo_concreto` o `set_zona_climatica_2A_aislada`.

## 2. Llenar los slots

El Construction Set tiene secciones por **condición de frontera** (Exterior, Interior, Ground Contact, Adiabatic) y dentro de cada una, slots por **tipo de superficie** (Wall, Roof, Floor, Sub-surfaces).

Para cada slot que aplica al modelo:

1. Localizar la fila correspondiente.
2. Desde el panel **My Model** (derecha), filtrar por **Constructions**.
3. Arrastrar la construction adecuada a la celda del slot.

### Ejemplo: cubo con dos zonas + piso adiabático

| Sección → Slot | Construction a arrastrar |
|----------------|---------------------------|
| Exterior Surface → Wall | `Tabique_14cm` |
| Exterior Surface → Roof | `Concreto_15cm` |
| Interior Surface → Wall | `Tabique_14cm` (si los muros entre zonas son del mismo material) |
| Adiabatic → Floor | `Concreto_15cm` |

### Slots de sub-superficies (ventanas)

Cuando el modelo tiene ventanas, hay slots adicionales para sus constructions:

| Sección → Slot | Aplica a |
|----------------|----------|
| **Exterior Sub Surface → Fixed Window** | Ventanas exteriores fijas |
| **Exterior Sub Surface → Operable Window** | Ventanas exteriores operables |
| **Exterior Sub Surface → Door** | Puertas opacas |
| **Exterior Sub Surface → Glass Door** | Puertas con cristal |
| **Exterior Sub Surface → Skylight** | Lucernarios |
| **Interior Sub Surface → ...** | Análogo para sub-superficies entre zonas |

A estos slots se arrastra la construction de ventana creada con `WindowMaterial:SimpleGlazingSystem` o `WindowMaterial:Glazing`. Procedimiento de creación de ventana en [[Agregar-Ventanas-OpenStudio]].

Slots **vacíos** son OK siempre que **ninguna superficie del modelo** caiga en ese match — si hay alguna, E+ devolverá un severe (ver [[Leer-Archivo-ERR]]).

## 3. Asignar el Construction Set a la edificación

1. Pestaña **Facility** (icono de edificio, columna izquierda).
2. Sección **Default Construction Set**.
3. Desde **My Model** arrastrar el Construction Set creado a esa celda.

## 4. Verificar

### En la pestaña Spaces → Surfaces

Las superficies que reciben construction desde el Construction Set la mostrarán en **verde** (default heredado). Si una superficie está vacía o sigue con construction local (negra), revisar:

- ¿Cae en algún slot del set?
- ¿La columna `Outside Boundary Condition` y la columna `Surface Type` corresponden a un slot ocupado?

### En el preview 3D

Pestaña **Geometry → 3D View → Render By → Construction**. Cada construction se colorea distinto. Ver que cada superficie tenga el color esperado.

Adicionalmente, **Render By → Boundary Conditions** ayuda a verificar que las condiciones de frontera (que disparan el match del set) son las correctas — ver [[../concepts/Condiciones-de-Frontera]].

## 5. Sobreescritura local (cuando sea necesario)

Para una superficie específica que necesita una construction distinta a la del set:

1. Pestaña **Spaces → Surfaces** → fila de la superficie.
2. Columna **Construction** → arrastrar la construction local.
3. La celda pasa de verde a negra (ya no responde al set).

Para revertir y volver al default del set, **borrar** el contenido de la celda (queda en verde otra vez).

## 6. Ground (caso especial)

Si el modelo tiene piso `Ground` (no adiabático), Open Studio asigna por default `ground_construction_default` que **no tiene** materiales. Resultado: severe al correr.

**Fix**: completar el slot **Ground Contact → Floor** del Construction Set con una construction válida (ej. `Concreto_25cm`), o cambiar la condición de frontera del piso a `Adiabatic`.

## Sub-pestañas relacionadas en Construction

- **Construction Sets** — los sets (este procedimiento).
- **Constructions** — las constructions individuales.
- **Materials** — los materiales (en su propia pestaña).

## Beneficios

- **Velocidad** — un edificio de 200 superficies queda asignado en un drag.
- **Reusabilidad** — el mismo set se puede aplicar a otros proyectos.
- **Estudios paramétricos** — cambiar el set asignado a la edificación cambia todo el modelo en un click. Ideal para comparar:
  - `set_no_aislado` vs `set_aislado_5cm` vs `set_aislado_10cm`.
  - `set_muros_pesados` vs `set_muros_ligeros`.

## Clases relacionadas

- [[../classes/004-InterpretandoMensajesConstructionSets]] — introducción y demo en vivo
- [[../classes/006-DosZonasTermicasVentanasAleros]] — uso del slot Sub Surface para ventanas
