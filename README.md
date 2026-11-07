# Tareas IA – Dashboard

Dashboard estático (HTML + JSON) con las tareas de IA de la empresa: tarjetas, resumen por estado/equipo, filtros, detalle en ventana flotante y exportación a PDF/Excel. Se publica con GitHub Pages; los datos viven en `data.json`.

## Funcionalidades

**Dashboard (`index.html`)**
- Resumen general arriba: total, contador por estado (EN PROGRESO, BLOQUEADA, FINALIZADA, PROGRAMADA) y tabla equipo × estado.
- Filtros combinables por equipo y por estado, separados visualmente, con "Limpiar filtros".
- Secciones por equipo (iOS 🍎, Android 🤖, Backend ⚙️, QA 🧪) con ícono, contador y divisor de ancho completo; una tarjeta por tarea con color pastel según estado (azul, rosa, verde, amarillo).
- Cada tarjeta es clicable (y accesible por teclado): abre una ventana flotante con el detalle completo y scroll si el texto es largo. Solo ahí se muestra, en chico, "Creado/Editado por @usuario el fecha".
- Botón "Exportar": filtros por equipo, autor y estado; exporta a Excel (CSV con separador `;`) o a PDF (vista de impresión A4 apaisado, "Guardar como PDF").
- Seguridad: todo texto de `data.json` se escapa antes de insertarse y solo se enlazan URLs `http(s)`.

**Datos (`data.json`)**

```json
{
  "actualizado": "YYYY-MM-DD",
  "tareas": [
    {
      "id": 1,
      "autor": "Persona dueña de la tarea",
      "titulo": "string",
      "tipo": "string (por defecto General)",
      "descripcion": "string (puede ser largo)",
      "equipo": "iOS | Android | Backend | QA",
      "link": "https://... (Jira o Confluence, puede ir vacío)",
      "estado": "EN PROGRESO | BLOQUEADA | FINALIZADA | PROGRAMADA",
      "creadoPor": "usuario de GitHub",
      "creadoEn": "YYYY-MM-DD",
      "editadoPor": "usuario de GitHub (solo si fue editada)",
      "editadoEn": "YYYY-MM-DD (solo si fue editada)"
    }
  ]
}
```

`id` es automático (máximo + 1), inmutable y nunca se reutiliza. `autor` es el dueño de negocio; `creadoPor`/`editadoPor` son el usuario de GitHub de quien hizo el cambio (`gh api user --jq .login`, con `git config user.name` de respaldo).

## Uso con Claude Code

- **`/tareas-ia-crear`**: crea tareas en 3 modos.
  1. *Manual*: pide título, autor, detalle, equipo y estado (tipo y link opcionales).
  2. *Link de Jira o Confluence*: lee el contenido (MCP de Atlassian si hay; si no, el usuario lo pega) y completa los campos con un mapeo fijo; lo que no sale de forma determinista se pregunta.
  3. *Archivo* (`.md`, `.csv`, `.json`, `.xlsx`, etc.): una tarea por fila, columnas asignadas por nombre exacto con una tabla de sinónimos; avisa duplicados y valores que no encajan.
- **`/tareas-ia-editar`**: pide el id y los campos a cambiar; registra `editadoPor` y `editadoEn`. No permite editar `id` ni `creadoPor`.
- **Skill `tareas-ia`**: se activa sola ante pedidos sobre "tareas ia" (crear, editar, listar, consultar) y sigue el procedimiento de los comandos.
- Todo cambio muestra un resumen y **pide confirmación antes del `git push`**. Nunca se inventan datos ni se borran tareas.

## Uso con GitHub Copilot (Android Studio y CLI)

Los comandos y la skill de `.claude/` son de Claude Code. Para Copilot el repo trae equivalentes que **reutilizan los mismos procedimientos** (única fuente de verdad: `.claude/commands/*.md` y `.claude/skills/tareas-ia/esquema.md`):

- **`.github/prompts/tareas-ia-crear.prompt.md` y `tareas-ia-editar.prompt.md`**: archivos de prompt para el chat de Copilot en modo agente (plugin de JetBrains / Android Studio). Se invocan como `/tareas-ia-crear` y `/tareas-ia-editar` si tu versión del plugin soporta prompt files.
- **`.github/copilot-instructions.md` y `AGENTS.md`**: instrucciones que Copilot (IDE, CLI y agente de GitHub) carga solas; hacen que pedidos del tipo "agregá una tarea ia ..." sigan el procedimiento correcto aunque no uses el slash command.
- En **Copilot CLI** (y si tu plugin no soporta prompt files) escribí el pedido en lenguaje natural, por ejemplo: `tareas ia: crear desde https://.../browse/IA-123`.

Soporte según versión: confirmá con tu admin qué funciones (prompt files, instrucciones del repo, AGENTS.md) están habilitadas en tu versión y política de la organización.

## Prompt para el agente de Copilot

Copiá el bloque siguiente y pegalo al agente de Copilot, en el repo nuevo de la organización (vacío).

````text
Quiero crear en este repositorio un dashboard estático de tareas de IA, publicado con GitHub Pages. No uses frameworks ni build: solo HTML, CSS y JS vanilla.

ARCHIVOS A CREAR (en la raíz de la rama main, con el contenido EXACTO que aparece al final, entre los marcadores INICIO/FIN, creando las carpetas necesarias):
1. index.html
2. data.json
3. .claude/commands/tareas-ia-crear.md
4. .claude/commands/tareas-ia-editar.md
5. .claude/skills/tareas-ia/SKILL.md
6. .claude/skills/tareas-ia/esquema.md
7. .github/copilot-instructions.md
8. AGENTS.md
9. .github/prompts/tareas-ia-crear.prompt.md
10. .github/prompts/tareas-ia-editar.prompt.md
11. .nojekyll   (archivo vacío)
12. .gitignore  (una sola línea: .idea/)

