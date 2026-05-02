---
title: TMY (Typical Meteorological Year)
type: concepto
tags: [concepto, clima, epw, datos, reanalisis]
aliases: [Typical Meteorological Year, año tipico meteorologico, año-tipico]
clases: [002]
updated: 2026-05-02
---

# TMY — Typical Meteorological Year

## Definición

**Año típico meteorológico**: año "representativo" del clima de una ubicación, construido a partir de varios años de datos. Se usa como entrada climática en simulaciones energéticas — es el contenido del archivo **EPW**.

> **Crucial:** el TMY **no es** el año promedio. Es un año cuyos meses **se parecen más a la distribución** de todos los meses correspondientes en el periodo de datos.

## Cómo se construye

Para cada mes del año (enero, febrero, ..., diciembre):

1. Toma todos los eneros del periodo de datos (ej. 2007-2021 → 15 eneros).
2. Mide qué tanto se "parece" cada enero a la distribución conjunta de todos los eneros (usando estadísticas tipo Finkelstein-Schafer — análogo conceptual al $R^2$ de un fit).
3. Selecciona el enero que minimiza la distancia → ese es el "enero típico".
4. Repite para los 12 meses.

El TMY resultante puede ser un **Frankenstein de años**: enero de 2016, febrero de 2018, marzo de 2016, abril de 2009, etc.

## Implicaciones

### Suaviza anomalías

Por construcción, el TMY descarta meses extremos. Un mes con una ola de calor o una helada anómala difícilmente sale seleccionado porque su distribución no se parece a la mayoría.

**Consecuencia:** simular con TMY tiende a **subdimensionar** estrategias de mitigación contra eventos extremos.

### Pierde efecto del cambio climático

Los eventos extremos cada vez son más frecuentes. Un TMY de los últimos 15 años promedia (vía representatividad) condiciones que ya no son las actuales:

- Subestima frecuencia y severidad de calor extremo.
- Subestima cambios en patrones de precipitación.
- No captura tendencias monotónicas.

Para análisis con horizonte > 10 años o evaluación de resiliencia, conviene complementar con escenarios de cambio climático (proyecciones del IPCC, downscaling regional).

### Datos de reanálisis (no de estaciones)

Los TMYs típicos provienen de **datos de reanálisis** (modelos meteorológicos rellenando gaps con física + observaciones), no directamente de estaciones meteorológicas. Se validan donde hay estación; en zonas rurales o sin estación, son la única fuente disponible.

## Periodos comunes

Diferentes versiones de TMY se construyen sobre periodos distintos. En OneBuilding, los nombres incluyen el periodo:

- `TMY-2007-2021`
- `TMY-2009-2023`
- `TMYx` — actualizaciones más recientes

**Implicación práctica:** dos TMYs de la misma ciudad pueden diferir si el periodo es distinto. Hay que verificar qué periodo se está usando para reportar resultados.

## Alternativas al TMY

| Tipo de archivo | Cuándo usar |
|-----------------|-------------|
| **TMY** | Comparar estrategias bioclimáticas en condiciones "normales" |
| **Año real específico** | Validar contra datos medidos; estudiar un evento concreto |
| **TMY filtrado / extremo (TDY, TWY)** | Dimensionamiento de equipos para condiciones desfavorables |
| **Proyecciones de cambio climático** (downscaled) | Resiliencia futura |
| **Construido localmente desde estación** | Cuando hay estación local confiable y se acepta no tener "típico" |

## Recursos

### OneBuilding.org

Repositorio global con archivos EPW para casi cualquier ciudad. Cada zip trae varios archivos relacionados; uno de ellos es el `.epw`.

### Año típico solar (Jesús Quiñones, instituto)

Versión específica para análisis solar (no meteorológico general), publicada en **ANES**. Útil para Temixco. Contactar al autor si se necesitan los datos.

### Estación meteorológica de Temixco

El grupo construye EPWs propios desde la estación cuando hace experimentos locales — pero esos archivos **no son TMY** (suelen ser de un año específico).

## En el curso

Cada equipo escogerá una **ciudad** para el proyecto final. Tendrán que:

1. Verificar que existe un EPW (TMY o equivalente) para esa ciudad.
2. Aprender a inspeccionar el EPW (rangos de temperatura, radiación, ventilación) — esto se cubrirá en clases posteriores.
3. Considerar las temporadas relevantes según el clima.

> **Tip estratégico:** climas con doble extremo (Monterrey, Sonora) generan TMYs con ambas temporadas exigentes → más trabajo de modelado.

## Clases relacionadas

- [[../classes/002-ConceptosBasicosBalancesCalor]] — introducción al concepto y su rol en el archivo EPW
