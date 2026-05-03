---
title: Estructura del proyecto de simulación
type: procedimiento
tags: [procedimiento, openstudio, organizacion, archivos, narrativa-computacional]
aliases: [estructura proyecto, organizacion archivos, narrativa computacional]
clases: [003, 004, 007, 008]
updated: 2026-05-02
---

# Estructura del proyecto de simulación

Convenciones del taller para organizar archivos de cada simulación. Si se siguen, las tareas se entregan limpias y el profesor puede revisar cualquier estado intermedio.

## Carpeta por proyecto / tarea

**Cada simulación es una carpeta independiente**:

```
~/Escritorio/septimo_semestre/IDB/
├── tarea_01_primer_cubo/
├── tarea_02_dos_zonas_aleros/
├── proyecto_final/
└── ...
```

> No mezclar OSMs de tareas distintas en una misma carpeta — los folders auxiliares que crea Open Studio (ver más abajo) chocan.

### Reglas de naming

- **Sin acentos** (`á é í ó ú`)
- **Sin eñes** (`ñ`)
- **Sin espacios** — usar `_` (snake_case), `-` (kebab-case) o CamelCase
- **Sin caracteres especiales**

> "En algunas computadoras falla, en otras no. Es el error más común — la gente lo sabe y aún así lo hace."

## Sub-carpetas mínimas dentro del proyecto

```
tarea_01_primer_cubo/
├── OSM/        # Archivos .osm (Open Studio Model)
├── EPW/        # Archivos de clima .epw
└── notebooks/  # Libretas Jupyter de análisis
```

**Por qué separar:**

- El **EPW** es un input independiente del modelo. La misma geometría con otro EPW evalúa otro clima → conviene tener los EPW separados.
- Los **notebooks** consumen los outputs de la simulación; no deben estar mezclados con los archivos de modelo.
- Los **OSMs** crecen en versiones; aislarlos evita confusión.

## Versionado de OSMs (narrativa computacional)

**Numerar los OSMs y nombrarlos descriptivamente:**

```
OSM/
├── 001_volumetria.osm
├── 002_dosZonas.osm
├── 003_construcciones.osm
├── 004_diabaticoPiso.osm
├── 005_ventanas.osm
├── 006_caso_base.osm
├── 007_color_blanco.osm           # ← rama de variantes
├── 008_alero_horizontal.osm
└── ...
```

**Reglas:**

- Cada cambio sustantivo → nuevo número.
- **Nunca borrar** versiones anteriores. Si una versión rompe, regresar a la anterior es trivial.
- En proyectos de tesis han llegado a la versión 50.
- Cuando el modelo está **listo como caso base**, ramificar a versiones que prueban estrategias (color, orientación, sombras) — todas heredan del caso base.
- **Para duplicar el OSM**: usar `File → Save As` desde Open Studio. **No** copiar y pegar en el Explorador (Finder/File Explorer) — eso solo duplica el `.osm` sin el folder hermano y se pierden los measures. Detalle del riesgo en la sección "Folder hermano del OSM" más abajo.

> "Aquí es donde hay que practicar el desapego. Si una simulación no sale en una hora, regresa al paso anterior y vuelve a hacerlo."

Ver [[../concepts/Caricatura-Computacional]] sobre el principio.

## Folder hermano del OSM (cuidado)

Cuando se guarda un archivo `001_volumetria.osm`, Open Studio crea **junto a él un folder con el mismo nombre**:

```
OSM/
├── 001_volumetria.osm
└── 001_volumetria/    # ← folder hermano creado automáticamente
    ├── files/
    ├── measures/
    └── run/           # ← se borra y regenera en cada Run
```

**Reglas críticas:**

- **No guardar nada propio dentro de ese folder hermano.** En cada `Run` se borra y se regenera.
- **Nunca mover el OSM dentro de ese folder hermano.** Si lo haces y vuelves a abrir el OSM desde ahí, Open Studio crea un nuevo folder hermano dentro del primero, **se rompe el path al EPW** y los measures se pierden. (Caso real observado en la clase 004 con la tarea de un equipo.)
- Pero **sí contiene la configuración de measures** del OSM. Si se comparte solo el `.osm` sin el folder, los measures se pierden.
- **Para compartir o entregar la tarea: comprimir el proyecto completo** (folder padre, no solo el OSM) — ver siguiente sección.

