---
title: 014 — Infiltración, Floorspace con Plano e Introducción a Window LBNL
type: clase
clase: 014
profesor: Guillermo Barrios del Valle
fuente: raw/videos/014_ACH_FloorSpace_WindowLBNL.md
fecha_clase: 2026-05-22
fecha_ingesta: 2026-05-22
tags: [clase, infiltracion, cambios-aire, floorspace, window-lbnl, shgc, ventanas, proyecto-final, asistente-rag]
aliases: [Clase 014, ACH FloorSpace WindowLBNL, Infiltración y Window]
---

# 014 — Infiltración, Floorspace con Plano e Introducción a Window LBNL

## Metadatos

- **Clase:** 014 (22 de mayo de 2026)
- **Profesor:** Guillermo Barrios del Valle
- **Fuente:** `raw/videos/014_ACH_FloorSpace_WindowLBNL.md`
- **Tipo:** Clase técnica densa, tres bloques: (1) configurar **infiltración constante** en Open Studio, (2) **importar plano como imagen** en FloorspaceJS y dibujar encima, (3) intro a **Window LBNL** + instalación. Adicionalmente: actualización del asistente del curso (migración a Mac Mini) y cambio de **Casa 1 → Casa 3** en el proyecto final.

## Resumen ejecutivo

| Bloque | Lo decisivo |
|---|---|
| **ACH** | El profesor se desdice: tras hablar con Miriam, **sí se incluye infiltración** en el proyecto (0.5 ACH constante). [[../procedures/Agregar-Infiltracion-OpenStudio]] |
| **Bot del curso** | Migrado de Raspberry-casa a Mac Mini-IER. Bot accesible remotamente. **Periodo de pruebas + premio en puntos por reportar errores**. |
| **Casa 3** | El PDF del proyecto cambió: la **Casa 1** de Decide y Construye **ya no está disponible online** → se usa la **Casa 3**. Sigue siendo vivienda progresiva (3 etapas). |
| **Floorspace** | Truco: usar el plano como **imagen de fondo** y calibrar el grid con un lado conocido (7 m). 7 zonas térmicas en su demo. [[../procedures/Importar-Plano-FloorspaceJS]] |
| **Window LBNL** | Solo Windows (Parallels en Mac). Calcula **SHGC** y **U-value**. Instalación requiere VC++ Redist x86 + Intel Fortran Compiler Runtime. **Sin AC, las ventanas dobles no rentan**. Uso real → clase 015. |

> "Hoy debo desdecirme: yo había dicho que no íbamos a meter cambios de aire, pero cuando vi el proyecto final y tuve una plática con Miriam, dijo: 'es muy importante'."

## Bloque 1 — Infiltración constante (ACH)

### Concepto

Los **cambios de aire por hora** (`ACH` = `Air Changes per Hour`) describen cuántas veces se renueva el volumen de aire de un espacio en una hora. **No es flujo másico** — un cuarto chico con el mismo ACH que uno grande tiene flujo másico menor. Detalle en [[../concepts/Infiltracion-Cambios-Aire]].

> "Los cambios de aire por hora son una unidad rara: si tenemos dos espacios con diferente volumen, el flujo másico no es el mismo."

Energy Plus aplica una **ecuación única** (siempre) a la infiltración:

$$
Q_{inf} = Q_{diseño} \cdot S(t) \cdot \left[ A + B\,|\Delta T| + C\,v + D\,v^2 \right]
$$

donde:

- `Q_diseño` — valor de diseño (en este caso ACH constante).
- `S(t)` — schedule fraccional 0-1 que modula en el tiempo.
- `A + B·|ΔT| + C·v + D·v²` — términos de **flotabilidad** (`B·|ΔT|`) y **viento** (`C·v + D·v²`).
- Para **constante**: A=1, B=C=D=0.

> "Es muy interesante porque desarrolla un módulo que define la infiltración total y ese módulo es enriquecido por diferentes parámetros. Si se ponen a diseñar software, esta idea de tener algo que controle todo y reciba parámetros de todos lados es super interesante."

### Por qué es una caricatura ([[../concepts/Caricatura-Computacional]])

