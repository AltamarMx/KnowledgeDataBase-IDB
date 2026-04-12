# Wiki IDB — Log

Registro cronológico de todas las acciones realizadas por el agente.

---

## 2026-04-10

### Ingesta: 001 — Introducción al Taller IDB
- **Video:** `raw/videos/001 introducción al taller IDB.mp4`
- **Método de transcripción:** Whisper (modelo base, español)
- **Acciones:**
  - Creada página de clase: `wiki/classes/001-IntroduccionTallerIDB.md`
  - Creadas 6 páginas de conceptos: Diseño Bioclimático, Simulación Energética, Confort Térmico, Envolvente Arquitectónica, Condiciones de Frontera, Sistemas Constructivos
  - Creadas 3 páginas de herramientas: Open Studio, EnergyPlus, Python
  - Creado 1 procedimiento: Instalar Open Studio
  - Creado `wiki/index.md`
  - Creado `wiki/log.md`

### Ingesta: 002 — Conceptos Básicos y Balances de Calor
- **Video:** `raw/videos/002 Conceptos Basicos y  Balances de Calor.mp4`
- **Método de transcripción:** Whisper (modelo base, español)
- **Acciones:**
  - Creada página de clase: `wiki/classes/002-ConceptosBasicosBalancesCalor.md`
  - Creadas 5 páginas de conceptos nuevos: Zona Térmica, Balance de Calor, Absorptancia Solar, Factor de Vista, TMY
  - Actualizadas 4 páginas de conceptos existentes: Simulación Energética, Condiciones de Frontera, Envolvente Arquitectónica, Sistemas Constructivos
  - Actualizado `wiki/index.md`

### Ingesta: 003 — Mi Primera Simulación
- **Video:** `raw/videos/003 Mi Primera Simulacion.mp4`
- **Método de transcripción:** Whisper (modelo base, español)
- **Acciones:**
  - Creada página de clase: `wiki/classes/003-MiPrimeraSimulacion.md`
  - Creadas 2 páginas de conceptos nuevos: Mezclado Perfecto, Emitancia
  - Actualizadas 3 páginas de conceptos existentes: Balance de Calor (interior completo), Zona Térmica (espacio vs zona), Condiciones de Frontera (colores en Open Studio)
  - Creado 1 procedimiento: Crear Simulación en Open Studio (paso a paso completo)
  - Actualizado `wiki/index.md`

## 2026-04-11

### Ingesta: 006 — 2 Zonas Térmicas con Ventanas y Aleros
- **Video:** `raw/videos/006 2 ZonasTermicas con Ventanas y Aleros.mp4`
- **Método de transcripción:** Transcripción preexistente (archivo .md)
- **Acciones:**
  - Creada página de clase: `wiki/classes/006-DosZonasTermicasVentanasAleros.md`
  - Creadas 2 páginas de conceptos nuevos: Ventanas, Superficies de Sombramiento
  - Actualizadas 3 páginas de conceptos existentes: Zona Térmica (diferentes alturas), Condiciones de Frontera (intersección automática), Sistemas Constructivos (densidad-conductividad)
  - Actualizadas 2 páginas de herramientas: Open Studio (ventanas, overhangs/fins), EnergyPlus (materiales de vidrio, sombramiento)
  - Creado 1 procedimiento: Agregar Ventanas en Open Studio
  - Actualizado `wiki/index.md`

### Ingesta: 004 — Interpretando los Mensajes de Simulaciones y Construction Sets
- **Video:** `raw/videos/004 Interpretando los mensajes de simulaciones y construction sets.mp4`
- **Método de transcripción:** Transcripción preexistente (archivo .md)
- **Acciones:**
  - Creada página de clase: `wiki/classes/004-InterpretandoMensajesConstructionSets.md`
  - Creadas 2 páginas de conceptos nuevos: Masa Térmica, Warm-up Period
  - Actualizadas 2 páginas de conceptos existentes: Sistemas Constructivos (Construction Sets), Condiciones de Frontera (sun/wind exposure, Other Side Coefficients)
  - Actualizadas 2 páginas de herramientas: EnergyPlus (warm-up, cálculo sombras), Open Studio (Construction Sets, Show Simulation, estructura carpetas)
  - Creado 1 procedimiento: Leer Archivo ERR
  - Actualizado `wiki/index.md`

### Ingesta: Notebook 002_EDA_EPW.ipynb
- **Fuente:** `raw/notebooks/002_EDA_EPW.ipynb`
- **Acciones:**
  - Creada página de procedimiento: `wiki/procedures/EDA-Archivo-EPW.md`
  - Actualizadas 2 páginas de conceptos: TMY (referencia a libreta), Confort Térmico (referencia a promedios mensuales)
  - Actualizada página de herramienta: Python (referencia a libreta)
  - Actualizado `wiki/index.md`

### Ingesta: Notebook 001_EDA.ipynb
- **Fuente:** `raw/notebooks/001_EDA.ipynb`
- **Acciones:**
  - Creada página de procedimiento: `wiki/procedures/EDA-Resultados-Simulacion.md`
  - Actualizada página de herramienta: Python (referencia a la libreta)
  - Actualizado `wiki/index.md`

### Ingesta: 008 — Shading en Ventanas
- **Video:** `raw/videos/008 Shading en ventanas.mp4`
- **Método de transcripción:** Transcripción preexistente (archivo .md)
- **Acciones:**
  - Creada página de clase: `wiki/classes/008-ShadingVentanas.md`
  - Actualizadas 3 páginas de conceptos existentes: Confort Térmico (grados-hora detalle, flujo de cálculo), Superficies de Sombramiento (Sunlit Fraction, algoritmo), Ventanas (quirk de radiación en sub-superficies)
  - Actualizadas 3 páginas de herramientas: EnergyPlus (Sunlit Fraction, Engineering Reference), Python (debugging, estructura libretas), Open Studio (verificación shading en IDF)
  - Actualizado `wiki/index.md`

### Ingesta: 007 — Caso Base y Aleros
- **Video:** `raw/videos/007 Caso base y aleros.mp4`
- **Método de transcripción:** Transcripción preexistente (archivo .md)
- **Acciones:**
  - Creada página de clase: `wiki/classes/007-CasoBaseAleros.md`
  - Actualizadas 2 páginas de conceptos existentes: Superficies de Sombramiento (efectividad según orientación), Ventanas (orientación y protección solar)
  - Actualizadas 3 páginas de herramientas: Python (buenas prácticas para estudios paramétricos), Open Studio (bug FloorSpaceJS, Save As), EnergyPlus (radiación incidente por superficie)
  - Actualizado `wiki/index.md`

### Ingesta: 005 — Primer Análisis de Simulaciones usando Python
- **Video:** `raw/videos/005 Primer Analisis de simulaciones usando Python.mp4`
- **Método de transcripción:** Transcripción preexistente (archivo .txt)
- **Acciones:**
  - Creada página de clase: `wiki/classes/005-AnalisisSimulacionesPython.md`
  - Creada 1 página de concepto nuevo: Temperatura Operativa
  - Actualizadas 4 páginas de conceptos existentes: Mezclado Perfecto (variable Zone Mean Air Temperature), Confort Térmico (modelo adaptativo Humphreys-Nicol), TMY (año 2006 en EnergyPlus), Simulación Energética (fase de análisis)
  - Actualizadas 3 páginas de herramientas: Python (uv, ear_tools, pandas, matplotlib), EnergyPlus (RDD, SQL, variables de salida), Open Studio (Measures, BCL)
  - Creados 2 procedimientos: Configurar Variables de Salida, Analizar Resultados con Python
  - Actualizado `wiki/index.md`