REQUISITOS (ya implementados en el index.html provisto; no los cambies):
- index.html carga data.json con fetch y renderiza todo en el cliente.
- Cada tarea tiene: id (número, automático, inmutable), autor, titulo, tipo, descripcion, equipo, link (Jira o Confluence), estado, creadoPor, creadoEn y, si fue editada, editadoPor y editadoEn.
- equipo in {iOS, Android, Backend, QA}. estado in {EN PROGRESO, BLOQUEADA, FINALIZADA, PROGRAMADA}, con colores pastel: azul, rosa, verde, amarillo.
- Arriba: resumen con total, contadores por estado y tabla equipo x estado. Filtros combinables por equipo y estado, con botón "Limpiar filtros".
- Abajo: una sección por equipo con ícono, nombre, contador y línea divisoria de ancho completo, y una tarjeta por tarea.
- Cada tarjeta es clicable y abre una ventana flotante con el detalle (scroll si es largo). Solo ahí se muestra, en chico, "Creado/Editado por @usuario el fecha".
- Botón "Exportar": filtros por equipo, autor y estado; exporta a Excel (CSV con separador ;) o PDF (vista de impresión).
- Todo texto de data.json se escapa antes de insertarse en el HTML y solo se enlazan URLs http(s).

PASOS:
1. Creá los 12 archivos y hacé commit a main con el mensaje "feat: dashboard de tareas IA".
2. No agregues dependencias externas, CDNs ni analítica.
3. Verificá que data.json sea JSON válido y que index.html lo cargue sin errores.
4. No toques la configuración del repo. Al terminar, dame como lista lo que debo hacer yo en Settings: activar Pages desde main, carpeta / (root), y visibilidad del sitio en Private.
5. Las URLs empresa.atlassian.net, los autores y los usuarios de data.json son datos de ejemplo: dejalos tal cual y avisame que hay que reemplazarlos.

