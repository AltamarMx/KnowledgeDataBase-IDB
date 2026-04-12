# Superficies de Sombramiento

Elementos opacos o semi-transparentes que bloquean la radiación solar sin participar en la transferencia de calor. En EnergyPlus se conocen como *shading surfaces*.

## Qué hacen

- **Bloquean radiación solar directa y difusa** — proyectan sombras sobre ventanas y muros
- Tienen **reflectancia/absorptancia** asignada que afecta cuánta radiación reflejan al entorno
- Pueden ser **semi-transparentes** (para modelar celosías)
- Emiten **radiación de onda larga** hacia superficies cercanas (esto sí se modela)

## Qué NO hacen

- **No participan en la transferencia de calor por conducción** — no tienen temperatura calculada
- **No obstruyen el viento** — EnergyPlus no resuelve CFD
- La conducción desde un alero caliente hacia el muro es despreciable (proceso unidimensional, el calor no "dobla la esquina")

## Tipos comunes

### Overhangs (aleros horizontales)

- Superficie horizontal arriba de una ventana
- Definidos por **Overhang Projection Factor** = profundidad / altura de la ventana
- Limitación en Open Studio: el overhang se genera del mismo ancho que la ventana

### Fins (aleros verticales)

- Superficies verticales a los lados de una ventana
- Definidos por **Fin Projection Factor** = profundidad / ancho de la ventana
- Complementan muy bien a los overhangs — cubren los ángulos laterales del sol

### Celosías (louvers)

- Conjunto repetido de superficies horizontales y verticales
- Excelentes para bloquear radiación mientras permiten ventilación
- Se modelan como un **alero equivalente** — una superficie grande con el mismo ángulo de protección
- Muy usadas en climas cálidos (costa mexicana)

## Diseño de protecciones solares

- El ángulo desde el borde inferior de la ventana hasta el extremo del alero define cuándo el sol penetra
- Un alero al ras de la ventana (sin extensión lateral) solo protege cuando el sol está perpendicular a la fachada
- **Recomendación:** combinar overhang + fin para protección completa
- Los pasillos de edificios funcionan como aleros para el piso inferior

## Efectividad según orientación

- **Fachadas sur:** overhangs muy efectivos (sol llega con ángulo alto)
- **Fachadas este-oeste:** overhangs poco efectivos (sol con ángulo bajo al amanecer/atardecer) → necesitan fins o no protegen bien
- **Fachada norte:** principalmente radiación difusa → protecciones tienen poco impacto
- Recomendación de diseño: maximizar ventanas norte-sur, minimizar este-oeste

## Sunlit Fraction y algoritmo de sombramiento

EnergyPlus calcula el sombramiento mediante la variable `Surface Outside Face Sunlit Fraction`:

- **Solo afecta radiación directa** — la difusa y reflejada se manejan vía factores de vista
- Para **ventanas (sub-superficies):** la radiación incidente reportada NO incluye el sombramiento; EnergyPlus aplica la Sunlit Fraction internamente al hacer el balance de energía
- Para **muros (superficies):** la radiación incidente SÍ incluye el efecto de sombramiento directamente
- **Overlapping:** cuando múltiples sombras se superponen, EnergyPlus detecta la intersección para no contar doble

Documentación: Engineering Reference, sección "Shading and Sunlit Area Calculations".

## Aparece en

- [[006-DosZonasTermicasVentanasAleros]] — Overhangs, fins, celosías, alero equivalente, limitaciones
- [[007-CasoBaseAleros]] — Efectividad limitada en fachadas este-oeste, radiación difusa vs directa
- [[008-ShadingVentanas]] — Sunlit Fraction, quirk de radiación en sub-superficies, algoritmo de sombramiento
- [[003-MiPrimeraSimulacion]] — Mención de shading surfaces como alternativa a modelar pasillos como zonas térmicas
