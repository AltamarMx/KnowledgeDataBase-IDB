---
title: 001 — Introducción al Taller IDB
type: clase
clase: 001
profesor: Guillermo Barrios del Valle
fuente: raw/videos/001_Intro_Taller.md
fecha_ingesta: 2026-05-02
tags: [clase, introduccion, taller, openstudio, energyplus]
aliases: [Clase 001, Intro Taller IDB]
---

# 001 — Introducción al Taller IDB

## Metadatos

- **Clase:** 001
- **Título:** Introducción al Taller de Diseño Bioclimático
- **Profesores:** Guillermo Barrios del Valle (taller) + Miriam (teoría)
- **Tipo:** Clase introductoria — sin ejercicios, sin física aún
- **Temas principales:** presentación del curso, objetivo, herramientas, alcance del modelo, simplificaciones, instalación de Open Studio

## Resumen

Primera clase del taller. Define el **objetivo del curso**, presenta las **herramientas** (Open Studio + Energy Plus + Python/Jupyter), explica el **alcance del modelo** y sus simplificaciones (qué SÍ y qué NO se va a simular), recorre el ecosistema de software de simulación energética (incluyendo alternativas de paga como Design Builder y Rhino+LadyBug), y deja como tarea la **instalación de Open Studio**.

También se establecen las reglas operativas del curso (recogidas aparte en [[REGLAS_CURSO]]).

## Objetivo del taller

> **Evaluar estrategias bioclimáticas cuantificando su impacto en edificaciones usando simulaciones numéricas.**

El "desempeño térmico" se desglosa en dos ejes:

- **Consumo de energía** — iluminación, calefacción, aire acondicionado.
- **Confort térmico** — el grupo enfatiza este eje porque es el problema invisibilizado de la vivienda social en México: el disconfort no se traduce fácil a emisiones, lo que hace que se pierda del radar en políticas públicas y métricas de sostenibilidad.

El proyecto final consistirá en una casa en un clima determinado donde el equipo propone estrategias para mejorar su desempeño térmico.

## Conceptos introducidos

- [[../concepts/Simulacion-Energetica]] — usar software para resolver el balance de calor dependiente del tiempo a través de la envolvente.
- [[../concepts/Envolvente-Arquitectonica]] — geometría de la edificación; cada superficie se vincula a un sistema constructivo.
- [[../concepts/Sistemas-Constructivos]] — secuencia ordenada de materiales con propiedades térmicas y espesor, asignada a una superficie.
- [[../concepts/Condiciones-de-Frontera]] — tipos: temperatura, flujo de calor (constante, variable o cero ⇒ adiabática), convectiva.
- [[../concepts/Confort-Termico]] — confort vs. disconfort; por qué se invisibiliza.
- [[../concepts/Balance-de-Calor]] — qué resuelve Energy Plus a través de la envolvente (introducción; se profundiza en clase 002).

### Archivos clave de Energy Plus (presentados)

- **IDF (Input Data File)** — archivo de texto plano que describe la edificación, compuesto de **objetos**.
- **EPW (Energy Plus Weather)** — archivo de clima con ubicación geográfica + datos horarios (T_amb, radiación global/directa/difusa, HR, viento, lluvia, presión atmosférica).
- **OSM (Open Studio Model)** — formato propio de Open Studio, también texto plano.

### Texto plano (aclaración didáctica)

- Es: `.idf`, `.epw`, `.osm`, `.csv`, `.tex`, `.md`, `.py`.
- No es: `.docx`, `.xlsx` (codificados — abrir con editor de texto muestra basura).
- El IDF es texto plano; Energy Plus y Open Studio lo leen y lo presentan en interfaces.

## Simplificaciones del modelo del curso

Suposiciones que limitan el realismo pero permiten enfocarse en lo bioclimático. **Todas se levantan en la siguiente materia (Energía en Edificaciones).**

| # | Simplificación | Razón |
|---|----------------|-------|
| 1 | **Sin ventilación natural** (zonas herméticas) | Modelar ventilación cruzada en Energy Plus es complejo; Open Studio ni siquiera la expone porque no es prioridad en el paradigma de construcción de EE.UU. |
| 2 | **Sin cargas térmicas internas** (personas ~70-100 W, equipos, iluminación) | Es laborioso y aleja del objetivo |
| 3 | **Geometrías simples** (cubos con ventanas) | Evita que el curso se vuelva clase de dibujo; Open Studio nativo no permite geometrías complejas |
| 4 | **Piso adiabático** (flujo de calor cero) | Determinar la temperatura del suelo (ground) es todo un arte que merece su propio tema |

