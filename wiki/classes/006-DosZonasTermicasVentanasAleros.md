---
title: 006 — Dos Zonas Térmicas con Ventanas y Aleros
type: clase
clase: 006
profesor: Guillermo Barrios del Valle
fuente: raw/videos/006_DosZonasTermicasVentanasAleros.md
fecha_ingesta: 2026-05-02
tags: [clase, openstudio, ventanas, aleros, sombreamiento, dos-zonas, geometria, dia-mas-calido]
aliases: [Clase 006]
---

# 006 — Dos Zonas Térmicas con Ventanas y Aleros

## Metadatos

- **Clase:** 006
- **Profesor:** Guillermo Barrios del Valle
- **Fuente:** `raw/videos/006_DosZonasTermicasVentanasAleros.md`
- **Tipo:** Clase práctica recreativa — el profesor reconstruye desde cero un modelo de dos zonas con ventanas y aleros

## Resumen

Clase larga (≈2 horas) donde el profesor **rehace desde cero** un modelo con dos zonas térmicas, materiales, Construction Set, ventanas y aleros, debugea en vivo y cierra con análisis Python. Mucho contenido técnico nuevo:

- **Frontera de superficie automática** y **limpieza de geometría** (cuándo FloorspaceJS corta solo, cuándo no).
- **Alturas distintas por Space** (Stories tienen default heredable; cada Space puede sobreescribir).
- **Ventanas como sub-superficies**: parámetros (WWR, sill height, height/width), materiales (Simple Glazing vs Glazing detallado), vidrio mexicano típico (3/6/9 mm flotado), framing (marcos), historia del IER de cubículos low-E mal aplicados.
- **Aleros y parteluces**: Projection Factor, qué SÍ y qué NO hacen físicamente, **limitación crítica de Open Studio** (alero del mismo ancho que la ventana) y workaround editando el OSM.
- **Aleros equivalentes (celosías)** y el paper del grupo IER sobre la cafetería.
- **Día más cálido**: por qué hay que explicitar el **criterio**.
- Recapitulación del análisis Python (`iertools`, recorte temporal con `dateutil + Timedelta`).

No hay tarea nueva — quien no haya entregado la anterior, debe entregarla.

## Recap clase 003-004 — frontera de superficie

> "La condición de frontera verde — de superficie o de interzona — a veces no se hace sola y hay que forzarla. Para que se haga sola tienen que pasar dos cosas: las superficies se traslapan, y son del mismo tamaño."

Cuando dos espacios se unen físicamente y sus muros coinciden:

- FloorspaceJS convierte automáticamente Outdoors → Surface (verde).
- El muro compartido es **uno solo**, no doble (las dos superficies "se vuelven una" en el modelo).
- El calor que sale por una zona entra por la otra.

Cuando los muros se traslapan **parcialmente** (tamaños distintos), FloorspaceJS no convierte la condición → hay que **limpiar la geometría**. Detalle en [[../concepts/Limpiar-Geometria]].

> "En SketchUp hay una herramienta que hace una intersección — proyecta una superficie sobre la otra y la corta. Todo eso lo hace FloorspaceJS de manera automática. **A veces.**"

## Stories y Spaces con alturas distintas

> "Si yo cambio la altura en el Space, solo ese Space va a tener esa altura. Si la cambio en el Story, todo lo que esté en ese nivel va a tener esa altura."

El profesor demuestra creando dos Spaces:

- Space `este`: 2.5 m de altura.
- Space `oeste`: 5 m de altura (doble).

Resultado al hacer Merge: FloorspaceJS detecta que el techo del cubo bajo se traslapa con la **parte inferior** del muro del cubo alto, y **corta automáticamente** ese muro alto en dos sub-superficies:

- La sub-superficie de la mitad inferior queda con condición **Surface** (verde) compartida con el techo del cubo bajo.
- La sub-superficie superior queda como **Outdoors** (azul).

Caso típico de geometría auto-limpiada por FloorspaceJS — sería **mucho más complicado en SketchUp** (manual con la herramienta Surface Intersect).

## Materiales típicos del taller — relación densidad ↔ conductividad

> "Entre más denso sea un material, normalmente su conductividad es mayor."

Materiales de la demo:

| Material | k (W/m·K) | ρ (kg/m³) | cₚ (J/kg·K) | α |
|----------|-----------|-----------|-------------|---|
| Tabique rojo 14 cm | 0.7 | 1400 | 1000 | 0.7 |
| Concreto alta densidad 15 cm (blanco) | 2.0 | 2400 | 1000 | 0.3 |
| EPS (poliestireno expandido) | 0.035 | 45 | — | — |

