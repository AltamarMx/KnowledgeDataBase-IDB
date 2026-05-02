---
title: Instalar Open Studio
type: procedimiento
tags: [procedimiento, instalacion, openstudio, setup]
herramienta: Open-Studio
version: 1.11.0-rc
clases: [001]
updated: 2026-05-02
---

# Instalar Open Studio

Procedimiento para instalar **Open Studio Application 1.11.0** — versión acordada para todo el grupo del curso.

## Prerrequisitos

- Sistema operativo soportado:
  - **Windows** (instalador `.exe`)
  - **macOS** 13 o superior
  - **Ubuntu** 22 o 24 (otras distribuciones de Linux **no** funcionan)
- ~320 MB de espacio para la descarga + espacio adicional para la instalación

## Pasos

1. **Ir al sitio oficial:** [openstudiocoalition.org](https://openstudiocoalition.org) (Open Studio Coalition).

2. **Buscar la sección "Open Studio Application".**

   > **No confundir con:**
   > - **SDK** — se instala a nivel de terminal, no provee interfaz gráfica. **No instalar.**
   > - **Plugin para SketchUp** — no se usa en el curso. **No instalar.**

3. **Click en el enlace de descarga** → redirige a la página de **GitHub Releases** del proyecto.

4. **En la sección "Assets"**, bajar el instalador correspondiente a tu sistema operativo:

   | SO | Archivo |
   |----|---------|
   | Windows | `.exe` |
   | macOS | `.dmg` |
   | Ubuntu 22 | `.deb` para 22 |
   | Ubuntu 24 | `.deb` para 24 |

5. **Ejecutar el instalador** y seguir el flujo (siguiente, siguiente, finalizar).

6. **Verificar la instalación:** abrir Open Studio y confirmar que aparece la versión **1.11.0** en la pantalla principal.

## Qué se instala junto

Open Studio 1.11.0 trae **Energy Plus integrado** (versión específica atada a esta release). **No es necesario instalar Energy Plus por separado.**

## Versión: por qué importa fijarla

> **Acuerdo del grupo:** todos usan **1.11.0**.
>
> **Razón:** una versión más nueva puede abrir archivos de versiones anteriores, pero **no al revés**. Si un estudiante usa una versión más reciente, el profesor no podrá abrir su archivo.

## Documentación opcional

Si se quiere tener la documentación offline, descargar los PDFs de Energy Plus:

- **Input Output Reference** (~2952 pp.)
- **Engineering Reference**

(En la práctica casi siempre hay wifi, así que no es indispensable.)

## Próxima clase

Llegar con Open Studio 1.11.0 instalado. La clase 002 empieza con la física (transferencia de calor, balances) — ver [[../classes/001-IntroduccionTallerIDB]] para contexto.

## Solución de problemas

(Esta sección se ampliará a medida que aparezcan casos en clase.)

- **Ubuntu distinto a 22/24:** no soportado. Considerar máquina virtual o cambiar de distribución.
- **macOS < 13:** no soportado. Actualizar el sistema.
- **Versión 1.10.x ya instalada:** actualizar a 1.11.0 — versiones distintas no leen archivos hacia adelante.

## Relacionado

- Herramienta: [[../tools/Open-Studio]]
- Clase donde se asignó: [[../classes/001-IntroduccionTallerIDB]]
