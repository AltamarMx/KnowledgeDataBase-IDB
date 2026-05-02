---
title: 005 — Primer Análisis de Simulaciones con Python
type: clase
clase: 005
profesor: Guillermo Barrios del Valle
fuente: raw/videos/005_PrimerAnalisisSimulacionesCoPython.md
fecha_ingesta: 2026-05-02
tags: [clase, python, ear-tools, sql, rdd, measures, confort-adaptativo, eda]
aliases: [Clase 005]
---

# 005 — Primer Análisis de Simulaciones con Python

## Metadatos

- **Clase:** 005
- **Profesor:** Guillermo Barrios del Valle
- **Fuente:** `raw/videos/005_PrimerAnalisisSimulacionesCoPython.md`
- **Tipo:** Clase práctica — postprocesamiento de simulaciones con Python

## Resumen

Primera clase enfocada en **análisis de datos**. Se justifica por qué el análisis es obligatorio (los reportes nativos de Open Studio están enfocados a normativas extranjeras de ahorro de energía con AC; el caso bioclimático sin AC requiere análisis propio).

La clase recorre tres bloques en orden:

1. **Pedir las variables correctas a la simulación** — leer el archivo RDD para descubrir qué hay disponible, copiar nombres exactos, configurar measures `Add Output Variable` y `Create CSV Output` del BCL.
2. **Cargar los resultados en Python** — `ear_tools.read_sql` lee el SQL nativo (más robusto que el CSV), aplica alias cortos, expone `.data` (DataFrame) y permite auditar sistemas constructivos.
3. **Análisis y plotting** — gráfica de doble panel con `plt.subplots(2, 1, sharex=True)`, recorte temporal con `dateutil.parser.parse + pd.Timedelta`, análisis del EPW con `read_epw`, introducción a confort adaptativo (Humphreys-Nicol).

Hilo transversal: **narrativa computacional y reproducibilidad** — uv, ambientes virtuales, naming consistente, restart-and-run-all, versionado.

Tarea: **dos zonas térmicas** (este y oeste, sin ventanas todavía), con output variables solicitadas y análisis en Python.

## Objetivos de aprendizaje

- Saber leer el archivo **RDD** para descubrir variables disponibles. Ver [[../concepts/RDD-Variables-Disponibles]].
- Conocer el **catálogo de variables** de salida más usadas (Site, Zone, Surface inside/outside). Ver [[../concepts/Variables-Output-EnergyPlus]].
- Configurar measures **`Add Output Variable`** y **`Create CSV Output`** para pedir variables específicas. Ver [[../procedures/Solicitar-Output-Variables-Measures]].
- Configurar un entorno Python reproducible con **uv** + dependencias del taller. Ver [[../procedures/Setup-Entorno-Python-uv]].
- Cargar el SQL de E+ con **`ear_tools.read_sql`** y producir gráficas básicas. Ver [[../procedures/Analizar-Resultados-Python]].
- Hacer un **EDA del EPW** y calcular zona de confort adaptativo. Ver [[../procedures/EDA-Archivo-EPW]].
- Entender el concepto de **temperatura operativa** y por qué importa cuando hay fuentes radiantes. Ver [[../concepts/Temperatura-Operativa]].

## Por qué hacer análisis de datos propio

> "La parte fácil es hacer una simulación con aire acondicionado, porque tengo que disminuir energía. Pero cuando no hay aire acondicionado — que es nuestra prioridad en el grupo y debería ser la prioridad en México — se vuelve más complicado, porque tenemos una serie de tiempo."

Razones que el profesor enfatiza:

- **Open Studio tiene templates de reporte**, pero están enfocados a **normativas extranjeras** (ASHRAE, LEED) basadas en **ahorro con AC**.
- El caso **sin AC** (bioclimático, ventilación natural, masa térmica) requiere métricas distintas: % del año en confort, grados-hora de disconfort, comparaciones temperatura interior vs exterior.
- El análisis también permite **detectar errores** que la simulación no marca como severo (gráficas con comportamiento implausible, propiedades térmicas incorrectas).

## Bloque 1 — Pedir variables a la simulación

### El archivo RDD

`eplusout.rdd` (Report Data Dictionary) lista **todas las variables que esta simulación puede reportar**. Detalle en [[../concepts/RDD-Variables-Disponibles]].

Estructura de cada línea:

```
Output:Variable,*,<nombre exacto>,<frecuencia>; !- <unidades>
```

> "El RDD es el archivo de diccionarios de variables que están disponibles para esta simulación específica."

**Cómo descubrir variables**:

1. Tras `Run`, **Show Simulation** → carpeta `run/` → abrir `eplusout.rdd` con Notepad/TextEdit.
2. Buscar (`Ctrl+F`) por keyword.
3. Copiar el nombre exacto.

