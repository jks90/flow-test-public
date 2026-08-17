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

## Las 18 tools, por grupos

| Grupo | Tools | Para qué |
|-------|-------|----------|
| **Observar** | `bridge_status`, `flow_state`, `console_read`, `runs_read` | Ver pestañas, nodos, consola e historial |
| **Construir** | `flow_create`, `flow_overwrite`, `node_add_request`, `node_add_sql`, `node_update`, `node_delete`, `nodes_connect`, `connection_delete`, `variables_set` | Crear/editar el flow en el canvas |
| **Ejecutar** | `flow_run`, `node_run` | Como pulsar Run Flow / el ▶ de un nodo (devuelve resultados) |
| **Disco** | `flow_save`, `flow_files_list`, `flow_file_read` | Leer/escribir `flows/*.flow.json` |

> `flow_overwrite` acepta un `.flow.json` completo — también con nodos Captura 🆕: una IA puede cargarte una documentación entera con screenshots en la pestaña.

## Seguridad

- Por defecto solo acepta clientes **locales** (localhost).
- `FLOW_MCP_TOKEN=xxx` exige `Authorization: Bearer xxx`.
- `FLOW_MCP_ALLOW_LAN=true` abre el acceso a la LAN (usa el token si lo haces).

> [!WARNING]
> **La pestaña controladora**
> El puente controla **la última pestaña conectada**. Si tienes abiertas la web del dev (5173) y la del contenedor (9998), cada servidor tiene su propio puente — asegúrate de a cuál apunta tu cliente MCP, o la IA pintará «en la otra».
