# Wiki-IDB — Guía de Configuración y Uso

## Qué es esto

Una **base de conocimiento persistente** para las clases del taller IDB. Un agente LLM lee tanto las transcripciones (`.md`) de los videos como las libretas Jupyter (`.ipynb`) de análisis, extrae el contenido, lo estructura, genera referencias cruzadas y mantiene el conocimiento organizado a medida que se procesan más clases.

### Principios clave:
- **La wiki es un artefacto persistente** — no respuestas temporales de chat
- **Las referencias cruzadas se construyen automáticamente** — conceptos, herramientas y técnicas se conectan entre clases
- **El LLM hace el trabajo pesado** (estructurar, vincular) — tú controlas la calidad
- **Tú apruebas todo antes de guardar** — human in the loop

---

## Tu rol

1. Indicar qué video procesar
2. Revisar los borradores generados (resúmenes, conceptos, pasos)
3. Hacer preguntas que se guardan como nuevas páginas
4. Supervisar cómo crece la wiki con el tiempo

### Flujo Human-in-the-Loop:
- **PASO 1:** Indicas al LLM qué contenido ingerir (transcripción `.md` de un video o libreta `.ipynb`) → el agente localiza el archivo
- **PASO 2:** El LLM procesa el contenido (lee transcripción / lee notebook con figuras), extrae contenido, genera borrador estructurado
- **PASO 3:** El LLM te presenta el borrador para revisión
- **PASO 4:** Apruebas (o pides cambios) → el LLM guarda en `wiki/`
- **PASO 5:** El LLM actualiza el índice y el log

---

## Estructura de carpetas

```
Wiki_IDB/
│
├── schema/                        # Instrucciones para el agente — NO tocar
│   ├── AGENTS.md                 # Contrato maestro del agente
│   └── INGESTION_GUIDE.md        # Guía paso a paso del flujo de ingesta
│
├── raw/                           # Inmutable: solo AGREGAR contenido, nunca modificar
│   ├── videos/                   # Transcripciones (.md) de las clases en video
│   │   ├── 001 introducción al taller IDB.md
│   │   ├── 002 Conceptos Basicos y Balances de Calor.md
│   │   ├── 003 Mi Primera Simulacion.md
│   │   ├── 004 Interpretando los mensajes de simulaciones y construction sets.md
│   │   ├── 005 Primer Analisis de simulaciones usando Python.md
│   │   ├── 006 2 ZonasTermicas con Ventanas y Aleros.md
│   │   ├── 007 Caso base y aleros.md
│   │   └── 008 Shading en ventanas.md
│   └── notebooks/                # Libretas Jupyter de análisis (.ipynb)
│       ├── 001_EDA.ipynb
│       └── 002_EDA_EPW.ipynb
│
├── wiki/                          # Todo el contenido generado por el LLM
│   ├── index.md                  # Catálogo de contenido, auto-actualizado
│   ├── log.md                    # Log cronológico de todas las acciones
│   │
│   ├── classes/                  # Resúmenes de clases: NNN-TituloBreve.md
│   ├── concepts/                 # Temas recurrentes: Balances-de-Calor.md
│   ├── tools/                    # Herramientas y software: EnergyPlus.md, Python.md
│   └── procedures/               # Procedimientos paso a paso: Crear-Simulacion.md
│
├── queries/                       # Respuestas a tus preguntas
│   └── q-YYYY-MM-DD-Titulo.md
│
└── notes/                         # Tus notas personales
    └── ...
```

---

## Convención de nombres para clases

Cada resumen de clase sigue: `NNN-TituloBreveCamelCase.md`

Donde `NNN` es el número de clase (coincide con el prefijo de la transcripción).

Ejemplos:
- `001-IntroduccionTallerIDB.md`
- `002-ConceptosBasicosBalancesCalor.md`
- `003-MiPrimeraSimulacion.md`
- `005-AnalisisSimulacionesPython.md`

---

## Tipos de contenido ingerible

| Tipo | Ubicación | Formato | Qué se extrae |
|------|-----------|---------|---------------|
| **Transcripción de clase** | `raw/videos/*.md` | Markdown transcrito del video | Resumen, conceptos, procedimientos, herramientas |
| **Libreta Jupyter** | `raw/notebooks/*.ipynb` | `.ipynb` (se lee directamente con celdas y figuras) | Código, flujo de análisis, gráficas, hallazgos |