Comparación: el EPS es ~50× menos denso y ~60× menos conductivo que el concreto. La relación ρ↔k no es lineal pero sí **monotónica**.

> "Si tienen una fuente y ven que las densidades y conductividades no corresponden — revisen su fuente. Hoy en día muchas vienen del internet o de Google AI y pueden tener errores."

Para validar materiales: chequear **consistencia ρ-k**.

### Pinturas e impermeabilizantes

- Espesores < 1 mm.
- Efecto térmico por conducción **despreciable**.
- Lo que importa es el **color** (absortancia solar).
- En el modelo: no se agregan como capa; se asigna la absortancia a la superficie expuesta.

## Ventanas — sub-superficies

Concepto y parámetros completos en [[../concepts/Ventanas]]. Procedimiento práctico en [[../procedures/Agregar-Ventanas-OpenStudio]].

### Parámetros del componente window en FloorspaceJS

| Parámetro | Significado |
|-----------|-------------|
| **Window-to-Wall Ratio (WWR)** | Fracción del muro padre |
| **Height / Width / Sill Height** | Dimensiones absolutas |
| **Window Type** | `FixedWindow` (default), `OperableWindow`, etc. |
| **Overhang Projection Factor** | Genera alero arriba |
| **Fin Projection Factor** | Genera parteluces a los lados |

> En NOM-008 mexicana: **20% WWR máximo en vivienda**, **25% en comerciales**. Estas máximos buscan limitar carga térmica.

### Restricción 100%

Una ventana **no puede ocupar el 100%** del muro padre. Caso típico: cafetería abierta — modelar muro virtual con ventana al 95-98%.

### Materiales de ventana — dos opciones

| Opción | Caracterización |
|--------|-----------------|
| **Glazing Window Material** | Capa por capa (vidrio + gas + low-E + ...) — fiel pero pide muchos parámetros |
| **Simple Glazing System** | 3 parámetros: U-factor, SHGC, Visible Transmittance — recomendado para el taller |

Vidrio mexicano típico: **flotado de 3/6/9 mm**, transmitancia ~0.88, k ~1 W/m·K. Open Studio trae precargado un material `glazing 3mm`.

### Marcos (framing) — el puente térmico ignorado

> "El marco ocupa un 10-20% del área. Y el área efectiva de vidrio no es la misma que si mido los marcos de aluminio."

Conductividades de marcos:

| Material | k (W/m·K) |
|----------|-----------|
| Vidrio | ~1.0 |
| Aluminio | ~150 (sin ruptura) / ~1.5 (con ruptura) |
| PVC | ~0.17 |

> "En todo el instituto donde hay aire acondicionado, tenemos ventanas de aluminio. Genera puente térmico — terrible si tenemos AC. PVC sería mejor."

En el taller los marcos se **ignoran** (el área de marco se cuenta como cristal). Caricatura aceptable para evaluar estrategias.

### Caso histórico — cubículos plataforma solar IER

Hace años se instalaron **ventanas con película low-E** en los cubículos de la plataforma solar pensando que reducirían carga térmica. Resultado: **se volvieron infernales** porque la película reflejaba el IR pero **absorbía** el visible — el vidrio se calentaba y radiaba en ambas direcciones (incluyendo hacia adentro).

Lección: las ventanas se diseñan por **conjunto de propiedades espectrales**, no por una sola. Aplicable hoy a **paneles fotovoltaicos translúcidos** que se proponen para fachadas — investigación pendiente del grupo (Aaron, Matthew, Fabi).

## Aleros y parteluces — superficies de sombramiento

Concepto completo en [[../concepts/Superficies-de-Sombramiento]]. Procedimiento en [[../procedures/Agregar-Aleros-OpenStudio]].

### Projection Factor

$$
PF = \frac{\text{longitud horizontal del alero}}{\text{altura de la ventana}}
$$

Análogo para `Fin Projection Factor` (parteluces verticales).

### Qué SÍ / NO hacen los aleros

| Mecanismo | E+ |
|-----------|-----|
| Bloquear radiación solar directa y difusa | **Sí** |
| Tener temperatura propia, conducir calor | **No** — los aleros no participan en transferencia de calor |
| Obstruir el viento | **No** — E+ no resuelve mecánica de fluidos externa |
| Reflejar radiación según su material | **Sí** |

> "Los aleros son superficies opacas que sí tienen reflectancia. Pero la absortancia me gusta pensarla más como reflectancia, porque la absortancia gana temperatura, y este no — solo refleja."

