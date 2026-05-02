---
title: Log de la Wiki IDB
type: log
updated: 2026-05-02
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
  - Herramienta nueva: `tools/ear-tools.md` (paquete del grupo IER)
  - Procedimientos nuevos: `Setup-Entorno-Python-uv`, `Solicitar-Output-Variables-Measures`, `Analizar-Resultados-Python`, `EDA-Archivo-EPW`
- **Actualizados:**
  - `Mezclado-Perfecto` — variable `Zone Mean Air Temperature` como output asociado
  - `Confort-Termico` — modelos PMV vs adaptativo, métricas (% confort, grados-hora)
  - `Salida-SQL-EnergyPlus` — confirmado el paquete `ear_tools`; problemas concretos del CSV (`24:00`, numeración de folders); alias
  - `Mensajes-EnergyPlus` — archivos hermanos del `.err` (`.rdd`, `.mdd`, `.sql`, `.csv`, `.htm`, `.eio`)
  - `Measures` — caso concreto de Reporting Measures (`Add Output Variable`, `Create CSV Output`)
  - `tools/Open-Studio.md` — secciones BCL, pedir variables al output (vías), `eplusout.rdd` en Show Simulation
  - `tools/EnergyPlus.md` — RDD/MDD/EIO, pedir variables al output, capa límite atmosférica
  - `tools/Python.md` — stack ampliado, setup uv detallado, patrones útiles (subplots, dateutil + Timedelta, resample), `ear_tools`
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
