# Temperatura Operativa

Indicador de confort térmico que combina el efecto del aire y la radiación sobre una persona. Se define como el promedio de la temperatura de bulbo seco (aire) y la temperatura radiante media.

## Definición

> **T_operativa = (T_bulbo_seco + T_radiante_media) / 2**

- **Temperatura de bulbo seco** = temperatura del aire medida con un termómetro convencional
- **Temperatura radiante media** = temperatura de un cuerpo en equilibrio térmico radiativo con todas las superficies que lo rodean (ponderada por factores de vista)

## Cuándo importa la diferencia

| Situación | T_operativa vs T_aire |
|-----------|----------------------|
| Interior sin fuentes radiantes importantes | Son prácticamente iguales |
| Radiación solar directa por ventana | T_operativa >> T_aire |
| Superficie exterior reflectante cercana (concreto, albedo) | T_operativa > T_aire |
| Equipo radiante (cañón, calefactor) | T_operativa > T_aire |
| Noche sin radiación | Son prácticamente iguales |

## Ejemplo de la clase

Una plancha de concreto exterior con absortancia relativamente baja refleja radiación solar hacia la planta baja de un edificio. Un sensor de temperatura del aire no detecta esta ganancia, pero la temperatura radiante (y por lo tanto la operativa) aumenta, causando disconfort.

## En EnergyPlus

- Variable de salida: `Zone Operative Temperature`
- Se puede solicitar junto con `Zone Mean Air Temperature` para comparar
- La diferencia entre ambas indica la importancia de las fuentes radiantes en la zona

## Relación con confort

Los modelos adaptativos de [[Confort-Termico]] (como Humphreys-Nicol) usan la temperatura operativa como referencia para evaluar si un espacio es confortable. En muchos análisis simplificados se asume igual a la temperatura del aire, pero esto solo es válido cuando no hay fuentes radiantes significativas.

## Aparece en

- [[005-AnalisisSimulacionesPython]] — Explicación del concepto y su importancia para análisis de resultados
