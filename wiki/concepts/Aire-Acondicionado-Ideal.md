---
title: Aire Acondicionado Ideal (Ideal Air Loads)
type: concepto
tags: [concepto, hvac, aire-acondicionado, energyplus, ideal-loads]
aliases: [ideal air loads, ideal hvac, ac ideal, mini-split, hvac]
clases: [009]
updated: 2026-05-02
---

# Aire Acondicionado Ideal (Ideal Air Loads)

## Qué es

Modelo simplificado de HVAC en Energy Plus (`HVACTemplate:Zone:IdealLoadsAirSystem` o el switch en Open Studio) que **proporciona la energía exacta necesaria** para mantener una zona dentro de los setpoints de heating/cooling, asumiendo **eficiencia 100%**:

| Característica | Valor en Ideal Air Loads |
|----------------|---------------------------|
| **Eficiencia** | 100% (1 W consumido = 1 W entregado) |
| **Capacidad** | Sin límite — provee tanta energía como se le pida |
| **Ductos** | No se modelan |
| **Ventilador** | No se modela |
| **Recirculación** | Solo aire interior — equivalente a un mini-split |
| **Renovación** | Cero por default; configurable como `Outdoor Air` |

> "Si yo le pido 1000 unidades de energía, me va a dar 1000. Si le pido 10,000, me va a dar 10,000. En la realidad eso no sucede así — los aires acondicionados tienen capacidad pico."

## Por qué usarlo en el taller

- **Suficiente para evaluar el efecto bioclimático**: la diferencia en energía entre dos diseños se mantiene aunque la eficiencia sea 100% — los números absolutos cambian, pero el **orden y la magnitud relativa** se conservan.
- **Sin curva de aprendizaje del HVAC**: no hay que dimensionar capacidades, ductos ni ventiladores.
- **Compatible con todas las zonas**: cualquier zona térmica puede activar Ideal Air Loads desde la pestaña Thermal Zones.

> "Yo no sé simular aires acondicionados reales. Y es muy claro porque mis intereses no son industriales."

## Cómo se activa

En Open Studio, pestaña **Thermal Zones**:

1. Localizar la zona donde se quiere AC.
2. Columna **Turn On Ideal Air Loads** → marcar.
3. Definir setpoints (heating y cooling) — ambos requieren un [[Schedules|schedule]] de tipo Temperature. Ver [[Setpoint]].

Procedimiento detallado en [[../procedures/Configurar-Aire-Acondicionado-Ideal]].

## Ambos setpoints son obligatorios

E+ exige un **heating schedule** y un **cooling schedule** — no se puede dejar vacío uno. Tres modos de uso:

| Modo | Heating | Cooling | Comportamiento |
|------|---------|---------|----------------|
| **T constante** | 20 °C | 20 °C | Mantiene exactamente 20 °C — calienta o enfría como sea necesario |
| **Banda de confort** | 20 °C | 25 °C | Flota libre entre 20-25 °C, calienta abajo de 20, enfría arriba de 25 |
| **Solo enfriar (México típico)** | −1 °C | 25 °C | Nunca calienta (en clima de Cuernavaca no llega a −1 °C); enfría arriba de 25 °C |
| **Solo calentar** | 20 °C | 50 °C | Calienta abajo de 20; nunca enfría (no llega a 50 °C) |

Detalle del truco de los valores extremos en [[Setpoint]].

## Comparación con HVAC real

Un HVAC real en E+ requiere:

- **Air Loops** — conducciones de aire con ductos.
- **Equipos**: enfriadores (cooling coils), calentadores (heating coils), ventiladores.
- **Controles**: termostatos, schedules de operación, lógica de dimensionamiento.
- **Compatibilidad** con [[Calculo-Sombramientos|Airflow Network]] en algunos casos.

Esto se cubre en la siguiente materia (Energía en Edificaciones), no en el taller. La [[Caricatura-Computacional|caricatura]] del Ideal Air Loads es suficiente para el alcance bioclimático.

## Variables de output asociadas

Disponibles en el RDD una vez que `Ideal Air Loads` está activo:

| Variable | Significado | Unidades |
|----------|-------------|----------|
| `Zone Ideal Loads Zone Total Cooling Energy` | Energía total enfriadora (sensible + latente) por paso | J |
| `Zone Ideal Loads Zone Total Heating Energy` | Análoga para calefacción | J |
| `Zone Ideal Loads Zone Sensible Cooling Rate` | Potencia de enfriamiento sensible | W |
| `Zone Ideal Loads Zone Sensible Heating Rate` | Potencia de calefacción sensible | W |
| `Zone Ideal Loads Supply Air Total Cooling Energy` | Energía suministrada por el AC (puede diferir si hay otros equipos) | J |

Distinción importante:

- **Total** vs **Sensible**: el sensible considera solo cambio de temperatura; el total incluye latente (cambio de humedad).
- **Energy** (J) vs **Rate** (W): para series temporales típicamente conviene **Rate** (no acumula). Para integrales mensuales usar **Energy** y `resample("ME").sum()`.

Detalle en [[Variables-Output-EnergyPlus]].

## Por qué no se incluye AC en el proyecto final del taller

Si el proyecto final exigiera AC, las simulaciones se duplican: 5 (sin AC) → 10 (con AC). Además **las mejores estrategias bioclimáticas para AC no son las mismas que para sin AC**:

| Estrategia | Mejor sin AC | Mejor con AC |
|------------|--------------|--------------|
| Aislamiento térmico | Variable — puede atrapar calor en climas cálidos sin AC | **Sí**, casi siempre — reduce carga térmica |
| Masa térmica | **Sí** — atenúa picos diurnos | Variable — puede acumular calor que el AC tiene que disipar |
| Ventilación nocturna | **Sí** — disipa calor acumulado | No relevante (el AC está encendido) |

> "Las mejores estrategias bioclimáticas para aire acondicionado no son las mejores para sin aire acondicionado. Por eso el grupo del IER se enfoca en sin AC, donde la vivienda social mexicana realmente vive."

## Crítica a NOM-008 / NOM-020

Pre-reforma, las normativas mexicanas exigían aislamiento térmico **independientemente** de si la edificación usa AC. Esto:

- En climas cálidos **sin AC**, atrapa el calor generado al interior (cargas internas + radiación que entra por ventanas) → puede **empeorar** el confort.
- Después la reforma agregó "para climas cálidos con AC, para ahorrar energía de enfriamiento" — pero la versión original era un absurdo.

Ver detalle en [[Posicion-Aislante]].

## Sistemas geotérmicos (mención)

Mencionado de paso: E+ puede simular **tubos enterrados** (geothermal heat exchangers) verticales u horizontales. El modelo necesita la **temperatura del suelo a profundidad**, que depende del material (roca, lodo, humedad). Recomendación: medir, no asumir.

> "En la oficina del nuevo edificio del IER hay datos a 1.8 m de profundidad — eventualmente se podrán usar para simular geotermia."

Fuera del alcance del taller.

## Clases relacionadas

- [[../classes/009-AireAcondicionadoSetPoints]] — introducción al concepto y demo en vivo de los tres modos
