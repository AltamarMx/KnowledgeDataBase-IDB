---
title: Solicitar variables de output con measures
type: procedimiento
tags: [procedimiento, openstudio, output, measures, bcl, rdd]
aliases: [pedir output variables, add output variable, create csv output]
clases: [005]
updated: 2026-05-02
---

# Solicitar variables de output con measures

Procedimiento para pedir a Energy Plus que reporte variables específicas (más allá de las que Open Studio expone en su pestaña Output Variables) usando measures del BCL.

## Por qué measures y no la pestaña Output Variables

La pestaña **Output Variables** de Open Studio permite activar variables, pero:

- Solo expone un **subconjunto** de las variables que E+ puede reportar.
- Variables clave para análisis bioclimático **no están** ahí: por ejemplo `Surface Outside Face Incident Solar Radiation Rate per Area`.
- Por eso, para acceso completo, se usan measures.

Las variables que sí están en la pestaña pueden activarse desde ahí — pero mezclar las dos vías genera confusión. **El profesor recomienda hacer todo por measures** y dejar la pestaña en sus defaults.

## Pre-requisitos

- Una simulación que ya corrió al menos una vez (para tener el RDD).
- Saber qué variables se quieren — leer el RDD primero para tener los nombres exactos. Ver [[../concepts/RDD-Variables-Disponibles]].

## 1. Descargar los measures del BCL

Una sola vez por instalación de Open Studio:

1. Pestaña **Measures** → en la parte inferior derecha **Find Measures on BCL** (Building Component Library).
2. En el panel del BCL:
   - Categoría **Reporting → QAQC**.
   - Buscar y seleccionar los siguientes:
     - **`Add Output Variable`** — pide una variable al output.
     - **`Create CSV Output`** — exporta los outputs a CSV legible.
3. Marcar la casilla de cada uno → **Download**.
4. Open Studio descarga e instala. Si ya tenías versiones anteriores, ofrecerá actualizar.

> Si Open Studio se siente lento o congelado durante la descarga, esperar — la GUI tarda.

## 2. Leer el RDD para conseguir el nombre exacto

Antes de configurar el measure, identificar el **nombre exacto** de la variable:

1. Tras la primera simulación, **Show Simulation** → carpeta `run/`.
2. Abrir `eplusout.rdd` con un editor de texto.
3. Buscar (`Ctrl+F`) por keyword: `temperature`, `radiation`, `incident`, etc.
4. Copiar el nombre exacto: `Site Outdoor Air Drybulb Temperature` (sin coma final, sin espacios extra, respetando mayúsculas).

Detalle del RDD en [[../concepts/RDD-Variables-Disponibles]].

## 3. Agregar el measure `Add Output Variable` (uno por variable)

En la pestaña **Measures** del modelo:

1. Sección **Reporting Measures** (no Open Studio Measures, no Energy Plus Measures).
2. Arrastrar el measure **`Add Output Variable`** desde el panel **My Library** al primer slot.
3. Si el measure aparece con un icono de warning, es que falta configurarlo.
4. Click en el measure → se abre un panel con campos:
   - **Variable Name** (descriptivo, libre — usar el mismo que la variable para no confundirse).
   - **Variable Name** (esto es el de E+) → pegar el nombre exacto del RDD.
   - **Reporting Frequency** → seleccionar `Timestep`.
   - **Key Value** → `*` para todas las superficies/zonas, o el **nombre específico** de la superficie/zona.

> **Recomendación de frecuencia**: `Timestep` (paso temporal de simulación, default 6/hora). Con `Hourly` se pierde resolución y con `Detailed` se generan demasiados datos.

5. **Repetir** el paso 2-4 por cada variable que se quiera. Cada variable necesita su **propio** measure `Add Output Variable`.

### Ejemplo: variables del primer análisis (clase 005)

