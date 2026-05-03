---
title: Algoritmo de Sombreamiento de Energy Plus
type: concepto
tags: [concepto, energyplus, sombreamiento, algoritmo, geometria, debugging]
aliases: [shadow algorithm, algoritmo sombras, polygon clipping, pixel counting, overlapping]
clases: [008]
updated: 2026-05-02
---

# Algoritmo de Sombreamiento de Energy Plus

## Cómo calcula E+ las sombras

E+ recorre, para cada paso temporal con sol (limitado por la frecuencia de [[Calculo-Sombramientos|actualización de sombras]] cada 20 días por default):

1. **Posición solar** — calcula altitud y acimut a partir de fecha/hora/lat/lon del EPW.
2. **Transformación de coordenadas** — convierte de coordenadas globales (referenciadas a un punto fijo) a coordenadas relativas a cada superficie (lagrangianas → eulerianas).
3. **Proyección de sombras** — proyecta cada superficie potencialmente sombreadora sobre la superficie analizada, usando geometría plana (intersecciones de polígonos).
4. **Detección de overlapping** — cuando varias proyecciones se traslapan, debe contabilizarse el área cubierta **una sola vez** (no sumar las áreas).
5. **Cálculo de Sunlit Fraction** — área no sombreada / área total = `SF`. Se aplica solo a la **radiación directa**.
6. **Factores de vista** — para difusa y reflejada del ground, E+ ajusta los factores de vista entre la superficie y el cielo/ground (atenuación por las sombras estructurales como aleros).

Detalle sobre el output: ver [[Sunlit-Fraction]].

## Tres algoritmos disponibles

E+ ofrece tres algoritmos de cálculo de sombreamiento (objeto `ShadowCalculation`):

| Algoritmo | Característica | Cuándo usar |
|-----------|----------------|-------------|
| **Polygon Clipping** | Default. Geometría exacta de polígonos. CPU-intensivo. | Caso general |
| **Pixel Counting** | Discretiza superficies en píxeles, cuenta cuántos están sombreados. Más rápido en geometrías complejas. Aproximado. | Modelos grandes con muchas superficies |
| **Imported** | Usa máscaras pre-calculadas con otra herramienta (Radiance, etc.) | Casos avanzados; fuera del taller |

## Algoritmo de overlapping

Cuando dos o más superficies de sombramiento se traslapan al proyectar sus sombras, E+ aplica un algoritmo de **detección de overlapping** para no contar el área doblemente.

Ejemplo: un alero horizontal y un parteluz vertical sobre la misma ventana. La esquina donde ambos sombrean se cubre por los dos — el algoritmo detecta y descuenta el área duplicada.

### Warning común — "many overlapping shadows"

Aparece en el `.err` cuando hay muchas combinaciones de sombras (típico en geometrías complejas con muchos aleros, parteluces, vecinos).

| Severidad real | Cuándo |
|----------------|--------|
| Inocuo | El algoritmo lo resolvió correctamente; reporta para informar |
| Sospechoso | Si hay demasiados overlaps el cálculo puede ser lento y se sugiere subdividir geometría |

> "Es un warning muy común que ya se le acabó. Hay muchos overlapping. Uno decide si está en zona segura o no — eso sí, no se es feliz."

Política: en el taller se ignora si la simulación corre en tiempo razonable y los resultados son plausibles. En investigación serio se reduce reorganizando geometría.

Detalle de la política de warnings en [[Mensajes-EnergyPlus]].

## Cómo se aplican las sombras al balance de calor

El algoritmo produce dos outputs distintos que afectan el balance:

### Sobre la radiación directa

`Surface Outside Face Sunlit Fraction` (o área absoluta) → multiplicador a la radiación directa $I_b$:

$$
q''_{directa,abs} = \alpha \cdot I_b \cdot \cos\theta \cdot SF_s
$$

Si `SF = 0`, no hay aporte de directa. Si `SF = 1`, contribuye toda.

### Sobre la radiación difusa y reflejada

Las protecciones (aleros, parteluces) **modifican los factores de vista** entre la superficie y el cielo / ground:

$$
q''_{difusa,abs} = \alpha \cdot \left[ I_d \cdot F_{s\to sky}^{*} + I_g \cdot F_{s\to ground}^{*} \right]
$$

donde $F^{*}$ es el factor de vista **reducido** por la presencia de elementos de sombramiento. La difusa **siempre** llega (mientras haya cielo visible) — un alero atenúa pero no anula.

## Implicaciones — la Sunlit Fraction no es la única vía de bloqueo

> Una protección con `SF = 0` durante todo el día **no bloquea** el 100% de la radiación: la difusa sigue llegando con factor de vista al cielo no nulo.

Casos extremos:

- **Día completamente nublado**: 100% de la radiación es difusa. La protección casi no atenúa.
- **Día despejado, sol bajo**: la directa domina; una buena SF reduce drásticamente la radiación efectiva.

Por eso al evaluar el desempeño de un alero conviene mirar:

- Sunlit Fraction → fracción de directa bloqueada.
- Radiación efectiva total (directa modulada + difusa atenuada).

## Mirror surfaces (`Mir-FACE`) — superficies internas del solver

Cuando E+ procesa un modelo con **superficies de sombramiento** (aleros, parteluces, vecinos), internamente crea **superficies espejo** para el cálculo del intercambio radiativo. En el output aparecen como columnas con prefijo **`Mir-FACE`**:

```
'FACE 8',        ← superficie original del modelo
'Mir-FACE 8',    ← su mirror creada por el solver
'FACE 18',
'Mir-FACE 18',
...
```

### Cuándo aparecen

- Cuando se solicita una variable de superficie con `Key Value = *` (todas las superficies).
- El solver las incluye porque, técnicamente, son superficies del modelo.

### Qué NO son

- **No están en el OSM**.
- **No son superficies dibujadas** por el usuario.
- Son **construcciones internas** del solver — equivalentes computacionales necesarios para el cálculo de overlapping en geometrías con shading.

### Cómo manejarlas

| Caso | Acción |
|------|--------|
| Análisis comparativo simple | **Ignorarlas** — usar columnas con nombres específicos (FACE N o renombradas con alias custom) |
| Auditoría avanzada del sombreamiento | Estudiar — los `Mir-FACE` corresponden a las superficies específicas con sombras |
| Reducir explosión de columnas | **No usar `*` como Key Value** — pedir cada variable con el nombre específico de la superficie |

> **Detección visual**: en una simulación con protecciones, el output puede tener 30+ columnas (`FACE 1` ... `FACE 23` + `Mir-FACE 8` ... `Mir-FACE 23` + `SURFACE 1`). En el caso base sin protecciones suele tener < 10. La explosión es un síntoma de uso de `*` como Key Value. Confirmado en el [[../notebooks/004_Comparacion_ConSinVentanas|notebook 004]].

## Aplicación a iluminación natural

E+ no es ideal para análisis fino de iluminación natural — la frecuencia de actualización de sombras (20 días default) suaviza efectos día-a-día. Para iluminación se prefiere **Radiance** con backward ray tracing horario. Detalle en [[Calculo-Sombramientos]].

## Documentación oficial

- **Engineering Reference** sección "Shading and Sunlit Area Calculations" — describe el algoritmo completo.
- **Engineering Reference** sección "Solar Gains" — la ecuación del balance solar con SF y factores de vista.

> "Algea plana, transformaciones de coordenadas. La gente que ha trabajado en montón ha hecho pruebas y se ve bien. No tengo que cuestionar todo de Energy Plus."

## Clases relacionadas

- [[../classes/008-ShadingVentanas]] — descubrimiento del algoritmo y consulta a la documentación
