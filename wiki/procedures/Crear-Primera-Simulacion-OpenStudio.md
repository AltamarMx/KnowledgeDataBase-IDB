---
title: Crear primera simulación en Open Studio (cubo + EPW)
type: procedimiento
tags: [procedimiento, openstudio, primera-simulacion, geometria, materiales, energyplus]
aliases: [primera simulacion, cubo openstudio, tarea clase 003]
clases: [003]
updated: 2026-05-02
---

# Crear primera simulación en Open Studio (cubo + EPW)

Procedimiento end-to-end para crear el primer modelo en Open Studio: una geometría simple, un EPW asignado, materiales y construction básicos, condiciones de frontera correctas, y `Run`. Es la **tarea de la semana** que cierra la clase 003 — un cubo de 3×3×3 m con un solo material.

> Antes de empezar: asegúrate de tener Open Studio instalado ([[Instalar-Open-Studio]]) y la **misma versión que el resto del grupo**.

## 0. Preparar la estructura del proyecto

Crear el proyecto con la convención de carpetas:

```
~/Escritorio/septimo_semestre/IDB/tarea_01_primer_cubo/
├── OSM/
├── EPW/
└── notebooks/
```

Detalle y convención de naming en [[Estructura-Proyecto-Simulacion]] (sin acentos, sin eñes, sin espacios).

## 1. Crear la geometría en FloorspaceJS

1. Abrir Open Studio Application → **File → New**.
2. En la columna izquierda elegir la pestaña **Geometry** (icono de cubo).
3. Pestaña **Editor** (dentro de Geometry) → botón **New Floorplan**.
4. (Opcional, divertido) **Search for location**: buscar la ciudad y dibujar sobre el mapa de OpenStreetMap como referencia. Para esta tarea no es necesario.
5. Configurar el **grid**:
   - Esquina superior derecha: campo de tamaño de grid.
   - Para un cubo de 3×3 m: poner **grid de 0.5 m** o **1 m**.
6. Herramientas (esquina superior izquierda):
   - **Rectangle** → dibujar rectángulo de 3×3 m con dos clics.
   - **Polygon** → para formas no rectangulares.
   - **Eraser** → para borrar.
7. Para definir el **alto del piso** (story height):
   - En la barra izquierda buscar el panel **Stories**.
   - **Floor to Ceiling Height** = `3` (default es 2.43 m).
8. **Renombrar el espacio** (en el panel Spaces): de `Space 1` a `S:Cubo` u otro nombre claro. Convención: prefijo `S:` para distinguir espacio de zona térmica — ver [[../concepts/Espacio-vs-ZonaTermica]].

> En FloorspaceJS solo se dibujan **plantas** — la altura se define por separado. Los espacios se extruyen automáticamente.

## 2. Merge al modelo OSM

1. Botón **Merge with Current OSM** (esquina superior derecha del editor).
2. Volver a la pestaña **3D View** → click en **Refresh**. Si pregunta `Choose ignore to...` → **Ignore**.
3. Verificar que se ve un cubo 3D.

## 3. Verificar tipos de superficie y condiciones de frontera

1. En el preview 3D, panel **Render By**:
   - **Surface Type** → 4 muros amarillos, 1 techo rojo, 1 piso gris. Si algo está mal pintado, hay un problema de orientación de polígonos (ver [[../concepts/Tipos-Superficie]]).
   - **Boundary Conditions** → 4 muros + techo en **azul** (Outdoor); piso en **café/marrón** (Ground).
2. **Forzar piso adiabático**:
   - Pestaña **Spaces** → sub-pestaña **Surfaces**.
   - Localizar la fila del piso (Surface Type = `Floor`).
   - Columna **Outside Boundary Condition** → cambiar de `Ground` a `Adiabatic`.
   - El color en el preview cambia a **rojo** (Adiabatic).

Detalle de los 4 colores en [[../concepts/Condiciones-de-Frontera]].

## 4. Guardar versión 001

1. **File → Save As** → al folder `OSM/` del proyecto.
2. Nombre: `001_volumetria.osm`.

> **No usar Save** (sobrescribe). Cada cambio sustantivo → nuevo número de versión. Ver [[Estructura-Proyecto-Simulacion]].

## 5. Descargar y asignar el EPW

Procedimiento completo en [[Descargar-EPW-OneBuilding]]. Resumen:

1. climate.onebuilding.org → buscar la ciudad → descargar `.zip`.
2. Extraer el `.epw` → mover a `EPW/` del proyecto.
3. En Open Studio: pestaña **Site** → **Weather File** → **Set Weather File** → seleccionar el `.epw`.
4. Verificar latitud, longitud, time zone.

Guardar como `002_conEPW.osm`.

## 6. Crear la zona térmica

1. Pestaña **Thermal Zones** (columna izquierda).
2. Botón verde **+** → se crea `Thermal Zone 1`.
3. Renombrarla a un nombre descriptivo (ej. `Cubo`). Sin espacios, sin acentos.
4. Volver a **Spaces** → en la fila del espacio, columna **Thermal Zone** → arrastrar la zona térmica `Cubo` desde el panel **My Model** (derecha) hasta esa celda.

