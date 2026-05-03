---
title: Mezclado Perfecto
type: concepto
tags: [concepto, suposicion, modelado, balance-aire, energyplus]
aliases: [perfect mixing, well-mixed, mezclado perfecto del aire]
clases: [003, 005]
updated: 2026-05-02
---

# Mezclado Perfecto

## Definición

Suposición central del balance de aire en Energy Plus: **todo el aire de una [[Zona-Termica]] tiene la misma temperatura instantánea**. En cada paso temporal, E+ suma todos los flujos de calor que entran a la zona (por conducción de muros, radiación que pasa por ventanas, cargas internas, infiltración) y los reparte uniformemente en la masa de aire — toda la zona cambia su temperatura **al mismo tiempo y por igual**.

> "Energy Plus va a agarrar ese flujo de calor y lo va a agarrar todo el aire y lo va a mezclar perfectamente."

## Qué implica (y qué se pierde)

La realidad tiene fenómenos que el mezclado perfecto **no representa**:

- **Estratificación térmica**: aire caliente arriba, frío abajo (gradiente vertical).
- **Plumas térmicas**: ascensos de aire calentado por personas, equipos, superficies.
- **Capas térmicas** cerca de muros fríos o calientes.
- **Ráfagas localizadas** (ej. salida de un difusor de aire acondicionado).

Para E+ todo el volumen es un único punto térmico — no se sabe si en una esquina hace 24°C y en otra 28°C; reporta una sola temperatura de zona.

## Cuándo la suposición se rompe

Mientras **más grande** sea el espacio, peor es la aproximación:

- En un cubo pequeño (3×3×3 m) la diferencia entre puntos suele ser despreciable.
- En una nave industrial, un atrio o un auditorio se observan gradientes verticales de varios °C.
- En espacios con **AC o calefacción** las ráfagas generan zonas de confort distinto para personas a diferentes alturas — algo que el modelo no captura.

**Mediciones de confort térmico** ideales (cuando se compara con simulación) toman al menos 4 alturas:

- Tobillos, cadera, pecho parado.
- Sentado: tobillos, cadera, pecho — quedan a alturas distintas.

Promediar varias mediciones se acerca a la "temperatura de zona" que reporta E+.

## Equivalente conceptual con superficies

El mezclado perfecto del aire es **paralelo** a la suposición de [[Sistemas-Constructivos]] de **material homogéneo**: igual que se asume una sola temperatura para todo un muro a lo largo de su superficie, se asume una sola temperatura para todo el volumen de aire de la zona. Ambas son simplificaciones unidimensionales.

## Si necesito modelar gradientes

Energy Plus no puede modelar el aire al interior con resolución espacial. Si el problema lo requiere:

- Subdividir la zona en varias zonas térmicas conectadas (no captura plumas, pero sí permite gradiente entre niveles).
- Usar **CFD** (Computational Fluid Dynamics) — herramienta distinta, costosa de configurar y resolver.
- En la siguiente materia (Energía en Edificaciones) se ven alternativas como **Airflow Network**.

## Por qué el curso vive con la suposición

El objetivo del taller es **evaluar el impacto relativo** de estrategias bioclimáticas (orientación, color, masa, sombreamiento). Aunque el mezclado perfecto distorsiona la temperatura absoluta, **el orden de las estrategias** suele preservarse: si A mejora el confort respecto a B en la simulación, lo más probable es que también lo haga en la realidad. Las **proporciones** sí pueden cambiar.

## Variable de output asociada

La temperatura uniforme de la zona se reporta como `Zone Mean Air Temperature` (o el alias `T_<zona>` en `iertools`). Detalle del catálogo en [[Variables-Output-EnergyPlus]].

> Existe también `Zone Air Temperature` — casi idéntica, solo difiere en simulaciones con modelos de aire detallado (no se usan en el curso).

## Clases relacionadas

- [[../classes/003-MiPrimeraSimulacion]] — introducción explícita a la suposición y sus implicaciones
- [[../classes/005-AnalisisSimulacionesPython]] — uso práctico de `Zone Mean Air Temperature` como output
