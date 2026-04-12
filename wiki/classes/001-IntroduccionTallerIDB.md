# 001 — Introducción al Taller IDB

## Metadatos
- **Clase:** 001
- **Título:** Introducción al Taller IDB
- **Duración:** ~1h 37min
- **Profesor:** Guillermo Barrios del Valle
- **Temas:** presentación del curso, reglas, objetivo, herramientas, simplificaciones, instalación de Open Studio

---

## Resumen

Clase introductoria del taller de Diseño Bioclimático. El profesor Guillermo Barrios del Valle establece las reglas del curso, presenta el objetivo principal, explica el ecosistema de software que se usará y las simplificaciones que se harán durante el semestre.

### Logística y reglas del curso

- Comunicación vía **Google Chat Spaces** (no WhatsApp ni mensajes privados de Classroom)
- Trabajo por **equipos fijos** — no se aceptan cambios una vez definidos
- Se deben aprobar tanto la parte teórica (Miriam) como el taller (Guillermo) para pasar la materia
- Ejercicios en clase por equipos; quien no termine se lo lleva de tarea
- Clases grabadas en video — se puede faltar pero se debe ver el video
- Política de **cero tolerancia a la violencia** y uso de lenguaje incluyente
- No se toma asistencia formal

### Objetivo del curso

> Evaluar estrategias bioclimáticas cuantificando el impacto en edificaciones usando simulaciones numéricas.

Los estudiantes podrán cuantificar diferencias (ej. casa blanca vs. azul) en temperatura máxima, promedio y disconfort térmico, para priorizar estrategias con relación desempeño térmico / costo.

### Desempeño térmico — dos dimensiones

1. **Confort térmico** — el problema "invisible" porque no se traduce directamente en emisiones ni ahorro de energía, pero afecta calidad de vida. Es el **énfasis del curso** y del grupo de investigación (Energías en Edificaciones). Enfoque particular en vivienda social en México.
2. **Consumo de energía** — relacionado con aire acondicionado, calefacción e iluminación. En México ~20% de edificaciones usan A/C.

### Ecosistema de software

- **[[Open-Studio]]** — interfaz gráfica gratuita y libre que conecta con Energy Plus y Radiance. Genera archivos OSM.
- **[[EnergyPlus]]** — motor de cálculo (kernel) para simulaciones energéticas. Lee archivos IDF y EPW.
- **[[Python]]** + Jupyter Notebook — para análisis de datos y resultados de simulaciones.
- **Design Builder** — alternativa de paga (~1,800 libras/año); no se usa en el curso. Facilita cumplimiento normativo pero propicia malas prácticas si no se entiende la física.
- **Ladybug Tools** — ecosistema que conecta Rhino con Energy Plus, Radiance, etc. (no se usará en el curso).
- **Radiance** — programa para simulación de iluminación natural (no se cubre en el curso).

### Documentación de Energy Plus

| Documento | Páginas | Contenido |
|-----------|---------|-----------|
| Input/Output Reference | ~2,952 | Entradas, salidas, opciones y objetos de Energy Plus |
| Engineering Reference | ~1,800 | Ecuaciones, correlaciones y métodos numéricos |
| Getting Started | — | Introducción general |
| Auxiliary Programs | — | Descripción de archivos de clima, etc. |

No hay que memorizarla, sino **aprender a consultarla**.

### Archivos clave

- **IDF** (Input Data File) — archivo de texto plano que describe la edificación: materiales, geometría, sistemas constructivos, condiciones de frontera. Es lo que lee Energy Plus.
- **EPW** (Energy Plus Weather) — archivo de clima con datos horarios: temperatura ambiente, radiación (global, directa, difusa), humedad relativa, lluvia, velocidad y dirección de viento, presión atmosférica, ubicación geográfica (latitud, longitud, elevación).
- **OSM** (Open Studio Model) — formato nativo de Open Studio.

Para simular se necesitan dos archivos: el IDF (la casa) y el EPW (el clima). Cambiar el EPW = cambiar la ubicación.

### Simplificaciones del curso

Estas simplificaciones se superan en la materia **Energía en Edificaciones** (siguiente semestre):

| Simplificación | Razón |
|---------------|-------|
| **Sin ventilación natural** | La ventilación cruzada es compleja y no está disponible desde Open Studio (paradigma de EE.UU. vs. México) |
| **Sin cargas térmicas internas** | No se modelan personas (~70-100 W/persona) ni equipos eléctricos |
| **Geometrías simples** | Cubos con ventanas; sin techos inclinados ni formas complejas. El editor de Open Studio lo limita. |
| **Piso adiabático** | La temperatura del suelo (ground) depende de material, humedad, clima — es compleja de modelar |

El resultado no representa una edificación real, pero el **orden de efectividad** de las estrategias bioclimáticas se conserva.

### Sobre el uso de IA

- El profesor es pro-IA (usa Claude, plan Max $100/mes)
- Recomienda que los estudiantes **no la usen** en esta etapa de aprendizaje
- IA es mala para simulaciones en Energy Plus (alucina con objetos inexistentes)
- Es buena para programar y análisis de datos
- No se penaliza su uso, pero hay que desarrollar pensamiento crítico primero

### Tarea

- Instalar **Open Studio versión 1.11.0** desde OpenStudio Coalition > GitHub releases
- Todos deben usar la misma versión para compatibilidad de archivos
- Disponible para Windows, Mac (>= macOS 13) y Ubuntu (22/24)

---

## Conceptos clave

- **[[Diseno-Bioclimatico]]** — diseño de edificaciones que aprovecha las condiciones climáticas para confort térmico
- **[[Simulacion-Energetica]]** — modelado numérico del comportamiento térmico y energético de edificaciones
- **[[Confort-Termico]]** — condición de bienestar respecto a la temperatura interior; problema "invisible" en la literatura
- **[[Envolvente-Arquitectonica]]** — conjunto de superficies que delimitan la edificación (muros, techo, piso, ventanas)
- **[[Condiciones-de-Frontera]]** — especificaciones matemáticas en los límites del modelo (adiabática, temperatura, flujo de calor)
- **[[Sistemas-Constructivos]]** — combinación ordenada de materiales que conforman una superficie (ej. concreto + tabique + concreto)

## Herramientas mencionadas

[[Open-Studio]] · [[EnergyPlus]] · [[Python]] · Design Builder · Ladybug Tools · Rhino · SketchUp · Radiance

## Conexiones

- **Siguiente:** [[002-ConceptosBasicosBalancesCalor]] — Conceptos básicos y balances de calor
- **Materia relacionada:** Energía en Edificaciones (siguiente semestre, cubre las simplificaciones)
