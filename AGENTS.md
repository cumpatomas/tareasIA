# Instrucciones para agentes (Copilot CLI y otros) – Tareas IA

Este repo es un dashboard estático (`index.html` + `data.json`) publicado con GitHub Pages. Las tareas viven en `data.json`.

## Cuando pidan algo sobre "tareas IA"

Si el pedido es crear, agregar, cargar, editar, actualizar, cambiar de estado, listar o consultar tareas de IA:

- **Crear** (a mano, desde un link de Jira/Confluence o desde un archivo): seguí el procedimiento de [tareas-ia-crear](.claude/commands/tareas-ia-crear.md).
- **Editar**: seguí el procedimiento de [tareas-ia-editar](.claude/commands/tareas-ia-editar.md).
- **Listar / consultar**: leé `data.json` y respondé, sin modificar nada.

Esos archivos son la fuente única del procedimiento. Adaptaciones para Copilot:

- `$ARGUMENTS` es el texto que escribió el usuario en su pedido.
- Donde dicen "preguntá con AskUserQuestion", hacé la pregunta en el chat.
- Para leer `.xlsx`, usá la terminal (Python con `openpyxl`) si está disponible; si no, pedí exportar a CSV.
- Para Jira/Confluence, usá una herramienta de Atlassian si está conectada; si no, pedí al usuario que pegue el contenido.

Esquema, reglas y normalizaciones: [esquema.md](.claude/skills/tareas-ia/esquema.md). Leelo siempre antes de tocar `data.json`.

## Reglas que no se negocian

- Nunca inventes datos: si un campo obligatorio no sale de forma determinista, preguntalo.
- `creadoPor`/`editadoPor` son el usuario de GitHub de quien hace el cambio, no el `autor`.
- Mostrá el resumen de cambios y pedí confirmación antes de hacer commit/push.
- Nunca borres tareas ni cambies ids.
- No agregues dependencias externas ni CDNs al HTML.
