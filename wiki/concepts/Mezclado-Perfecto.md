# Mezclado Perfecto (Well-Mixed)

Suposición fundamental de EnergyPlus para el aire dentro de una zona térmica: todo el flujo de calor que entra (conducción, radiación, personas, infiltración) se mezcla instantáneamente con todo el volumen de aire, resultando en **una sola temperatura uniforme** para toda la zona.

## Implicaciones

- No hay gradientes de temperatura dentro de la zona (no distingue cerca de ventana vs. centro)
- No hay estratificación vertical (la temperatura es la misma arriba que abajo)
- No hay plumas térmicas visibles en el modelo
- Cuanto más grande el espacio, peor la aproximación

## Validación experimental

Para comparar una simulación con mediciones reales:
- Medir en **múltiples puntos** del espacio y promediar
- Para confort térmico: medir a altura de tobillos, cadera y pecho (sentado o parado)
- Las diferencias entre puntos suelen ser pequeñas pero existen (plumas térmicas, ráfagas de A/C)

## Alternativas

Para resolver gradientes de temperatura dentro de un espacio se necesitan herramientas de **mecánica de fluidos computacional (CFD)**, que no están dentro de EnergyPlus.

## Variable de salida asociada

La variable `Zone Mean Air Temperature` de EnergyPlus es el resultado directo de este modelo: una sola temperatura que representa todo el volumen de aire de la zona térmica. Se documenta en el Input/Output Reference como "the average temperature of the air temperatures at the system time step".

## Aparece en

- [[003-MiPrimeraSimulacion]] — Explicación detallada con ejemplos
- [[005-AnalisisSimulacionesPython]] — Zone Mean Air Temperature como resultado del modelo, consulta en documentación