- ACH **constante** = "un ventilador prendido bien calibrado que sabe cuánto aire mete".
- E+ asume [[../concepts/Mezclado-Perfecto|mezclado instantáneo]] — el aire entra y se mezcla con todo el cuarto en el timestep.
- En la realidad hay **jets direccionales** (la corriente que sale por una puerta hacia una ventana). E+ no resuelve eso sin un módulo CFD externo.

> "Ahorita que se cerró la puerta yo sentía una corriente que llegaba a mí y se salía por la ventana. Es un jet con dirección clara, no necesariamente se mezcla."

### Procedimiento en Open Studio (7 pasos)

Detalle completo en [[../procedures/Agregar-Infiltracion-OpenStudio]]. Resumen:

1. **Crear Schedule** tipo `Fractional` (0-1) con valor 1 si la infiltración es constante. Renombrar (ej. `ventilación nocturna`).
2. **Crear Space Type** (ej. `cuarto ventilado`) para servir de contenedor de cargas.
3. En `Loads → Library`, arrastrar **`Space Infiltration Design Flow Rate`** al Space Type. **Open Studio no permite crearlo desde cero**, hay que arrastrar uno existente y editarlo.
4. **Editar el objeto**: cambiar el método a `Air Changes per Hour`, poner valor (ej. 0.5), dejar `Constant Coefficient = 1`, `Temperature Coefficient = 0`, `Velocity Coefficient = 0`, `Velocity Squared Coefficient = 0`.
5. **Asignar el Schedule** al Space Type → Loads → la fila de infiltración.
6. **Asignar el Space Type a los espacios** que se quiere ventilar (vía `Spaces → Properties`).
7. **Verificar en el RDD** que aparezca `Zone Infiltration Standard Density Volume Flow Rate` (u otras de la familia `Infiltration ...`). Solicitar al output y correr.

### Bug confesional del profesor

Puso `Schedule value = 2` con un `Fractional` schedule (cuyo rango es 0-1) → la simulación falló con error de validación. El error de E+ era descriptivo. Lección: ir **paso a paso** y revisar tras cada cambio.

> "Que no les pase lo que me pasó: le quise poner un 2 y la fracción va entre 0 y 1. Pero los errores siempre son descriptivos."

### Verificación en Python

Cargar el SQL y graficar la variable `Zone Infiltration Standard Density Volume Flow Rate` (o ACH). Debe ser **casi constante** (varía en el tercer decimal por cambios de densidad del aire con la temperatura).

```python
from iertools.read import read_sql
import matplotlib.pyplot as plt
from dateutil.parser import parse
import pandas as pd

base = read_sql("../osm/008_air_changes_per_hour/run/eplusout.sql", alias=True).data

fig, ax = plt.subplots(figsize=(12, 3))
f1 = parse("2006-01-01")
f2 = f1 + pd.Timedelta(days=2)
ax.plot(base["Zone Infiltration Standard Density Volume Flow Rate"])
ax.set_xlim(f1, f2)
ax.set_ylabel("ACH (1/h)")
```

### Ventilación nocturna como estrategia bioclimática (pendiente Miriam)

Una variante del schedule (constante de día = 0, alto de noche = 1) **podría** funcionar como estrategia bioclimática:

- **Climas cálidos**: ventilar de noche para enfriar masa térmica.
- **Climas fríos**: ventilar en horas pico (2-4 pm) para introducir calor.

El profesor lo va a consultar con Miriam antes de liberarlo como estrategia válida del proyecto.

> "Si hace mucho calor, puedes ventilar en la noche y si tienes masa térmica eso va a enfriar la casa. En climas fríos puedes ventilar en los momentos de máxima temperatura."

## Bloque 2 — Asistente del curso, migración a Mac Mini

### Lo que cambió

| Antes | Ahora |
|---|---|
| Bot corriendo en Raspberry **en casa del profesor** | Bot en **Mac Mini en el IER** |
| Aprobaciones manuales solo cuando el profesor estaba en casa | Profesor puede aprobar desde cualquier lado |
| Solo Claude (API externa) | Va a migrar a **LLM local Llama ~27B** corriendo en la Mac Mini |

> "Si estoy aquí en el instituto, me puedo conectar a esa máquina. Pero también tenemos estrategias de que yo me puedo conectar desde mi casa."

### Estado actual del corpus

Conteos que da el bot al preguntarle:

