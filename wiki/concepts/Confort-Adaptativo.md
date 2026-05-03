---
title: Confort Adaptativo (Humphreys-Nicol)
type: concepto
tags: [concepto, confort, adaptativo, humphreys, nicol, bioclimatico]
aliases: [adaptive comfort, modelo adaptativo, humphreys nicol, confort adaptativo]
clases: [005, 008, 009, 010]
updated: 2026-05-02
---

# Confort Adaptativo (Humphreys-Nicol)

## Premisa

Las personas **se adaptan al clima** en el que viven. Una persona en Cuernavaca tolera bien 28 °C interior; una en Toronto pediría AC. Los modelos clásicos de confort (PMV-PPD de Fanger) ignoran este efecto y predicen una zona de confort fija para todo el mundo, lo que sobreestima el AC requerido en climas templados-cálidos.

Los **modelos adaptativos** corrigen esto haciendo la zona de confort **función del clima reciente**, no un valor fijo.

## Ecuación de Humphreys-Nicol

La forma clásica expresa la **temperatura de neutralidad** (T donde la persona se siente confortable) como función de la T exterior promedio mensual:

$$
T_{neutralidad} = 0.54 \cdot \overline{T}_{out,mes} + 13.5
$$

donde $\overline{T}_{out,mes}$ es la temperatura **promedio mensual** del exterior.

> "Yo sé que ustedes ya están convencidos que Python y Pandas es la opción. Aprendan a usar `resample` mensual: `df['T_out'].resample('ME').mean()`."

### Interpretación

| $\overline{T}_{out}$ del mes | $T_{neutralidad}$ |
|------------------------------|----------------------|
| 10 °C (invierno templado) | 18.9 °C |
| 20 °C (templado) | 24.3 °C |
| 25 °C (Cuernavaca) | 27.0 °C |
| 30 °C (Mérida) | 29.7 °C |

A más calor exterior promedio, más alto el setpoint de neutralidad — la gente se adapta.

## Zona de confort

Alrededor de $T_{neutralidad}$ se define una banda **± ΔT** dentro de la cual se considera confortable:

$$
T_{neutralidad} - \Delta T \leq T_{conf} \leq T_{neutralidad} + \Delta T
$$

ΔT depende del modelo y del **porcentaje de aceptabilidad**:

| Modelo | Aceptabilidad | ΔT típico |
|--------|---------------|-----------|
| ASHRAE 55 adaptativo | 80% | 3.5 °C |
| ASHRAE 55 adaptativo | 90% | 2.5 °C |
| Humphreys-Nicol | 80% | ~3.5 °C |

## Variable a evaluar

Los modelos adaptativos se aplican sobre la **[[Temperatura-Operativa]]** ($T_{op}$), no sobre $T_{aire}$ pura — porque la persona percibe radiación además del aire. Cuando hay fuentes radiantes importantes (sol incidente, superficies muy calientes), reportar confort sobre $T_{aire}$ subestima el disconfort.

En la práctica, si no hay fuentes radiantes asimétricas $T_{op} \approx T_{aire}$ y se usa la disponible.

## Métricas de evaluación

Una sola métrica casi nunca cuenta toda la historia. Anti-patrones:

| Métrica única | Qué pierde |
|---------------|-----------|
| **Promedio anual** | La oscilación |
| **Amplitud** | El centro |
| **Máximo / Mínimo** | El tiempo |
| **% del año en confort** | La magnitud de la salida |

### Tiempo fuera de confort

Fracción del año en que $T_{op}$ cae fuera de la banda. Útil como porcentaje (% del año confortable / disconfortable). Útil para audiencia no técnica.

### Grados-hora de disconfort (recomendado)

Métrica principal del taller — combina **tiempo + magnitud** integrando las desviaciones de la zona de confort. Detalle completo en [[Grados-Hora-Disconfort]].

$$
GH_{cálido} = \sum_t \max(T_{op}(t) - (T_{neutralidad,mes(t)} + \Delta T), 0) \cdot \Delta t
$$

$$
GH_{frío} = \sum_t \max((T_{neutralidad,mes(t)} - \Delta T) - T_{op}(t), 0) \cdot \Delta t
$$

donde $\Delta t$ es el paso temporal en horas. Unidades: °C·h.