---

## Qué se genera al ingerir una transcripción de clase

Cuando apruebas una ingesta, el agente LLM:

1. **Crea página de clase** en `wiki/classes/NNN-Titulo.md`
   - Metadatos: número de clase, título, duración, temas principales
   - Resumen estructurado del contenido
   - Conceptos clave explicados
   - Pasos prácticos / procedimientos mostrados en la clase
   - Timestamps de secciones importantes (cuando aparecen en la transcripción)
   - Referencias a clases anteriores y siguientes

2. **Actualiza `index.md`** — agrega entrada al catálogo de clases

3. **Crea o actualiza páginas de conceptos** según sea necesario:
   - Si "balance de calor" aparece en varias clases → `wiki/concepts/Balances-de-Calor.md` se actualiza
   - Cada página de concepto enlaza a todas las clases relevantes

4. **Crea o actualiza páginas de herramientas** para software mencionado:
   - EnergyPlus, OpenStudio, Python, IDF Editor, etc.
   - Cómo se usa cada herramienta, en qué clases aparece

5. **Crea o actualiza páginas de procedimientos** para flujos paso a paso:
   - "Cómo crear tu primera simulación"
   - "Cómo interpretar mensajes de error"
   - Cada procedimiento referencia la clase donde se enseñó

6. **Actualiza `log.md`** — entrada cronológica de la acción

---

## Qué se genera al ingerir una libreta Jupyter

Cuando apruebas la ingesta de un `.ipynb`, el agente LLM:

1. **Lee la libreta directamente** — celdas de código, markdown y **figuras incluidas** (las figuras embebidas en el output de las celdas son visibles para el LLM)
2. **Crea página de análisis** en `wiki/procedures/` o `wiki/classes/` según corresponda:
   - Descripción del objetivo del análisis
   - Flujo de trabajo paso a paso (qué hace cada bloque de código)
   - Descripción de las gráficas generadas y qué muestran
   - Librerías y funciones usadas (pandas, matplotlib, ear_tools, etc.)
   - Hallazgos o patrones observados en los datos
3. **Actualiza páginas de herramientas** (`wiki/tools/Python.md`, etc.) con funciones y patrones nuevos
4. **Actualiza `index.md`** y **`log.md`**

### Sobre las figuras

- El LLM puede **ver las figuras** directamente en el `.ipynb` (no se necesita exportarlas)
- Las figuras se **describen textualmente** en la wiki (qué ejes, qué variables, qué patrón se observa)
- **No se necesita convertir a `.py`** — eso pierde los outputs y las gráficas, que es lo más valioso
- Si se necesita un artefacto exportable: `uv run jupyter nbconvert --to markdown notebook.ipynb` genera un `.md` con las figuras como PNGs en una carpeta adjunta

### Ejemplo: ingerir una libreta

```markdown
Ingest: raw/notebooks/001_EDA.ipynb
```

El LLM:
1. Lee todas las celdas (código + outputs + figuras)
2. Genera un borrador con el flujo de análisis, descripción de gráficas y hallazgos
3. Tú revisas y apruebas → se guarda en la wiki

---

## Cómo ingerir una transcripción de clase

### Ejemplo: primera clase

```markdown
Ingest: raw/videos/001 introducción al taller IDB.md
```

El LLM:
1. Lee la transcripción completa
2. Extrae el contenido estructurado (temas, explicaciones, demostraciones)
3. Genera un borrador:

```markdown
# 001-IntroduccionTallerIDB.md

## Metadatos
- **Clase:** 001
- **Título:** Introducción al Taller IDB
- **Temas:** [presentación del curso, objetivos, herramientas necesarias]

## Resumen
[Descripción estructurada del contenido de la clase]

## Conceptos clave
- **IDB:** [qué es, para qué sirve]
- **Simulación energética:** [introducción al concepto]

## Herramientas mencionadas
- EnergyPlus, OpenStudio, etc.

## Conexiones con otras clases
- → Siguiente: [[002-ConceptosBasicosBalancesCalor]]

---
¿Es correcto? ¿Algún cambio antes de guardar?
```

Tú revisas y apruebas → el LLM guarda la página, actualiza el índice, crea páginas de conceptos y herramientas, registra en el log.

