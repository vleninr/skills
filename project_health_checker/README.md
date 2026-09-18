# project_health_checker

Diagnóstico de estado y avance de cualquier proyecto. Solo lectura, sin edición.

## What it does

Detecta la estructura real del proyecto (código, documentación/specs, multi-agente, mixto)
antes de decidir qué chequear — no asume un tipo de proyecto fijo. Prioriza `AGENTS.md` /
`CLAUDE.md` como fuente de verdad si existen.

Reporta:

- Tipo de proyecto detectado y en qué se basó.
- Completitud: README, tests/CI (código); secciones esperadas y checklists sin marcar
  (docs/specs).
- Marcadores explícitos de trabajo pendiente — `TODO`, `FIXME`, `TBD`, "Puntos Abiertos",
  "Pendiente", "WIP" — listados tal cual, sin resolverlos.
- Consistencia entre artefactos relacionados (ej. requisitos vs. historias de usuario vs.
  diseño, o specs vs. código implementado): qué falta contraparte y qué no traza a ningún
  origen.
- Estado de git: cambios sin commitear, divergencia de rama, actividad reciente.
- Opcional (`--agents`): si el proyecto usa un sistema multi-agente, qué agentes están
  vivos/activos y su estado.

No modifica ningún archivo ni ejecuta nada destructivo. Si encuentra algo que corregir, lo
reporta — corregirlo es trabajo de quien sea dueño de ese artefacto.

## How to invoke

```
/Project_Health_Checker            # chequeo de estado y avance
/Project_Health_Checker --agents   # + estado de agentes vivos, si el proyecto es multi-agente
```

También se dispara con frases como "estado del proyecto", "cómo va el proyecto", "avance",
"health check".

## Output

Reporte en el chat (no se guarda como archivo, salvo que se pida):

- Tipo de proyecto detectado (una línea).
- Estado por área relevante — Código / Docs / Consistencia / Git — en verde (completo y
  consistente), amarillo (incompleto) o rojo (inconsistente, bloqueado o estancado).
- Puntos pendientes/abiertos, listados tal cual.
- Brechas de cobertura o consistencia entre artefactos, si aplica.
- Estado de agentes vivos, si se pidió `--agents`.
- Próxima acción recomendada, concreta y accionable.

## See also

- [`SKILL.md`](./SKILL.MD): instrucciones completas para el LLM
- `AGENTS.md` / `CLAUDE.md` del proyecto (si existen): fuente de verdad que esta skill
  prioriza antes de aplicar chequeos genéricos
