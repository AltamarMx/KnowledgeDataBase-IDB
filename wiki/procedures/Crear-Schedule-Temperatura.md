---
title: Crear un schedule de temperatura en Open Studio
type: procedimiento
tags: [procedimiento, openstudio, schedule, temperatura, setpoint, hvac]
aliases: [crear schedule, schedule temperatura, default profile, schedule openstudio]
clases: [009]
updated: 2026-05-02
---

# Crear un schedule de temperatura en Open Studio

Procedimiento para crear un [[../concepts/Schedules|schedule]] de temperatura — típicamente para setpoints de heating o cooling — en la pestaña Schedules de Open Studio.

## Pre-requisitos

- Modelo abierto en Open Studio.
- Si es para AC: planear los valores de heating y cooling antes de crear los schedules. Ver [[../concepts/Setpoint]].

## 1. Ir a la pestaña Schedules

Pestaña **Schedules** en la columna izquierda. Tiene dos sub-pestañas:

- **Schedules** — los schedules individuales (este procedimiento).
- **Schedule Sets** — agrupaciones (análogo a Construction Sets — fuera de este procedimiento).

## 2. Crear un nuevo schedule

1. Sub-pestaña **Schedules**.
2. Botón verde **+** abajo a la derecha.
3. Aparece un diálogo: **Schedule Type**.

## 3. Elegir el tipo de schedule

El tipo determina qué unidades y límites tendrá el schedule. Para setpoints de termostato:

- Categoría: **Temperature** (no `Dimensionless` ni `Activity Level` ni otros).

Open Studio muestra varias variantes — escoger una con `Numeric Type: Continuous` y `Lower Limit Value: non / Upper Limit Value: non` (sin límites — los valores pueden ser cualquier T).

> **Trampa**: el `lower limit` y `upper limit` del **tipo** de schedule **no son** el valor del schedule. Son los límites permitidos. El valor real se define después en la interfaz visual.

## 4. Renombrar el schedule

Default: `Schedule X` (genérico). Renombrar a algo descriptivo:

- `setpoint_cooling_25C` para un setpoint de cooling a 25 °C.
- `setpoint_heating_20C` para heating a 20 °C.
- `heating_disabled_-1C` para el truco de "no calentar nunca".

Sin acentos, sin eñes, sin espacios — convención del taller.

## 5. Editar el Default Profile

La interfaz muestra un panel con una **línea negra horizontal** que representa el valor del schedule a lo largo del día. Por default:

```
   30 °C ┤
         │
         │
   23 °C ━━━━━━━━━━━━━━━━━━━━━━━━━ ← valor default (línea negra)
         │
   16 °C ┤
         └────────────────────────────────
        00h    06h    12h    18h    24h
```

### Cambiar el valor (todo el día constante)

1. Pasar el cursor **sobre la línea negra** (no antes — primero tocar la línea).
2. **Mientras el cursor está sobre la línea**, escribir el valor numérico (ej. `25`).
3. **Enter**.

> "Si pones esto se mueve, pero no se está moviendo el valor. Cuando yo me pongo encima, ahí escribo el valor."

Resultado: la línea queda en `25 °C` todo el día.

### Subdividir el día (valores por hora)

Para un schedule que cambia durante el día:

1. **Doble click** en el punto del día donde quieres dividir (ej. 8 AM).
2. Aparece un corte vertical → el día se divide en dos segmentos.
3. Pasar el cursor sobre **cada segmento** y escribir su valor.
4. Repetir para más cortes (ej. otro doble click en 6 PM).

```
   30 °C ┤            ╔════════════════
         │            ║
         │            ║
   25 °C ━━━━━━━━━━━━━╝
         │
         │
   20 °C ┤            
         └────────────────────────────────
        00h    08h            18h    24h
                ↑                ↑
            doble click      doble click
            agrega corte    agrega corte
```

### Quitar un corte

**Doble click** sobre el corte → desaparece (los segmentos a sus lados se fusionan).

### Resolución temporal

Barra inferior con tres niveles de zoom:

- **Cada hora** (default).
- **Cada 15 minutos**.
- **Cada minuto** (máxima — coincide con el límite de E+).

A mayor resolución, más precisión pero más segmentos para gestionar.

## 6. Run Period (overrides para temporadas)

Si quieres un schedule distinto en, por ejemplo, vacaciones de diciembre:

1. Botón **+ Run Period** (o equivalente).
2. Definir el rango de fechas (ej. 15-dic a 5-ene).
3. Editar el profile específico para ese rango.

Durante ese rango, el Run Period **sobreescribe** el Default Profile. Fuera del rango, manda el Default.

## 7. Distinguir lunes-viernes vs. fin de semana

Algunos schedules permiten un **schedule por día de la semana**:

- Default profile = lunes a viernes.
- Sub-profile = sábado.
- Sub-profile = domingo.

Útil para horarios de oficina (8-18 lunes a viernes, sin actividad los fines de semana).

> Para el taller — donde solo se usan setpoints constantes — el Default Profile basta.

## 8. Verificar el resultado

Pedir como output variable:

```
Schedule Value
```

con el `Key Value` siendo el nombre del schedule. Después de correr, graficar:

```python
fig, ax = plt.subplots()
ax.plot(df["heating_setpoint"], label="heating")
ax.plot(df["cooling_setpoint"], label="cooling")
ax.legend()
```

Útil para verificar que el schedule llegó al IDF como esperabas (timezone correcto, valores correctos).

## Ejemplo completo — los tres modos de [[../concepts/Setpoint]]

| Schedule | Heating profile | Cooling profile |
|----------|-----------------|-----------------|
| **T constante a 20 °C** | Default profile = 20 °C todo el día | Default profile = 20 °C todo el día |
| **Banda de confort 20-25 °C** | Default profile = 20 °C | Default profile = 25 °C |
| **Solo enfriar (México)** | Default profile = −1 °C (nunca se alcanza) | Default profile = 25 °C |

Cada modo se configura creando dos schedules (uno para heating, otro para cooling) con los valores correspondientes.

## Trampas comunes

| Síntoma | Causa |
|---------|-------|
| Escribir el valor y no cambia la línea | El cursor no estaba sobre la línea. Volver a posicionar y escribir |
| Cambia el valor pero solo en un segmento | Hay un corte intermedio — verificar que no se hizo doble click accidental |
| Severe en `.err`: "schedule type limits violation" | Valor fuera del rango del tipo de schedule. Cambiar el tipo o ajustar |
| Schedule no aparece en la lista de Thermal Zones | El tipo es incorrecto (ej. Fraction en lugar de Temperature) |

## Clases relacionadas

- [[../classes/009-AireAcondicionadoSetPoints]] — demo en vivo de la interfaz visual

## Procedimientos relacionados

- [[Configurar-Aire-Acondicionado-Ideal]] — usa estos schedules para setpoints