### Implicación práctica

Los resultados absolutos **no son trasladables a una edificación real**, pero el **orden relativo entre estrategias se conserva**: se puede usar como guía de priorización. Ejemplo: comparar casa pintada de blanco vs. azul → la cuantificación absoluta variará en condiciones reales, pero el orden (cuál es mejor) suele mantenerse.

> **Advertencia explícita del profesor:** estos resultados sirven para entender, no para tomar decisiones de eficiencia energética en una empresa real ni diseñar edificaciones reales sin saber hacer simulaciones completas.

## Herramientas presentadas

### Que se usarán en el curso

- [[../tools/Open-Studio]] **versión 1.11.0 (release candidate)** — interfaz gráfica; trae Energy Plus integrado. **Acuerdo:** todo el grupo usa la misma versión (las versiones nuevas pueden abrir archivos viejos, no al revés).
- [[../tools/EnergyPlus]] — motor de cálculo. No se instala por separado, viene dentro de Open Studio. Documentación clave: **Input Output Reference** (~2952 pp.) y **Engineering Reference**.
- [[../tools/Python]] + Jupyter Notebook — para análisis y visualización de resultados.

### Mencionadas pero no se usan

- **Design Builder** — software de paga (~£1800/año), enfocado a cumplir normativas, automatiza reportes. Crítica del profesor: propicia malas prácticas porque oculta supuestos (condiciones de frontera mal puestas pasan desapercibidas).
- **Rhino + LadyBug/Honeybee** — simulaciones paramétricas, muy usado en consultoría internacional; Rhino es de paga.
- **SketchUp** — antes gratis, ahora $1600 MXN/año estudiantes; útil para geometrías complejas. El profesor lo usa con licencia personal pero no exige que los estudiantes paguen.
- **IES (ISP)**, **TRNSYS** — alternativas comerciales europea/estadounidense.
- **Radiance** — iluminación natural (fuera del alcance del curso).
- **Climate Consultant** — mencionado al pasar.

### Filosofía de software

- Solo **software libre**: filosofía + razón pragmática.
- En el instituto **han corrido a personas por usar software pirata** — en máquinas institucionales son rastreables.
- La curva de aprendizaje del software libre es más dura, pero permite entender los fundamentos en lugar de "dar clic sin saber".

## Tarea de la clase

Instalar Open Studio 1.11.0. Ver [[../procedures/Instalar-Open-Studio]] para el procedimiento detallado.

## Recursos mencionados

- Documentación Energy Plus (PDFs):
  - **Input Output Reference** — entradas, opciones, salidas (uso principal)
  - **Engineering Reference** — ecuaciones, correlaciones, métodos numéricos (uso principal)
  - **Getting Started** — visión general
  - **Auxiliary Programs** — descripción del archivo de clima
- Cursos previos del profesor:
  - **"De Cero a Infinito"** — Educación Continua UNAM, gratis para estudiantes del instituto. Si el acceso se cerró, mandar correo a Educación Continua para extender; si no responden, escribir al profesor.
  - **Especialización en Coursera** (3 cursos): Python → análisis de datos → buenas prácticas de developer en ciencia de datos. Videos de 5-7 min, ~65-70 videos por curso, libretas de autoevaluación.
- **Hackatón de visualización de datos** (extra al curso): premio 3 computadoras al 1° y 2° lugar.

## Reglas del curso

Las reglas de operación (comunicación, equipos, evaluación, IA, asistencia, política de violencia) están en archivo aparte: [[REGLAS_CURSO]].

## Conexiones

- → **Siguiente clase:** [[002-ConceptosBasicosBalancesCalor]] — empieza la física (balances de calor, transferencia de calor)
- → **Materia siguiente del plan de estudios:** Energía en Edificaciones — levanta las simplificaciones (ventilación natural, cargas térmicas, geometrías complejas, ventilación cruzada)
- → Reglas: [[REGLAS_CURSO]]
