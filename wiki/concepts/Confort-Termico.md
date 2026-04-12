# Confort Térmico

Condición de bienestar de una persona respecto al ambiente térmico que la rodea. En el contexto de edificaciones, se refiere a las condiciones de temperatura interior que permiten una vida adecuada.

## Ideas clave

- Es el **énfasis del curso** y del grupo de investigación de Energías en Edificaciones
- Es un problema "invisible" porque:
  - No se traduce directamente en emisiones de carbono
  - No siempre implica ahorro de energía (una casa confortable puede tener mayor huella de carbono por uso de materiales)
  - No se le puede poner precio fácilmente
- El **disconfort térmico** es uno de los grandes problemas en México, especialmente en vivienda social
- Existe diferencia entre **temperatura del aire** y **temperatura radiante** — Energy Plus puede calcular ambas
- El eje social de la sostenibilidad exige considerar el confort, no solo las emisiones
- Carla (investigadora mencionada) trabaja en índices de pobreza multidimensional que incluyen confort térmico e injusticia energética

## Modelo adaptativo de Humphreys-Nicol

Ecuación para calcular la temperatura de neutralidad (temperatura a la que las personas se sienten confortables) en edificaciones sin aire acondicionado:

> **T_neutralidad = 0.54 × T_exterior_promedio_mensual + 13.5**

- Se calcula por mes usando la temperatura exterior promedio mensual del EPW
- La zona de confort se define como [T_n - ΔT, T_n + ΔT], donde ΔT es la amplitud aceptable
- Los **grados-hora de disconfort** (cálido o frío) cuantifican el tiempo y magnitud fuera de la zona de confort
- La variable de referencia debería ser la **[[Temperatura-Operativa]]**, no solo la temperatura del aire

## Grados-hora de disconfort (detalle)

Métrica principal para evaluar estrategias bioclimáticas en el proyecto final:

- **Grados-hora cálidos:** suma de (Ti − límite superior) para cada hora en que Ti > límite superior
- **Grados-hora fríos:** suma de (límite inferior − Ti) para cada hora en que Ti < límite inferior
- Unidades: °C·h — números típicamente grandes (ej. 32,535 para un año)
- **Ventaja sobre % de tiempo en disconfort:** captura no solo cuándo hay disconfort, sino cuánto (magnitud). Dos edificaciones con el mismo % de tiempo pueden tener grados-hora muy diferentes
- La amplitud de la zona de confort varía por sitio: climas con mayor oscilación térmica → la gente se adapta más → franja más amplia

### Flujo de cálculo

1. Calcular temperatura de neutralidad mensual (modelo adaptativo)
2. Definir límites superior e inferior
3. Para cada hora del año: calcular exceso o déficit respecto a los límites
4. Sumar grados-hora cálidos y fríos por separado
5. Comparar caso base vs estrategias bioclimáticas (reducción porcentual)

## Aparece en

- [[001-IntroduccionTallerIDB]] — Presentación como eje central del curso
- [[005-AnalisisSimulacionesPython]] — Modelo adaptativo, temperatura operativa, grados-hora de disconfort
- [[008-ShadingVentanas]] — Grados-hora como métrica del proyecto final, por qué no usar solo % de tiempo
- [[EDA-Archivo-EPW]] — Promedios mensuales de To para calcular temperatura de neutralidad (Cuernavaca)
