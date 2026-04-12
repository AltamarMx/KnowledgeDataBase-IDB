# Open Studio

Programa gratuito y de código abierto que proporciona una interfaz gráfica para realizar simulaciones energéticas de edificaciones. Es la herramienta principal del taller.

**Sitio web:** openstudio.net
**Descargas:** OpenStudio Coalition > GitHub releases

## Características

- Interfaz gráfica con drag-and-drop para configurar simulaciones
- Editor de geometrías simple (dibujo en planta que genera volumetría 3D)
- Previsualización 3D de la geometría (algo que Energy Plus no ofrece)
- Visualización de condiciones de frontera
- Creación de schedules (horarios de ocupación, cargas térmicas)
- Enfocado en cumplimiento normativo (contexto estadounidense)
- Genera archivos **OSM** (Open Studio Model)

## Relación con Energy Plus

- Open Studio es una **interfaz** de Energy Plus (y Radiance)
- Al instalar Open Studio, se instala automáticamente una versión de Energy Plus
- No tiene todas las capacidades de Energy Plus (sería demasiados botones/menús)
- Cuando se necesita algo que Open Studio no puede hacer, se exporta el IDF y se trabaja directo en Energy Plus

## Versión del curso

- **Open Studio 1.11.0** (release candidate)
- Versiones superiores pueden abrir archivos de versiones anteriores, pero no al revés
- Todos deben usar la misma versión para compatibilidad

## Measures

Scripts en Ruby que modifican el modelo o los resultados en dos puntos de inserción:
1. **Open Studio Measures** — antes de convertir OSM → IDF
2. **Energy Plus Measures** — después de generar el IDF, antes de correr EnergyPlus
3. **Reporting Measures** — post-procesamiento de resultados

**Fuente:** BCL (Building Component Library) — accesible desde la esquina inferior derecha de Open Studio ("Find Measures on BCL").

**Measures principales del curso:**
- `Add Output Variable` (EnergyPlus) — solicita variables de salida específicas
- `Create CSV Output` (Reporting/QAQC) — exporta resultados del SQL a CSV
- `OpenStudio Results` (Reporting) — reporte HTML estándar (viene por defecto)

**Limitación:** Open Studio solo expone un subconjunto de variables de salida de EnergyPlus en su interfaz. Para solicitar variables adicionales (ej. coeficientes convectivos, radiación incidente por superficie), se necesita el measure `Add Output Variable`.

## Limitaciones

- No soporta ventilación natural (no está como opción desde la interfaz)
- Geometrías limitadas al editor integrado (para geometrías complejas se necesita SketchUp o Rhino)
- Evoluciona más lento que Energy Plus en features experimentales
- No expone todas las variables de salida de EnergyPlus — requiere measures para las faltantes

## Construction Sets

Agrupación que asigna sistemas constructivos a todas las superficies según tipo y condición de frontera. Se crean en la pestaña Construction y se asignan desde **Facility**. Superficies asignadas por el set aparecen en verde (default). Ver [[Sistemas-Constructivos]].

## Show Simulation

Botón que abre la carpeta de resultados donde están el archivo `.err`, el IDF traducido, el SQL y otros archivos de salida.

## Estructura de carpetas

- Open Studio crea una carpeta junto al .osm con archivos de simulación
- **No mover** el .osm dentro de esa carpeta (genera carpetas anidadas)
- **No guardar** archivos propios ahí (se borran al re-correr)

## Bug conocido: FloorSpaceJS pierde condiciones de frontera

Al hacer cambios geométricos (agregar/quitar overhangs, modificar componentes), FloorSpaceJS regenera la geometría y puede perder:
- Condiciones de frontera del piso (Ground → Adiabatic)
- Nombres personalizados de superficies

**Workaround:** verificar condiciones de frontera y nombres después de cada cambio geométrico. Asignar Construction Set antes de cambiar la condición para que no se pierda el sistema constructivo.

## Save As para variantes

- **Nunca** copiar la carpeta del .osm desde el explorador — se pierde la carpeta de Measures
- **Siempre** usar File → Save As → nuevo nombre dentro de Open Studio

## Aparece en

- [[001-IntroduccionTallerIDB]] — Presentación e instrucciones de instalación
- [[004-InterpretandoMensajesConstructionSets]] — Construction Sets, Show Simulation, estructura de carpetas, Measures (OSM e IDF)
- [[005-AnalisisSimulacionesPython]] — Measures, BCL, configuración de variables de salida
- [[006-DosZonasTermicasVentanasAleros]] — Ventanas (Component), overhangs/fins, materiales de vidrio, flujo incremental
- [[007-CasoBaseAleros]] — Bug de FloorSpaceJS, Save As para variantes, nombrado de superficies
- [[008-ShadingVentanas]] — Verificación de shading surfaces en IDF, Sunlit Fraction como variable de salida
