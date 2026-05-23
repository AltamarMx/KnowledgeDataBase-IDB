---
title: Open Studio
type: herramienta
tags: [herramienta, gui, software-libre, openstudio]
aliases: [OpenStudio, Open-Studio]
version_curso: 1.11.0-rc
clases: [001, 003, 004, 005, 006]
updated: 2026-05-02
---

# Open Studio

## Qué es

Interfaz gráfica **libre** para [[EnergyPlus]] (y para Radiance, aunque en este curso no se usa esa parte). Permite:

- Editar geometrías simples (envolvente, ventanas, vecinos)
- Definir materiales y sistemas constructivos visualmente
- Asignar **construction sets** para acelerar la configuración
- Crear schedules (horarios de ocupación, cargas térmicas)
- Visualizar la geometría 3D (algo que Energy Plus directo no permite)
- Ejecutar la simulación con Energy Plus integrado

Sitio: openstudio.net (Open Studio Coalition).

## Versión usada en el curso

**Open Studio Application 1.11.0** (release candidate al momento de la clase 001).

> **Acuerdo del grupo:** todos usan la misma versión. Razón: una versión más nueva puede abrir archivos de versiones anteriores, pero no al revés. Si algún estudiante usa una versión más nueva, el profesor no podrá abrir su archivo.

Cada versión de Open Studio viene atada a una versión específica de Energy Plus y un SDK. Esto importa cuando se hacen simulaciones colaborativas o se reutilizan archivos.

## Instalación

Ver [[../procedures/Instalar-Open-Studio]].

**No instalar:**
- El **SDK** — se instala a nivel de terminal, no provee GUI.
- El **plugin** (para SketchUp) — no se usa en el curso.

**Sí instalar:** la **Open Studio Application**.

Open Studio trae **Energy Plus integrado** — no es necesario instalar Energy Plus por separado.

## Archivos

- **OSM (Open Studio Model)** — formato propio, **texto plano**. Es una reescritura del IDF de Energy Plus: mismos objetos, sintaxis distinta.
- Puede **importar IDF** (input file de Energy Plus) y **exportar IDF** (cuando se necesita pasar a Energy Plus directo para usar features que la GUI no expone).

### Hashes en los nombres de objetos

Cada objeto que crea Open Studio recibe un **hash** (cadena alfanumérica única) como nombre interno. **El hash no se puede cambiar al renombrar un objeto** desde la GUI — el hash sigue ahí en el OSM (texto plano).

> El profesor lo usa como detector de plagio: si dos tareas tienen los mismos hashes, una se copió de la otra. Solo cambiar nombres no engaña al hash.

### Folder hermano del OSM

Al guardar un `.osm`, Open Studio crea **junto a él un folder con el mismo nombre** (ej. `001_volumetria.osm` + `001_volumetria/`). Ese folder:

- Se **borra y regenera en cada Run**: nunca guardar archivos propios ahí.
- Contiene la **configuración de measures** del OSM. Compartir solo el `.osm` sin el folder pierde los measures.
- **Para entregar tareas: ZIP del proyecto completo**, no solo el OSM. Ver [[../procedures/Estructura-Proyecto-Simulacion]].

## Editor de geometrías

### FloorspaceJS (integrado, gratis)

Editor 2D escrito en JavaScript que viene con Open Studio. Workflow:

1. **Geometry → New Floorplan** abre el editor.
2. (Opcional) **Search for location** integra OpenStreetMap → buscar la ciudad y dibujar la planta sobre el mapa como referencia.
3. **Grid configurable** (esquina superior derecha) — típicamente 0.5 m o 1 m.
4. Herramientas: **Rectangle**, **Polygon**, **Eraser**.
5. Las plantas se **extruyen automáticamente** según la altura definida en `Stories → Floor to Ceiling Height` (default 2.43 m).
6. Cada espacio se renombra desde el panel Spaces (convención del profesor: prefijo `S:` para distinguir de zonas térmicas — ver [[../concepts/Espacio-vs-ZonaTermica]]).
7. **Merge with Current OSM** → la geometría pasa al modelo.
8. Pestaña **3D View → Refresh** para verla en 3D.

FloorspaceJS guarda en formato **JSON** (sub-folder del OSM). Existe también una versión web pero no aporta sobre la integrada.

> En FloorspaceJS solo se dibuja **planta**. La altura no se controla con el ratón — se define numéricamente en Stories.

### Trampa típica: paredes "casi pegadas"

Si dos espacios se dibujan **separados por 1 cm o 10 cm**, Open Studio **no los conecta** — los muros quedan como Outdoor (azul), no Surface (verde). E+ resolverá una transferencia de calor incorrecta. **Las paredes deben tocarse físicamente** (línea punteada al unirlas) para que Open Studio convierta la condición a `Surface` automáticamente.

### Render By (preview 3D)

Selector clave en el preview 3D para verificar el modelo:

- **Surface Type** — colorea por tipo: muros amarillos, techos rojos, pisos grises. Ver [[../concepts/Tipos-Superficie]].
- **Boundary Conditions** — colorea por condición: Outdoor azul, Surface verde, Ground café, Adiabatic rojo. Ver [[../concepts/Condiciones-de-Frontera]].

