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

## Las 39 tools

### Observar

| Tool | Qué hace |
|------|----------|
| `flow_state` | Estado vivo: pestañas + detalle de una (nodos con status, posición, `order`/`cell`/`pinned`/`disabled`, resumen de respuesta; conexiones con su id y behavior; variables de entorno y runtime; desde 4.33 también `globalVariables`, `sqlConnections` y `viewSettings`; desde 4.48 `settings` = configuración guardada en el flow: `view`, `hideConnections`, `whiteboardStyle`, `viewport`) |
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
| `node_add_info` 🆕 4.24 | Nota: texto (`renderMode: "text"`), diagrama Mermaid (`"mermaid"`, `content` = código, admite `{{variables}}`) o captura (`"image"` + `imageSrc`). Desde 4.26 acepta `scripts` (`[{varName, code, when?}]` — se ejecutan al Run Flow; 4.51: `when: "after"` corre al terminar el run con todas las extracciones en `vars`, por defecto antes de la primera petición; todo script recibe `vars`), `order` y `pinned` |
| `node_add_web` 🆕 4.26 | Nodo Web (URL / modo Live) |
| `node_update` | Actualiza campos de **cualquier** nodo: nombre, `position`, `collapsed`, `order`, `cell` (celda `{col,row}`), `pinned`, `disabled` 🆕 4.31 (se salta al ejecutar); HTTP `curl`/`extractions`; SQL `query`/conexión/`extractions`; nota `content`/`renderMode`/`imageSrc`/`scripts`; web `url`. Ignora campos de solo lectura (status, response…) |
| `node_delete` | Borra un nodo y sus conexiones |
| `nodes_connect` | Conecta origen → destino (`next` / `on_error` / `parallel` / `none`); `delayMs` 🆕 4.34 = pausa antes de lanzar el destino |
| `connection_delete` | Borra una conexión |
| `variables_set` | Variables de entorno de la pestaña (para `{{var}}`) |
| `global_variables_set` 🆕 4.33 | Variables **globales** (compartidas por todos los flows; precedencia global < entorno < runtime). Se leen en `flow_state.globalVariables` |
| `tab_rename` 🆕 4.33 | Renombra la pestaña / el flow (el `name` que se guarda en el `.flow.json`) |
| `connection_update` 🆕 4.33 | Cambia el comportamiento de una conexión existente (`next` / `on_error` / `parallel` / `none`) sin borrarla; los ids salen de `flow_state.connections` |
| `sql_connections_list` 🆕 4.33 | Perfiles de conexión SQL configurados en la web (id, nombre, motor, host, base — nunca la contraseña) para usarlos como `connectionProfileId` |
| `tab_close` 🆕 4.23 | Cierra una pestaña (`tabId` obligatorio; no cierra una pestaña que esté ejecutando) |

### Ejecutar

| Tool | Qué hace |
|------|----------|
| `flow_run` | Ejecuta el flow (el botón «Run Flow») y **espera al resultado**: run completo + variables extraídas. Primero los **scripts JS de las notas**, después **requests y SQL** en el mismo orden topológico (4.34; entradas `JS`, `SQL` y HTTP en el run), respetando las pausas `delayMs` de las flechas. Si no ejecuta nada (pestaña ocupada, flow sin nodos HTTP ni SQL) responde `finished:false` con el motivo |
| `node_run` | Ejecuta un nodo de **cualquier tipo** y su cadena descendente hacia nodos de **cualquier tipo** (4.34): `next` si fue bien, `on_error` si falló, `parallel` siempre; respeta `delayMs`. Una nota lanza sus scripts; una web se recarga |
| `flow_reset` 🆕 4.26 | Limpia respuestas, resultados, estados y variables runtime de la pestaña (el «Reset») |

### Vigilancia en el servidor 🆕 5.17

| Tool | Qué hace |
|------|----------|
| `flow_monitor` | `mode: "set"` programa el flow **en el servidor** (`intervalMin`, sin navegador) y define `rules` sobre las variables con las que termina el run — `{variable, op, value, message?}` con `op` `gt` `gte` `lt` `lte` `equals` `not-equals` `contains` `not-contains` `changed`. El servidor hace POST a `notifyUrl` (admite `{{secret:X}}`) **solo cuando una regla se dispara, se recupera o el valor cambia**, y cuando el run falla. Fusiona con lo que haya; `rules` reemplaza la lista. **Se publica al guardar**: llama a `flow_save` después. `"clear"` lo quita. `"status"` **no necesita pestaña web**: estado de cada regla (`active`, `value`, `since`), último aviso y los 10 últimos runs (de todos los flows vigilados o de `path`) |

