---
title: 003 — Mi Primera Simulación
type: clase
clase: 003
profesor: Guillermo Barrios del Valle
fuente: raw/videos/003_MiPrimeraSimulacion.md
fecha_ingesta: 2026-05-02
tags: [clase, openstudio, primera-simulacion, balance-aire, mezclado-perfecto, epw]
aliases: [Clase 003]
---

# 003 — Mi Primera Simulación

## Metadatos

- **Clase:** 003
- **Profesor:** Guillermo Barrios del Valle
- **Fuente:** `raw/videos/003_MiPrimeraSimulacion.md`
- **Tipo:** Clase mixta — recap teórico (~30%) + tour práctico de Open Studio (~70%)

## Resumen

Primera clase práctica del taller. Cierra el marco teórico iniciado en 001 y 002 con dos piezas que faltaban — el **balance de calor en la superficie interior** y el **balance de aire de la zona térmica** — y la suposición central que los conecta: **mezclado perfecto** del aire (toda la zona, una sola temperatura). En la segunda mitad, el profesor demuestra **end-to-end** la creación de un primer modelo en Open Studio: dibujo de geometría en FloorspaceJS, espacios y zonas térmicas, condiciones de frontera por color, descarga y asignación de un EPW desde OneBuilding, definición de un material y una construction, asignación a superficies, piso adiabático, y `Run`. Cierra anunciando la tarea: replicar el procedimiento con un cubo de 3×3×3 m.

Aparecen además dos hilos transversales del curso: la **narrativa computacional** (versionado de OSMs, estructura de carpetas, ZIP completos) y el principio de **caricatura computacional** — Energy Plus es una caricatura de la realidad, lo importante es saber qué se está descartando.

## Objetivos de aprendizaje

- Plantear el [[../concepts/Balance-de-Calor]] en la **superficie interior** y entender qué difiere respecto al exterior.
- Entender la suposición de [[../concepts/Mezclado-Perfecto]] y sus implicaciones.
- Distinguir [[../concepts/Espacio-vs-ZonaTermica|espacio y zona térmica]] en Open Studio.
- Conocer los cuatro tipos de [[../concepts/Condiciones-de-Frontera]] (Outdoor, Surface, Ground, Adiabática) y sus colores en Render by Boundary.
- Conocer los tres [[../concepts/Tipos-Superficie]] (Wall, Roof, Floor) y por qué importan para la convección.
- Entender qué es una [[../concepts/Subsuperficie]] (ventana, puerta) y la jerarquía superficie → sub-superficie.
- Conocer la suposición de [[../concepts/Radiacion-Interior-Distribuida]] y el modelo `FullInteriorAndExterior`.
- Ser capaz de seguir el procedimiento [[../procedures/Crear-Primera-Simulacion-OpenStudio]] de principio a fin.

## Recapitulación de las clases anteriores

Antes de entrar a Open Studio, el profesor recapitula y completa el balance:

1. **Balance en la superficie exterior** (visto en [[002-ConceptosBasicosBalancesCalor]]): tres componentes — radiación de onda corta absorbida, radiación de onda larga (con ground/sky/air/surroundings) y convección — igualados a la conducción `−k ∂T/∂x|_{x=0}`.
2. **Conducción a través del muro** (1D, dependiente del tiempo, con masa térmica):
   $$
   \rho \, c_p \, \frac{\partial T}{\partial t} = k \, \frac{\partial^2 T}{\partial x^2}
   $$
   En sistemas constructivos multi-capa, cada capa tiene su `ρᵢ`, `cₚ,ᵢ`, `kᵢ`. Se discretiza y se resuelve con CTF (default) o Diferencias Finitas.
3. **Las "caricaturas" se acumulan** — solo líneas rectas, flujo 1D perpendicular, material homogéneo. Ver [[../concepts/Caricatura-Computacional]].

## Balance de calor en la superficie interior

Pieza nueva: la condición de frontera en la cara **interior** del muro tiene tres componentes paralelos al exterior, pero con diferencias importantes:

$$
q''_{conv,i} + q''_{LWR,i} + q''_{SW,i} = -k \frac{\partial T}{\partial x}\bigg|_{x=L}
$$

### Componente 1 — Convección con el aire interior

$$
q''_{conv,i} = h_{c,i} \, (T_s - T_I)
$$

donde:
- `T_s` es la temperatura de la superficie interior.
- `T_I` es la **temperatura del aire indoor** de la zona — la incógnita que finalmente le interesa al diseñador.
- `h_{c,i}` depende del tipo de superficie ([[../concepts/Tipos-Superficie]]) — un techo no tiene el mismo `h_c` que un piso.

### Componente 2 — Radiación de onda larga (LWR) interior

