# Balance de Calor

Ecuación fundamental que EnergyPlus resuelve para cada superficie de la envolvente. Iguala la suma de los flujos de energía incidentes con el flujo conductivo que entra (o sale) del muro.

## Balance en la superficie exterior

Tres componentes entran al balance:

### 1. Radiación de onda corta (solar)
> q_SW = (I_directa_proyectada + I_difusa) × α

- **α** = absorptancia solar (blanco ~0.2–0.3, negro ~0.8)
- EnergyPlus calcula la proyección sobre la superficie y el sombreamiento

### 2. Radiación de onda larga
Intercambio radiativo (Stefan-Boltzmann) con 4 fuentes:
- **Ground** — temperatura del suelo, factor de vista según inclinación
- **Cielo** — temperatura cercana a 0 K → efecto de enfriamiento (~-40 W/m² en la noche)
- **Aire** — la atmósfera participa radiativamente a grandes distancias
- **Alrededores** — otros edificios/objetos

Se puede simplificar usando un **coeficiente radiativo equivalente** h_r (forma tipo Newton).

### 3. Convección
> q_conv = h_conv × (T_aire - T_superficie)

Depende de: inclinación, ΔT, rugosidad, velocidad del viento. EnergyPlus usa correlaciones experimentales.

### Ecuación completa
> I_s · α + q_LWR + q_conv = -k · ∂T/∂x |_{x=0}

Donde x=0 es la superficie exterior. Luego se resuelve la transferencia de calor dependiente del tiempo en 1D a través del muro hasta la superficie interior, donde hay un balance similar.

## Balance en la superficie interior

Tres componentes, análogos al exterior pero con diferencias clave:

### 1. Convección interior
> q_conv_int = h_conv_int × (T_superficie_int - T_indoor)

Coeficiente depende del tipo de superficie (techo, piso, muro) por la dirección de la convección natural.

### 2. Radiación de onda larga interior
Intercambio **solo entre superficies interiores** (la radiación de onda larga no atraviesa vidrios). Depende de factores de vista y [[Emitancia]] de los materiales. Puede ser el **60-70%** de la transferencia de calor — mecanismo dominante.

### 3. Radiación de onda corta interior
Luz solar que entra por ventanas + equipos/luces. EnergyPlus la distribuye uniformemente en todas las superficies (simplificación). Modelo "full interior exterior" proyecta la directa al piso.

### Modelo de mezclado perfecto
Los flujos de calor de todas las superficies se mezclan instantáneamente con todo el aire → toda la zona tiene **una sola temperatura** ([[Mezclado-Perfecto]]).

## Aparece en

- [[002-ConceptosBasicosBalancesCalor]] — Derivación completa del balance exterior
- [[003-MiPrimeraSimulacion]] — Balance interior, mezclado perfecto, emitancia
