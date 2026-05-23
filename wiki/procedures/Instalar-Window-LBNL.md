---
title: Instalar Window LBNL (con dependencias)
type: procedimiento
tags: [procedimiento, instalacion, windows-lbnl, lbnl, dependencias, parallels, ventanas]
aliases: [instalar window, instalar window lbnl, descargar window, install window berkeley]
clases: [014]
updated: 2026-05-22
---

# Instalar Window LBNL (con dependencias)

Procedimiento para instalar [[../tools/Window-LBNL|Window LBNL 7.8]] en Windows. La descarga es laboriosa porque depende de **dos librerías externas** que no vienen incluidas.

> "Yo siempre le doy dos vueltas, porque a veces no leo las instrucciones bien. Justo luego ese es mi error."

## Pre-requisitos

- **Sistema operativo: Windows 10/11** (o macOS con [Parallels Desktop](https://www.parallels.com/), VirtualBox, o BootCamp).
- **Acceso a internet** para descargar las tres componentes.
- **~500 MB de espacio libre** (Window 7.8 + dependencias).

> "Una pregunta recurrente: ¿qué computadora compro para trabajar en simulación? Una Windows. Si tienen recursos, una Mac con mucha RAM para Parallels."

## Las tres componentes

| # | Componente | Por qué |
|---|---|---|
| 1 | **Microsoft Visual C++ Redistributable 14 (x86)** | Window usa runtime de Visual Studio |
| 2 | **Intel Fortran Compiler Runtime for Windows** | Window está escrito en Fortran |
| 3 | **Window 7.8 Full Setup** | El programa propiamente |

⚠️ **Orden importa**: instalar `1` y `2` **antes** de `3`. Si se instala Window primero, da error `DLL not found` al ejecutarlo.

## Paso 1 — Microsoft Visual C++ Redistributable

1. Ir a la página de Window LBNL: <https://windows.lbl.gov> → sección Downloads → instrucciones de instalación.
2. Encontrar el link directo a **Microsoft Visual C++ Redistributable**.
3. En la página de Microsoft, descargar la **versión 14, arquitectura x86** (no x64 — la liga del LBNL apunta directo a x86).
4. Ejecutar el instalador → `Accept the license terms` → `Install`.
5. Esperar a que termine (segundos, no minutos).

Tamaño: ~6.7 MB.

> "Click on the link below to access Microsoft Visual Redistributable web page. In selection: visual B14 under architecture X86."

## Paso 2 — Intel Fortran Compiler Runtime

1. Volver a la página de Window LBNL.
2. Link directo a **Intel Fortran Compiler Runtime for Windows**.
3. Descargar la versión recomendada — **IFX 2023.1 o superior**.
4. Ejecutar `w_ifort-runtime_p_2023.1.xxx_offline.exe` (o similar).
5. **Accept all defaults** → `Install` → `Finish`.

> "Acepta todo, instalar, finish."

Esta instalación es **más lenta** (~1-2 min) porque registra el compilador Fortran en el sistema.

## Paso 3 — Window 7.8 Full Setup

Solo después de los pasos 1 y 2:

1. Volver a Window LBNL → Downloads.
2. Buscar **Window 7.8** (la versión estable). **No usar betas**.
3. Descargar **Window Full Setup** (~200 MB).
4. Ejecutar → `Accept` → `Next` → poner el nombre de usuario cuando se pida → `Install` → `Finish`.

> "Quilata."

(El profesor lee "Quilata" en lugar de "Quit / Install / Finish" — efecto de la prisa.)

Window queda disponible en `Start Menu → Recently Added → Window 7.8`.

## Verificación

1. Abrir Window LBNL.
2. **Si carga**: instalación correcta. Aparece la GUI con paneles para `Glass`, `Gap`, `Glazing System`, `Frame`, etc.
3. **Si da error `DLL not found`**: faltó alguna dependencia. Revisar el mensaje de error — suele decir qué `.dll` falta. Reinstalar el componente correspondiente.

## En Mac con Parallels

1. Tener una licencia activa de **Parallels Desktop**.
2. Tener una máquina virtual con Windows 10/11.
3. Iniciar la VM.
4. Seguir Pasos 1-3 **dentro** de la VM.
5. Acceder a Window desde la VM. En Coherence Mode, Window aparece como una app más en el dock del Mac.

> "Yo tengo Parallels y estoy en Windows. Mac y Windows pueden compartir archivos — los OSMs los puedo abrir desde la misma carpeta."

## Después de instalar — siguiente paso

[[Usar-Window-LBNL]] — workflow para crear un sistema de vidrio y exportar SHGC + U-value a Open Studio.

## Recomendación de versionado

- **Estable: 7.8** (mayo 2026).
- **Las betas pueden tener bugs no reportados**. Para producción o tareas críticas (proyecto final), siempre estable.
- Si la versión 7.8 ya está obsoleta cuando lo leas, **buscar la última estable** en LBNL.

## Notas históricas / contexto

- El profesor instaló Window en vivo durante la clase 014 (22 mayo 2026), tomó **~15-20 minutos** incluyendo dependencias.
- La instalación es laboriosa pero **estable** una vez completada — el profesor no la actualiza con frecuencia.
- Antes de versión 7.x, Window incluía sus dependencias en el setup. La separación es relativamente reciente para reducir tamaño del instalador.

## Clases relacionadas

- [[../classes/014-InfiltracionFloorspaceWindowLBNL]] — instalación en vivo

## Ver también

- [[../tools/Window-LBNL]] — para qué sirve
- [[Usar-Window-LBNL]] — workflow básico tras la instalación
- [[../concepts/Solar-Heat-Gain-Coefficient]] — qué calcula Window
