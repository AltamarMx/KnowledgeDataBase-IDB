# 005 — Primer Análisis de Simulaciones usando Python

## Metadatos
- **Clase:** 005
- **Título:** Primer Análisis de Simulaciones usando Python
- **Temas:** [variables de salida EnergyPlus, archivo RDD, measures en Open Studio, flujo de análisis con Python, uv, ear_tools, pandas, matplotlib, análisis de EPW, temperatura operativa, modelo adaptativo de confort]

## Resumen

La clase marca la transición de "hacer simulaciones" a "analizar resultados". El profesor enfatiza que el análisis de datos es **obligatorio**, especialmente para edificaciones sin aire acondicionado, donde no basta minimizar energía sino que hay que analizar series temporales de temperatura y radiación.

Se recorre el flujo completo: desde identificar qué variables de salida ofrece EnergyPlus (archivo RDD), configurar measures en Open Studio para extraerlas, hasta cargar los datos en Python y hacer gráficas exploratorias (EDA). Se cierra con una demostración de lectura de archivos EPW y una introducción al modelo adaptativo de confort (ecuación de Humphreys-Nicol).

## Contenido detallado

### 1. Variables de salida de EnergyPlus (archivo RDD)

Después de correr una simulación, EnergyPlus genera un archivo **RDD** (Report Data Dictionary) que lista **todas las variables de salida disponibles para esa simulación específica**. Esto es clave: si configuré aire acondicionado y no aparece ninguna variable de A/C en el RDD, algo está mal.

**Variables solicitadas en el ejercicio:**
| Variable | Descripción | Alias |
|----------|-------------|-------|
| `Zone Mean Air Temperature` | Temperatura promedio del aire en la zona (modelo de [[Mezclado-Perfecto]]) | `Ti` |
| `Site Outdoor Air Drybulb Temperature` | Temperatura exterior de bulbo seco (del EPW) | `To` |
| `Site Diffuse Solar Radiation Rate Per Area` | Radiación difusa (W/m²) | `Id` |
| `Site Direct Solar Radiation Rate Per Area` | Radiación directa normal (W/m²) | `Ib` |
| `Surface Outside Face Incident Solar Radiation Rate Per Area` | Radiación incidente total sobre una superficie específica (directa proyectada + difusa) | `Is` |

**Observaciones importantes:**
- Las variables `Site` son del clima (EPW); las `Zone` son de la zona térmica; las `Surface` son por superficie (inside/outside)
- La radiación global horizontal no existe como variable directa — hay que pedir difusa y directa por separado, o medir la incidente sobre una superficie horizontal sin sombreamiento
- EnergyPlus puede calcular la temperatura exterior a la **altura del centroide de la zona térmica** (capa límite atmosférica), no solo a los 10 m de la estación meteorológica
- Hay que distinguir entre variables en **Watts** (heat rate) y en **Joules** (heat gain)
- Se puede pedir la variable para todas las zonas/superficies (asterisco) o para una específica (nombre exacto)

### 2. Documentación Input/Output Reference

Para entender qué significa cada variable, se consulta el **Input/Output Reference** de EnergyPlus. Ejemplo: `Zone Mean Air Temperature` se define como "the average temperature of the air temperatures at the system time step" — directamente ligada al modelo de [[Mezclado-Perfecto]].

### 3. Temperatura operativa vs. temperatura del aire

- **Temperatura operativa** = promedio de temperatura de bulbo seco + temperatura radiante media
- **Temperatura radiante** = temperatura de un cuerpo en equilibrio radiativo con las superficies que lo rodean
- Son iguales cuando no hay fuentes radiantes importantes; difieren cuando hay radiación solar directa, superficies muy calientes, etc.
- Ejemplo práctico: una plancha de concreto exterior con baja absortancia refleja radiación → la temperatura operativa en planta baja es mayor que la temperatura del aire → fuente de disconfort
- En el ejercicio se mencionan las variables `Zone Mean Air Temperature` y `Zone Operative Temperature`

### 4. Measures en Open Studio

Open Studio tiene dos puntos de inserción para **measures** (scripts en Ruby):
1. Antes de convertir OSM → IDF
2. Después de generar el IDF, antes de correr EnergyPlus

**Measures usados en el ejercicio:**
| Measure | Categoría | Función |
|---------|-----------|---------|
| `Add Output Variable` | Energy Plus Measures | Agrega una variable de salida al modelo. Requiere nombre exacto (copiar del RDD), frecuencia (timestep recomendado) y clave (asterisco para todas, o nombre específico) |
| `Create CSV Output` | Reporting / QAQC | Exporta los resultados del SQL a un archivo CSV legible |
| `OpenStudio Results` | Reporting | Genera un reporte HTML con resultados estándar (viene por defecto) |

**Fuente de measures:** BCL (Building Component Library) — servicio en línea accesible desde Open Studio (esquina inferior derecha, "Find Measures on BCL").

**Cuidado:** cada vez que se agrega un measure, el nombre de la carpeta de salida cambia (ej. de `001/` a `005/`), lo que rompe las rutas en notebooks. Por eso el profesor prefiere trabajar con el **SQL** directamente.

### 5. SQL vs CSV para resultados

