# 006 — 2 Zonas Térmicas con Ventanas y Aleros

## Metadatos
- **Clase:** 006
- **Título:** 2 Zonas Térmicas con Ventanas y Aleros
- **Profesor:** Guillermo Barrios del Valle
- **Temas:** geometría con diferentes alturas, ventanas como sub-superficies, materiales de vidrio, overhangs y fins (projection factors), aleros y celosías, superficies de sombramiento, alero equivalente, flujo de trabajo incremental, análisis rápido con Python

---

## Resumen

Clase práctica donde se construye desde cero una simulación con **dos zonas térmicas de diferentes alturas**, se agregan **ventanas** (sub-superficies), se introducen **dispositivos de protección solar** (overhangs y fins) y se demuestra que los aleros **solo bloquean radiaci��n, no participan en transferencia de calor ni obstruyen viento**. Se cierra con un análisis rápido en Python para identificar el día más cálido.

---

## 1. Geometría con diferentes alturas

### Stories vs Spaces en FloorSpaceJS

- Cada **Story** (piso) tiene una altura floor-to-ceiling por defecto (2.43m)
- Si se cambia la altura en el **Story**, todos los espacios de ese piso heredan la nueva altura
- Si se cambia la altura en un **Space** individual, solo ese espacio se modifica
- Para tener dos zonas con alturas diferentes en el mismo piso → definir la altura en cada Space, no en el Story

### Intersección automática de superficies

- Cuando dos espacios comparten una pared en FloorSpaceJS, la herramienta **corta automáticamente** las superficies y asigna condición de frontera **Surface/Interzone**
- Si las superficies no son del mismo tamaño, la intersección no se genera automáticamente
- Si un espacio tiene doble altura, la pared compartida se corta: parte inferior con condición Surface, parte superior queda como Outdoor
- **Limpiar geometría:** en modelos complejos (BIM), las intersecciones automáticas fallan y hay que corregir manualmente — en este curso se evitan geometrías complejas para no caer en eso

### Principio de la superficie compartida

- La condición Surface/Interzone significa que **es un solo muro**, no doble
- Dos superficies empalmadas = una superficie física
- El flujo de calor que sale de una zona entra a la otra

---

## 2. Flujo de trabajo incremental

**Principio fundamental:** hacer cambios pequeños, correr la simulación después de cada paso y verificar que funciona antes de seguir.

1. Crear geometría → verificar condiciones de frontera → guardar como v001
2. Agregar materiales, construcciones, Construction Set → correr simulación → verificar ERR → guardar como v002
3. Agregar ventanas → correr → verificar → guardar como v003
4. Agregar overhangs/fins → correr → verificar → guardar como v004

**Raz��n:** si acumulas muchos cambios sin correr, cuando aparece un error no sabrás cuál paso lo causó.

Para el proyecto final: terminar completamente el **caso de referencia** (incluyendo variables de salida) antes de crear variantes. Así solo se cambia un parámetro por variante sin duplicar trabajo.

---

## 3. Materiales: relación densidad-conductividad

- Materiales más densos tienen **mayor conductividad** — es una propiedad molecular
- Si una fuente reporta alta densidad con baja conductividad (o viceversa), es sospechosa

| Material | Densidad (kg/m³) | Conductividad (W/m·K) |
|----------|-------------------|------------------------|
| EPS (poliestireno expandido) | ~45 | ~0.035 |
| Tabique rojo | ~1400 | ~0.7 |
| Concreto alta densidad | ~2400 | ~2.0 |

- **Pinturas e impermeabilizantes** no se simulan como capas — son < 1-2 mm y su efecto térmico es despreciable. Lo que importa es el **color** (absorptancia solar)

---

## 4. Ventanas

### Agregar ventanas en FloorSpaceJS

- En el editor: **Component** → seleccionar tipo Window
- Hacer clic sobre un muro para colocar la ventana
- Propiedades configurables:

