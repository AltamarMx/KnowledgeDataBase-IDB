---
title: Índice de la Wiki IDB
type: index
updated: 2026-05-22
---

# Wiki — Taller de Diseño Bioclimático (IDB)

Catálogo del contenido procesado. Cada entrada se actualiza cuando se ingiere una clase o libreta.

## Reglas y operación

- [[REGLAS_CURSO]] — Comunicación, equipos, evaluación, política de violencia, uso de IA, asistente IA del curso (RAG), asistencia

## Clases

| # | Página | Temas |
|---|--------|-------|
| 001 | [[classes/001-IntroduccionTallerIDB]] | Presentación, objetivo, herramientas, simplificaciones del modelo, instalación de Open Studio |
| 002 | [[classes/002-ConceptosBasicosBalancesCalor]] | Zona térmica, modelo dependiente del tiempo, módulos de Energy Plus, balance en superficie exterior, EPW/TMY |
| 003 | [[classes/003-MiPrimeraSimulacion]] | Balance interior, balance de aire, mezclado perfecto, primer modelo en Open Studio (FloorspaceJS, EPW, materiales, condiciones de frontera, Run) |
| 004 | [[classes/004-InterpretandoMensajesConstructionSets]] | Flujo OSM→IDF, lectura del `.err`, errores vs warnings, Construction Sets, Warm-up Period, Shadow Update, salidas SQL/CSV/HTML, Site/Source factors |
| 005 | [[classes/005-AnalisisSimulacionesPython]] | RDD y catálogo de variables, T operativa, capa límite, measures de output, paquete `iertools`, setup uv, plotting con matplotlib, EDA del EPW, confort adaptativo |
| 006 | [[classes/006-DosZonasTermicasVentanasAleros]] | Dos zonas con alturas distintas, limpieza de geometría, ventanas (sub-superficies, materiales Glazing/SimpleGlazing, marcos), aleros y parteluces, aleros equivalentes, día más cálido |
| 007 | [[classes/007-CasoBaseAleros]] | Caso base + variantes (estudio paramétrico), workflow del proyecto final, comparación en Python, trayectoria solar, bugs recurrentes |
| 008 | [[classes/008-ShadingVentanas]] | Sunlit Fraction y resolución del bug de la 007, algoritmo de sombreamiento, enfriamiento radiativo al cielo, grados-hora de disconfort, estructura de libretas |
| 009 | [[classes/009-AireAcondicionadoSetPoints]] | Aire acondicionado ideal, schedules, setpoints (T constante / banda / hackeado), posición del aislante, crítica a NOM y Cool Biz Japón |
| 010 | [[classes/010-EnerHabitatParte1]] | Herramienta EnerHabitat (web + paquete Python), temperatura sol-aire, estado oscilatorio permanente, factor de decremento, demo con bug |
| 011 | [[classes/011-EnerHabitatParte2]] | Asistente virtual del curso (RAG + Telegram), fix del bug pandas 3.0, estudio paramétrico en Python, anti-patrones (referencias compartidas, NumPy vs DataFrame), tarea final |
| 012 | [[classes/012-ProyectoFinal]] | Encuadre del proyecto final 2026-2: Casa 11 de Decide y Construye, especificación del caso base (α=0.4, sin AC, piso adiabático), CONUEE, 5 simulaciones, métricas por mes crítico, presentación 5 jun, onboarding del bot por screenshot |
| 013 | [[classes/013-CalculoGradosHoraDisconfort]] | Implementación en pandas del cálculo de grados-hora cálidos/fríos: Humphreys-Nicol mensual, banda de Morillón, `.clip(lower=0)`, máscaras de tres colores; hallazgo Chilpancingo (Tn constante en climas estables); promedio pesado por volumen; crítica al uso indebido de simulaciones (Design Builder en Oaxaca) |

## Conceptos