### Variables clave que se piden en la clase

| Variable | Para qué |
|----------|----------|
| `Site Outdoor Air Drybulb Temperature` | T exterior — referencia obligatoria |
| `Site Direct Solar Radiation Rate per Area` | Radiación directa |
| `Site Diffuse Solar Radiation Rate per Area` | Radiación difusa |
| `Surface Outside Face Incident Solar Radiation Rate per Area` (sobre el techo) | Truco para tener radiación global |
| `Zone Mean Air Temperature` | T del aire interior (mezclado perfecto) |
| `Zone Operative Temperature` | T operativa — cuando hay fuentes radiantes |

> **Truco para la radiación global**: E+ no tiene una variable `Site Global Solar Radiation`. Pero `Surface Outside Face Incident Solar Radiation` sobre el **techo** (superficie horizontal sin sombreamiento) da exactamente eso.

Catálogo completo en [[../concepts/Variables-Output-EnergyPlus]].

### Por qué importa la temperatura operativa

> "Si pongo un sensor de temperatura de aire, no se va a dar cuenta. Aquí donde hay grandes niveles de radiación, la temperatura radiante y por lo tanto la temperatura operativa van a ser muy importantes."

Casos donde $T_{op} \neq T_{aire}$:

- Sol incidente directo sobre alguien parado.
- Pared muy caliente o muy fría enfrente.
- Plancha de concreto que reflejó sol todo el día.

Detalle en [[../concepts/Temperatura-Operativa]].

### Capa límite atmosférica

E+ ajusta la T del EPW (medida a 10 m) a la altura del centroide de la zona térmica antes de calcular la convección. Por eso en el RDD hay variables `Site:*` (clima crudo) y variables `Outdoor:*` (ajustadas a la altura del objeto). Detalle en [[../concepts/Capa-Limite-Atmosferica]].

### Asterisco como wildcard

`*` como Key Value devuelve la variable para todas las superficies/zonas. Cuidado con explosión: un edificio de 200 superficies × `Surface Outside Face Temperature *` = 200 columnas con ~52,560 puntos cada una.

> Recomendación: dar **nombres descriptivos** a las superficies clave (`Techo`, `MuroNorte`) y pedir la variable para esos nombres.

### Configuración de measures

La pestaña **Output Variables** de Open Studio expone solo un subconjunto. Para acceso completo se usan measures del BCL:

1. **`Add Output Variable`** — uno por cada variable que se quiera (Reporting → QAQC en el BCL).
2. **`Create CSV Output`** — uno solo, para verificar visualmente.

Configuración:

- **Variable Name** (E+) → pegar nombre exacto del RDD.
- **Reporting Frequency** → `Timestep` (cada 10 min en el default).
- **Key Value** → `*` o nombre específico.

Procedimiento detallado en [[../procedures/Solicitar-Output-Variables-Measures]].

## Bloque 2 — Cargar resultados en Python

### Setup del entorno con uv

> "Yo me moví a ambientes virtuales el día que perdí 3 días tratando de arreglar mi Mac y descubrí que tenía 7 versiones de Python instaladas."

Stack del taller:

```bash
cd ~/.../tarea_02_dos_zonas/
uv init
uv add pandas jupyter matplotlib python-dateutil
uv add git+https://github.com/<grupo-IER>/ear-tools.git
uv run jupyter notebook
```

Reglas críticas:

- **Nunca `pip install`** dentro de notebooks o terminal — siempre `uv add`.
- **Nunca `python script.py`** — siempre `uv run script.py`.
- `.venv/` no se sube al repo (se regenera con `uv sync`).

Procedimiento completo en [[../procedures/Setup-Entorno-Python-uv]].

### Por qué uv (y no pip / poetry / conda solo)

- **Velocidad**: escrito en Rust, ~10× más rápido que pip.
- **Reproducibilidad**: `pyproject.toml` + `uv.lock` funcionan igual en Mac/Win/Linux.
- **Aislamiento**: cada proyecto tiene su `.venv`.
- **Sin sufrir**: si algo se rompe, `rm -rf .venv && uv sync` lo arregla en segundos.

> "Linux ya no te deja instalar cosas a menos que sean en ambientes virtuales. uv es lo mejor que hay."

### El paquete `ear_tools`

Paquete del grupo de Energía en Edificaciones del IER. Detalle en [[../tools/ear-tools]].

```python
from ear_tools.read import read_sql, read_epw
```

#### `read_sql(file, alias=True)`

Lee `eplusout.sql`. Devuelve un objeto con:

- `.data` — DataFrame con datetime como índice y todas las variables solicitadas.
- `.construction_systems` — lista de nombres de constructions.
- `.get_constructions(nombres)` — detalle de cada construction (capas, propiedades térmicas).