| Propiedad | Descripción | Default |
|-----------|-------------|---------|
| **Window to Wall Ratio** | Fracción del área de ventana respecto al muro | — |
| **Height** | Altura de la ventana | — |
| **Width** | Ancho de la ventana | — |
| **Sill Height** | Altura de antepecho (del piso al borde inferior de la ventana) | 0.91 m |

- Todas las ventanas creadas con el mismo componente son idénticas — para diferentes tamaños, crear nuevos componentes
- En la realidad, se simplifica: múltiples ventanas de una fachada → una sola ventana de **área equivalente** (el marco se ignora como simplificación, aunque ocupa ~10% del área)

### Ventanas como sub-superficies

- Las ventanas son **sub-superficies** — necesitan una superficie padre (muro)
- No pueden ocupar el 100% del muro
- Aparecen como transparentes en el 3D View
- Se listan en la pestaña **Sub Surfaces** (no en Surfaces)
- Necesitan su propio sistema constructivo (material de vidrio)

### Materiales de vidrio

Dos opciones en Open Studio:

| Tipo | Uso | Complejidad |
|------|-----|-------------|
| **Glazing Window Material** | Vidrio simple, una capa | Requiere: espesor, transmitancia solar/visible, reflectancia, transmitancia IR, conductividad |
| **Simple Glazing Window Material** | Ventana compleja simplificada | Reduce una ventana multicapa (doble vidrio + argón + low-E) a pocos parámetros |

**Recomendación para el curso:** usar el vidrio **Clear 3mm** que ya viene incluido en la librería de EnergyPlus. En México las ventanas estándar son de 3, 6 y 9 mm de vidrio flotado.

**Propiedades espectrales:** se puede definir comportamiento por intervalo de longitud de onda (para materiales experimentales) o dar promedios espectrales.

### Marcos de ventana (framing)

- En EnergyPlus se puede definir el material del marco
- **Aluminio** (conductividad ~1.5 W/m·K) crea un **puente térmico** — muy malo con A/C
- **PVC** (conductividad ~0.7 W/m·K) es mejor opción térmica
- En el curso no se modelan marcos, pero es importante saber que existen y afectan

### Normativa mexicana

- NOM-008: ventanas máximo ~20% del área de fachada en vivienda, ~25% en comerciales

---

## 5. Overhangs y fins (protecciones solares)

### Definición desde FloorSpaceJS

En el componente de ventana se puede configurar:

| Propiedad | Descripción |
|-----------|-------------|
| **Overhang Projection Factor** | Superficie horizontal arriba de la ventana. Factor = profundidad del alero / altura de la ventana |
| **Fin Projection Factor** | Superficies verticales a los lados. Factor = profundidad / ancho de la ventana |

Con factor = 1, el alero tiene la misma profundidad que la altura de la ventana.

### Limitaciones de la interfaz

- El overhang se genera **exactamente del ancho de la ventana** — no se puede hacer más ancho desde Open Studio
- **Hack:** abrir el archivo .osm con un editor de texto, buscar la superficie del overhang (ej. "Face 18"), modificar las coordenadas de los 4 puntos para alargar
- También se puede pedir a un LLM que modifique las coordenadas

### Problema de diseño

- Un alero del mismo ancho que la ventana solo protege bien cuando el sol está perpendicular
- Cuando el sol viene de un ángulo lateral (trayectoria solar aparente), la sombra se desplaza y el alero pierde efectividad
- **Solución:** combinar overhang horizontal + fin vertical
- Los **ángulos importantes** son: el ángulo desde el borde inferior de la ventana hasta el extremo del alero (define cuándo el sol entra), y el ��ngulo horizontal hacia las esquinas del alero

---

## 6. Superficies de sombramiento (aleros)

### Qué hacen

- **Bloquean radiación solar directa y difusa** — proyectan sombras
- Pueden tener **reflectancia/absorptancia** asignada
- Pueden ser **semi-transparentes** (celosías)

