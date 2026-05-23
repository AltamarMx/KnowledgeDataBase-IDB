---
title: Importar Plano como Imagen en FloorspaceJS
type: procedimiento
tags: [procedimiento, openstudio, floorspace, plano, imagen, grid, proyecto-final]
aliases: [importar plano floorspace, plano imagen, calibrar grid, dibujar sobre plano]
clases: [014]
updated: 2026-05-22
---

# Importar Plano como Imagen en FloorspaceJS

Procedimiento para reconstruir rápido la geometría de una casa real usando el plano como imagen de fondo en [[../tools/Open-Studio]] FloorspaceJS. Demostrado en [[../classes/014-InfiltracionFloorspaceWindowLBNL]] para la Casa 3 del proyecto final.

## Cuándo usarlo

- Tienes un plano arquitectónico (PDF, JPG, screenshot).
- Conoces **al menos una distancia** del plano (un lado, una pared, una ventana).
- No necesitas precisión milimétrica — basta con la **volumetría correcta**.

> "Si tengo eso, tengo el plano y tengo las medidas, ya lo puedo dibujar. Por supuesto que me puedo fallar por un poquito, pero eso no importa tanto. No quiero que nos entretengamos en los centímetros."

## El truco general

1. **Cargar la imagen del plano** como capa de fondo.
2. **Calibrar el grid** con una distancia conocida (cada cuadrito = una unidad real).
3. **Dibujar zonas térmicas encima** — el dibujo se ajusta al grid pero la geometría es la del plano.

Es análogo a una **calibración fotogramétrica**: si conoces una distancia en la imagen, todas las demás se infieren proporcionalmente (asumiendo que el plano no está deformado, lo cual se cumple para PDFs arquitectónicos vistos en planta).

## Pasos

### 1 — Obtener la imagen del plano

Opciones:

- **Screenshot del PDF** (`Shift+Cmd+4` en Mac, `Win+Shift+S` en Windows).
- Foto del plano impreso si es la única fuente.
- Imagen exportada del software original (Revit, AutoCAD).

Recomendación: **recortar al área útil** (la planta, sin márgenes ni leyendas) para que el grid se aplique solo a lo relevante.

### 2 — Crear nuevo Floor Plan en Open Studio

`Editor (pestaña) → Open Floor Space → New`:

1. Aparece la interfaz de FloorspaceJS.
2. En la barra de herramientas superior, ícono **`Image`**.

### 3 — Cargar la imagen

1. Click en el ícono Image.
2. `Browse` → seleccionar la imagen.
3. La imagen aparece como capa de fondo, escalada arbitrariamente.

⚠️ A veces falla — síntomas:

- La imagen aparece pero **no se puede mover ni escalar**.
- Open Studio se congela al cambiar al ícono Image.

Solución: cerrar Open Studio y volver a abrirlo. Si persiste, abrir en modo limpio sin OSM previo.

> "Yo creo que esta versión tiene un bug y no me deja. Vamos a fallar estrepitosamente."

### 4 — Calibrar el grid con un lado conocido

Identifica una distancia conocida en el plano. Para la Casa 3 del proyecto final: el ancho de la fachada es **7 m**.

1. En `Grid Settings`, poner el espaciado del grid en la **unidad real conocida** (ej. **`Spacing = 1 m`** si la fachada mide 7 m).
2. Reposicionar la imagen para que el lado conocido cubra **el número correcto de cuadritos** (7 cuadritos para 7 m).
3. Alinear la esquina inferior izquierda de la planta al origen `(0,0)` del grid — recomendación general de [[../procedures/Estructura-Proyecto-Simulacion]].

### 5 — Dibujar zonas térmicas

Con la imagen como referencia visual:

1. Herramienta **`Polygon`** o **`Rectangle`**.
2. Trazar cada zona térmica siguiendo los muros del plano.
3. Pequeñas decisiones (¿esta puerta dónde cae?, ¿este nicho se contabiliza?) **harán que cada equipo tenga una casa ligeramente diferente** — está bien.

### 6 — Nombrar cada espacio con sufijo `_E` o `_S`

