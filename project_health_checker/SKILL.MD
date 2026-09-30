---
name: Project_Health_Checker
description: Revisa el estado de salud del proyecto actual — consistencia y completitud entre PRD, HUs y specs UX, Puntos Abiertos sin resolver, brechas de cobertura entre roles, y (opcional) estado de agentes herdr activos. Use when user runs /Project_Health_Checker, or asks for project status, health check, seguimiento del proyecto, or "cómo va el proyecto".
type: skill
tags: [seguimiento, salud-proyecto, prd, herdr]
---

# Project Health Checker — Estado del proyecto

## Goal

Dar un diagnóstico rápido y honesto del estado del proyecto en el que se invoca (cualquier
repo que siga la estructura PRD → HUs → specs UX descrita en su `AGENTS.md`): qué tan completos y consistentes están `PRD.md`, `HU-*.md` y `Especificacion-UX-*.md`, qué Puntos
Abiertos siguen sin resolver, y dónde hay brechas de handoff entre los roles, en el orden que
declare el `AGENTS.md` del proyecto. No modifica
ningún archivo — es solo lectura y reporte.

## Input

Ninguno requerido. Parámetro opcional:

- `--agents` — además del chequeo de documentos, revisa si hay agentes vivos de los roles del
  proyecto (los que declare `AGENTS.md`; por defecto PM/BA/UX) en el
  workspace herdr actual y su estado (activo/idle). Sin este flag, el chequeo es solo sobre
  archivos.

## Workflow

1. **Confirmar estructura vigente.** Leer `AGENTS.md` en la raíz del repo y extraer:
   - La tabla de "Key Files" — si cambió, usar esa versión como fuente de verdad en vez de la
     lista fija de abajo.
   - Los roles del proyecto y el **orden de handoff** declarado (ej. `PM → BA → ARCH → UX`).
     Si `AGENTS.md` no declara un orden, usar `PM → BA → UX` como valor por defecto y avisarlo
     en el reporte.

2. **Chequear `PRD.md`** (dueño: PM):
   - Existe y tiene las secciones que exige su template. Si el PRD se generó con
     `/generar-prd-conversacional`, usar las secciones de ese template; si no, esperar como
     mínimo: Problema, Objetivo, Alcance (incluye / no incluye), Supuestos y Puntos Abiertos,
     Criterios de éxito.
   - Puntos Abiertos pueden estar en una sección propia o en "Supuestos y riesgos abiertos",
     además de marcadores inline `<!-- REVISAR: ... -->` — contar ambos.
   - Listar cada "Punto abierto" tal cual está escrito — no resolverlos ni inventar respuesta.
   - Confirmar que el Alcance distingue explícitamente qué incluye y qué no.

3. **Chequear `HU-*.md`** (dueño: BA), uno por uno:
   - Tiene criterios de aceptación en formato Gherkin (Given/When/Then).
   - Su alcance cae dentro de lo que `PRD.md` define como incluido — si una HU cubre algo que
     el PRD marcó como fuera de alcance (leer la lista real de "No alcance" / "No incluye" del
     PRD del proyecto), marcarlo como
     inconsistencia, no corregirlo en silencio.
   - Si una HU depende de un Punto Abierto no resuelto del PRD, señalarlo citando el punto.

4. **Chequear `Especificacion-UX-*.md`** (dueño: UX):
   - Existe y referencia formato Stitch-consumable (markdown determinista, tablas, jerarquías
     explícitas), como pide `AGENTS.md`.
   - Cruzar contra `HU-*.md`: toda HU con impacto en UI debería tener cobertura en la spec UX;
     toda pantalla/flujo en la spec UX debería trazar a al menos una HU. Reportar huecos en
     ambos sentidos.

4b. **Chequear artefactos de otros roles declarados.** Para cada rol del handoff que no sea
   PM/BA/UX (ej. ARCH con Technical Design + ADRs), verificar que su artefacto exista según
   `AGENTS.md` y que trace al PRD/HUs. Si `AGENTS.md` no dice dónde vive ese artefacto,
   reportarlo como "ubicación no declarada" — no adivinar.

5. **Handoff según el orden declarado.** Con los pasos 2–4b ya hechos, resumir en qué etapa
   está el proyecto siguiendo el orden de handoff extraído en el paso 1 (ej. "PRD cerrado, HUs
   completas, Tech Design pendiente, UX no iniciado") y si el orden de entrega se respetó
   (un rol produjo su artefacto antes de que el rol previo cerrara el suyo) (BA/UX pueden coordinar directo entre sí, pero PM es quien resuelve
   conflictos de alcance).

6. **Si se pasó `--agents`:** revisar agentes herdr vivos.
   - Resolver workspace actual (`HERDR_WORKSPACE_ID`, o `herdr pane get $HERDR_PANE_ID` si no
     está en el entorno).
   - `herdr pane list --workspace <workspace_id>` y `herdr agent list`, cruzarlos igual que hace
     `invoke-agent`, para ver qué roles del proyecto (en minúsculas, ej. `pm`/`ba`/`arch`/`ux`)
     están vivos en este workspace y su
     estado.
   - No crear ni invocar ningún agente — este chequeo es de solo lectura.

7. **Entregar el reporte en el chat** (es un diagnóstico puntual, no un artefacto que viva en el
   repo — no se guarda como archivo salvo que el usuario lo pida explícitamente). Estructura
   sugerida:
   - Estado por área, una por rol del handoff (ej. PRD / HUs / Tech Design / UX) (verde = completo y consistente, amarillo = incompleto,
     rojo = inconsistente o bloqueado).
   - Puntos Abiertos pendientes (lista textual, sin resolver).
   - Brechas de cobertura HU ↔ UX.
   - Estado de agentes vivos, si se pidió `--agents`.
   - Próxima acción recomendada (a qué rol le toca mover, según playbooks).

## Notas

- Solo lectura: nunca edita `PRD.md`, `HU-*.md`, `Especificacion-UX-*.md` ni artefactos de otros
  roles. Si el chequeo
  encuentra algo que corregir, se lo reporta al usuario — corregirlo es trabajo del rol dueño
  del archivo (`init-agent`/`invoke-agent` con el rol correspondiente).
- No inventar resolución de Puntos Abiertos ni de brechas: reportarlos tal cual, igual que
  exigen los playbooks de cada rol.
- Si `AGENTS.md` no existe o cambió de estructura, avisar y ajustar el chequeo a lo que
  realmente hay en el repo antes de reportar "faltante" sobre archivos que ya no aplican.
