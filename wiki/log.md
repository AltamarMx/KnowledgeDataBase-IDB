---
title: Log de la Wiki IDB
type: log
updated: 2026-05-09
---

# Log cronológico

Registro de todas las ingestas y mantenimientos de la wiki.

## 2026-05-02

### Ingesta: clase 001

- **Fuente:** `raw/videos/001_Intro_Taller.md` → [[classes/001-IntroduccionTallerIDB]]
- **Creados (estructura inicial):**
  - `wiki/index.md`, `wiki/log.md`, `wiki/REGLAS_CURSO.md`
  - Conceptos: `Simulacion-Energetica`, `Balance-de-Calor`, `Envolvente-Arquitectonica`, `Sistemas-Constructivos`, `Condiciones-de-Frontera`, `Confort-Termico`
  - Herramientas: `Open-Studio`, `EnergyPlus`, `Python`
  - Procedimientos: `Instalar-Open-Studio`

### Ingesta: clase 002

- **Fuente:** `raw/videos/002_ConceptosBasicos.md` → [[classes/002-ConceptosBasicosBalancesCalor]]
- **Creados:**
  - Conceptos nuevos: `Zona-Termica`, `Factor-de-Vista`, `Absortancia-Solar`, `Emisividad`, `Masa-Termica`, `TMY`, `Radiacion-Onda-Larga`
- **Actualizados:**
  - `Balance-de-Calor` — agregadas ecuaciones del balance dependiente del tiempo y del balance en superficie exterior con sus 3 componentes
  - `Sistemas-Constructivos` — convención exterior→interior, restricción de homogeneidad, referencia a `InternalMass`
  - `Envolvente-Arquitectonica` — restricciones de E+ (líneas rectas, flujo 1D)
  - `Condiciones-de-Frontera` — condición de frontera externa típica (radiación + convección)
  - `tools/EnergyPlus.md` — sección de módulos (CTF, FD, Window, Shading, Sky, Day Lighting, Air HB, Airflow Network, etc.), restricciones, time steps, EPW/TMY
  - `REGLAS_CURSO.md` — sección "Asistente IA del curso (RAG)" con NotebookLM, Gems, motivación de las grabaciones, proyecto de máquina propia
  - `index.md` — agregadas clase 002 y 7 conceptos nuevos

### Ingesta: clase 003

- **Fuente:** `raw/videos/003_MiPrimeraSimulacion.md` → [[classes/003-MiPrimeraSimulacion]]
- **Creados:**
  - Conceptos nuevos: `Mezclado-Perfecto`, `Espacio-vs-ZonaTermica`, `Caricatura-Computacional`, `Tipos-Superficie`, `Subsuperficie`, `Radiacion-Interior-Distribuida`
  - Procedimientos nuevos: `Estructura-Proyecto-Simulacion`, `Descargar-EPW-OneBuilding`, `Crear-Primera-Simulacion-OpenStudio`
- **Actualizados:**
  - `Balance-de-Calor` — agregado el balance en superficie interior (3 componentes) y el balance de aire de la zona térmica con mezclado perfecto
  - `Condiciones-de-Frontera` — agregado el catálogo de colores Open Studio (Outdoor azul, Surface verde, Ground café, Adiabatic rojo); cómo el editor convierte automáticamente Outdoor→Surface al unir físicamente espacios; trampa de paredes "casi pegadas"; pisos adiabáticos en multi-story
  - `Zona-Termica` — referencias a `Mezclado-Perfecto` y `Espacio-vs-ZonaTermica`
  - `Factor-de-Vista` — nota sobre LWR interior (no atraviesa vidrios), factores de vista nulos por geometría
  - `Sistemas-Constructivos` — flujo concreto Materials → Constructions → Surface en Open Studio, no usar `No Mass Materials`
  - `tools/Open-Studio.md` — secciones nuevas: hashes anti-plagio, folder hermano del OSM, FloorspaceJS, Render By, trampa de paredes casi pegadas, catálogo de pestañas
  - `tools/EnergyPlus.md` — restricciones extendidas (mezclado perfecto, radiación interior distribuida, sub-superficies, tipos de superficie, caricatura)
  - `index.md` — clase 003, 6 conceptos nuevos, 3 procedimientos nuevos

### Ingesta: clase 004