Detalle de la distinción en [[../concepts/Espacio-vs-ZonaTermica]].

Guardar como `003_conZonaTermica.osm`.

## 7. Definir el material

1. Pestaña **Materials** (columna izquierda — icono de ladrillos).
2. Sub-pestaña **Materials** (no `No Mass Materials` — esos no respetan masa térmica).
3. Botón verde **+** → nuevo material.
4. Renombrar: ej. `Concreto_alta_densidad_15cm_a07`.
5. Llenar las propiedades:

   | Campo | Valor (concreto alta densidad, 15 cm) |
   |-------|----------------------------------------|
   | Roughness | `MediumRough` |
   | Thickness (m) | `0.15` |
   | Conductivity (W/m·K) | `2.0` |
   | Density (kg/m³) | `2500` |
   | Specific Heat (J/kg·K) | `1000` |
   | Thermal Absorptance | `0.9` (emisividad) |
   | Solar Absorptance | `0.7` |
   | Visible Absorptance | `0.7` |

> Para la **tarea** real (clase 003 → 004) el profesor pide tabique en muros y concreto en piso/techo. Aquí va un solo material para el ejemplo más simple. Ajusta según las indicaciones del profesor.

> Los campos en **verde** son valores default; cuando los modificas pierden el color verde.

## 8. Definir la construction

1. Pestaña **Constructions** (sub-pestaña dentro de Materials).
2. Botón verde **+** → nueva construction.
3. Renombrar: ej. `Construccion_concreto15`.
4. **Arrastrar el material** desde **My Model** (derecha) al panel central — **de exterior a interior**.
   - Para una construction de una sola capa solo se arrastra el material una vez.
   - Para varias capas: orden estricto exterior → interior (ver [[../concepts/Sistemas-Constructivos]]).

## 9. Asignar la construction a las superficies

1. Pestaña **Spaces** → sub-pestaña **Surfaces**.
2. En cada fila (6 superficies del cubo), columna **Construction** → arrastrar `Construccion_concreto15` desde **My Model**.

> Esto es repetitivo. Hay maneras más rápidas (Default Construction Sets) que se ven en clases siguientes.

Guardar como `004_conMateriales.osm`.

## 10. Verificación final antes del Run

Antes de correr, confirmar:

- [ ] EPW asignado (Site → Weather File con datos)
- [ ] Una zona térmica creada y asignada al espacio
- [ ] Las 6 superficies tienen construction asignada (no celda vacía)
- [ ] Piso en Adiabatic; muros y techo en Outdoors
- [ ] No hay nombres con acentos / eñes / espacios

## 11. Run

1. Pestaña **Run Simulation** (icono de play, columna izquierda).
2. Botón **Run**.
3. La simulación corre — barra de progreso. Para un cubo simple, < 1 minuto.
4. Cuando llegue al **100%** y diga `Complete`, listo.
5. Si hay errores, revisar la pestaña **Output** y el archivo `eplusout.err` (ver clase 004).

## 12. Ver resultados (vista preliminar)

- Pestaña **Results Summary** → tablas de resumen del HTML que genera E+.
- Para análisis serio: pasar al notebook → ver flujo en clases 005+.

## Atajo: Construction Sets

Asignar la construction superficie por superficie es práctico solo para 6 superficies (un cubo). En cuanto el modelo crece, conviene definir un **Construction Set** que asigna constructions automáticamente según tipo+condición de frontera. Procedimiento en [[Configurar-Construction-Set]]. Concepto en [[../concepts/Construction-Set]].

## Pedir variables al output

Antes de correr (o tras un primer Run para tener el RDD), conviene configurar measures de output: `Add Output Variable` (uno por variable, ej. T zona, T exterior, radiación) + `Create CSV Output`. Procedimiento en [[Solicitar-Output-Variables-Measures]]. Sin esto la simulación corre pero no hay datos útiles para análisis.

## Agregar ventanas y aleros

Una vez que la simulación con muros opacos corre limpia, agregar ventanas y aleros como capas de complejidad incremental:

- [[Agregar-Ventanas-OpenStudio]] — sub-superficies de ventana con material y construction.
- [[Agregar-Aleros-OpenStudio]] — Overhang/Fin Projection Factors y workaround para extender el alero.

## Si la simulación falla

Procedimiento de debugging end-to-end en [[Debuggear-Simulacion-OpenStudio]]. Lectura del archivo `.err` paso a paso en [[Leer-Archivo-ERR]].

## Tarea de la semana

> **Cubo de 3×3×3 m** con tabique en muros y concreto en piso y techo (es decir, **dos sistemas constructivos** distintos pero del mismo orden), un EPW de la ciudad escogida, piso adiabático, sin ventanas. Entrega el proyecto completo en ZIP — ver [[Estructura-Proyecto-Simulacion]].

## Clases relacionadas

- [[../classes/003-MiPrimeraSimulacion]] — clase donde se demuestra el procedimiento
- [[../classes/001-IntroduccionTallerIDB]] — instalación de Open Studio
- [[../classes/002-ConceptosBasicosBalancesCalor]] — los conceptos físicos detrás de cada paso
