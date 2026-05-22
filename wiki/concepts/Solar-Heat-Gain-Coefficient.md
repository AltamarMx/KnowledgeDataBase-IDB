---
title: Solar Heat Gain Coefficient (SHGC) y U-value
type: concepto
tags: [concepto, ventanas, shgc, u-value, vidrio, radiacion, energyplus, window-lbnl, proyecto-final]
aliases: [SHGC, solar heat gain coefficient, factor solar, U-value, U-factor, transmitancia térmica, ventanas SHGC]
clases: [014]
updated: 2026-05-22
---

# Solar Heat Gain Coefficient (SHGC) y U-value

## Definiciones

### SHGC — Solar Heat Gain Coefficient

$$
SHGC = \frac{\text{Flujo de calor que entra al interior por la ventana}}{\text{Radiación solar incidente sobre la ventana}}
$$

Es el **factor de ganancia solar** — la fracción del flujo solar incidente que termina como calor dentro del cuarto. Cuenta:

1. **Radiación directamente transmitida** a través del vidrio.
2. **Radiación absorbida** por el vidrio, que lo calienta, y **se re-emite como infrarrojo** hacia el interior.

Adimensional, entre 0 y 1. SHGC bajo = menos calor entra (ventana "fría"). SHGC alto = vidrio "transparente al calor".

> "El SHGC es mejor que la resistencia térmica porque la resistencia solo es por conducción. El SHGC está volteando a ver la parte que proviene de la radiación."

### U-value (U-factor)

$$
U = \frac{1}{R_{\text{térmica total}}}
$$

Coeficiente global de **transferencia de calor** [W/m²K] que cuantifica la conducción + convección + radiación IR a través de la ventana cuando hay una **diferencia de temperatura** interior-exterior.

| Tipo de ventana | U-value típico [W/m²K] |
|---|---|
| Vidrio simple 3 mm | ~5.8 |
| Vidrio doble con aire | ~2.8 |
| Vidrio doble con argón | ~2.6 |
| Vidrio doble + low-E | ~1.8 |
| Vidrio triple con argón + 2 low-E | ~0.8 |

U bajo = mejor aislamiento; U alto = la ventana "pierde calor" fácilmente.

### Por qué SHGC y U se reportan juntos

Capturan **dos mecanismos de transferencia distintos** que dominan en climas distintos:

| Clima | Mecanismo dominante | Métrica importante |
|---|---|---|
| Cálido, sol intenso | Radiación solar | **SHGC** (queremos bajo) |
| Frío, sin sol | Conducción por ΔT | **U** (queremos bajo) |
| Mixto | Ambos | SHGC y U juntos |

## Los dos modelos de ventana en Energy Plus

Documentación oficial: `Engineering Reference → Window Heat Balance Calculation`.

### Modelo Simple — `WindowMaterial:SimpleGlazingSystem`

Una sola "capa" caracterizada por **3 números**:

| Parámetro | Significado |
|---|---|
| **U-factor** [W/m²K] | Transmitancia térmica global |
| **SHGC** [0-1] | Factor de ganancia solar |
| **Visible Transmittance** [0-1] | Transmitancia de la luz visible |

Lo que **recomienda el profesor** en el taller. Origen de los números:

1. **Bases de datos de fabricantes** — ventanas comerciales certificadas (frecuente en EUA/Europa, **poco común en México**).
2. **Calculadas con Window LBNL** — definir el sistema multicapa una vez en Window, exportar el SHGC y U, pegarlos en E+.

### Modelo Complex — `Construction:ComplexFenestrationState`

Modela la ventana **capa por capa**:

- Cada vidrio: espesor, conductividad, transmitancia/reflectancia/emisividad espectral.
- Espacios de gas entre vidrios: aire, argón, kriptón, vacío.
- Capas low-E con sus propiedades direccionales.
- Marcos y divisores.

Requiere **muchos parámetros** que normalmente no se tienen:

- Transmitancia y reflectancia a **varios ángulos de incidencia**.
- Transmitancia visible vs solar **separadas**.
- Reflectancia frontal vs trasera.
- Emisividad IR.
- Factor de ensuciamiento.

> "Si E+ tiene el de ventanas complejas, ¿por qué no lo uso? Pues lo pueden usar — pero hay que saber un montón de propiedades del vidrio que normalmente no tenemos."

### El workflow recomendado

```
Ventana real (vidrios + gas + low-E)
        │
        ▼
   Window LBNL  (modela capa por capa)
        │
        ▼
   SHGC + U-value + VT
        │
        ▼
WindowMaterial:SimpleGlazingSystem  →  Energy Plus
```

Window LBNL **calcula** el modelo complejo una vez; E+ **usa** el modelo simple en cada simulación. Mucho más eficiente y manejable.

Detalle del flujo en [[../tools/Window-LBNL]] y [[../procedures/Usar-Window-LBNL]].

## Por qué las ventanas no tienen masa térmica

> "Los vidrios son muy delgados, 3, 6, 9 mm los más gruesos, y la conductividad es alta. Entonces se puede asumir que no tienen masa térmica."

