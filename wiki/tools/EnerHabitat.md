---
title: EnerHabitat
type: herramienta
tags: [herramienta, ier, web-app, python, sistemas-constructivos, primeras-decisiones]
aliases: [EnerHabitat, ener habitat, enerhabitat]
clases: [010, 011]
updated: 2026-05-02
---

# EnerHabitat

## Qué es

Herramienta del **grupo del IER (UNAM)** para evaluar rápidamente sistemas constructivos de muros o techos opacos bajo distintos climas mexicanos. Resuelve transferencia de calor 1D dependiente del tiempo en [[../concepts/Estado-Oscilatorio-Permanente|estado oscilatorio permanente]] y reporta métricas bioclimáticas (factor de decremento, tiempo de retraso, energía).

> "Es una herramienta para tomar primeras decisiones."

## Dos interfaces

| Interfaz | Tecnología | Uso |
|----------|------------|-----|
| **Web app** | Python + Shiny | Acceso rápido sin instalar nada — `enerhabitat.com` (servidor del IER) |
| **Paquete Python** | Instalable vía PyPI (`pip install enerhabitat` o `uv add enerhabitat`) | Versatilidad, scripts paramétricos, integración en notebooks |

La web app se construyó **encima del paquete Python** — el paquete es la fuente única de verdad. Quien quiera más control que la GUI puede ir directo al paquete.

## Para qué SÍ sirve

- **Primeras decisiones rápidas**: ¿tabique 14 cm vs 18 cm? ¿con o sin aislante? ¿qué color es mejor?
- **Evaluar la [[../concepts/Posicion-Aislante|posición del aislante]]** — interior vs exterior vs intermedio.
- **Comparar 2-3 sistemas** lado a lado en pocos segundos.
- **Cuantificar el impacto del color** (absortancia solar) — caso de uso histórico de COMEX.
- **Estudios académicos** del impacto de cada parámetro aislado.

## Para qué NO sirve

- **Cálculos energéticos reales** (kWh anuales, emisiones). La caricatura del cuarto idealizado pierde demasiado.
- **Análisis de confort completo** (necesitaría incluir ventilación, ventanas, ocupación).
- **Edificios complejos** (multi-zona, geometrías irregulares).

> "No lo deben usar para hacer cálculos reales de energía, de emisiones, de confort. Está muy limitado. Es una herramienta con un uso muy claro."

Para análisis serio: pasar a Energy Plus + Open Studio.

## Modelo físico

Resuelve la PDE 1D dependiente del tiempo:

$$
\rho c_p \frac{\partial T}{\partial t} = -k \frac{\partial^2 T}{\partial x^2}
$$

con condiciones de frontera:

- **Exterior** ($x = 0$): convección con [[../concepts/Temperatura-Sol-Aire|temperatura sol-aire]] — encapsula radiación de onda corta absorbida + convección + LWR en una sola T equivalente.
- **Interior** ($x = L$): convección con T del aire interior. **Sin radiación de onda larga** (no hay otras superficies con quién intercambiar — solo se modela una pared).
- **Condición inicial**: T uniforme = $\overline{T}_{aire}$ promedio del día representativo.

Resuelve hasta alcanzar [[../concepts/Estado-Oscilatorio-Permanente|estado oscilatorio permanente]] (típicamente 5-10 días repetidos).

## Caricaturas

EnerHabitat asume:

| Suposición | Realidad | Consecuencia |
|------------|----------|--------------|
| **Una sola pared** (no la edificación completa) | Edificios tienen 4-6 superficies | No hay LWR interior, no hay zona térmica completa |
| **Cuarto idealizado** de 2.5 m con pared opuesta adiabática | Cuartos son rectangulares con 4 muros + techo + piso | El aire interior recibe solo el flujo convectivo de UN muro |
| **Sin ventanas** | Casi todas tienen | No captura ganancia solar interior ni transmitancia |
| **Sin ventilación** | Vivienda mexicana ventila | No captura el efecto de cambios de aire |
| **Día representativo del mes** | Cada día es distinto | Resultado es un "promedio" — no hay extremos |
| **Estado oscilatorio permanente** | Días reales no son iguales | Análogo a clases de DC en circuitos: útil para entender pero no es la realidad transitoria |
| **No considera humedad ni HVAC realista** | Ambos importan | EnerHabitat con AC = setpoint de T constante (ideal loads) |

Detalle en [[../concepts/Caricatura-Computacional]].

## Algoritmo numérico