| # | Measure | Variable Name | Key Value |
|---|---------|---------------|-----------|
| 1 | Add Output Variable | `Site Outdoor Air Drybulb Temperature` | `*` |
| 2 | Add Output Variable | `Site Direct Solar Radiation Rate per Area` | `*` |
| 3 | Add Output Variable | `Site Diffuse Solar Radiation Rate per Area` | `*` |
| 4 | Add Output Variable | `Zone Mean Air Temperature` | `*` |
| 5 | Add Output Variable | `Zone Operative Temperature` | `*` |
| 6 | Add Output Variable | `Surface Outside Face Incident Solar Radiation Rate per Area` | `Techo` (nombre específico) |

## 4. Agregar el measure `Create CSV Output`

Solo uno (no por variable):

1. Sección **Reporting Measures** del mismo panel.
2. Arrastrar **`Create CSV Output`** al final.
3. Configurar:
   - **Reporting Frequency** → `Timestep` (debe coincidir con la de Add Output Variable; si difieren, falla).

> El CSV es opcional para el análisis (porque `ear_tools` lee directo del SQL — ver [[../tools/ear-tools]]). Pero **conviene tenerlo** como verificación visual rápida: si no tiene la columna esperada, algo en la configuración falló.

## 5. Asignar nombres descriptivos a superficies (cuando se usa Key Value específico)

Si pides una variable de Surface con un Key Value específico (no `*`):

1. Pestaña **Spaces → Surfaces**.
2. Localizar la fila correspondiente (puedes identificarla por tipo + condición + área).
3. Columna **Name** → escribir un nombre descriptivo: `Techo`, `MuroNorte`, `MuroSur`.
4. Sin acentos, sin eñes, sin espacios.

> Alternativa: usar el preview 3D para identificar visualmente la superficie y luego copiar el nombre que Open Studio le asignó por default (algo tipo `Face 5`).

## 6. Run

`Run Simulation → Run`. El log de la simulación debe mostrar:

```
Processing OpenStudio Measures
   ... add_output_variable
   ... add_output_variable
   ...
Processing Reporting Measures
   ... create_csv_output
```

Si un measure no se procesa (por nombre incorrecto, por ejemplo), aparece un error en la pestaña Output. Comparar contra el `.err`.

## 7. Verificar

### En el RDD post-simulación

Abrir `eplusout.rdd` de nuevo. Debe seguir mostrando todas las variables (el RDD es general, no afectado por las solicitudes).

### En el SQL / CSV

- Abrir el `eplusout.csv` (en el folder generado por el measure de CSV — ej. `run/004_create_csv_output/`) → confirmar que las columnas corresponden a lo solicitado.
- O bien, en Python: `read_sql(file).data.columns` → confirmar que están todas.

Si una columna falta:

- Verificar que el nombre de la variable es **idéntico** al RDD (un espacio extra, una mayúscula equivocada, una coma sobrante → el measure falla silenciosamente).
- Verificar que el Key Value (si específico) es exactamente el nombre de la superficie/zona.

## Trampas comunes

| Síntoma | Causa probable |
|---------|----------------|
| Variable no aparece en CSV/SQL | Nombre con typo, mayúscula equivocada, o coma final residual |
| Frecuencias mezcladas (Timestep en variables, Hourly en CSV) | El CSV falla — ajustar a misma frecuencia |
| Folder `run/00X_addOutputVariable/` cambia de número | Normal: depende de cuántos measures hay antes. **No** apuntar libretas a esos paths — usar el SQL en `run/eplusout.sql` (estable) |
| Open Studio no descargó los measures | Reintentar en Find Measures on BCL; verificar conexión a internet |

## Cómo afecta esto al folder de outputs

Cada Reporting Measure crea su propio sub-folder en `run/` con un número en orden de ejecución (`004_addOutputVariable`, `005_addOutputVariable`, …). Por eso `ear_tools` lee del SQL — el SQL **no se mueve** y siempre está en `run/eplusout.sql`. Detalle en [[../concepts/Salida-SQL-EnergyPlus]].

## Clases relacionadas

- [[../classes/005-AnalisisSimulacionesPython]] — demo en vivo del flujo