### Lienzo 🆕 4.26

| Tool | Qué hace |
|------|----------|
| `node_focus` | Centra el lienzo en un nodo (por `nodeId` o `nodeName`) y lo resalta ~2 s — para señalar algo al usuario |
| `canvas_layout` | Ordena el lienzo: `auto` (por el grafo), `row` / `column` (por nº de orden), `grid` 🆕 4.28 (por la celda `cell: {col, row}` de cada nodo — 1,1 arriba a la izquierda; sin celda no se mueven), `separate` 🆕 4.33 (aparta las cajas solapadas hasta la separación mínima de la vista; devuelve `moved`), `collapse_all`, `expand_all`, `pin_all` / `unpin_all` 🆕 4.29 (📌 en todas las cajas / liberarlas). Las cajas 📌 no se mueven |
| `whiteboard_update` (4.50) | Acepta también `type: "image"` con `imageSrc` (imagen o GIF; w/h medidos en el navegador si faltan) |
| `flow_background` 🆕 4.49 | Pone o quita la **imagen de fondo** del flow (`settings.background`): `imageSrc` (`/flow-assets/…`, http(s) o data), `x`, `y`, `width`, `height`, `opacity`; `mode: clear` la elimina. Las imágenes se suben con `POST /workspace/asset {name, data base64}` |
| `view_settings` 🆕 4.33 | Lee o cambia la **vista** del lienzo del usuario (Config ▸ Vista): `nodeScale` 0.3–1.5, `compactMode` auto/always/never, `compactThreshold`, `compactSize`, `hoverInfo`, `clickOpens`, `minGap`, `autoSeparate`. Sin `settings` devuelve la actual. Desde 4.48 edita la vista **del flow activo** (`settings.view`, se guarda con el `.flow.json`) |
| `whiteboard_update` | Dibuja en la **pizarra** (`rect`, `ellipse`, `arrow`, `line`, `pen`, `text`, mismas coordenadas que los nodos): `mode` `add` / `replace` / `clear`; los `text` admiten `font` `hand` \| `sketch` \| `excali` \| `indie` \| `marker` \| `draft` \| `architect` \| `note` \| `sans` \| `mono` (4.35) y se miden solos si no llevan `w`/`h`. Se guarda con el flow como `drawings` |

### Proyecto `flows/` (enlaza con el panel Proyecto y con el CLI)

La carpeta `flows/` del servidor es el proyecto (en Docker `/app/flows`, o `FLOW_FLOWS_DIR`). Estas tools hacen lo mismo que el panel **Proyecto** de la web:

| Tool | Qué hace |
|------|----------|
| `flow_files_list` | Lista los `.flow.json` con metadatos (ruta, carpeta, tamaño, fecha, nombre del flow, nº de nodos) — lo mismo que muestra el panel |
| `flow_open` 🆕 4.26 | Abre un fichero en una pestaña **enlazada** al fichero (ids de nodos conservados); si ya está abierto, la activa. Desde la 5.19.2, si ya estaba abierto **lo recarga desde el disco** (no te quedas con una copia vieja). Después `flow_save` / Ctrl+S escriben en él |
| `flow_save` | Guarda la pestaña en el proyecto **como Ctrl+S**: la pestaña queda enlazada al fichero y deja de estar «sin guardar». Sin `fileName` usa el fichero enlazado (o el nombre del flow); admite subcarpetas (`int/login`). Desde la 5.19.2, si el fichero cambió en disco desde que se abrió, **no lo pisa** (409) salvo `force: true` |
| `flow_file_delete` 🆕 4.26 | Borra un fichero del proyecto; si estaba abierto cierra su pestaña |
| `flow_file_read` | Lee un `.flow.json` como JSON sin abrirlo |

`flow_state` devuelve por pestaña `filePath` (fichero enlazado) y `dirty` (cambios sin guardar), y por nodo `position`, `collapsed`, `order` y `pinned`; las notas incluyen `renderMode`, contenido y scripts; `drawings` es el nº de dibujos de la pizarra.

### Correo de prueba 🆕 4.37 (funcionan sin pestaña web)

