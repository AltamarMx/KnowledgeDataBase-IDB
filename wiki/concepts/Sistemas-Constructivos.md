# Sistemas Constructivos

Combinación ordenada de capas de materiales que conforman una superficie de la envolvente arquitectónica. Cada capa tiene propiedades térmicas y un espesor definidos.

## Ideas clave

- En Energy Plus, primero se definen los **materiales** individuales (con propiedades térmicas y espesor)
- Luego se ordenan los materiales para crear un **sistema constructivo** (ej. concreto + tabique + concreto para un muro de tabique recubierto)
- El sistema constructivo se asigna a cada superficie de la geometría
- En Open Studio existe el concepto de **Construction Sets** que agrupa los sistemas constructivos para asignarlos de forma más eficiente

## Ejemplo

Un muro de "ladrillo recubierto":
1. Capa exterior: concreto (acabado)
2. Capa media: tabique (ladrillo)
3. Capa interior: concreto (acabado)

## Relación con métodos numéricos

Los coeficientes del **Conduction Transfer Function (CTF)** — método por defecto de EnergyPlus — dependen del sistema constructivo: materiales, espesores y su orden. El sistema constructivo siempre se describe de **exterior a interior**.

## Construction Sets

A partir de la clase 004, se introduce el concepto de **Construction Set**: una agrupación que asigna sistemas constructivos automáticamente a todas las superficies según su tipo y condición de frontera (Outdoor, Surface, Ground, Adiabatic). Esto evita asignar manualmente superficie por superficie — esencial cuando un edificio tiene 200+ superficies.

- Se crean en la pestaña Construction de Open Studio
- Se asignan al edificio completo desde **Facility**
- Las superficies asignadas por el set aparecen en **verde** (default)
- Se puede **sobrescribir** individualmente si una superficie necesita un sistema diferente

## Aparece en

- [[001-IntroduccionTallerIDB]] — Explicación de cómo se definen materiales y construcciones en IDF y Open Studio
- [[002-ConceptosBasicosBalancesCalor]] — Relación con CTF y modelado como "caricatura"
- [[004-InterpretandoMensajesConstructionSets]] — Construction Sets, asignación masiva, verificación visual
- [[006-DosZonasTermicasVentanasAleros]] — Relación densidad-conductividad, pinturas despreciables como capa