#### Por qué SQL y no CSV

Dos razones específicas:

1. **El CSV pone `24:00:00`** para el último paso del día — `pandas.to_datetime` no parsea eso.
2. **Los reporting measures numeran sus folders dinámicamente** (`004_addOutputVariable`, `005_addOutputVariable`) — agregar/quitar un measure cambia los paths y rompe libretas. El SQL siempre está en `run/eplusout.sql` (estable).

> "Me cansé y luego me di cuenta que lo podíamos hacer desde el SQL e implementar algunas cositas."

Detalle en [[../concepts/Salida-SQL-EnergyPlus]].

#### Convención de alias

Con `alias=True`, las columnas se renombran a versiones cortas:

| Original | Alias |
|----------|-------|
| `Zone Mean Air Temperature` (zona X) | `T_<X>` |
| `Site Outdoor Air Drybulb Temperature` | `TO` |
| `Site Direct Solar Radiation` | `IB` (beam) |
| `Site Diffuse Solar Radiation` | `ID` (diffuse) |
| Global solar | `IG` |
| Wind Speed / Direction | `WS` / `WD` |
| Relative Humidity | `RH` |

Ventaja: acceso por punto-atributo (`df.T_cubo`) en lugar de `df["CUBO:Zone Mean Air Temperature [C]"]`.

#### Auditoría de constructions

> "En el grupo solemos tener: 'Memo, ahí está la simulación, revísala.' Lo primero que hago es verificar que las propiedades estén bien."

```python
sc = cubo.construction_systems
cubo.get_constructions(sc)
```

Devuelve nombre, espesor, capas, conductividad, densidad, calor específico, absortancias, rugosidad. Sin tener que abrir el OSM.

## Bloque 3 — Plotting y análisis

### Gráfica de doble panel

```python
fig, ax = plt.subplots(2, 1, sharex=True, figsize=(12, 4))

ax[0].plot(df.T_cubo, label="T_cubo")
ax[0].plot(df.TO,     label="TO")
ax[0].legend()

ax[1].plot(df.ID, label="diffuse")
ax[1].plot(df.IB, label="beam")
ax[1].legend()
```

> "**No hagan figuras con dobles ejes Y**. Son horrorosas, nos toma 10 minutos entenderlas. Para un paper está bien; para una presentación, no."

`sharex=True` evita repetir las fechas en ambos paneles y mantiene zoom sincronizado.

### Recorte temporal con dateutil + Timedelta

```python
from dateutil.parser import parse

f1 = parse("2006-03-13")
f2 = f1 + pd.Timedelta(days=7, hours=3, minutes=50)

ax[1].set_xlim(f1, f2)
```

Patrón potente: cambiar de "una semana" a "12 horas" o "3 días con 8 minutos" es trivial sin pelearse con suma de fechas a mano.

### El año `2006`

E+ pone `2006` por default en todos los timestamps (independiente del año real del EPW, que es un TMY). En el RunPeriod del IDF se puede cambiar. En el alcance del curso, `2006` es solo una etiqueta — los datos siguen siendo válidos.

### Análisis del EPW

```python
from ear_tools.read import read_epw

epw = read_epw("../EPW/cuernavaca.epw", alias=True, year=2006, suppress_warnings=True)
```

`year=2006` reemplaza el año en cada timestamp para evitar saltos del Frankenstein TMY (cada mes viene de un año real distinto).

#### Bug — años bisiestos

Si el EPW base tiene un mes de un año bisiesto con el 29-feb, `year=2006` puede fallar. Workaround: filtrar el 29-feb antes de aplicar el cambio. El profesor debugó esto en vivo con ayuda de IA. Detalle en [[../procedures/EDA-Archivo-EPW]].

> "Yo creo que ya están convencidos que Python y Pandas es la opción. La parte difícil es aprender a manejar los datetime — pero es la base de todo."

#### Visualización rápida — subplots por columna

```python
epw.plot(subplots=True, figsize=(12, 20))
```

Genera un panel por variable. **No documentado oficialmente** pero funciona ("un huevo de Pascua en pandas").

### Confort adaptativo — Humphreys-Nicol

Modelo central para evaluar diseño bioclimático sin AC:

$$
T_{neutralidad} = 0.54 \cdot \overline{T}_{out,mes} + 13.5
$$

donde $\overline{T}_{out,mes}$ es la T promedio mensual del exterior.

Zona de confort = $T_{neutralidad} \pm \Delta T$ con $\Delta T \approx 3.5$ °C (80% aceptabilidad, ASHRAE 55 adaptativo).

```python
T_mes = epw.TO.resample("ME").mean()      # ME = Month-End
T_neut = 0.54 * T_mes + 13.5
delta = 3.5
T_conf_min = T_neut - delta
T_conf_max = T_neut + delta
```

