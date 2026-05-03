---
title: Usar EnerHabitat (web app)
type: procedimiento
tags: [procedimiento, enerhabitat, web-app, sistemas-constructivos, primeras-decisiones]
aliases: [enerhabitat web, usar enerhabitat]
clases: [010]
updated: 2026-05-02
---

# Usar EnerHabitat (web app)

Procedimiento para usar la versión web de [[../tools/EnerHabitat]] para evaluar y comparar sistemas constructivos. Es la vía más rápida cuando solo se necesitan primeras decisiones.

## 1. Acceder a la web app

URL: `enerhabitat.com` (servidor del IER — sujeto a cambios). Si la web cae, el código está en GitHub (organización `enerhabitat`, repo `web-app`) y se puede correr local con Shiny.

> Las sesiones en Shiny tienen timeout de ~5 minutos. Si dejas de interactuar, la sesión se desconecta y hay que recargar.

## 2. Cargar un EPW

Por default EnerHabitat trae un EPW de Temixco. Para usar otra ciudad:

### Opción A — usar el EPW por default

Avanzar al paso 3.

### Opción B — subir un EPW propio

Funcionalidad nueva (no estaba en la versión vieja). Útil para climas no incluidos por default:

1. Botón **Subir EPW** (en la sección de configuración del clima).
2. Seleccionar archivo `.epw` desde tu disco.
3. EnerHabitat carga lat/lon/timezone automáticamente.