### Editores alternativos

| Editor | Estado | Notas |
|--------|--------|-------|
| **FloorspaceJS** (integrado, gratis) | Lo que se usa en el curso | Suficiente para cubos y casos sencillos |
| **SketchUp** (con plugin OS) | Excelente; SketchUp ya es de paga |
| **Rhino** | Profesional, soportado |
| **Blender** | Soportado | Open source |
| **Design Builder** (programa aparte) | Paga; trae editor propio | El más cómodo, pero esconde fundamentos |

**No** soporta geometrías complejas (techos a doble agua, volúmenes irregulares) sin un editor externo.

### Components — ventanas y aleros en FloorspaceJS

El editor permite agregar **ventanas** como componentes de librería:

1. **Components** (icono en la columna izquierda) → **Window Definitions**.
2. Crear o seleccionar un componente con sus parámetros (WWR o Height + Width + Sill Height).
3. Click en los muros para colocar la ventana.

Los **aleros (Overhang)** y **parteluces (Fin)** se generan junto con la ventana — el componente acepta un `Overhang Projection Factor` y `Fin Projection Factor`.

> Limitación crítica: el alero generado tiene **el mismo ancho que la ventana**. Para extenderlo lateralmente hay que editar el OSM directamente — ver [[../procedures/Agregar-Aleros-OpenStudio]].

Detalle de ventanas en [[../concepts/Ventanas]] y de sombras en [[../concepts/Superficies-de-Sombramiento]].

### Limpieza automática de geometría

Cuando dos espacios se unen físicamente, FloorspaceJS **corta automáticamente** las superficies que se traslapan (necesario para que la condición de frontera Surface se aplique). Funciona bien en geometrías simples; en casos complejos puede requerir intervención manual — ver [[../concepts/Limpiar-Geometria]].

## Pestañas principales del Open Studio Application