- 48 conceptos
- 5 herramientas
- 7 libretas
- **Falta la clase 013** (DDH) — error que el profesor reconoce.

### "Minar errores" oficial — premio en puntos

El periodo de pruebas se vuelve formal:

- Los estudiantes pueden **interactuar con el bot** y reportar errores.
- Cada error reportado **vale puntos** del curso.
- Hay que hacerlo cuando el profesor lo libere formalmente — está afinando la captura de reportes en la nueva infraestructura.

> "Ahorita no lo hagan porque todavía no lo tengo bien terminado, pero podrían empezar a platicar con él y cuando les diga, reportarlo."

### Tensión ética / privacidad

> "Los mensajes no están cifrados. Yo puedo escuchar los puertos y ver las conversaciones en tiempo real. ¿Quién tiene tiempo para ponerse a verlo? Yo no. Pero técnicamente se puede."

- No hay registro intencional de información personal.
- Estos sistemas son **hackeables**; el profesor está abierto a que los estudiantes intenten romperlo.
- Si recabar info pasara a tener un objetivo de investigación → consentimiento + comité de ética.
- Recomendación: no poner nombres ni datos sensibles en el chat con el bot.

### Tensión pedagógica — abierto

> "¿Y si alguien puede entrar y pasar la clase con el asistente nada más preguntándole? Te da hasta el código… ¿está chido o no? Yo tengo un conflicto."

El asistente actual responde con código completo cuando se le pide. El profesor anticipa **modos socráticos**: el asistente pregunta de vuelta antes de contestar, o limita interacciones para forzar preguntas más precisas. **Sigue siendo problema abierto** — invita propuestas.

Detalle ampliado en [[../concepts/Asistente-Virtual-RAG]].

## Bloque 3 — Floorspace con imagen del plano

### Cambio en el enunciado del proyecto: Casa 1 → Casa 3

Importante:

> "Cambié la casa porque la casa que dije que iba a estar disponible resulta que ya no está. En la página de Decide y Construye ya no está el PDF que era la **Casa 1**. Lo cambiamos a la **Casa 3**, hice unos pequeños cambios y subí el proyecto."

El PDF actualizado está en Classroom. La metodología es idéntica — solo cambia la edificación de referencia. Sigue siendo **vivienda progresiva** (3 etapas), todas comparten la planta baja. Para el proyecto se simula la etapa 3 (la casa terminada). Detalle en [[../concepts/Caso-Base]].

### Truco: plano como imagen de fondo

Workflow para reconstruir la casa rápido en FloorspaceJS:

1. Tomar **screenshot del plano** del PDF (Shift+Cmd+4 en Mac).
2. En Open Studio: `Editor → New Floor Plan` → ícono `Image`.
3. Cargar el screenshot — aparece como capa de fondo.
4. **Calibrar el grid** con un lado de medida conocida. Si la fachada del plano mide **7 m**, poner grid = 1 m y mover/escalar la imagen hasta que la fachada cubra 7 cuadritos.
5. **Dibujar zonas térmicas encima** — los espacios se ajustan al grid pero el dibujo es libre.
6. Para la **planta alta**: crear un segundo piso y repetir. **Bug**: la imagen del piso 1 no persiste al cambiar de piso — hay que cargar el plano de la segunda planta como nueva imagen.

> "Tener una referencia es como cuando hacen una calibración digital con rejillas — basta una distancia."

Procedimiento detallado en [[../procedures/Importar-Plano-FloorspaceJS]].

### Decisiones de zonificación

Para la Casa 3 (3 zonas térmicas por planta en el demo):

- Planta baja: estancia (incluye baño), cocina, recámara.
- Planta alta: 4 zonas (recámaras + estancia).
- **Total: ~7 zonas térmicas**.

> "Cada uno de ustedes va a tener una casa ligeramente diferente porque dónde toman el muro, qué tanto refinan el espacio, va a hacer que todas las casas sean distintas."

### Convención de nombrado: `E` o `S` de Space

> "Acuérdense que el espacio y las zonas térmicas no se pueden llamar de la misma manera."

Sufijo `_E` o `_S` para distinguir el Space del ThermalZone:

- Space: `recamara_E` (o `recamara_S`)
- ThermalZone: `recamara` (auto-generada o renombrada)

