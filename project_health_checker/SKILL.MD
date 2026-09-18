---
name: Project_Health_Checker
description: Revisa el estado de salud del proyecto Clínica Salud Total — consistencia y completitud entre PRD, HUs y specs UX, Puntos Abiertos sin resolver, brechas de cobertura entre roles, y (opcional) estado de agentes herdr activos. Use when user runs /Project_Health_Checker, or asks for project status, health check, seguimiento del proyecto, or "cómo va el proyecto".
type: skill
tags: [seguimiento, salud-proyecto, prd, herdr]
---

# Project Health Checker — Estado del proyecto

## Goal

Dar un diagnóstico rápido y honesto del estado del proyecto **Clínica Salud Total**: qué tan
completos y consistentes están `PRD.md`, `HU-*.md` y `Especificacion-UX-*.md`, qué Puntos
Abiertos siguen sin resolver, y dónde hay brechas de handoff entre PM → BA → UX. No modifica
ningún archivo — es solo lectura y reporte.

## Input

Ninguno requerido. Parámetro opcional:

- `--agents` — además del chequeo de documentos, revisa si hay agentes PM/BA/UX vivos en el
  workspace herdr actual y su estado (activo/idle). Sin este flag, el chequeo es solo sobre
  archivos.

## Workflow

1. **Confirmar estructura vigente.** Leer `AGENTS.md` en la raíz del repo — si la tabla de
   "Key Files" cambió, usar esa versión como fuente de verdad en vez de la lista fija de abajo.

2. **Chequear `PRD.md`** (dueño: PM):
   - Existe y tiene las 5 secciones esperadas: Problema, Objetivo, Alcance, Supuestos y Puntos
     Abiertos, Criterios de éxito.
   - Listar cada "Punto abierto" tal cual está escrito — no resolverlos ni inventar respuesta.
   - Confirmar que el Alcance distingue explícitamente qué incluye y qué no.

3. **Chequear `HU-*.md`** (dueño: BA), uno por uno:
   - Tiene criterios de aceptación en formato Gherkin (Given/When/Then).
   - Su alcance cae dentro de lo que `PRD.md` define como incluido — si una HU cubre algo que
     el PRD marcó como fuera de alcance (pagos, historia clínica, seguros), marcarlo como
     inconsistencia, no corregirlo en silencio.
   - Si una HU depende de un Punto Abierto no resuelto (ej. canal exacto: app/WhatsApp/web),
     señalarlo.

4. **Chequear `Especificacion-UX-*.md`** (dueño: UX):
   - Existe y referencia formato Stitch-consumable (markdown determinista, tablas, jerarquías
     explícitas), como pide `AGENTS.md`.
   - Cruzar contra `HU-*.md`: toda HU con impacto en UI debería tener cobertura en la spec UX;
     toda pantalla/flujo en la spec UX debería trazar a al menos una HU. Reportar huecos en
     ambos sentidos.

5. **Handoff PM → BA → UX.** Con los pasos 2–4 ya hechos, resumir en qué etapa está el
   proyecto (ej. "PRD cerrado, HUs completas, UX parcial — falta spec de HU-05") y si el orden
   de entrega se respetó (BA/UX pueden coordinar directo entre sí, pero PM es quien resuelve
   conflictos de alcance).

6. **Si se pasó `--agents`:** revisar agentes herdr vivos.
   - Resolver workspace actual (`HERDR_WORKSPACE_ID`, o `herdr pane get $HERDR_PANE_ID` si no
     está en el entorno).
   - `herdr pane list --workspace <workspace_id>` y `herdr agent list`, cruzarlos igual que hace
     `invoke-agent`, para ver qué roles (`pm`/`ba`/`ux`) están vivos en este workspace y su
     estado.
   - No crear ni invocar ningún agente — este chequeo es de solo lectura.

7. **Entregar el reporte en el chat** (es un diagnóstico puntual, no un artefacto que viva en el
   repo — no se guarda como archivo salvo que el usuario lo pida explícitamente). Estructura
   sugerida:
   - Estado por área: PRD / HUs / UX (verde = completo y consistente, amarillo = incompleto,
     rojo = inconsistente o bloqueado).
   - Puntos Abiertos pendientes (lista textual, sin resolver).
   - Brechas de cobertura HU ↔ UX.
   - Estado de agentes vivos, si se pidió `--agents`.
   - Próxima acción recomendada (a qué rol le toca mover, según playbooks).

## Notas

- Solo lectura: nunca edita `PRD.md`, `HU-*.md` ni `Especificacion-UX-*.md`. Si el chequeo
  encuentra algo que corregir, se lo reporta al usuario — corregirlo es trabajo del rol dueño
  del archivo (`init-agent`/`invoke-agent` con el rol correspondiente).
- No inventar resolución de Puntos Abiertos ni de brechas: reportarlos tal cual, igual que
  exigen los playbooks de PM/BA/UX.
- Si `AGENTS.md` no existe o cambió de estructura, avisar y ajustar el chequeo a lo que
  realmente hay en el repo antes de reportar "faltante" sobre archivos que ya no aplican.