EPW se puede descargar de [OneBuilding](https://climate.onebuilding.org/) — ver [[Descargar-EPW-OneBuilding]].

> "Antes la gente nos escribía 'oye, sube el EPW de mi ciudad'. Ahora le damos control al usuario."

## 3. Inspeccionar el clima

Tras cargar el EPW, EnerHabitat grafica automáticamente:

- **Temperatura del aire exterior** (T_aire) durante el mes seleccionado.
- **Radiación global, difusa y directa** sobre plano horizontal.
- **Zona de confort** según [[../concepts/Confort-Adaptativo|Humphreys-Nicol]] (línea verde, calculada con T promedio del mes).

Verificación visual: confirmar que la T y radiación corresponden al clima esperado.

## 4. Seleccionar el mes

Dropdown de mes. EnerHabitat **no resuelve el año completo** — solo un mes a la vez.

Estrategia recomendada:

- **Primer análisis**: el mes más cálido del año (típicamente abril-mayo en México central).
- **Segundo análisis**: el mes más frío (diciembre-enero).
- Comparar el comportamiento del sistema constructivo en ambos extremos.

> "EnerHabitat agarra el día promedio del mes — entonces yo puedo cambiar el mes y ver cómo cambia el comportamiento."

## 5. Definir geometría

### Tipo de superficie

| Opción | Inclinación | Uso |
|--------|-------------|-----|
| **Muro** | 90° (vertical) | Muros perimetrales |
| **Techo** | 180° (horizontal hacia arriba) | Cubiertas planas |

> Solo dos opciones — no hay inclinaciones intermedias en la GUI. Para techo a doble agua hay que ir al paquete Python.

### Orientación

Cuatro orientaciones cardinales: **Norte, Sur, Este, Oeste**.

Recomendación: usar **Este** o **Oeste** para casos donde el sombreamiento es difícil — son las orientaciones más exigentes (ver [[../concepts/Trayectoria-Solar]]).

### Absortancia solar

Slider o input numérico (0-1):

| Color | α típico |
|-------|----------|
| Blanco | 0.3 |
| Amarillo claro | 0.5 |
| Rojo / café | 0.7 |
| Negro | 0.9 |

Detalle en [[../concepts/Absortancia-Solar]].

## 6. Definir las capas del sistema constructivo

Botón **+ Capa** agrega una capa al construction. Para cada capa:

- **Material**: dropdown desde la base de datos local de EnerHabitat (tabique, concreto, EPS, etc.).
- **Espesor**: en metros.

**Orden**: de **exterior a interior** (igual convención que Energy Plus — ver [[../concepts/Sistemas-Constructivos]]).

Ejemplo: muro tradicional mexicano:

| Capa | Material | Espesor |
|------|----------|---------|
| 1 (exterior) | Mortero / repellado | 0.02 m |
| 2 | Tabique recocido | 0.14 m |
| 3 (interior) | Yeso | 0.015 m |

## 7. Definir el modo HVAC

Toggle:

- **Sin aire acondicionado**: la T interior flota libre — se reportan FD, tiempo de retraso, T máxima/mínima interior.
- **Con aire acondicionado**: T interior se mantiene constante (ideal loads) — se reporta energía cooling y heating consumida.

Para análisis bioclimático puro: **sin AC**. Detalle en [[../concepts/Aire-Acondicionado-Ideal]].

## 8. Calcular

Botón **Calcular** (o `Run`). Tiempo: **~5 segundos** por sistema constructivo. EnerHabitat resuelve hasta alcanzar [[../concepts/Estado-Oscilatorio-Permanente]].

## 9. Leer los resultados

EnerHabitat muestra:

### Gráfica de temperaturas

- **Temperatura sol-aire** ($T_{sa}$) — el "forzamiento" exterior. Pico al mediodía (sur), mañana (este), tarde (oeste).
- **Temperatura interior** del cuarto.
- Zona de confort sombreada (banda verde).

### Métricas

| Métrica | Significado | Recomendación |
|---------|-------------|----------------|
| **Factor de Decremento** | $\Delta T_i / \Delta T_o$ | Solo informativo (puede ser > 1) |
| **Factor de Decremento sol-aire** | $\Delta T_i / \Delta T_{sa}$ | **Métrica clave** (siempre < 1) |
| **Tiempo de retraso** | Desfase entre pico exterior e interior | Mayor = más inercia |
| **Energía transmitida** | (Bug — sale 0 en versión actual) | Ignorar hasta que se corrija |

Detalle en [[../concepts/Factor-de-Decremento]].

## 10. Comparar dos o más sistemas constructivos

EnerHabitat permite agregar **2+ sistemas** en la misma simulación:

1. Botón **+ Sistema constructivo** después del primero.
2. Definir las capas del segundo (puede compartir el primero pero con un cambio — color, espesor, posición del aislante).
3. **Calcular** — EnerHabitat resuelve los dos y los muestra superpuestos.

Caso típico de uso: **mismo tabique, dos colores** (blanco vs rojo) para cuantificar el impacto del color.

```
                        Sistema 1: tabique 14 cm absortancia 0.7
                        Sistema 2: tabique 14 cm absortancia 0.3
                        
Resultado: T interior diferente, FD_sa similar (pero T menor en sistema 2)
```

## 11. Exportar datos

Sección **Resultados** tiene botón **Descargar datos**: CSV con las series temporales de cada sistema. Útil para análisis externo en Python o Excel.

Columnas típicas:

- `T_aire`, `T_neut`, `delta_t`, `T_sa`, `T_i_sistema_1`, `T_i_sistema_2`, ...
- `IS` (radiación incidente sobre la superficie).

## Trampas comunes

| Síntoma | Causa probable |
|---------|----------------|
| FD > 1 | **Esperado** con FD ingenuo si la radiación es alta. Mirar FD sol-aire |
| T interior > T sol-aire | Imposible — algo está mal en el cálculo. Reportar bug |
| Sesión se desconecta | Timeout de Shiny (~5 min sin actividad). Recargar |
| Energía transmitida = 0 | Bug conocido, no usar esa métrica |
| No aparece un material | No está en la base de datos local. Para agregarlo: paquete Python o pedir al grupo IER |

## Cuándo pasar al paquete Python

Limitaciones de la GUI que justifican migrar al paquete:

- Necesitas **inclinaciones intermedias** (ej. techo 30°).
- Necesitas **orientaciones no cardinales** (ej. SW, ENE).
- Necesitas hacer **estudios paramétricos** (variar 10 valores de absortancia automáticamente).
- Necesitas **agregar materiales custom** sin pedirlos al grupo.
- Necesitas **integrar con flujo Python** (combinar EnerHabitat con análisis pandas, gráficas matplotlib custom).

Procedimiento: [[Usar-EnerHabitat-Python]].

## Clases relacionadas

- [[../classes/010-EnerHabitatParte1]] — introducción a la herramienta y demo en vivo de la web
