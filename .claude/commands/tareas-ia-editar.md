---
description: Editar una tarea de IA existente en data.json (por id y campos)
argument-hint: [id] [campo=valor ...]
---

Editá una tarea de `data.json`. Antes de nada leé `.claude/skills/tareas-ia/esquema.md` (esquema, reglas, normalizaciones y publicación) y respetalo.

Entrada del usuario: $ARGUMENTS

## Pasos

1. `git pull --rebase`.
2. **Id**: si no vino en `$ARGUMENTS`, pedilo. Si no existe en `data.json`, avisá y pedí otro (podés mostrar los ids y títulos existentes).
3. Mostrá la tarea actual (todos los campos) para dar contexto.
4. **Campos a editar**: si no vinieron en `$ARGUMENTS`, preguntá cuáles y los nuevos valores. Campos editables: `titulo`, `autor`, `tipo`, `descripcion`, `equipo`, `link`, `estado`. No se pueden editar `id` ni `creadoPor`.
5. Validá: `equipo` y `estado` con los valores exactos (normalizá con las tablas; si no encaja, preguntá).
6. Obtené el usuario de GitHub como indica el esquema y escribilo en `editadoPor`, junto con `editadoEn` = fecha de hoy (pisan los anteriores). No toques `creadoPor` ni `creadoEn`.
7. Mostrá un antes/después de cada campo cambiado y pedí confirmación. Si no cambió nada, no hagas commit.
8. Con confirmación: guardá, commit (`tarea-ia: editar #<id> <campos>`) y push como indica el esquema.
