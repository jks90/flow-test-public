# Ejemplos

| Fichero | Qué demuestra |
|---------|---------------|
| `api-login-cadena.flow.json` | Cadena HTTP clásica: login → extraer `{{token}}` → llamada autenticada → detalle con id extraído |
| `sql-verificacion.flow.json` | Nodo SQL que extrae `userId`/`email` de la BBDD y alimenta una verificación HTTP |
| `sql-connections.example.json` | Perfiles de conexión SQL (cópialo como `sql-connections.json` junto a tus flows) |

## Probarlos

```bash
# Copiar al contenedor y ejecutar (ajusta variables con --var)
docker cp api-login-cadena.flow.json flow:/tmp/
docker exec flow node cli/run-flow.js --flow /tmp/api-login-cadena.flow.json \
  --var apiBase=http://host.docker.internal:8080 --no-report
```

O cárgalos en el canvas vía MCP: `flow_file_read` + `flow_overwrite` (si están en `flows/`),
o pídele a tu agente que los recree con `node_add_request`/`nodes_connect`.