- [[concepts/Simulacion-Energetica]]
- [[concepts/Balance-de-Calor]]
- [[concepts/Zona-Termica]]
- [[concepts/Envolvente-Arquitectonica]]
- [[concepts/Sistemas-Constructivos]]
- [[concepts/Condiciones-de-Frontera]]
- [[concepts/Confort-Termico]]
- [[concepts/Confort-Adaptativo]]
- [[concepts/Masa-Termica]]
- [[concepts/Factor-de-Vista]]
- [[concepts/Absortancia-Solar]]
- [[concepts/Emisividad]]
- [[concepts/Radiacion-Onda-Larga]]
- [[concepts/TMY]]
- [[concepts/Mezclado-Perfecto]]
- [[concepts/Espacio-vs-ZonaTermica]]
- [[concepts/Caricatura-Computacional]]
- [[concepts/Tipos-Superficie]]
- [[concepts/Subsuperficie]]
- [[concepts/Radiacion-Interior-Distribuida]]
- [[concepts/Warm-up-Period]]
- [[concepts/Mensajes-EnergyPlus]]
- [[concepts/Construction-Set]]
- [[concepts/Measures]]
- [[concepts/Site-Source-Factor]]
- [[concepts/Calculo-Sombramientos]]
- [[concepts/Salida-SQL-EnergyPlus]]
- [[concepts/RDD-Variables-Disponibles]]
- [[concepts/Variables-Output-EnergyPlus]]
- [[concepts/Temperatura-Operativa]]
- [[concepts/Capa-Limite-Atmosferica]]
- [[concepts/Ventanas]]
- [[concepts/Superficies-de-Sombramiento]]
- [[concepts/Limpiar-Geometria]]
- [[concepts/Caso-Base]]
- [[concepts/Estudio-Parametrico]]
- [[concepts/Trayectoria-Solar]]
- [[concepts/Sunlit-Fraction]]
- [[concepts/Algoritmo-Sombreamiento]]
- [[concepts/Enfriamiento-Radiativo-Cielo]]
- [[concepts/Grados-Hora-Disconfort]]
- [[concepts/Aire-Acondicionado-Ideal]]
- [[concepts/Schedules]]
- [[concepts/Setpoint]]
- [[concepts/Posicion-Aislante]]
- [[concepts/Temperatura-Sol-Aire]]
- [[concepts/Estado-Oscilatorio-Permanente]]
- [[concepts/Factor-de-Decremento]]
- [[concepts/Asistente-Virtual-RAG]]

## Herramientas

- [[tools/Open-Studio]]
- [[tools/EnergyPlus]]
- [[tools/Python]]
- [[tools/iertools]]
- [[tools/EnerHabitat]]

## Procedimientos

- [[procedures/Instalar-Open-Studio]]
- [[procedures/Estructura-Proyecto-Simulacion]]
- [[procedures/Descargar-EPW-OneBuilding]]
- [[procedures/Crear-Primera-Simulacion-OpenStudio]]
- [[procedures/Leer-Archivo-ERR]]
- [[procedures/Configurar-Construction-Set]]
- [[procedures/Debuggear-Simulacion-OpenStudio]]
- [[procedures/Solicitar-Output-Variables-Measures]]
- [[procedures/Setup-Entorno-Python-uv]]
- [[procedures/Analizar-Resultados-Python]]
- [[procedures/EDA-Archivo-EPW]]
- [[procedures/Agregar-Ventanas-OpenStudio]]
- [[procedures/Agregar-Aleros-OpenStudio]]
- [[procedures/Comparar-Simulaciones-Python]]
- [[procedures/Auditar-Sombreamiento-Ventanas]]
- [[procedures/Crear-Schedule-Temperatura]]
- [[procedures/Configurar-Aire-Acondicionado-Ideal]]
- [[procedures/Usar-EnerHabitat-Web]]
- [[procedures/Usar-EnerHabitat-Python]]

## Libretas Jupyter procesadas

| # | Página | Tema |
|---|--------|------|
| 001 | [[notebooks/001_EDA]] | Primera demo de `iertools.read_sql`: cargar SQL, auditar constructions, plot doble panel |
| 002 | [[notebooks/002_EDA_EPW]] | `iertools.read_epw`: workaround manual del 29-feb, exploración con `subplots=True`, resample mensual |
| 003 | [[notebooks/003_EDA]] | EDA del caso base con 2 zonas térmicas (ESTE, OESTE); patrón del día más cálido; antipatrón de frecuencias mezcladas que crean NaNs |
| 004 | [[notebooks/004_Comparacion_ConSinVentanas]] | Comparación caso base vs con protecciones (clase 008); función reusable de carga; Sunlit Fraction; muro padre como referencia; `Mir-FACE` (mirror surfaces internas de E+) |
| 005 | [[notebooks/005_revision_1setpoint]] | Caso `007_CB_aa` con AC en T constante (clase 009); filtro de año `index.year==2006`; análisis energético mensual y anual; OESTE consume 4× más cooling que ESTE |
| 006 | [[notebooks/006_Adobe_con_sin_AC]] | Estudio paramétrico con EnerHabitat; **API real verificada** (`System`, `absortance`, `meanDay`, `Tsa`, `solveAC`); Adobe en Campeche; delta Morillón 1.25°C |
| 007 | [[notebooks/007_DDH]] | Cálculo completo de GH cálidos/fríos sobre `004_dos_zonas` con banda Morillón=1.25; patrones `groupby(index.month)`, `index.month.map`, `.clip(lower=0)`; antipatrón de `plot()` con índice booleano |

## Log

Ver [[log]] para el historial cronológico de ingestas y mantenimiento.