Detalle en [[../concepts/Espacio-vs-ZonaTermica]].

### Limitaciones de FloorspaceJS

> "FloorspaceJS es educativo porque es gratuito. Yo esperaría que en un análisis serio usen SketchUp o Revit."

Limitaciones recurrentes:

- Bug al cambiar de piso pierde la imagen.
- No es robusto para ventanas/puertas interiores entre zonas.
- A veces falla en escala/movimiento — requiere cerrar y abrir el plugin.

### Tip operativo nuevo — Classic CLI

> "Si le activo `Classic` (Classic Command Line Interface), me da una versión minimalista. Está como más limpio para leer el proceso de E+, sin los plugins de Ruby."

`Show Simulation → Classic CLI` produce log de E+ más fácil de leer (sin envoltura Ruby). Útil para debugging.

## Bloque 4 — Window LBNL (introducción + instalación)

### Por qué necesario

Energy Plus tiene **dos modelos** de ventanas (detalle en [[../concepts/Ventanas]] y [[../concepts/Solar-Heat-Gain-Coefficient]]):

| Modelo | Inputs | Cuándo usar |
|---|---|---|
| **Simple Glazing System** | `SHGC` + `U-value` + `Visible Transmittance` | Cuando se conocen los certificados de la ventana |
| **Complex Fenestration** (Window:Construction) | Capa por capa: vidrios, gases, low-E, propiedades ópticas por ángulo | Caracterización detallada / vidrios no estándar |

**Problema**: el modelo Complex requiere propiedades que normalmente **no tenemos** (reflectancias y absortancias a diferentes ángulos, transmitancia visible vs solar separadas, emisividades en IR). El modelo Simple requiere solo 3 números pero hay que **calcularlos** primero.

**Solución**: usar **Window LBNL** — un programa especializado que resuelve la transferencia de calor multi-capa y devuelve el SHGC y U que se pegan al modelo Simple de E+.

### Sobre Window LBNL

- Desarrollado por **Lawrence Berkeley National Laboratory** (no NREL — el profesor se corrige).
- **Solo corre en Windows**. En Mac: Parallels o similar.
- Estado estacionario (no dinámico). Suposiciones internas estándar.
- **Versión actual recomendada: 7.8** — **no usar betas**.
- Detalle en [[../tools/Window-LBNL]].

### Por qué SHGC importa más que U

$$
SHGC = \frac{\text{Flujo de calor que entra por la ventana}}{\text{Radiación solar incidente}}
$$

Comparado con la resistencia térmica (que solo considera conducción), el SHGC captura la **componente radiativa** — que en climas cálidos domina el flujo de calor a través de la ventana.

> "El solar heat gain coefficient es mejor que la resistencia térmica porque la resistencia térmica solo es por conducción. Este está volteando a ver la parte que proviene de la radiación."

### Anti-patrón comercial — películas que "absorben 80% del calor"

> "La gente erróneamente compra esas películas porque alarman que absorba 80% del calor. No es lo deseable. Lo va a absorber, se va a calentar y lo va a emitir."

El razonamiento erróneo: "si la película absorbe el calor del sol, no entra al cuarto". El razonamiento correcto:

1. La película **absorbe** la radiación solar.
2. La película **se calienta**.
3. Como toda superficie caliente, **emite radiación infrarroja** — hacia afuera y **hacia adentro**.
4. El cuarto recibe la radiación IR de la película + cualquier flujo conductivo.

**Lo que se quiere**: ventanas **reflejantes** o con **baja emisividad** (low-E) — ahí el calor se devuelve hacia afuera, no se reemite hacia el interior. Las superficies low-E **normalmente van en ventanas dobles** porque la película low-E se protege entre los dos vidrios.

> "Si yo pongo un termómetro [en un jardín con acrílico] no se va a ver, porque eso se ve en la temperatura radiante. El AC tampoco ve ese efecto — ve la temperatura de bulbo seco."

Ver [[../concepts/Temperatura-Operativa]] y [[../concepts/Emisividad]].

### Veredicto: ventanas dobles en climas cálidos sin AC

> "Ventanas dobles en edificaciones sin aire acondicionado es tirar el dinero. Aumentan la resistencia térmica, pero aquí lo que queremos es ventilar."

