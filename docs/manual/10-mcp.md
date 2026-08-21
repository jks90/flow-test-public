# 🤖 10 · El MCP embebido

Flow-test lleva un **servidor MCP dentro** (endpoint `/mcp`): cualquier cliente MCP —Claude Code, Claude Desktop— puede **construir, editar y ejecutar flows en tu canvas mientras miras**. La IA pinta las cajitas en directo en la última pestaña del navegador que esté conectada (la «controladora»).

## Conectarlo a Claude Code

```bash
# contra el dev (npm run dev)
claude mcp add --transport http flow-test http://localhost:3001/mcp

# contra el contenedor Docker
claude mcp add --transport http flow-test http://localhost:9998/mcp
```

Y en una conversación: *«créame un flow que haga login en mi API y liste los usuarios»* — verás aparecer las cajitas, conectarse y ejecutarse solas.

## Las 37 tools, por grupos

| Grupo | Tools | Para qué |
|-------|-------|----------|
| **Observar** | `bridge_status`, `flow_state`, `console_read`, `runs_read` | Ver pestañas, nodos (posición, orden, 📌, notas, web, dibujos, fichero enlazado, sin guardar), consola e historial |
| **Pestañas** | `flow_create`, `tab_select`, `tab_close`, `flow_overwrite` | Crear, activar, cerrar o reemplazar pestañas |
| **Construir** | `node_add_request`, `node_add_sql`, `node_add_info`, `node_add_web`, `node_update`, `node_delete`, `nodes_connect`, `connection_delete`, `variables_set` | Crear/editar cualquier tipo de nodo (HTTP, SQL, nota/Mermaid/captura con scripts, web), conexiones y variables |
| **Ejecutar** | `flow_run`, `node_run`, `flow_reset` | Como pulsar Run Flow / el ▶ de un nodo (devuelve resultados) / Reset. `flow_run` ejecuta antes los scripts JS de las notas |
| **Lienzo** 🆕 4.26 | `node_focus`, `canvas_layout`, `whiteboard_update` | Centrar y resaltar un nodo, ordenar el lienzo (auto / fila / columna / **cuadrícula** por `cell {col,row}` 🆕 4.28 / separar solapes 🆕 4.33 / colapsar / expandir / fijar-liberar todas 🆕 4.29), **`view_settings`** (tamaño de nodos, modo compacto, separación 🆕 4.33) y dibujar en la pizarra |
| **Proyecto `flows/`** | `flow_files_list`, `flow_open`, `flow_save`, `flow_file_delete`, `flow_file_read` | Lo mismo que el panel Proyecto: listar con metadatos, abrir enlazado al fichero, guardar como Ctrl+S (la pestaña queda enlazada), borrar, leer |
| **Correo de prueba** 🆕 4.37 | `mail_state`, `mail_server`, `mail_address`, `mail_messages` | Arrancar/parar/configurar el SMTP de prueba ([08 Paneles](08-paneles.md#correo-de-prueba--smtp-embebido--436)), crear buzones y leer los correos recibidos (`latest` con filtros `to`/`subject`) para verificar que tu servicio los envió y sacar el OTP/enlace. No necesitan pestaña web |

> `flow_overwrite` acepta un `.flow.json` completo — también con nodos Captura y con los dibujos de la pizarra (`drawings`): una IA puede cargarte una documentación entera, anotada, en la pestaña.

## Seguridad

- Por defecto solo acepta clientes **locales** (localhost).
- `FLOW_MCP_TOKEN=xxx` exige `Authorization: Bearer xxx`.
- `FLOW_MCP_ALLOW_LAN=true` abre el acceso a la LAN (usa el token si lo haces).

> [!WARNING]
> **La pestaña controladora**
> El puente controla **la última pestaña conectada**. Si tienes abiertas la web del dev (5173) y la del contenedor (9998), cada servidor tiene su propio puente — asegúrate de a cuál apunta tu cliente MCP, o la IA pintará «en la otra».