## Cómo entregar / compartir

**Comprimir el proyecto completo en ZIP** desde el folder padre del proyecto:

- macOS: clic derecho sobre la carpeta → **Comprimir**.
- Windows: clic derecho → **Enviar a → Carpeta comprimida (ZIP)**.

**No usar TAR.** El profesor lo expresa explícitamente:

> "Por favor no usen Tar. ZIP está chido y está integrado en todos lados."

**No enviar solo el OSM.** Sin el folder hermano se pierden measures; sin el EPW no se puede correr; sin notebooks no se ve el análisis.

## Si se usa control de versiones (opcional, recomendado)

En lugar del versionado manual numerado, se puede usar **Git** (con remoto en GitHub):

- Cada cambio significativo es un commit.
- Las "ramas" de variantes se modelan como branches.
- Se evita el ZIP (compartir = compartir el repo).
- El profesor lo usa en sus proyectos pero **no es requisito** del curso.

## Estructura para el proyecto final — caso base + variantes

Para el [[../concepts/Estudio-Parametrico|estudio paramétrico]] del proyecto final (5 simulaciones):

```
proyecto_final/
├── OSM/
│   ├── 001_volumetria.osm
│   ├── 002_dosZonas.osm
│   ├── 003_constructions.osm
│   ├── 004_outputVars.osm
│   ├── 005_caso_base.osm           ← caso base congelado
│   ├── 006_estrategia_color.osm    ← variante 1
│   ├── 007_estrategia_aleros.osm   ← variante 2
│   ├── 008_estrategia_aislamiento.osm ← variante 3
│   └── 009_combinado.osm           ← combinada
├── EPW/
│   └── <ciudad>.epw
└── notebooks/
    ├── 001_EDA_simulacion.ipynb
    ├── 002_EDA_EPW.ipynb
    └── 003_comparacion_caso_base.ipynb
```

Cada variante se crea con `Save As` desde el caso base — nunca con copia/pega en el Explorador. Detalle en [[../concepts/Caso-Base]] y [[Comparar-Simulaciones-Python]].

## Estructura de libretas Jupyter para el proyecto final

Convención recomendada por el profesor (clase 008): **separar el análisis en libretas con responsabilidades claras**, terminando con una libreta que **unifica resultados**.

```
notebooks/
├── 001_EDA_simulacion.ipynb        ← verificar propiedades, sistemas constructivos, sanity
├── 002_EDA_EPW.ipynb               ← cargar EPW, T neutralidad mensual, zona de confort
├── 003_analisis_individual.ipynb   ← para cada caso: cargar SQL, calcular grados-hora
└── 004_unificacion_resultados.ipynb ← tabla comparativa, gráficas finales del reporte
```

Beneficios:

- **División del trabajo**: el equipo puede repartirse las libretas (alguien la simulación, alguien EDA, alguien análisis).
- **Reproducibilidad**: cada libreta corre con `Restart and Run All` independientemente.
- **Reuso**: la libreta `001_EDA_simulacion.ipynb` se aplica a las 5 simulaciones cambiando solo el path.

> "Mi carta santa es que todos sepan hacer todo. Pero sé que es complicado."

Detalle de la libreta `004` (unificación) en [[Comparar-Simulaciones-Python]] sección "Comparación cuantitativa".

## Clases relacionadas

- [[../classes/003-MiPrimeraSimulacion]] — primera tarea que requiere esta estructura
- [[../classes/004-InterpretandoMensajesConstructionSets]] — caso real del OSM movido al folder hermano y consecuencias
- [[../classes/007-CasoBaseAleros]] — estructura del proyecto final con 5 simulaciones; regla de Save As vs copia en Explorador
- [[../classes/008-ShadingVentanas]] — estructura de libretas Jupyter para el proyecto final con libreta unificadora