El SMTP de prueba de **Config ▸ Correo** ([08 Paneles](manual/08-paneles.md#correo-de-prueba--smtp-embebido--436)) se controla entero desde el MCP: la IA lo arranca, crea buzones y **verifica que tu servicio envió el correo** (y saca el OTP o el enlace de confirmación) sin que tú toques nada.

| Tool | Qué hace |
|------|----------|
| `mail_state` | Estado: `running`, puerto, dominio, `acceptAny`, `autoStart`, buzones con contador, último correo y `provider` (smtp\|mailtm), nº de correos guardados, config de extracción (`codeLength`, `codeRegex`) y bloque `mailtm` (activado, **dominio activo** — rota —, `pollMs`, cuentas, último error) |
| `mail_server` | `action: start` (opcional `port`; recuerda `autoStart`), `stop`, `configure` (`domain`, `acceptAny`, `autoStart`, `port` con el SMTP parado; 4.42: `mailtmEnabled`, `mailtmPollMs`, `codeLength`, `codeRegex`). Si el puerto está ocupado vuelve `state.error`, no un error de tool |
| `mail_address` | `action: add` (`local` → `local@dominio`, o `address` completa; `provider: mailtm` crea la **cuenta remota real** en el dominio activo de mail.tm) / `delete` (borra el buzón y sus correos; en mail.tm borra también la cuenta) |
| `mail_messages` | `action: list` (resúmenes; filtros `to`, `from`, `subject`, `since` ms, `q`, `limit`), `latest` (el último completo que cumpla el filtro: `{found, message}` con **`code`/`codeSource`** ya extraídos — keyword·fallback·regex; con mail.tm activo, preguntar por `to=x@<dominio-activo>` auto-crea el buzón y sondea el remoto con throttle), `read` (`id`; `raw: true` = origen RFC822), `delete` (`id`), `clear` (todos los que cumplan el filtro) |

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

**Botones, páginas y acciones dibujados por el flow** (5.14): un `node_add_info` con un script
`when: "after"` puede crear una barra de botones o un diálogo sobre FlowTest y, desde esos botones,
llamar al propio MCP por HTTP — `POST /mcp`, JSON-RPC `tools/call`, cabecera
`Accept: application/json, text/event-stream`, sin sesión — para reejecutar (`flow_run`), cambiar
Variables (`variables_set`) o abrir otro flow (`flow_open`). Con `FLOW_MCP_TOKEN` el script debe
mandar `Authorization: Bearer`. Ejemplos y contrato de higiene (una instancia, Shadow DOM, sin
localStorage/Notification/bucles, botón ✕) en el manual (04 ▸ «Scripts que dibujan botones…») y en la
galería `scripts/04 Divisas` y `scripts/05 Tiempo`. Desde la 5.15 solo los **flows de confianza**
ejecutan scripts en la página: los abiertos de fichero, cloud o import van a un sandbox sin DOM ni
red y la app pregunta al usuario antes de ejecutar un script que la necesite (manual 04 ▸ «Sandbox y
flows de confianza»); un flow creado con `flow_create` en la pestaña del usuario nace de confianza.

**Verificar un desarrollo con BBDD**: `node_add_sql` para sembrar/consultar datos +
`node_run` sobre el nodo SQL para lanzar la cadena SQL→HTTP con las variables extraídas.

## Troubleshooting

| Error | Solución |
|-------|----------|
| *No flow-test web tab is connected* | Abre http://localhost:9998 en un navegador (la pestaña se conecta sola) |
| 403 al conectar desde otra máquina | Es el modo por defecto (solo local): usa `FLOW_MCP_TOKEN` o `FLOW_MCP_ALLOW_LAN=true` |
| 401 Unauthorized | Hay `FLOW_MCP_TOKEN` definido: configura el bearer en tu cliente MCP |
| `flow_run` responde `finished:false` | Lee el `reason`: pestaña ya ejecutando, o el flow no tiene nodos HTTP ni SQL |
| El comando caduca (*did not answer within…*) | La pestaña se cerró a mitad, o el run superó los 10 min del puente |

### Novedades 5.3

| Tool | Cambio |
|------|--------|
| `whiteboard_update` | Operaciones por **elemento suelto**: `mode: "list"` (todos los elementos con su id), `"edit"` (merge parcial por `id`; los textos se re-miden) y `"delete"` (por `ids`) — antes solo add/replace/clear |
| `node_add_request` | Acepta `asserts: [{kind: status\|jsonpath\|time, expected, path, op, value, maxMs}]` — checks tras la respuesta; si fallan, el nodo queda en error y el CLI sale ≠ 0 |
| `variables_set` | `environment: "prod"` escribe en un **entorno con nombre** (se crea si falta) y `activate: true` lo deja activo |
| `flow_state` | Devuelve `environments` (nombres y claves de cada set) y `activeEnvironment` |