| Pestaña | Para qué |
|---------|----------|
| **Site** | Cargar el EPW (`Set Weather File`); ver lat/lon/timezone |
| **Schedules** | Horarios (no se usan en el curso) |
| **Construction** | Definir Materials, Constructions, **Construction Sets** ([[../concepts/Construction-Set]]) |
| **Loads** | Cargas internas (no se usan en el curso, salvo **infiltración** desde clase 014) |
| **Space Types** | Tipos de espacio reusables — **se usan en el curso desde clase 014** para asignar infiltración. Ver [[#space-types]] abajo. |
| **Facility** | Asignar Construction Set y Schedule Set por default a la edificación; rotar la edificación respecto al norte (`North Axis`) |
| **Geometry → Editor** | FloorspaceJS para dibujar plantas |
| **Geometry → 3D View** | Preview 3D con Render By |
| **Spaces** | Espacios y sus superficies; asignar zona térmica y constructions; columnas Sun/Wind Exposure |
| **Thermal Zones** | Crear y nombrar zonas térmicas |
| **HVAC** | Sistemas de aire acondicionado (no se usan en el curso) |
| **Output Variables** | Qué reportar al simular |
| **Measures** | Cargar/configurar measures ([[../concepts/Measures]]) |
| **Run Simulation** | Botón Run; **Show Simulation** abre el folder de outputs |
| **Results Summary** | Vista preliminar del HTML de resultados |

## Flujo interno al dar Run

Cuando se da `Run`, Open Studio ejecuta:

```
OSM → [OSM Measures] → OSM modificado → Traductor → IDF → [IDF Measures] → IDF modificado → Energy Plus → SQL + CSV + HTML + ERR
```

Hay **dos puntos** donde el usuario puede inyectar measures (ver [[../concepts/Measures]]). E+ produce salidas en varios formatos en paralelo (ver [[../concepts/Salida-SQL-EnergyPlus]]).

## Show Simulation

Botón en la pestaña Run Simulation. Abre el folder hermano del OSM en el explorador del sistema. Útil para:

- Acceder al **`eplusout.err`** para debuggear ([[../procedures/Leer-Archivo-ERR]]).
- Acceder al **`eplusout.rdd`** para descubrir variables disponibles ([[../concepts/RDD-Variables-Disponibles]]).
- Acceder al **`eplusout.sql`** para análisis con Python.
- Inspeccionar el **`in.idf`** que Open Studio le pasó a E+.

> Si no hay EPW asignado, `Show Simulation` puede fallar — primero asignar EPW y correr.

## Pedir variables al output

Hay dos vías:

| Vía | Pros | Contras |
|-----|------|---------|
| Pestaña **Output Variables** | GUI directa | Subset limitado de variables; no incluye `Surface Outside Face Incident Solar Radiation`, etc. |
| **Reporting Measures** del BCL (`Add Output Variable` + `Create CSV Output`) | Acceso a cualquier variable del RDD | Requiere descargar measures y configurar uno por variable |

> El curso recomienda **Reporting Measures** y dejar la pestaña Output Variables en defaults — para evitar mezclar las dos vías. Procedimiento en [[../procedures/Solicitar-Output-Variables-Measures]].

## BCL — Building Component Library

Repositorio público de measures, alojado en NREL. Acceso desde Open Studio:

1. Pestaña **Measures** → botón inferior derecho **Find Measures on BCL**.
2. Categorías: Envelope, Electric Lighting, HVAC, Reporting, etc.
3. Para Add Output Variable y Create CSV Output: **Reporting → QAQC**.
4. Marcar y descargar.

Categorías relevantes para el taller:

- **Reporting → QAQC** — measures de salida/verificación.
- **Envelope** — para futuros estudios paramétricos sobre la envolvente.

## Limitaciones (vs. Energy Plus directo)

Open Studio expone solo un subconjunto de los objetos de Energy Plus — los más usados en simulaciones para cumplir normativas estadounidenses. No expone, por ejemplo:

- **Ventilación natural** (cruzada) — no es prioridad en EE.UU.
- Features experimentales o de investigación que Energy Plus va incorporando.

**Workflow típico cuando algo no se puede en Open Studio:**

1. Avanzar todo lo posible en Open Studio.
2. Exportar IDF.
3. Continuar editando en el IDF Editor (interfaz tabular de Energy Plus) o directamente en el archivo de texto.

El mismo problema lo tiene Design Builder y otras GUIs — por eso conviene aprender Energy Plus aunque uno use una GUI cómoda.

## Por qué se eligió para el curso

- **Software libre** (filosofía + razón pragmática: el instituto no permite software pirata).
- **Suficiente** para el alcance del curso (geometrías simples, sin ventilación natural, sin cargas internas).
- Curva de aprendizaje más dura que GUIs de paga, pero **no oculta los fundamentos** — fuerza a entender condiciones de frontera y supuestos.
- La gente egresada del instituto trabaja en consultoría con Energy Plus, Design Builder o IES indistintamente — aprender los fundamentos en software libre transfiere bien al resto.

## Space Types

> Hasta clase 013 no se usaban; en clase 014 entran al curso como **contenedores de cargas** (específicamente: infiltración).

Un Space Type es una **plantilla reusable** que agrupa cargas (ocupación, iluminación, equipos, infiltración) con sus schedules. Se asigna a varios Spaces de un golpe → los Spaces heredan las cargas del Space Type.

### Workflow

1. `Space Types → New` → nombrar (e.g. `cuarto_ventilado`).
2. Dentro del Space Type → pestaña `Loads` → panel `Library`.
3. **Arrastrar** una carga existente (no permite crear vacía) — por ejemplo `Space Infiltration Design Flow Rate`.
4. Editar la carga (método, valores, coeficientes).
5. Asignar **Schedule** a la carga.
6. En `Spaces` → arrastrar el Space Type a la columna `Space Type` de los Spaces deseados.

### Anti-patrón confesional

> "Open Studio no me deja crear uno nuevo desde cero. Tengo que agarrar algo que ya existe y modificarlo." — clase 014

Si intentas crear una carga vacía, no se puede — la GUI obliga a arrastrar desde la Library. No es bug, es diseño.

Detalle en [[../procedures/Agregar-Infiltracion-OpenStudio]].

## Classic CLI — log de E+ minimalista

> "Si le activo `Classic` (Classic Command Line Interface), me da una versión minimalista — sin los plugins de Ruby y todas esas cosas." — clase 014

`Show Simulation → Classic CLI` produce un log de E+ más legible (sin el envoltorio Ruby de Open Studio). Útil para debugging fino — los mensajes de E+ aparecen directos.

## Clases relacionadas

- [[../classes/001-IntroduccionTallerIDB]] — introducción y tarea de instalación
- [[../classes/003-MiPrimeraSimulacion]] — tour completo de FloorspaceJS, OSM, Materials/Constructions, Render By, Run
- [[../classes/004-InterpretandoMensajesConstructionSets]] — flujo OSM→IDF, Construction Sets, Show Simulation, lectura del `.err`
- [[../classes/005-AnalisisSimulacionesPython]] — pedir variables con measures del BCL, leer el RDD, postprocesamiento con Python
- [[../classes/006-DosZonasTermicasVentanasAleros]] — components de ventanas y aleros, limitación del alero, edición manual del OSM
- [[../classes/014-InfiltracionFloorspaceWindowLBNL]] — Space Types como contenedores de cargas (infiltración); Classic CLI; plano como imagen en FloorspaceJS

## Procedimientos

- [[../procedures/Instalar-Open-Studio]]
- [[../procedures/Crear-Primera-Simulacion-OpenStudio]]
- [[../procedures/Estructura-Proyecto-Simulacion]]
- [[../procedures/Descargar-EPW-OneBuilding]]
- [[../procedures/Configurar-Construction-Set]]
- [[../procedures/Leer-Archivo-ERR]]
- [[../procedures/Debuggear-Simulacion-OpenStudio]]
- [[../procedures/Solicitar-Output-Variables-Measures]]
- [[../procedures/Agregar-Ventanas-OpenStudio]]
- [[../procedures/Agregar-Aleros-OpenStudio]]