- **Fuente:** `raw/videos/004_InterpretandoMensajesSimulacionesConstructionSets.md` → [[classes/004-InterpretandoMensajesConstructionSets]]
- **Creados:**
  - Conceptos nuevos: `Warm-up-Period`, `Mensajes-EnergyPlus`, `Construction-Set`, `Measures`, `Site-Source-Factor`, `Calculo-Sombramientos`, `Salida-SQL-EnergyPlus`
  - Procedimientos nuevos: `Leer-Archivo-ERR`, `Configurar-Construction-Set`, `Debuggear-Simulacion-OpenStudio`
- **Actualizados:**
  - `Masa-Termica` — particiones interiores y mobiliario como `InternalMass`; metáfora de las casas vacías frías
  - `Sistemas-Constructivos` — referencia a Construction Sets como atajo
  - `Condiciones-de-Frontera` — Sun y Wind Exposure como dimensiones independientes; caso del estacionamiento subterráneo
  - `tools/Open-Studio.md` — pestañas Facility y Measures, flujo OSM→IDF, Show Simulation, Construction Set en pestaña Construction
  - `tools/EnergyPlus.md` — secciones nuevas: Warm-up, Shadow Update, Mensajes y debugging, Salidas (SQL/CSV/HTML/ERR)
  - `tools/Python.md` — paquete del profesor para leer SQL directo a pandas; flujo de uso por etapas; volumen típico de datos
  - `procedures/Estructura-Proyecto-Simulacion.md` — refuerzo: nunca mover el OSM al folder hermano
  - `procedures/Crear-Primera-Simulacion-OpenStudio.md` — referencias a Construction Sets como atajo y a procedimientos de debugging
  - `index.md` — clase 004, 7 conceptos nuevos, 3 procedimientos nuevos

### Ingesta: clase 005

- **Fuente:** `raw/videos/005_PrimerAnalisisSimulacionesCoPython.md` → [[classes/005-AnalisisSimulacionesPython]]
- **Creados:**
  - Conceptos nuevos: `RDD-Variables-Disponibles`, `Variables-Output-EnergyPlus`, `Temperatura-Operativa`, `Capa-Limite-Atmosferica`, `Confort-Adaptativo`
  - Herramienta nueva: `tools/iertools.md` (paquete del grupo IER)
  - Procedimientos nuevos: `Setup-Entorno-Python-uv`, `Solicitar-Output-Variables-Measures`, `Analizar-Resultados-Python`, `EDA-Archivo-EPW`
- **Actualizados:**
  - `Mezclado-Perfecto` — variable `Zone Mean Air Temperature` como output asociado
  - `Confort-Termico` — modelos PMV vs adaptativo, métricas (% confort, grados-hora)
  - `Salida-SQL-EnergyPlus` — confirmado el paquete `iertools`; problemas concretos del CSV (`24:00`, numeración de folders); alias
  - `Mensajes-EnergyPlus` — archivos hermanos del `.err` (`.rdd`, `.mdd`, `.sql`, `.csv`, `.htm`, `.eio`)
  - `Measures` — caso concreto de Reporting Measures (`Add Output Variable`, `Create CSV Output`)
  - `tools/Open-Studio.md` — secciones BCL, pedir variables al output (vías), `eplusout.rdd` en Show Simulation
  - `tools/EnergyPlus.md` — RDD/MDD/EIO, pedir variables al output, capa límite atmosférica
  - `tools/Python.md` — stack ampliado, setup uv detallado, patrones útiles (subplots, dateutil + Timedelta, resample), `iertools`
  - `procedures/Crear-Primera-Simulacion-OpenStudio.md` — referencia a Solicitar-Output-Variables-Measures
  - `procedures/Debuggear-Simulacion-OpenStudio.md` — verificar variables solicitadas en RDD/CSV/SQL
  - `index.md` — clase 005, 5 conceptos nuevos, 1 herramienta nueva, 4 procedimientos nuevos

### Ingesta: clase 006

- **Fuente:** `raw/videos/006_DosZonasTermicasVentanasAleros.md` → [[classes/006-DosZonasTermicasVentanasAleros]]
- **Creados:**
  - Conceptos nuevos: `Ventanas`, `Superficies-de-Sombramiento` (recreación), `Limpiar-Geometria`
  - Procedimientos nuevos: `Agregar-Ventanas-OpenStudio`, `Agregar-Aleros-OpenStudio`