### Limitación crítica de Open Studio

> El alero generado por FloorspaceJS tiene **el mismo ancho que la ventana**. Eso es **terrible** porque el sol oblicuo (mañanas, tardes, fachadas E/W) proyecta sombras laterales que caen fuera del alero.

#### Ángulos de diseño

- **Vertical**: ángulo desde la base de la ventana al borde del alero.
- **Horizontal/azimut**: ángulo desde el borde de la ventana al borde lateral del alero.

Un alero efectivo se extiende **lateralmente más allá del ancho de la ventana**. Open Studio no lo permite desde GUI.

### Workaround — editar el OSM

El OSM es texto plano. Procedimiento (ver [[../procedures/Agregar-Aleros-OpenStudio]]):

1. Identificar el `Face N` del alero en el preview 3D.
2. Cerrar Open Studio.
3. Abrir el OSM con un editor de texto, buscar `Face N`.
4. Modificar las coordenadas de los 4 vértices (mover de a pares para mantener coplanaridad).
5. Recargar el OSM y verificar visualmente.

> El profesor también demuestra usar **IA (ChatGPT/Claude)** para hacer la transformación geométrica. Tasa de éxito ~50%; cuando falla suele ser por orden de vértices o coplanaridad rota.

### Aleros equivalentes — celosías

Una **celosía** (rejilla horizontal de listones) bloquea radiación tanto como un alero **mucho más grande** mientras se conserve el ángulo crítico. Y permite **paso de aire** — combinación ideal para climas cálidos.

#### Caso del paper de la cafetería del IER

El grupo modeló la cafetería con celosía + sistema evaporativo. La celosía se modeló como un **alero equivalente** — preserva los ángulos sin dibujar cada listón. Paper recién enviado (Miriam, Guadalupe y otros). Tomó tres años.

> "Las celosías son buenísimas — tienen lo mejor de los dos mundos: bloquean radiación con efectividad de un alero gigante y permiten ventilación."

### Reflectancia del material del alero

El material del alero **refleja** una fracción de la radiación. Esa radiación reflejada puede **caer en la ventana** que se intentaba sombrear → contraproducente.

Estrategias:

- Aleros con perfil **reflectante hacia afuera**.
- Análogo en iluminación natural: **light shelves** que redirigen luz al techo.

### Recomendación para el proyecto final

Combinar **alero horizontal + parteluz vertical**:

| Orientación | Estrategia |
|-------------|------------|
| Sur | Alero horizontal extendido lateralmente |
| Norte | Sombreamiento mínimo |
| Este / Oeste | Parteluces verticales |

## Workflow recomendado — paso a paso

> "Hagan paso por paso. Vayan agregando una cosa, corran, revisen el `.err`, agreguen la siguiente. Si se equivocan al final ya no sabrán cuál de las 10 cosas fue."

Patrón que el profesor enfatiza:

1. **Geometría limpia** → correr → revisar `.err` (cero severes) → guardar `001_volumetria.osm`.
2. **Materiales y Constructions** → correr → revisar → `002_constructions.osm`.
3. **Construction Set** asignado a la edificación → correr → revisar → `003_constructionSet.osm`.
4. **Zonas térmicas** asignadas → correr → revisar → `004_zonas.osm`.
5. **EPW** asignado → correr → revisar → `005_conEPW.osm`.
6. **Output Variables** vía measures → correr → verificar SQL/CSV → `006_outputVars.osm`.
7. **Caso base estable** → ramificar variantes:
   - `007_color.osm`, `008_alero.osm`, `009_orientacion.osm`...

> "Si empiezo a trabajar y cualquier cosa que le haga al modelo me voy a tener que ir al otro y hacer lo mismo, eso me genera trabajo doble."

Reproducibilidad y narrativa computacional — ver [[../procedures/Estructura-Proyecto-Simulacion]] y [[../concepts/Caricatura-Computacional]].

## Día más cálido — explicitar el criterio

> "Yo a propósito ambivalentemente no especifiqué 'el día más cálido'. Y vi que Ale escogió 'el día con la temperatura más alta'. Pero hay otros criterios."

Criterios posibles para "el día más cálido":

| Criterio | Cómo se calcula |
|----------|------------------|
| Día con T máxima absoluta | `df.TO.idxmax()` |
| Día con T promedio diario más alto | `df.TO.resample("D").mean().idxmax()` |
| Día con más grados-hora cálidos | Acumulado del modelo adaptativo |

Para el clima de Cuernavaca: el día con T promedio más alto sale **31 de mayo** (no en pleno verano, porque después llegan las lluvias).

