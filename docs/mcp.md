# El MCP — una IA construye y ejecuta flows en tu canvas

Desde la **4.2.0** la imagen lleva un **servidor MCP embebido** (Streamable HTTP) en `/mcp`.
Un agente (Claude Code, Claude Desktop u otro cliente MCP) puede crear, editar, ejecutar y
leer flows **en la web en directo**: tú miras el canvas y ves aparecer los nodos, conectarse
y encenderse; el agente recibe de vuelta el estado, la consola y los resultados. Comunicación
total en los dos sentidos.

## Conectar Claude Code

```bash
# con el contenedor en el puerto 9998:
claude mcp add --transport http flow-test http://localhost:9998/mcp
```

O por proyecto, con un `.mcp.json` en la raíz (hay un ejemplo en este repo:
[`.mcp.json.example`](../.mcp.json.example)):

```json
{
  "mcpServers": {
    "flow-test": { "type": "http", "url": "http://localhost:9998/mcp" }
  }
}
```

## Conectar Claude Desktop

**Camino 1 — conector custom** (recomendado, sin ficheros): Ajustes → *Connectors* →
*Add custom connector* → URL: `http://localhost:9998/mcp`.

**Camino 2 — `claude_desktop_config.json`** (config clásica; el fichero solo admite
servidores stdio, así que se usa el puente `mcp-remote`, que requiere Node.js instalado).
Copia el contenido de [`mcp-config.stdio.example.json`](../mcp-config.stdio.example.json) en:

| SO | Ruta |
|----|------|
| macOS | `~/Library/Application Support/Claude/claude_desktop_config.json` |
| Windows | `%APPDATA%\Claude\claude_desktop_config.json` |

y reinicia Claude Desktop.

## Clientes MCP solo-stdio

