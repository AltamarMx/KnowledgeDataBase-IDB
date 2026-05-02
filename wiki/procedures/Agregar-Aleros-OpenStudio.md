---
title: Agregar aleros y parteluces en Open Studio
type: procedimiento
tags: [procedimiento, openstudio, aleros, overhangs, fins, parteluces, sombreamiento, hack]
aliases: [aleros openstudio, overhangs, parteluces, fins, projection factor, alero hack]
clases: [006, 007]
updated: 2026-05-02
---

# Agregar aleros y parteluces en Open Studio

Procedimiento para colocar [[../concepts/Superficies-de-Sombramiento|aleros (overhangs) y parteluces (fins)]] sobre ventanas en Open Studio, e incluye el **workaround crítico** para extender la longitud lateral del alero (limitación de la GUI).

## Pre-requisitos

- Modelo con ventanas ya colocadas. Ver [[Agregar-Ventanas-OpenStudio]].
- Simulación corriendo limpia antes de agregar aleros (facilita debug).

## 1. Configurar el componente window con Projection Factors

En el editor de FloorspaceJS:

1. **Components → Window Definitions** → seleccionar el componente que se usará.
2. En el panel de propiedades:

   | Parámetro | Significado |
   |-----------|-------------|
   | **Overhang Projection Factor** | Longitud horizontal del alero / altura de la ventana |
   | **Fin Projection Factor** | Longitud horizontal del parteluz / altura de la ventana |

3. Valores típicos:

   | Valor | Geometría |
   |-------|-----------|
   | `0` | Sin alero/parteluz |
   | `0.5` | Alero la mitad de la altura de la ventana |
   | `1.0` | Alero igual a la altura de la ventana |

> Cambiar estos valores **antes** de colocar las ventanas. Cambiarlos después no actualiza ventanas ya insertadas — habría que borrarlas y volver a colocar.

## 2. Insertar las ventanas

Como en [[Agregar-Ventanas-OpenStudio]] paso 3 — el alero/parteluz se genera **automáticamente** junto con cada ventana al colocarla.

## 3. Merge y verificar

1. **Merge with Current OSM**.
2. **3D View → Refresh**.
3. Verificar visualmente: encima de cada ventana hay una losa horizontal pequeña (alero); a los lados, paneles verticales (parteluces).

## 4. Limitación crítica — el alero tiene el mismo ancho que la ventana

> El alero generado por FloorspaceJS tiene **exactamente el ancho de la ventana**. No protege cuando el sol viene oblicuo.

Razón física: el sol bajo (mañana, tarde, fachadas E/O) proyecta sombras laterales que **caen fuera del ancho del alero**. Ver explicación detallada en [[../concepts/Superficies-de-Sombramiento]].

## 5. Workaround — extender el alero editando el OSM

### Opción A — Editar manualmente el OSM (texto plano)

El profesor demuestra esto en clase. El OSM es texto plano editable.

#### 1. Identificar la cara del alero

1. En **3D View**, click sobre el alero a modificar.
2. Panel inferior muestra el nombre del objeto: `Face N` (ej. `Face 18`) o un nombre custom si se renombró.
3. **Anotar el nombre**.

#### 2. Guardar el OSM con un número de versión nuevo

`File → Save As` → `005_aleroExtendido.osm`. **No sobreescribir** la versión que funciona — si rompemos algo, regresamos.

#### 3. Cerrar Open Studio

Open Studio bloquea el archivo si está abierto. Hay que cerrarlo o al menos cerrar el modelo.

#### 4. Abrir el OSM con un editor de texto

Notepad++, VS Code, Sublime Text. Cualquiera con búsqueda y soporte de archivos grandes (OSMs pueden ser de varios MB).

#### 5. Buscar la cara

`Ctrl+F` → `Face 18` (o el nombre anotado).

Las caras están definidas como objetos `OS:Surface` o `OS:ShadingSurface`:

```
OS:ShadingSurface,
  {f9a3b2c1-...},                  !- Handle
  Face 18,                         !- Name
  ...
  {x1, y1, z1},                    !- Vertex 1 X,Y,Z
  {x2, y2, z2},                    !- Vertex 2 X,Y,Z
  {x3, y3, z3},                    !- Vertex 3 X,Y,Z
  {x4, y4, z4};                    !- Vertex 4 X,Y,Z
```

#### 6. Modificar las coordenadas

Identificar qué eje quieres extender. Si `z` es vertical (altura) y `x`, `y` son horizontales en planta:

- Para extender **lateralmente** (en `x`): mover los vértices del lado izquierdo `−1` m y los del lado derecho `+1` m.
- **Mover de a 2 vértices** (los dos del mismo borde lateral). Si mueves uno solo, el polígono se deforma.

> El profesor sugiere ir conservador la primera vez (extender 1 m por lado), guardar, recargar, verificar visualmente. Iterar.

#### 7. Recargar en Open Studio