A diferencia del exterior, la onda larga **interior** intercambia **solo entre las superficies del cuarto**. Las ventanas no participan: la radiación de onda larga **no atraviesa el vidrio** (el vidrio es opaco a IR). Aunque haya ventana en el muro, el balance LWR se cierra entre las paredes, piso y techo del cuarto.

$$
q''_{LWR,i} = \sum_j \varepsilon_j \, \sigma \, F_{s \to j} \, (T_j^4 - T_s^4)
$$

donde la suma corre sobre todas las superficies internas que **ven** a la superficie `s` (el [[../concepts/Factor-de-Vista]] vale 0 para superficies paralelas que están detrás).

> **Insight:** este intercambio radiativo es **instantáneo** y **muy potente** — proporcional a `(T⁴ − T_s⁴)`. Ejemplos del profesor: pasar al lado de un muro de piedra caliente; sentir el calor del tragafuegos de un semáforo (>700 K); el calor que se siente cuando una pared está fría enfrente. Una estrategia bioclimática puede usar este intercambio (paredes frías como sumidero radiativo).

### Componente 3 — Radiación de onda corta sobre la cara interior

A diferencia del exterior (donde es radiación solar global), la onda corta interior **viene de fuentes que emiten luz visible**:

- Radiación solar que **entra por una ventana**.
- Luminarias y proyectores (parte de su potencia es luz visible; el resto, calor convectivo).

E+ aplica una caricatura clave aquí: la radiación se **distribuye uniformemente**, y la **directa se asume al piso** por default. Detalle en [[../concepts/Radiacion-Interior-Distribuida]].

## Balance de aire en la zona térmica

Es el balance que cierra el problema y produce `T_I`, la temperatura del aire interior. Se construye sumando todas las contribuciones que entran/salen del volumen de aire:

- Convección con cada superficie interior (suma de `h_{c,i} A_i (T_s − T_I)`).
- Radiación que entra por ventanas y se queda en el cuarto.
- Cargas internas — personas, equipos, luces (no se modelan en el curso).
- Infiltración / ventilación — entrada de aire exterior con su propia temperatura y humedad (no se modela en el curso). Conservación de masa obliga a tratar también la humedad cuando entra aire seco/húmedo.

> "No voy a hablar de 'va a agregar' o 'va a quitar' porque puede entrar más frío, más caliente, o a la misma temperatura. Pero la masa también cuenta — tiene que haber conservación de masa."

### Suposición clave: mezclado perfecto

Toda esa suma se aplica a un volumen de aire con **una sola temperatura instantánea**. No hay estratificación, no hay plumas, no hay gradiente. Cada paso temporal:

1. E+ junta todos los flujos de calor que cruzaron las fronteras de la zona.
2. Los reparte uniformemente en la masa de aire.
3. Calcula la nueva `T_I`.

Detalle y consecuencias prácticas en [[../concepts/Mezclado-Perfecto]].

> "Energy Plus va a agarrar ese flujo de calor y lo va a agarrar todo el aire y lo va a mezclar perfectamente. Entre más grande sea mi espacio, más alejado voy a estar de esa suposición."

### Mediciones reales para comparar

Si se quiere comparar simulación vs. medición experimental, lo ideal es medir a varias alturas (tobillos, cadera, pecho — sentado y parado) y promediar. Una sola medición puntual no se compara directamente con la "T_I" reportada por E+.

## Acotación: qué se simula en el curso

El profesor reitera las simplificaciones que se mantienen vigentes:

- **Sin ventilación natural ni mecánica.** Open Studio expone poco de Airflow Network, y resolverlo bien requiere mucho más esfuerzo.
- **Sin cargas térmicas** — sin personas, sin equipos, sin iluminación.
- **Piso adiabático** — no se modela el ground.
- Sí se hace: ventanas, protecciones (aleros), sistemas constructivos.

Es **un cascarón sellado**. Los valores absolutos de temperatura no son realistas, pero el **orden** entre estrategias se preserva — ver [[../concepts/Caricatura-Computacional]].

## Open Studio — tour práctico

### Open Studio Model (OSM) y el hash anti-plagio

El archivo `.osm` es **texto plano**. Es una reescritura del IDF de Energy Plus — los mismos objetos, con sintaxis distinta.

Detalle no obvio: cada objeto que crea Open Studio recibe un **hash** (cadena alfanumérica única) como nombre interno. **El hash no se puede cambiar al renombrar un objeto.** El profesor usa esto como **detector de plagio**: si dos tareas tienen los mismos hashes, una se copió de la otra.

> "No me gusta jugar al policía, pero pero sí me gusta cuestionar."

### El folder hermano del OSM

Al guardar `001_volumetria.osm`, Open Studio crea junto a él un **folder con el mismo nombre** (`001_volumetria/`). Reglas:

