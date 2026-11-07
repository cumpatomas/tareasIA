# Esquema y reglas de `data.json` (tareas IA)

La raíz del repo es el directorio que contiene `data.json` y `index.html`. Todos los comandos trabajan ahí.

## Esquema

```json
{
  "actualizado": "YYYY-MM-DD",
  "tareas": [
    {
      "id": 16,
      "autor": "Nombre de quien lidera/escribió la tarea (texto libre)",
      "titulo": "string",
      "tipo": "string (por defecto: General)",
      "descripcion": "string (puede ser largo)",
      "equipo": "iOS | Android | Backend | QA",
      "link": "https://... (Jira o Confluence; puede ir vacío)",
      "estado": "EN PROGRESO | BLOQUEADA | FINALIZADA | PROGRAMADA",
      "creadoPor": "usuario de GitHub de quien creó la tarea",
      "creadoEn": "YYYY-MM-DD (fecha de creación)",
      "editadoPor": "usuario de GitHub de quien la editó por última vez (solo si fue editada)",
      "editadoEn": "YYYY-MM-DD (fecha de la última edición, solo si fue editada)"
    }
  ]
}
```

## Reglas

- `id`: número automático = máximo `id` existente + 1. Nunca se reutiliza ni se edita. Si se crean varias tareas, ids consecutivos.
- `creadoPor` y `creadoEn` (fecha de hoy) son obligatorios al crear y no se modifican nunca. `editadoPor` y `editadoEn` (fecha de hoy) se escriben juntos en cada edición y pisan el valor anterior. Solo se muestran en la vista de detalle.
- `autor` es la persona dueña de la tarea (dato de negocio); `creadoPor`/`editadoPor` son la identidad técnica de GitHub. No confundirlos ni derivar uno del otro.
- `equipo` y `estado` solo admiten los valores exactos de arriba.
- Al guardar: JSON válido UTF-8, indentado, una tarea por bloque; actualizar `actualizado` con la fecha de hoy (YYYY-MM-DD).
- Nunca inventar datos. Si un campo obligatorio no se puede obtener de forma determinista, preguntarlo al usuario.

## Usuario de GitHub del dev

Obtenerlo en este orden y usar el primero que funcione:

1. `gh api user --jq .login`
2. `git config user.name`
3. Preguntar al usuario.

## Normalización de estado (sin distinguir mayúsculas/tildes)

| Valor final | Se acepta como |
|---|---|
| EN PROGRESO | en progreso, in progress, doing, en curso, en desarrollo |
| BLOQUEADA | bloqueada, blocked, impedida, on hold |
| FINALIZADA | finalizada, done, closed, resolved, completada, terminada, hecha |
| PROGRAMADA | programada, to do, todo, backlog, planned, selected for development, open, abierta, pendiente |

Cualquier otro valor no se adivina: se pregunta.

## Normalización de equipo

`ios`→iOS · `android`→Android · `backend`, `back-end`, `back end`→Backend · `qa`, `quality assurance`→QA. Cualquier otro valor se pregunta.

## Sinónimos de columnas/campos (para importar desde archivos o Jira)

Se comparan normalizados (minúsculas, sin tildes, sin espacios extra) y por igualdad exacta, no por parecido:

| Campo | Nombres aceptados |
|---|---|
| titulo | titulo, title, summary, resumen, nombre, tarea |
| autor | autor, author, reporter, creador, owner, responsable, assignee |
| descripcion | descripcion, detalle, detalles, description |
| equipo | equipo, team, squad, componente, component |
| tipo | tipo, type, issuetype, issue type |
| link | link, url, enlace |
| estado | estado, status |

Si dos columnas coinciden con el mismo campo, preguntar cuál usar.

## Confirmación y publicación

1. `git pull --rebase` antes de editar.
2. Mostrar un resumen de los cambios (tareas nuevas o antes/después) y **pedir confirmación**.
3. Con confirmación: `git add data.json`, commit con mensaje corto (`tarea-ia: crear #16 "Título"` / `tarea-ia: editar #4 estado`) y `git push`.
4. Informar qué cambió. El sitio se republica en ~1 minuto.