- **Actualizados:**
  - `Subsuperficie` — Window-to-Wall Ratio (NOM-008), referencia a Ventanas
  - `Mensajes-EnergyPlus` — warnings de `Coliniar vertices` y `Weather location difference`; analogía con compiladores C; política del grupo (cero warnings en investigación)
  - `Sistemas-Constructivos` — materiales de ventana (Glazing vs Simple Glazing), relación ρ-k, pinturas e impermeabilizantes despreciables
  - `Construction-Set` — slots Sub Surface (Window/Door/Skylight) detallados
  - `Espacio-vs-ZonaTermica` — alturas heredables/sobreescribibles por Space, corte automático de superficies
  - `Caricatura-Computacional` — caricaturas nuevas (aleros sin transferencia de calor, marcos ignorados, alero del mismo ancho que la ventana)
  - `tools/Open-Studio.md` — Components de ventanas y aleros en FloorspaceJS, limitación del alero, limpieza automática de geometría
  - `tools/EnergyPlus.md` — Materiales de ventana (Glazing, Gas, SimpleGlazingSystem, FrameAndDivider), superficies de sombramiento
  - `procedures/Configurar-Construction-Set.md` — slots Sub Surface (Exterior Window, Interior Window, Door, Glass Door, Skylight)
  - `procedures/Crear-Primera-Simulacion-OpenStudio.md` — referencia a Agregar-Ventanas y Agregar-Aleros
  - `procedures/Analizar-Resultados-Python.md` — patrón "día más cálido" con criterio explícito
  - `index.md` — clase 006, 3 conceptos nuevos, 2 procedimientos nuevos

### Ingesta: clase 007

- **Fuente:** `raw/videos/007CasoBaseAleros.md` → [[classes/007-CasoBaseAleros]]
- **Creados:**
  - Conceptos nuevos: `Caso-Base`, `Estudio-Parametrico`, `Trayectoria-Solar`
  - Procedimiento nuevo: `Comparar-Simulaciones-Python`
- **Actualizados:**
  - `Caricatura-Computacional` — caricatura "rayo de luz local"; anécdota Paloma de validación de alero equivalente <2%
  - `Temperatura-Operativa` — sensores virtuales de confort
  - `Radiacion-Interior-Distribuida` — efecto local sobre ocupantes que la suposición no captura
  - `Variables-Output-EnergyPlus` — distinción "incidente exterior" vs "entrada a la zona"; refinamiento de `Zone Windows Total Heat Gain Rate`
  - `Mensajes-EnergyPlus` — bugs recurrentes (piso adiabático que se revierte, nombres custom borrados tras cambios geométricos)
  - `Limpiar-Geometria` — bug FloorspaceJS al rehacer el modelo
  - `procedures/Estructura-Proyecto-Simulacion.md` — regla `Save As` en lugar de copiar en Explorador; estructura del proyecto final con caso base + variantes
  - `procedures/Analizar-Resultados-Python.md` — list comprehensions, renombrado con diccionario, función de carga reutilizable, bug confesional
  - `procedures/Debuggear-Simulacion-OpenStudio.md` — bugs recurrentes a chequear primero; debugging de simulaciones aparentemente idénticas
  - `procedures/Agregar-Aleros-OpenStudio.md` — anécdota Paloma de validación
  - `index.md` — clase 007, 3 conceptos nuevos, 1 procedimiento nuevo

### Ingesta: clase 008

- **Fuente:** `raw/videos/008_ShadingEnVentanas.md` → [[classes/008-ShadingVentanas]]
- **Creados:**
  - Conceptos nuevos: `Sunlit-Fraction`, `Algoritmo-Sombreamiento`, `Enfriamiento-Radiativo-Cielo`, `Grados-Hora-Disconfort`
  - Procedimiento nuevo: `Auditar-Sombreamiento-Ventanas`
