# Configurar Variables de Salida en Open Studio

Procedimiento para solicitar variables de salida específicas de EnergyPlus usando measures en Open Studio.

## Requisitos previos

- Simulación ya corrida al menos una vez (para generar el archivo RDD)
- Acceso a internet (para descargar measures del BCL la primera vez)

## Paso a paso

### 1. Identificar variables disponibles (archivo RDD)

1. Correr la simulación en Open Studio
2. Ir a **Show Simulation** → abre la carpeta `run/`
3. Abrir el archivo **`.rdd`** (Report Data Dictionary) con un editor de texto
4. Buscar las variables deseadas (ej. `Zone Mean Air Temperature`, `Site Outdoor Air Drybulb Temperature`)
5. **Copiar el nombre exacto** — cualquier error de mayúsculas, espacios o comas impedirá que funcione

### 2. Descargar measures del BCL (solo la primera vez)

1. En Open Studio, esquina inferior derecha: **Find Measures on BCL**
2. Categoría **Reporting** → buscar:
   - `Add Output Variable` — para solicitar variables
   - `Create CSV Output` — para exportar resultados a CSV
3. Seleccionar ambos y descargar

### 3. Agregar measure "Add Output Variable"

1. En Open Studio, pestaña **Measures** → sección **Energy Plus Measures**
2. Desde Library, arrastrar `Add Output Variable` al área de trabajo
3. Configurar:
   - **Variable Name:** pegar nombre exacto del RDD (sin la coma final)
   - **Reporting Frequency:** `timestep` (recomendado para máxima resolución)
   - **Key Value:** `*` para todas las zonas/superficies, o el nombre específico de una zona/superficie
4. **Repetir** por cada variable que se quiera solicitar (un measure por variable)

### 4. Agregar measure "Create CSV Output" (opcional pero recomendado)

1. En sección **Reporting Measures**, arrastrar `Create CSV Output`
2. Asegurar que la frecuencia coincida con la del `Add Output Variable` (ambos en `timestep`)
3. Esto genera un CSV para verificación rápida de que las variables se exportaron correctamente

### 5. Correr la simulación

1. Ejecutar la simulación normalmente
2. Verificar en la consola que los measures se procesaron correctamente
3. En **Show Simulation** → verificar que aparece la carpeta del measure con el CSV

### 6. Verificar resultados

- Abrir el CSV y confirmar que están todas las variables solicitadas
- El archivo `run/eplusout.sql` contiene los mismos datos en formato SQL (preferido para análisis con Python)

## Errores comunes

- **Nombre de variable mal escrito** — copiar siempre del RDD, no escribir de memoria
- **Frecuencias distintas** entre Add Output Variable y Create CSV Output → valores NaN
- **Acentos, eñes o espacios en la ruta** del proyecto → EnergyPlus no puede escribir los archivos de salida
- **Measure desactualizado** → verificar en la consola; si falla, buscar alternativa en BCL

## Nota sobre el SQL

Cada vez que se agrega o quita un measure, el nombre de la carpeta de salida del CSV cambia (ej. `001/` → `005/`), lo que rompe rutas en notebooks. El archivo `run/eplusout.sql` siempre mantiene la misma ruta, por lo que es más robusto para análisis automatizado.

## Aparece en

- [[005-AnalisisSimulacionesPython]] — Demostración completa del procedimiento
