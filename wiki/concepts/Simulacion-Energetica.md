---
title: Simulación Energética
type: concepto
tags: [concepto, simulacion, energia, edificaciones]
clases: [001]
updated: 2026-05-02
---

# Simulación Energética

## Definición

Uso de software para resolver el **balance de calor dependiente del tiempo** a través de la envolvente arquitectónica de una edificación, dadas unas condiciones climáticas externas. Permite cuantificar el comportamiento térmico antes de construir.

## Para qué sirve en este curso

**Cuantificar el impacto** de estrategias bioclimáticas: comparar configuraciones (color, orientación, aleros, ventanas, materiales) y obtener métricas objetivas — temperatura interior, horas de disconfort, consumo energético — en lugar de razonar cualitativamente.

## Componentes mínimos de una simulación

1. **Geometría** ([[Envolvente-Arquitectonica]]): la volumetría de la edificación.
2. **Sistemas constructivos** ([[Sistemas-Constructivos]]): qué materiales tiene cada superficie.
3. **Condiciones de frontera** ([[Condiciones-de-Frontera]]): cómo interactúa cada superficie con su entorno (otra zona, exterior, suelo, adiabática).
4. **Archivo de clima (EPW)**: el forzante externo — temperatura ambiente, radiación solar, humedad, viento, lluvia, presión atmosférica para una ubicación geográfica específica.
5. **Motor de cálculo**: en este curso, [[../tools/EnergyPlus]].

## Alcance del modelo en este curso

Las simulaciones del taller son una **caricatura** de la realidad — útil para entender pero no para diseño profesional sin más:

- Sin ventilación natural (zonas herméticas)
- Sin cargas térmicas internas (personas, equipos, iluminación)
- Geometrías simples (cubos)
- Piso adiabático

Detalles y razones en [[../classes/001-IntroduccionTallerIDB]].

## Implicación interpretativa

Los **resultados absolutos no son trasladables** a edificaciones reales, pero el **orden relativo entre estrategias se conserva**: la mejor estrategia en el modelo simplificado tiende a ser la mejor en la realidad. Por eso sirve como **guía de priorización**, no como cifra final.

## Ecosistema de software (panorama)

| Programa | Tipo | Uso |
|----------|------|-----|
| Energy Plus | Motor de cálculo, libre | Núcleo del curso |
| Open Studio | Interfaz libre sobre Energy Plus | Lo usamos como GUI |
| Design Builder | Interfaz comercial sobre Energy Plus | No se usa (de paga, propicia malas prácticas) |
| Rhino + LadyBug | Suite paramétrica, comercial | No se usa (Rhino de paga) |
| TRNSYS | Motor europeo | Alternativa a Energy Plus |
| Radiance | Iluminación natural | Fuera del alcance |
| IES | Interfaz comercial | No se usa |

## Clases relacionadas

- [[../classes/001-IntroduccionTallerIDB]] — introducción al concepto y al alcance del curso
