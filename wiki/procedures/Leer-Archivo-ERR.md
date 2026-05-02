---
title: Leer el archivo ERR de Energy Plus
type: procedimiento
tags: [procedimiento, debugging, energyplus, openstudio, errores, warnings]
aliases: [leer err, eplusout.err, debug err]
clases: [004]
updated: 2026-05-02
---

# Leer el archivo ERR de Energy Plus

Después de cada `Run`, el primer reflejo debe ser **abrir el archivo `.err`**. Es donde E+ reporta qué pasó durante la simulación: errores fatales, warnings, e información del progreso.

## 1. Abrir la carpeta de outputs

En Open Studio, después de un `Run`:

1. Pestaña **Run Simulation**.
2. Botón **Show Simulation** → abre el folder hermano del OSM en el explorador.
3. Dentro de ese folder, navegar a `run/` (o equivalente según la versión).
4. Localizar el archivo `eplusout.err` (o `<nombre>.err`).

> Si Open Studio no tiene un EPW asignado, **Show Simulation puede fallar** en vez de abrir la carpeta. Cargar el EPW (Site → Set Weather File) y volver a correr.

## 2. Abrir el `.err` con un editor de texto

Doble click puede preguntar con qué programa abrirlo (es un `.err`, sin asociación default):

- **Windows**: elegir **Notepad**.
- **macOS**: **TextEdit** o cualquier editor de código (VS Code).
- **Linux**: cualquier editor de texto.

Es texto plano. **No** abrir con Word.

## 3. Estructura típica del `.err`

```
Program Version,EnergyPlus, Version 22.2.0, ...
   ** Warning ** Weather file location ...
   ** Warning ** GetEnvironmentList: Warmup ...
   ** Severe  ** Construction:Material missing material assignments ...
   **  Fatal  ** GetSurfaceData: Errors discovered, program terminates.
   ...
   ************* EnergyPlus Run Time=00hr 00min 0.50sec
   ************* EnergyPlus Completed Successfully -- 14 Warning; 0 Severe Errors
```

Al final se reporta el **conteo agregado**: cuántos warnings y cuántos severes. Si severes > 0, la simulación abortó.

## 4. Buscar Severes / Fatals primero

```
** Severe  ** ...
**  Fatal  ** ...
```

Cualquier severe **detiene la simulación** — no hay resultados utilizables. Resolver todos antes de mirar warnings.

### Errores severos típicos y fix

| Mensaje | Causa | Fix |
|---------|-------|-----|
| `Construction <X>: missing material assignments` / `outside layer not found` | La construction quedó sin materiales | Constructions → abrir → arrastrar material(es) ext→int |
| `No design environments specified` / `no weather file found` | EPW no asignado o path roto | Site → Set Weather File |
| `Surface <X>: invalid polygon` | Geometría rota (vértices duplicados, polígono no plano) | A veces más rápido **redibujar** que arreglar |
| `Zone <X> has no surfaces` | Zona térmica creada sin asignarse a un espacio | Spaces → asignar Thermal Zone al Space |
| `Construction <X> not found for surface` | El Construction Set no cubre esa combinación tipo+condición | Completar el slot del set, o asignar localmente |

## 5. Revisar Warnings

Después de eliminar severes, los warnings pueden ser docenas. **No todos son problemas reales**. Estrategia:

### 5.1. Agrupar por tipo

Muchos warnings son repeticiones del mismo mensaje (ej. uno por cada superficie sin output específico). Agrupar mentalmente:

- "10 warnings de `Output Variable not found` para variables de Lifecycle Assessment".
- "1 warning de design days no especificados".
- "3 warnings de surfaces with insufficient sun exposure".

### 5.2. Ignorar los del catálogo conocido

| Warning | Por qué aparece | Por qué se ignora en el curso |
|---------|-----------------|--------------------------------|
| `No design days defined` | Open Studio espera días de diseño para HVAC sizing | El curso no dimensiona HVAC |
| `Output:Variable <X> not found` para variables de LCA | Open Studio espera outputs de ciclo de vida | El curso no hace LCA |
| `Site:Energy not specified` | Espera Site/Source factors locales | El curso no calcula consumo neto |
| `EPW does not have design conditions` | Algunos EPW de OneBuilding no traen design days | No los necesita el curso |

### 5.3. Investigar los demás

Los warnings que **no caen** en ese catálogo merecen lectura. Ejemplos donde sí importan:

- `Convergence not achieved during warm-up` — el warm-up se cortó por límite de días, no por convergencia. Resultados pueden estar contaminados.
- `Surface <X> never receives solar radiation` — ¿es esperado? Si es un muro al sur sin sombreamiento, hay un error de orientación.
- `Material <X> property out of typical range` — propiedades térmicas mal definidas.
- `Temperature went below/above limits` — la simulación produjo valores irreales.

## 6. Re-correr e iterar

Tras corregir:

1. Guardar el OSM (`File → Save As` → nuevo número de versión, ver [[Estructura-Proyecto-Simulacion]]).
2. `Run` otra vez.
3. Volver a abrir el `.err`.
4. Repetir hasta tener 0 severes y warnings comprendidos.

## 7. Sanity checks tras pasar el `.err`

Cero severes no garantiza que la simulación sea correcta — solo que E+ pudo terminar. Hacer además:

- **Inspección de geometría** en preview 3D (Render By Surface Type y por Boundary Conditions).
- **Lectura rápida de la temperatura interior**: graficar en pandas y verificar que oscila en rangos plausibles para la ciudad.
- **Comparar contra clima exterior**: la temperatura interior debe oscilar amortiguada respecto a la exterior; si está perfectamente igual, la edificación no tiene masa o está abierta de más; si está plana, hay sobre-amortiguamiento o un setpoint impuesto.

Más en [[Debuggear-Simulacion-OpenStudio]].

## Ejemplo: caso típico de la clase 004

1. Equipo subió tarea — moveron el OSM al folder hermano por accidente → al re-abrirlo perdió la ruta del EPW.
2. `Run` → `Severe: no weather file found`.
3. Fix: Site → Set Weather File.
4. `Run` → corre, pero `Severe: Construction "Cubo": missing material assignments`.
5. Fix: Constructions → la construction estaba vacía → arrastrar el material.
6. `Run` → `Complete with 14 Warnings`.
7. Lectura: 14 warnings, todos del catálogo LCA → seguir.

## Clases relacionadas

- [[../classes/004-InterpretandoMensajesConstructionSets]] — clase donde se demuestra el flujo en vivo con la tarea de un equipo