> "Resampleo mensual punto mean. Y eso es la temperatura promedio mensual. Si tengo eso, ya tengo la temperatura de neutralidad. Y entonces puedo determinar el delta T y mi zona de confort."

Detalle en [[../concepts/Confort-Adaptativo]] y procedimiento en [[../procedures/EDA-Archivo-EPW]].

#### Métricas — grados-hora de disconfort

$$
GH_{cálido} = \sum_t \max\left( T_{op}(t) - (T_{neut,mes} + \Delta T),\ 0 \right) \cdot \Delta t
$$

(análogo para frío). Métrica que el grupo del IER usa para comparar estrategias bioclimáticas.

### Climate Consultant

GUI alternativa para análisis adaptativo del clima. Pros: cubre varios modelos. Cons (según el profesor): "las gráficas son horrorosas, juntan demasiada información, no se pueden modificar". Por eso el curso hace todo en Python.

## Narrativa computacional — recordatorios

> "Aquí es donde hay que practicar el desapego."

### Restart and Run All

Cada cierto tiempo: **Kernel → Restart and Run All**. Si la libreta corre limpia, está reproducible. Si falla, hay variables fantasma o orden no-lineal — corregir.

### Naming consistente

Folders sin acentos/eñes/espacios — el autocompletado de Tab en Jupyter falla con caracteres especiales. Refuerzo de la regla de [[../procedures/Estructura-Proyecto-Simulacion]].

### Versionado de OSMs

Numerar OSMs (`001_volumetria.osm`, `002_dosZonas.osm`, …) para regresar a la última versión que funcionaba sin perder horas debugueando.

### Setup → ramificación

Cuando el modelo está estable como **caso base**, ramificar en variantes (`007_color`, `008_alero`, `009_orientacion`) — todas heredan del caso base. Permite estudios paramétricos limpios.

## Tarea de la semana

> **Dos zonas térmicas**: una hacia el **este**, otra hacia el **oeste**. Sin ventanas todavía (las ventanas se ven en clase 006).
>
> Pedir output variables a la simulación (ver [[../procedures/Solicitar-Output-Variables-Measures]]) y producir las gráficas básicas con Python (ver [[../procedures/Analizar-Resultados-Python]]).

Recomendaciones:

- Nombrar las zonas `Este` y `Oeste` (o `Derecha` e `Izquierda`) — algo descriptivo, no `Thermal Zone 1`.
- Leer el RDD para confirmar que las variables se generaron.
- Hacer al menos: T interior de cada zona vs T exterior + radiación incidente.

Cálculo de volumen:

- 2 zonas × paso de 10 min × 365 días = ~105,000 puntos de T.
- Múltiples variables → cientos de miles de filas → pandas obligado.

## Conceptos derivados

Conceptos nuevos:

- [[../concepts/RDD-Variables-Disponibles]]
- [[../concepts/Variables-Output-EnergyPlus]]
- [[../concepts/Temperatura-Operativa]]
- [[../concepts/Capa-Limite-Atmosferica]]
- [[../concepts/Confort-Adaptativo]]

Conceptos profundizados:

- [[../concepts/Salida-SQL-EnergyPlus]] — confirmado el paquete `ear_tools` y los problemas concretos del CSV
- [[../concepts/Mezclado-Perfecto]] — variable `Zone Mean Air Temperature`
- [[../concepts/Confort-Termico]] — modelo adaptativo
- [[../concepts/Measures]] — caso concreto de Reporting Measures

Herramienta nueva:

- [[../tools/ear-tools]] — paquete del grupo

## Conexiones

- ← **Anterior:** [[004-InterpretandoMensajesConstructionSets]] — cómo correr y debuggear simulaciones
- → **Siguiente:** _006-DosZonasTermicasVentanasAleros_ — agregar ventanas y aleros, multi-zona
- → Procedimientos clave:
  - [[../procedures/Solicitar-Output-Variables-Measures]]
  - [[../procedures/Setup-Entorno-Python-uv]]
  - [[../procedures/Analizar-Resultados-Python]]
  - [[../procedures/EDA-Archivo-EPW]]

## Recursos mencionados

- **Climate Consultant** — alternativa GUI para análisis del EPW; útil para sugerencias adaptativas.
- **Hackatón del profesor** — hay una clase de 2 horas con un segmento de ~20 min sobre instalar Miniconda + uv. El profesor planea compartir el video.
- **Cookiecutter** — herramienta de Python para inicializar estructuras de proyecto desde plantillas. El profesor usa templates de GitHub directamente.
- **Quarto** — alternativa moderna a Jupyter para libretas reproducibles. Convive con uv.
- **NREL BCL** — repositorio de measures de Open Studio (donde están `Add Output Variable` y `Create CSV Output`).
