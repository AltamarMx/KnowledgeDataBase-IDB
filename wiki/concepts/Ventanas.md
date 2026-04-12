# Ventanas

Sub-superficies transparentes o semi-transparentes que permiten el paso de radiación solar al interior de la edificación. En EnergyPlus, las ventanas son elementos clave que afectan la ganancia de calor, la iluminación natural y la ventilación.

## En EnergyPlus / Open Studio

### Sub-superficies

- Las ventanas son **sub-superficies** — deben estar contenidas dentro de una superficie padre (muro)
- No pueden ocupar el 100% del área del muro
- Se listan en la pestaña **Sub Surfaces** de Open Studio

### Materiales de vidrio

| Tipo | Uso |
|------|-----|
| **Glazing Window Material** | Vidrio simple, una capa. Requiere: espesor, transmitancia solar/visible, reflectancia, transmitancia IR, conductividad |
| **Simple Glazing Window Material** | Simplificación de ventanas multicapa (doble vidrio + argón + low-E) a pocos parámetros |

**Estándar en México:** vidrio flotado Clear de 3, 6 y 9 mm. El Clear 3mm viene incluido en la librería de EnergyPlus.

### Propiedades relevantes

- **Transmitancia solar:** fracción de radiación solar que atraviesa el vidrio
- **Reflectancia:** fracción reflejada (frente y atrás)
- **Transmitancia visible:** fracción de luz visible que pasa
- **Transmitancia infrarroja:** casi cero para la mayoría de vidrios
- **Conductividad:** vidrio común ≈ 1 W/m·K
- **SHGC (Solar Heat Gain Coefficient):** fracción total de energía solar que entra (transmitida + absorbida y re-emitida)

### Marcos (framing)

- Aluminio: conductividad ~1.5 W/m·K → **puente térmico** severo con A/C
- PVC: conductividad ~0.7 W/m·K → mejor opción térmica
- El marco ocupa ~10% del área de la ventana
- En simulaciones simplificadas se incluye el marco en el área de vidrio

### Normativa mexicana

- NOM-008: máximo ~20% WWR (window-to-wall ratio) en vivienda, ~25% en comerciales

### Colocación en FloorSpaceJS

- **Component** → Window → clic sobre un muro
- Configurar: height, width, sill height (antepecho, default 0.91m), window-to-wall ratio
- Para múltiples tamaños → crear múltiples componentes

## Orientación y protección solar

- **Norte-sur:** orientación preferida. La ventana sur es fácil de proteger (sol alto), la norte recibe casi solo radiación difusa
- **Este-oeste:** orientación problemática. El sol llega con ángulo bajo → overhangs poco efectivos. La ventana oeste recibe un pico fuerte de radiación directa por la tarde

## Aparece en

- [[006-DosZonasTermicasVentanasAleros]] — Primer uso de ventanas, materiales de vidrio, marcos, normativa
- [[007-CasoBaseAleros]] — Orientación y efectividad de protecciones, radiación incidente como variable de análisis
- [[008-ShadingVentanas]] — Radiación incidente en sub-superficies no incluye sombramiento (quirk de EnergyPlus)
- [[003-MiPrimeraSimulacion]] — Mención de sub-superficies (sin ventanas en el ejercicio)