- **Actualizados:**
  - `Variables-Output-EnergyPlus` — `Sunlit Fraction` y `Sunlit Area`; nota crítica sobre sub-superficies (la radiación incidente no refleja sombreamiento en ventanas)
  - `Superficies-de-Sombramiento` — algoritmo y atenuación selectiva (directa via SF, difusa via factores de vista)
  - `Confort-Adaptativo` — métricas de evaluación, anti-patrones; grados-hora como métrica principal
  - `Mensajes-EnergyPlus` — warning "many overlapping shadows"; reflexión 10:2 sobre cuándo culpar a E+
  - `Caricatura-Computacional` — Sunlit Fraction multiplicativa; sombreamiento solo bloquea directa
  - `Radiacion-Onda-Larga` — T del cielo a −15°C como sumidero radiativo
  - `procedures/Solicitar-Output-Variables-Measures.md` — Sunlit Fraction al catálogo
  - `procedures/Comparar-Simulaciones-Python.md` — sospechar variable de radiación en sub-superficies si los casos salen iguales
  - `procedures/Estructura-Proyecto-Simulacion.md` — estructura de libretas Jupyter del proyecto final (4 libretas + unificadora)
  - `procedures/EDA-Archivo-EPW.md` — refinamiento del cálculo de grados-hora con anti-patrón de sumar
  - `index.md` — clase 008, 4 conceptos nuevos, 1 procedimiento nuevo

### Ingesta: clase 009

- **Fuente:** `raw/videos/009_AireAcondicionado_SetPoints.md` → [[classes/009-AireAcondicionadoSetPoints]]
- **Creados:**
  - Conceptos nuevos: `Aire-Acondicionado-Ideal`, `Schedules`, `Setpoint`, `Posicion-Aislante`
  - Procedimientos nuevos: `Crear-Schedule-Temperatura`, `Configurar-Aire-Acondicionado-Ideal`
- **Actualizados:**
  - `Caricatura-Computacional` — Ideal Air Loads como caricatura de HVAC (eficiencia 100%, sin ductos)
  - `Sistemas-Constructivos` — orden importa en modelos dependientes del tiempo; referencia a Posicion-Aislante
  - `Variables-Output-EnergyPlus` — variables de Ideal Air Loads (cooling/heating energy y rate, thermostat setpoint)
  - `Confort-Adaptativo` — setpoint óptimo desde el modelo adaptativo + anécdota Cool Biz
  - `procedures/Solicitar-Output-Variables-Measures.md` — variables de AC al catálogo
  - `procedures/Analizar-Resultados-Python.md` — patrones de resample mensual, gráfica de barras, workaround ylim
  - `index.md` — clase 009, 4 conceptos nuevos, 2 procedimientos nuevos

### Ingesta: clase 010

- **Fuente:** `raw/videos/010_EnerHabitat_Parte1.md` → [[classes/010-EnerHabitatParte1]]
- **Creados:**
  - Conceptos nuevos: `Temperatura-Sol-Aire`, `Estado-Oscilatorio-Permanente`, `Factor-de-Decremento`
  - Herramienta nueva: `tools/EnerHabitat.md` (paquete Python + web app del IER)
  - Procedimientos nuevos: `Usar-EnerHabitat-Web`, `Usar-EnerHabitat-Python`
- **Actualizados:**
  - `Caricatura-Computacional` — caricaturas de EnerHabitat (un solo muro, día representativo, T sol-aire encapsulada)
  - `Posicion-Aislante` — EnerHabitat como herramienta primaria de evaluación
  - `Confort-Adaptativo` — uso de Humphreys-Nicol en EnerHabitat para zona de confort
  - `Warm-up-Period` — comparación con el oscilatorio permanente de EnerHabitat
  - `procedures/Setup-Entorno-Python-uv.md` — `enerhabitat` como paquete opcional
  - `index.md` — clase 010, 3 conceptos nuevos, 1 herramienta nueva, 2 procedimientos nuevos

### Ingesta: clase 011

- **Fuente:** `raw/videos/011_EnerHabitat_Parte2.md` → [[classes/011-EnerHabitatParte2]]
- **Creados:**
  - Concepto nuevo: `Asistente-Virtual-RAG` (asistente del curso con OpenCode + Claude + Telegram)
- **Actualizados:**
  - `Caricatura-Computacional` — detalles del cuarto idealizado de EnerHabitat (2.5 m, 200 elementos, h_c específicos de NOM-008/020)
  - `Temperatura-Sol-Aire` — "susto feliz" del primer time step (radiación = 0); asociación T_sa ↔ wall específico (anti-patrón de pegar T_sa entre walls distintos)
  - `Factor-de-Decremento` — uso en estudios paramétricos en Python
  - `procedures/Usar-EnerHabitat-Python.md` — API verificada en 0.1.9 (fix del bug pandas 3.0); `solve()` vs `solve_ac()`; `materials.ini` auto-detectado; `config.h0` y bug de configuración global; patrón de loop paramétrico; anti-patrones (`layers` vs `absorptance`, sobreexplicar resultados raros)
  - `procedures/Analizar-Resultados-Python.md` — anti-patrones Python (referencias compartidas, NumPy vs DataFrame, Numba, fragilidad Jupyter)
  - `index.md` — clase 011, 1 concepto nuevo, fin del taller