Para cualquier otro cliente sin transporte HTTP, el mismo puente
[`mcp-remote`](https://www.npmjs.com/package/mcp-remote) convierte el endpoint a stdio —
ejemplo listo en [`mcp-config.stdio.example.json`](../mcp-config.stdio.example.json):

```json
{
  "mcpServers": {
    "flow-test": {
      "command": "npx",
      "args": ["-y", "mcp-remote", "http://localhost:9998/mcp"]
    }
  }
}
```

## Cómo funciona por dentro

1. La web (http://localhost:9998) se conecta **sola** al puente del servidor
   (`/mcp-bridge/events`, SSE) al abrirse.
2. Cada tool del agente se convierte en un comando que llega a la pestaña abierta y se
   ejecuta como una operación real del canvas.
3. El resultado (ids de nodos, run completo, variables extraídas…) vuelve al agente por el
   mismo puente.

- Si hay **varias pestañas** abiertas, controla la **última** conectada.
- **Sin pestaña abierta**, las tools de web devuelven un error claro; las de disco
  (`flow_save`, `flow_file_read`, `flow_files_list`) siguen funcionando.
- `bridge_status` te dice cuántas pestañas hay conectadas.

## Seguridad

| Modo | Cómo | Cuándo |
|------|------|--------|
| **Solo local** (por defecto) | Nada que configurar | Claude corre en la misma máquina que el contenedor — el caso normal |
| **Token** | `-e FLOW_MCP_TOKEN=<secreto>` y el cliente manda `Authorization: Bearer <secreto>` | Acceso remoto controlado |
| **LAN abierta** | `-e FLOW_MCP_ALLOW_LAN=true` | Solo en redes de total confianza |

Además: los snapshots de estado que ve la IA **no incluyen las passwords** de los nodos SQL,
y `flow_file_read` está limitado a la carpeta `flows/` (sin path traversal).

## Las 28 tools

### Observar

| Tool | Qué hace |
|------|----------|
| `flow_state` | Estado vivo: pestañas + detalle de una (nodos con status y resumen de respuesta, conexiones, variables) |
| `bridge_status` | Pestañas web conectadas |
| `console_read` | La consola de la web (lo mismo que ve el usuario abajo) |
| `runs_read` | Historial de ejecuciones con resultados por nodo |

### Construir

| Tool | Qué hace |
|------|----------|
| `flow_create` | Nueva pestaña vacía (devuelve `tabId`) |
| `tab_select` 🆕 4.26 | Activa una pestaña (la que ve el usuario) |
| `flow_overwrite` | Carga un `.flow.json` completo (nodos, notas, capturas y dibujos de la pizarra) como **copia nueva sin enlazar a fichero** — con `tabId` **reemplaza** esa pestaña; sin él crea una nueva. Para abrir un fichero del proyecto usa `flow_open` |
| `node_add_request` | Nodo HTTP desde un `curl` (+ extracciones JSONPath) |
| `node_add_sql` | Nodo SQL (postgres/mysql/oracle; perfil o conexión inline; extracciones por columna) |
| `node_add_info` 🆕 4.24 | Nota: texto (`renderMode: "text"`), diagrama Mermaid (`"mermaid"`, `content` = código, admite `{{variables}}`) o captura (`"image"` + `imageSrc`). Desde 4.26 acepta `scripts` (`[{varName, code}]`, se ejecutan al Run Flow), `order` y `pinned` |
| `node_add_web` 🆕 4.26 | Nodo Web (URL / modo Live) |
| `node_update` | Actualiza campos de **cualquier** nodo: nombre, `position`, `collapsed`, `order`, `pinned`; HTTP `curl`/`extractions`; SQL `query`/conexión/`extractions`; nota `content`/`renderMode`/`imageSrc`/`scripts`; web `url`. Ignora campos de solo lectura (status, response…) |
| `node_delete` | Borra un nodo y sus conexiones |
| `nodes_connect` | Conecta origen → destino (`next` / `on_error` / `parallel` / `none`) |
| `connection_delete` | Borra una conexión |
| `variables_set` | Variables de entorno de la pestaña (para `{{var}}`) |
| `tab_close` 🆕 4.23 | Cierra una pestaña (`tabId` obligatorio; no cierra una pestaña que esté ejecutando) |

### Ejecutar

| Tool | Qué hace |
|------|----------|
| `flow_run` | Ejecuta el grafo de nodos HTTP (el botón «Run Flow») y **espera al resultado**: run completo + variables extraídas. Desde la 4.23 ejecuta antes los **scripts JS de las notas** y mete sus valores como variables. Si no ejecuta nada (pestaña ocupada, flow sin nodos HTTP) responde `finished:false` con el motivo |
| `node_run` | Ejecuta un nodo y su cadena descendente. Las cadenas que **arrancan en un nodo SQL** encadenan SQL y HTTP; las que arrancan en HTTP solo siguen nodos HTTP (misma semántica que la web) |
| `flow_reset` 🆕 4.26 | Limpia respuestas, resultados, estados y variables runtime de la pestaña (el «Reset») |

### Lienzo 🆕 4.26

| Tool | Qué hace |
|------|----------|
| `node_focus` | Centra el lienzo en un nodo (por `nodeId` o `nodeName`) y lo resalta ~2 s — para señalar algo al usuario |
| `canvas_layout` | Ordena el lienzo: `auto` (por el grafo), `row` / `column` (por nº de orden), `grid` 🆕 4.28 (por la celda `cell: {col, row}` de cada nodo — 1,1 arriba a la izquierda; sin celda no se mueven), `collapse_all`, `expand_all`. Las cajas 📌 no se mueven |
| `whiteboard_update` | Dibuja en la **pizarra** (`rect`, `ellipse`, `arrow`, `line`, `pen`, `text`, mismas coordenadas que los nodos): `mode` `add` / `replace` / `clear`. Se guarda con el flow como `drawings` |

### Proyecto `flows/` (enlaza con el panel Proyecto y con el CLI)

La carpeta `flows/` del servidor es el proyecto (en Docker `/app/flows`, o `FLOW_FLOWS_DIR`). Estas tools hacen lo mismo que el panel **Proyecto** de la web:

| Tool | Qué hace |
|------|----------|
| `flow_files_list` | Lista los `.flow.json` con metadatos (ruta, carpeta, tamaño, fecha, nombre del flow, nº de nodos) — lo mismo que muestra el panel |
| `flow_open` 🆕 4.26 | Abre un fichero en una pestaña **enlazada** al fichero (ids de nodos conservados); si ya está abierto, la activa. Después `flow_save` / Ctrl+S escriben en él |
| `flow_save` | Guarda la pestaña en el proyecto **como Ctrl+S**: la pestaña queda enlazada al fichero y deja de estar «sin guardar». Sin `fileName` usa el fichero enlazado (o el nombre del flow); admite subcarpetas (`int/login`) |
| `flow_file_delete` 🆕 4.26 | Borra un fichero del proyecto; si estaba abierto cierra su pestaña |
| `flow_file_read` | Lee un `.flow.json` como JSON sin abrirlo |

`flow_state` devuelve por pestaña `filePath` (fichero enlazado) y `dirty` (cambios sin guardar), y por nodo `position`, `collapsed`, `order` y `pinned`; las notas incluyen `renderMode`, contenido y scripts; `drawings` es el nº de dibujos de la pizarra.

## Flujos de trabajo típicos

**La IA construye mientras miras** — abre la web y pide:

> «Crea un flow que haga POST /login con estas credenciales, extraiga el token y llame a
> GET /users con Authorization Bearer. Ejecútalo y dime qué devuelve.»

El agente encadena `flow_create` → `node_add_request` ×2 → `nodes_connect` → `flow_run` y te
lee los resultados. Tú lo ves todo en el canvas y puedes retocar cualquier nodo a mano.

**De la web a CI**: cuando el flow esté fino, `flow_save` lo deja en `flows/` y tu pipeline
lo ejecuta con `docker exec flow node cli/run-flow.js --flow flows/mi-flow.flow.json`.

**Trabajar sobre el proyecto** (4.26): `flow_files_list` → `flow_open` (la pestaña queda enlazada al
fichero) → editar con `node_update` / `node_add_*` → `flow_save` (sin nombre: mismo fichero). El usuario
ve lo mismo que si hubiera pulsado Ctrl+S. `flow_overwrite` queda para cargar un documento que no está
en `flows/` (copia sin enlazar).

**Documentar encima del lienzo**: `node_add_info` con `renderMode: "mermaid"` para el esquema,
`whiteboard_update` para rodear/señalar grupos de cajas y `node_focus` para llevar al usuario a un nodo.
En las notas, `[[otro-flow#Nodo]]` enlaza flows del proyecto.

**Verificar un desarrollo con BBDD**: `node_add_sql` para sembrar/consultar datos +
`node_run` sobre el nodo SQL para lanzar la cadena SQL→HTTP con las variables extraídas.

## Troubleshooting

| Error | Solución |
|-------|----------|
| *No flow-test web tab is connected* | Abre http://localhost:9998 en un navegador (la pestaña se conecta sola) |
| 403 al conectar desde otra máquina | Es el modo por defecto (solo local): usa `FLOW_MCP_TOKEN` o `FLOW_MCP_ALLOW_LAN=true` |
| 401 Unauthorized | Hay `FLOW_MCP_TOKEN` definido: configura el bearer en tu cliente MCP |
| `flow_run` responde `finished:false` | Lee el `reason`: pestaña ya ejecutando, o el flow no tiene nodos HTTP (usa `node_run` sobre el nodo SQL raíz) |
| El comando caduca (*did not answer within…*) | La pestaña se cerró a mitad, o el run superó los 10 min del puente |