| Situación | Veredicto ventanas dobles |
|---|---|
| Clima cálido **sin AC** | **Casi inútiles** — la mejora es marginal y el costo alto. Mejor ventilar + pintar de color claro |
| Clima cálido **con AC** | **Sí valen** — impiden entrada de calor radiativo y conductivo, mantienen el frío del AC |
| Clima templado/frío | **A veces valen** — reducen pérdida nocturna, pero también reducen ganancia solar diurna |

Aplicación al proyecto final: cada equipo puede usar Window para **evaluar** si una estrategia de doble vidrio mejora o no su caso, y reportar la decisión.

### Cadena de instalación (Windows)

> "Window es bien lata. Hasta le doy dos vueltas, porque a veces no leo las instrucciones bien."

Hay que instalar dos dependencias **antes** del Window Setup:

1. **Microsoft Visual C++ Redistributable** (versión 14, arquitectura **x86** — la liga del LBNL apunta directo a esta).
2. **Intel Fortran Compiler Runtime** for Windows (`IFX_2023.1` o equivalente).
3. **Window 7.8 Full Setup**.

Si se instala Window primero, da error `DLL not found` al correr. Detalle paso a paso en [[../procedures/Instalar-Window-LBNL]].

> "Acepta todo, instala, finish."

## Pendientes para clase 015 (29 mayo, última clase)

| Pendiente | Origen |
|---|---|
| Cómo usar Window LBNL para construir un sistema de ventanas | Clase 014 (solo se instaló) |
| Cálculo de la **banda de Morillón** desde la amplitud de la serie histórica de To | Clase 013 |
| Temperatura **pesada por volumen** — actualizar enunciado del proyecto | Clase 013 |
| Cafecito + cierre del taller | Plan oficial |

Adicionalmente, el profesor mencionó que la clase 015 podría dedicarse a **ventanas o repaso de Floorspace** según necesidad del grupo. Decisión final el día 29.

## Conexiones

- ← **Anterior:** [[013-CalculoGradosHoraDisconfort]]
- → **Siguiente:** _(clase final 29 de mayo — cierre del taller, cafecito + uso de Window LBNL)_
- → Conceptos nuevos:
  - [[../concepts/Infiltracion-Cambios-Aire]]
  - [[../concepts/Solar-Heat-Gain-Coefficient]]
- → Conceptos actualizados:
  - [[../concepts/Caricatura-Computacional]] — mezclado instantáneo de infiltración, ACH constante
  - [[../concepts/Asistente-Virtual-RAG]] — migración Mac Mini, premio por reportar errores
  - [[../concepts/Caso-Base]] — Casa 1 → Casa 3
  - [[../concepts/Ventanas]] — SHGC/U y modelos simple/complex
  - [[../concepts/Schedules]] — schedule fraccional 0-1
  - [[../concepts/Espacio-vs-ZonaTermica]] — sufijo `E`/`S`
- → Herramienta nueva: [[../tools/Window-LBNL]]
- → Procedimientos nuevos:
  - [[../procedures/Agregar-Infiltracion-OpenStudio]]
  - [[../procedures/Importar-Plano-FloorspaceJS]]
  - [[../procedures/Instalar-Window-LBNL]]
  - [[../procedures/Usar-Window-LBNL]]

## Anécdotas

- **Fallecimiento de Jesús** — la clase empieza con la mención de un funeral del IER. El profesor toma 10 minutos para asistir. Reflexión sobre "hacer comunidad" contra el colonialismo, ligada al feminismo y a comunidades como Oaxaca/Teposcolula.
- **Cloud cayó la noche anterior** — el profesor estaba trabajando a las 11 pm y se quedó sin asistente. Refuerza el argumento para **modelos locales** (Llama 27B en la Mac Mini).
- **Cubículos del IER con frío** — observación práctica: el salón no baja de 29 °C en la noche porque **no abren ventanas**. Ventilación nocturna funcionaría obviamente, pero las ventilas no están automatizadas.
- **"Películas que absorben 80% del calor"** — venden esto en Saint-Gobain, 3M, y similares. Marketing que aprovecha el desconocimiento físico. El profesor lo flagea como **fraude técnico** (no necesariamente intencional).