### Ingesta: notebooks 001 + 002 + corrección masiva del paquete

- **Fuentes:**
  - `raw/notebooks/001_EDA.ipynb` → [[notebooks/001_EDA]]
  - `raw/notebooks/002_EDA_EPW.ipynb` → [[notebooks/002_EDA_EPW]]
- **Creados:**
  - Sección `wiki/notebooks/` con páginas para 001 y 002.
- **Hallazgo crítico — corrección de nombre del paquete:**
  - El paquete real es **`iertools`** (de "IER tools"), no `ear_tools` ni `ear-tools`.
  - Las primeras transcripciones automáticas de las clases 005-011 capturaron mal la pronunciación.
  - **Renombrado**: `wiki/tools/ear-tools.md` → `wiki/tools/iertools.md`. Contenido reescrito.
  - **Reemplazo masivo** en 22 archivos: `ear_tools` / `ear-tools` → `iertools` (preservando aliases en el archivo principal).
- **Correcciones a la API verificada en los notebooks:**
  - `read_epw` **NO tiene parámetro `year`** — el reemplazo de año es manual con `.index.map(lambda x: x.replace(year=YYYY))`.
  - El **workaround del 29-feb es responsabilidad del usuario**, no del paquete (filtrar antes del reemplazo si el año destino no es bisiesto).
  - Índice del DataFrame de `read_epw` se llama `tiempo` (no `date`).
  - Alias del EPW: solo renombra catálogo conocido (`To`, `RH`, `Ib`, `Id`, `Ig`, `WS`, `WD`, `P`); las 22+ columnas restantes conservan su nombre EPW original.
  - `read_sql` con `alias=True` produce columnas como `Ti_<zona>` (Title Case), `To`, `Ib`, `Id` — confirmado en notebook 001.
  - Método `get_construction()` (singular), no plural.
- **Actualizados:**
  - `procedures/EDA-Archivo-EPW.md` — flujo corregido sin `year=2006` ni `suppress_warnings=True`; workaround manual del 29-feb explícito; nota sobre alias parcial.
  - `index.md` — sección "Libretas Jupyter procesadas" con 001 y 002.
- **Pendiente futuro** (no bloqueante):
  - ~25 menciones de aliases con casing antiguo (`df.TO`, `df.IB`, `df.ID`, `T_cubo`, `T_este`) en `procedures/Analizar-Resultados-Python.md`, `procedures/Configurar-Aire-Acondicionado-Ideal.md`, `tools/Python.md`, `classes/005-AnalisisSimulacionesPython.md`, `classes/006-DosZonasTermicasVentanasAleros.md`. La capitalización real es `To`, `Ib`, `Id`, `Ti_cubo`, `Ti_este`. Corregir en próxima sesión.

### Ingesta: notebook 003

- **Fuente:** `raw/notebooks/003_EDA.ipynb` → [[notebooks/003_EDA]]
- **Naturaleza:** EDA del **caso base** del proyecto final (clase 007) con 2 zonas térmicas (ESTE, OESTE).
- **Hallazgos para integrar:**
  - **Frecuencias mezcladas** en output variables crean NaNs (~99% de la columna). Solo el último valor de cada intervalo horario se llena. Causa: `Zone Air Temperature` y `Zone Air Relative Humidity` solicitadas a `Hourly` mientras otras a `Timestep`.
  - **Alias preserva mayúsculas** del nombre de zona en el OSM: `Ti_ESTE` (no `Ti_este`). Si la zona se nombró en mayúsculas en Open Studio, el alias también.
  - **`Zone Mean Air Temperature`** sí recibe alias automático (`Ti_<zona>`); **`Zone Air Temperature`** (sin "Mean") **no** lo recibe — conserva nombre completo.
  - **Patrón del día más cálido** confirmado: `df.To.resample("D").mean().idxmax()`.
- **Creados:**
  - `wiki/notebooks/003_EDA.md`
- **Actualizados:**
  - `concepts/Variables-Output-EnergyPlus.md` — sección "Frecuencias mezcladas — antipatrón" y "`Zone Mean Air Temperature` vs `Zone Air Temperature`".
  - `index.md` — agregar 003 a notebooks.

