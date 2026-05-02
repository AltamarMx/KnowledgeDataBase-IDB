---
title: Ventanas
type: concepto
tags: [concepto, ventanas, sub-superficie, vidrio, glazing, energyplus, marcos]
aliases: [ventanas, windows, glazing, vidrios, fenestracion]
clases: [006]
updated: 2026-05-02
---

# Ventanas

## Qué son en E+

Las ventanas son **sub-superficies** ([[Subsuperficie]]) que se anidan dentro de una superficie tipo **Wall**. No son superficies independientes — modifican el balance de su muro padre sustrayendo área opaca y agregando un camino con propiedades ópticas y térmicas distintas.

## Parámetros geométricos típicos

Cuando se inserta una ventana desde FloorspaceJS, los parámetros del componente son:

| Parámetro | Significado |
|-----------|-------------|
| **Window-to-Wall Ratio (WWR)** | Fracción del muro padre cubierta por la ventana (0–0.99). 1.0 no se permite. |
| **Height** | Alto de la ventana (m). |
| **Width** | Ancho de la ventana (m). |
| **Sill Height** | Altura del antepecho — distancia desde el piso hasta el borde inferior de la ventana (m). |
| **Window Type** | `FixedWindow`, `OperableWindow`, `Door`, `GlassDoor`, ... |
| **Overhang Projection Factor** | Genera un [[Superficies-de-Sombramiento|alero]] horizontal arriba de la ventana. |
| **Fin Projection Factor** | Genera parteluces verticales a los lados. |

> Modos alternativos: o se especifican `Height` + `Width` + `Sill Height`, o solo el `WWR`. No es necesario dar ambos.

### Window-to-Wall Ratio (WWR)

Métrica clave para análisis y normativas:

$$
WWR = \frac{A_{ventana}}{A_{muro}}
$$

Valores normativos en México (NOM-008):

| Tipo | WWR máximo |
|------|------------|
| Vivienda | 20% |
| Comercial | 25% |

(Áreas máximas para evitar carga térmica excesiva; existen excepciones según orientación y sombreamiento.)

## Restricción crítica — sub-superficie ≠ 100%

Una ventana **no puede ocupar el 100%** del muro padre. Hay un margen mínimo (~1-2%). Caso típico: cafetería con pared "abierta" — se modela un muro virtual con una ventana al ~95-98% que después se trata como abierta. Detalle en [[Subsuperficie]].

## Materiales de ventana en E+

Hay **dos formas** de definir el material óptico-térmico de una ventana:

### 1. Glazing Window Material (capas reales)

Modela la ventana **capa por capa**:

- Cada vidrio (espesor, conductividad, transmitancia/reflectancia/emisividad por banda).
- Espacios de gas entre vidrios (aire, argón, kriptón).
- Películas low-E.

Más fiel a la realidad pero requiere **muchos parámetros** que no siempre se conocen:

- Transmitancia solar perpendicular y a otros ángulos.
- Reflectancia frontal y trasera.
- Transmitancia visible (distinta a la solar).
- Reflectancia visible.
- Transmitancia y emisividad en infrarrojo.
- Conductividad del vidrio (~1 W/m·K).
- Factor de ensuciamiento.

> Para caracterizar un material nuevo experimentalmente: el grupo IER tiene un **simulador solar** (lámpara que reproduce el espectro solar con potencia controlada) — pruebas conforme a normativa.

### 2. Simple Glazing System (simplificación)

Una sola "capa" caracterizada por **3 parámetros**:

| Parámetro | Significado |
|-----------|-------------|
| **U-factor** | Transmitancia térmica global de la ventana (W/m²K) |
| **SHGC** | Solar Heat Gain Coefficient — fracción de radiación solar que entra (transmitida + absorbida y re-emitida hacia adentro) |
| **Visible Transmittance** | Fracción de luz visible que pasa |