En el balance de energía de la ventana:

- $\rho c_p \delta \approx$ pequeño comparado con el flujo conductivo $k \, \Delta T / \delta$.
- Tiempo característico $\tau = \delta^2 / \alpha$ es de **fracción de segundo**.
- En timesteps de 10 minutos, la ventana ya está en cuasi-equilibrio.

Por eso E+ resuelve la ventana en **estado estacionario** dentro de cada timestep, no como problema dependiente del tiempo. Esto justifica el modelo simple con solo SHGC + U.

## Anti-patrón comercial — películas que "absorben 80% del calor"

> "La gente erróneamente compra esas películas. Alarman que absorba 80% del calor — no es lo deseable. Lo va a absorber, se va a calentar y lo va a emitir."

### La trampa física

Razonamiento erróneo:

> "Si la película absorbe el calor del sol, no entra al cuarto."

Realidad:

1. La película **absorbe** la radiación solar de onda corta.
2. La película **se calienta** (su temperatura sube).
3. Como toda superficie caliente, **emite radiación infrarroja** (onda larga) en ambas direcciones — hacia afuera **y hacia adentro**.
4. El cuarto recibe la **componente IR re-emitida hacia el interior** + el flujo conductivo a través del vidrio.

Una ventana con película absorbente puede tener **SHGC mayor** que un vidrio simple, según la geometría de absorción. El producto del marketing es vendido como "anti-calor" pero **calienta el cuarto vía emisión IR**.

### Lo que sí funciona

Ventanas que **reflejan** el infrarrojo y el visible que no se necesita:

- **Superficies reflejantes** (películas espejo / metálicas externas).
- **Superficies con baja emisividad (low-E)** — reducen la emisión IR.
- Las low-E **normalmente van entre dos vidrios** para protegerse mecánicamente y porque ahí su efecto reduce el U junto con el SHGC.

> "Uno quiere ventanas que reflejen el calor. Para eso necesitas superficies reflejantes o superficies con baja emisividad — pero las superficies low-E normalmente se usan en ventanas dobles."

### Por qué los termómetros no captan el problema

> "Si yo pongo un termómetro [en un patio con acrílico caliente], no se va a ver — porque eso se ve en la **temperatura radiante**. El AC tampoco ve ese efecto — ve la temperatura de bulbo seco."

Un termómetro de bulbo seco mide la temperatura del **aire**. La radiación IR desde una superficie caliente afecta la temperatura **operativa** ([[Temperatura-Operativa]]) y el confort percibido — pero **no** la temperatura del aire directamente. Esto explica que:

- En un patio con acrílico, **se siente más calor** que el que marca el termómetro.
- Un AC controlado por bulbo seco **no se entera** del efecto radiante extra y subdimensiona la corrección.

Ver [[Temperatura-Operativa]] y [[Emisividad]].

## Veredicto: ventanas dobles según contexto

| Situación | Veredicto | Por qué |
|---|---|---|
| Clima cálido, **sin AC** | **Casi inútiles** | La estrategia debería ser ventilación + sombreamiento, no aislamiento de la ventana |
| Clima cálido, **con AC** | **Sí valen** | Mantienen el frío del AC; reducen carga sensible |
| Clima templado | **A veces valen** | Reducen pérdida nocturna + ganancia diurna — neto depende del clima |
| Clima frío | **Sí valen** | Reducen pérdida de calor |

> "Ventanas dobles en edificaciones sin AC es tirar el dinero. Aumentan la resistencia térmica, pero aquí lo que queremos es ventilar."

### Trade-off oculto — iluminación

Cada vidrio adicional **reduce la transmitancia visible**. Doble vidrio + low-E puede bajar VT de ~0.89 (simple) a ~0.60 — pierdes 30% de luz natural. Hay que **iluminar más con luz artificial**, lo que puede comer el ahorro térmico.

> "Cada vez que pongo una ventana doble, también disminuye la iluminación porque va de la mano."

## En el alcance del taller

El proyecto final 2026-2 ([[../classes/012-ProyectoFinal]]) parte de **vidrio simple 3 mm** ([[Ventanas]]).

Una de las estrategias bioclimáticas válidas es **cambiar el vidrio**:

1. Definir el sistema deseado en [[../tools/Window-LBNL|Window LBNL]].
2. Exportar SHGC + U-value.
3. Crear un nuevo `WindowMaterial:SimpleGlazingSystem` en Open Studio con esos valores.
4. Comparar caso base vs variante.

Ver [[../procedures/Agregar-Ventanas-OpenStudio]] para el procedimiento.

## Clases relacionadas

- [[../classes/014-InfiltracionFloorspaceWindowLBNL]] — introducción a Window LBNL, dos modelos de ventana en E+, anti-patrón de películas absorbentes

## Ver también

- [[Ventanas]] — concepto general
- [[Emisividad]] — propiedad clave para low-E
- [[Absortancia-Solar]] — para superficies opacas
- [[Temperatura-Operativa]] — por qué los termómetros no captan el efecto radiante
- [[../tools/Window-LBNL]] — herramienta para calcular SHGC + U
