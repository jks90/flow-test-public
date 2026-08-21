---
name: flows
description: Crear, ejecutar y verificar flows de flow-test (.flow.json) contra cualquier API REST — en vivo por MCP sobre el canvas del usuario, o como ficheros ejecutados con el CLI. Úsala cuando el usuario pida crear un flow, probar endpoints recién desarrollados o montar una batería de verificación.
user-invocable: true
---
# Flows skill

Tras terminar un desarrollo (o cuando el usuario lo pida), implementa los flows de flow-test
que ejercitan los endpoints afectados y ejecútalos para verificar la API de punta a punta.

> Copia esta carpeta (con `references/`) a `.claude/skills/flows/` de tu proyecto para
> activarla en Claude Code.

## Role

Actúa como un ingeniero de QA automation que domina la API objetivo y la herramienta
flow-test (web + CLI + MCP).

## Task

Crear o actualizar flows `.flow.json` que cubran los endpoints afectados, ejecutarlos y
reportar el resultado real (nunca asumido).

## Context

### Input
- El desarrollo recién terminado: endpoints nuevos/cambiados (git diff, spec o conversación).
- Los flows existentes del proyecto (habitualmente en `flows/`; créala si no existe).

### References
- [`references/flow-schema.md`](./references/flow-schema.md) — **el formato `.flow.json` al
  detalle**: nodos HTTP y SQL, conexiones, variables, extracciones, convenciones de curl,
  UUIDs, ejecución y diagnóstico. Síguelo exactamente al escribir flows.

### Tools
- **MCP `flow-test`** (si está conectado): construye y ejecuta en el canvas del usuario en
  directo. Endpoint típico: `http://localhost:9998/mcp` (contenedor) o `:3001` (local).
  Mapa de las 33 tools (4.33):
  - Observar: `bridge_status`, `flow_state` (pestañas con `filePath`/`dirty`; nodos con
    `position`/`order`/`pinned`; notas con `renderMode`/contenido/`scripts`; `drawings`),
    `console_read`, `runs_read`.
  - Pestañas: `flow_create`, `tab_select`, `tab_close`, `flow_overwrite` (copia sin enlazar).
  - Nodos: `node_add_request`, `node_add_sql`, `node_add_info` (text | mermaid | image, con
    `scripts`), `node_add_web` — todos aceptan `order`, `cell {col,row}`, `pinned` —,
    `node_update` (cualquier tipo y campo editable, incl. `cell`, `pinned`, `disabled`), `node_delete`,
    `nodes_connect`, `connection_update` (cambiar el behavior), `connection_delete`,
    `variables_set` (entorno de la pestaña), `global_variables_set` (globales), `tab_rename`,
    `sql_connections_list` (perfiles SQL para `connectionProfileId`).
  - Ejecutar: `flow_run`, `node_run`, `flow_reset`.
  - Lienzo: `node_focus` (señalar un nodo al usuario), `canvas_layout`
    (`auto|row|column|grid|separate|collapse_all|expand_all|pin_all|unpin_all`; `grid` usa la celda `cell: {col,row}` de cada
    nodo — 1,1 arriba a la izquierda — que aceptan `node_add_*`/`node_update`; `separate` aparta
    solapes), `view_settings` (tamaño de las cards, modo compacto, separación mínima — la vista del
    usuario), `whiteboard_update` (anotar la pizarra).
  - **Proyecto `flows/`** (la carpeta del servidor = el panel Proyecto de la web):
    `flow_files_list` (con metadatos), `flow_open` (abre **enlazado** al fichero),
    `flow_save` (= Ctrl+S: enlaza la pestaña y la deja limpia; subcarpetas), `flow_file_delete`,
    `flow_file_read`.
- **CLI**: `docker exec flow node cli/run-flow.js …` (contenedor) o
  `node cli/run-flow.js …` (instalación local).

### Constraints (Step 0 — resuelve antes de nada; pregunta lo que falte, de una vez)
1. **¿Dónde corre la API objetivo?** (host y puerto → variable `{{apiBase}}`).
2. **¿Autenticación?** (ninguna / bearer estático / login que devuelve token / basic).
3. **¿Dónde corre flow-test?** (contenedor `:9998` por defecto / local).
4. Los flows **ejecutan de verdad** (HTTP con efectos y SQL con INSERT/UPDATE): apunta a
   entornos de prueba; ante endpoints destructivos, pregunta antes de incluirlos.
5. Un flow por dominio/caso: `flows/{dominio}-{caso}.flow.json`. Todas las URLs con
   `{{apiBase}}`; datos de prueba en `envVariables`, nunca hardcodeados en el curl.

## Steps

### Step 1: Scope
- Identifica los endpoints afectados y el camino realista que los encadena
  (crear → extraer id → consultar/limpiar).
- Si ya existe un flow que cubre el área (`flow_files_list` por MCP, o mira `flows/`),
  **ábrelo con `flow_open`** (la pestaña queda enlazada al fichero y conserva los ids) y
  actualízalo en vez de duplicar. `flow_state` te dice si la pestaña tiene cambios sin guardar
  (`dirty`) y a qué fichero está enlazada (`filePath`).