### Qué NO hacen

- **No participan en la transferencia de calor** — no tienen temperatura, no conducen calor
- **No obstruyen el viento** — EnergyPlus no resuelve mecánica de fluidos computacional (CFD)
- La conducción desde un alero caliente hacia el muro es despreciable (proceso unidimensional, el calor no "dobla la esquina")
- Sí emiten radiación de onda larga hacia las superficies cercanas (esto sí se toma en cuenta)

### Implicación para muros exteriores

- Un muro dibujado como superficie de sombramiento solo bloqueará radiación, no viento
- Para simular el efecto del viento bloqueado → hay que modificar **coeficientes de descarga** de las ventanas

---

## 7. Celosías y alero equivalente

### Celosías (louvers)

Combinación de superficies horizontales y verticales repetidas, espaciadas uniformemente:

- **Ventaja:** bloquean radiación directa muy efectivamente mientras permiten ventilación
- Muy usadas en climas cálidos (costa mexicana)
- El ángulo de protección depende del espaciamiento y la profundidad de cada elemento

### Concepto de alero equivalente

- Un conjunto de celosías pequeñas con un ángulo de protección determinado equivale a un **alero gigante** con el mismo ángulo
- Entre más fina la rejilla (menor espaciamiento) → mayor ángulo → mayor alero equivalente
- En la simulación se modela el **alero equivalente** (una superficie grande) en vez de cada celosía individual
- Ejemplo real: cafetería universitaria con celosías tipo serpentín → alero equivalente enorme

### Calibración con ventilación natural

- El alero equivalente no absorbe velocidad del viento
- Para calibrar: medir velocidad de viento real con anemómetros 3D y ajustar coeficientes de descarga en la simulación
- Este es uno de los grandes retos de la simulación en México: las edificaciones dependen de ventilación natural, pero los modelos son limitados para esto

---

## 8. Análisis rápido con Python

Demostración de un análisis exploratorio en ~10 minutos:

1. Crear ambiente con `uv init` + `uv add pandas matplotlib jupyter notebook` + `uv add git+<ear_tools>`
2. `uv run jupyter notebook`
3. Cargar datos: `read_sql(f, alias=True)`
4. Identificar día más cálido: `To.resample('D').mean().idxmax()`
5. Graficar ventana temporal de ~7 días alrededor del día más cálido

**Concepto clave:** el `datetime` como índice es fundamental para el manejo eficiente de series temporales. Convertir fechas a datetime y usarlas como índice del DataFrame es un "parteaguas" en el análisis de datos.

---

## Conceptos clave

- **[[Zona-Termica]]** — diferentes alturas, superficie compartida como muro único
- **[[Condiciones-de-Frontera]]** — intersección automática, Surface/Interzone con alturas diferentes
- **[[Sistemas-Constructivos]]** — relación densidad-conductividad, pinturas despreciables como capa
- **Ventanas** — sub-superficies, materiales de vidrio, marcos como puentes térmicos
- **Superficies de sombramiento** — solo radiación, sin calor ni viento
- **Celosías y alero equivalente** — protección solar efectiva que permite ventilación

Conceptos previos referenciados: [[Balance-de-Calor]], [[Masa-Termica]], [[Absorptancia-Solar]]

## Herramientas mencionadas

[[Open-Studio]] · [[EnergyPlus]] · [[Python]] · FloorSpaceJS · SketchUp · Radiance

## Conexiones

- **Anterior:** [[005-AnalisisSimulacionesPython]] — Flujo de análisis con Python que aquí se aplica
- **Anterior:** [[004-InterpretandoMensajesConstructionSets]] — Construction Sets que aquí se usan
- **Siguiente:** [[007-CasoBaseAleros]] — Caso base de referencia y estrategias de sombreado

## Tarea

- No se dejó tarea nueva esta clase
- Si algún equipo no terminó la tarea anterior (dos zonas térmicas), completarla
