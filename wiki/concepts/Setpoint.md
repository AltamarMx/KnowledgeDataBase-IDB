---
title: Setpoint (Heating / Cooling)
type: concepto
tags: [concepto, hvac, setpoint, thermostat, schedule, energyplus]
aliases: [setpoint, thermostat, heating setpoint, cooling setpoint, banda de confort]
clases: [009]
updated: 2026-05-02
---

# Setpoint (Heating / Cooling)

## Qué es

Temperatura objetivo a la que debe operar un sistema de [[Aire-Acondicionado-Ideal|HVAC]]. Energy Plus distingue **dos setpoints** que actúan asimétricamente:

| Setpoint | Acción si T zona < setpoint | Acción si T zona > setpoint |
|----------|------------------------------|------------------------------|
| **Heating** | Calentar hasta llegar al setpoint | Nada |
| **Cooling** | Nada | Enfriar hasta llegar al setpoint |

Cada setpoint requiere un [[Schedules|schedule]] de tipo Temperature — pueden ser constantes (un valor todo el año) o variar por hora/día/temporada.

## Tres modos de uso

### 1. T constante (heating = cooling)

Heating = 20 °C, Cooling = 20 °C.

Resultado: la zona se mantiene **exactamente a 20 °C** todo el tiempo. El AC calienta o enfría según sea necesario.

Aplicación: museos, laboratorios, espacios con control estricto.

> Ejemplo: el **MUAC** (Museo Universitario de Arte Contemporáneo de la UNAM) mantiene sus 9 salas a 20 °C ± humedad específica para preservar las obras. En climas fríos, requiere calefacción además del AC.

### 2. Banda de confort (heating < cooling)

Heating = 20 °C, Cooling = 25 °C.

Resultado: la zona **flota libre** entre 20 y 25 °C. El AC solo actúa cuando la T sale de la banda.

Aplicación: oficinas, residencias — la mayoría de los casos. Compatible con **modelos adaptativos de confort** ([[Confort-Adaptativo]]).

### 3. Solo enfriar (caso típico de México)

Heating = −1 °C (o cualquier valor que nunca se alcance), Cooling = 25 °C.

Resultado: la zona se enfría arriba de 25 °C; nunca se calienta (la T del aire no baja a −1 °C en climas mexicanos).

Aplicación: vivienda en climas cálidos donde no hay calefacción.

> "En México casi nunca hay sistemas de calefacción. ¿Cómo engaño a Energy Plus? Pongo el heating setpoint en −1 °C — nunca va a llegar."

### Caso simétrico — solo calentar

Heating = 20 °C, Cooling = 50 °C.

Resultado: calienta abajo de 20 °C; nunca enfría.

Aplicación: vivienda en climas fríos sin AC.

## Por qué E+ requiere ambos schedules

El objeto de termostato (`HVACTemplate:Thermostat` o `ThermostatSetpoint:DualSetpoint`) **siempre** requiere los dos schedules. **No** se puede dejar uno vacío:

- Si se dejan **ambos** vacíos → severe en `.err`.
- Si se ponen **iguales** → T constante (modo 1).
- **No hay forma "oficial" de deshabilitar uno** — el truco de los valores extremos es el workaround estándar.

## Setpoint óptimo desde modelo adaptativo

> "¿Cuál es mi límite superior de confort? Lo puedo calcular con el modelo de Humphreys-Nicol."

Para minimizar consumo energético manteniendo confort:

$$
T_{cooling, óptimo} = T_{neutralidad,mes} + \Delta T
$$

donde $\Delta T \approx 3.5$ °C (banda de 80% aceptabilidad).

Ejemplo Cuernavaca:

| Mes | $\overline{T}_{out,mes}$ | $T_{neut}$ | Setpoint cooling sugerido |
|-----|--------------------------|------------|----------------------------|
| Enero | 19 °C | 23.8 °C | 27.3 °C |
| Mayo | 24 °C | 26.5 °C | 30.0 °C |
| Octubre | 21 °C | 24.8 °C | 28.3 °C |

Comparado con un setpoint fijo a 22 °C (común en oficinas), el adaptativo ahorra **mucha energía** sin sacrificar confort percibido. Detalle en [[Confort-Adaptativo]].

> "Pueden ahorrar energía si en lugar de poner el AC a 21 °C lo ponen a 22 o 23. O mejor aún, al límite superior de confort según el modelo."

## Anécdota — Cool Biz Japón

Caso real de política pública: el emperador de Japón decretó que se podía dejar de usar **saco y corbata** en oficinas durante el verano. Esto permitió subir el setpoint de cooling y **ahorrar energía** sin pérdida de confort percibido (la gente se adaptó vistiendo más ligero).

Lección: el setpoint no es solo un parámetro técnico — está acoplado a **convenciones culturales y de vestimenta**. Cambiar la convención cambia el setpoint posible.

> "Las ventajas de tener un emperador, en algunos casos."

### Crítica relacionada — bancos y cines en México

Los bancos exigen manga larga + saco a empleados → AC bajo (18-20 °C) para que esa gente no sufra → el resto de los clientes/empleados con vestimenta normal **tienen frío**.

> "¿Cómo es posible que tengamos que ir a la costa y para entrar al congreso te tengas que poner una chamarra? Es ridículo."

Solución democrática: relajar el dress code → subir el setpoint → ahorrar energía.

## Cómo se configura en Open Studio

Procedimiento detallado en [[../procedures/Configurar-Aire-Acondicionado-Ideal]]. Resumen:

1. Pestaña **Schedules** → crear schedule de tipo **Temperature** para cada setpoint.
2. Pestaña **Thermal Zones** → activar **Ideal Air Loads**.
3. Asignar el heating schedule a `Thermostat Heating Setpoint Schedule`.
4. Asignar el cooling schedule a `Thermostat Cooling Setpoint Schedule`.

## Output relacionado

Variable que reporta el setpoint usado en cada paso (útil para verificar que el schedule llegó al IDF):

```
Zone Thermostat Cooling Setpoint Temperature [C]
Zone Thermostat Heating Setpoint Temperature [C]
```

## Clases relacionadas

- [[../classes/009-AireAcondicionadoSetPoints]] — introducción al concepto y a los tres modos de uso
