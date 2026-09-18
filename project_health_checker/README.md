# project_health_checker

Diagnóstico de estado del proyecto Clínica Salud Total. Solo lectura, sin edición.

## What it does

Revisa `PRD.md`, `HU-*.md` y `Especificacion-UX-*.md` y reporta:

- Completitud de `PRD.md` (Problema, Objetivo, Alcance, Supuestos y Puntos Abiertos,
  Criterios de éxito).
- Puntos Abiertos pendientes, listados tal cual — nunca resueltos ni inventados.
- Consistencia de alcance: HUs que cubren algo marcado fuera de alcance en el PRD.
- Brechas de cobertura HU ↔ UX en ambos sentidos (HU sin spec, spec sin HU).
- En qué etapa del handoff PM → BA → UX está el proyecto.
- Opcional (`--agents`): qué agentes herdr (`pm`/`ba`/`ux`) están vivos en el workspace
  actual y su estado.

No modifica ningún archivo. Si encuentra algo que corregir, lo reporta — corregirlo es
trabajo del rol dueño del documento (`init-agent` / `invoke-agent`).

## How to invoke

```
/Project_Health_Checker            # chequeo de documentos
/Project_Health_Checker --agents   # + estado de agentes herdr vivos
```

También se dispara con frases como "estado del proyecto", "cómo va el proyecto",
"health check".

## Output

Reporte en el chat (no se guarda como archivo, salvo que se pida):

- Estado por área — PRD / HUs / UX — en verde (completo y consistente), amarillo
  (incompleto) o rojo (inconsistente o bloqueado).
- Puntos Abiertos pendientes.
- Brechas de cobertura HU ↔ UX.
- Estado de agentes vivos, si se pidió `--agents`.
- Próxima acción recomendada, según a qué rol le toca mover.

## See also

- [`SKILL.md`](./SKILL.MD): instrucciones completas para el LLM
- [`AGENTS.md`](../../../AGENTS.md): estructura de archivos y roles del proyecto
- `init-agent` / `invoke-agent`: para actuar sobre lo que este chequeo reporta
