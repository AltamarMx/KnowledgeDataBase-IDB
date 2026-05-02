---
title: Índice de la Wiki IDB
type: index
updated: 2026-05-02
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
| 005 | [[classes/005-AnalisisSimulacionesPython]] | RDD y catálogo de variables, T operativa, capa límite, measures de output, paquete `ear_tools`, setup uv, plotting con matplotlib, EDA del EPW, confort adaptativo |
| 006 | [[classes/006-DosZonasTermicasVentanasAleros]] | Dos zonas con alturas distintas, limpieza de geometría, ventanas (sub-superficies, materiales Glazing/SimpleGlazing, marcos), aleros y parteluces, aleros equivalentes, día más cálido |
| 007 | [[classes/007-CasoBaseAleros]] | Caso base + variantes (estudio paramétrico), workflow del proyecto final, comparación en Python (función de carga, renombrado custom, plot color/estilo), trayectoria solar, bugs recurrentes |

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

## Herramientas

- [[tools/Open-Studio]]
- [[tools/EnergyPlus]]
- [[tools/Python]]
- [[tools/ear-tools]]

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

## Libretas Jupyter procesadas

_(ninguna aún)_

## Log

Ver [[log]] para el historial cronológico de ingestas y mantenimiento.
