---
title: Configurar aire acondicionado ideal en Open Studio
type: procedimiento
tags: [procedimiento, openstudio, hvac, ideal-air-loads, setpoint, schedule]
aliases: [activar AC ideal, ideal air loads, configurar HVAC, thermostat]
clases: [009]
updated: 2026-05-02
---

# Configurar aire acondicionado ideal en Open Studio

Procedimiento para activar **Ideal Air Loads** en una zona térmica y asignarle setpoints de heating y cooling. Es la vía más sencilla de incorporar HVAC al modelo. Detalle del concepto en [[../concepts/Aire-Acondicionado-Ideal]].

## Pre-requisitos

- Modelo con al menos una zona térmica funcionando sin AC.
- Si vas a comparar caso base vs caso con AC: **siempre desde Save As del caso base** ([[../concepts/Caso-Base]]).

## 1. Crear los schedules de setpoint

Antes de configurar el termostato, hay que tener los schedules de heating y cooling listos. Procedimiento detallado en [[Crear-Schedule-Temperatura]].

Para los tres modos típicos:

| Modo | Heating schedule | Cooling schedule |
|------|------------------|-------------------|
| **T constante a 20 °C** | `setpoint_20C` (= 20) | `setpoint_20C` (mismo) |
| **Banda 20-25 °C** | `setpoint_heating_20C` (= 20) | `setpoint_cooling_25C` (= 25) |
| **Solo enfriar** | `heating_disabled_-1C` (= −1) | `setpoint_cooling_25C` (= 25) |
| **Solo calentar** | `setpoint_heating_20C` (= 20) | `cooling_disabled_50C` (= 50) |

Tip: crear los schedules como constantes (un solo valor todo el día) para los primeros experimentos. Después se pueden enriquecer con horarios variables.

## 2. Activar Ideal Air Loads en la zona térmica

1. Pestaña **Thermal Zones** en la columna izquierda.
2. Localizar la fila de la zona donde quieres AC.
3. Columna **Turn On Ideal Air Loads** → marcar la casilla.

Resultado: la zona ahora tiene un sistema de HVAC ideal que actúa según los setpoints que se le asignen.

## 3. Asignar el termostato y los setpoints

En la misma pestaña Thermal Zones, en la fila de la zona:

1. **Heating Schedule** → arrastrar el schedule de heating desde **My Model** (panel derecho) o seleccionar del menú.
2. **Cooling Schedule** → arrastrar el de cooling.

> Si los schedules no aparecen en el menú: verificar que tienen `Schedule Type: Temperature` (no Dimensionless ni otro). Open Studio filtra por tipo.

## 4. Pedir variables de output del AC

Configurar measures de output ([[Solicitar-Output-Variables-Measures]]) para las variables clave:

| Variable | Para qué |
|----------|----------|
| `Zone Mean Air Temperature` | Verificar que la T se mantiene en/cerca del setpoint |
| `Zone Ideal Loads Zone Total Cooling Energy` | Energía total de enfriamiento (J) — usar para integrar mensual |
| `Zone Ideal Loads Zone Total Heating Energy` | Análoga para calefacción |
| `Zone Ideal Loads Zone Sensible Cooling Rate` | Potencia instantánea de cooling (W) — para series temporales |
| `Zone Thermostat Cooling Setpoint Temperature` | Verificar que el schedule llegó al IDF |
| `Zone Thermostat Heating Setpoint Temperature` | Análoga |

Detalle del catálogo en [[../concepts/Variables-Output-EnergyPlus]].

## 5. Guardar y correr

1. **File → Save As** → nuevo número de versión (ej. `007_caso_base_aa.osm`).
2. **Run Simulation → Run**.
3. Show Simulation → revisar `.err` (cero severes esperados).

## 6. Validar el comportamiento

### Caso T constante (heating = cooling = 20)

Esperado en la T de la zona:

- Línea **constante a 20 °C** durante todo el año.
- Sin oscilación.

Si oscila: el termostato no se aplicó correctamente. Verificar que `Turn On Ideal Air Loads` está marcado y los schedules están asignados.

### Caso banda de confort (heating = 20, cooling = 25)

Esperado en la T de la zona:

- **Flota libremente entre 20 y 25 °C** cuando el clima exterior está en ese rango.
- Cuando el exterior es muy frío → la T se queda **pegada en 20 °C** (calentando).
- Cuando el exterior es muy caliente → la T se queda **pegada en 25 °C** (enfriando).

Validación con plot:

```python
fig, ax = plt.subplots(figsize=(12, 4))
ax.plot(df.T_este, label="T zona")
ax.plot(df.TO,     label="T exterior")
ax.axhline(20, color="red", linestyle="--", label="heating SP")
ax.axhline(25, color="blue", linestyle="--", label="cooling SP")
ax.legend()
```

### Caso solo enfriar (heating = −1, cooling = 25)

Esperado:

- T puede caer **debajo de 20** sin que actúe el calentamiento (clima frío).
- Cuando el exterior es caliente → la T se queda en 25 °C.

Verificación numérica: la energía de heating debe ser **0 J** todo el año.

```python
total_heating = df["heating_energy_J"].sum()
print(f"Energía heating total: {total_heating:.0f} J")
# Esperado: 0
```

## 7. Análisis de energía mensual

Patrón típico para reportar consumo:

```python
import pandas as pd

# Energía mensual sumada
energia_mes = df["cooling_energy_J"].resample("ME").sum()

# Convertir a kWh para legibilidad
energia_mes_kWh = energia_mes / 3.6e6

# Gráfica de barras
fig, ax = plt.subplots(figsize=(10, 4))
ax.bar(range(1, 13), energia_mes_kWh.values)
ax.set_xticks(range(1, 13))
ax.set_xticklabels(["E", "F", "M", "A", "M", "J", "J", "A", "S", "O", "N", "D"])
ax.set_ylabel("Energía cooling (kWh)")
```

Detalle del patrón en [[Analizar-Resultados-Python]].

## 8. Sanity check

| Síntoma | Causa probable |
|---------|----------------|
| T zona sale del rango setpoint | `Ideal Air Loads` no está marcado, o schedule sin asignar |
| Energía cooling = 0 todo el año | Setpoint cooling muy alto (ej. 50 °C nunca alcanzado) |
| Energía heating = 0 todo el año | Setpoint heating muy bajo (ej. −1 °C nunca alcanzado) — **esperado** en modo "solo enfriar" |
| T zona constante todo el año | heating = cooling — modo T constante |
| Severe `Schedule X not found` | El schedule no se asignó correctamente o tiene typo |
| `.err` con setpoints faltantes | Una zona con AC activo no tiene heating o cooling schedule asignado |

## Comparación caso base sin AC vs caso con AC

Para evaluar **cuánta energía requeriría** una edificación si se le pusiera AC:

1. Caso base sin AC → reporta T del aire interior.
2. Caso base con AC banda de confort → reporta energía consumida.

Comparar T máxima sin AC con la T máxima en el cooling setpoint del caso con AC.

> Recordar: como anuncia la clase 009, **el proyecto final del taller no incluye AC** — son demasiadas simulaciones y las mejores estrategias para AC son distintas a las de sin AC. Pero saber simular AC es útil para casos profesionales y para la siguiente materia.

## Clases relacionadas

- [[../classes/009-AireAcondicionadoSetPoints]] — demo en vivo del flujo y de los tres modos

## Procedimientos relacionados

- [[Crear-Schedule-Temperatura]] — pre-requisito
- [[Solicitar-Output-Variables-Measures]] — variables de AC
- [[Analizar-Resultados-Python]] — análisis de energía mensual