1. Abrir Open Studio → File → Open → seleccionar el OSM modificado.
2. **3D View → Refresh** → verificar visualmente que el alero se extendió.

#### 8. Re-correr y validar

`Run` → revisar `.err`. Geometría inválida (vértices coplanares mal alineados) generará severe — hay que ajustar.

### Opción B — Pedirle a una IA que aplique la transformación

El profesor menciona que ha usado **Claude / ChatGPT** para esto:

1. Copiar el bloque `OS:ShadingSurface` con sus 4 vértices.
2. Pedir a la IA: "Estos son los 4 vértices de un alero rectangular. Extiéndelo 1 metro a cada lado en el eje X. Devuélveme el bloque con las nuevas coordenadas."
3. Validar que los 4 vértices **siguen siendo coplanares** y forman un rectángulo (la IA a veces los ordena mal).
4. Reemplazar en el OSM.

Tasa de éxito según el profesor: ~50%. Cuando falla suele ser por orden de vértices o coplanaridad.

## 6. Diseño efectivo — recomendaciones

Para el proyecto final el profesor recomienda **alero horizontal + parteluz vertical** combinados:

| Orientación | Estrategia primaria |
|-------------|----------------------|
| **Sur** (hem. norte) | Alero horizontal + extensión lateral |
| **Norte** | Sombreamiento mínimo — sol siempre bajo |
| **Este / Oeste** | Parteluces verticales (sol bajo y oblicuo en mañanas/tardes) |

### Aleros equivalentes — celosías

Para diseños arquitectónicos avanzados, simular **celosías** como un alero único de gran tamaño preservando los ángulos críticos. Detalle en [[../concepts/Superficies-de-Sombramiento]] sección "Aleros equivalentes — celosías".

### Validación de la caricatura — anécdota Paloma

Una alumna del grupo IER (Paloma) comparó dos simulaciones de la misma protección compleja:

| Simulación | Geometría |
|------------|-----------|
| Detallada | Cada listón de la rejilla dibujado en SketchUp |
| Simplificada | Una superficie equivalente con **transmitancia agregada** |

**Diferencia: <2%** en radiación recibida en la superficie de medición.

> "Bueno, dijimos: entonces hacemos estas con toda la confianza."

Conclusión: usar **alero equivalente con transmitancia** en lugar de dibujar cada listón es válido para estudios paramétricos — el orden y magnitud del efecto se conservan. Justificación práctica para usar caricaturas en geometrías complejas. Ver [[../concepts/Caricatura-Computacional]].

## 7. Verificar el efecto del alero

Pedir como output variable la radiación incidente sobre la ventana:

```
Surface Outside Face Incident Solar Radiation Rate per Area  (sobre la ventana)
```

Comparar dos simulaciones:

- Sin alero (`PF = 0`).
- Con alero extendido (después del workaround).

La diferencia entre ambas series es la radiación que el alero está bloqueando. Útil para:

- **Cuantificar** la efectividad del alero.
- Reportar **% de reducción** en radiación incidente.
- Estudiar el efecto sobre **temperatura interior** y **grados-hora de disconfort**.

Procedimiento de análisis en [[Analizar-Resultados-Python]].

## Aleros que no son ventanas — vecinos, vegetación

Para sombras de **edificios vecinos** o **vegetación**, en lugar de un Projection Factor, se dibujan **superficies de sombramiento independientes**:

1. En FloorspaceJS, crear un **Shading** (no un Space).
2. Dibujar la geometría del vecino o del árbol como superficie opaca.
3. Asignarle una construction (define reflectancia/absortancia).

Mismo comportamiento físico: bloquea radiación, no transfiere calor, no obstruye viento.

## Trampas comunes

| Síntoma | Causa |
|---------|-------|
| Cambiar el `Projection Factor` no actualiza ventanas existentes | Solo afecta ventanas **nuevas** — borrar y re-colocar |
| Alero generado, pero el preview no lo muestra | Al hacer Merge se actualizó el OSM pero el 3D View tiene el modelo viejo — Refresh |
| Severe geometric error tras editar OSM | Vértices no coplanares o mal ordenados — revisar a mano |
| Alero "desaparece" tras editar | Coordenadas mal — el polígono colapsó o se invirtió la normal |
| Resultado térmico no cambia | Verificar que el alero aparece en `Surface Outside Face Incident Solar Radiation` — si la radiación incidente sobre la ventana es la misma, el alero no está bloqueando |

## Estado del problema en versiones futuras de Open Studio

A la fecha de la clase 006, Open Studio **no permite** desde GUI extender el alero más allá del ancho de la ventana. La OpenStudio Coalition tiene un issue tracker — vale la pena revisarlo si pasa tiempo. Mientras tanto, el workaround manual es la vía oficiosa.

## Clases relacionadas

- [[../classes/006-DosZonasTermicasVentanasAleros]] — demo en vivo del Projection Factor y del workaround manual
- [[../classes/007-CasoBaseAleros]] — anécdota Paloma de validación, comparación caso base vs con aleros
