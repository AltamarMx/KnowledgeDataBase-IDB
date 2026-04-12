# Emitancia

Fracción de radiación térmica que emite una superficie respecto a un cuerpo negro a la misma temperatura. Bajo la suposición de **cuerpo gris** (usada en EnergyPlus), la emitancia es igual a la absorptancia térmica.

## Valores típicos

| Material | Emitancia |
|----------|-----------|
| Materiales de construcción (concreto, ladrillo, yeso) | ~0.9 |
| Aluminio pulido | ~0.1 |
| Vidrio de baja emitancia (Low-E) | ~0.01 |

## Importancia

- Controla el intercambio radiativo de onda larga entre superficies interiores
- La radiación de onda larga puede ser el **60-70%** de la transferencia de calor en un muro interior
- Un material de baja emitancia reduce significativamente el intercambio radiativo
- Las ventanas Low-E usan esta propiedad para reducir pérdidas térmicas

## En EnergyPlus

- Se define como "Thermal Absorptance" en las propiedades del material
- Participa en el balance de calor tanto exterior (intercambio con ground, cielo, aire, alrededores) como interior (intercambio entre superficies)

## Aparece en

- [[003-MiPrimeraSimulacion]] — Definición y valores en el contexto de materiales
- [[002-ConceptosBasicosBalancesCalor]] — Uso en ecuaciones de Stefan-Boltzmann del balance exterior
