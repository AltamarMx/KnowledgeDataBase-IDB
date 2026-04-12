# Factor de Vista

Fracción de la radiación emitida por una superficie que llega a otra superficie. Es un concepto puramente geométrico que determina la intensidad del intercambio radiativo entre dos superficies.

## Propiedades

- Los factores de vista de una superficie con todas las demás suman 1
- F_{A→B} = F_{B→A} (reciprocidad, ajustada por áreas)
- Para una **superficie plana**, el factor de vista consigo misma es **0** (no se "ve" a sí misma)
- Para una **superficie curva**, el factor de vista consigo misma es **> 0** (se ve a sí misma)

## En EnergyPlus

- EnergyPlus asume que todas las superficies son planas → factor de vista consigo misma = 0 siempre
- Esta es una de las razones por las que **no permite superficies curvas ni ventanas circulares**
- Se usa en el cálculo de radiación de onda larga entre:
  - Superficie ↔ ground (techo = 0, muro ≈ 0.5, depende de inclinación)
  - Superficie ↔ cielo (techo = 1, muro ≈ 0.5)
  - Superficie ↔ alrededores (depende de tamaño y distancia de los objetos)
  - Superficies interiores entre sí

## Aparece en

- [[002-ConceptosBasicosBalancesCalor]] — Uso en el balance de radiación de onda larga
