# Masa Térmica

Capacidad de un material o sistema constructivo para almacenar energía térmica. Es una propiedad clave para entender el comportamiento dinámico de una edificación.

## Definición

> Masa_térmica = ρ × c × L

| Variable | Unidades | Descripción |
|----------|----------|-------------|
| ρ | kg/m³ | Densidad del material |
| c | J/(kg·K) | Calor específico |
| L | m | Espesor |
| **Resultado** | **J/(m²·K)** | Energía por unidad de área para elevar 1 K |

Al multiplicar por el área de la superficie → **J/K** = energía total para elevar todo el material 1 K.

## Rangos típicos de densidad

| Material | Densidad (kg/m³) |
|----------|-------------------|
| Poliestireno expandido (EPS) | ~35 |
| Tabique (ladrillo) | ~1200–1800 |
| Concreto alta densidad | ~2500 |

El concreto almacena ~70× más energía por volumen que un aislante como EPS.

## Efectos en la edificación

- **Más masa térmica → menores oscilaciones de temperatura interior** — el material absorbe energía cuando hace calor y la libera cuando se enfría
- Las **particiones interiores** (cubículos, divisiones) agregan masa térmica
- **Muebles, libros y objetos** también contribuyen — una casa vacía tiene mayor amplitud térmica que una amueblada
- Casas abandonadas se sienten frías precisamente por la ausencia de masa térmica interior

## Relación con el warm-up de EnergyPlus

Materiales con alta masa térmica requieren más iteraciones en el warm-up period para alcanzar el estado oscilatorio permanente, porque tardan más en "olvidar" la condición inicial de 23°C.

## Aparece en

- [[004-InterpretandoMensajesConstructionSets]] — Definición formal, fórmula, rangos de densidad, efecto estabilizador
- [[003-MiPrimeraSimulacion]] — Definición de materiales con densidad, conductividad y calor específico