### Clase siguiente (acumula conocimiento)

```markdown
Ingest: raw/videos/002 Conceptos Basicos y Balances de Calor.md
```

El LLM procesa y detecta que "simulación energética" ya existe como concepto...
- Crea `wiki/classes/002-ConceptosBasicosBalancesCalor.md`
- Actualiza `wiki/concepts/Simulacion-Energetica.md` — ahora referencia ambas clases
- Crea `wiki/concepts/Balances-de-Calor.md` — nuevo concepto
- La página de concepto muestra la progresión entre clases

---

## Consultar la wiki (después de varias clases)

Puedes hacer preguntas como:

```markdown
"¿Qué pasos se siguen para crear una simulación desde cero?"
"¿En qué clases se habla de zonas térmicas?"
"Resumen de todo lo relacionado con shading y aleros"
"¿Qué herramientas de Python se usan para analizar simulaciones?"
```

El LLM busca en `wiki/`, encuentra las páginas relevantes, las lee, sintetiza una respuesta con referencias a las clases. Si la respuesta es sustancial, ofrece guardarla como página nueva.

---

## Progresión del taller (transcripciones disponibles)

| # | Transcripción | Temas esperados |
|---|---------------|----------------|
| 001 | Introducción al taller IDB | Presentación, objetivos, herramientas |
| 002 | Conceptos Básicos y Balances de Calor | Fundamentos térmicos, balances energéticos |
| 003 | Mi Primera Simulación | Primer modelo, configuración básica |
| 004 | Interpretando mensajes y construction sets | Debugging, conjuntos de construcción |
| 005 | Primer Análisis con Python | Scripts de análisis, visualización de resultados |
| 006 | 2 Zonas Térmicas con Ventanas y Aleros | Multi-zona, elementos de fachada |
| 007 | Caso base y aleros | Caso de referencia, estrategias de sombreado |
| 008 | Shading en ventanas | Dispositivos de control solar |

---

## Entorno de desarrollo — uv

Este proyecto usa **[uv](https://docs.astral.sh/uv/)** como gestor de paquetes y entorno Python. Todo se ejecuta a través de `uv`:

| Acción | Comando |
|--------|---------|
| Agregar una dependencia | `uv add nombre-paquete` |
| Ejecutar un script Python | `uv run script.py` |
| Ejecutar main.py | `uv run main.py` |
| Agregar dependencia de desarrollo | `uv add --dev nombre-paquete` |

**Reglas:**
- **Nunca** usar `pip install` — siempre `uv add`
- **Nunca** usar `python script.py` — siempre `uv run script.py`
- `uv` maneja automáticamente el entorno virtual, las versiones de Python y el lockfile
- Las dependencias quedan registradas en `pyproject.toml`

---

## Qué NO hacer

- No modificar archivos en `wiki/` directamente — solo el agente LLM los actualiza
- No borrar archivos de `wiki/concepts/`, `wiki/tools/` o `wiki/procedures/` sin razón
- No renombrar transcripciones en `raw/videos/` ni notebooks en `raw/notebooks/` — son la fuente inmutable

## Qué SÍ hacer

- Agregar nuevas transcripciones a `raw/videos/` y libretas a `raw/notebooks/`
- Revisar borradores antes de aprobar
- Hacer preguntas — las respuestas sustanciales se convierten en páginas
- Pedir mantenimiento periódico de la wiki

---

## Mantenimiento periódico

Cada cierto tiempo, pide al agente:

```markdown
"Revisa la wiki — busca páginas huérfanas, conceptos sin conectar, procedimientos incompletos"
```

El agente:
- Encuentra páginas sin referencias → sugiere consolidar o eliminar
- Identifica conceptos que necesitan páginas propias
- Verifica que los procedimientos estén completos y actualizados
- Sugiere conexiones faltantes entre clases

---

## Para empezar ahora

**Siguiente paso:** indica qué contenido quieres procesar:

```markdown
# Ingerir la transcripción de una clase
Ingest: raw/videos/001 introducción al taller IDB.md

# Ingerir una libreta Jupyter
Ingest: raw/notebooks/001_EDA.ipynb
```

El agente procesará el contenido (transcripción o libreta), extraerá y estructurará la información, te presentará un borrador → tú apruebas → todo se guarda.
# KnowledgeDataBase-IDB