> Las grados-hora son la métrica que el grupo del IER usa principalmente para evaluar estrategias bioclimáticas — comparar caso base vs. variantes en términos de $GH_{cálido}$ y $GH_{frío}$ acumulados al año.

> **Reportar SIEMPRE separados cálido y frío** — una estrategia que reduce GH cálido suele aumentar GH frío. Sumarlos en una sola métrica oculta el trade-off.

## Por qué adaptativo en lugar de PMV

| Modelo PMV (ASHRAE 55 estático) | Modelo adaptativo |
|----------------------------------|-------------------|
| Zona de confort fija (~22-26 °C todo el año) | Zona depende del clima reciente |
| Asume HVAC controlado | Aplicable a edificaciones **naturalmente ventiladas** |
| Subestima tolerancia en climas cálidos | Refleja adaptación cultural y fisiológica |
| Sobreestima necesidad de AC | Permite diseño bioclimático sin HVAC |

Para el taller — donde el objetivo es **diseño bioclimático sin AC** — el modelo adaptativo es el natural.

## Setpoint óptimo desde el modelo adaptativo

Cuando se incorpora HVAC al modelo, el [[Setpoint]] del termostato se puede colocar en el **límite superior de la banda de confort**:

$$
T_{cooling, óptimo} = T_{neutralidad,mes} + \Delta T
$$

Comparado con un setpoint fijo a 22 °C (común en oficinas), el adaptativo:

- **Ahorra energía** — el AC arranca a una T más alta.
- **Mantiene el confort** porque la gente está adaptada al clima local.
- Permite cambiar el setpoint **mes a mes** según la T del clima reciente.

Ejemplo Cuernavaca:

| Mes | $\overline{T}_{out,mes}$ | $T_{cooling, óptimo}$ |
|-----|--------------------------|------------------------|
| Enero | 19 °C | 27.3 °C |
| Mayo | 24 °C | 30.0 °C |

> Caso real (anécdota Cool Biz Japón): subir el setpoint relajando dress codes ahorra mucha energía sin sacrificar confort. Detalle en [[Setpoint]].

## Cálculo en Python (flujo)

1. Cargar el EPW con `read_epw(file, year=2006, alias=True)` — ver [[../tools/iertools]].
2. Calcular T promedio mensual:
   ```python
   T_mes = epw['TO'].resample('ME').mean()
   T_neut = 0.54 * T_mes + 13.5
   ```
3. Cargar la simulación con `read_sql(file, alias=True)`.
4. Para cada paso temporal de la simulación, usar el `T_neut` del mes correspondiente.
5. Calcular grados-hora cálidos / fríos.

Detalle del flujo en [[../procedures/EDA-Archivo-EPW]].

## En EnerHabitat

[[../tools/EnerHabitat]] usa el modelo Humphreys-Nicol para mostrar la **zona de confort sombreada** en sus gráficas — lo calcula con la T promedio del mes seleccionado del EPW. Reportar grados-hora de disconfort no está integrado nativamente; hay que exportar las series y calcular en Python ([[EDA-Archivo-EPW]]).

## Climate Consultant — alternativa

Hay una herramienta GUI llamada **Climate Consultant** que aplica modelos adaptativos automáticamente y sugiere estrategias bioclimáticas sobre el EPW. Pros: cubre varios modelos, aplica reglas. Cons (según el profesor): "las gráficas son horrorosas, juntan demasiada información, no se pueden modificar". Por eso en el curso se hace todo en Python.

## Clases relacionadas

- [[../classes/005-AnalisisSimulacionesPython]] — introducción a la ecuación de Humphreys-Nicol y al cálculo mensual de T de neutralidad
- [[../classes/008-ShadingVentanas]] — métricas de evaluación, anti-patrones, grados-hora como métrica principal
- [[../classes/009-AireAcondicionadoSetPoints]] — setpoint óptimo desde el modelo adaptativo
- [[../classes/010-EnerHabitatParte1]] — uso del modelo en EnerHabitat para visualizar zona de confort

## Ver también

- [[Confort-Termico]] — concepto general
- [[Temperatura-Operativa]] — variable sobre la que se evalúa
- [[TMY]] — el clima de input al cálculo
