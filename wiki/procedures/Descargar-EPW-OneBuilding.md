---
title: Descargar un EPW desde OneBuilding y cargarlo en Open Studio
type: procedimiento
tags: [procedimiento, epw, openstudio, clima, onebuilding]
aliases: [descargar epw, onebuilding, asignar epw]
clases: [003]
updated: 2026-05-02
---

# Descargar un EPW desde OneBuilding y cargarlo en Open Studio

Procedimiento para obtener un archivo de clima [[../concepts/TMY]] (Typical Meteorological Year) y asignárselo a un proyecto de Open Studio.

## 1. Acceder a OneBuilding

- Sitio: **climate.onebuilding.org**
- Navegar a la región → país → estado/región → ciudad.
- Para México: Norteamérica → México → Estado.

## 2. Escoger el TMY adecuado

Cada ciudad suele tener **varios EPWs** con periodos distintos:

| Sufijo típico | Significado |
|---------------|-------------|
| `TMYx.2004-2018` | Año típico construido con los datos del periodo 2004–2018 |
| `TMYx.2007-2021` | Construido con 2007–2021 |
| `TMYx.2009-2023` | Construido con 2009–2023 |

**Criterio:** preferir el periodo **más reciente** — refleja mejor el clima actual. Aun así el TMY suaviza anomalías por construcción (ver [[../concepts/TMY]]) y pierde gran parte del efecto del cambio climático.

**Caso Cuernavaca/Temixco:** el aeropuerto MX_MO_Cuernavaca está físicamente en Temixco. Si se quiere algo distinto, la estación de **Zacatepec** ofrece una alternativa cercana.

## 3. Descargar el ZIP

OneBuilding entrega un `.zip` que contiene varios archivos:

- `*.epw` — **el archivo que importa**
- `*.stat` — estadísticas
- `*.ddy` — design days
- `*.clm`, `*.pvsyst`, etc. — para otros programas

## 4. Extraer y mover el EPW

1. Descomprimir el ZIP (en el escritorio, p. ej.).
2. Localizar el `*.epw`.
3. Copiarlo al folder **`EPW/`** del proyecto:

   ```
   tarea_01_primer_cubo/
   └── EPW/
       └── MX_MO_Cuernavaca.AP.766800_TMYx.2009-2023.epw
   ```

4. El resto de archivos del ZIP se pueden tirar (a menos que se necesite el `.stat` para inspeccionar el clima).

Ver [[Estructura-Proyecto-Simulacion]] para la convención de carpetas.

## 5. Asignar el EPW al modelo en Open Studio

En el Open Studio Application:

1. Pestaña **Site** (icono de globo, columna izquierda).
2. Sección **Weather File**.
3. Botón **Set Weather File**.
4. Navegar al `EPW/` del proyecto y seleccionar el `.epw`.

Open Studio mostrará automáticamente:

- **Latitud, longitud**
- **Time zone** (UTC offset) — las simulaciones de E+ usan **horario civil** (uso horario), no horario solar
- **Elevation**
- Nombre de la estación

## 6. Verificación rápida

Antes de correr la simulación:

- Confirmar que la **latitud/longitud** corresponde a la ciudad esperada.
- Confirmar que el **time zone** es el correcto (México centro: UTC-6; CDMX: UTC-6).
- Si no carga, revisar que el path del EPW no tenga **acentos, eñes o espacios** (ver [[Estructura-Proyecto-Simulacion]]).

## Alternativas a OneBuilding

- **Estación de Temixco** del IER → permite construir EPW propios para experimentos locales (no es TMY — es un año específico, con anomalías).
- **Año típico solar** publicado por **Jesús Quiñones** (IER) en ANES — usa criterio solar para construir el año típico, distinto al criterio meteorológico general.
- **EnergyPlus.net Weather** — colección oficial, formato similar.

## Para el proyecto final

Cada equipo elige una **ciudad** y verifica que existe un EPW. Tip estratégico (de la clase 002): **evitar ciudades con doble extremo** (Monterrey, Sonora) — exigen diseñar para temporada cálida y fría con soluciones que se contradicen.

## Clases relacionadas

- [[../classes/002-ConceptosBasicosBalancesCalor]] — explicación del TMY
- [[../classes/003-MiPrimeraSimulacion]] — primera vez que se descarga y carga un EPW