===== INICIO index.html =====
<!doctype html>
<html lang="es">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Tareas IA</title>
<style>
  :root {
    --bg:#f6f7fb; --panel:#fff; --fg:#1f2430; --muted:#6b7280; --line:#e5e7ee;
    --prog:#cfe3ff; --prog-t:#1d4f91;
    --bloq:#ffd3d3; --bloq-t:#9b2c2c;
    --fin:#d4f1dc; --fin-t:#256b3a;
    --pro:#fff0c2; --pro-t:#8a6212;
  }
  @media (prefers-color-scheme: dark) {
    :root {
      --bg:#14161c; --panel:#1d2029; --fg:#eceef4; --muted:#9aa1b2; --line:#2c303c;
      --prog:#27405f; --prog-t:#bcd7ff;
      --bloq:#5e2a2e; --bloq-t:#ffc9c9;
      --fin:#25483199; --fin-t:#bfe9cb;
      --pro:#5a4a1e; --pro-t:#ffe7a3;
    }
  }
  * { box-sizing:border-box; }
  body { margin:0; background:var(--bg); color:var(--fg); font-family:system-ui,-apple-system,"Segoe UI",sans-serif; }
  .wrap { max-width:1200px; margin:0 auto; padding:20px 16px 48px; }
  header { display:flex; justify-content:space-between; align-items:baseline; flex-wrap:wrap; gap:8px; margin-bottom:16px; }
  h1 { font-size:1.5rem; margin:0; }
  .meta { color:var(--muted); font-size:.85rem; }

  .estado-prog { background:var(--prog); color:var(--prog-t); }
  .estado-bloq { background:var(--bloq); color:var(--bloq-t); }
  .estado-fin  { background:var(--fin);  color:var(--fin-t); }
  .estado-pro  { background:var(--pro);  color:var(--pro-t); }

  /* Resumen */
  .totales { display:grid; grid-template-columns:repeat(auto-fit,minmax(140px,1fr)); gap:10px; margin-bottom:14px; }
  .total { border-radius:12px; padding:12px 14px; }
  .total .n { font-size:1.9rem; font-weight:700; line-height:1; }
  .total .l { font-size:.78rem; margin-top:4px; letter-spacing:.04em; }
  .total.todas { background:var(--panel); border:1px solid var(--line); }
  .matriz { background:var(--panel); border:1px solid var(--line); border-radius:12px; overflow-x:auto; margin-bottom:20px; }
  table { border-collapse:collapse; width:100%; min-width:520px; font-size:.9rem; }
  th, td { padding:9px 12px; text-align:center; border-bottom:1px solid var(--line); }
  th:first-child, td:first-child { text-align:left; font-weight:600; }
  tr:last-child td { border-bottom:0; }
  thead th { font-size:.72rem; letter-spacing:.05em; color:var(--muted); font-weight:600; }
  td.zero { color:var(--muted); opacity:.5; }

  /* Filtros */
  .filtros { display:flex; flex-wrap:wrap; gap:14px 28px; align-items:center; background:var(--panel); border:1px solid var(--line); border-radius:12px; padding:14px 16px; margin-bottom:28px; }
  .grupo { display:flex; flex-wrap:wrap; gap:6px; align-items:center; }
  .grupo + .grupo { border-left:2px solid var(--line); padding-left:28px; }
  .grupo > span { font-size:.75rem; color:var(--muted); font-weight:700; letter-spacing:.06em; margin-right:6px; }
  @media (max-width:760px) {
    .grupo { width:100%; }
    .grupo + .grupo { border-left:0; padding-left:0; border-top:2px solid var(--line); padding-top:14px; }
    #limpiar { margin-left:0; }
  }
  .chip { border:1px solid var(--line); background:transparent; color:var(--fg); border-radius:999px; padding:5px 12px; font-size:.82rem; cursor:pointer; font-family:inherit; }
  .chip[aria-pressed="true"] { outline:2px solid var(--fg); outline-offset:-1px; font-weight:600; }
  .chip.estado-prog, .chip.estado-bloq, .chip.estado-fin, .chip.estado-pro { border-color:transparent; }
  #limpiar { margin-left:auto; border:0; background:none; color:var(--muted); cursor:pointer; font-size:.82rem; text-decoration:underline; font-family:inherit; }

  /* Secciones y tarjetas */
  section.equipo { margin-bottom:44px; }
  section.equipo h2 { font-size:1.25rem; margin:0 0 16px; display:flex; align-items:center; gap:10px; }
  section.equipo h2::after { content:""; flex:1; height:2px; background:var(--line); border-radius:2px; margin-left:6px; }
  section.equipo h2 .ico { width:38px; height:38px; display:grid; place-items:center; font-size:1.3rem; background:var(--panel); border:1px solid var(--line); border-radius:10px; }
  section.equipo h2 .cnt { font-size:.75rem; font-weight:600; background:var(--line); border-radius:999px; padding:2px 9px; color:var(--muted); }
  td .ico { margin-right:6px; }
  .grid { display:grid; grid-template-columns:repeat(auto-fill,minmax(290px,1fr)); gap:12px; }
  .card { background:var(--panel); border:1px solid var(--line); border-radius:12px; padding:14px; display:flex; flex-direction:column; gap:8px; border-top:5px solid transparent; }
  .card.estado-prog { border-top-color:var(--prog); background:var(--panel); color:var(--fg); }
  .card.estado-bloq { border-top-color:var(--bloq); background:var(--panel); color:var(--fg); }
  .card.estado-fin  { border-top-color:var(--fin);  background:var(--panel); color:var(--fg); }
  .card.estado-pro  { border-top-color:var(--pro);  background:var(--panel); color:var(--fg); }
  .card .top { display:flex; justify-content:space-between; align-items:center; gap:8px; }
  .id { font-size:.78rem; color:var(--muted); font-weight:600; }
  .badge { font-size:.68rem; font-weight:700; letter-spacing:.05em; padding:3px 9px; border-radius:999px; white-space:nowrap; }
  .card h3 { margin:0; font-size:1rem; line-height:1.3; }
  .desc { margin:0; font-size:.86rem; color:var(--muted); line-height:1.45; flex:1; }
  .pie { display:flex; justify-content:space-between; align-items:center; gap:8px; font-size:.78rem; color:var(--muted); flex-wrap:wrap; }
  .tipo { border:1px solid var(--line); border-radius:6px; padding:1px 7px; }
  .card a { font-size:.82rem; color:inherit; font-weight:600; text-decoration:none; border-bottom:1px dashed var(--muted); align-self:flex-start; }
  .card a:hover { border-bottom-style:solid; }
  .vacio { text-align:center; color:var(--muted); padding:40px 0; }

  /* Tarjeta clicable */
  .card[data-id] { cursor:pointer; transition:transform .12s, box-shadow .12s; }
  .card[data-id]:hover, .card[data-id]:focus-visible { transform:translateY(-2px); box-shadow:0 6px 16px rgba(0,0,0,.12); outline:none; }

  /* Botones */
  .btn { border:1px solid var(--line); background:var(--panel); color:var(--fg); border-radius:8px; padding:7px 14px; font-size:.85rem; cursor:pointer; font-family:inherit; }
  .btn:hover { border-color:var(--muted); }
  .btn.primario { background:var(--fg); color:var(--bg); border-color:var(--fg); }
  .acciones { display:flex; gap:12px; align-items:center; flex-wrap:wrap; }

  /* Ventanas flotantes */
  dialog { border:0; border-radius:14px; padding:0; background:var(--panel); color:var(--fg); width:min(640px, calc(100vw - 24px)); max-height:85vh; box-shadow:0 20px 60px rgba(0,0,0,.35); }
  dialog[open] { display:flex; flex-direction:column; }
  dialog::backdrop { background:rgba(10,12,20,.55); }
  .dhead { display:flex; justify-content:space-between; align-items:center; gap:10px; padding:14px 18px; border-bottom:1px solid var(--line); }
  .dhead h2 { margin:0; font-size:1.05rem; line-height:1.3; }
  .cerrar { border:0; background:none; color:var(--muted); font-size:1.5rem; line-height:1; cursor:pointer; padding:0 4px; }
  .dbody { padding:16px 18px; overflow-y:auto; flex:1; }
  .dfoot { display:flex; justify-content:flex-end; gap:8px; padding:12px 18px; border-top:1px solid var(--line); flex-wrap:wrap; }
  .detalle-meta { display:grid; grid-template-columns:repeat(auto-fit,minmax(140px,1fr)); gap:10px; margin-bottom:14px; }
  .detalle-meta div span { display:block; font-size:.7rem; color:var(--muted); letter-spacing:.05em; font-weight:600; }
  .detalle-desc { white-space:pre-wrap; overflow-wrap:anywhere; line-height:1.55; font-size:.92rem; margin:0 0 14px; }
  .campo { display:flex; flex-direction:column; gap:4px; margin-bottom:12px; font-size:.78rem; color:var(--muted); font-weight:600; letter-spacing:.05em; }
  .campo select { font:inherit; font-size:.9rem; font-weight:400; letter-spacing:0; padding:8px; border-radius:8px; border:1px solid var(--line); background:var(--bg); color:var(--fg); }
  .auditoria { margin:16px 0 0; padding-top:10px; border-top:1px solid var(--line); font-size:.72rem; color:var(--muted); line-height:1.6; }
  #x-cuenta { font-size:.88rem; color:var(--muted); margin:4px 0 0; }

  /* Vista de impresión (PDF) */
  #print-area { display:none; }
  @page { size:A4 landscape; margin:12mm; }
  @media print {
    body { background:#fff !important; color:#000 !important; }
    .wrap, dialog { display:none !important; }
    #print-area { display:block; color:#000; font-family:system-ui,sans-serif; -webkit-print-color-adjust:exact; print-color-adjust:exact; }
    #print-area h1 { font-size:16pt; margin:0 0 2px; }
    #print-area .sub { font-size:9pt; color:#444; margin-bottom:10px; }
    #print-area table { width:100%; min-width:0; border-collapse:collapse; font-size:8.5pt; }
    #print-area th, #print-area td { border:1px solid #bbb; padding:4px 6px; text-align:left; vertical-align:top; }
    #print-area th { background:#eee; }
    #print-area tr { break-inside:avoid; }
    #print-area .p-prog { background:#cfe3ff; } #print-area .p-bloq { background:#ffd3d3; }
    #print-area .p-fin { background:#d4f1dc; } #print-area .p-pro { background:#fff0c2; }
  }
</style>
</head>
<body>
<div class="wrap">
  <header>
    <h1>Tareas de IA</h1>
    <div class="acciones">
      <div class="meta" id="meta"></div>
      <button class="btn primario" id="btn-exportar" type="button">Exportar</button>
    </div>
  </header>

  <div class="totales" id="totales"></div>
  <div class="matriz"><table id="matriz"></table></div>

  <div class="filtros">
    <div class="grupo" id="f-equipo"><span>EQUIPO</span></div>
    <div class="grupo" id="f-estado"><span>ESTADO</span></div>
    <button id="limpiar" type="button">Limpiar filtros</button>
  </div>

  <div id="secciones">Cargando…</div>
</div>

<dialog id="detalle" aria-labelledby="d-titulo">
  <div class="dhead"><h2 id="d-titulo"></h2><button class="cerrar" type="button" data-cerrar aria-label="Cerrar">×</button></div>
  <div class="dbody" id="d-cuerpo"></div>
</dialog>

<dialog id="exportar" aria-labelledby="x-titulo">
  <div class="dhead"><h2 id="x-titulo">Exportar tareas</h2><button class="cerrar" type="button" data-cerrar aria-label="Cerrar">×</button></div>
  <div class="dbody">
    <label class="campo">EQUIPO<select id="x-equipo"></select></label>
    <label class="campo">AUTOR<select id="x-autor"></select></label>
    <label class="campo">ESTADO<select id="x-estado"></select></label>
    <p id="x-cuenta"></p>
  </div>
  <div class="dfoot">
    <button class="btn" type="button" data-cerrar>Cancelar</button>
    <button class="btn primario" type="button" id="x-csv">Excel (CSV)</button>
    <button class="btn primario" type="button" id="x-pdf">PDF</button>
  </div>
</dialog>

<div id="print-area"></div>

<script>
const EQUIPOS = ['iOS', 'Android', 'Backend', 'QA'];
const ESTADOS = [
  { nombre: 'EN PROGRESO', clase: 'estado-prog' },
  { nombre: 'BLOQUEADA',   clase: 'estado-bloq' },
  { nombre: 'FINALIZADA',  clase: 'estado-fin' },
  { nombre: 'PROGRAMADA',  clase: 'estado-pro' }
];
const ICONOS = { 'iOS': '🍎', 'Android': '🤖', 'Backend': '⚙️', 'QA': '🧪' };
const claseDe = e => (ESTADOS.find(s => s.nombre === e) || {}).clase || '';
const esc = s => String(s ?? '').replace(/[&<>"']/g, c => ({'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;',"'":'&#39;'}[c]));
const urlSegura = u => /^https?:\/\//i.test(u) ? u : '';
const etiquetaLink = u => /\/wiki\//i.test(u) ? 'Abrir en Confluence' : /\/browse\//i.test(u) ? 'Abrir en Jira' : 'Abrir enlace';

let tareas = [];
const filtro = { equipo: null, estado: null };

function chip(texto, clase, grupo, valor) {
  const b = document.createElement('button');
  b.type = 'button';
  b.className = 'chip ' + (clase || '');
  b.textContent = texto;
  b.dataset.grupo = grupo;
  b.dataset.valor = valor ?? '';
  b.setAttribute('aria-pressed', 'false');
  return b;
}

function armarFiltros() {
  const fe = document.getElementById('f-equipo');
  const fs = document.getElementById('f-estado');
  fe.append(chip('Todos', '', 'equipo', ''));
  EQUIPOS.forEach(e => fe.append(chip(ICONOS[e] + ' ' + e, '', 'equipo', e)));
  fs.append(chip('Todos', '', 'estado', ''));
  ESTADOS.forEach(s => fs.append(chip(s.nombre, s.clase, 'estado', s.nombre)));
  document.querySelectorAll('.chip').forEach(b => b.addEventListener('click', () => {
    filtro[b.dataset.grupo] = b.dataset.valor || null;
    render();
  }));
  document.getElementById('limpiar').addEventListener('click', () => {
    filtro.equipo = filtro.estado = null; render();
  });
}

function pintarChips() {
  document.querySelectorAll('.chip').forEach(b => {
    const activo = (filtro[b.dataset.grupo] || '') === b.dataset.valor;
    b.setAttribute('aria-pressed', String(activo));
  });
}

function resumen() {
  const cont = (eq, es) => tareas.filter(t => (!eq || t.equipo === eq) && (!es || t.estado === es)).length;
  document.getElementById('totales').innerHTML =
    `<div class="total todas"><div class="n">${tareas.length}</div><div class="l">TOTAL</div></div>` +
    ESTADOS.map(s => `<div class="total ${s.clase}"><div class="n">${cont(null, s.nombre)}</div><div class="l">${s.nombre}</div></div>`).join('');

  document.getElementById('matriz').innerHTML =
    '<thead><tr><th>EQUIPO</th>' + ESTADOS.map(s => `<th>${s.nombre}</th>`).join('') + '<th>TOTAL</th></tr></thead><tbody>' +
    EQUIPOS.map(e => `<tr><td><span class="ico">${ICONOS[e]}</span>${esc(e)}</td>` +
      ESTADOS.map(s => { const n = cont(e, s.nombre); return `<td class="${n ? '' : 'zero'}">${n ? `<span class="badge ${s.clase}">${n}</span>` : '0'}</td>`; }).join('') +
      `<td><b>${cont(e)}</b></td></tr>`).join('') + '</tbody>';
}

function tarjeta(t) {
  const link = urlSegura(t.link);
  return `<article class="card ${claseDe(t.estado)}" data-id="${esc(t.id)}" tabindex="0" role="button" aria-label="Ver detalle: ${esc(t.titulo)}">
    <div class="top"><span class="id">#${esc(t.id)}</span><span class="badge ${claseDe(t.estado)}">${esc(t.estado)}</span></div>
    <h3>${esc(t.titulo)}</h3>
    <p class="desc">${esc(t.descripcion)}</p>
    ${link ? `<a href="${esc(link)}" target="_blank" rel="noopener noreferrer">${etiquetaLink(link)} ↗</a>` : ''}
    <div class="pie"><span>${esc(t.autor)}</span><span class="tipo">${esc(t.tipo)}</span></div>
  </article>`;
}

function render() {
  pintarChips();
  const visibles = tareas.filter(t =>
    (!filtro.equipo || t.equipo === filtro.equipo) && (!filtro.estado || t.estado === filtro.estado));
  const orden = Object.fromEntries(ESTADOS.map((s, i) => [s.nombre, i]));
  const html = EQUIPOS.map(e => {
    const ts = visibles.filter(t => t.equipo === e).sort((a, b) => orden[a.estado] - orden[b.estado] || a.id - b.id);
    if (!ts.length) return '';
    return `<section class="equipo"><h2><span class="ico" aria-hidden="true">${ICONOS[e]}</span>${esc(e)} <span class="cnt">${ts.length}</span></h2><div class="grid">${ts.map(tarjeta).join('')}</div></section>`;
  }).join('');
  document.getElementById('secciones').innerHTML = html || '<div class="vacio">No hay tareas con esos filtros.</div>';
}

/* ---------- Detalle (ventana flotante) ---------- */
const $ = id => document.getElementById(id);
function auditoria(t) {
  const linea = (txt, quien, cuando) => quien || cuando
    ? `<div>${txt}${quien ? ' por <b>@' + esc(quien) + '</b>' : ''}${cuando ? ' el ' + esc(cuando) : ''}</div>` : '';
  const html = linea('Creado', t.creadoPor, t.creadoEn) + linea('Editado', t.editadoPor, t.editadoEn);
  return html ? `<div class="auditoria">${html}</div>` : '';
}
function abrirDetalle(t) {
  const link = urlSegura(t.link);
  $('d-titulo').textContent = t.titulo;
  $('d-cuerpo').innerHTML = `
    <div style="margin-bottom:12px"><span class="id">#${esc(t.id)}</span> <span class="badge ${claseDe(t.estado)}">${esc(t.estado)}</span></div>
    <div class="detalle-meta">
      <div><span>AUTOR</span>${esc(t.autor)}</div>
      <div><span>EQUIPO</span>${esc(t.equipo)}</div>
      <div><span>TIPO</span>${esc(t.tipo)}</div>
    </div>
    <p class="detalle-desc">${esc(t.descripcion)}</p>
    ${link ? `<a class="btn" href="${esc(link)}" target="_blank" rel="noopener noreferrer" style="text-decoration:none;display:inline-block">${etiquetaLink(link)} ↗</a>` : ''}
    ${auditoria(t)}`;
  $('detalle').showModal();
}

document.addEventListener('click', e => {
  if (e.target.closest('[data-cerrar]')) { e.target.closest('dialog').close(); return; }
  if (e.target instanceof HTMLDialogElement) { e.target.close(); return; } // clic en el fondo
  const card = e.target.closest('.card[data-id]');
  if (card && !e.target.closest('a')) abrirDetalle(tareas.find(t => String(t.id) === card.dataset.id));
});
document.addEventListener('keydown', e => {
  if ((e.key === 'Enter' || e.key === ' ') && e.target.matches('.card[data-id]')) {
    e.preventDefault();
    abrirDetalle(tareas.find(t => String(t.id) === e.target.dataset.id));
  }
});

/* ---------- Exportar (PDF / Excel) ---------- */
function opciones(sel, valores, actual) {
  sel.innerHTML = '<option value="">Todos</option>' +
    valores.map(v => `<option value="${esc(v)}"${v === actual ? ' selected' : ''}>${esc(v)}</option>`).join('');
}
const filasExport = () => tareas.filter(t =>
  (!$('x-equipo').value || t.equipo === $('x-equipo').value) &&
  (!$('x-autor').value || t.autor === $('x-autor').value) &&
  (!$('x-estado').value || t.estado === $('x-estado').value));
const cuenta = () => { const n = filasExport().length; $('x-cuenta').textContent = n + (n === 1 ? ' tarea a exportar' : ' tareas a exportar'); };

$('btn-exportar').addEventListener('click', () => {
  opciones($('x-equipo'), EQUIPOS, filtro.equipo);
  opciones($('x-autor'), [...new Set(tareas.map(t => t.autor))].sort((a, b) => a.localeCompare(b)), '');
  opciones($('x-estado'), ESTADOS.map(s => s.nombre), filtro.estado);
  cuenta();
  $('exportar').showModal();
});
['x-equipo', 'x-autor', 'x-estado'].forEach(id => $(id).addEventListener('change', cuenta));

const hoy = () => new Date().toLocaleDateString('sv'); // YYYY-MM-DD local

$('x-csv').addEventListener('click', () => {
  // Evita inyección de fórmulas en Excel (=, +, -, @) y escapa comillas
  const celda = v => { let s = String(v ?? '').replace(/\r?\n/g, ' '); if (/^[=+\-@]/.test(s)) s = "'" + s; return '"' + s.replace(/"/g, '""') + '"'; };
  const cols = [['ID', 'id'], ['Autor', 'autor'], ['Título', 'titulo'], ['Tipo', 'tipo'], ['Descripción', 'descripcion'], ['Equipo', 'equipo'], ['Estado', 'estado'], ['Link', 'link']];
  const lineas = ['sep=;', cols.map(c => celda(c[0])).join(';'),
    ...filasExport().map(t => cols.map(c => celda(t[c[1]])).join(';'))];
  const blob = new Blob(['﻿' + lineas.join('\r\n')], { type: 'text/csv;charset=utf-8' });
  const a = document.createElement('a');
  a.href = URL.createObjectURL(blob);
  a.download = `tareas-ia-${hoy()}.csv`;
  document.body.append(a); a.click(); a.remove();
  setTimeout(() => URL.revokeObjectURL(a.href), 1000);
  $('exportar').close();
});

$('x-pdf').addEventListener('click', () => {
  const claseP = { 'EN PROGRESO': 'p-prog', 'BLOQUEADA': 'p-bloq', 'FINALIZADA': 'p-fin', 'PROGRAMADA': 'p-pro' };
  const f = [['Equipo', $('x-equipo').value], ['Autor', $('x-autor').value], ['Estado', $('x-estado').value]]
    .filter(x => x[1]).map(x => `${x[0]}: ${x[1]}`).join(' · ') || 'Sin filtros';
  const filas = filasExport();
  $('print-area').innerHTML = `
    <h1>Tareas de IA</h1>
    <div class="sub">${esc(f)} · ${filas.length} tareas · ${esc(hoy())}</div>
    <table><thead><tr><th>ID</th><th>Equipo</th><th>Estado</th><th>Título</th><th>Autor</th><th>Tipo</th><th>Descripción</th><th>Link</th></tr></thead><tbody>` +
    filas.map(t => `<tr><td>${esc(t.id)}</td><td>${esc(t.equipo)}</td><td class="${claseP[t.estado] || ''}">${esc(t.estado)}</td><td>${esc(t.titulo)}</td><td>${esc(t.autor)}</td><td>${esc(t.tipo)}</td><td>${esc(t.descripcion)}</td><td>${esc(urlSegura(t.link))}</td></tr>`).join('') +
    '</tbody></table>';
  $('exportar').close();
  setTimeout(() => window.print(), 50); // en el diálogo de impresión: "Guardar como PDF"
});

fetch('data.json', { cache: 'no-store' })
  .then(r => r.json())
  .then(d => {
    tareas = d.tareas;
    document.getElementById('meta').textContent = 'Actualizado: ' + d.actualizado;
    armarFiltros(); resumen(); render();
  })
  .catch(() => { document.getElementById('secciones').textContent = 'No se pudo cargar data.json'; });
</script>
</body>
</html>
===== FIN index.html =====

===== INICIO data.json =====
{
  "actualizado": "2026-11-07",
  "tareas": [
    {"id": 1, "autor": "Lucía Fernández", "titulo": "Integrar asistente IA en pantalla de soporte", "tipo": "Feature", "descripcion": "Chat con el asistente de IA dentro de la app iOS, con streaming de respuestas y manejo de errores de red.", "equipo": "iOS", "link": "https://empresa.atlassian.net/browse/IA-101", "estado": "EN PROGRESO", "creadoPor": "lucia-fernandez", "creadoEn": "2026-10-11", "editadoPor": "cumpatomas", "editadoEn": "2026-11-05"},
    {"id": 2, "autor": "Martín Gómez", "titulo": "Migrar SDK de analítica de prompts", "tipo": "Deuda técnica", "descripcion": "Actualizar el SDK que registra uso de prompts para cumplir con la nueva política de privacidad.", "equipo": "iOS", "link": "https://empresa.atlassian.net/browse/IA-102", "estado": "PROGRAMADA", "creadoPor": "martin-gomez", "creadoEn": "2026-10-12"},
    {"id": 3, "autor": "Sofía Ramírez", "titulo": "Resúmenes automáticos en notificaciones", "tipo": "Feature", "descripcion": "Generar resúmenes cortos con IA para las notificaciones push largas.", "equipo": "iOS", "link": "https://empresa.atlassian.net/wiki/spaces/IA/pages/2001", "estado": "FINALIZADA", "creadoPor": "sofia-ramirez", "creadoEn": "2026-10-13"},
    {"id": 4, "autor": "Diego Torres", "titulo": "Modelo on-device para sugerencias", "tipo": "Investigación", "descripcion": "Evaluar modelo liviano on-device para sugerencias de texto sin enviar datos al servidor.", "equipo": "iOS", "link": "https://empresa.atlassian.net/wiki/spaces/IA/pages/2002", "estado": "BLOQUEADA", "creadoPor": "diego-torres", "creadoEn": "2026-10-14", "editadoPor": "cumpatomas", "editadoEn": "2026-11-05"},
    {"id": 5, "autor": "Valentina Cruz", "titulo": "Chat con asistente IA en Android", "tipo": "Feature", "descripcion": "Pantalla de chat en Compose con streaming de tokens y reintentos automáticos.", "equipo": "Android", "link": "https://empresa.atlassian.net/browse/IA-201", "estado": "EN PROGRESO", "creadoPor": "valentina-cruz", "creadoEn": "2026-10-15"},
    {"id": 6, "autor": "Joaquín Méndez", "titulo": "Reducir consumo de batería del módulo IA", "tipo": "Performance", "descripcion": "Perfilar y optimizar las llamadas en segundo plano del módulo de recomendaciones.", "equipo": "Android", "link": "https://empresa.atlassian.net/browse/IA-202", "estado": "EN PROGRESO", "creadoPor": "joaquin-mendez", "creadoEn": "2026-10-16"},
    {"id": 7, "autor": "Camila Ortiz", "titulo": "Accesibilidad en respuestas del asistente", "tipo": "Mejora", "descripcion": "Soporte de TalkBack y contraste adecuado en las tarjetas de respuesta generadas.", "equipo": "Android", "link": "https://empresa.atlassian.net/wiki/spaces/IA/pages/3001", "estado": "PROGRAMADA", "creadoPor": "camila-ortiz", "creadoEn": "2026-10-17"},
    {"id": 8, "autor": "Nicolás Peña", "titulo": "Fallback cuando el modelo no responde", "tipo": "Bug", "descripcion": "La app se queda cargando indefinidamente si el backend de IA excede el timeout.", "equipo": "Android", "link": "https://empresa.atlassian.net/browse/IA-203", "estado": "FINALIZADA", "creadoPor": "nicolas-pena", "creadoEn": "2026-10-18"},
    {"id": 9, "autor": "Ana Beltrán", "titulo": "Gateway unificado de modelos LLM", "tipo": "Arquitectura", "descripcion": "Servicio que enruta peticiones a distintos proveedores de LLM con límites y métricas de costo.", "equipo": "Backend", "link": "https://empresa.atlassian.net/browse/IA-301", "estado": "EN PROGRESO", "creadoPor": "ana-beltran", "creadoEn": "2026-10-19", "editadoPor": "cumpatomas", "editadoEn": "2026-11-05"},
    {"id": 10, "autor": "Ricardo Sosa", "titulo": "Caché semántica de respuestas", "tipo": "Performance", "descripcion": "Reutilizar respuestas de prompts similares usando embeddings para bajar costos y latencia.", "equipo": "Backend", "link": "https://empresa.atlassian.net/wiki/spaces/IA/pages/4001", "estado": "BLOQUEADA", "creadoPor": "ricardo-sosa", "creadoEn": "2026-10-20"},
    {"id": 11, "autor": "Paula Medina", "titulo": "Filtro de datos personales en prompts", "tipo": "Seguridad", "descripcion": "Detectar y enmascarar PII antes de enviar los prompts a proveedores externos.", "equipo": "Backend", "link": "https://empresa.atlassian.net/browse/IA-302", "estado": "FINALIZADA", "creadoPor": "paula-medina", "creadoEn": "2026-10-21"},
    {"id": 12, "autor": "Esteban Rojas", "titulo": "Pipeline de indexado de documentos", "tipo": "Feature", "descripcion": "Ingesta de Confluence y Jira a la base vectorial para el asistente interno.", "equipo": "Backend", "link": "https://empresa.atlassian.net/browse/IA-303", "estado": "PROGRAMADA", "creadoPor": "esteban-rojas", "creadoEn": "2026-10-22"},
    {"id": 13, "autor": "Florencia Vega", "titulo": "Suite de evaluación de respuestas del asistente", "tipo": "Testing", "descripcion": "Conjunto de casos dorados para medir calidad, alucinaciones y tono de las respuestas.", "equipo": "QA", "link": "https://empresa.atlassian.net/wiki/spaces/IA/pages/5001", "estado": "EN PROGRESO", "creadoPor": "florencia-vega", "creadoEn": "2026-10-23"},
    {"id": 14, "autor": "Gabriel Ibáñez", "titulo": "Pruebas de regresión del chat en ambas apps", "tipo": "Testing", "descripcion": "Automatizar los flujos principales del chat en iOS y Android.", "equipo": "QA", "link": "https://empresa.atlassian.net/browse/IA-401", "estado": "PROGRAMADA", "creadoPor": "gabriel-ibanez", "creadoEn": "2026-10-24"},
    {"id": 15, "autor": "Marina Lagos", "titulo": "Pruebas de carga del gateway LLM", "tipo": "Performance", "descripcion": "Esperando entorno de staging con cuotas de proveedor para ejecutar las pruebas.", "equipo": "QA", "link": "https://empresa.atlassian.net/browse/IA-402", "estado": "BLOQUEADA", "creadoPor": "marina-lagos", "creadoEn": "2026-10-25"}
  ]
}
===== FIN data.json =====

===== INICIO .claude/commands/tareas-ia-crear.md =====
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
===== FIN .claude/commands/tareas-ia-crear.md =====

===== INICIO .claude/commands/tareas-ia-editar.md =====
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
===== FIN .claude/commands/tareas-ia-editar.md =====

===== INICIO .claude/skills/tareas-ia/SKILL.md =====
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
===== FIN .claude/skills/tareas-ia/SKILL.md =====

===== INICIO .claude/skills/tareas-ia/esquema.md =====
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
===== FIN .claude/skills/tareas-ia/esquema.md =====
===== INICIO .github/copilot-instructions.md =====
# Instrucciones para Copilot – Tareas IA

Este repo es un dashboard estático (`index.html` + `data.json`) publicado con GitHub Pages. Las tareas viven en `data.json`.

## Cuando pidan algo sobre "tareas IA"

Si el pedido es crear, agregar, cargar, editar, actualizar, cambiar de estado, listar o consultar tareas de IA:

- **Crear** (a mano, desde un link de Jira/Confluence o desde un archivo): seguí el procedimiento de [tareas-ia-crear](../.claude/commands/tareas-ia-crear.md).
- **Editar**: seguí el procedimiento de [tareas-ia-editar](../.claude/commands/tareas-ia-editar.md).
- **Listar / consultar**: leé `data.json` y respondé, sin modificar nada.

Esos archivos son la fuente única del procedimiento. Adaptaciones para Copilot:

- `$ARGUMENTS` es el texto que escribió el usuario en su pedido.
- Donde dicen "preguntá con AskUserQuestion", hacé la pregunta en el chat.
- Para leer `.xlsx`, usá la terminal (Python con `openpyxl`) si está disponible; si no, pedí exportar a CSV.
- Para Jira/Confluence, usá una herramienta de Atlassian si está conectada; si no, pedí al usuario que pegue el contenido.

Esquema, reglas y normalizaciones: [esquema.md](../.claude/skills/tareas-ia/esquema.md). Leelo siempre antes de tocar `data.json`.

## Reglas que no se negocian

- Nunca inventes datos: si un campo obligatorio no sale de forma determinista, preguntalo.
- `creadoPor`/`editadoPor` son el usuario de GitHub de quien hace el cambio, no el `autor`.
- Mostrá el resumen de cambios y pedí confirmación antes de hacer commit/push.
- Nunca borres tareas ni cambies ids.
- No agregues dependencias externas ni CDNs al HTML.
===== FIN .github/copilot-instructions.md =====

===== INICIO AGENTS.md =====
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
===== FIN AGENTS.md =====

===== INICIO .github/prompts/tareas-ia-crear.prompt.md =====
---
mode: agent
description: Crear tareas de IA en data.json (manual, desde un link de Jira/Confluence o desde un archivo)
---

Creá una o más tareas de IA en `data.json`.

Seguí el procedimiento de [tareas-ia-crear](../../.claude/commands/tareas-ia-crear.md) y las reglas de [esquema.md](../../.claude/skills/tareas-ia/esquema.md), con estas adaptaciones:

- `$ARGUMENTS` es lo que el usuario escribió junto a este prompt (modo `manual`, una URL de Jira/Confluence o la ruta de un archivo). Si no escribió nada, preguntale en el chat cuál de los 3 modos quiere: 1) manual, 2) link de Jira/Confluence, 3) archivo.
- Las preguntas al usuario se hacen en el chat.
- Pedí confirmación con el resumen de cambios antes de commit/push.
===== FIN .github/prompts/tareas-ia-crear.prompt.md =====

===== INICIO .github/prompts/tareas-ia-editar.prompt.md =====
---
mode: agent
description: Editar una tarea de IA existente en data.json (por id y campos)
---

Editá una tarea de IA existente en `data.json`.

Seguí el procedimiento de [tareas-ia-editar](../../.claude/commands/tareas-ia-editar.md) y las reglas de [esquema.md](../../.claude/skills/tareas-ia/esquema.md), con estas adaptaciones:

- `$ARGUMENTS` es lo que el usuario escribió junto a este prompt (id y/o campos). Lo que falte, pedilo en el chat.
- Registrá `editadoPor` (usuario de GitHub) y `editadoEn` (fecha de hoy). No toques `creadoPor` ni `creadoEn`.
- Pedí confirmación con el antes/después antes de commit/push.
===== FIN .github/prompts/tareas-ia-editar.prompt.md =====
````

## Después de que el agente termine

1. *Settings → Pages → Build and deployment → Deploy from a branch → `main` / `(root)`*.
2. En la misma pantalla, *Visibility → Private* (si no aparece, un owner debe habilitar "Publish private GitHub Pages sites" en la configuración de la organización).
3. Reemplazar los datos de ejemplo de `data.json` por los reales.

## Probar en local

```
python -m http.server 8000
```

y abrir `http://localhost:8000` (con doble clic `fetch` no funciona).
