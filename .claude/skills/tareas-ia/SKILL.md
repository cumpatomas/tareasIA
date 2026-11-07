---
name: tareas-ia
description: Gestiona las "tareas IA" del dashboard (data.json del repo). Usar cuando el usuario pida crear, agregar, cargar, editar, actualizar, cambiar de estado, listar o consultar tareas de IA / "tareas ia", incluso desde un link de Jira o Confluence o desde un archivo.
---

Gestionás las tareas del dashboard de IA, que viven en `data.json` en la raíz del repo. Leé siempre `esquema.md` (en esta carpeta) antes de tocar datos.

## Qué hacer según el pedido

- **Crear / agregar / cargar tareas** (a mano, desde un link de Jira o Confluence, o desde un archivo): leé `.claude/commands/tareas-ia-crear.md` y ejecutá ese procedimiento completo, tomando el pedido del usuario como entrada.
- **Editar / cambiar estado / reasignar / corregir una tarea**: leé `.claude/commands/tareas-ia-editar.md` y ejecutá ese procedimiento completo.
- **Listar / consultar / resumir** (por ejemplo "qué está bloqueado en Backend"): leé `data.json` y respondé. No modifiques nada ni hagas commit.

Podés sugerir al usuario que también puede usar directamente `/tareas-ia-crear` y `/tareas-ia-editar`.

## Reglas que no se negocian

- Nunca inventes datos: si un campo obligatorio no sale de forma determinista, preguntalo.
- `creadoPor` y `editadoPor` salen del usuario de GitHub del dev, no del `autor`.
- Siempre pedí confirmación mostrando el resumen de cambios antes de hacer `git push`.
- Nunca borres tareas ni cambies ids.
