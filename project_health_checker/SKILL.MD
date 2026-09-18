---
name: Project_Health_Checker
description: Da un diagnóstico de salud y avance del proyecto actual, sea del tipo que sea (código, documentación/specs, mixto) — detecta la estructura real del repo, chequea completitud, puntos abiertos/bloqueos, consistencia entre artefactos, y estado de git. Use when user runs /Project_Health_Checker, or asks for project status, health check, seguimiento del proyecto, avance, or "cómo va el proyecto".
type: skill
tags: [seguimiento, salud-proyecto, genérico, git]
---

# Project Health Checker — Estado y avance del proyecto

## Goal

Dar un diagnóstico rápido y honesto del estado del proyecto en el que se invoque esta skill —
sin asumir de antemano qué tipo de proyecto es. Detecta la estructura real (código, docs/specs,
multi-agente, mixto) y adapta el chequeo a eso. Es solo lectura: nunca edita, resuelve ni
inventa contenido de los archivos que revisa.

## Input

Ninguno requerido. Parámetro opcional:

- `--agents` — si el proyecto usa un sistema multi-agente (ej. herdr, con roles definidos en
  `AGENTS.md` u otro archivo equivalente), además revisa qué agentes están vivos/activos y su
  estado. Sin este flag, el chequeo es solo sobre archivos y git.

## Workflow

1. **Detectar convenciones propias del proyecto primero.** Buscar `AGENTS.md`, `CLAUDE.md`, o
   equivalente en la raíz. Si existe, leerlo completo — ahí suele estar la fuente de verdad
   sobre: qué archivos son clave, quién es dueño de cada uno, qué reglas de proceso aplican
   (ej. "no inventar requisitos", orden de handoff entre roles, dónde marcar incertidumbre).
   Usar esas reglas para decidir qué chequear y cómo, en vez de una lista fija.

2. **Clasificar el tipo de proyecto** con lo que hay en el repo, sin preguntar primero:
   - **Código**: presencia de `package.json`, `pyproject.toml`, `go.mod`, `Cargo.toml`, carpeta
     `src/`, tests, CI config (`.github/workflows`, etc.).
   - **Documentación/specs**: archivos markdown de planificación (PRD, backlog, historias de
     usuario, specs de diseño, ADRs) sin build/tests.
   - **Mixto**: ambos presentes.
   - **Multi-agente**: `AGENTS.md` describe roles/playbooks y un CLI de coordinación (ej.
     herdr) — habilita el chequeo `--agents`.

3. **Chequeo de completitud**, según lo que aplique:
   - *Código*: ¿hay `README`? ¿hay tests y corren (si es rápido y seguro ejecutarlos, no
     asumir)? ¿hay lint/build configurado? ¿hay TODOs/FIXMEs sin trackear en issues?
   - *Docs/specs*: ¿cada documento de planificación tiene las secciones que su propia
     convención espera (objetivo, alcance, criterios de éxito, etc.)? ¿hay checklists con
     items sin marcar?
   - En ambos casos: buscar marcadores explícitos de incertidumbre o trabajo pendiente en el
     texto — "TODO", "FIXME", "TBD", "Puntos Abiertos", "Pendiente", "WIP" — y listarlos tal
     cual, sin resolverlos.

4. **Chequeo de consistencia entre artefactos**, si el proyecto tiene más de un tipo de
   documento que debería estar alineado (ej. specs de producto vs. historias de usuario vs.
   diseño UI; o requirements vs. código implementado). Cruzar referencias y señalar:
   - Algo definido en un artefacto "fuente" (ej. PRD/backlog) que no tiene contraparte en los
     artefactos derivados (ej. ninguna historia lo cubre).
   - Algo en un artefacto derivado que no traza a ningún requisito de origen (posible scope
     creep).
   - No corregir estas brechas — reportarlas.

5. **Chequeo de git**, si el directorio es un repo git:
   - `git status` — cambios sin commitear, archivos untracked relevantes.
   - Rama actual vs. rama principal — ahead/behind, si diverge.
   - Fecha del último commit — señal de proyecto activo vs. estancado.
   - No ejecutar nada destructivo ni hacer commit/push como parte de este chequeo.

6. **Si se pasó `--agents` y el proyecto es multi-agente:** revisar agentes vivos y su estado
   (activo/idle/bloqueado) usando el CLI que `AGENTS.md` describa, cruzando panes/workspace con
   agentes igual que haría una skill de invocación de agentes ya existente en el proyecto (si
   la hay). No crear ni invocar ningún agente — solo leer estado.

7. **Entregar el reporte en el chat** (diagnóstico puntual, no se guarda como archivo salvo que
   el usuario lo pida). Estructura sugerida, adaptada a lo que aplicó en los pasos anteriores:
   - Tipo de proyecto detectado y en qué se basó la detección (una línea).
   - Estado por área relevante (ej. Código / Docs / Consistencia / Git) en verde (completo y
     consistente), amarillo (incompleto) o rojo (inconsistente, bloqueado o estancado).
   - Puntos pendientes/abiertos encontrados, listados tal cual.
   - Brechas de cobertura o consistencia entre artefactos, si aplicó el paso 4.
   - Estado de agentes vivos, si se pidió `--agents`.
   - Próxima acción recomendada, concreta y accionable.

## Notas

- Solo lectura: nunca edita, corrige ni completa los archivos que revisa. Corregir lo
  encontrado es trabajo de quien sea dueño de ese artefacto, no de esta skill.
- No inventar resolución de puntos pendientes ni de brechas: reportarlos tal cual estén, aunque
  el proyecto no tenga una convención explícita para marcarlos.
- Si el proyecto no tiene `AGENTS.md`/`CLAUDE.md` ni ninguna convención detectable, decirlo
  explícitamente y basar el chequeo solo en señales genéricas (git, TODOs, README, tests) en
  vez de simular que existe una estructura que no está.
- Si `AGENTS.md` (o equivalente) cambió de estructura desde la última vez, priorizar siempre lo
  que hay ahora en el repo sobre cualquier supuesto previo.
