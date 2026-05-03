---
title: Python
type: herramienta
tags: [herramienta, lenguaje, python, jupyter, analisis-datos]
clases: [001, 004, 005]
updated: 2026-05-02
---

# Python

## Rol en el curso

**Análisis y visualización** de los resultados de las simulaciones. Open Studio y Energy Plus generan datos (CSVs, archivos de salida tabulares); Python + Jupyter Notebook permite:

- Procesar series de tiempo (temperaturas horarias, flujos)
- Comparar escenarios (caso base vs. estrategia X)
- Generar gráficas para reportes y proyecto final
- Calcular métricas de confort/disconfort
- Hacer EDA del archivo EPW antes de simular

## Por qué Python (y no Excel u otro)

- **Software libre.**
- **Multipropósito** — análisis de datos, visualización, automatización, web, etc.
- Programa de ciencia más usado en ciencia de datos → comunidad enorme.
- Los problemas comunes de datos se resuelven en 2-3 líneas; con Excel pueden ser horas.
- El profesor: "a mí me cambió la vida Python".

> **No es obligatorio.** Quien quiera resolver con Excel, R u otro puede hacerlo, pero el profesor solo apoya en Python.

## Stack del curso

| Paquete | Para qué |
|---------|----------|
| `pandas` | Series temporales, DataFrames |
| `jupyter` | Notebook interactivo |
| `matplotlib` | Gráficas |
| `python-dateutil` | Parser flexible de fechas (`dateutil.parser.parse`) |
| **`iertools`** | Lectura directa del SQL/EPW de Energy Plus, alias cortos, auditoría de constructions. Paquete del grupo IER. Ver [[iertools]] |

(numpy entra como dependencia transitiva de pandas.)

## Setup con uv

Procedimiento completo en [[../procedures/Setup-Entorno-Python-uv]]. Resumen:

```bash
cd <folder-del-proyecto>
uv init
uv add pandas jupyter matplotlib python-dateutil
uv add git+https://github.com/<grupo-IER>/iertools.git
uv run jupyter notebook
```

**Reglas críticas**:

- Nunca `pip install` → siempre `uv add`
- Nunca `python script.py` → siempre `uv run script.py`
- `.venv/` no se sube al repo (se regenera con `uv sync`)

> Si el ambiente se corrompe: `rm -rf .venv && uv sync` lo arregla en segundos.

## Para qué se usa Python en el flujo del curso

| Etapa | Uso de Python |
|-------|---------------|
| Antes de simular | EDA del EPW con `read_epw` ([[../procedures/EDA-Archivo-EPW]]) |
| Después de simular | Leer el SQL con `read_sql` → pandas ([[../procedures/Analizar-Resultados-Python]]) |
| Análisis | Comparar caso base vs. variantes; calcular métricas de confort/disconfort ([[../concepts/Confort-Adaptativo]]) |
| Reporte | Gráficas para entregables y proyecto final |

Volumen típico: dos zonas térmicas, paso de 10 minutos, año completo → ~52,560 puntos por zona por variable. Pandas es prácticamente obligado.

## Patrones útiles del taller

### Plots de doble panel

```python
fig, ax = plt.subplots(2, 1, sharex=True, figsize=(12, 4))
ax[0].plot(df.T_cubo, label="T_cubo")
ax[0].plot(df.TO, label="TO")
ax[0].legend()
ax[1].plot(df.IB, label="beam")
ax[1].plot(df.ID, label="diffuse")
ax[1].legend()
```

> **Evitar dobles ejes Y** en presentaciones — son horrorosas, toman 10 min de entender.

### Recorte temporal

```python
from dateutil.parser import parse
f1 = parse("2006-03-13")
f2 = f1 + pd.Timedelta(days=7, hours=3)
ax[1].set_xlim(f1, f2)
```

### Resampling

```python
df.resample("ME").mean()   # ME = Month-End (ya no "M", deprecated en pandas reciente)
df.resample("D").max()
```

### Auto-completado en Jupyter

`Tab` después de un path o un atributo de objeto. Si no completa archivos, estás en otro directorio del que crees — verifica con `!pwd`.

### Restart and Run All

> Kernel → Restart and Run All

Verifica que la libreta es reproducible. Si falla, hay variables fantasma.

## Gestor de paquetes y entorno

El proyecto del curso usa **[uv](https://docs.astral.sh/uv/)**:

| Acción | Comando |
|--------|---------|
| Agregar dependencia | `uv add nombre-paquete` |
| Ejecutar script | `uv run script.py` |
| Ejecutar main.py | `uv run main.py` |
| Dependencia de desarrollo | `uv add --dev nombre-paquete` |

**Reglas:**

- Nunca `pip install` → siempre `uv add`
- Nunca `python script.py` → siempre `uv run script.py`
- uv maneja entorno virtual, versión de Python y lockfile automáticamente
- Dependencias quedan registradas en `pyproject.toml`

## Recursos del profesor

- **"De Cero a Infinito"** (Educación Continua UNAM) — gratis para estudiantes del instituto, videos largos (~50 min), un poco viejo.
- **Especialización en Coursera** (3 cursos): aprender Python → análisis de datos → buenas prácticas de developer en ciencia de datos. Videos de 5-7 min, ~65-70 videos por curso, libretas de autoevaluación, libro acompañante.

## Sobre IA y Python

A diferencia de Energy Plus, en Python la IA es **útil**: análisis de datos, código, visualizaciones. Pero el profesor recomienda **no abusar** durante el aprendizaje — perdiendo la práctica de analizar datos manualmente, no se desarrolla criterio.

## Clases relacionadas

- [[../classes/001-IntroduccionTallerIDB]] — introducción al rol de Python en el curso
- [[../classes/004-InterpretandoMensajesConstructionSets]] — mención del paquete del profesor para leer SQL directo, anuncio del análisis de series temporales en clases siguientes
- [[../classes/005-AnalisisSimulacionesPython]] — primera demo end-to-end del análisis con `iertools`, plotting con matplotlib, EDA del EPW

## Procedimientos

- [[../procedures/Setup-Entorno-Python-uv]]
- [[../procedures/Analizar-Resultados-Python]]
- [[../procedures/EDA-Archivo-EPW]]