Sintetiza un sistema multi-capa (vidrio + aire/argón + vidrio + low-E) en tres números. **Lo que recomienda usar el profesor en el taller** salvo que se esté caracterizando un material nuevo.

> Hay bases de datos internacionales con `U-factor`, `SHGC` y `Visible Transmittance` para sistemas comerciales.

## Vidrio mexicano típico

- **Vidrio flotado** sin tratamiento.
- Espesores comerciales: **3, 6 y 9 mm**.
- **Transmitancia solar:** ~0.88.
- **Conductividad térmica:** ~1.0 W/m·K.

Open Studio trae precargado un material `glazing 3mm` que sirve como punto de partida para vidrios sencillos del curso.

## Marcos (framing)

E+ permite definir el **marco** de la ventana aparte (objeto `WindowProperty:FrameAndDivider`):

- Material del marco con su propia conductividad.
- Ancho del marco (genera puente térmico al sumar/sustraer área del cristal).
- Divisores internos.

### Conductividades típicas de marcos

| Material | Conductividad (W/m·K) | Comportamiento |
|----------|------------------------|----------------|
| **Vidrio** | ~1.0 | Referencia |
| **Aluminio** | ~150 (sin ruptura), ~1.5 (con ruptura térmica) | Fuerte puente térmico |
| **PVC** | ~0.17 | Bajo puente térmico — recomendable con AC |

> Caso del IER: todos los cubículos con AC tienen **ventanas con marco de aluminio sin ruptura térmica**. Es contraproducente — el marco conduce más que el cristal y genera puentes que disipan el frío del AC.

### En el alcance del taller

El curso **ignora el marco** — la "ventana" del modelo incluye todo el marco como si fuera vidrio. Es una caricatura ([[Caricatura-Computacional]]) que sobreestima un poco la transmitancia solar y subestima el puente térmico. Diseño detallado de marco se ve en cursos posteriores.

## Caso histórico — cubículos de la plataforma solar IER

Hace años se instalaron **ventanas con película low-E** en los cubículos cercanos a la plataforma solar pensando que reducirían carga térmica. Resultado: **se volvieron infernales**.

Mecanismo:

1. La película reflejaba bien el infrarrojo.
2. Pero **absorbía** una fracción significativa del visible.
3. El vidrio se calentaba (alta absortancia visible).
4. Al calentarse, **emitía radiación térmica en ambas direcciones** — incluyendo hacia adentro.
5. Resultado neto: más calor adentro que con vidrio simple.

Lección: una ventana se diseña por **conjunto de propiedades espectrales**, no por una sola (low-E o anti-IR no es suficiente). Aplicable hoy a **paneles fotovoltaicos translúcidos** en fachadas — investigación pendiente del grupo.

## Para análisis bioclimático

Variables clave de output asociadas (ver [[Variables-Output-EnergyPlus]]):

| Variable | Para qué |
|----------|----------|
| `Surface Outside Face Incident Solar Radiation Rate per Area` (sobre la ventana) | Cuánta radiación llega — útil para evaluar aleros |
| `Zone Windows Total Heat Gain Rate` | Calor neto que entra por todas las ventanas de la zona |
| `Surface Window Transmitted Solar Radiation Rate` | Radiación que pasó al interior |

## Estrategias bioclimáticas relacionadas

- **Tamaño y orientación**: WWR pequeño en orientaciones expuestas, mayor en orientaciones protegidas.
- **Sombreamiento**: aleros, parteluces, vegetación — ver [[Superficies-de-Sombramiento]].
- **Material**: doble vidrio con cámara de aire/argón para mejorar U; SHGC bajo para climas cálidos.
- **Marcos PVC** en lugar de aluminio (cuando hay AC).
- **Operables** vs **fijas** según estrategia de ventilación natural.

## Clases relacionadas

- [[../classes/006-DosZonasTermicasVentanasAleros]] — introducción al objeto ventana, parámetros, materiales y casos prácticos