- En cada `Run` ese folder se **borra y se regenera** — no guardar nada propio ahí.
- Sí contiene la **configuración de measures** del OSM. Compartir solo el `.osm` pierde los measures.
- Para entregar la tarea: **ZIP del proyecto completo** — ver [[../procedures/Estructura-Proyecto-Simulacion]].

### Editores de geometría

Open Studio puede usar varios editores de geometría:

| Editor | Estado en el curso |
|--------|--------------------|
| **FloorspaceJS** (integrado, JS, gratis) | El que se usa |
| **SketchUp** (con plugin) | Excelente, pero SketchUp ya es de paga |
| **Rhino** | Profesional, soportado |
| **Blender** | Soportado |
| **Design Builder** (GUI propia) | Programa aparte; ya trae editor |

FloorspaceJS guarda en formato **JSON**. Permite buscar la ubicación en un mapa de **OpenStreetMap** y dibujar la base sobre la imagen como referencia.

### Flujo de dibujo en FloorspaceJS

1. Geometry → New (o Edit Floorplan).
2. Configurar **grid** (esquina superior derecha) — típicamente 0.5 m o 1 m.
3. Herramienta **Rectangle** o **Polygon** → dibujar la planta.
4. Story height (panel Stories) → cambiar de 2.43 m default a la altura deseada (3 m en el ejemplo).
5. Renombrar espacios — convención `S:Norte`, `S:Sur` para distinguir de zonas térmicas (ver [[../concepts/Espacio-vs-ZonaTermica]]).
6. **Merge with Current OSM** → la geometría pasa al modelo principal.
7. Pestaña 3D View → Refresh para ver el resultado.

### Render by Boundary — el catálogo de colores

En el preview 3D, el selector **Render By → Boundary Conditions** colorea las superficies por su tipo de condición:

| Color | Condición | Significado |
|-------|-----------|-------------|
| **Azul** | Outdoors | Expuesto al sol y al viento — radiación incidente, convección, LWR con ground/sky/air |
| **Verde** | Surface (interzona) | Frontera entre dos zonas térmicas — el flujo que sale por una entra a la otra |
| **Café/marrón** | Ground | En contacto con el suelo — temperatura de ground |
| **Rojo** | Adiabatic | Flujo cero — superficie aislada del modelo |

Detalle de cada uno en [[../concepts/Condiciones-de-Frontera]].

### El truco de los espacios cercanos pero no pegados

El profesor demuestra una sutileza: **dos espacios separados por 1 cm en el editor NO comparten condición de frontera** — Open Studio los trata como dos muros independientes hacia Outdoors. Si quiero dos cuartos vecinos que comparten muro, las paredes deben **tocarse físicamente** en FloorspaceJS (línea punteada al unir).

Si se dejan separadas:

- E+ **sí** detecta que un objeto bloquea radiación al otro (sombreamiento).
- Pero E+ **no** conecta el calor — la transferencia que sale del muro va al ambiente, no al cuarto vecino.

Resultado: cosas que parecen "casi pegadas" se simulan mal. **Hay que pegar las paredes** y dejar que Open Studio convierta automáticamente Outdoor → Surface (verde).

### Espacios vs zonas térmicas — el doble paso

Aunque en el curso son 1:1, **espacio y zona térmica son objetos distintos** en Open Studio. Hay que crearlas por separado y mapearlas. Detalle en [[../concepts/Espacio-vs-ZonaTermica]].

> Para 200 zonas térmicas, este paso (drag-and-drop una a una) se vuelve insoportable — la salida es **scripting con Python o Ruby** (ej. Gabi, egresada del IER, trabaja en EE.UU. desarrollando estos scripts).

### Forzar piso adiabático

Default: el piso queda como **Ground**. Para el curso se cambia a **Adiabatic**:

- Pestaña Spaces → sub-pestaña Surfaces.
- Para cada piso: columna `Outside Boundary Condition` → de `Ground` a `Adiabatic`.

Cuando el modelo tiene varios pisos (multi-story): **todos los pisos intermedios pueden ser adiabáticos arriba y abajo** si la temperatura es similar entre niveles. Solo el piso de planta baja toca el ground; solo el techo del último piso toca outdoors.

### EPW y el archivo de clima

Detalle en [[../procedures/Descargar-EPW-OneBuilding]]. Resumen:

1. climate.onebuilding.org → ciudad → descargar ZIP.
2. Extraer el `.epw` → mover a `EPW/` del proyecto.
3. Pestaña **Site → Set Weather File** → seleccionar el `.epw`.

Las simulaciones de E+ corren en **horario civil** (uso horario), no horario solar.

### Materials y Constructions

Diferencia clave:

- **Material** = una capa con sus propiedades térmicas y un espesor.
- **Construction** = una secuencia ordenada de materiales — de **exterior a interior**.