### Ingesta: notebook 004

- **Fuente:** `raw/notebooks/004_Comparacion_ConSinVentanas.ipynb` → [[notebooks/004_Comparacion_ConSinVentanas]]
- **Naturaleza:** Comparación caso base (`005_CBs`) vs con protecciones (`006_Protecciones`). Aplica el flujo de auditoría de sombreamiento de la clase 008.
- **Hallazgos:**
  - **`Mir-FACE` (mirror surfaces)**: superficies espejo internas que E+ crea para el cálculo de overlapping cuando hay shading. Aparecen como columnas en el output (`Mir-FACE 8`, `Mir-FACE 18`, etc.) cuando se usa `*` como Key Value. **No están en el OSM**. El caso con protecciones tenía 31 columnas vs ~10 en el caso base, mostrando la explosión.
  - **Patrón color + marker** como alternativa al color + linestyle (`"g-"` para base, `"go"` para variante). Útil cuando las dos series están cercanas.
  - **Vía alternativa de auditoría**: pedir radiación incidente sobre el **muro padre** (no la ventana) — en muros opacos sí refleja el sombreamiento.
  - Confirmación del patrón Sunlit Fraction de la clase 008 con datos reales.
- **Creados:**
  - `wiki/notebooks/004_Comparacion_ConSinVentanas.md`
- **Actualizados:**
  - `concepts/Algoritmo-Sombreamiento.md` — sección "Mirror surfaces (`Mir-FACE`)" con explicación, cuándo aparecen, qué hacer.
  - `procedures/Auditar-Sombreamiento-Ventanas.md` — vía alternativa pidiendo radiación sobre muro padre; tabla SF vs muro padre; cuándo elegir cada vía.
  - `procedures/Solicitar-Output-Variables-Measures.md` — advertencia sobre frecuencias mezcladas; advertencia sobre `*` que genera Mir-FACE.
  - `procedures/Comparar-Simulaciones-Python.md` — patrón color + marker como variante del color + linestyle.
  - `index.md` — agregar 004 a notebooks.

### Ingesta: notebook 005

- **Fuente:** `raw/notebooks/005_revision_1setpoint.ipynb` → [[notebooks/005_revision_1setpoint]]
- **Naturaleza:** Caso `007_CB_aa` (caso base + Ideal Air Loads modo T constante 20 °C). Verificación de la simulación + análisis energético.
- **Hallazgos:**
  - **Patrón filtro de año**: `df = df[df.index.year == AÑO]` post-carga elimina el timestep extra de `2007-01-01 00:00:00` (cierre del año en E+).
  - **Variables de Ideal Air Loads sin alias** — `iertools` no las renombra; nombre largo (`ESTE IDEAL LOADS AIR SYSTEM:Zone Ideal Loads Zone Total Cooling Energy (J)`).
  - **OESTE = 4× ESTE en consumo** (23 GJ/año vs 5.8 GJ/año) — confirma desbalance por orientación.
  - Bug menor: `range(13)` en lugar de `range(12)` en bar plot mensual.
- **Creados:**
  - `wiki/notebooks/005_revision_1setpoint.md`
- **Actualizados:**
  - `index.md` — agregar 005 a notebooks.

### Ingesta: notebook 006

- **Fuente:** `raw/notebooks/006_Adobe_con_sin_AC.ipynb` → [[notebooks/006_Adobe_con_sin_AC]]
- **Naturaleza:** Estudio paramétrico con EnerHabitat (clase 011 — la libreta que falló en vivo y ahora corre con el fix de pandas 3.0). Adobe en Campeche con/sin AC, variando absortancia.
- **Hallazgos críticos — API real de EnerHabitat verificada:**
  - **`enerhabitat.System(enerhabitat.Location(epw_file))`** — clase real es `System`, no `Wall`.
  - **`wall.absortance`** (sin "p") — typo del paquete. Calco del español "absortancia". `absorptance` no existe.
  - **`wall.location.meanDay(month, year)`** — método de location, no atributo de wall. No existe `set_day()`.
  - **`wall.Tsa()`** (T mayúscula) — no `tsa()`.
  - **`wall.solveAC()`** (camelCase) — no `solve_ac()`.
  - **Columnas del output**: `Ti` (interior — no `T_int`), **`Ta`** (ambiente — no `To`), `Tsa`, `Tn`, `DeltaTn`, `Is`, `Ig`, `Ib`, `Id`, `zenith`, `elevation`, `azimuth`, `equation_of_time`.
  - **`DeltaTn = 1.25 °C`** en Campeche → **modelo de Morillón**, no Humphreys-Nicol/ASHRAE 55. Variable según oscilación local del clima.
  - **`cooling_energy` y `heating_energy` son escalares** (J/(m²·día)) — no series temporales. No usar `.sum()`.
  - **Concatenar Tsa al output**: el solver no lo incluye por default; `pd.concat([result, wall.Tsa().asfreq("10min")], axis=1)`.