### Step 2: Implementar — elige el modo

**Modo A — MCP en vivo** (preferido si el servidor MCP `flow-test` está conectado; el
usuario ve el canvas):
1. `bridge_status`: si no hay pestaña web conectada, pide al usuario abrir la web y reintenta.
2. `flow_open` (flow existente) o `flow_create` (nuevo) → guarda el `tabId` y úsalo en todas
   las llamadas.
3. `node_add_request` / `node_add_sql` por paso, `nodes_connect` (behavior `next` ordena el
   grafo; `connection_update` para cambiarlo después). Guarda los `nodeId` que devuelven.
   **Disposición**: da a cada nodo una **celda `cell: {col, row}`** al crearlo — filas = capas
   (1 notas, 2 cadena HTTP, 3 SQL de verificación, 4 web/capturas), columnas = pasos — y al final
   `canvas_layout grid` (o `auto`, que respeta las celdas y ordena por el grafo lo que no tenga).
   Alternativa rápida: `order` = nº de paso + `canvas_layout row`. Si ves solapes, `canvas_layout
   separate`. Nunca calcules coordenadas a mano salvo que copies un flow existente.
   Pasos opcionales o destructivos que no deban correr por defecto: déjalos con `disabled: true`
   (se saltan en Run Flow/CLI; el usuario los activa desde el menú de la caja).
4. `variables_set` para la base (`apiBase`, credenciales de prueba); extracciones en cada
   nodo para encadenar (`jsonPath` en HTTP; `column`+`rowIndex` en SQL). Datos únicos por
   ejecución (emails, ids): un `node_add_info` con `scripts` (`return 'qa+' + Date.now() + '@x.com'`).
5. Documenta dentro del canvas si aporta: `node_add_info` con `renderMode: "mermaid"` para el
   esquema del flujo, `whiteboard_update` para rodear/etiquetar grupos de cajas, y en el texto de
   las notas `[[otro-flow#Nodo]]` para enlazar flows relacionados del proyecto.
6. Al terminar, `flow_save` (sin `fileName` si la pestaña ya está enlazada; con subcarpeta
   `dominio/caso` si es nuevo) — equivale al Ctrl+S del usuario: el `.flow.json` queda en `flows/`
   y es ejecutable con el CLI. `node_focus` sobre el nodo clave para señalárselo al usuario.
7. Todo lo que hagas por MCP **se pinta en la pestaña del usuario al instante** (puente SSE): crea
   los nodos de uno en uno y en orden, nombra bien cada caja y comenta al usuario qué estás
   añadiendo — está mirando. `flow_state` te devuelve siempre el estado real (nodos, conexiones
   con id y behavior, variables de entorno/runtime/globales, vista). Si el flow es grande, sube la
   legibilidad con `view_settings` (`compactMode: 'always'` o `nodeScale: 0.75`) — es la vista
   del usuario, restáurala al acabar si la cambiaste.

**Modo B — Fichero + CLI** (sin MCP, o para CI): escribe el `.flow.json` siguiendo
[`references/flow-schema.md`](./references/flow-schema.md) — estructura exacta, convención
`\\\n\n` en los curl, UUIDs v4 frescos por nodo/conexión/extracción, toda `{{var}}`
declarada en `envVariables` o producida por una extracción.

### Step 3: Ejecutar y verificar
- Comprueba que la API objetivo responde antes de ejecutar (un curl a su health/status); si
  está caída, avisa en vez de ejecutar a ciegas.
- **MCP**: `flow_run` (grafo HTTP) o `node_run` sobre el nodo raíz de una cadena SQL.
  Espera el resultado y léelo: status por nodo, `runtimeVariables`, bodies. Si responde
  `finished:false`, lee el `reason` — no asumas que corrió.
- **CLI**: `… cli/run-flow.js --flow flows/x.flow.json --report-root /tmp/resumen`
  (+ `--var` para credenciales). Exit code `0` = verde; con `1`, lee
  `report.md` y `debug/*.debug.md` del report para diagnosticar.
- Diagnóstico: si el flow está mal, corrígelo (`node_update` o editar el fichero) y reejecuta;
  si la API está mal, **reporta el bug** con la evidencia del debug — no maquilles el flow
  para que pase.

## Output
- Flows nuevos/actualizados (en `flows/` vía `flow_save` — la pestaña del usuario queda enlazada y
  sin cambios pendientes —, o ficheros commiteables).
- Resumen de la ejecución: nodos passed/failed, variables extraídas, ruta del report.
- Si el proyecto versiona los flows y el run pasa: commit `test(flows): {descripción}`.

## Verification
- [ ] El flow es JSON válido y sigue el schema (version, nodes, connections, envVariables).
- [ ] Toda `{{variable}}` usada existe en `envVariables` o viene de una extracción previa.
- [ ] Verifica efectos, no solo status: tras un POST que crea algo, un GET (o nodo SQL)
      confirma que existe de verdad.
- [ ] La ejecución terminó en verde de verdad (exit 0 / `finished:true` con nodos OK) y el
      resultado reportado sale del run real, no de una suposición.
- [ ] Sin credenciales reales ni entornos de producción salvo petición explícita del usuario.