| Aspecto | CSV | SQL |
|---------|-----|-----|
| Legibilidad | Fácil (abrir con cualquier editor) | Requiere herramienta |
| Estabilidad de ruta | Cambia con cada measure agregado | Siempre en `run/eplusout.sql` |
| Compatibilidad OS/EP | Open Studio pone 00:00; EnergyPlus pone 24:00 (rompe pandas) | Consistente |
| Recomendación | Útil para verificación rápida | **Preferido para análisis** |

### 6. Entorno Python con uv

Se configura el entorno de trabajo con **uv** (gestor de paquetes escrito en Rust):
- `uv init` — inicializa proyecto con `pyproject.toml`, `.python-version`, ambiente virtual (`.venv`)
- `uv add pandas jupyter notebook matplotlib` — instala dependencias
- `uv add git+<url_ear_tools>` — instala paquete del grupo desde GitHub
- `uv run jupyter notebook` — ejecuta Jupyter usando el ambiente virtual del proyecto
- **Nunca** usar `pip install` dentro del notebook — usar `uv add` desde la terminal

**Ventajas de ambientes virtuales:**
- Reproducibilidad (cualquier máquina recrea el ambiente desde `pyproject.toml`)
- Aislamiento (no contamina la instalación base de Python del SO)
- Mac y Linux ya no permiten instalar paquetes globalmente
- Si se rompe, se borra `.venv` y se recrea

### 7. ear_tools — paquete del grupo

Paquete Python desarrollado por el grupo de Energía en Edificaciones que automatiza tareas comunes:
- `read_sql(path, alias=True)` — carga simulación desde `eplusout.sql`, retorna objeto con `.data` (DataFrame) y `.construction_systems` (lista de sistemas constructivos)
- **Alias:** renombra columnas largas de EnergyPlus a nombres cortos y amigables:
  - `CUBO:Zone Mean Air Temperature [C]` → `Ti_cubo`
  - `Site Outdoor Air Drybulb Temperature` → `To`
  - `Site Diffuse Solar Radiation Rate Per Area` → `Id`
  - `Site Direct Solar Radiation Rate Per Area` → `Ib`
- `read_epw(path, year=2006, alias=True)` — carga archivo EPW como DataFrame
- `get_constructions(sc)` — muestra sistemas constructivos con capas, espesores y propiedades térmicas (útil para verificación/QA)

### 8. Flujo de análisis EDA (Exploratory Data Analysis)

1. Definir ruta al SQL con ruta relativa (`f = "../osm/mi_primer_cubo_002/run/eplusout.sql"`)
2. Cargar con `read_sql(f, alias=True)`
3. Verificar sistemas constructivos (propiedades térmicas, orden de capas)
4. Graficar temperaturas (interior vs exterior) y radiación en subplots compartiendo eje X
5. Hacer zoom temporal con `pd.Timedelta(days=7)` + `set_xlim`
6. Usar `dateutil.parser.parse` para convertir strings a datetime

**Buenas prácticas mencionadas:**
- Siempre graficar la variable climática de referencia (temperatura exterior, radiación) junto con la temperatura interior
- No hacer gráficas con doble eje Y (difíciles de leer en presentaciones) — usar subplots
- `fig, ax = plt.subplots(2, 1, sharex=True, figsize=(12, 4))`
- Ejecutar "Restart & Run All" periódicamente para verificar robustez del notebook
- Mantener estructura de carpetas ordenada para automatizar rutas relativas

### 9. Análisis del EPW y modelo adaptativo

Con `read_epw` se carga el archivo climático y se puede:
- Graficar todas las variables con `df.plot(subplots=True)`
- Calcular temperatura exterior promedio mensual: `df['To'].resample('ME').mean()`
- Aplicar el modelo de **Humphreys-Nicol**:
  > **T_neutralidad = 0.54 × T_exterior_promedio_mensual + 13.5**
- Con T_neutralidad y la amplitud promedio (ΔT), definir zona de confort: [T_n - ΔT, T_n + ΔT]
- Calcular grados-hora de disconfort (cálido o frío)

### 10. Tarea asignada

Crear una simulación con **dos zonas térmicas** (una orientada al este, otra al oeste) sin ventanas. Nombres descriptivos (Este/Oeste, no ThermalZone1/2). Analizar comportamiento diferenciado.

## Conceptos clave

- **[[Temperatura-Operativa]]** — promedio de temperatura del aire y temperatura radiante; relevante cuando hay fuentes radiantes importantes
- **[[Mezclado-Perfecto]]** — `Zone Mean Air Temperature` es resultado directo de este modelo
- **[[Confort-Termico]]** — modelo adaptativo de Humphreys-Nicol para zona de confort
- **[[TMY]]** — el EPW puede tener meses de años distintos; EnergyPlus asigna año 2006 por defecto
- **[[Balance-de-Calor]]** — las variables de salida permiten inspeccionar componentes individuales del balance
- **[[Absorptancia-Solar]]** — se puede verificar en los sistemas constructivos cargados desde el SQL

## Herramientas y software

- **[[EnergyPlus]]** — archivo RDD, variables de salida, SQL, documentación Input/Output
- **[[Open-Studio]]** — Measures (BCL), Add Output Variable, Create CSV Output
- **[[Python]]** — pandas, matplotlib, Jupyter, ear_tools, uv, dateutil

## Conexiones con otras clases

- ← Anterior: [[003-MiPrimeraSimulacion]] — primera simulación que ahora se analiza
- → Siguiente: [[006-DosZonasTermicasVentanasAleros]] — se aplica este flujo a un modelo con dos zonas y ventanas