> "El espacio y la zona térmica no se pueden llamar de la misma manera. Pónganle `_E` o `_S` de Space."

Ver [[../concepts/Espacio-vs-ZonaTermica]].

Renombrar inmediatamente en FloorspaceJS:

| Espacio (en FloorspaceJS) | ThermalZone (auto-generada) |
|---|---|
| `recamara_norte_E` | `recamara_norte` |
| `estancia_E` | `estancia` |
| `cocina_E` | `cocina` |

Sin acentos, eñes ni espacios — son fuente de bugs en E+.

### 7 — Subir a la segunda planta

Para la planta alta:

1. `Stories → New Story` o el botón `+` en la columna de pisos.
2. **Bug conocido**: la imagen del piso 1 **no persiste**. Hay que cargar la imagen de la planta alta como **nueva imagen** en el piso 2.
3. Repetir Pasos 3-6 para la planta alta.

Si la planta alta tiene **dobles altura** (huecos abiertos), dejar esa zona sin polígono — el algoritmo de E+ entiende el espacio vacío como exterior.

> "En el piso de arriba tiene un hueco, ¿por qué? Pues así diseñaron la casa, no sé."

### 8 — Preview en 3D

`Preview` (botón superior derecho de FloorspaceJS) → vista 3D del modelo. Verifica:

- Las dos plantas están alineadas verticalmente.
- Las alturas son razonables (~2.5-3 m por planta).
- No hay zonas térmicas duplicadas o solapadas.

### 9 — Asignar a ThermalZones y proceder

En Open Studio (no en FloorspaceJS):

1. `Thermal Zones → Auto-assign` o asignar manualmente cada Space a una ThermalZone.
2. Continuar con [[Crear-Primera-Simulacion-OpenStudio]] (Construction Set, materiales, etc.).

## Limitaciones de FloorspaceJS

> "FloorspaceJS es educativo porque es gratuito. Yo esperaría que en un análisis serio estén usando SketchUp o Revit."

Anti-patrones recurrentes:

- **Geometrías curvas** → no se pueden dibujar; aproximar con polígonos.
- **Puertas/ventanas interiores entre zonas** → buggy, mejor dejar las zonas como cubos cerrados (caso del proyecto final 2026-2).
- **Imagen no persiste entre plantas** → recargar.
- **Software se congela** ocasionalmente → reiniciar.

Para proyectos reales, usar:

- **SketchUp** + plugin OpenStudio (pago).
- **Revit** + exportador a gbXML (workflow profesional).
- **Edición directa del IDF** (avanzado).

## Cada equipo tendrá una casa diferente

> "Va a hacer que cada uno de ustedes tenga una casa ligeramente diferente. Cuando tengo ganas me pongo a revisar las geometrías para ver quién le copió a quién — cada vez tengo menos tiempo, pero la curiosidad no se me ha muerto."

Las pequeñas decisiones de zonificación son **inevitables** y **deseables** — refuerzan el aprendizaje (cada equipo se compromete con sus decisiones). El profesor las acepta mientras estén justificadas.

## Aplicación al proyecto final 2026-2

Para la **Casa 3** de Decide y Construye ([[../concepts/Caso-Base]]):

- Importar el PDF actualizado de Classroom.
- Una dimensión conocida que basta: el ancho de la fachada es **7 m**.
- Zonificación sugerida: ~7 zonas térmicas (3 abajo + 4 arriba, con la zona del hueco en planta alta como exterior).
- **Tener cuidado al subir** la imagen de la planta alta cuando se cambia de piso.

## Clases relacionadas

- [[../classes/014-InfiltracionFloorspaceWindowLBNL]] — demostración con la Casa 3
- [[../classes/003-MiPrimeraSimulacion]] — introducción a FloorspaceJS

## Ver también

- [[../tools/Open-Studio#floorspacejs]]
- [[../concepts/Espacio-vs-ZonaTermica]]
- [[../concepts/Limpiar-Geometria]]
- [[Crear-Primera-Simulacion-OpenStudio]]
- [[Estructura-Proyecto-Simulacion]]