- **Discretización espacial**: volúmenes de control (medios nodos en superficies, nodos completos en el interior).
- **Discretización temporal**: paso de 1 hora típicamente.
- **Esquema**: **semi-implícito** (más robusto que explícito; explícito tiene problemas de inestabilidad).
- **Solución del sistema lineal**: **TDMA** (Tridiagonal Matrix Algorithm) optimizado con NumPy.
- **Reuso de pvlib**: para cálculo de radiación solar incidente sobre superficies orientadas (proyección por trayectoria solar). No reinventar la rueda.

> "Optimizado con [Numba] y se pasó a NumPy para que funcione más rápido."

## Métricas reportadas

| Métrica | Significado | Recomendación de uso |
|---------|-------------|----------------------|
| [[../concepts/Factor-de-Decremento|Factor de Decremento]] (FD) ingenuo | $\Delta T_i / \Delta T_o$ | Solo informativo — puede ser > 1 con alta radiación |
| [[../concepts/Factor-de-Decremento|FD sol-aire]] | $\Delta T_i / \Delta T_{sa}$ | **Métrica recomendada** — siempre 0-1, comparable entre climas |
| **Tiempo de retraso** | Desfase entre pico exterior e interior | Mayor = más inercia. Combinada con FD para caracterizar el sistema |
| **Energía transmitida** | Integral del flujo entrante por ciclo | Bug actual: sale 0. No usar |

Detalle de las métricas en [[../concepts/Factor-de-Decremento]].

## Repositorios GitHub (organización `enerhabitat`)

| Repo | Para qué |
|------|----------|
| **`enerhabitat`** | El paquete Python (publicado en PyPI) |
| **`web-app`** | La aplicación web Shiny (puede correrse local) |
| **`validation`** | Scripts de validación contra Energy Plus (mismas condiciones, comparar resultados) |

## Contexto histórico

- **Versión inicial**: HTML + Java, ventana fija de 1024×768. Funcionó muchos años pero quedó anticuada.
- **Servidor antiguo** del IER cayó hace > 1 año, no se pudo recuperar.
- **Re-implementación**: Python + Shiny, escrita por el profesor + estudiantes (Alex / Fernando) durante el último año.
- **Estado actual**: en producción pero con bugs conocidos (ej. métrica de energía transmitida da 0; documentación del paquete Python incompleta — descubierto en clase 010).

## Reuso de software

EnerHabitat decide reutilizar paquetes existentes en lugar de reinventar:

- **`pvlib`** — proyección de radiación solar sobre superficies orientadas. Estándar de la industria solar.
- **`numpy`** — cálculo numérico.
- **`pandas`** — manejo de series temporales.
- **`shiny`** — framework web Python.

Filosofía: "no reinventar la rueda — el paquete ya lo hace bien". Buena práctica de software libre.

## Casos de uso reales

- **COMEX** — usó EnerHabitat para cuantificar el impacto de absortancia de pinturas. Pidieron al grupo dar de alta más materiales en la base. Concedieron una carta de uso.
- **Tesis del IER** — herramienta primaria para análisis paramétrico inicial antes de pasar a E+.
- **Casos académicos** — clases de diseño bioclimático en otras universidades mexicanas.

## Documentación

- **Web app**: `enerhabitat.com` (URL pública del IER, sujeta a cambio).
- **Paquete Python**: README en GitHub. **La documentación es incompleta** (descubierto en clase 010 — hay parámetros como `energy=True` no documentados). El profesor planea actualizarla.

## Diferencia con Energy Plus

| Aspecto | EnerHabitat | Energy Plus |
|---------|-------------|-------------|
| **Alcance** | Una pared opaca | Edificación completa multi-zona |
| **Tiempo de cálculo** | Segundos | Minutos a horas (annual) |
| **Geometría** | 1D, una capa de capas | 3D real |
| **Climas** | Cualquier EPW | Cualquier EPW |
| **Salida** | Un día representativo | Series anuales completas |
| **Métricas** | FD, tiempo de retraso, energía 1D | T zona, T operativa, energía HVAC, etc. |
| **Cuándo usar** | Primeras decisiones, comparativos rápidos | Análisis serio, proyectos, regulaciones |

Son **complementarios** — un flujo profesional usa EnerHabitat para descartar opciones rápidamente, luego E+ para análisis fino del finalista.

## Clases relacionadas

- [[../classes/010-EnerHabitatParte1]] — introducción a la herramienta, web app, ecuaciones, demo Python con bug
- [[../classes/011-EnerHabitatParte2]] — continuación (próxima ingesta)

## Procedimientos

- [[../procedures/Usar-EnerHabitat-Web]] — flujo en la web app
- [[../procedures/Usar-EnerHabitat-Python]] — flujo en el paquete Python