- **Creados:**
  - `wiki/notebooks/006_Adobe_con_sin_AC.md`
- **Actualizados (correcciones de API):**
  - `procedures/Usar-EnerHabitat-Python.md` — código completo corregido con `System`, `absortance`, `location.meanDay`, `Tsa`, `solveAC`. Tabla de diferencias entre transcripción de clase y API real. Estudio paramétrico corregido. Recomendación de crear nuevo `wall` por iteración.
  - `concepts/Confort-Adaptativo.md` — modelo de Morillón añadido como entrada en la tabla de modelos adaptativos. Sección dedicada explicando que `DeltaTn` es variable según oscilación local (1.25-4 °C), no fijo en 3.5 °C.
  - `index.md` — agregar 006 a notebooks (cierra la lista de 6 notebooks del taller).

## 2026-05-08

### Ingesta: clase 012 — Proyecto Final

- **Fuente:** `raw/videos/012_ProyectoFinal.md` → [[classes/012-ProyectoFinal]]
- **Naturaleza:** Clase atípica — encuadre logístico + metodológico del proyecto final del semestre 2026-2. No introduce conceptos nuevos de simulación; consolida los anteriores y fija reglas de entrega.
- **Hechos clave registrados:**
  - **Casa 11** del programa **Decide y Construye** (vivienda social MX, 60-65 m², dos plantas) como edificación de referencia.
  - **Caso base fijo**: absortancia 0.4 en todas las superficies, sin AC, sin sombreado, sin cargas térmicas, piso adiabático, ventanas vidrio simple 3 mm, sub-superficies interiores no se simulan.
  - **Workflow**: caso base + 3 estrategias bioclimáticas (que **mejoren**) + 1 caso integrado = 5 simulaciones.
  - **Meses críticos** vía CONUEE (no análisis anual).
  - **Métricas**: promedio mensual del máx/mín diario + grados-hora cálidos/fríos.
  - **Presentación 5 jun 2026 a las 10 AM**: 15 min + 10 min preguntas. Reporte 5 págs máx, Google Doc preferido. Total 250 puntos.
  - **Onboarding nuevo del bot**: screenshot del mensaje del bot → tarea en Classroom. Privacidad — sin contacto directo profesor↔alumno.
  - **Pregunta abierta**: cómo hacer un asistente IA pedagógico (no "barco") — el profesor pide propuestas al grupo.
- **Creados:**
  - `wiki/classes/012-ProyectoFinal.md`
- **Actualizados:**
  - `concepts/Caso-Base.md` — sección "Caso base del proyecto final 2026-2" con la especificación tabulada (α=0.4, piso adiabático, etc.).
  - `concepts/Estudio-Parametrico.md` — sección "Encuadre del proyecto final 2026-2" (etiquetas por nombre, mejora obligatoria, no automatizar, mes crítico, priorización en climas extremosos).
  - `concepts/Asistente-Virtual-RAG.md` — sección "Onboarding por screenshot" + falla por calor de la Raspberry + pregunta abierta sobre pedagogía del asistente.
  - `concepts/Grados-Hora-Disconfort.md` — sección "Aplicación al proyecto final 2026-2" (mes crítico, matriz caso × mes × estrategia).
  - `REGLAS_CURSO.md` — sección nueva "Proyecto final 2026-2" (fechas, formato, evaluación 250 puntos).
  - `procedures/Estructura-Proyecto-Simulacion.md` — sub-sección "Entrega del proyecto final 2026-2" (zip del workspace, una persona por equipo, Google Doc preferido).
  - `index.md` — fila 012 agregada a la tabla de clases.
- **Decisiones del usuario:**
  - Las páginas opcionales (`Definir-Mes-Critico-CONUEE`, `Presentar-Proyecto-Final`) **no se crean** — quedan como secciones dentro de la clase 012.
  - Nombre exacto del bot **no se registra en la wiki** — la nota dice "ver Classroom".
