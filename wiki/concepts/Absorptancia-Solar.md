# Absorptancia Solar

Fracción de la radiación solar incidente que una superficie absorbe (valor entre 0 y 1). Es una propiedad del material/acabado superficial que determina cuánta energía solar se convierte en calor en la superficie.

## Valores típicos

| Color/Material | Absorptancia (α) |
|---------------|-------------------|
| Blanco | 0.2 – 0.3 |
| Colores claros | 0.3 – 0.5 |
| Colores medios | 0.5 – 0.7 |
| Negro | ~0.8 |

## En EnergyPlus

- Se define como propiedad de la superficie exterior de cada sistema constructivo
- Se usa en el balance de calor exterior: q_SW = (I_directa + I_difusa) × α
- Es uno de los parámetros más fáciles de modificar en una estrategia bioclimática (cambiar el color de un muro) y puede tener un impacto significativo en la temperatura interior

## Importancia para diseño bioclimático

En la clase 001 se menciona como ejemplo clave: cuantificar la diferencia entre una casa blanca (α ≈ 0.2) y una azul (α ≈ 0.6). Es una estrategia barata con impacto medible.

## Aparece en

- [[002-ConceptosBasicosBalancesCalor]] — Definición y uso en el balance de calor
- [[001-IntroduccionTallerIDB]] — Ejemplo de estrategia bioclimática cuantificable
