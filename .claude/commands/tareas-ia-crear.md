---
description: Crear tareas de IA en data.json (manual, desde un link de Jira/Confluence o desde un archivo)
argument-hint: [manual | <url Jira/Confluence> | <ruta de archivo .md/.xlsx/.csv/.json/...>]
---

Creá una o más tareas en `data.json`. Antes de nada leé `.claude/skills/tareas-ia/esquema.md` (esquema, reglas, normalizaciones y publicación) y respetalo.

Entrada del usuario: $ARGUMENTS

## 1. Elegir el modo

- Si `$ARGUMENTS` empieza con `http://` o `https://` → modo **2 (link)**.
- Si es una ruta a un archivo existente → modo **3 (archivo)**.
- Si dice `manual` → modo **1**.
- Si está vacío o es ambiguo → preguntá con AskUserQuestion: 1) Manual, 2) Link de Jira/Confluence, 3) Archivo.

## Modo 1: manual

Pedí al usuario estos datos (puede darlos todos juntos en un mensaje):

- **titulo**
- **autor** (persona dueña de la tarea)
- **detalle** → se guarda en `descripcion`
- **equipo** (iOS, Android, Backend o QA)
- **estado** (EN PROGRESO, BLOQUEADA, FINALIZADA o PROGRAMADA)

Opcionales, solo si el usuario los menciona: `tipo` (si no, `General`) y `link` (si no, vacío). No los preguntes.

## Modo 2: link de Jira o Confluence

1. Obtené el contenido de la URL con una herramienta de Atlassian si hay un MCP conectado. Si no hay ninguna, o la URL no es accesible (requiere login), pedile al usuario que pegue el contenido de la página/issue. No uses WebFetch contra URLs privadas.
2. Mapeo determinista:
   - **Jira** (`/browse/KEY-123`): `summary`→titulo · `description`→descripcion · `reporter` (nombre visible)→autor · `issuetype`→tipo · `status`→estado (según la tabla de normalización) · `equipo` solo si una *component* o *label* coincide exactamente con uno de los 4 equipos · `link`=la URL.
   - **Confluence** (`/wiki/...`): título de la página→titulo · texto del cuerpo→descripcion (sin recortar, en texto plano) · creador de la página→autor · `tipo`=`Documentación` · `link`=la URL. El estado y el equipo no existen en Confluence: preguntarlos.
3. Todo campo obligatorio que no salga de forma determinista se pregunta. No lo deduzcas.

## Modo 3: archivo

1. Leé el archivo según su formato: `.md`/`.txt` (tabla Markdown o secciones con encabezados claros), `.csv`, `.json`, `.xlsx` (usá Python con `openpyxl` vía Bash si está disponible; si no, decí qué falta y pedí exportarlo a CSV), u otros formatos legibles.
2. Cada fila/registro es una tarea. Asigná columnas con la tabla de **sinónimos** del esquema (igualdad exacta tras normalizar). Las columnas que no coincidan se ignoran y se lo informás al usuario.
3. Normalizá `estado` y `equipo` con las tablas. Los valores que no encajen no se adivinan: listalos y preguntá cómo resolverlos (una pregunta por valor distinto, no por fila).
4. Si faltan campos obligatorios en todas las filas (por ejemplo no hay columna de equipo), preguntá un valor único para el lote.
5. Detectá duplicados contra `data.json` (mismo `link`, o mismo `titulo`+`equipo`) y avisá antes de crearlos.

## Todos los modos

1. Obtené el usuario de GitHub (`creadoPor`) como indica el esquema, y poné `creadoEn` = fecha de hoy.
2. `git pull --rebase`, asigná ids consecutivos desde el máximo + 1.
3. Mostrá una tabla con las tareas a crear (id, equipo, estado, titulo, autor) y pedí confirmación.
4. Con confirmación: guardá, commit y push como indica el esquema. Informá los ids creados.