Pasos:

1. Pestaña Materials → sub-pestaña **Materials** (no `No Mass Materials` — esos no respetan masa térmica).
2. Crear el material; rellenar conductivity, density, specific heat, thermal absorptance (emisividad), solar absorptance, visible absorptance.
3. Pestaña Constructions → crear construction → arrastrar materiales en orden ext→int.
4. Pestaña Spaces → Surfaces → asignar la construction a cada superficie (drag-and-drop desde My Model).

> Los campos en **verde** son defaults de E+. Cuando los modificas pierden el color verde — pista visual para saber qué tocaste.

### Tipos de superficie y `h_c`

Tres tipos: Wall, Roof, Floor. **El coeficiente convectivo depende del tipo** — un techo no tiene el mismo `h_c` que un piso. E+ aplica correlaciones distintas según inclinación. Detalle en [[../concepts/Tipos-Superficie]].

### Sub-superficies (ventanas, puertas)

Ventanas y puertas son **sub-superficies** que viven dentro de un muro. No pueden ocupar el 100% del muro — hay un margen mínimo. Si en la realidad no hay muro (cafetería abierta), se crea un **muro virtual** y dentro una ventana muy grande. Detalle en [[../concepts/Subsuperficie]].

## Narrativa computacional

El profesor introduce una metodología que se aplicará todo el semestre:

1. **Carpeta por proyecto** — sin acentos, sin eñes, sin espacios.
2. **Sub-carpetas** mínimas: `OSM/`, `EPW/`, `notebooks/`.
3. **Versionado numerado** de OSMs: `001_volumetria.osm`, `002_dosZonas.osm`, …
4. **Nunca borrar** versiones anteriores. Si una rompe, regresar a la anterior.
5. **Save As** (no Save) para cada cambio sustantivo → nueva versión.
6. Cuando el modelo está listo como **caso base**, ramificar: `006_caso_base.osm` → variantes 007 (color), 008 (orientación), 009 (sombras), …
7. **Entrega = ZIP del folder completo** (no solo el OSM). No usar TAR.

> "Aquí es donde hay que practicar el desapego. No me sale en una hora — regrésate al paso anterior y vuelve a hacerlo."

Detalle en [[../procedures/Estructura-Proyecto-Simulacion]].

## Tarea de la semana

> **Cubo de 3×3×3 m**, dos sistemas constructivos:
> - **Tabique** en los 4 muros.
> - **Concreto** en piso y techo.
>
> Construction de un solo material en cada caso, EPW de la ciudad escogida (asignada en clase 002), piso adiabático, sin ventanas. Ejecutar `Run` y entregar el proyecto completo en ZIP.

Procedimiento detallado: [[../procedures/Crear-Primera-Simulacion-OpenStudio]].

**Comunicación durante la semana:** chat del grupo. Si algo no sale, describir bien el problema y adjuntar el ZIP del proyecto. El profesor responde "en algún momento de la madrugada".

## Conceptos derivados (referencias)

Conceptos nuevos introducidos o profundizados:

- [[../concepts/Mezclado-Perfecto]] — toda la zona, una sola temperatura
- [[../concepts/Espacio-vs-ZonaTermica]] — distinción de Open Studio
- [[../concepts/Caricatura-Computacional]] — principio metodológico
- [[../concepts/Tipos-Superficie]] — Wall/Roof/Floor y dependencia de `h_c`
- [[../concepts/Subsuperficie]] — ventanas y puertas dentro de superficies
- [[../concepts/Radiacion-Interior-Distribuida]] — la directa al piso, la difusa repartida
- [[../concepts/Balance-de-Calor]] — actualizado con balance interior y balance de aire
- [[../concepts/Condiciones-de-Frontera]] — actualizado con catálogo de colores OS

## Conexiones

- ← **Anterior:** [[002-ConceptosBasicosBalancesCalor]] — fundamentos físicos
- → **Siguiente:** _004-InterpretandoMensajesConstructionSets_ — debugging de la simulación, construction sets
- → Procedimiento de la tarea: [[../procedures/Crear-Primera-Simulacion-OpenStudio]]
- → Estructura de carpetas: [[../procedures/Estructura-Proyecto-Simulacion]]
- → EPW: [[../procedures/Descargar-EPW-OneBuilding]]

## Recursos mencionados

- **OneBuilding.org** — repositorio global de archivos EPW.
- **OpenStreetMap** — integrado en FloorspaceJS para dibujar sobre mapas reales.
- **Gabi** (egresada del IER) — ejemplo de carrera basada en scripting (Python/Ruby) para automatizar Open Studio en consultoría.
- **SketchUp, Rhino, Blender, Design Builder** — alternativas al editor FloorspaceJS.