> Lección: **siempre explicitar el criterio** cuando se reporta un análisis. "Día más cálido" no es suficiente.

## Warnings nuevos observados

Aparecidos en esta simulación al agregar ventanas y aleros:

| Warning | Significado | Acción |
|---------|-------------|--------|
| `Coliniar vertices` | E+ detectó vértices alineados redundantes y los borró. Inocuo. | Ignorar |
| `Weather location difference` | Pequeña diferencia (~0) en coordenadas al traducir OSM→IDF | Ignorar |

> "En investigación seria, eliminamos **todos** los warnings. En el curso vivimos con los del catálogo conocido si los entendemos."

Comparación con C: warnings peligrosos vs aceptables. Usar un float como int es un warning peligroso por redondeo del compilador. Análogo: tipo de warning que ignoras solo si **sabes** que es inocuo en tu caso.

Detalle en [[../concepts/Mensajes-EnergyPlus]].

## Análisis Python — recap clase 005

Hecho en vivo al final de la clase:

```python
import pandas as pd
import matplotlib.pyplot as plt
from iertools.read import read_sql
from dateutil.parser import parse

dos = read_sql("../OSM/006_outputVars/run/eplusout.sql", alias=True).data

# Día más cálido por promedio diario
f1 = dos.TO.resample("D").mean().idxmax()
f2 = f1 + pd.Timedelta(days=2, hours=7)

fig, ax = plt.subplots(figsize=(12, 4))
ax.plot(dos.T_este,   label="T_este")
ax.plot(dos.T_oeste,  label="T_oeste")
ax.plot(dos.TO,       label="TO")
ax.set_xlim(f1, f2)
ax.legend()
```

Comentario sobre `dateutil + Timedelta` (preferido sobre fecha hardcodeada):

> "Cuando ponen en corchetes la fecha, la tienen que estar especificando en todos lados. Definir `f1` y `f2` con dateutil + Timedelta es más elegante."

## Tarea

> No hay tarea nueva. Quien no haya entregado la anterior debe entregarla. La progresión sigue siendo dos zonas térmicas — agregar ventanas y aleros es opcional pero recomendado para empezar a estudiar estrategias bioclimáticas.

## Conceptos derivados

Conceptos nuevos:

- [[../concepts/Ventanas]]
- [[../concepts/Superficies-de-Sombramiento]]
- [[../concepts/Limpiar-Geometria]]

Conceptos profundizados:

- [[../concepts/Subsuperficie]] — caso completo de ventanas, WWR
- [[../concepts/Sistemas-Constructivos]] — materiales de ventana (Simple Glazing vs Glazing)
- [[../concepts/Construction-Set]] — slot Sub Surface
- [[../concepts/Mensajes-EnergyPlus]] — warnings de colineares y weather location
- [[../concepts/Caricatura-Computacional]] — caricaturas nuevas: aleros sin transferencia de calor, marcos ignorados, alero del mismo ancho que la ventana
- [[../concepts/Espacio-vs-ZonaTermica]] — alturas heredables/sobreescribibles por Space

Procedimientos nuevos:

- [[../procedures/Agregar-Ventanas-OpenStudio]]
- [[../procedures/Agregar-Aleros-OpenStudio]]

## Conexiones

- ← **Anterior:** [[005-AnalisisSimulacionesPython]] — análisis Python
- → **Siguiente:** _007-CasoBaseAleros_ — proyecto bioclimático: caso base, evaluación de aleros
- → Procedimientos clave:
  - [[../procedures/Agregar-Ventanas-OpenStudio]]
  - [[../procedures/Agregar-Aleros-OpenStudio]]
  - [[../procedures/Configurar-Construction-Set]] (slot Sub Surface)
  - [[../procedures/Analizar-Resultados-Python]]

## Recursos mencionados

- **Paper de la cafetería del IER** — Miriam, Guadalupe et al. — celosía + evaporative cooling, alero equivalente. Recién enviado.
- **Plataforma solar IER** — caso histórico de cubículos con low-E mal aplicada.
- **Aaron, Matthew, Fabi de Braille** — proyecto en formación sobre paneles fotovoltaicos translúcidos en fachadas.
- **OneBuilding** — descarga del EPW (igual que clase 003).
- **Quarto + uv** — mencionado de paso para cuadernos reproducibles.
- **Notepad++ / VS Code / Sublime** — editores de texto para modificar el OSM directamente.
- **ChatGPT / Claude** — usados como hack para transformaciones geométricas en el OSM.
