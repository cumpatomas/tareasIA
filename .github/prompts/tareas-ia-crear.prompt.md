---
mode: agent
description: Crear tareas de IA en data.json (manual, desde un link de Jira/Confluence o desde un archivo)
---

Creá una o más tareas de IA en `data.json`.

Seguí el procedimiento de [tareas-ia-crear](../../.claude/commands/tareas-ia-crear.md) y las reglas de [esquema.md](../../.claude/skills/tareas-ia/esquema.md), con estas adaptaciones:

- `$ARGUMENTS` es lo que el usuario escribió junto a este prompt (modo `manual`, una URL de Jira/Confluence o la ruta de un archivo). Si no escribió nada, preguntale en el chat cuál de los 3 modos quiere: 1) manual, 2) link de Jira/Confluence, 3) archivo.
- Las preguntas al usuario se hacen en el chat.
- Pedí confirmación con el resumen de cambios antes de commit/push.
