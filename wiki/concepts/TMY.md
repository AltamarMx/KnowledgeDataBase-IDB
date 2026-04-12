# TMY — Typical Meteorological Year

Año meteorológico típico usado como entrada climática para simulaciones energéticas. **No es un promedio** de todos los años disponibles.

## Cómo se construye

1. Para cada mes (enero, febrero, ...) se compara ese mes en todos los años disponibles
2. Se selecciona el mes que más se **parece** a todos los demás (usando distancias estadísticas, no promedios)
3. El resultado es un año compuesto donde cada mes puede venir de un año diferente

**Ejemplo:** un TMY podría tener enero de 2016, febrero de 2018, marzo de 2014, etc.

## Nomenclatura en archivos EPW

- **TMY3** — formato común, el número indica la generación del dataset
- El rango de años se indica en el nombre (ej. 2009-2023 o 2007-2021)
- Diferentes rangos de años producen diferentes TMY

## Fuente principal

**One Building** (climate.onebuilding.org) — colección gratuita de archivos EPW organizados por continente, país y ciudad. Los datos provienen de reanálisis (no de estaciones meteorológicas directamente).

## Limitaciones

- **Pierde anomalías atípicas:** si un mes tuvo un evento extremo, probablemente no queda seleccionado como "típico"
- **Pierde tendencias de cambio climático:** el TMY representa un clima "promedio" histórico, no proyecciones futuras
- **No es un año real:** la transición entre meses de diferentes años puede no ser suave

## Alternativas

- Usar datos de un año específico (real) — pero podría ser atípico
- Construir EPW propio desde estación meteorológica local
- TMY solar (variante enfocada en representar bien la radiación solar)

## Año por defecto en EnergyPlus

EnergyPlus asigna **año 2006** por defecto a todas las simulaciones, independientemente de los años reales del TMY. Al cargar resultados en Python, los datetime tendrán año 2006. Si el EPW contiene un 29 de febrero (año bisiesto) y se reasigna a un año no bisiesto, habrá errores — hay que filtrar o elegir un año bisiesto.

## Aparece en

- [[002-ConceptosBasicosBalancesCalor]] — Explicación detallada y fuentes de datos
- [[005-AnalisisSimulacionesPython]] — Año 2006, manejo de datetime en pandas
- [[EDA-Archivo-EPW]] — Libreta 002_EDA_EPW: carga EPW de Cuernavaca, filtro del 29 de febrero, promedios mensuales
