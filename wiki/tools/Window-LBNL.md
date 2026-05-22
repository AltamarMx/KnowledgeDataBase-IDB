---
title: Window LBNL
type: herramienta
tags: [herramienta, ventanas, lbnl, shgc, u-value, windows, parallels]
aliases: [Window, Window LBNL, Window Berkeley, LBNL Window, WINDOW]
clases: [014]
updated: 2026-05-22
---

# Window LBNL

## Qué es

**Window** es un programa desarrollado por el **Lawrence Berkeley National Laboratory** (LBNL) para calcular las propiedades térmico-ópticas de sistemas de ventana multi-capa.

**Función principal**: dado un sistema de ventana (vidrios, gases entre vidrios, capas low-E, marcos), calcular en estado estacionario:

- **U-factor** (W/m²K) — transmitancia térmica global.
- **SHGC** (Solar Heat Gain Coefficient) — fracción de radiación solar que termina como calor en el interior.
- **Visible Transmittance** — fracción de luz visible que pasa.

Estos tres valores se pegan directamente a Energy Plus (`WindowMaterial:SimpleGlazingSystem`) — ver [[../concepts/Solar-Heat-Gain-Coefficient]].

> "Energy Plus tiene un modelo complejo de ventanas, pero hay que saber un montón de propiedades del vidrio que normalmente no tenemos. Por eso usamos Window — calcula esas propiedades por nosotros."

## Limitaciones