- **Pendientes flagged:**
  - Procedimiento de **infiltración** — el profesor lo grabará en video de 15 min o lo verá el 22 de mayo. Se ingerirá cuando aparezca el material.
  - **Materiales de ventanas complejas** (sistema equivalente) — pendiente para 22 mayo.
  - **Rúbrica detallada (250 puntos)** — el profesor la entregará ~13 de mayo. Cuando llegue, actualizar `REGLAS_CURSO.md` y la página 012.

## 2026-05-09

### Ingesta: notebook 007 — Cálculo de Grados-Hora de Disconfort

- **Fuente:** `raw/notebooks/007_DDH.ipynb` → [[notebooks/007_DDH]]
- **Naturaleza:** Implementación completa del cálculo de GH cálidos/fríos sobre el caso `004_dos_zonas` (modelo de la clase 006). Cubre carga, derivación de Tn mensual con Humphreys-Nicol, banda Morillón=1.25, integración con dt=10/60 h, y plot por banda con tres colores.
- **Resultado del cálculo:** GHDC = 6,884.5 °C·h ; GHDF = 5,001.4 °C·h. Clima de doble extremo (cálido + frío significativos).
- **Hallazgos para integrar:**
  - **`banda = 1.25` → modelo de Morillón** (no Humphreys-Nicol). Decisión deliberada documentada — afecta la magnitud de GH; banda estrecha → GH altos. Se debe mantener consistente al comparar entre simulaciones.
  - **Patrón limpio `groupby(index.month).mean()`** — alternativa a `resample("ME").mean()` cuando se quiere agregar por número de mes para mapear de vuelta a la serie.
  - **Patrón limpio `pd.Series(index.month.map(Tn_m), index=df.index)`** — broadcast de Series mensual a serie temporal completa; reemplaza el lambda `df.index.to_series().map(lambda t: ...)` usado antes.
  - **`.clip(lower=0)`** — alternativa pandas-native a `np.maximum(..., 0)` para grados-hora.
  - **Antipatrón nuevo: `plot()` con índice booleano** conecta puntos no adyacentes con líneas falsas. Soluciones: `Ti.where(mask)` (inserta NaN, rompe línea) o `scatter` (sin línea).
  - **Antipatrón complementario**: olvidar `ax.legend()` después de poner `label="..."`.
  - **Confirmaciones de patrones previos:**
    - `Zone Mean Air Temperature` se aliasa a `Ti_<ZONA>`; `Zone Air Temperature` (sin "Mean") no — confirma [[notebooks/003_EDA]].
    - Frecuencias mezcladas (Hourly + Timestep) → 99% NaN — confirma [[notebooks/003_EDA]].
    - Cierre `2007-01-01 00:00:00` presente — confirma [[notebooks/005_revision_1setpoint]].
- **Creados:**
  - `wiki/notebooks/007_DDH.md`
- **Actualizados:**
  - `concepts/Grados-Hora-Disconfort.md` — snippet de Python reescrito con patrones idiomáticos pandas (`groupby(index.month)`, `index.month.map`, `.clip(lower=0)`); advertencia explícita sobre `banda` no ser siempre 3.5 (puede ser Morillón 1.25-4); cross-link a [[notebooks/007_DDH]].
  - `procedures/Analizar-Resultados-Python.md` — sección nueva "Plot con índice booleano conecta puntos no adyacentes (anti-patrón)" con soluciones `.where()` y `scatter`; entradas en la tabla de trampas comunes.
  - `index.md` — fila 007 en la tabla de notebooks.
- **Decisiones del usuario:**
  - Banda 1.25 confirmada como elección de Morillón.
  - `VNORTE`/`VOESTE` confirmadas como ventana norte/oeste; `TECHO` como techo.
  - `004_dos_zonas` confirmado como el modelo de la clase 006.
- **Pendientes flagged:**
  - Análogo para `Ti_OESTE` (la libreta sólo procesa ESTE).
  - Reportar GH **por mes**, no sólo anual — alinear con encuadre del proyecto final (clase 012).
  - Aplicar filtro de año `df[df.index.year == 2006]` antes del cálculo.
  - Comparar contra caso con protecciones usando el patrón de [[notebooks/004_Comparacion_ConSinVentanas]].
