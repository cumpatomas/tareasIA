---
mode: agent
description: Editar una tarea de IA existente en data.json (por id y campos)
---

Editá una tarea de IA existente en `data.json`.

Seguí el procedimiento de [tareas-ia-editar](../../.claude/commands/tareas-ia-editar.md) y las reglas de [esquema.md](../../.claude/skills/tareas-ia/esquema.md), con estas adaptaciones:

- `$ARGUMENTS` es lo que el usuario escribió junto a este prompt (id y/o campos). Lo que falte, pedilo en el chat.
- Registrá `editadoPor` (usuario de GitHub) y `editadoEn` (fecha de hoy). No toques `creadoPor` ni `creadoEn`.
- Pedí confirmación con el antes/después antes de commit/push.