| Limitación | Implicación |
|---|---|
| **Solo corre en Windows** | En Mac: Parallels, VirtualBox, BootCamp; en Linux: Wine (no soportado oficialmente). El profesor usa Parallels. |
| **Estado estacionario** | No resuelve transitorios — coherente con el supuesto de ventanas sin masa térmica ([[../concepts/Solar-Heat-Gain-Coefficient#por-qué-las-ventanas-no-tienen-masa-térmica]]). |
| **Suposiciones fijas internas** | Condiciones de borde estandarizadas (NFRC). No representa exactamente las condiciones de la casa real, pero permite comparaciones reproducibles. |
| **Interfaz de los 90** | Look-and-feel de Windows XP. Es funcional pero no moderno. |
| **Curva de aprendizaje** | Crear un sistema desde cero requiere conocer las bases de datos internas (Glass, Gap, Frame). Ver [[../procedures/Usar-Window-LBNL]]. |

## Versión recomendada

**Window 7.8 estable** — sin betas.

> "Nunca bajen las betas. Las betas son mala idea."

Descarga: <https://windows.lbl.gov> → sección Downloads.

## Cadena de instalación

La instalación es **laboriosa** porque Window depende de dos librerías que no vienen incluidas:

1. **Microsoft Visual C++ Redistributable 14, arquitectura x86**
2. **Intel Fortran Compiler Runtime for Windows** (IFX 2023.1 o superior)
3. **Window 7.8 Full Setup**

Si se instala Window sin las dependencias, da error `DLL not found` al ejecutar. Detalle paso a paso en [[../procedures/Instalar-Window-LBNL]].

> "Si no los instalas, vas a obtener un DLL error cuando trates de correr. Por eso voy a abrirlo en una nueva pestaña, descargo todo primero. Justo luego ese es mi error."

## Workflow básico

Detalle en [[../procedures/Usar-Window-LBNL]]. Esquema:

```
1. Abrir Window LBNL
2. Definir capas (Glass library + Gap library)
3. Construir el sistema (Glazing System)
4. Calcular → exporta U-factor, SHGC, VT
5. Copiar valores a Open Studio (Simple Glazing System)
6. Correr la simulación con la nueva ventana
```

## Bases de datos incluidas

Window incluye bases de datos del **IGDB** (International Glazing Database) y otras:

- **Glass library** — miles de vidrios comerciales con sus propiedades espectrales (transmitancia + reflectancia a 300-2500 nm).
- **Gap library** — gases (aire, argón, kriptón, vacío) con sus propiedades de conducción y convección.
- **Frame library** — marcos comerciales (aluminio con/sin ruptura, PVC, madera).
- **Shading library** — persianas, cortinas, películas (más reciente).

> "Las bases de datos tienen sistemas que en México no se comercializan, pero sirven de referencia."

## Cuándo usarlo en el taller

Para el **proyecto final 2026-2** ([[../classes/012-ProyectoFinal]]):

- **No es obligatorio** — el caso base usa vidrio simple 3 mm ya disponible en Open Studio.
- **Si una estrategia bioclimática propuesta involucra cambio de vidrio**, usar Window para calcular SHGC + U y pegarlos al `Simple Glazing System` en Open Studio.

Ejemplos de estrategias que requieren Window:

| Estrategia | Sistema en Window | Por qué |
|---|---|---|
| Doble vidrio con aire | Glass-Air-Glass | Reducir U |
| Doble vidrio con argón + low-E | Glass-Argon-Low-E Glass | Reducir U y SHGC |
| Vidrio polarizado / espejo | Glass especial con propiedades direccionales | Reducir SHGC |
| Tragaluz con vidrio absorbente vs reflejante | Para comparar dos opciones | Anti-patrón vs estrategia válida |

## Cuándo NO usarlo

- Si se está en clima cálido sin AC y se busca una mejora real → **ventilación, sombreamiento y color** dominan ([[../concepts/Solar-Heat-Gain-Coefficient#veredicto-ventanas-dobles-según-contexto]]).
- Si no se va a comparar con caso base → el SHGC absoluto no significa mucho sin un contexto comparativo.
- Cuando ya hay un certificado del fabricante (raro en México).

## Anti-patrón comercial detectado

Window puede usarse para **demostrar que una película "absorbente" no funciona**: modelar el sistema con la película → ver SHGC efectivo → comparar con vidrio simple → mostrar que el SHGC no baja tanto como se anuncia y/o que la emisión IR hacia adentro sube. Útil en **consultoría** cuando un cliente insiste en una solución mal informada.

> "Las ventanas dobles son bien pedidas por los clientes: 'quiero ventanas dobles'. Si no te sirven, le demuestras que no funciona."

Detalle en [[../concepts/Solar-Heat-Gain-Coefficient#anti-patrón-comercial-—-películas-que-absorben-80-del-calor]].

## Relación con otras herramientas

| Herramienta | Función | Relación con Window |
|---|---|---|
| [[Open-Studio]] | GUI para crear el modelo | Recibe los SHGC + U calculados por Window |
| [[EnergyPlus]] | Motor de simulación | Usa el `SimpleGlazingSystem` con esos valores |
| **IDF Editor (E+)** | Editor de archivos `.idf` | Otra herramienta de LBNL/DOE solo Windows que muchas veces se usa junto con Window |
| [[iertools]] | Análisis post-simulación | Lee las simulaciones que usaron Window indirectamente |

## Recomendación general — sistema operativo para trabajar en simulación

Pregunta recurrente: "¿qué computadora compro para trabajar en simulación energética?"

> "Una Windows. Si tienen recursos, una Mac con mucha RAM para que tengan Windows adentro."

Razones:

- **Window LBNL** solo en Windows.
- **IDF Editor** solo en Windows.
- **Design Builder, eQuest** y otros — solo Windows.
- **Open Studio** es multiplataforma pero su comunidad es Windows-céntrica.

Mac/Linux funciona para gran parte del workflow (Open Studio, EnergyPlus, Python), pero **tarde o temprano se necesita Windows** para una herramienta específica.

## Clases relacionadas

- [[../classes/014-InfiltracionFloorspaceWindowLBNL]] — introducción + instalación

## Ver también

- [[../concepts/Solar-Heat-Gain-Coefficient]] — qué significa lo que Window calcula
- [[../concepts/Ventanas]] — concepto general de ventanas en E+
- [[../procedures/Instalar-Window-LBNL]] — cadena de dependencias
- [[../procedures/Usar-Window-LBNL]] — workflow paso a paso
- [[../procedures/Agregar-Ventanas-OpenStudio]] — cómo pasar el SHGC/U al modelo
