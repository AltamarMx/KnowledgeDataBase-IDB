# Envolvente Arquitectónica

Conjunto de superficies que delimitan una edificación y la separan del exterior: muros, techo, piso y ventanas. Es a través de la envolvente que ocurre la transferencia de calor entre el interior y el exterior.

## Ideas clave

- Cada superficie de la envolvente tiene asignado un **sistema constructivo** (capas de materiales con propiedades térmicas)
- Las superficies tienen **condiciones de frontera** que las conectan con el exterior, con otros espacios, o las definen como adiabáticas
- En Energy Plus, la envolvente se define en el archivo IDF mediante objetos de geometría, materiales y construcciones
- Open Studio permite previsualizar la geometría de la envolvente (Energy Plus no)
- La volumetría se dibuja en planta en el editor de geometrías de Open Studio

## Restricciones de modelado

- **Solo superficies planas** — no hay líneas curvas ni ventanas circulares (factor de vista propio = 0)
- **Transferencia de calor 1D** — perpendicular a cada superficie; no se modela flujo lateral
- **Temperatura uniforme por superficie** — cada muro tiene una temperatura equivalente (promedio ponderado)
- Los modelos son siempre "caricaturas" — se pueden dividir o combinar superficies según la física del problema
- Existe el objeto **masa térmica** para representar elementos no modelados en la geometría (ej. trabes interiores)

## Aparece en

- [[001-IntroduccionTallerIDB]] — Descripción de cómo se modela la envolvente en Energy Plus y Open Studio
- [[002-ConceptosBasicosBalancesCalor]] — Restricciones 1D, superficies planas, modelado como "caricatura"
