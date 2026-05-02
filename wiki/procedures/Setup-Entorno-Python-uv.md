---
title: Setup del entorno Python con uv
type: procedimiento
tags: [procedimiento, python, uv, ambientes-virtuales, jupyter, ear-tools]
aliases: [setup uv, instalar python, ambiente virtual]
clases: [005]
updated: 2026-05-02
---

# Setup del entorno Python con uv

Procedimiento para crear un ambiente virtual reproducible para el análisis de simulaciones del taller, usando **uv** como gestor de paquetes.

## Por qué uv

- **Velocidad**: uv está escrito en Rust → instalación ~10× más rápida que pip.
- **Reproducibilidad**: maneja `pyproject.toml` + `uv.lock` automáticamente. Funciona igual en Mac, Windows y Linux.
- **Aislamiento**: cada proyecto tiene su `.venv` independiente. No hay conflictos entre proyectos.
- **Multi-versión**: puede instalar y cambiar versiones de Python sin tocar el sistema.

> "Yo me moví a ambientes virtuales el día que perdí 3 días tratando de arreglar mi Mac y me di cuenta que tenía 7 versiones de Python instaladas."

Alternativas conocidas:

- **Miniconda**: el profesor lo usa en paralelo porque su terminal activa bien los ambientes virtuales. Buena opción para quien viene de Anaconda.
- **pip + venv puro**: funciona pero más manual.
- **Poetry**: similar a uv pero más lento.

## Pre-requisitos

- **uv instalado** en el sistema. Instalación: ver `https://docs.astral.sh/uv/`. En Mac/Linux: `curl -LsSf https://astral.sh/uv/install.sh | sh`.
- **Git** instalado (para descargar `ear_tools` desde GitHub).

## 1. Posicionarse en el folder del proyecto

```bash
cd ~/Escritorio/septimo_semestre/IDB/tarea_02_dos_zonas/
```

Estructura ya creada con [[Estructura-Proyecto-Simulacion]]:

```
tarea_02_dos_zonas/
├── OSM/
├── EPW/
└── notebooks/
```

## 2. Inicializar el proyecto Python

```bash
uv init
```

Esto crea:

- **`pyproject.toml`** — declarativo: lista de dependencias del proyecto.
- **`main.py`** — script de ejemplo (se puede borrar).
- **`README.md`** — generado vacío.
- **`.python-version`** — versión de Python que el proyecto usa.
- Inicia un repo Git (`.git/`) si no había.

> Si **ya estás en un repo Git existente**, `uv init` lo detecta y no crea uno nuevo. Bien.

## 3. Agregar dependencias del taller

```bash
uv add pandas jupyter matplotlib python-dateutil
uv add git+https://github.com/<grupo-IER>/ear-tools.git
```

(URL exacta del repo según el README del taller — el profesor la comparte por GitHub directo.)

Esto:

- Resuelve dependencias.
- Crea `.venv/` con todos los paquetes.
- Actualiza `pyproject.toml` con las dependencias declaradas.
- Crea/actualiza `uv.lock` con las versiones exactas (incluye dependencias transitivas).

> **Tiempo**: la primera vez tarda — descarga e instala. Las siguientes veces es casi instantáneo (cache).

### Las dependencias del curso

| Paquete | Para qué |
|---------|----------|
| `pandas` | Series temporales, DataFrames |
| `jupyter` | Notebook interactivo |
| `matplotlib` | Gráficas |
| `python-dateutil` | Parser flexible de fechas (`dateutil.parser.parse`) |
| `ear-tools` | Lectura de SQL/EPW + auditoría de constructions |

(numpy entra como dependencia transitiva de pandas.)

## 4. .gitignore

uv crea uno básico. Asegurar que **`.venv/`** está ignorada (puede pesar varios GB):

```
.venv/
__pycache__/
*.pyc
.ipynb_checkpoints/
```

> uv usa cache + symlinks → no replica contenidos físicamente. Aún así, no se sube al repo.

## 5. Lanzar Jupyter Notebook

```bash
uv run jupyter notebook
```

`uv run` busca el ambiente virtual del folder actual (o ancestros) y ejecuta el comando dentro de él. **No es necesario `activar`** explícitamente.

> Si te molesta no tener el activador clásico: `source .venv/bin/activate` (Mac/Linux) o `.venv\Scripts\activate` (Windows). Pero `uv run` es más limpio.

## Reglas críticas (no romper)

### NO usar `pip install`

```bash
# ❌ MAL — dentro de un notebook o terminal
!pip install seaborn

# ✅ BIEN — desde la terminal del proyecto
uv add seaborn
```

Razón: `pip install` se sale del seguimiento de `pyproject.toml`. La instalación queda local pero no documentada — al recrear el ambiente desde otra máquina, falta esa dependencia.

uv expone `uv pip install` como puente para que la gente con muscle memory de pip no extrañe demasiado, pero **lo recomendado es `uv add`**.

### NO usar `python script.py`

```bash
# ❌ MAL — usa el Python del sistema, no del ambiente
python script.py

# ✅ BIEN
uv run script.py
```

### NO instalar Python "por encima" del de Mac

macOS usa Python para su sistema operativo. Desinstalar Python de Mac puede romper el OS. uv (y Miniconda) instalan versiones aisladas que no tocan el Python del sistema — usar siempre.

> "En Windows tú puedes desinstalar Python y volver a instalarlo. En Mac no — Mac usa Python para su sistema operativo."

## Cambiar la versión de Python del proyecto

```bash
uv python install 3.11
uv python pin 3.11
uv sync
```

uv actualiza el `.python-version`, recrea `.venv` con esa versión y resuelve dependencias.

## Reproducir el ambiente en otra máquina

Quien clone el repo:

```bash
cd <repo>
uv sync
```

Esto recrea el `.venv` con las versiones exactas del `uv.lock`. Cero pasos manuales adicionales.

## Si algo se rompe — borrar y recrear

```bash
rm -rf .venv
uv sync
```

Regenera el ambiente desde cero con las versiones del lock. **Solución universal** cuando el ambiente se corrompe (típicamente <30 segundos).

> "Con el ambiente virtual borras el `.venv` y lo vuelves a crear y ya quedó."

## Estructura final esperada

```
tarea_02_dos_zonas/
├── .git/
├── .venv/                 # ← NO subir al repo
├── .python-version
├── .gitignore
├── pyproject.toml         # ← versionar
├── uv.lock                # ← versionar
├── README.md
├── OSM/
├── EPW/
└── notebooks/
    └── 001_EDA.ipynb
```

## Para Quarto (si lo usas)

Quarto puede embeber Python. Para que use el ambiente uv del proyecto:

1. Crear el proyecto Quarto (`quarto create-project`) — sin nombre, en el folder actual, **no** crea sub-folder.
2. `uv init` en ese mismo folder — convive sin conflicto.
3. Para renderizar: `uv run quarto render` (o `quarto preview`).

> Si `uv init <nombre>` con nombre, crea sub-folder y genera conflicto con la carpeta de Quarto. Mejor sin nombre.

## Clases relacionadas

- [[../classes/005-AnalisisSimulacionesPython]] — primera demo del setup completo
