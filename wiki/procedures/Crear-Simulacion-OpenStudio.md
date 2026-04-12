# Crear una Simulación en Open Studio

Procedimiento paso a paso para crear y ejecutar una simulación energética completa desde cero en Open Studio.

## Requisitos previos

- Open Studio 1.11.0 instalado ([[Instalar-Open-Studio]])
- Archivo EPW descargado de One Building (climate.onebuilding.org)

## Estructura de proyecto

Crear la siguiente estructura de carpetas (sin acentos, eñes ni espacios):

```
MiProyecto/
├── OSM/          # Archivos Open Studio Model (versionados)
├── EPW/          # Archivos de clima
└── notebooks/    # Libretas de Jupyter para análisis
```

## Pasos

### 1. Crear geometría

1. Abrir Open Studio → pestaña **Geometry** → **Editor**
2. Seleccionar "New" → opción con mapa (buscar ubicación) o vacío
3. Configurar el **grid** (ej. 1m o 0.5m) en esquina superior
4. Dibujar espacios con herramienta **rectángulo** o **polígono**
5. Cada espacio = un volumen (futura zona térmica)
6. Verificar en **Preview** (3D View) que se ve correcto
7. **Merge with Current OSM** para transferir geometría al modelo

### 2. Configurar Stories (pisos)

1. En el editor, verificar la altura **floor-to-ceiling** (default: 2.43m)
2. Cambiar si es necesario (ej. 3m)
3. Renombrar el story (ej. "PB" para planta baja)

### 3. Guardar primera versión

1. **File → Save As** → carpeta OSM/
2. Nombre: `001_descripcion.osm` (ej. `001_cubo_volumetria`)
3. Open Studio crea una carpeta asociada — no guardar cosas ahí

### 4. Crear zonas térmicas

1. Pestaña **Thermal Zones** → botón verde (+) para agregar zonas
2. Nombrar descriptivamente: Norte, Sur, Cocina, etc. (una palabra, sin espacios)
3. Pestaña **Spaces** → asignar cada espacio a su zona térmica (columna Thermal Zone)

### 5. Verificar condiciones de frontera

1. En **3D View** → **Render by Boundary**
2. Verificar que:
   - Superficies exteriores = **azul** (Outdoor)
   - Superficies compartidas entre zonas = **verde** (Surface)
   - Pisos = cambiar de Ground a **Adiabatic** (rojo) si es simplificación del curso
3. Si hay errores, corregir manualmente en la tabla de superficies

### 6. Definir materiales

1. Pestaña de materiales (icono ladrillos) → **Materials** (NO "No Mass")
2. Botón (+) para crear material
3. Definir: roughness, thickness, conductivity, density, specific heat, thermal absorptance, solar absorptance, visible absorptance
4. Nombrar descriptivamente (ej. `CAD_15cm_Abs07`)

### 7. Crear sistemas constructivos

1. Misma pestaña → **Constructions**
2. Botón (+) para crear
3. Nombrar descriptivamente
4. Arrastrar materiales desde **My Model** → de exterior a interior

### 8. Asignar construcciones a superficies

1. Pestaña **Spaces** → sub-pestaña **Surfaces**
2. Arrastrar la construcción a la columna **Construction** de cada superficie

### 9. Configurar archivo de clima

1. Pestaña **Site** → **Set Weather File**
2. Seleccionar el archivo EPW de la carpeta EPW/
3. Verificar latitud, longitud, time zone, elevación

### 10. Guardar versión final

1. **File → Save As** → `003_primeraSimulacion.osm` (o número correspondiente)

### 11. Ejecutar simulación

1. Botón **Run** en la parte inferior
2. Esperar barra de progreso hasta 100%
3. Si hay errores → revisar mensajes (ver [[004-InterpretandoMensajesConstructionSets]])

## Consejos

- **Guardar como (Save As)** después de cada paso importante — nunca sobreescribir
- Nunca borrar versiones anteriores
- Si algo falla y no se puede arreglar en 15-30 min → regresar a la versión anterior
- Compartir proyectos como **ZIP** del folder completo (no solo el OSM)
- Las últimas versiones serán las variantes bioclimáticas (cambio de color, materiales, orientación, protecciones)

## Enseñado en

- [[003-MiPrimeraSimulacion]] — Demostración completa paso a paso
