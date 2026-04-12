# Python

Lenguaje de programación multipropósito usado en el taller para análisis de datos y visualización de resultados de simulaciones. El profesor lo impulsa fuertemente pero no es obligatorio (se puede usar Excel, R u otro).

## Por qué Python

- Software libre
- Multipropósito (análisis de datos, juegos, apps, etc.)
- El lenguaje más usado en ciencia de datos → comunidad grande
- Permite conectar con otros programas del ecosistema de simulación
- "Me cambió la vida" — profesor Guillermo

## Entorno: uv

El curso usa **uv** (escrito en Rust) como gestor de paquetes y ambientes virtuales:

| Comando | Acción |
|---------|--------|
| `uv init` | Inicializa proyecto (`pyproject.toml`, `.python-version`, `.venv`) |
| `uv add pandas jupyter notebook matplotlib` | Instala dependencias |
| `uv add git+<url>` | Instala paquete desde repositorio Git |
| `uv run jupyter notebook` | Ejecuta Jupyter en el ambiente virtual |

**Reglas:** nunca `pip install` (ni dentro de notebooks) — siempre `uv add`. Si el ambiente se rompe, borrar `.venv` y recrear.

## Paquetes del curso

- **pandas** — DataFrames, series temporales, datetime, resample
- **matplotlib** — gráficas (subplots, sharex, figsize)
- **jupyter notebook** — libretas interactivas para EDA
- **dateutil** — `parser.parse` para convertir strings a datetime
- **ear_tools** — paquete del grupo de Energía en Edificaciones:
  - `read_sql(path, alias=True)` — carga resultados desde `eplusout.sql`
  - `read_epw(path, year=2006, alias=True)` — carga archivos EPW
  - `get_constructions(sc)` — inspecciona sistemas constructivos
  - Alias: renombra columnas largas a nombres cortos (`Ti`, `To`, `Id`, `Ib`, `Is`)

## En el curso

- Se usa con **Jupyter Notebooks** para análisis de resultados
- No es un curso de Python — se asume conocimiento básico o se aprende en paralelo
- Se va "casi directo" a la libreta de Jupyter
- El profesor habla Python; si se usa otro lenguaje, no puede ayudar directamente
- Flujo EDA: cargar SQL → verificar sistemas constructivos → graficar temperaturas y radiación → zoom temporal

## Recursos recomendados por el profesor

1. **Coursera** — programa de especialización del IER-UNAM (3 cursos): Python básico, análisis de datos, estrategias de desarrollo con ciencia de datos. Videos de 5-7 min, libretas de autoevaluación.
2. **Curso UNAM Educación Continua** (con Timah) — "De Cero a Infinito". Videos más largos (~50 min), gratuito para personas del instituto.
3. **PEPs** (Python Enhancement Proposals) — el profesor lee las PEPs para desarrollar mejores prácticas.

## Buenas prácticas para estudios paramétricos

- **Un DataFrame por simulación** — no mezclar casos en el mismo DataFrame
- **Función de carga reutilizable** — `carga_datos(f)` que aplica `read_sql` + renombrado de columnas
- **Prefijos en columnas** — `Ti_` para temperaturas, `Is_` para radiación incidente → permite filtrar con list comprehension
- **Dict comprehension** para generar diccionarios de renombrado automáticamente y luego editar a mano
- **Visualización comparativa** — mismo color = misma variable, estilo de línea diferente = caso diferente (sólida vs punteada)
- **`sharex=True`** en subplots para zoom sincronizado entre gráficas

## Aparece en

- [[001-IntroduccionTallerIDB]] — Presentación como herramienta de análisis
- [[005-AnalisisSimulacionesPython]] — Uso completo: uv, ear_tools, pandas, matplotlib, flujo EDA
- [[007-CasoBaseAleros]] — Análisis comparativo, funciones de carga, dict comprehension, visualización de múltiples casos
- [[008-ShadingVentanas]] — Debugging de datos, verificación de simulaciones, estructura de libretas para proyecto final
- [[EDA-Resultados-Simulacion]] — Libreta 001_EDA: carga SQL, inspección de construcciones, gráficas de Ti/To/radiación
- [[EDA-Archivo-EPW]] — Libreta 002_EDA_EPW: carga EPW, variables climáticas, promedios mensuales
